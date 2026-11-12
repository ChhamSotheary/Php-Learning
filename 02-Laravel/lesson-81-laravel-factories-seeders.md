# មេរៀនទី ៨១៖ Laravel Model Factories និង Database Seeders

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីភាពខុសគ្នារវាង Factory និង Seeder។
- បង្កើត fake records សម្រាប់ local development និង tests។
- កំណត់ factory state សម្រាប់ data variations។
- រៀបចំ Seeder ច្រើន និងហៅវាពី DatabaseSeeder។
- ជៀសវាងការបញ្ចូល sample data ទៅ production ដោយចៃដន្យ។

## ១. Factory និង Seeder មានតួនាទីអ្វី?

- **Model Factory** កំណត់លំនាំសម្រាប់បង្កើត attributes និង records របស់ Model។
- **Database Seeder** រៀបចំថាត្រូវបញ្ចូលទិន្នន័យអ្វី និងហៅ factories ឬ seeders ផ្សេងៗតាមលំដាប់។

Factories មានប្រយោជន៍សម្រាប់ tests និង sample data។ Seeders មានប្រយោជន៍សម្រាប់ទិន្នន័យចាប់ផ្តើម ឬ development fixtures ដែលមានបំណងច្បាស់។

## ២. បង្កើត Course Factory

បង្កើត factory៖

```bash
php artisan make:factory CourseFactory --model=Course
```

កែ `database/factories/CourseFactory.php`៖

```php
<?php

namespace Database\\Factories;

use Illuminate\\Database\\Eloquent\\Factories\\Factory;

/**
 * @extends Factory<\\App\\Models\\Course>
 */
class CourseFactory extends Factory
{
    public function definition(): array
    {
        return [
            'title' => fake()->sentence(4),
            'description' => fake()->paragraph(),
            'level' => fake()->randomElement([
                'beginner',
                'intermediate',
                'advanced',
            ]),
        ];
    }
}
```

Model ត្រូវប្រើ `HasFactory` trait ដើម្បីហៅ `Course::factory()`។ Models របស់ Laravel ជាទូទៅមាន trait នេះរួច ប៉ុន្តែសូមពិនិត្យ Model របស់អ្នក។

## ៣. បង្កើត Models ពី Factory

បង្កើត Model ក្នុង memory ដោយមិនរក្សាទុក database៖

```php
$course = Course::factory()->make();
```

បង្កើត និងរក្សាទុក record មួយ៖

```php
$course = Course::factory()->create();
```

បង្កើតច្រើន records៖

```php
$courses = Course::factory()->count(10)->create();
```

កំណត់ attributes ជាក់លាក់ដែល override factory defaults៖

```php
$course = Course::factory()->create([
    'title' => 'PHP មូលដ្ឋាន',
    'level' => 'beginner',
]);
```

## ៤. Factory States

State ជាវិធីបង្កើត variation ដែលមានឈ្មោះ និងអាចប្រើឡើងវិញបាន។ ក្នុង CourseFactory បន្ថែម៖

```php
public function beginner(): static
{
    return $this->state(fn (array $attributes) => [
        'level' => 'beginner',
    ]);
}
```

ប្រើ state៖

```php
$course = Course::factory()->beginner()->create();
```

State ជួយកុំឱ្យ test និង seeder ចម្លង attributes ដដែលៗ។

## ៥. បង្កើត Seeder

បង្កើត seeder៖

```bash
php artisan make:seeder CourseSeeder
```

ក្នុង `database/seeders/CourseSeeder.php`៖

```php
<?php

namespace Database\\Seeders;

use App\\Models\\Course;
use Illuminate\\Database\\Seeder;

class CourseSeeder extends Seeder
{
    public function run(): void
    {
        Course::factory()
            ->count(12)
            ->create();
    }
}
```

ហៅ Seeder នេះពី `DatabaseSeeder`៖

```php
public function run(): void
{
    $this->call([
        CourseSeeder::class,
    ]);
}
```

រត់ seeders៖

```bash
php artisan db:seed
```

រត់ seeder មួយជាក់លាក់៖

```bash
php artisan db:seed --class=CourseSeeder
```

## ៦. ប្រុងប្រយ័ត្នចំពោះ Database

`php artisan migrate:fresh --seed` លុប tables ទាំងអស់ រត់ migrations ឡើងវិញ ហើយ seed ទិន្នន័យឡើងវិញ។ វាមានប្រយោជន៍សម្រាប់ local/test database ដែលអាចលុបទិន្នន័យបាន ប៉ុន្តែអាចបំផ្លាញទិន្នន័យសំខាន់ ប្រសិនបើរត់លើ database ខុស។

មុនប្រើ database command សូមពិនិត្យ environment និង connection។ Seeder ដែលបង្កើត default administrator ឬ credential ត្រូវប្រើ password hashing, កុំដាក់ password ពិតក្នុង source control, និងកំណត់ឱ្យដំណើរការតែក្នុង environment ដែលបានអនុញ្ញាត។

## ៧. ប្រើ Factory ក្នុង Test

Factories ធ្វើឱ្យ test រៀបចំ record បានខ្លី និងច្បាស់៖

```php
$course = Course::factory()->create([
    'level' => 'beginner',
]);

$this->assertDatabaseHas('courses', [
    'id' => $course->id,
    'level' => 'beginner',
]);
```

ប្រើ `RefreshDatabase` ក្នុង Feature tests ដើម្បីធ្វើឱ្យការផ្លាស់ប្ដូរ database របស់ test មិនបំពុល test ផ្សេង។

## ៨. លំហាត់អនុវត្ត

1. បង្កើត `CourseFactory` ដែលបង្កើត title, description និង level។
2. បង្កើត state `beginner()` និងប្រើវាបង្កើត Courses 5។
3. បង្កើត `CourseSeeder` សម្រាប់ sample data ក្នុង local development។
4. ហៅ `CourseSeeder` ពី `DatabaseSeeder`។
5. រត់ `php artisan db:seed --class=CourseSeeder` លើ local database ប៉ុណ្ណោះ។
6. ប្រើ factory ក្នុង Feature test ហើយ assert record មានក្នុង database។
7. ពន្យល់ពីហានិភ័យនៃ `migrate:fresh --seed`។

## កំហុសដែលជួបញឹកញាប់

- **ភ្លេច `HasFactory` trait**៖ `Course::factory()` មិនអាចប្រើបាន។
- **factory fields មិនត្រូវ schema**៖ ត្រូវឱ្យ factory attributes ត្រូវនឹង columns និង constraints។
- **ប្រើ fake value ដែលបំពាន unique constraint**៖ ប្រើ Faker unique generator ឬកំណត់តម្លៃក្នុង test ឱ្យខុសគ្នា។
- **រត់ seeder លើ production ដោយចៃដន្យ**៖ ពិនិត្យ environment និងអាន confirmation prompt។
- **គិតថា seeder គ្រប់គ្រងតាមចំណោទប្រតិបត្តិការបានដោយស្វ័យប្រវត្តិ**៖ ពិនិត្យថាការរត់ម្ដងទៀតនឹងបង្កើត duplicate data ឬទេ។
- **បញ្ចូល secrets ក្នុង sample records**៖ មិនត្រូវដាក់ credentials ពិតក្នុង factory ឬ seeder ដែល commit ទៅ Git។

## សំណួរត្រួតពិនិត្យ

1. Factory និង Seeder ខុសគ្នាដូចម្ដេច?
2. `make()` ខុសពី `create()` យ៉ាងដូចម្ដេច?
3. Factory state ជួយអ្វី?
4. តើ command ណាអាចរត់ seeder ជាក់លាក់?
5. ហេតុអ្វី `migrate:fresh --seed` គ្រោះថ្នាក់លើ database ដែលមានទិន្នន័យពិត?

## ឯកសារយោង

- [Laravel 13 Eloquent Factories](https://laravel.com/docs/13.x/eloquent-factories)
- [Laravel 13 Database Seeding](https://laravel.com/docs/13.x/seeding)
- [Laravel 13 Database Testing](https://laravel.com/docs/13.x/database-testing)
