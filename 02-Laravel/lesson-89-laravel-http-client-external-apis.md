# មេរៀនទី ៨៩៖ Laravel HTTP Client សម្រាប់ External APIs

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ហៅ external API ដោយ Laravel HTTP Client។
- កំណត់ headers, query parameters និង bearer token។
- កំណត់ timeout និង retry តាមភាពសមស្រប។
- ពិនិត្យ HTTP error responses ដោយច្បាស់។
- សាកល្បង API integration ដោយ response fakes ដោយមិនផ្ញើ request ពិត។

## ១. Laravel HTTP Client ជាអ្វី?

Laravel HTTP Client ជា wrapper លើ Guzzle សម្រាប់ផ្ញើ outbound HTTP requests។ វាមាន API សាមញ្ញសម្រាប់ GET, POST, JSON, headers, timeout, retry និង faking responses ក្នុង tests។

កុំហៅ external API ពី Blade view។ ប្រើ service class ឬ application layer ដើម្បីឱ្យការកំណត់ API, error handling និង tests ងាយថែទាំ។

## ២. GET Request និង JSON Response

ឧទាហរណ៍ហៅ API៖

```php
use Illuminate\\Support\\Facades\\Http;

$response = Http::acceptJson()
    ->timeout(5)
    ->get('https://api.example.test/v1/courses', [
        'page' => 1,
    ]);

if ($response->successful()) {
    $courses = $response->json('data', []);
}
```

`acceptJson()` ស្នើឱ្យ server ឆ្លើយតបជា JSON។ Query parameters អាចផ្ដល់ជា array ទីពីររបស់ `get()`។ Timeout កំណត់ពេលអតិបរមារង់ចាំ response; ប្រសិនបើភ្ជាប់មិនបាន ឬ timeout អាចកើត connection exception។

## ៣. POST JSON និង Authentication

ផ្ញើ JSON body ដោយ `post()`៖

```php
$response = Http::acceptJson()
    ->timeout(5)
    ->withToken(config('services.partner.token'))
    ->post('https://api.example.test/v1/courses', [
        'title' => 'PHP មូលដ្ឋាន',
        'level' => 'beginner',
    ]);
```

`withToken()` បន្ថែម Bearer token ទៅ Authorization header។ ទុក token ក្នុង environment/secret manager ហើយយកតាម configuration ដូចជា `config/services.php`។ កុំ hard-code ឬ log token ពិត។

## ៤. Error Handling និង Retry

Laravel HTTP Client មិនបោះ exception ដោយស្វ័យប្រវត្តិចំពោះ HTTP 4xx/5xx responses ទេ។ ពិនិត្យ response methods៖

```php
if ($response->clientError()) {
    // 4xx: request ឬ permission មានបញ្ហា
}

if ($response->serverError()) {
    // 5xx: upstream server មានបញ្ហា
}

$response->throw();
```

ប្រើ retry ដោយមានកំណត់៖

```php
$response = Http::timeout(5)
    ->retry(3, 200)
    ->get('https://api.example.test/v1/status');
```

កុំ retry សកម្មភាពមិនមានសុវត្ថិភាពសម្រាប់ការធ្វើម្ដងទៀត ដូចជា charge payment ដោយគ្មាន idempotency key។ Retry សមស្របជាងសម្រាប់ transient failures ហើយគួរមាន timeout និង attempt limit។ កំហុស connection អាចបោះ `ConnectionException`; សូមកំណត់ការគ្រប់គ្រង exception តាម use case។

## ៥. Faking API Responses ក្នុង Tests

ប្រើ `Http::fake()` ដើម្បីជំនួស external API response៖

```php
use Illuminate\\Support\\Facades\\Http;

Http::preventStrayRequests();

Http::fake([
    'api.example.test/v1/courses*' => Http::response([
        'data' => [
            ['id' => 1, 'title' => 'PHP មូលដ្ឋាន'],
        ],
    ], 200),
]);

// ហៅ service method ដែលប្រើ HTTP client...

Http::assertSent(function ($request) {
    return $request->url() === 'https://api.example.test/v1/courses?page=1';
});
```

`preventStrayRequests()` បង្កឱ្យ test បរាជ័យ ប្រសិនបើ code ផ្ញើ request ដែលមិនបាន fake ជាមុន។ វាជួយកុំឱ្យ test ទាក់ទង network ខាងក្រៅដោយចៃដន្យ។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត service class សម្រាប់ហៅ partner API។
2. ប្រើ `acceptJson()`, timeout និង query parameters ក្នុង GET request។
3. ប្រើ token ដែលបានផ្ទុកក្នុង configuration សម្រាប់ POST។
4. ចាត់ចែង 4xx, 5xx និង connection failures ដាច់ដោយឡែក។
5. បន្ថែម retry តែសម្រាប់ request ដែលអាចធ្វើឡើងវិញបានដោយសុវត្ថិភាព។
6. សរសេរ test ដោយ `Http::fake()` និង `Http::preventStrayRequests()`។
7. ពិនិត្យថា logs មិនមាន bearer token ឬ private request body។

## ៧. កំហុសដែលជួបញឹកញាប់

- **មិនកំណត់ timeout**៖ request អាចរង់ចាំយូរ។
- **សន្មតថា 4xx/5xx បោះ exception**៖ ពិនិត្យ `successful()`, `failed()`, `clientError()` ឬប្រើ `throw()`។
- **Retry POST ដោយគ្មាន idempotency**៖ អាចបង្កើត order ឬ payment ស្ទួន។
- **Hard-code token ក្នុង PHP**៖ ប្រើ secret configuration។
- **មិន validate upstream response**៖ API ខាងក្រៅអាចត្រឡប់ schema ខុស ឬ data មិនគ្រប់។
- **Tests ផ្ញើ request ពិត**៖ fake responses និងទប់ stray requests។
- **បង្ហោះ response body មានទិន្នន័យរសើបក្នុង log**៖ log តែ context ចាំបាច់ ហើយ redact secrets។

## សំណួរត្រួតពិនិត្យ

1. តើ `acceptJson()` ធ្វើអ្វី?
2. តើ Laravel HTTP Client ប្រព្រឹត្តយ៉ាងដូចម្ដេចចំពោះ 4xx/5xx តាមលំនាំដើម?
3. ហេតុអ្វីត្រូវកំណត់ timeout?
4. តើពេលណា retry អាចបង្ក side effect ស្ទួន?
5. `Http::preventStrayRequests()` ជួយ test ដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 HTTP Client](https://laravel.com/docs/13.x/http-client)
- [Laravel 13 Configuration](https://laravel.com/docs/13.x/configuration)
- [Laravel 13 Testing](https://laravel.com/docs/13.x/testing)
