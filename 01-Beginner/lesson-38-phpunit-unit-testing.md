# មេរៀនទី ៣៨៖ Unit Testing ក្នុង PHP ដោយ PHPUnit

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីភាពខុសគ្នារវាង Unit Test និង Integration Test។
- រៀបចំ PHPUnit តាម Composer និងដំណើរការ Test។
- សរសេរ Test ជាមួយ Arrange, Act, Assert។
- ប្រើ Data Provider សម្រាប់តេស្ត Input ច្រើនករណី។
- រចនា Function ដែលងាយតេស្ត និងមិនពឹងលើ Global State ដោយមិនចាំបាច់។

> ប្រើ PHPUnit Version ដែលត្រូវគ្នានឹង PHP Version ក្នុង Project ហើយពិនិត្យ Documentation ផ្លូវការសម្រាប់ Version របស់អ្នក។

## ១. Unit Test ជាអ្វី?

Unit Test ពិនិត្យឥរិយាបថផ្នែកតូចមួយ ដូចជា Function ឬ Class Method ដោយផ្តល់ Input ហើយបញ្ជាក់ Output/Result ដែលរំពឹងទុក។ Test គួរដំណើរការលឿន និងអាចដំណើរការដដែលបាន ដោយមិនត្រូវការសេវាខាងក្រៅ។

- **Unit Test**៖ ពិនិត្យ Logic របស់ Unit តូចមួយ។
- **Integration Test**៖ ពិនិត្យថា Component ច្រើនដំណើរការរួមគ្នា ដូចជា PDO ជាមួយ Database។
- **End-to-End Test**៖ សាកល្បង Flow តាមប្រព័ន្ធទាំងមូលដូច User ប្រើ។

Test មិនបញ្ជាក់ថាគ្មាន Bug ទាំងស្រុងទេ; វាជួយចាប់ការផ្លាស់ប្តូរដែលបំបែកឥរិយាបថដែលបានកំណត់។

## ២. ដំឡើង PHPUnit

ក្នុង Project ដែលប្រើ Composer ដំឡើងជា Development Dependency៖

```bash
composer require --dev phpunit/phpunit
```

កំណត់ Script ក្នុង `composer.json`៖

```json
{
  "scripts": {
    "test": "phpunit"
  }
}
```

បង្កើត `phpunit.xml` សាមញ្ញ៖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit bootstrap="vendor/autoload.php" colors="true">
    <testsuites>
        <testsuite name="Application">
            <directory>tests</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

រចនាសម្ព័ន្ធអាចជា៖

```text
src/
  PriceCalculator.php
tests/
  PriceCalculatorTest.php
```

## ៣. សរសេរ Unit ដំបូង

ឯកសារ `src/PriceCalculator.php`៖

```php
<?php
declare(strict_types=1);

namespace App;

final class PriceCalculator
{
    public function total(int $unitPriceInCents, int $quantity): int
    {
        if ($unitPriceInCents < 0 || $quantity < 1) {
            throw new InvalidArgumentException('Price or quantity is invalid.');
        }

        return $unitPriceInCents * $quantity;
    }
}
```

ឯកសារ `tests/PriceCalculatorTest.php`៖

```php
<?php
declare(strict_types=1);

use App\PriceCalculator;
use PHPUnit\Framework\TestCase;

final class PriceCalculatorTest extends TestCase
{
    public function testCalculatesTotalInCents(): void
    {
        $calculator = new PriceCalculator();

        $total = $calculator->total(250, 3);

        self::assertSame(750, $total);
    }

    public function testRejectsQuantityBelowOne(): void
    {
        $calculator = new PriceCalculator();

        $this->expectException(InvalidArgumentException::class);
        $calculator->total(250, 0);
    }
}
```

ឧទាហរណ៍ខាងលើប្រើ `InvalidArgumentException` របស់ PHP; ក្នុង Namespace ត្រូវបន្ថែម `use InvalidArgumentException;` ឬសរសេរ `\\InvalidArgumentException`។

ដំណើរការ៖

```bash
composer test
```

## ៤. Arrange, Act, Assert

Test ជាច្រើនអាចរៀបចំជាបីដំណាក់កាល៖

1. **Arrange**៖ រៀបចំ Object និង Input។
2. **Act**៖ ហៅ Method/Function ដែលត្រូវសាកល្បង។
3. **Assert**៖ ប្រៀបធៀបលទ្ធផលនឹងអ្វីដែលរំពឹង។

ដាក់ឈ្មោះ Test ឱ្យបញ្ជាក់ឥរិយាបថ ដូចជា `testRejectsNegativePrice()`។ Test មួយគួរផ្តោតលើអាកប្បកិរិយាមួយ ដើម្បីឱ្យ Failure ងាយស្វែងរក។

## ៥. Data Provider

ប្រើ Data Provider ពេលត្រូវសាកល្បង Input ច្រើនប្រភេទ៖

```php
<?php
use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\TestCase;

final class SlugTest extends TestCase
{
    public static function validCases(): iterable
    {
        yield 'simple' => ['php-basics', true];
        yield 'with digits' => ['lesson-12', true];
        yield 'contains spaces' => ['bad slug', false];
        yield 'empty' => ['', false];
    }

    #[DataProvider('validCases')]
    public function testSlugValidation(string $value, bool $expected): void
    {
        self::assertSame($expected, isValidSlug($value));
    }
}
```

Syntax របស់ Attributes/Data Provider អាចខុសតាម PHPUnit Major Version; សូមប្រើ Syntax ក្នុង Documentation របស់ Version ដែល Project បានដំឡើង។

## ៦. រចនា Code ឱ្យងាយតេស្ត

- បំបែក Business Logic ពី HTTP, Database និង Email I/O។
- ផ្ញើ Dependency ចូល Constructor ជំនួសឱ្យបង្កើត Global Connection ក្នុង Method។
- ឱ្យ Function ទទួល Input ជាក់លាក់ ហើយត្រឡប់លទ្ធផលច្បាស់។
- Mock ឬ Fake External Dependency តែពេលចាំបាច់; កុំ Mock Logic របស់ Unit ខ្លួនឯង។
- កំណត់ Clock/Time Provider ប្រសិនបើ Test ពឹងលើពេលវេលា។
- កុំចង Test ទៅនឹង Private Implementation; តេស្ត Behavior ដែលអ្នកប្រើ Code រំពឹងទុក។

## ៧. កំហុសដែលជួបញឹកញាប់

- Test ទៅ Database/Internet ទាំងអស់ ហើយ Test យឺត ឬមិនស្ថិតស្ថេរ។
- Assert តែថា Method មិនបោះ Exception ដោយមិនពិនិត្យលទ្ធផលសំខាន់។
- Test មាន Dependency លំដាប់ពី Test មុន។
- Test លម្អិតតាម Internal Implementation ដូច្នេះ Refactor ធម្មតាធ្វើឱ្យ Test ខូច។
- មិនដាក់ Test ជា Composer Development Dependency។
- មិនអាន PHPUnit Version Constraint ហើយប្រើ PHP/ PHPUnit ដែលមិនឆបគ្នា។

## ៨. លំហាត់

1. សរសេរ Test សម្រាប់ Function បូកពន្ធលើតម្លៃជាសេន។
2. បន្ថែមករណី Input អវិជ្ជមាន និង Quantity សូន្យ។
3. ប្រើ Data Provider សាកល្បង Email/Slug Validator។
4. បំបែក Logic គណនាតម្លៃចេញពី Controller ដើម្បី Unit Test បាន។
5. ដំណើរការ Test Suite ដោយ Composer Script។

## ៩. ចម្លើយណែនាំខ្លី

- ដំឡើង PHPUnit ជា `--dev` dependency និងកំណត់ Test Suite នៅ `phpunit.xml`។
- Test មួយត្រូវរៀបចំ Input, ហៅ Code, ហើយ Assert Result/Exception។
- ប្រើ `assertSame()` ពេលត្រូវការប្រៀបធៀបទាំងតម្លៃ និង Type។
- ប្រើ Data Provider ដើម្បីឱ្យ Input Cases នីមួយៗបង្ហាញជាលទ្ធផល Test ដាច់ដោយឡែក។
- រក្សា Unit Test ឱ្យឯករាជ្យពី Database/Network; បង្កើត Integration Test ដាច់ដោយឡែកសម្រាប់ពិនិត្យការភ្ជាប់។

## ឯកសារយោង

- [PHPUnit Manual: Writing Tests](https://docs.phpunit.de/en/13.4/writing-tests-for-phpunit.html)
- [PHPUnit Manual](https://docs.phpunit.de/)
- [មេរៀនទី ២៤៖ PSR-12 និង Unit Testing ដោយ PHPUnit](lesson-24-psr12-and-phpunit-testing.md)
