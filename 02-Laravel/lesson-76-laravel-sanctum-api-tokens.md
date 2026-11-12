# មេរៀនទី ៧៦៖ Laravel Sanctum និង API Token Authentication

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីការប្រើ Laravel Sanctum សម្រាប់ personal access tokens។
- ដំឡើង API support ក្នុង Laravel 13។
- ការពារ API Route ដោយ `auth:sanctum`។
- ចេញ token មាន abilities និងដកសុពលភាព token។
- បែងចែក API bearer tokens ពី session-based SPA authentication។

## ១. តើ Sanctum ដោះស្រាយអ្វី?

Laravel Sanctum ផ្តល់ authentication សម្រាប់ personal access tokens និង first-party single-page applications (SPA)។ ករណីប្រើខុសគ្នាមានពីរ៖

- **Third-party client ឬ mobile app**៖ ប្រើ API token ក្នុង `Authorization: Bearer ...` header។
- **First-party SPA ដែលជាកម្មវិធីរបស់អ្នកផ្ទាល់**៖ Sanctum ប្រើ session cookie និង CSRF protection។ កុំចេញ bearer token ទៅ JavaScript SPA របស់អ្នកដោយគ្មានហេតុផល; ប្រើ cookie-based SPA flow ដែល Sanctum ផ្តល់។

បើ API ត្រូវផ្តល់ OAuth2 flows ឬ delegation ស្មុគស្មាញ សូមពិនិត្យថាតើ Sanctum គ្រប់គ្រាន់ឬត្រូវការជម្រើសផ្សេងដូចជា Passport។

## ២. ដំឡើង API Support

ក្នុង Laravel 13 project អាចដំណើរការ៖

```bash
php artisan install:api
php artisan migrate
```

ពាក្យបញ្ជាដំឡើង API បន្ថែមការរៀបចំ API routes និង Sanctum តាម project skeleton បច្ចុប្បន្ន។ អាន output របស់ Artisan និងពិនិត្យ `routes/api.php`, migration និង configuration ដែលវាបង្កើត មុនបន្ត។

ក្នុង User model បន្ថែម trait `HasApiTokens`៖

```php
<?php

namespace App\\Models;

use Illuminate\\Foundation\\Auth\\User as Authenticatable;
use Laravel\\Sanctum\\HasApiTokens;
use Illuminate\\Notifications\\Notifiable;

class User extends Authenticatable
{
    use HasApiTokens;
    use Notifiable;
}
```

បើ User model មាន traits ផ្សេងទៀតរួច សូមបន្ថែម `HasApiTokens` ទៅក្នុងបញ្ជី traits ដោយរក្សា traits ដែលមានស្រាប់។

## ៣. ការពារ API Route

ភ្ជាប់ `auth:sanctum` ទៅ route ដែលទាមទារ user authenticated៖

```php
use Illuminate\\Http\\Request;
use Illuminate\\Support\\Facades\\Route;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

Sanctum guard អាចទទួល session-authenticated request របស់ SPA ឬ bearer token request ពី client ផ្សេងៗ អាស្រ័យលើ configuration និងប្រភេទ request។

## ៤. ចេញ API Token

បន្ទាប់ពីផ្ទៀងផ្ទាត់អ្នកប្រើ និងអនុញ្ញាតឱ្យបង្កើត token រួច អាចចេញ token ដោយ៖

```php
$accessToken = $user->createToken(
    'mobile-app',
    ['courses:read']
);

$plainTextToken = $accessToken->plainTextToken;
```

តម្លៃ `plainTextToken` ត្រូវផ្ញើទៅអ្នកប្រើតែពេលចេញ token ប៉ុណ្ណោះ ហើយគួរបង្ហាញឱ្យចម្លង/រក្សាទុកភ្លាមៗ។ Database របស់ Sanctum រក្សាទុក token ជាទម្រង់ hash ដើម្បីកុំឱ្យមាន bearer secret ដើមនៅក្នុង database។

Client បញ្ជូន token ក្នុង HTTPS request header៖

```http
Authorization: Bearer YOUR_PERSONAL_ACCESS_TOKEN
Accept: application/json
```

កុំដាក់ token ក្នុង URL query string ឬ log។ URL អាចចូលទៅ browser history, proxy logs និង analytics។ Client គួររក្សា token ក្នុង secure storage សមស្របនឹង platform ហើយ server ត្រូវប្រើ HTTPS។

## ៥. Token Abilities

Abilities ជា scopes សម្រាប់កំណត់សកម្មភាពដែល token នោះអាចធ្វើបាន។ ពេលចេញ token អាចកំណត់ abilities៖

```php
$user->createToken('report-client', [
    'reports:read',
    'reports:export',
]);
```

ពិនិត្យ ability ក្នុង code ឬ route middleware៖

```php
if ($request->user()->tokenCan('reports:read')) {
    // បន្តអនុញ្ញាតការអានរបាយការណ៍
}
```

Abilities មិនជំនួស Policy/Authorization ទេ។ ពិនិត្យទាំង ability របស់ token និងសិទ្ធិរបស់ user លើ resource ជាក់លាក់។ សម្រាប់ first-party SPA, Sanctum ផ្តល់លក្ខណៈ tokenCan ដែលអាចឆ្លើយ true តាមលំនាំសម្រាប់ការប្រើក្នុង policy; ដូច្នេះកុំប្រើ ability តែមួយមុខដើម្បីគ្រប់គ្រង ownership ឬសិទ្ធិលើទិន្នន័យ។

## ៦. ដកសុពលភាព Token

ដកសុពលភាព token ដែលកំពុងប្រើ៖

```php
$request->user()->currentAccessToken()->delete();
```

ដក token ជាក់លាក់របស់ user ក្រោយបាន authorize ថា token នោះជារបស់គាត់៖

```php
$user->tokens()->where('id', $tokenId)->delete();
```

ដកសុពលភាព token ទាំងអស់របស់ user (ឧទាហរណ៍ ពេល account ត្រូវបាន compromise)៖

```php
$user->tokens()->delete();
```

Token Sanctum ជាទូទៅមិនផុតកំណត់ដោយស្វ័យប្រវត្តិតាម default។ កំណត់ expiration តាម configuration និងគោលនយោបាយសុវត្ថិភាពរបស់កម្មវិធី ហើយបង្កើត UI ឱ្យអ្នកប្រើមើល និង revoke tokens បាន។

## ៧. លំហាត់អនុវត្ត

1. ដំឡើង API support ក្នុង Laravel project សាកល្បង ហើយពិនិត្យ files ដែលត្រូវបានបង្កើត។
2. បន្ថែម `HasApiTokens` ទៅ User model។
3. ការពារ `GET /api/user` ដោយ `auth:sanctum`។
4. បង្កើត token ដែលមាន ability `courses:read` តែបន្ទាប់ពី user បាន authenticate និងមានសិទ្ធិ។
5. សាកល្បង request ដោយមិនដាក់ token និងដោយដាក់ bearer token ត្រឹមត្រូវ។
6. បង្កើតមុខងារ revoke token បច្ចុប្បន្ន ហើយសាកល្បងប្រើ token ចាស់ម្ដងទៀត។
7. ពន្យល់ថាហេតុអ្វី SPA first-party គួរប្រើ cookie-based flow របស់ Sanctum ជំនួស personal access token។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ប្រើ token ដើម្បី login SPA របស់ខ្លួន**៖ ប្រើ Sanctum SPA session flow ដែលមាន CSRF protection។
- **បង្ហាញ plain token ម្តងទៀតក្រោយចេញ**៖ ត្រូវឱ្យអ្នកប្រើបង្កើត token ថ្មី ប្រសិនបើបាត់។
- **ដាក់ token ក្នុង URL ឬ source code**៖ ប្រើ HTTPS header និង secure storage។
- **ភ្លេច `auth:sanctum`**៖ route នឹងមិនត្រូវការអត្តសញ្ញាណ authenticated តាមការរំពឹង។
- **គិតថា ability ជា permission លើ record**៖ ត្រូវពិនិត្យ ownership/Policy ដាច់ដោយឡែក។
- **មិនមានវិធី revoke token**៖ អនុញ្ញាតឱ្យ user ដក token ពេលឧបករណ៍បាត់ ឬ secret លេចធ្លាយ។
- **ទុក token មិនផុតកំណត់ដោយគ្មានគោលនយោបាយ**៖ កំណត់ lifetime, rotation និង revocation សមនឹងហានិភ័យ។

## សំណួរត្រួតពិនិត្យ

1. តើ Sanctum មាន flow authentication សំខាន់ពីរអ្វីខ្លះ?
2. Bearer token ត្រូវបញ្ជូននៅក្នុង HTTP header ណា?
3. ហេតុអ្វីត្រូវបង្ហាញ plain text token តែពេលចេញ?
4. Abilities ខុសពី Policy យ៉ាងដូចម្ដេច?
5. តើ command ណាអាចដក token បច្ចុប្បន្ន?

## ឯកសារយោង

- [Laravel 13 Sanctum](https://laravel.com/docs/13.x/sanctum)
- [Laravel 13 API Routing](https://laravel.com/docs/13.x/routing#api-routes)
