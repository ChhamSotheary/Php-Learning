# មេរៀនទី ៧៤៖ Eloquent Relationships និង Laravel API Resources

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- តំណាងទំនាក់ទំនងរវាង Eloquent Models។
- ប្រើ `belongsTo` និង `hasMany` សម្រាប់ one-to-many។
- ទាញទិន្នន័យទាក់ទងដោយ eager loading។
- បង្កើត API Resource ដើម្បីគ្រប់គ្រង JSON output។
- កាត់បន្ថយហានិភ័យ N+1 query និងការបង្ហាញ fields មិនចាំបាច់។

## ១. ឧទាហរណ៍ទំនាក់ទំនង

សូមប្រើគំរូ Course និង Lesson៖ Course មួយមាន Lessons ច្រើន ហើយ Lesson មួយស្ថិតនៅក្នុង Course មួយ។ Table `lessons` គួរមាន `course_id` ជា foreign key។

Migration អាចកំណត់ foreign key ដូចនេះ៖

```php
Schema::create('lessons', function (Blueprint $table) {
    $table->id();
    $table->foreignId('course_id')->constrained()->cascadeOnDelete();
    $table->string('title');
    $table->unsignedInteger('position');
    $table->timestamps();
});
```

`constrained()` ប្រើ convention ដើម្បីយោងទៅ `courses.id`។ `cascadeOnDelete()` មានន័យថា database លុប lessons របស់ course ពេល course ត្រូវបានលុប។ ប្រើវាតែពេលច្បាប់ទិន្នន័យរបស់កម្មវិធីពិតជាចង់បានលទ្ធផលនេះ។

## ២. កំណត់ Eloquent Relationships

ក្នុង `app/Models/Course.php`៖

```php
<?php

namespace App\\Models;

use Illuminate\\Database\\Eloquent\\Model;
use Illuminate\\Database\\Eloquent\\Relations\\HasMany;

class Course extends Model
{
    public function lessons(): HasMany
    {
        return $this->hasMany(Lesson::class);
    }
}
```

ក្នុង `app/Models/Lesson.php`៖

```php
<?php

namespace App\\Models;

use Illuminate\\Database\\Eloquent\\Model;
use Illuminate\\Database\\Eloquent\\Relations\\BelongsTo;

class Lesson extends Model
{
    public function course(): BelongsTo
    {
        return $this->belongsTo(Course::class);
    }
}
```

Convention របស់ Laravel សន្មតថា foreign key គឺ `course_id`។ បើ schema របស់អ្នកប្រើឈ្មោះខុសពី convention អាចផ្ដល់ឈ្មោះ foreign key ជា argument ទីពីរ។

## ៣. ប្រើទំនាក់ទំនង

អាន Course និង Lessons របស់វា៖

```php
$course = Course::findOrFail(1);
$lessons = $course->lessons;
```

អាន Course របស់ Lesson៖

```php
$lesson = Lesson::findOrFail(10);
$course = $lesson->course;
```

បង្កើត Lesson តាម relationship ដើម្បីឱ្យ Laravel កំណត់ `course_id` ដោយស្វ័យប្រវត្តិ៖

```php
$course = Course::findOrFail(1);

$lesson = $course->lessons()->create([
    'title' => 'Routing មូលដ្ឋាន',
    'position' => 1,
]);
```

ត្រូវកំណត់ `$fillable` របស់ Lesson ឱ្យមានតែ fields ដែលអនុញ្ញាត ដូចជា `title` និង `position`។ ឱ្យ relationship កំណត់ foreign key ជំនួសការទទួល `course_id` ពី client ដោយផ្ទាល់។

## ៤. Eager Loading និង N+1 Query

កូដខាងក្រោមអាចបង្កើត query មួយសម្រាប់ Courses ហើយ query បន្ថែមមួយសម្រាប់ Lessons នៃ Course នីមួយៗ៖

```php
$courses = Course::all();

foreach ($courses as $course) {
    echo $course->lessons->count();
}
```

នេះគេហៅថាបញ្ហា N+1 query។ ប្រើ eager loading ដើម្បីទាញទំនាក់ទំនងជាមុន៖

```php
$courses = Course::with('lessons')->get();

foreach ($courses as $course) {
    echo $course->lessons->count();
}
```

អាចជ្រើស fields ដើម្បីកាត់បន្ថយទិន្នន័យ ប៉ុន្តែត្រូវរួមបញ្ចូល key ដែលត្រូវការសម្រាប់ភ្ជាប់ Models ដូចជា primary key និង foreign key។

## ៥. API Resource ជាអ្វី?

API Resource ជាស្រទាប់បម្លែង Model ទៅជា JSON response។ វាអនុញ្ញាតឱ្យជ្រើស fields ដែលបង្ហាញ និងទម្រង់ payload ដោយមិនបញ្ចេញគ្រប់ attribute របស់ Model តាមស្វ័យប្រវត្តិ។

បង្កើត Resource៖

```bash
php artisan make:resource CourseResource
```

កែ `app/Http/Resources/CourseResource.php`៖

```php
<?php

namespace App\\Http\\Resources;

use Illuminate\\Http\\Request;
use Illuminate\\Http\\Resources\\Json\\JsonResource;

class CourseResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'level' => $this->level,
            'lessons' => LessonResource::collection(
                $this->whenLoaded('lessons')
            ),
        ];
    }
}
```

បង្កើត `LessonResource` ដើម្បីជ្រើស fields របស់ Lesson៖

```bash
php artisan make:resource LessonResource
```

```php
<?php

namespace App\\Http\\Resources;

use Illuminate\\Http\\Request;
use Illuminate\\Http\\Resources\\Json\\JsonResource;

class LessonResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'position' => $this->position,
        ];
    }
}
```

បញ្ជូន Resource ពី Controller៖

```php
use App\\Http\\Resources\\CourseResource;
use App\\Models\\Course;

public function show(string $id): CourseResource
{
    $course = Course::with('lessons')->findOrFail($id);

    return new CourseResource($course);
}
```

`whenLoaded('lessons')` បញ្ចូល lessons តែពេល Controller បាន load relationship រួច។ វាជួយកុំឱ្យ Resource ទាញ relationship ដោយមិនដឹងខ្លួន និងអនុញ្ញាតឱ្យ route ជ្រើសរើស payload ដែលត្រូវការ។

## ៦. លំហាត់អនុវត្ត

1. បន្ថែម `lessons` table ដែលមាន foreign key ទៅ `courses`។
2. កំណត់ `Course::lessons()` និង `Lesson::course()`។
3. បង្កើត Lesson តាម `$course->lessons()->create(...)`។
4. ទាញ courses ជាមួយ lessons ដោយ `with('lessons')`។
5. បង្កើត CourseResource ដែលបង្ហាញតែ `id`, `title`, `level` និង Lessons ដែលបាន load។
6. ប្រៀបធៀប query behavior ពេលប្រើ និងមិនប្រើ eager loading។

## ៧. កំហុសដែលជួបញឹកញាប់

- **ឈ្មោះ foreign key មិនត្រូវ convention**៖ បញ្ជាក់ foreign key ឱ្យច្បាស់ក្នុង relationship។
- **ភ្លេច load relationship មុន Resource**៖ `whenLoaded()` នឹងមិនបញ្ចូល relationship ទេ បើមិនបាន eager load។
- **N+1 queries**៖ ប្រើ `with()` ឬ `load()` សម្រាប់ទិន្នន័យទំនាក់ទំនងដែលត្រូវបង្ហាញ។
- **បង្ហាញ Model ទាំងមូលក្នុង API**៖ ប្រើ Resource ដើម្បីកំណត់ fields ជាក់លាក់ និងកុំបញ្ចេញទិន្នន័យឯកជន។
- **ប្រើ cascade delete ដោយមិនពិនិត្យផលប៉ះពាល់**៖ ពិនិត្យថាតើត្រូវរក្សា child records ឬ audit history ដែរឬទេ។
- **បញ្ចូល foreign key ពី input ដោយផ្ទាល់**៖ រក្សា parent resource ដែលបាន authorize ហើយបង្កើត child តាម relationship។

## សំណួរត្រួតពិនិត្យ

1. តើ `hasMany` និង `belongsTo` ប្រើនៅ Model ណាខ្លះក្នុងឧទាហរណ៍?
2. ហេតុអ្វីត្រូវប្រើ eager loading?
3. API Resource ជួយអ្វីក្នុងការបង្កើត JSON?
4. `whenLoaded()` មានប្រយោជន៍ដូចម្ដេច?
5. ហេតុអ្វីមិនគួរបង្ហាញ Model attributes ទាំងអស់ដោយមិនជ្រើសរើស?

## ឯកសារយោង

- [Laravel 13 Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships)
- [Laravel 13 Eloquent API Resources](https://laravel.com/docs/13.x/eloquent-resources)
