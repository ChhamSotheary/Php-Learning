# មេរៀនទី ៧២៖ Laravel Request និងការផ្ទៀងផ្ទាត់ទិន្នន័យ

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ទទួលទិន្នន័យពី HTTP request តាម Laravel Request object។
- សរសេរ validation rules សម្រាប់ form input។
- ប្រើ validated data ដោយមិនទុកចិត្ត input ដើម។
- ឆ្លើយតបខុសគ្នារវាង form request និង JSON API។
- រៀបចំ validation ដាច់ទៅ Form Request class។

## ១. ទទួល Request

Laravel អាច inject `Illuminate\\Http\\Request` ទៅ Controller ឬ Route callback តាម service container។ ឧទាហរណ៍៖

```php
use Illuminate\\Http\\Request;
use Illuminate\\Support\\Facades\\Route;

Route::post('/contact', function (Request $request) {
    $email = $request->input('email');

    return response()->json([
        'email' => $email,
    ]);
});
```

តម្លៃដែលបានទទួលពី client គួរត្រូវចាត់ទុកថាមិនទាន់ទុកចិត្ត។ មុនរក្សាទុក ឬប្រើវាជាបញ្ជា ត្រូវ validate និងអនុវត្តការការពារតាម context។

## ២. Validate ក្នុង Controller

ប្រើ `$request->validate()` ដើម្បីពិនិត្យទិន្នន័យ៖

```php
use Illuminate\\Http\\Request;

public function store(Request $request)
{
    $validated = $request->validate([
        'name' => ['required', 'string', 'max:100'],
        'email' => ['required', 'email', 'max:255'],
        'message' => ['required', 'string', 'max:5000'],
    ]);

    // ប្រើតែ $validated សម្រាប់ logic បន្ទាប់
    return response()->json([
        'message' => 'ទិន្នន័យត្រឹមត្រូវ',
        'data' => $validated,
    ]);
}
```

Rules ទូទៅ៖

- `required` — ត្រូវមានតម្លៃ។
- `string` — តម្លៃត្រូវជាអត្ថបទ។
- `email` — ត្រូវមានទម្រង់ email។
- `integer` — ត្រូវជាចំនួនគត់។
- `min` និង `max` — កំណត់ទំហំ ឬជួរតម្លៃ អាស្រ័យលើប្រភេទ។
- `in` — តម្លៃត្រូវស្ថិតក្នុងជម្រើសដែលអនុញ្ញាត។

ឧទាហរណ៍សម្រាប់តម្លៃស្ថានភាព៖

```php
'status' => ['required', 'in:draft,published'],
```

កុំយកទិន្នន័យទាំងមូលពី request ហើយបញ្ចូលទៅ database ដោយផ្ទាល់។ ប្រើតែទិន្នន័យដែលឆ្លង validation រួច និងអនុញ្ញាតតាម business rules។

## ៣. អ្វីកើតឡើងពេល Validation បរាជ័យ?

សម្រាប់សំណើ web ដែលមិនមែនជា JSON, Laravel ជាទូទៅ redirect ត្រឡប់ទៅទំព័រមុន ហើយរក្សា validation errors និង input ចាស់សម្រាប់បង្ហាញឡើងវិញ។

សម្រាប់សំណើដែល Laravel សម្គាល់ថារំពឹង JSON (ឧទាហរណ៍ AJAX/API request) វាត្រឡប់ JSON error response ជាមួយ HTTP status `422`។ ដូច្នេះ client អាចបង្ហាញកំហុសជាប់ field ដែលត្រូវការ។

ស្ថានភាព `422 Unprocessable Content` មានន័យថា server អាចអានទម្រង់សំណើបាន ប៉ុន្តែទិន្នន័យមិនឆ្លង validation។

## ៤. Form Request សម្រាប់ Validation ដែលមានច្រើន

បើ Controller ចាប់ផ្តើមមាន validation rules ច្រើន អាចបង្កើត Form Request class៖

```bash
php artisan make:request StoreContactRequest
```

ក្នុង `app/Http/Requests/StoreContactRequest.php` កំណត់ rules៖

```php
<?php

namespace App\\Http\\Requests;

use Illuminate\\Foundation\\Http\\FormRequest;

class StoreContactRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:100'],
            'email' => ['required', 'email', 'max:255'],
            'message' => ['required', 'string', 'max:5000'],
        ];
    }
}
```

បន្ទាប់មក type-hint request class នេះក្នុង Controller៖

```php
use App\\Http\\Requests\\StoreContactRequest;

public function store(StoreContactRequest $request)
{
    $validated = $request->validated();

    return response()->json([
        'message' => 'ទទួលសាររួច',
        'data' => $validated,
    ]);
}
```

Laravel validate Form Request មុនពេលចូល method របស់ Controller។ Method `authorize()` គឺសម្រាប់សម្រេចថាអ្នកប្រើមានសិទ្ធិធ្វើសកម្មភាពនេះឬអត់; ការត្រឡប់ `true` ក្នុងឧទាហរណ៍នេះគ្រាន់តែអនុញ្ញាតសម្រាប់លំហាត់ប៉ុណ្ណោះ។ កម្មវិធីពិតត្រូវសរសេរ authorization តាមតម្រូវការ។

## ៥. សារកំហុសជាភាសាខ្មែរ

អាចកំណត់សារផ្ទាល់ក្នុង validation call ដើម្បីបង្ហាញអ្វីដែលអ្នកប្រើងាយយល់៖

```php
$validated = $request->validate(
    [
        'name' => ['required', 'string', 'max:100'],
        'email' => ['required', 'email'],
    ],
    [
        'name.required' => 'សូមបញ្ចូលឈ្មោះ។',
        'email.required' => 'សូមបញ្ចូលអាសយដ្ឋានអ៊ីមែល។',
        'email.email' => 'សូមបញ្ចូលអ៊ីមែលដែលមានទម្រង់ត្រឹមត្រូវ។',
    ],
);
```

កំណត់សារឱ្យត្រូវតាម field និង rule ដើម្បីឱ្យអ្នកប្រើដឹងថាត្រូវកែអ្វី។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត `StoreCourseRequest` សម្រាប់ទទួល `title`, `description`, និង `level`។
2. កំណត់ `title` ជា required string អតិបរមា 120 តួអក្សរ។
3. កំណត់ `description` ជា required string អតិបរមា 3000 តួអក្សរ។
4. អនុញ្ញាត `level` តែ `beginner`, `intermediate`, ឬ `advanced`។
5. ប្រើ `$request->validated()` ក្នុង Controller ហើយកុំយក request data ទាំងមូលទៅរក្សាទុក។

## ៧. កំហុសដែលជួបញឹកញាប់

- **គិតថា HTML form validation គ្រប់គ្រាន់**៖ Client អាចរំលង ឬកែ client-side validation បាន; server ត្រូវ validate ជានិច្ច។
- **ប្រើ input មិនទាន់ validate**៖ ប្រើ `validated()` ឬតម្លៃដែលទទួលបានពី `validate()`។
- **ភ្លេច authorization**៖ Validation ពិនិត្យទម្រង់/ច្បាប់ទិន្នន័យ; វាមិនបញ្ជាក់ថាអ្នកប្រើមានសិទ្ធិលើ resource ទេ។
- **រំពឹង JSON តែបាន redirect**៖ ពិនិត្យថា API client ផ្ញើ `Accept: application/json`។
- **ប្រើ rule `in` សម្រាប់តម្លៃដែលមិនកំណត់**៖ ជម្រើសដែលអនុញ្ញាតត្រូវបានគ្រប់គ្រងខាង server មិនមែនជឿលើ dropdown របស់ browser ទេ។

## សំណួរត្រួតពិនិត្យ

1. តើ Request object ជួយអ្វី?
2. ហេតុអ្វីត្រូវប្រើទិន្នន័យ validated?
3. តើ validation failure ឆ្លើយតបយ៉ាងដូចម្ដេចចំពោះ web request និង JSON request?
4. Form Request class មានអត្ថប្រយោជន៍អ្វី?
5. Validation និង authorization ខុសគ្នាដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 Validation](https://laravel.com/docs/13.x/validation)
- [Laravel 13 Requests](https://laravel.com/docs/13.x/requests)
