# មេរៀនទី ៩៣៖ Symfony Twig Templates និង Layouts

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បំបែក HTML ចេញពី Controller ដោយប្រើ Twig templates។
- បញ្ជូន variables ពី Controller ទៅ template។
- ប្រើ Twig syntax សម្រាប់ loop និង condition។
- បង្កើត layout ដោយ template inheritance និង blocks។
- យល់ពី autoescaping និងហានិភ័យរបស់ `raw` filter។

## ១. Twig ជាអ្វី?

Twig ជា template engine ដែលប្រើជាទូទៅក្នុង Symfony ដើម្បីបង្កើត HTML។ វាបំបែក presentation ចេញពី Controller ហើយមាន syntax សម្រាប់បង្ហាញ variables, loops, conditions, includes និង layouts។

Template files ជាទូទៅស្ថិតក្នុង `templates/` និងប្រើ extension `.html.twig`។ បើ project skeleton មិនទាន់មាន Twig អាចដំឡើង package ដោយ៖

```bash
composer require twig
```

Project ដែលបង្កើតដោយ `--webapp` ជាទូទៅមាន Twig រួចជាស្រេច។

## ២. Render Template ពី Controller

Controller អាច extends `AbstractController` ដើម្បីប្រើ helper `render()`៖

```php
<?php

namespace App\\Controller;

use Symfony\\Bundle\\FrameworkBundle\\Controller\\AbstractController;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class CourseController extends AbstractController
{
    #[Route('/courses', name: 'course_index', methods: ['GET'])]
    public function index(): Response
    {
        $courses = [
            ['title' => 'PHP មូលដ្ឋាន', 'level' => 'beginner'],
            ['title' => 'Symfony ចាប់ផ្តើម', 'level' => 'beginner'],
        ];

        return $this->render('course/index.html.twig', [
            'courses' => $courses,
        ]);
    }
}
```

ឈ្មោះ `course/index.html.twig` សំដៅទៅ `templates/course/index.html.twig`។ Array keys ដែលបានបញ្ជូនពី Controller អាចប្រើជាអថេរក្នុង template។

## ៣. Twig Variables, Conditions និង Loops

បង្កើត `templates/course/index.html.twig`៖

```twig
<h1>វគ្គសិក្សា</h1>

{% if courses is empty %}
    <p>មិនទាន់មានវគ្គសិក្សាទេ។</p>
{% else %}
    <ul>
        {% for course in courses %}
            <li>
                {{ course.title }}
                <small>({{ course.level }})</small>
            </li>
        {% endfor %}
    </ul>
{% endif %}
```

Syntax សំខាន់ៗ៖

- `{{ value }}` — បង្ហាញតម្លៃ។
- `{% if ... %}` — បង្ហាញដោយផ្អែកលើលក្ខខណ្ឌ។
- `{% for ... %}` — ធ្វើ loop លើ collection។
- `{# comment #}` — comment ដែលមិនបង្ហាញនៅ HTML output។

រក្សា business logic ធ្ងន់នៅក្នុង Controller/service/model layer; Twig គួរផ្តោតលើ presentation។

## ៤. Template Inheritance និង Layout

បង្កើត `templates/base.html.twig`៖

```twig
<!doctype html>
<html lang="km">
<head>
    <meta charset="utf-8">
    <title>{% block title %}កម្មវិធី Symfony{% endblock %}</title>
</head>
<body>
    <header>
        <a href="{{ path('course_index') }}">វគ្គសិក្សា</a>
    </header>

    <main>
        {% block body %}{% endblock %}
    </main>
</body>
</html>
```

ទំព័រអាច extend layout៖

```twig
{% extends 'base.html.twig' %}

{% block title %}បញ្ជីវគ្គសិក្សា{% endblock %}

{% block body %}
    <h1>វគ្គសិក្សា</h1>
    {% for course in courses %}
        <p>{{ course.title }}</p>
    {% endfor %}
{% endblock %}
```

`{% extends %}` ជ្រើស parent template ហើយ `{% block %}` កំណត់តំបន់ដែល child template អាចផ្លាស់ប្តូរ។ Child template មិនគួរដាក់ content ចេញពី blocks ដែល parent អនុញ្ញាត។

## ៥. Include និង Route URLs

អាចបំបែកផ្នែកតូចទៅ partial template ដោយ `include()`៖

```twig
{{ include('course/_card.html.twig', {
    course: course
}) }}
```

បង្កើត URL ដោយ route name តាម `path()`៖

```twig
<a href="{{ path('course_show', {id: course.id}) }}">
    {{ course.title }}
</a>
```

ប្រើ route name ជំនួសការសរសេរ URL string ដោយដៃ ដើម្បីឱ្យការកែ route មិនបំបែក link នៅគ្រប់ template។

## ៦. សុវត្ថិភាព៖ Autoescaping

Symfony/Twig បើក HTML output escaping ដោយស្វ័យប្រវត្តិសម្រាប់ HTML templates។ ដូច្នេះ `{{ course.title }}` បម្លែងតួអក្សរដូចជា `<` និង `>` ទៅ HTML entities ដើម្បីជួយការពារ XSS។

`|raw` បិទ escaping៖

```twig
{{ course.description|raw }}
```

ប្រើ `raw` តែពេល HTML ត្រូវបានជឿទុកចិត្ត និង sanitize រួចតាមរបៀបត្រឹមត្រូវ។ កុំប្រើវាលើ user-generated content ឬ HTML ដែលមកពី request។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត route និង controller ដែល render `course/index.html.twig`។
2. បញ្ជូន array នៃ courses ទៅ template។
3. ប្រើ `for` loop និង condition សម្រាប់ករណី list ទទេ។
4. បង្កើត `base.html.twig` ដែលមាន title និង body blocks។
5. ប្រើ `path()` ដើម្បីបង្កើត link ទៅ named route។
6. បង្កើត course card partial ដោយ `include()`។
7. បញ្ចូល string ដែលមាន `<script>` ទៅ template ដោយ `{{ }}` ហើយពិនិត្យថា Twig escape វា។
8. ពន្យល់ថាហេតុអ្វី `|raw` អាចបង្ក XSS។

## ៨. កំហុសដែលជួបញឹកញាប់

- **Template path មិនត្រូវ**៖ `course/index.html.twig` ត្រូវនឹង `templates/course/index.html.twig`។
- **ភ្លេចបញ្ជូន variable ពី Controller**៖ ត្រូវបន្ថែម key/value ក្នុង `render()`។
- **ភ្លេច `{% endfor %}` ឬ `{% endif %}`**៖ ពិនិត្យ directive pair។
- **Child template ដាក់ content ក្រៅ block**៖ content ដែលត្រូវប្ដូរត្រូវដាក់ក្នុង block។
- **ប្រើ `|raw` លើ input មិនទុកចិត្ត**៖ អាចបង្ក XSS។
- **សរសេរ URL ជា string ក្នុងគ្រប់ template**៖ ប្រើ `path()` និង route names។
- **ដាក់ database/business logic ក្នុង Twig**៖ ផ្លាស់ទៅ Controller ឬ service។

## សំណួរត្រួតពិនិត្យ

1. Symfony រក្សា Twig templates នៅទីតាំងណា?
2. `render()` បញ្ជូនទិន្នន័យទៅ template ដូចម្ដេច?
3. `{% extends %}` និង `{% block %}` មានតួនាទីអ្វី?
4. តើ `path()` ជួយអ្វី?
5. ហេតុអ្វី `{{ value }}` មានសុវត្ថិភាពជាង `{{ value|raw }}` សម្រាប់ input ពី user?

## ឯកសារយោង

- [Symfony 7.4 Creating and Using Templates](https://symfony.com/doc/7.4/templates.html)
- [Symfony 7.4 Controllers](https://symfony.com/doc/7.4/controller.html)
- [Symfony 7.4 Routing](https://symfony.com/doc/7.4/routing.html)
