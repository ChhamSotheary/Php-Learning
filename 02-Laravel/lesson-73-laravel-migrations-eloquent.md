# មេរៀនទី ៧៣៖ Laravel Database Migrations និង Eloquent ORM

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីតួនាទីរបស់ database migration។
- បង្កើត និងដំណើរការ migration ដើម្បីបង្កើត table។
- បង្កើត Eloquent model និងប្រើវាសម្រាប់ CRUD។
- កំណត់ mass assignment ដែលអនុញ្ញាត។
- យល់ពីការប្រុងប្រយ័ត្នពេល rollback ឬកែ schema។

## ១. Migration ជាអ្វី?

Migration ជាកូដ PHP សម្រាប់កំណត់ការផ្លាស់ប្តូរ database schema។ វាដូចជា version control សម្រាប់តារាង និង column ដូច្នេះក្រុមការងារអាចអនុវត្តការផ្លាស់ប្តូរដូចគ្នាលើ database ផ្សេងៗបាន។

Migration ស្ថិតនៅ `database/migrations/` ហើយឈ្មោះ file មាន timestamp ដើម្បីកំណត់លំដាប់ដំណើរការ។

បង្កើត migration សម្រាប់ courses table៖

```bash
php artisan make:migration create_courses_table
```

កែ file ថ្មីដែល Laravel បង្កើត៖

```php
<?php

use Illuminate\\Database\\Migrations\\Migration;
use Illuminate\\Database\\Schema\\Blueprint;
use Illuminate\\Support\\Facades\\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('courses', function (Blueprint $table) {
            $table->id();
            $table->string('title', 120);
            $table->text('description');
            $table->string('level', 20);
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('courses');
    }
};
```

- `up()` ពិពណ៌នាការផ្លាស់ប្តូរដែលអនុវត្តទៅមុខ។
- `down()` ពិពណ៌នារបៀបបញ្ច្រាសការផ្លាស់ប្តូរនោះ។
- `$table->id()` បង្កើត primary key។
- `$table->timestamps()` បង្កើត `created_at` និង `updated_at`។

បន្ទាប់ពីកំណត់ connection ក្នុង `.env` រួច រត់ migration៖

```bash
php artisan migrate
```

មើល migration ដែលបានអនុវត្ត៖

```bash
php artisan migrate:status
```

ពាក្យបញ្ជា `php artisan migrate:rollback` បញ្ច្រាស batch ចុងក្រោយ។ ត្រូវយល់ពីផលប៉ះពាល់មុនរត់វា ព្រោះ `down()` អាចលុប table ឬទិន្នន័យដែលពាក់ព័ន្ធ។ កុំរត់ `migrate:fresh` លើ database ដែលមានទិន្នន័យចាំបាច់៖ វាលុបតារាងទាំងអស់ ហើយបង្កើតឡើងវិញ។

## ២. Eloquent Model ជាអ្វី?

Eloquent ជា ORM របស់ Laravel។ Model តំណាងឱ្យ record ក្នុង table និងអនុញ្ញាតឱ្យធ្វើ query ដោយប្រើ PHP objects ជំនួសការសរសេរ SQL គ្រប់ករណី។

បង្កើត Model និង migration ព្រមគ្នា៖

```bash
php artisan make:model Course --migration
```

តាមលំនាំ Laravel ភ្ជាប់ Model `Course` ទៅ table `courses`។ កំណត់ fields ដែលអនុញ្ញាតឱ្យ mass assign៖

```php
<?php

namespace App\\Models;

use Illuminate\\Database\\Eloquent\\Model;

class Course extends Model
{
    protected $fillable = [
        'title',
        'description',
        'level',
    ];
}
```

បញ្ជី `$fillable` គឺជាការការពារសម្រាប់ mass assignment។ វាមិនជំនួស validation ទេ៖ ត្រូវ validate request ជាមុន ហើយបញ្ចូលតែ validated fields ដែលអនុញ្ញាត។

## ៣. CRUD ដោយ Eloquent

បង្កើត record៖

```php
$course = Course::create([
    'title' => 'PHP មូលដ្ឋាន',
    'description' => 'រៀនមូលដ្ឋាន PHP និងការសរសេរកម្មវិធី។',
    'level' => 'beginner',
]);
```

អានទិន្នន័យ៖

```php
$allCourses = Course::all();
$course = Course::find(1);
$beginnerCourses = Course::where('level', 'beginner')->get();
```

កែប្រែ record៖

```php
$course = Course::findOrFail(1);
$course->update([
    'title' => 'PHP សម្រាប់អ្នកចាប់ផ្តើម',
]);
```

លុប record៖

```php
$course = Course::findOrFail(1);
$course->delete();
```

`findOrFail()` បង្កើត Model Not Found exception ប្រសិនបើមិនមាន record។ ក្នុង HTTP request Laravel ជាទូទៅបម្លែងវាទៅជា 404 response។

## ៤. ភ្ជាប់ Validation និង Model

ក្នុង Controller អាច validate មុនបង្កើត Model៖

```php
use App\\Models\\Course;
use Illuminate\\Http\\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'title' => ['required', 'string', 'max:120'],
        'description' => ['required', 'string', 'max:5000'],
        'level' => ['required', 'in:beginner,intermediate,advanced'],
    ]);

    $course = Course::create($validated);

    return response()->json($course, 201);
}
```

ការពារមានពីរជំហានផ្សេងគ្នា៖

1. **Validation** ពិនិត្យថា input ត្រឹមត្រូវតាម rules របស់កម្មវិធី។
2. **Mass assignment configuration** កំណត់ថា Model fields ណាខ្លះអាចបញ្ចូលតាម array។

កុំប្រើ `$request->all()` ដើម្បីបង្កើត ឬកែ Model ព្រោះ client អាចបញ្ជូន fields ដែលមិនគួរអនុញ្ញាត។

## ៥. លំហាត់អនុវត្ត

1. បង្កើត migration `create_courses_table` មាន `title`, `description`, `level` និង timestamps។
2. បង្កើត Model `Course` ហើយកំណត់ `$fillable`។
3. រត់ migration ក្នុង local development database។
4. សរសេរ Controller method សម្រាប់បង្កើត Course ដោយ validate fields មុន។
5. សរសេរ query ស្វែងរក Courses ដែលមាន level ជា `beginner`។
6. សាកល្បង `php artisan migrate:status` ហើយកត់ត្រាលទ្ធផល។

## ៦. កំហុសដែលជួបញឹកញាប់

- **ភ្លេចកំណត់ database connection**៖ ពិនិត្យ `DB_CONNECTION` និង credentials ក្នុង environment។
- **កែ migration ដែលបានរត់នៅក្រុម ឬ production រួច**៖ ជាទូទៅបង្កើត migration ថ្មីសម្រាប់ schema change ដើម្បីរក្សាប្រវត្តិច្បាស់។
- **ភ្លេចពាក្យ `down()`**៖ ត្រូវកំណត់ rollback behavior ឱ្យសមនឹង `up()`។
- **MassAssignmentException**៖ ពិនិត្យថា fields ដែលចង់បញ្ចូលមានក្នុង `$fillable`។
- **គិតថា `$fillable` ជា validation**៖ វាកំណត់ fields ដែលអាច mass assign ប៉ុណ្ណោះ; វាមិនពិនិត្យប្រភេទ ឬតម្លៃ input ទេ។
- **លុបទិន្នន័យដោយ command សម្អាត database**៖ អានឈ្មោះ command និង environment ឱ្យច្បាស់មុនដំណើរការ។

## សំណួរត្រួតពិនិត្យ

1. តើ `up()` និង `down()` មានតួនាទីអ្វី?
2. Migration ជួយក្រុមការងារយ៉ាងដូចម្ដេច?
3. Model `Course` ត្រូវនឹង table ឈ្មោះអ្វីតាម convention?
4. តើ `$fillable` ការពារអ្វី?
5. ហេតុអ្វីត្រូវ validate មុនប្រើ `Course::create()`?

## ឯកសារយោង

- [Laravel 13 Database Migrations](https://laravel.com/docs/13.x/migrations)
- [Laravel 13 Eloquent: Getting Started](https://laravel.com/docs/13.x/eloquent)
