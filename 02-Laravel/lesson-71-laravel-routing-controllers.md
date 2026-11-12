# មេរៀនទី ៧១៖ Laravel Routing និង Controllers

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- កំណត់ Route ដោយប្រើ HTTP methods។
- ប្រើ route parameter និងដាក់កម្រិតលើតម្លៃ parameter។
- ដាក់ឈ្មោះ Route ដើម្បីបង្កើត URL ងាយស្រួលកែប្រែ។
- បំបែកការងារចេញពី Route ទៅ Controller។
- បង្កើត Resource Controller សម្រាប់ CRUD។

## ១. Route ជាអ្វី?

Route ជាការកំណត់ថា កម្មវិធីត្រូវឆ្លើយតបយ៉ាងដូចម្ដេច នៅពេលទទួល HTTP request ដែលមាន method និង URL ជាក់លាក់។ ជាទូទៅ Web routes ស្ថិតនៅក្នុង `routes/web.php` ហើយប្រើ web middleware ដែលផ្តល់ session និង CSRF protection។

ឧទាហរណ៍ Route មូលដ្ឋាន៖

```php
<?php

use Illuminate\\Support\\Facades\\Route;

Route::get('/greeting', function () {
    return 'សួស្តី Laravel!';
});
```

HTTP method គួរត្រូវនឹងសកម្មភាពរបស់សំណើ៖

- `GET` — អាន ឬបង្ហាញទិន្នន័យ។
- `POST` — បង្កើត ឬដាក់ស្នើទិន្នន័យ។
- `PUT` / `PATCH` — កែប្រែទិន្នន័យ។
- `DELETE` — លុបទិន្នន័យ។

កុំប្រើ `GET` ដើម្បីធ្វើសកម្មភាពបំផ្លាញ ឬកែទិន្នន័យ។

## ២. Route Parameters និង Constraints

Parameter ក្នុង URL ដាក់ក្នុងសញ្ញា `{}`៖

```php
Route::get('/posts/{id}', function (string $id) {
    return "អត្ថបទលេខ {$id}";
});
```

អាចដាក់ constraint ដើម្បីទទួលតែលេខ៖

```php
Route::get('/posts/{id}', function (string $id) {
    return "អត្ថបទលេខ {$id}";
})->whereNumber('id');
```

បើ URL មិនត្រូវនឹងលក្ខខណ្ឌ Route នោះ Laravel មិនចូល closure នេះទេ ហើយជាទូទៅឆ្លើយតប 404។

## ៣. Named Routes

ដាក់ឈ្មោះឱ្យ Route ដើម្បីបង្កើត URL ដោយមិនចងភ្ជាប់កូដទៅនឹង URL string៖

```php
Route::get('/posts/{id}', function (string $id) {
    return "អត្ថបទលេខ {$id}";
})->name('posts.show');
```

បង្កើត URL ដោយឈ្មោះ៖

```php
$url = route('posts.show', ['id' => 12]);
```

បើ URL ផ្លាស់ប្តូរពេលក្រោយ កូដដែលប្រើ `route('posts.show', ...)` មិនចាំបាច់កែតាមគ្រប់កន្លែងទេ។

## ៤. ពេលណាត្រូវប្រើ Controller?

Closure សមស្របសម្រាប់ Route តូចៗ។ បើ Route មាន logic ច្រើន ឬមាន actions ជាច្រើន សូមផ្ទេរការងារទៅ Controller។ Controller ជួយរៀបចំកូដ និងធ្វើឱ្យ Route file អានងាយ។

បង្កើត Controller៖

```bash
php artisan make:controller PostController
```

Laravel បង្កើតឯកសារនៅ `app/Http/Controllers/PostController.php`។ ឧទាហរណ៍ method មួយ៖

```php
<?php

namespace App\\Http\\Controllers;

use Illuminate\\Http\\Response;

class PostController extends Controller
{
    public function show(string $id): Response
    {
        return response("អត្ថបទលេខ {$id}");
    }
}
```

ភ្ជាប់ Route ទៅ Controller method៖

```php
use App\\Http\\Controllers\\PostController;
use Illuminate\\Support\\Facades\\Route;

Route::get('/posts/{id}', [PostController::class, 'show'])
    ->name('posts.show');
```

កុំទុក database queries, validation និង business logic ទាំងអស់ក្នុង Route closure ធំមួយ។ បែងចែកទៅ Controller និង classes សមស្របតាមតម្រូវការ។

## ៥. Resource Controller សម្រាប់ CRUD

Resource Controller ផ្តល់ methods តាមលំនាំ CRUD ដូចជា index, create, store, show, edit, update និង destroy។ បង្កើតវាដោយ៖

```bash
php artisan make:controller PostController --resource
```

បន្ទាប់មកចុះ Route resource៖

```php
Route::resource('posts', PostController::class);
```

សម្រាប់ API ដែលមិនមានទំព័រ create/edit អាចបង្កើត API resource controller និងប្រើ `Route::apiResource()`៖

```php
Route::apiResource('posts', PostController::class);
```

បង្ហាញ Route ដែលបានចុះឈ្មោះ៖

```bash
php artisan route:list
```

ប្រើ `route:list` ដើម្បីពិនិត្យ method, URI, name និង action ពេលមិនប្រាកដថា Route ត្រូវបានចុះឈ្មោះយ៉ាងដូចម្ដេច។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត Route `GET /courses` ដែលត្រឡប់បញ្ជីវគ្គសិក្សាជា JSON។
2. បង្កើត Route `GET /courses/{id}` ដែលទទួលលេខសម្គាល់ និងដាក់ constraint ជាលេខ។
3. ដាក់ឈ្មោះ Route ទី ២ ថា `courses.show` ហើយបង្កើត URL ទៅវគ្គលេខ 5 ដោយ helper `route()`។
4. បង្កើត `CourseController` ហើយផ្លាស់ទី logic សម្រាប់បង្ហាញវគ្គមួយទៅ method `show`។
5. សាកល្បង `php artisan route:list` ហើយសង្កេតមើល Route ដែលបានបង្កើត។

## ៧. កំហុសដែលជួបញឹកញាប់

- **ប្រើ method ខុស**៖ Route `POST` មិនឆ្លើយតបនឹង browser `GET` request ទេ។
- **ភ្លេច import class**៖ ត្រូវ import `Route` facade និង Controller class នៅខាងលើ file។
- **ឈ្មោះ parameter មិនត្រូវគ្នា**៖ ត្រូវរក្សាលំដាប់ parameter របស់ method ឱ្យត្រូវនឹង URL segments។
- **ច្រឡំ Route web និង API**៖ `routes/web.php` មាន web middleware ដូចជា session និង CSRF; API routes មានការកំណត់ផ្សេងតាម project។
- **បង្កើត Route ស្ទួន**៖ ប្រើ `php artisan route:list` ដើម្បីស្វែងរក method និង URI ដែលជាន់គ្នា។

## សំណួរត្រួតពិនិត្យ

1. Route មានតួនាទីអ្វី?
2. ហេតុអ្វីបានជា `GET` មិនគួរប្រើសម្រាប់លុបទិន្នន័យ?
3. Named Route ជួយអ្វីខ្លះ?
4. តើពាក្យបញ្ជាណាបង្កើត Resource Controller?
5. តើ `Route::resource()` ខុសពី `Route::apiResource()` យ៉ាងដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 Routing](https://laravel.com/docs/13.x/routing)
- [Laravel 13 Controllers](https://laravel.com/docs/13.x/controllers)
