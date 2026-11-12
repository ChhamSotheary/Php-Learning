# មេរៀនទី ៩១៖ Symfony 7.4 LTS — រៀបចំ Project និង Request Flow

> មេរៀននេះចាប់ផ្តើមផ្នែក Symfony ក្នុងវគ្គ PHP។ ឯកសារយោងដែលបានពិនិត្យកំណត់ Symfony 7.4 ជា LTS បច្ចុប្បន្ន ដែលត្រូវការ PHP 8.2 ឬថ្មីជាងនេះ។ ព័ត៌មាន version អាចផ្លាស់ប្តូរ; សូមពិនិត្យទំព័រ Releases ផ្លូវការមុនបង្កើត project ថ្មី។

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីតួនាទី Symfony ក្នុងកម្មវិធី PHP។
- បង្កើត Symfony web application ដោយ Symfony CLI។
- បង្កើត Route និង Controller ដំបូង។
- ពន្យល់ពី request-response lifecycle តាម HttpKernel។
- ប្រើ `bin/console` ដើម្បីពិនិត្យ project។

## ១. Symfony ជាអ្វី?

Symfony ជា PHP framework និងជាក្រុម reusable components សម្រាប់បង្កើត web applications, APIs និង command-line applications។ វាប្រើ Composer packages និងផ្តល់ឧបករណ៍ដូចជា Routing, HttpFoundation, HttpKernel, Dependency Injection, Twig និង Console។

Symfony ជា framework ដែលមានរចនាសម្ព័ន្ធច្បាស់។ អ្នកសរសេរកូដជុំវិញ PHP objects ដូចជា `Request`, `Response`, Controllers និង Services។

## ២. តម្រូវការមុនចាប់ផ្តើម

Symfony 7.4 ត្រូវការ៖

- PHP 8.2 ឬថ្មីជាងនេះ។
- PHP extensions ដែលត្រូវការ ដូចជា Ctype, iconv, PCRE, Session, SimpleXML និង Tokenizer។
- Composer។
- Symfony CLI ជាជម្រើស ប៉ុន្តែមានប្រយោជន៍សម្រាប់បង្កើត និងបើក local server។

ពិនិត្យ requirements តាម Symfony CLI៖

```bash
symfony check:requirements
```

## ៣. បង្កើត Symfony Web Application

ដោយ Symfony CLI បង្កើត web application ដែលមាន packages សម្រាប់ web៖

```bash
symfony new php-symfony-app --version="7.4.*" --webapp
cd php-symfony-app
```

ជម្រើស `--webapp` ដំឡើង packages បន្ថែមសម្រាប់ traditional web app។ សម្រាប់ API ឬ microservice សាមញ្ញ អាចបង្កើត skeleton តិចជាងនេះ ហើយបន្ថែម packages តាមតម្រូវការ។

បើមិនប្រើ Symfony CLI អាចបង្កើតដោយ Composer៖

```bash
composer create-project symfony/skeleton:"7.4.*" php-symfony-app
cd php-symfony-app
composer require webapp
```

ចាប់ផ្តើម local server៖

```bash
symfony server:start
```

ចូល URL ដែល CLI បង្ហាញក្នុង browser។ បញ្ឈប់ server ដោយចុច Ctrl+C នៅ terminal ដែលកំពុងដំណើរការ។

## ៤. ស្គាល់ Project Structure

Project អាចមានថតសំខាន់ៗដូចជា៖

- `src/` — application PHP code រួមមាន Controllers និង Services។
- `config/` — configuration និង route/service setup។
- `public/` — web document root និង front controller `index.php`។
- `templates/` — Twig template files សម្រាប់ HTML (បើបានដំឡើង)។
- `var/cache/` — generated cache files។
- `var/log/` — application logs។
- `vendor/` — Composer dependencies ដែលគ្រប់គ្រងដោយ Composer។
- `composer.json` និង `composer.lock` — package requirements និង versions ដែលបាន resolve។

Web server គួរតែបម្រើតាម `public/`។ `var/cache` និង `var/log` ត្រូវអាចសរសេរបានដោយ process ដែលដំណើរការ app។ កុំកែឯកសារ `vendor/` ដោយដៃ។

ពិនិត្យ project information និង commands ដោយ៖

```bash
php bin/console about
php bin/console list
```

## ៥. បង្កើត Route និង Controller ដំបូង

បង្កើត `src/Controller/HelloController.php`៖

```php
<?php

namespace App\\Controller;

use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class HelloController
{
    #[Route('/hello', name: 'app_hello', methods: ['GET'])]
    public function index(): Response
    {
        return new Response('សួស្តីពី Symfony!');
    }
}
```

Symfony ប្រើ PHP Attribute `#[Route]` ដើម្បីភ្ជាប់ URL ទៅ Controller method។ Controller ទទួល request data តាម argument ឬ service injection ហើយត្រឡប់ `Response` object។

បើក `/hello` ក្នុង browser។ សាកពិនិត្យ route list៖

```bash
php bin/console debug:router
```

## ៦. Symfony Request-Response Flow

លំហូរទូទៅ៖

1. Browser ផ្ញើ HTTP request ទៅ web server។
2. Web server បញ្ជូន request ទៅ `public/index.php`។
3. Symfony បង្កើត `Request` object ហើយ HttpKernel ចាប់ផ្តើមដំណើរការ។
4. Router ស្វែងរក route ដែលត្រូវនឹង URL និង HTTP method។
5. Controller និង arguments ត្រូវបាន resolve ដោយ framework។
6. Controller បង្កើត `Response` (ឧទាហរណ៍ HTML, JSON ឬ file)។
7. Kernel និង event listeners អាចកែ response មុនបញ្ជូនត្រឡប់។
8. Web server ផ្ញើ response ទៅ browser។

HttpKernel ប្រើ events ជាច្រើនក្នុង lifecycle ដូចជា `kernel.request`, `kernel.controller`, `kernel.response` និង `kernel.exception` ដើម្បីអនុញ្ញាតឱ្យ framework និង extensions ចូលរួមនៅចំណុចសមស្រប។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត project `php-symfony-app` តាម Symfony 7.4 LTS។
2. ដំណើរការ `symfony check:requirements` និងកត់ត្រាលទ្ធផល។
3. បង្កើត `GET /hello` route ដោយ Route Attribute។
4. បន្ថែម `GET /courses` route ដែលត្រឡប់ JSON ដោយ `JsonResponse`។
5. ប្រើ `php bin/console debug:router` ដើម្បីរក route ទាំងពីរ។
6. សរសេរ request-response flow ជា ៦–៨ ជំហានដោយពាក្យផ្ទាល់ខ្លួន។
7. ពិនិត្យ project structure និងកំណត់ថា web root និង logs directory នៅឯណា។

## ៨. កំហុសដែលជួបញឹកញាប់

- **PHP version មិនគ្រប់**៖ ពិនិត្យ CLI PHP ដោយ `php -v` និង web server PHP version។
- **ភ្លេច `--webapp`**៖ project skeleton មិនមាន package ដែល traditional web app ត្រូវការ; អាចបន្ថែមតាម Composer។
- **Route មិនបង្ហាញ**៖ ពិនិត្យ Attribute namespace, class namespace និង `debug:router`។
- **បម្រើ project root ជា document root**៖ web server ត្រូវចង្អុលទៅ `public/` ដើម្បីកុំឱ្យបង្ហាញ files ខាងក្នុង។
- **Permissions ខុស**៖ web process ត្រូវសរសេរ `var/cache` និង `var/log` បាន។
- **កែ `vendor/` ដោយដៃ**៖ ការកែអាចបាត់ពេល Composer ដំឡើង ឬ update packages។
- **ច្រឡំ Symfony LTS និង latest stable**៖ ជ្រើស version តាម PHP version, support duration និង project constraints។

## សំណួរត្រួតពិនិត្យ

1. តើ Symfony 7.4 ត្រូវការ PHP version អប្បបរមាប៉ុន្មាន?
2. `--webapp` បន្ថែមអ្វី?
3. តើ Front Controller របស់ Symfony ស្ថិតនៅ file ណា?
4. Route Attribute ភ្ជាប់អ្វីទៅអ្វី?
5. តើ HttpKernel ផ្លាស់ប្តូរ Request ទៅជា object អ្វី?
6. ហេតុអ្វី document root ត្រូវជា `public/`?

## ឯកសារយោង

- [Symfony Releases](https://symfony.com/releases)
- [Symfony 7.4 Setup](https://symfony.com/doc/7.4/setup.html)
- [Symfony 7.4 Create Your First Page](https://symfony.com/doc/7.4/page_creation.html)
- [Symfony 7.4 HttpKernel Lifecycle](https://symfony.com/doc/7.4/components/http_kernel.html)
