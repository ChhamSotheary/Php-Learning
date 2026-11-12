# មេរៀនទី ៦៨៖ Magic Methods និង Object Serialization ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ស្គាល់ Magic Methods និងដឹងថាវាដំណើរការនៅពេលណា។
- ប្រើ `__serialize()`/`__unserialize()` ដើម្បីគ្រប់គ្រងទម្រង់ Object Serialization។
- Validate State ឡើងវិញពេល Restore Object។
- ការពារកុំឱ្យ Debug Output បង្ហាញ Secret។
- យល់ពីហានិភ័យខ្ពស់នៃការប្រើ `unserialize()` លើ Input មិនទុកចិត្ត។

> `__serialize()` និង `__unserialize()` មានចាប់ពី PHP 7.4។ Magic Method គួរប្រើតែពេលចាំបាច់ ព្រោះវាធ្វើឱ្យ Behavior កើតឡើងដោយស្វ័យប្រវត្តិ។

## ១. Magic Methods ជាអ្វី?

Magic Methods ជា Method ដែលមានឈ្មោះចាប់ផ្តើមដោយ `__` ហើយ PHP ហៅវាដោយស្វ័យប្រវត្តិនៅពេលមាន Event ជាក់លាក់។ ឧទាហរណ៍៖

- `__construct()` ពេលបង្កើត Object។
- `__toString()` ពេលបម្លែង Object ទៅ String។
- `__invoke()` ពេលហៅ Object ដូច Function។
- `__debugInfo()` ពេលបង្ហាញ Object ដោយ `var_dump()`។
- `__serialize()`/`__unserialize()` ពេល Serialize/Restore Object។

កុំប្រើ Magic Method ដើម្បីលាក់ API ដែលគួរតែជាឈ្មោះ Method ធម្មតា។ អ្នកអាន Code គួរអាចស្វែងយល់បានថា Behavior មកពីណា។

## ២. គ្រប់គ្រង Object Serialization

បើមានហេតុផលត្រូវ Serialize Object ដែលទុកចិត្តបាន អាចកំណត់ទិន្នន័យច្បាស់ដែលត្រូវរក្សា៖

```php
<?php
final class Money
{
    public function __construct(
        private int $minorUnits,
        private string $currency
    ) {
        $this->assertValid($minorUnits, $currency);
    }

    public function __serialize(): array
    {
        return [
            'minor_units' => $this->minorUnits,
            'currency' => $this->currency,
        ];
    }

    public function __unserialize(array $data): void
    {
        $minorUnits = $data['minor_units'] ?? null;
        $currency = $data['currency'] ?? null;

        if (!is_int($minorUnits) || !is_string($currency)) {
            throw new UnexpectedValueException('Serialized Money is invalid.');
        }

        $this->assertValid($minorUnits, $currency);
        $this->minorUnits = $minorUnits;
        $this->currency = $currency;
    }

    private function assertValid(int $minorUnits, string $currency): void
    {
        if (
            $minorUnits < 0
            || !preg_match('/^[A-Z]{3}$/', $currency)
        ) {
            throw new InvalidArgumentException('Money value is invalid.');
        }
    }
}
```

`__unserialize()` អាចត្រូវបានហៅពេល Object ត្រូវបាន Restore ដោយមិនឆ្លង Constructor ដូចធម្មតា ដូច្នេះត្រូវ Validate State ឡើងវិញ និងកុំសន្មតថា Invariant នៅតែត្រឹមត្រូវ។ Serialization Format ក៏អាចផ្លាស់ប្តូរតាម Version របស់ Class; កំណត់ Compatibility Policy ប្រសិនបើទិន្នន័យនេះត្រូវរក្សារយៈពេលវែង។

## ៣. កុំ Serialize Secret ក្នុង Debug Output

`__debugInfo()` អាចកំណត់អ្វីដែល `var_dump()` បង្ហាញ៖

```php
<?php
final class ApiCredential
{
    public function __construct(
        private string $clientId,
        private string $secret
    ) {
    }

    public function __debugInfo(): array
    {
        return [
            'clientId' => $this->clientId,
            'secret' => '[REDACTED]',
        ];
    }
}
```

នេះជាការកាត់បន្ថយការលេចធ្លាយតាម Debug Output តែប៉ុណ្ណោះ។ វាមិនការពារ Secret ពីការ Log តាមវិធីផ្សេង, Reflection, Memory Dump ឬការចូលប្រើ Object ដោយគ្មានសិទ្ធិទេ។ កុំ Log Object ដែលផ្ទុក Secret ទាំងមូលដោយស្វ័យប្រវត្តិ។

## ៤. Serialization មិនមែនជា JSON

PHP `serialize()` បង្កើត Format ជាក់លាក់របស់ PHP ដែលអាចរក្សា Object/Class State។ JSON ជា Data Format សម្រាប់ផ្លាស់ប្តូររវាងប្រព័ន្ធ និងមាន Schema ដែលអាចកំណត់បាន។

សម្រាប់ API, Cookie, Browser Storage និងទិន្នន័យពី User ជាទូទៅប្រើ JSON/ទម្រង់ស្តង់ដារដែលមាន Schema ជំនួស PHP Object Serialization។ កុំចាត់ JSON ជា Trusted ដោយស្វ័យប្រវត្តិ; ត្រូវ Validate ពេល Decode ដែរ។

## ៥. ហានិភ័យនៃ `unserialize()`

**កុំហៅ `unserialize()` លើ Input ដែលមិនទុកចិត្ត**—ទោះបីកំណត់ `allowed_classes` ក៏ដោយ។ Object Restoration អាចបង្កើត Object, រត់ Autoloading និងហៅ Magic Methods ដូចជា `__unserialize()`/`__wakeup()` ដែលអាចបង្ក Side Effect ឬ Object Injection។

```php
<?php
// សម្រាប់ទិន្នន័យ JSON ពី Client សូមប្រើ JSON Decoder និង Validate Schema។
try {
    $payload = json_decode(
        $requestBody,
        true,
        512,
        JSON_THROW_ON_ERROR
    );
} catch (JsonException) {
    http_response_code(400);
    exit('Request Data មិនត្រឹមត្រូវ។');
}
```

`allowed_classes => false` អាចកាត់បន្ថយការបង្កើត Class Objects ក្នុងករណីប្រើជាមួយទិន្នន័យ Legacy ដែលទុកចិត្ត ប៉ុន្តែមិនធ្វើឱ្យ Input ពី User មានសុវត្ថិភាពទេ។

## ៦. ពេលណាគួរប្រើ Magic Method?

ប្រើនៅពេលវាបំពេញកិច្ចសន្យាដែលអ្នកអាន Code រំពឹងទុក ដូចជា `__construct()`, `__toString()`, `__serialize()` ឬ `__debugInfo()`។ ជៀសវាង៖

- `__get()`/`__set()` ដែលលាក់ Property និង Typo។
- `__call()` ដែលលាក់ទុកថា Method ណាអាចហៅបាន។
- Magic Method ដែលធ្វើ Database/Network I/O ដោយមិនឃើញច្បាស់ពី Call Site។
- `__destruct()` ដែលមាន Side Effect សំខាន់ពិបាកគ្រប់គ្រង Lifecycle។

បើ API ធម្មតាអាចពន្យល់ Behavior បានច្បាស់ សូមប្រើ Method ធម្មតា។

## ៧. កំហុសដែលជួបញឹកញាប់

- ប្រើ `unserialize()` លើ Cookie, Request ឬ Cache ដែល User អាចកែបាន។
- គិតថា `allowed_classes => false` គ្រប់គ្រាន់សម្រាប់ Input មិនទុកចិត្ត។
- Serialize Password/API Secret ដោយមិនចាំបាច់។
- មិន Validate Data ក្នុង `__unserialize()`។
- បង្ហាញ Credential តាម `var_dump()` ឬ Log Object។
- ប្រើ Magic Methods ច្រើនរហូតធ្វើឱ្យ Behavior មិនច្បាស់។

## ៨. លំហាត់

1. បន្ថែម `__serialize()`/`__unserialize()` ទៅ Value Object និង Validate Field ឡើងវិញ។
2. បង្កើត Class Credential ដែល `__debugInfo()` លាក់ Secret។
3. Serialize Object ក្នុង Test ហើយ Restore ដើម្បីប្រៀបធៀប Value។
4. សរសេរ JSON API Decoder ដែល Validate Schema ជំនួស `unserialize()`។
5. ពន្យល់ពីមូលហេតុដែល Serialization Format អាចបែកពេល Class Structure ផ្លាស់ប្តូរ។

## ៩. ចម្លើយណែនាំខ្លី

- Magic Methods ជា Hook ដែល PHP ហៅស្វ័យប្រវត្តិ; ប្រើតែពេលមានអត្ថប្រយោជន៍ច្បាស់។
- `__unserialize()` ត្រូវ Validate Data និងរក្សា Class Invariants ឡើងវិញ។
- `__debugInfo()` ជួយ Mask Secret ពេល Debug តែមិនជំនួស Secret Handling។
- កុំប្រើ `unserialize()` លើ User-Controlled Data ទោះមាន `allowed_classes` ក៏ដោយ។
- ប្រើ JSON និង Schema Validation សម្រាប់ Data Exchange ជាមួយ Client។

## ឯកសារយោង

- [PHP Manual: Magic Methods](https://www.php.net/manual/en/language.oop5.magic.php)
- [PHP Manual: unserialize()](https://www.php.net/manual/en/function.unserialize.php)
- [PHP Manual: json_decode()](https://www.php.net/manual/en/function.json-decode.php)
- [មេរៀនទី ១៥៖ PHP JSON Handling](lesson-15-php-json-handling.md)
- [មេរៀនទី ៥៦៖ Enums និង Value Objects](lesson-56-php-enums-value-objects.md)
