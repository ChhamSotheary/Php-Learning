# មេរៀនទី ៧៧៖ Laravel Middleware

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីតួនាទីរបស់ middleware ក្នុង HTTP request flow។
- បង្កើត middleware ដោយ Artisan។
- ពិនិត្យ request មុនឱ្យ Controller ដំណើរការ។
- ចុះឈ្មោះ middleware alias ក្នុង Laravel 13។
- បែងចែក middleware សម្រាប់ Route និង middleware group។

## ១. Middleware ជាអ្វី?

Middleware ជាស្រទាប់ដែល request ឆ្លងកាត់មុនទៅ Route/Controller ហើយ response អាចឆ្លងត្រឡប់តាមស្រទាប់នោះផងដែរ។ វាសមស្របសម្រាប់ការត្រួតពិនិត្យទូទៅ ដូចជា authentication, role checks, logging ឬការបន្ថែម headers។

សង្ខេបលំហូរ៖

```text
HTTP Request → Middleware មុន Controller → Route/Controller
HTTP Response ← Middleware ក្រោយ Controller ← Route/Controller
```

Laravel មាន middleware ដូចជា `auth` និង CSRF protection រួចហើយ។ បង្កើត custom middleware ពេលមានច្បាប់ពិសេសរបស់កម្មវិធី។

## ២. បង្កើត Middleware

ប្រើ Artisan៖

```bash
php artisan make:middleware EnsureUserIsActive
```

Laravel បង្កើត class ក្នុង `app/Http/Middleware/`។ ឧទាហរណ៍៖

```php
<?php

namespace App\\Http\\Middleware;

use Closure;
use Illuminate\\Http\\Request;
use Symfony\\Component\\HttpFoundation\\Response;

class EnsureUserIsActive
{
    public function handle(Request $request, Closure $next): Response
    {
        $user = $request->user();

        if ($user !== null && ! $user->is_active) {
            abort(403, 'គណនីនេះមិនអាចប្រើបានទេ។');
        }

        return $next($request);
    }
}
```

`$next($request)` បញ្ជូន request ទៅ middleware បន្ទាប់ ឬទៅ Route។ បើ middleware ត្រឡប់ response ឬបង្កើត HTTP error ដោយ `abort()` វាអាចបញ្ឈប់លំហូរមុន Controller។

ឧទាហរណ៍នេះសន្មតថា User model មាន field `is_active`; ត្រូវបន្ថែម schema និងច្បាប់ account status ឱ្យពេញលេញក្នុងកម្មវិធីពិត។

## ៣. ចុះឈ្មោះ Alias ក្នុង Laravel 13

ក្នុង project Laravel 13, middleware configuration របស់កម្មវិធីស្ថិតក្នុង `bootstrap/app.php`។ បន្ថែម alias ក្នុង `withMiddleware`៖

```php
use App\\Http\\Middleware\\EnsureUserIsActive;
use Illuminate\\Foundation\\Configuration\\Middleware;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'active' => EnsureUserIsActive::class,
    ]);
})
```

កូដនេះបង្ហាញផ្នែកនៃ application builder chain ក្នុង `bootstrap/app.php`; កុំជំនួស file ទាំងមូល។ បញ្ចូល alias ទៅក្នុង `withMiddleware` ដែលមានស្រាប់ ប្រសិនបើមាន។

## ៤. ភ្ជាប់ Middleware ទៅ Route

ប្រើ alias ជាមួយ route មួយ៖

```php
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware(['auth', 'active']);
```

ឬដាក់លើក្រុម Route៖

```php
Route::middleware(['auth', 'active'])->group(function () {
    Route::get('/dashboard', DashboardController::class);
    Route::get('/reports', [ReportController::class, 'index']);
});
```

Middleware `auth` ពិនិត្យថាអ្នកប្រើបាន authenticate។ `active` ក្នុងឧទាហរណ៍នេះពិនិត្យ account status។ គោលបំណងទាំងពីរខុសគ្នា។

## ៥. Middleware Group និង Global Middleware

Laravel មាន middleware groups សម្រាប់ការកំណត់ជាក្រុម ដូចជា `web` និង `api`។ ក្នុង Laravel 13 អាចបន្ថែម ឬដក middleware ពី group តាម `withMiddleware` ក្នុង `bootstrap/app.php`។ កែ group ដោយប្រុងប្រយ័ត្ន ព្រោះការដក middleware ស្តង់ដារ ដូចជា CSRF protection ឬ session middleware អាចប៉ះពាល់សុវត្ថិភាព និងមុខងារទាំងមូល។

Global middleware ដំណើរការលើគ្រប់ request ដែល Laravel ទទួល។ ប្រើវាសម្រាប់ការងារដែលត្រូវការជាសកលពិតៗ; ច្បាប់ដែលមានតែ route ខ្លះត្រូវភ្ជាប់ទៅ route group ជំនួស។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត middleware `EnsureUserIsActive`។
2. ចុះឈ្មោះ alias `active` ក្នុង `bootstrap/app.php` ដោយរក្សា configuration ផ្សេងៗ។
3. ភ្ជាប់ `['auth', 'active']` ទៅ `/dashboard`។
4. សាកល្បងក្នុង local environment ជាមួយ user ដែល active និង inactive។
5. ពន្យល់ថា `$next($request)` ធ្វើអ្វី។
6. ប្រៀបធៀប middleware លើ route មួយ និង middleware group។

## ៧. កំហុសដែលជួបញឹកញាប់

- **ភ្លេចត្រឡប់ `$next($request)`**៖ request អាចមិនទៅដល់ Route/Controller។
- **ចុះឈ្មោះ alias ខុសទីតាំង**៖ ក្នុង Laravel 13 app middleware configuration ជាទូទៅស្ថិតក្នុង `bootstrap/app.php`។
- **យក middleware ជំនួស authorization policy ទាំងអស់**៖ ច្បាប់សិទ្ធិលើ resource ជាក់លាក់គួរប្រើ Policy/Gate។
- **សន្មតថា user មានជានិច្ច**៖ public route អាចគ្មាន authenticated user; ពិនិត្យ null ឬប្រើ `auth` middleware មុន។
- **ដាក់ logic ច្រើនពេកក្នុង middleware តែមួយ**៖ បំបែក responsibilities ឱ្យតូច និងងាយសាកល្បង។
- **ដក middleware សុវត្ថិភាពដោយមិនដឹងផលប៉ះពាល់**៖ ពិនិត្យឯកសារផ្លូវការ និង scope នៃការកែ។

## សំណួរត្រួតពិនិត្យ

1. Middleware ស្ថិតនៅកន្លែងណាក្នុង request flow?
2. `$next($request)` មានន័យអ្វី?
3. តើ Laravel 13 app ចុះឈ្មោះ middleware alias នៅ file ណា?
4. Middleware `auth` និង `active` ក្នុងឧទាហរណ៍មានតួនាទីខុសគ្នាដូចម្ដេច?
5. ហេតុអ្វីមិនគួរដក CSRF middleware ដើម្បីដោះស្រាយបញ្ហា form ដោយងាយៗ?

## ឯកសារយោង

- [Laravel 13 Middleware](https://laravel.com/docs/13.x/middleware)
- [Laravel 13 Request Lifecycle](https://laravel.com/docs/13.x/lifecycle)
