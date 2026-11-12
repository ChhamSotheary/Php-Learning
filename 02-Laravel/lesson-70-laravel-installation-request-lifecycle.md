# មេរៀនទី ៧០៖ Laravel 13 — រៀបចំកម្មវិធី និងដំណើរការសំណើ

> មេរៀននេះបន្តពី PHP មូលដ្ឋានទៅកាន់ Framework សម្រាប់ PHP។ ឧទាហរណ៍ប្រើ Laravel 13 និង PHP 8.3 ឬថ្មីជាងនេះ តាមឯកសារផ្លូវការ។ Version របស់ Framework និងតម្រូវការអាចផ្លាស់ប្តូរ ដូច្នេះត្រូវពិនិត្យឯកសារយោងមុនដំឡើង។

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ថា Laravel ជា PHP framework មានតួនាទីអ្វី។
- រៀបចំតម្រូវការមុនបង្កើតកម្មវិធី Laravel។
- បង្កើត និងដំណើរការ project Laravel ថ្មី។
- ស្គាល់ថតសំខាន់ៗក្នុង project។
- ពណ៌នាលំហូរពី HTTP request ទៅ response។

## ១. Laravel ជាអ្វី?

Laravel ជា framework សម្រាប់បង្កើត web applications និង APIs ដោយ PHP។ វាផ្តល់ឧបករណ៍ដែលប្រើញឹកញាប់ ដូចជា routing, middleware, controllers, validation, database migrations និង views ដើម្បីឱ្យកូដមានរចនាសម្ព័ន្ធ និងងាយថែទាំ។

Framework មិនលុបបំបាត់តម្រូវការយល់ PHP ទេ។ អ្នកនៅតែត្រូវយល់ variables, functions, arrays, classes, Composer, HTTP និង database។

## ២. តម្រូវការមុនចាប់ផ្តើម

សម្រាប់ Laravel 13៖

- PHP 8.3 ឬថ្មីជាងនេះ និង extensions ដែល Laravel ត្រូវការ។
- Composer សម្រាប់ដំឡើង PHP dependencies។
- Laravel installer សម្រាប់បង្កើត project តាម command line។
- Node.js និង NPM ឬ Bun ប្រសិនបើចង់ដំឡើង និង compile frontend assets។

ពិនិត្យឧបករណ៍៖

```bash
php -v
composer --version
node --version
npm --version
```

នៅលើ Windows អាចប្រើ PowerShell, WSL ឬ Laravel Herd។ បើមាន PHP និង Composer រួច អាចដំឡើង Laravel installer ដោយ៖

```bash
composer global require laravel/installer
```

បន្ទាប់មកបង្កើតកម្មវិធីថ្មី៖

```bash
laravel new learning-app
cd learning-app
```

ពេល installer សួរជម្រើស ដូចជា database ឬ starter kit អ្នកអាចជ្រើសតាមគោលដៅសិក្សា។ សម្រាប់មេរៀនដំបូង សូមជ្រើសជម្រើសសាមញ្ញដែលអ្នកអាចពន្យល់បាន។

បើ project មាន frontend assets សូមដំឡើង package និងចាប់ផ្តើម development processes៖

```bash
npm install
npm run build
composer run dev
```

បើ installer ជំនាន់ដែលអ្នកប្រើបានបង្កើតការកំណត់សមស្របរួច សូមអានសេចក្ដីណែនាំដែលវាបង្ហាញ ហើយប្រើ script របស់ project នោះ។ បន្ទាប់ពី server ចាប់ផ្តើម សាកបើក `http://localhost:8000` ក្នុង browser។

## ៣. ថត និងឯកសារសំខាន់ៗ

Project ថ្មីអាចមានថត និងឯកសារដូចខាងក្រោម៖

- `app/` — កូដកម្មវិធី ដូចជា models, controllers និង classes ផ្សេងៗ។
- `bootstrap/` — កូដសម្រាប់ចាប់ផ្តើម framework និងកំណត់រចនាសម្ព័ន្ធដំបូង។
- `config/` — configuration របស់កម្មវិធី។
- `database/` — migrations, factories និង seeders។
- `public/` — web document root; request ចូលកម្មវិធីតាម entry point នៅទីនេះ។
- `resources/` — views និង frontend source files។
- `routes/` — route definitions របស់ web ឬ API។
- `storage/` — logs, cache និងឯកសារដែលកម្មវិធីបង្កើត។
- `tests/` — automated tests។
- `vendor/` — packages ដែល Composer ដំឡើង; ជាទូទៅកុំកែឯកសារនៅក្នុងថតនេះដោយដៃ។
- `.env` — settings ជាក់លាក់របស់ម៉ាស៊ីន ឬ environment ដូចជា database connection។

កុំដាក់ password, API key ឬ secret ក្នុងកូដដែល commit ទៅ Git។ ឯកសារ `.env` គួរត្រូវបានរក្សាចេញពី version control ហើយប្រើ `.env.example` ដើម្បីបង្ហាញឈ្មោះ settings ដែលត្រូវការ ដោយមិនដាក់តម្លៃសម្ងាត់ពិត។

## ៤. ដំណើរការសំណើ (Request Lifecycle)

លំហូរទូទៅរបស់សំណើ HTTP ក្នុង Laravel៖

1. Browser ឬ client ផ្ញើ HTTP request ទៅ web server។
2. Web server បញ្ជូនសំណើទៅ entry point ក្នុង `public/index.php`។
3. កម្មវិធី Laravel ត្រូវបាន bootstrap និងរៀបចំ services/configuration។
4. Middleware ដែលពាក់ព័ន្ធពិនិត្យ ឬកែសំណើ ដូចជា session, authentication ឬ CSRF protection។
5. Router រក route ដែលត្រូវនឹង HTTP method និង URL។
6. Route បញ្ជូនការងារទៅ closure ឬ controller; កូដអាចហៅ service, model និង database។
7. Laravel បង្កើត HTTP response ហើយឆ្លងកាត់ middleware ចេញ។
8. Web server ផ្ញើ response ត្រឡប់ទៅ browser ឬ client។

ឧទាហរណ៍ផ្លូវសាមញ្ញមួយនៅក្នុង `routes/web.php`៖

```php
<?php

use Illuminate\\Support\\Facades\\Route;

Route::get('/hello', function () {
    return response('សួស្តីពី Laravel!');
});
```

បន្ទាប់ពី server ដំណើរការ សាកបើក `http://localhost:8000/hello`។ Route នេះទទួល GET request ហើយត្រឡប់ response ជាអត្ថបទ។

## ៥. សាកល្បងកែ Route

កែ `routes/web.php` ដើម្បីបន្ថែម៖

```php
Route::get('/about', function () {
    return response()->json([
        'project' => 'learning-app',
        'framework' => 'Laravel',
    ]);
});
```

បើក `http://localhost:8000/about`។ អ្នកគួរឃើញ JSON response។ សូមកត់សម្គាល់ថា route មានតួនាទីភ្ជាប់ URL ទៅនឹងកូដដែលបង្កើត response។

## ៦. កំហុសដែលជួបញឹកញាប់

- **រកមិនឃើញពាក្យបញ្ជា `laravel`**៖ ពិនិត្យថា Composer global bin directory ស្ថិតក្នុង PATH ឬប្រើ terminal session ថ្មី។
- **PHP version ទាបជាងតម្រូវការ**៖ ពិនិត្យ `php -v` និងកំណត់ PHP version ដែល terminal កំពុងប្រើ។
- **Dependencies មិនទាន់មាន**៖ នៅក្នុង project រត់ `composer install`។
- **Frontend assets មិនដំណើរការ**៖ ពិនិត្យថាបានដំឡើង Node/NPM និងរត់ `npm install` រួច។
- **404 នៅលើ route**៖ ពិនិត្យ URL, HTTP method, និងឯកសារ route ដែលបានកែ។
- **កែ `vendor/` ដោយដៃ**៖ ការកែអាចបាត់ពេល Composer ដំឡើង dependencies ម្ដងទៀត។ កែ source code របស់កម្មវិធី ឬ dependency configuration ជំនួស។

## លំហាត់

1. បង្កើត project Laravel ថ្មីឈ្មោះ `php-practice`។
2. បន្ថែម GET route `/greeting` ដែលត្រឡប់សារជាភាសាខ្មែរ។
3. បន្ថែម GET route `/profile` ដែលត្រឡប់ JSON មានឈ្មោះ និងជំនាញ PHP។
4. សរសេរលំហូរ request lifecycle ជា ៥–៨ ជំហាន ដោយប្រើពាក្យផ្ទាល់ខ្លួន។
5. ពន្យល់ថាហេតុអ្វីមិនគួរ commit តម្លៃពិតក្នុង `.env`។

<details>
<summary>ចម្លើយគំរូ</summary>

```php
Route::get('/greeting', function () {
    return response('សួស្តី! ខ្ញុំកំពុងរៀន PHP និង Laravel។');
});

Route::get('/profile', function () {
    return response()->json([
        'name' => 'Sotheary',
        'skill' => 'PHP',
    ]);
});
```

លំហូរអាចសង្ខេបថា៖ client ផ្ញើ request → web server ទទួល → Laravel bootstrap → middleware ពិនិត្យ → router ជ្រើស route → handler បង្កើត response → response ត្រឡប់ទៅ client។ `.env` អាចមាន secrets ដែលអ្នកដទៃមិនគួរឃើញ ដូច្នេះត្រូវរក្សាវាចេញពី Git។

</details>

## សំណួរត្រួតពិនិត្យ

1. Laravel ជា framework សម្រាប់ភាសាអ្វី?
2. `public/index.php` មានតួនាទីអ្វីក្នុងលំហូរទូទៅ?
3. Middleware អាចដំណើរការនៅចំណុចណាខ្លះនៃ request?
4. ហេតុអ្វីបានជាត្រូវប្រើ Composer?
5. តើ `.env` និង `.env.example` ខុសគ្នាដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 Installation](https://laravel.com/docs/13.x/installation)
- [Laravel 13 Request Lifecycle](https://laravel.com/docs/13.x/lifecycle)
- [Laravel 13 Deployment Requirements](https://laravel.com/docs/13.x/deployment)
