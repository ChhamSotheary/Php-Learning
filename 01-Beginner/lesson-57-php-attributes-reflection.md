# មេរៀនទី ៥៧៖ PHP Attributes និង Reflection

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី PHP Attribute ជា Metadata ដែលភ្ជាប់ទៅ Code។
- បង្កើត Custom Attribute ដោយ `#[Attribute]`។
- ប្រើ Reflection ដើម្បីអាន Attribute នៅ Runtime។
- យល់ពី Use Case ដូចជា Route/Validation Metadata។
- ជៀសវាងការប្រើ Reflection ដោយមិនចាំបាច់ ឬអនុញ្ញាតឱ្យ Input កំណត់ Code ដែលត្រូវហៅ។

> PHP Attributes មានចាប់ពី PHP 8.0។ Reflection អាចប្រើជាមួយ Attribute ដើម្បីស្វែងយល់ពី Metadata របស់ Class, Method, Property ឬ Parameter។

## ១. Attribute ជាអ្វី?

Attribute ជា Metadata ដែលសរសេរលើ Code ដោយ Syntax `#[...]`។ វាអាចផ្ទុក Argument ដែល Framework ឬកម្មវិធីអាននៅពេល Runtime។

ឧទាហរណ៍៖

```php
<?php
#[Attribute(Attribute::TARGET_METHOD)]
final class Route
{
    public function __construct(
        public string $method,
        public string $path
    ) {
    }
}

final class PostController
{
    #[Route('GET', '/posts')]
    public function index(): void
    {
        // បង្ហាញបញ្ជីអត្ថបទ
    }
}
```

`Attribute::TARGET_METHOD` កំណត់ថា Attribute នេះអាចភ្ជាប់លើ Method ប៉ុណ្ណោះ។ អាចកំណត់ Target ផ្សេងទៀត ឬ `Attribute::IS_REPEATABLE` ប្រសិនបើចង់អនុញ្ញាតឱ្យភ្ជាប់ច្រើនដង។

## ២. អាន Attribute ដោយ Reflection

Reflection អាចស្វែងរក Method និង Attribute ដែលភ្ជាប់លើវា៖

```php
<?php
$reflection = new ReflectionClass(PostController::class);

foreach ($reflection->getMethods() as $method) {
    $attributes = $method->getAttributes(Route::class);

    foreach ($attributes as $attribute) {
        /** @var Route $route */
        $route = $attribute->newInstance();

        printf(
            "%s %s -> %s::%s\n",
            $route->method,
            $route->path,
            $reflection->getName(),
            $method->getName()
        );
    }
}
```

`getAttributes()` ត្រឡប់ ReflectionAttribute Objects។ `newInstance()` បង្កើត Instance នៃ Attribute Class និងផ្តល់ Arguments។ បើត្រូវការឈ្មោះ/Argument ដោយមិនបង្កើត Instance អាចប្រើ `getName()` និង `getArguments()`។

## ៣. បង្កើត Route Registry

Framework អាចអាន Route Attributes នៅ Bootstrap ហើយបង្កើត Route Table។ នៅពេលទទួល Request ត្រូវផ្គូផ្គង Method និង Path ទៅ Handler ដែល Server បានកំណត់ជាមុន។

ចំណុចសំខាន់ៗ៖

- Validate HTTP Method/Path និងកុំឱ្យ Route ស្ទួនដោយមិនចេតនា។
- បង្កើត Registry ពី Class ដែលបាន Allowlist/Autoload ដោយ Server។
- កុំឱ្យ Request Parameter ជ្រើសឈ្មោះ Class/Method ដើម្បី Reflection ហៅដោយផ្ទាល់។
- បន្ទាប់ពី Route Match នៅតែត្រូវ Authenticate, Authorize និង Validate Input។
- Cache Route Metadata នៅ Deployment/Bootstrap ប្រសិនបើការអាន Reflection រាល់ Request មាន Cost។

Attribute ផ្តល់ Metadata; វាមិនអនុវត្ត Route, Security ឬ Validation ដោយស្វ័យប្រវត្តិទេ។

## ៤. Attribute និង PHPDoc Comment

មុន PHP 8 Framework ជាច្រើនប្រើ DocBlock ដូចជា `@Route` ហើយ Parse ជា String។ Native Attribute មាន Syntax របស់ភាសា និងអាចអានដោយ Reflection API។ ទោះយ៉ាងណា សូមជ្រើសរើសវិធីដែល Project/Framework ប្រើឱ្យស្របគ្នា; កុំបង្កើត Metadata System ថ្មីបើ Framework មានរួចហើយ។

## ៥. Reflection លើ Property ឬ Parameter

Reflection មិនកំណត់តែ Method ទេ។ អាចភ្ជាប់ Metadata ទៅ Property/Parameter ហើយអានដោយ ReflectionProperty/ReflectionParameter។ ឧទាហរណ៍ Framework អាចអាន Attribute សម្រាប់ Serialization ឬ Validation Rule។

ត្រូវកំណត់ Attribute Constructor និង Target ឱ្យច្បាស់ ហើយ Validate Argument នៅពេលបង្កើត Attribute។ កុំគិតថាតម្លៃ Metadata សុវត្ថិភាពគ្រាន់តែវាស្ថិតក្នុង Source Code; វាអាចខុសដោយ Bug ឬ Configuration។

## ៦. កំហុសដែលជួបញឹកញាប់

- ប្រើ Reflection ដើម្បីហៅ Method តាមឈ្មោះដែល User ផ្ញើមក។
- គិតថា Attribute បញ្ជាក់ថា Request មានសិទ្ធិ។
- មិនកំណត់ `Attribute::TARGET_*` ហើយភ្ជាប់ Attribute ខុសទីតាំង។
- ប្រើ Reflection សម្រាប់បញ្ហាដែលអាចដោះស្រាយដោយ Function/Configuration ធម្មតា។
- អាន Metadata រាល់ Request ដោយមិនគិតពី Bootstrap Cost។
- បង្កើត Custom Framework Annotation ទោះ Project ប្រើ Framework ដែលមាន Route System រួច។

## ៧. លំហាត់

1. បង្កើត Attribute `#[Cacheable(int $seconds)]` ដែលអាចភ្ជាប់លើ Method។
2. អាន Attribute ពី Method ដោយ Reflection និងបង្ហាញតម្លៃ Seconds។
3. បង្កើត Registry ដែលបដិសេធ Route Method/Path ស្ទួន។
4. ពន្យល់ថាហេតុអ្វី Request មិនគួរផ្ញើឈ្មោះ Method ហើយឱ្យ Reflection ហៅផ្ទាល់។
5. សាកល្បង Attribute លើ Target ខុស ហើយពិនិត្យ PHP Error។

## ៨. ចម្លើយណែនាំខ្លី

- បង្កើត Attribute Class ដោយដាក់ `#[Attribute(Attribute::TARGET_METHOD)]`។
- អានវាតាម Reflection Method `getAttributes(Route::class)`។
- ប្រើ `newInstance()` ដើម្បីទទួល Custom Attribute Object។
- Route Metadata មិនជំនួស Authorization ឬ Input Validation។
- Reflection ត្រូវធ្វើលើ Code ដែល Server កំណត់ មិនមែនលើ Class/Method ដែល Client ស្នើដោយផ្ទាល់។

## ឯកសារយោង

- [PHP Manual: Attributes](https://www.php.net/manual/en/language.attributes.php)
- [PHP Manual: ReflectionAttribute](https://www.php.net/manual/en/class.reflectionattribute.php)
- [PHP Manual: ReflectionAttribute::newInstance()](https://www.php.net/manual/en/reflectionattribute.newinstance.php)
- [មេរៀនទី ២២៖ Namespaces, Autoloading និង Composer](lesson-22-php-namespaces-autoloading-composer.md)
