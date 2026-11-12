# មេរៀនទី ៩២៖ Symfony Routes, Controllers និង HTTP Request/Response

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ភ្ជាប់ URL និង HTTP method ទៅ Controller ដោយ Route Attribute។
- ទទួល route parameters និង query parameters។
- បង្កើត HTML, text និង JSON responses។
- ដាក់ requirements និងឈ្មោះឱ្យ routes។
- ពិនិត្យ route configuration ដោយ `bin/console`។

## ១. Route និង Controller

Route កំណត់ថា URL និង HTTP method មួយត្រូវដំណើរការ controller method ណា។ Controller ទទួលព័ត៌មាន request ហើយត្រឡប់ `Response` object។

ក្នុង Symfony 7.4, PHP Attributes ជាវិធីងាយសម្រាប់កំណត់ route នៅជិត controller ដែលប្រើវា។

ឧទាហរណ៍ `src/Controller/CourseController.php`៖

```php
<?php

namespace App\\Controller;

use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class CourseController
{
    #[Route('/courses', name: 'course_index', methods: ['GET'])]
    public function index(): Response
    {
        return new Response('បញ្ជីវគ្គសិក្សា');
    }
}
```

ឈ្មោះ route `course_index` អាចប្រើបង្កើត URL ពីកូដ ដូច្នេះកុំចងជាប់នឹង URL string នៅគ្រប់កន្លែង។

## ២. Route Parameters និង Requirements

កំណត់ parameter ក្នុង path ដោយសញ្ញា `{}`៖

```php
#[Route(
    '/courses/{id}',
    name: 'course_show',
    requirements: ['id' => '\\d+'],
    methods: ['GET'],
)]
public function show(int $id): Response
{
    return new Response("វគ្គសិក្សាលេខ {$id}");
}
```

Requirement `\\d+` កំណត់ឱ្យ `id` មានតែលេខ។ បើ URL មិនត្រូវ requirement នោះ route មិន match ហើយ Symfony ត្រឡប់ 404។ ការកំណត់ HTTP method ជួយបែងចែកសំណើអានទិន្នន័យពីសំណើកែទិន្នន័យ។

Route parameters ត្រូវបានបញ្ជូនទៅ controller argument តាមឈ្មោះ parameter។ ប្រើ type declarations ដើម្បីឱ្យច្បាស់ថា controller រំពឹងតម្លៃប្រភេទអ្វី។

## ៣. Request Object និង Query Parameters

Symfony អាចបញ្ជូន `Request` object ទៅ Controller ដោយ type hint៖

```php
use Symfony\\Component\\HttpFoundation\\Request;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

#[Route('/courses/search', name: 'course_search', methods: ['GET'])]
public function search(Request $request): Response
{
    $query = $request->query->get('q', '');

    return new Response('ពាក្យស្វែងរក៖ '.htmlspecialchars(
        (string) $query,
        ENT_QUOTES,
        'UTF-8',
    ));
}
```

`$request->query` តំណាង query string parameters ដូចជា `?q=php`។ Request body មាន API ផ្សេងៗសម្រាប់ form ឬ JSON payload; ត្រូវ validate និង sanitize input មុនប្រើ ឬបង្ហាញវា។

កុំ echo user input ដោយមិន escape។ ឧទាហរណ៍ខាងលើប្រើ `htmlspecialchars()` ព្រោះវាបង្ហាញតម្លៃចូលក្នុង HTML/text response។

## ៤. JSON Response

ប្រើ `JsonResponse` សម្រាប់ JSON API៖

```php
use Symfony\\Component\\HttpFoundation\\JsonResponse;

#[Route('/api/courses/{id}', name: 'api_course_show', methods: ['GET'])]
public function apiShow(int $id): JsonResponse
{
    return new JsonResponse([
        'id' => $id,
        'title' => 'PHP មូលដ្ឋាន',
    ]);
}
```

`JsonResponse` encode array ទៅ JSON ហើយកំណត់ Content-Type សមស្រប។ កុំបញ្ជូន Model/entity ទាំងមូលដោយមិនជ្រើស fields ព្រោះអាចបង្ហាញព័ត៌មានមិនគួរឱ្យ API client ឃើញ។

## ៥. បង្កើត URL តាម Route Name

Controller ដែល extends `AbstractController` អាចប្រើ helper `generateUrl()`៖

```php
use Symfony\\Bundle\\FrameworkBundle\\Controller\\AbstractController;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class CourseController extends AbstractController
{
    #[Route('/courses/{id}', name: 'course_show', methods: ['GET'])]
    public function show(int $id): Response
    {
        $url = $this->generateUrl('course_show', ['id' => $id]);

        return new Response('URL៖ '.htmlspecialchars($url, ENT_QUOTES, 'UTF-8'));
    }
}
```

ក្នុង production code ជាទូទៅប្រើ URL generator ក្នុង template, redirect ឬ controller workflow តាមតម្រូវការ។ Route name ជួយឱ្យប្ដូរ URL path ដោយកូដដែលយោងតាមឈ្មោះមិនចាំបាច់កែទាំងអស់។

## ៦. ពិនិត្យ Routes

បង្ហាញ routes ដែល Symfony បានចុះឈ្មោះ៖

```bash
php bin/console debug:router
```

ពិនិត្យ route ជាក់លាក់៖

```bash
php bin/console router:match /courses/12
```

បើ `router:match` command មានក្នុង project version/configuration របស់អ្នក វាបង្ហាញថា route ណានឹង match URL នោះ។ ប្រើ `debug:router` ដើម្បីពិនិត្យឈ្មោះ, method និង path។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត `GET /courses` route ដែលត្រឡប់សារ ឬ HTML response។
2. បង្កើត `GET /courses/{id}` ដែលទទួល integer និងប្រើ numeric requirement។
3. បង្កើត `GET /courses/search?q=php` ដែលអាន query parameter។
4. បង្កើត `GET /api/courses/{id}` ដែលត្រឡប់ `JsonResponse`។
5. ដាក់ឈ្មោះ routes ហើយប្រើ route name ដើម្បីបង្កើត URL។
6. សាកល្បង URL ដែលមាន ID មិនមែនលេខ និងពន្យល់ response។
7. ពិនិត្យ routes ដោយ `php bin/console debug:router`។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ភ្លេចកំណត់ HTTP methods**៖ route អាចឆ្លើយតប method ដែលមិនចង់បាន។
- **Route parameter និង controller argument មិនត្រូវឈ្មោះ**៖ រក្សាឈ្មោះឱ្យស្របគ្នា។
- **មិនកំណត់ requirement**៖ route ជាច្រើនអាចប្រកួត match URL ដូចគ្នា។
- **មិន escape request input**៖ អាចបង្ក XSS នៅពេលបង្ហាញ HTML។
- **ត្រឡប់ array ជា text response ដោយផ្ទាល់**៖ ប្រើ `JsonResponse` សម្រាប់ JSON។
- **Route Attribute មិនបាន load**៖ ពិនិត្យ project setup និង `debug:router`។
- **បញ្ចេញ entity ទាំងមូលទៅ API**៖ ជ្រើស fields ដែលអនុញ្ញាត និងមានសុវត្ថិភាព។

## សំណួរត្រួតពិនិត្យ

1. Route និង Controller មានតួនាទីខុសគ្នាដូចម្ដេច?
2. តើ `#[Route]` Attribute កំណត់អ្វី?
3. `$request->query` មានទិន្នន័យអ្វី?
4. ហេតុអ្វី `JsonResponse` ល្អសម្រាប់ API response?
5. តើ route requirements ជួយអ្វី?
6. តើ command ណាបង្ហាញ routes ក្នុង Symfony?

## ឯកសារយោង

- [Symfony 7.4 Routing](https://symfony.com/doc/7.4/routing.html)
- [Symfony 7.4 Controllers](https://symfony.com/doc/7.4/controller.html)
- [Symfony 7.4 HttpFoundation Request/Response](https://symfony.com/doc/7.4/components/http_foundation.html)
