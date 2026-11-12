# មេរៀនទី ៦៦៖ Traits និង Composition ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ថា Trait ជាយន្តការចែករំលែក Method ក្នុង PHP។
- ប្រើ Trait នៅ Class ច្រើនដោយមិនបង្កើត Parent Class ថ្មី។
- ដោះស្រាយ Method Name Conflict ដោយ `insteadof` និង `as`។
- បែងចែក Trait, Interface និង Composition។
- ជ្រើសរើសចែករំលែក Behavior ដោយមិនបង្កើត Trait ធំពេក។

## ១. Trait ជាអ្វី?

PHP មាន Single Inheritance សម្រាប់ Class។ Trait អនុញ្ញាតឱ្យបញ្ចូល Set នៃ Method ទៅ Class ច្រើនដែលមិនចាំបាច់មាន Parent Class ដូចគ្នា។ Trait មិនអាច Instantiate ដោយផ្ទាល់ ហើយមិនមែនជា Object/Service ដាច់ដោយឡែកទេ។

ឧទាហរណ៍៖

```php
<?php
trait HasTimestamps
{
    private ?DateTimeImmutable $createdAt = null;

    public function markCreated(DateTimeImmutable $at): void
    {
        $this->createdAt = $at;
    }

    public function createdAt(): ?DateTimeImmutable
    {
        return $this->createdAt;
    }
}

final class Article
{
    use HasTimestamps;

    public function __construct(
        public readonly string $title
    ) {
    }
}
```

Class ដែលប្រើ Trait ទទួល Method/Property របស់ Trait នៅក្នុង Class Scope។ Trait គួរមានទំហំតូច និងឈ្មោះបញ្ជាក់ Behavior ដែលវាផ្តល់។

## ២. Trait និង Interface មានតួនាទីខុសគ្នា

- **Interface** កំណត់ Contract ថា Class ត្រូវមាន Method/Signature អ្វី។
- **Trait** ផ្តល់ Implementation ដែលអាច Reuse បាន។
- **Composition/Dependency Injection** ផ្គុំ Object ដែលមានតួនាទីផ្សេងៗគ្នា។

Class អាច Implements Interface និងប្រើ Trait ព្រមគ្នា៖

```php
<?php
interface JsonSerializableEntity
{
    public function toArray(): array;
}

trait ConvertsToJson
{
    public function toJson(): string
    {
        return json_encode(
            $this->toArray(),
            JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE
        );
    }
}

final class Product implements JsonSerializableEntity
{
    use ConvertsToJson;

    public function __construct(
        private string $name,
        private int $priceInCents
    ) {
    }

    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'price_in_cents' => $this->priceInCents,
        ];
    }
}
```

Trait មិនបង្ខំឱ្យ Class មាន Contract ច្បាស់ដូច Interface ទេ។ បើ Class អ្នកប្រើត្រូវការ Method ជាក់លាក់ពី Trait អាចកំណត់ Interface ផង ហើយអនុវត្តវានៅ Class។

## ៣. ដោះស្រាយ Method Conflict

បើ Traits ច្រើនមាន Method ឈ្មោះដូចគ្នា ត្រូវដោះស្រាយ Conflict ដោយច្បាស់៖

```php
<?php
trait WritesJson
{
    public function export(): string
    {
        return 'json';
    }
}

trait WritesText
{
    public function export(): string
    {
        return 'text';
    }
}

final class Report
{
    use WritesJson, WritesText {
        WritesJson::export insteadof WritesText;
        WritesText::export as exportAsText;
    }
}
```

- `insteadof` ជ្រើសរើស Implementation ដែលឈ្នះ Conflict។
- `as` បន្ថែម Alias ឬប្តូរ Visibility នៃ Method។

កុំទុក Conflict ឱ្យមើលមិនឃើញ; សរសេរជម្រើសឱ្យច្បាស់ ដើម្បីឱ្យអ្នកអានដឹងថា Behavior មកពី Trait ណា។

## ៤. Trait អាចទាមទារ Method ពី Class

Trait អាចកំណត់ Abstract Method ដែល Class ត្រូវអនុវត្ត ដើម្បីផ្តល់ Dependency/Behavior ដែល Trait ត្រូវការ៖

```php
<?php
trait LogsActivity
{
    abstract protected function logger(): LoggerInterface;

    protected function logActivity(string $message): void
    {
        $this->logger()->info($message);
    }
}
```

Class ដែលប្រើ Trait ត្រូវផ្តល់ Method `logger()` តាម Signature។ ចំពោះ Dependency សំខាន់ ដូចជា Logger, Mailer ឬ Repository ជាញឹកញាប់ Composition/Constructor Injection បង្ហាញ Dependency បានច្បាស់ និងងាយ Test ជាង Trait ដែលលាក់ការពឹងផ្អែក។

## ៥. Trait ឬ Composition?

ប្រើ Trait ពេល៖

- មាន Behavior តូច និងស្របគ្នា ដែល Class ច្រើនត្រូវការដូចគ្នា។
- Behavior អាចធ្វើការជាមួយ Class Host ដោយ Contract ច្បាស់។
- មិនចាំបាច់មាន State/Dependency ស្មុគស្មាញ។

ប្រើ Composition/Dependency Injection ពេល៖

- Behavior មាន Lifecycle/State ឬ Dependency ផ្ទាល់ខ្លួន។
- ត្រូវការប្តូរ Implementation នៅ Runtime/Test។
- Class ត្រូវប្រើ Service ច្រើន ហើយចង់ឱ្យ Dependency មើលឃើញក្នុង Constructor។
- Trait ចាប់ផ្តើមមានច្រើន Method, Property ឬ Requirement ដែលលាក់។

ឧទាហរណ៍៖ Controller គួរទទួល `MailerInterface` តាម Constructor ជំនួសឱ្យប្រើ Trait ដែលបង្កើត/រក្សា Mailer នៅខាងក្នុង។

## ៦. កំហុសដែលជួបញឹកញាប់

- ប្រើ Trait ជំនួសគ្រប់ Class Inheritance/Dependency Injection។
- បង្កើត God Trait ដែលមាន State និង Method មិនពាក់ព័ន្ធច្រើន។
- មិនដឹងថា Trait មាន Property/Method ឈ្មោះប៉ះទង្គិចជាមួយ Class។
- លាក់ Dependency ក្នុង Trait ធ្វើឱ្យ Test/Configuration ពិបាក។
- គិតថា Trait ជា Interface ឬអាច Instantiate ដាច់ដោយឡែក។
- ប្រើ Alias/Conflict Rule ដោយមិនសរសេរច្បាស់។

## ៧. លំហាត់

1. បង្កើត Trait `HasUuid` ដែលផ្តល់ Method អាន UUID។
2. បង្កើតពីរ Traits ដែលមាន Method `format()` ហើយដោះស្រាយដោយ `insteadof`/Alias។
3. បង្កើត Interface សម្រាប់ Contract និង Trait សម្រាប់ Implementation Reusable។
4. ប្រៀបធៀប Logger ដែលភ្ជាប់ដោយ Trait និង Logger ដែល Inject តាម Constructor។
5. ពិនិត្យ Trait ក្នុង Project ថាតើវាមាន Dependency/State លាក់ឬអត់។

## ៨. ចម្លើយណែនាំខ្លី

- Trait ជួយ Reuse Implementation ក្នុង Class ច្រើន ប៉ុន្តែមិនមែនជា Object ឬ Contract ដោយខ្លួនឯង។
- Interface កំណត់ Contract; Trait ផ្តល់ Implementation; Composition ផ្គុំ Objects។
- ដោះស្រាយ Method Conflict ដោយ `insteadof` និង `as`។
- ប្រើ Constructor Injection សម្រាប់ Dependency សំខាន់ៗ និង Behavior ដែលមាន Lifecycle។
- រក្សា Traits ឱ្យតូច និងមានតួនាទីច្បាស់។

## ឯកសារយោង

- [PHP Manual: Traits](https://www.php.net/manual/en/language.oop5.traits.php)
- [PHP Manual: Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php)
- [មេរៀនទី ២១៖ Inheritance, Abstract Classes និង Interfaces](lesson-21-php-oop-inheritance-abstract-interfaces.md)
- [មេរៀនទី ២៥៖ Dependency Injection និង Design Patterns](lesson-25-php-dependency-injection-design-patterns.md)
