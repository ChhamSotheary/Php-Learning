# មេរៀនទី ៧៨៖ Laravel Exception Handling និង Logging

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក exception reporting និង exception rendering។
- យល់ពីតួនាទី `APP_DEBUG` ក្នុង local និង production។
- កត់ត្រា log តាម channel និង severity។
- បង្កើត HTTP error response ដោយ status code សមស្រប។
- ជៀសវាងការបង្ហាញ stack trace ឬ secrets ដល់អ្នកប្រើ។

## ១. Laravel គ្រប់គ្រង Exception

Laravel បានរៀបចំ exception handling ស្រាប់។ ជាទូទៅពេល exception កើតឡើង framework នឹង៖

1. រាយការណ៍វា (report) ដើម្បីកត់ត្រា ឬផ្ញើទៅ error monitoring។
2. បម្លែងវាទៅ HTTP response (render) ដែលសមនឹង web page ឬ API។
3. ប្រគល់ response ត្រឡប់ទៅ client។

កុំចាប់ exception ទាំងអស់ហើយលាក់ចោលដោយមិន log។ កំហុសត្រូវមានសញ្ញាណដែល developer អាចស៊ើបអង្កេតបាន ខណៈអ្នកប្រើទទួលបានសារសុវត្ថិភាព និងសមស្រប។

## ២. Debug Mode និងសុវត្ថិភាព

`APP_DEBUG` គ្រប់គ្រងកម្រិតព័ត៌មាន error ដែលបង្ហាញទៅអ្នកប្រើ៖

- នៅ local development អាចកំណត់ `APP_DEBUG=true` ដើម្បីស៊ើបអង្កេត។
- នៅ production ត្រូវកំណត់ `APP_DEBUG=false` ជានិច្ច។

បើបើក debug នៅ production វាអាចបង្ហាញ stack trace, paths, configuration ឬព័ត៌មានរសើបផ្សេងទៀត។ កុំយក error page ដែលបង្ហាញលម្អិតពី local ទៅបង្ហាញអ្នកប្រើពិត។

## ៣. Log Channels និង Severity

Laravel logging ប្រើ channels ដែលកំណត់នៅ `config/logging.php`។ Channel កំណត់កន្លែងសរសេរ log ដូចជា file ឬ system log។ លំនាំដើមជាទូទៅប្រើ `stack` ដែលអាចប្រមូលពី channels ច្រើន។

កត់ត្រា message ដោយ `Log` facade៖

```php
use Illuminate\\Support\\Facades\\Log;

Log::info('Course created', [
    'course_id' => $course->id,
]);

Log::warning('Course import skipped', [
    'row' => $rowNumber,
]);

Log::error('Payment provider request failed', [
    'order_id' => $order->id,
]);
```

Levels ដែលប្រើញឹកញាប់មាន `debug`, `info`, `notice`, `warning`, `error`, `critical`, `alert` និង `emergency`។ ជ្រើស level តាមភាពធ្ងន់ធ្ងរ និងកំណត់ channel threshold ដើម្បីកុំឱ្យ log មានសំឡេងរំខានពេក។

កុំដាក់ password, bearer token, session cookie, payment card data ឬព័ត៌មានផ្ទាល់ខ្លួនមិនចាំបាច់ក្នុង log។ Log ក៏ជាទិន្នន័យដែលត្រូវគ្រប់គ្រងសុវត្ថិភាពដែរ។

## ៤. បង្កើត HTTP Error

សម្រាប់កំហុស HTTP សាមញ្ញ អាចប្រើ `abort()`៖

```php
abort(404);
```

ឧទាហរណ៍៖

- `401` — មិនទាន់ authenticate។
- `403` — បាន authenticate ប៉ុន្តែគ្មានសិទ្ធិ។
- `404` — resource មិនមាន ឬមិនបង្ហាញថាមាន។
- `422` — input មិនឆ្លង validation។
- `500` — កំហុសផ្នែក server ដែលមិនរំពឹងទុក។

សូមប្រើ status code ត្រឹមត្រូវ ហើយកុំត្រឡប់ stack trace ឬ exception message មានអាថ៌កំបាំងទៅ client។

## ៥. Custom Exception Rendering ក្នុង Laravel 13

ការកំណត់ exception handling របស់ project ថ្មីស្ថិតក្នុង `bootstrap/app.php`។ អាចចុះឈ្មោះ renderer ដោយ `withExceptions`។ ឧទាហរណ៍សម្រាប់ API route ដែលមិនរកឃើញ៖

```php
use Illuminate\\Http\\Request;
use Symfony\\Component\\HttpKernel\\Exception\\NotFoundHttpException;

// ផ្នែកនៃ application builder chain ក្នុង bootstrap/app.php
->withExceptions(function ($exceptions): void {
    $exceptions->render(function (
        NotFoundHttpException $exception,
        Request $request
    ) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'រកមិនឃើញធនធាននេះទេ។',
            ], 404);
        }

        // បើមិន return response, ប្រើ default renderer
    });
})
```

កូដខាងលើគ្រាន់តែជាផ្នែកមួយនៃ chain; ដាក់វាក្នុង `bootstrap/app.php` ដែលមានស្រាប់ មិនមែនជំនួស file ទាំងមូល។ ប្រសិនបើ renderer មិន return response, Laravel អាចបន្តប្រើ default behavior។

ជាទូទៅមិនចាំបាច់ customize exception ទាំងអស់ទេ។ ប្រើ default behavior ហើយបន្ថែម renderer តែពេលចង់បាន response format ឬការបង្ហាញខុសគ្នាច្បាស់លាស់។

## ៦. បង្កើត Custom Exception Class

បើច្បាប់អាជីវកម្មអាចបង្កើតកំហុសដែលមានន័យជាក់លាក់ អាចបង្កើត exception class ដើម្បីធ្វើឱ្យ code អានយល់៖

```bash
php artisan make:exception CourseEnrollmentClosed
```

បន្ទាប់មក code អាចបោះ exception នេះនៅពេលការចុះឈ្មោះវគ្គបិទ។ កំណត់របៀប report ឬ render នៅកន្លែងដែលសមស្រប ដោយមិនបង្ហាញព័ត៌មានខាងក្នុងទៅ user។ ប្រើ exception សម្រាប់លក្ខខណ្ឌ exception ពិតៗ; កុំប្រើវាជំនួស control flow ធម្មតាដូចជា validation errors។

## ៧. លំហាត់អនុវត្ត

1. កំណត់ `APP_DEBUG=true` ក្នុង local environment ហើយសង្កេត error page សាកល្បង។ បន្ទាប់មកកំណត់ `false` ហើយសង្កេតភាពខុសគ្នា។
2. បន្ថែម `Log::info()` ពេលបង្កើត Course ដោយកត់ត្រា Course ID។
3. បន្ថែម `Log::warning()` នៅពេល business rule មួយមិនអនុញ្ញាតឱ្យចុះឈ្មោះ។
4. បង្កើត route សាកល្បងដែលត្រឡប់ 404 ដោយ `abort(404)`។
5. ពិនិត្យ logs ក្នុង `storage/logs/` ហើយធានាថាមិនមាន secrets។
6. សរសេរថា production គួរឆ្លើយតបដូចម្ដេចពេលមាន unexpected exception។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ទុក `APP_DEBUG=true` នៅ production**៖ អាចបង្ហាញព័ត៌មានរសើប។
- **log ព័ត៌មានសម្ងាត់**៖ Redact ឬកុំកត់ត្រាវា។
- **ចាប់ exception រួចមិន log និងមិនឆ្លើយតប**៖ វាធ្វើឱ្យកំហុសបាត់ពីការស៊ើបអង្កេត។
- **ប្រើ status code មិនត្រឹមត្រូវ**៖ client អាចមិនដឹងថាត្រូវ login, កែ input ឬសាកម្ដងទៀត។
- **បង្ហាញ exception message ផ្ទាល់**៖ វាអាចបង្ហាញ table names, file paths ឬ implementation details។
- **កត់ត្រា log ច្រើនពេកដោយគ្មាន context**៖ រួមបញ្ចូល identifier មានសុវត្ថិភាព ដើម្បីស្វែងរកបញ្ហាបាន តែជៀសវាងទិន្នន័យផ្ទាល់ខ្លួន។

## សំណួរត្រួតពិនិត្យ

1. Reporting និង rendering exception ខុសគ្នាដូចម្ដេច?
2. ហេតុអ្វី production ត្រូវបិទ debug mode?
3. Channel មានតួនាទីអ្វីក្នុង Laravel logging?
4. តើ HTTP status code 403 និង 404 ប្រើពេលណា?
5. តើព័ត៌មានអ្វីខ្លះមិនគួរដាក់ក្នុង log?

## ឯកសារយោង

- [Laravel 13 Error Handling](https://laravel.com/docs/13.x/errors)
- [Laravel 13 Logging](https://laravel.com/docs/13.x/logging)
- [Laravel 13 Deployment](https://laravel.com/docs/13.x/deployment)
