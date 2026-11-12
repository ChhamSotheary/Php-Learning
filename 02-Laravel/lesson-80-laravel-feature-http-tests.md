# មេរៀនទី ៨០៖ Laravel Feature Tests និង HTTP Tests

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក Unit tests និង Feature tests ក្នុង Laravel។
- សរសេរ HTTP test សម្រាប់ Route និង JSON response។
- ប្រើ assertions ដើម្បីពិនិត្យ status code និង response body។
- សាកល្បង validation និង authentication behavior។
- ធ្វើឱ្យ database tests អាចដំណើរការដដែលៗដោយ `RefreshDatabase`។

## ១. ប្រភេទ Test ក្នុង Laravel

Project Laravel មានថត `tests/Unit` និង `tests/Feature` ជាទូទៅ៖

- **Unit test** ពិនិត្យផ្នែកតូច និងដាច់ដោយឡែក។ ជាទូទៅវាមិន boot Laravel app ហើយមិនប្រើ database ឬ framework services ទេ។
- **Feature test** អាចពិនិត្យ interaction រវាង components ឬ simulate HTTP request ពេញមួយលំហូរ ដូចជា Route → middleware → Controller → response។

សម្រាប់ web/API behavior ភាគច្រើន Feature test ផ្តល់ទំនុកចិត្តលើការភ្ជាប់រវាងផ្នែកច្រើន។

## ២. បង្កើត Test

បង្កើត Feature test៖

```bash
php artisan make:test CourseIndexTest
```

Laravel ដាក់វាក្នុង `tests/Feature/CourseIndexTest.php`។ HTTP test methods របស់ Laravel simulate request ទៅ app ដោយផ្ទៃក្នុង មិនបញ្ជូន network request ពិតតាម internet ទេ។

ឧទាហរណ៍ពិនិត្យ Route ដែលបង្ហាញ JSON៖

```php
<?php

namespace Tests\\Feature;

use Tests\\TestCase;

class CourseIndexTest extends TestCase
{
    public function test_courses_endpoint_returns_json(): void
    {
        $response = $this->getJson('/api/courses');

        $response->assertOk()
            ->assertJsonStructure([
                'data',
            ]);
    }
}
```

`getJson()` បន្ថែម JSON expectation សម្រាប់ test request។ `assertOk()` ពិនិត្យ status 200 ហើយ `assertJsonStructure()` ពិនិត្យទម្រង់ JSON response។

## ៣. សាកល្បងទិន្នន័យក្នុង Database

បើ test ត្រូវការ database records អាចប្រើ Model factory និង `RefreshDatabase`៖

```php
<?php

namespace Tests\\Feature;

use App\\Models\\Course;
use Illuminate\\Foundation\\Testing\\RefreshDatabase;
use Tests\\TestCase;

class CourseIndexTest extends TestCase
{
    use RefreshDatabase;

    public function test_it_lists_courses(): void
    {
        $course = Course::factory()->create([
            'title' => 'PHP មូលដ្ឋាន',
            'level' => 'beginner',
        ]);

        $response = $this->getJson('/api/courses');

        $response->assertOk()
            ->assertJsonPath('data.0.title', $course->title);
    }
}
```

ឧទាហរណ៍នេះសន្មតថា project មាន Course factory និង API Resource ដែលត្រឡប់ `data` collection។ ប្រសិនបើ factory ឬ response structure ខុស ត្រូវកែ test ឱ្យត្រូវនឹង application ពិត។ `RefreshDatabase` រៀបចំ database សម្រាប់ tests ហើយជាទូទៅប្រើ transaction ដើម្បីកុំឱ្យទិន្នន័យ test មួយលេចទៅ test បន្ទាប់។

ប្រើ test database ដាច់ពី development/production។ កុំរត់ destructive database commands លើ database ដែលមានទិន្នន័យពិត។

## ៤. ពិនិត្យ Validation និង Error Response

អាចផ្ញើទិន្នន័យមិនត្រឹមត្រូវ ហើយពិនិត្យ validation error៖

```php
public function test_course_title_is_required(): void
{
    $response = $this->postJson('/api/courses', [
        'description' => 'សេចក្ដីពិពណ៌នាវគ្គសិក្សា',
        'level' => 'beginner',
    ]);

    $response->assertUnprocessable()
        ->assertJsonValidationErrors(['title']);
}
```

Test នេះសន្មតថា endpoint តម្រូវ `title` និង JSON request ត្រូវទទួល 422។ បើ endpoint ត្រូវការអត្តសញ្ញាណ អាច authenticate test user ជាមុន ដោយ helper ដូចជា `actingAs($user)`។

## ៥. ពិនិត្យ Authentication និង Authorization

ឧទាហរណ៍ពិនិត្យថា guest មិនអាចចូល Route ការពារ៖

```php
public function test_guest_cannot_access_private_course_route(): void
{
    $response = $this->getJson('/api/private-courses');

    $response->assertUnauthorized();
}
```

Status response អាស្រ័យលើ middleware និងថា request រំពឹង JSON ឬ browser redirect។ សរសេរ assertion ឱ្យស្របនឹង behavior ដែល API កំណត់។

Test authorization គួរមានយ៉ាងហោចណាស់៖

- User ដែលមានសិទ្ធិអាចធ្វើសកម្មភាព។
- Authenticated user ដែលគ្មានសិទ្ធិទទួល 403 ឬ behavior ដែលកំណត់។
- Guest ទទួល response ត្រឹមត្រូវតាម auth middleware។

## ៦. រត់ Test

រត់ suite ទាំងមូល៖

```bash
php artisan test
```

ឬរត់ file មួយ៖

```bash
php artisan test --filter=CourseIndexTest
```

ពេល test បរាជ័យ សូមអាន failure message និងបំបែកថាបញ្ហាមកពី test expectation, application logic, test database ឬ configuration។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត Feature test សម្រាប់ `GET /api/courses`។
2. បង្កើត Course តាម factory ហើយពិនិត្យថាវាលេចក្នុង JSON response។
3. សាក `POST /api/courses` ដោយគ្មាន `title` ហើយពិនិត្យ validation error។
4. សាក Route ការពារជា guest និង authenticated user។
5. បន្ថែម `RefreshDatabase` និងប្រើ test database ដាច់ដោយឡែក។
6. រត់ `php artisan test` ហើយកែបញ្ហារហូត tests ឆ្លង។

## ៨. កំហុសដែលជួបញឹកញាប់

- **Test ពឹងលើ database local ពិត**៖ ប្រើ test database និងកំណត់ environment ដាច់ដោយឡែក។
- **ភ្លេច `RefreshDatabase`**៖ records ពី test មុនអាចប៉ះពាល់លទ្ធផល។
- **Assume response JSON ខុសពី Resource ពិត**៖ ពិនិត្យ API response format ហើយសរសេរ assertions ត្រឹមត្រូវ។
- **សាកតែ success case**៖ សាក validation, guest និង user គ្មានសិទ្ធិផងដែរ។
- **ធ្វើ assertion ទូលំទូលាយពេក**៖ ពិនិត្យ status, JSON path និង error fields ដែលសំខាន់។
- **បង្កើត tests មិនមាន isolation**៖ កុំឱ្យលទ្ធផលពឹងលើលំដាប់នៃការរត់ test។

## សំណួរត្រួតពិនិត្យ

1. Unit test និង Feature test ខុសគ្នាយ៉ាងដូចម្ដេច?
2. តើ `getJson()` ប្រើសម្រាប់អ្វី?
3. `RefreshDatabase` ជួយអ្វី?
4. តើ test គួរពិនិត្យអ្វីខ្លះសម្រាប់ endpoint ដែលត្រូវ authenticate?
5. ហេតុអ្វីមិនត្រូវប្រើ production database សម្រាប់ tests?

## ឯកសារយោង

- [Laravel 13 Testing](https://laravel.com/docs/13.x/testing)
- [Laravel 13 HTTP Tests](https://laravel.com/docs/13.x/http-tests)
- [Laravel 13 Database Testing](https://laravel.com/docs/13.x/database-testing)
