
# មេរៀនទី ២៤៖ PSR-12 និង Unit Testing ដោយ PHPUnit

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពី Coding Standard និងមូលហេតុដែល Project គួរមានរចនាបថរួម។
- អនុវត្តចំណុចសំខាន់ៗរបស់ PSR-12។
- ដំឡើង PHP_CodeSniffer ដើម្បីពិនិត្យ Coding Style។
- យល់ពី Unit Test, Test Case និង Assertion។
- បង្កើត Test Class និងសរសេរ Tests ដោយ PHPUnit។
- ប្រើ Data Provider ដើម្បីសាកល្បង Input ច្រើនករណី។
- រត់ Tests តាម Command Line និងដោះស្រាយលទ្ធផល។

> កំណែ PHP និង PHPUnit ត្រូវគ្នាតាមតម្រូវការរបស់ Project។ នៅពេលសរសេរមេរៀននេះ PHPUnit 13 ត្រូវការ PHP 8.4 ឡើងទៅ។ PHPUnit 9 អាចប្រើជាមួយ PHP 8.0 ប៉ុន្តែបានផុតរយៈពេល Support ហើយ។ សម្រាប់ Project ថ្មី គួរប្រើ PHP Version និង PHPUnit Version ដែលនៅមាន Support។

---

## ១. Coding Standard ជាអ្វី?

Coding Standard ជាច្បាប់រួមសម្រាប់រចនាបថសរសេរកូដ ដូចជា៖

- របៀបដាក់វង់ក្រចក និងចន្លោះ។
- ការដាក់ឈ្មោះ Class, Method និង Variable។
- ការរៀបលំដាប់ Namespace និង Import Statements។
- ការបំបែក File និងបន្ទាត់កូដ។

Coding Standard មិនប្ដូររបៀបដំណើរការកម្មវិធីទេ។ វាជួយឱ្យក្រុមអាន និងកែសម្រួលកូដបានងាយស្រួល និងមានរចនាបថស្របគ្នា។

## ២. ចំណុចសំខាន់ៗក្នុង PSR-12

PSR-12 ជា Extended Coding Style Guide របស់ PHP-FIG ហើយពង្រីក PSR-1។ ចំណុចដែលជួបញឹកញាប់៖

- ប្រើ 4 Spaces សម្រាប់ Indentation មិនប្រើ Tab។
- Keywords និង Types សរសេរជាអក្សរតូច ដូចជា if, string និង int។
- Class Name ប្រើ PascalCase ដូចជា StudentRepository។
- Method Name ប្រើ camelCase ដូចជា findStudentById()។
- ដាក់ Opening Brace របស់ Class និង Method នៅបន្ទាត់ថ្មី។
- រៀប Declare, Namespace, Use Statements និង Code តាមលំដាប់។
- File ដែលមាន PHP Code តែប៉ុណ្ណោះ មិនចាំបាច់បិទ PHP Tag ចុងក្រោយទេ។
- បំបែក Statement មួយៗទៅបន្ទាត់ដាច់ដោយឡែក។

ឧទាហរណ៍៖

```php
<?php

declare(strict_types=1);

namespace LearningPhp\Domain;

final class Student
{
    public function __construct(
        private int $id,
        private string $name,
    ) {
    }

    public function displayName(): string
    {
        return $this->name;
    }
}
```

Coding Style មួយចំនួនអាចអាស្រ័យលើ Version របស់ PHP និងច្បាប់បន្ថែមរបស់ Project។ គួរអានឯកសារ Coding Standard ដែល Project ជ្រើសរើស មិនសន្មតថា Tool ទាំងអស់មានច្បាប់ដូចគ្នាទេ។

## ៣. ពិនិត្យ Style ដោយ PHP_CodeSniffer

PHP_CodeSniffer អាចពិនិត្យថា File អនុវត្ត Coding Standard ឬអត់។ បន្ថែមវាជា Development Dependency៖

```bash
composer require --dev squizlabs/php_codesniffer
```

ពិនិត្យ Source និង Test Files តាម PSR-12៖

```bash
vendor/bin/phpcs --standard=PSR12 src tests
```

PHP_CodeSniffer រាយការណ៍ទីតាំងដែលមិនតាមច្បាប់។ អាចប្រើ PHP Code Beautifier and Fixer ដើម្បីជួសជុលបញ្ហាដែល Tool អាចកែដោយសុវត្ថិភាព៖

```bash
vendor/bin/phpcbf --standard=PSR12 src tests
```

បន្ទាប់ពី Fix សូមអាន Diff ដើម្បីធានាថាការកែទម្រង់មិនប្ដូរ Logic ដែលមិនបានគ្រោងទុក។

## ៤. Unit Test ជាអ្វី?

Unit Test ពិនិត្យឯកតាតូចមួយនៃកម្មវិធី ដូចជា Function ឬ Method មួយ។ Test គួរផ្ដល់ Input ដែលដឹងជាមុន ហើយប្រៀបធៀបលទ្ធផលជាមួយអ្វីដែលរំពឹង។

ពាក្យសំខាន់ៗ៖

- **Test Case**៖ ករណីសាកល្បងមួយ។
- **Assertion**៖ ការអះអាងថាលទ្ធផលពិតត្រូវនឹងលទ្ធផលរំពឹង។
- **Test Suite**៖ ក្រុម Tests ដែលរត់ជាមួយគ្នា។
- **Unit Under Test**៖ Function, Method ឬ Class ដែលកំពុងត្រូវបានសាកល្បង។

ឧទាហរណ៍៖ ប្រសិនបើ Function បន្ថែម 2 និង 3 នោះ Test អះអាងថាលទ្ធផលត្រូវជា 5។

## ៥. ដំឡើង PHPUnit

កំណែ PHPUnit ត្រូវជ្រើសតាម PHP Version។ តាមតារាង Support បច្ចុប្បន្ន៖

- PHPUnit 13 ត្រូវការ PHP 8.4 ឡើងទៅ។
- PHPUnit 12 ត្រូវការ PHP 8.3 ឡើងទៅ។
- PHPUnit 9 អាចប្រើ PHP 8.0 ប៉ុន្តែបានផុត Support ហើយ។

សម្រាប់ Project ដែលប្រើ PHP 8.4 ឬថ្មីជាងនេះ អាចដំឡើង PHPUnit 13 ជា Development Dependency៖

```bash
composer require --dev phpunit/phpunit:^13
```

កំណត់ Composer Scripts ក្នុង composer.json ដើម្បីរត់ Tools ងាយស្រួល៖

```json
{
  "scripts": {
    "test": "phpunit",
    "style": "phpcs --standard=PSR12 src tests"
  },
  "require-dev": {
    "phpunit/phpunit": "^13",
    "squizlabs/php_codesniffer": "^3.0"
  }
}
```

Version Constraint របស់ Tool អាចត្រូវកែតាម PHP Version និង Project។ ដំណើរការ Composer Scripts ដោយ៖

```bash
composer test
composer style
```

## ៦. បង្កើត Class សម្រាប់សាកល្បង

បង្កើត File src/Domain/PriceCalculator.php៖

```php
<?php

declare(strict_types=1);

namespace LearningPhp\Domain;

use InvalidArgumentException;

final class PriceCalculator
{
    public function totalWithTax(int $priceInCents, int $taxPercent): int
    {
        if ($priceInCents < 0) {
            throw new InvalidArgumentException(
                'តម្លៃមិនអាចអវិជ្ជមានបានទេ។'
            );
        }

        if ($taxPercent < 0 || $taxPercent > 100) {
            throw new InvalidArgumentException(
                'អត្រាពន្ធត្រូវនៅចន្លោះ 0 និង 100។'
            );
        }

        return (int) round(
            $priceInCents * (1 + $taxPercent / 100)
        );
    }
}
```

ឧទាហរណ៍ប្រើចំនួនសេនជា Integer ដើម្បីជៀសវាងការរក្សាទុកប្រាក់ជា Floating Point ដោយផ្ទាល់។

បន្ថែម Autoload និង Test Configuration ទៅ composer.json៖

```json
{
  "autoload": {
    "psr-4": {
      "LearningPhp\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "LearningPhp\\Tests\\": "tests/"
    }
  }
}
```

បន្ទាប់ពីកែ Autoload Mapping ដំណើរការ៖

```bash
composer dump-autoload
```

## ៧. សរសេរ Test ដំបូង

បង្កើត File tests/Domain/PriceCalculatorTest.php៖

```php
<?php

declare(strict_types=1);

namespace LearningPhp\Tests\Domain;

use InvalidArgumentException;
use LearningPhp\Domain\PriceCalculator;
use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\TestCase;

final class PriceCalculatorTest extends TestCase
{
    public function testCalculatesTotalWithTax(): void
    {
        $calculator = new PriceCalculator();

        $total = $calculator->totalWithTax(10000, 10);

        self::assertSame(11000, $total);
    }

    public function testRejectsNegativePrice(): void
    {
        $calculator = new PriceCalculator();

        $this->expectException(InvalidArgumentException::class);

        $calculator->totalWithTax(-1, 10);
    }

    #[DataProvider('taxCases')]
    public function testCalculatesDifferentTaxRates(
        int $priceInCents,
        int $taxPercent,
        int $expected
    ): void {
        $calculator = new PriceCalculator();

        self::assertSame(
            $expected,
            $calculator->totalWithTax($priceInCents, $taxPercent)
        );
    }

    public static function taxCases(): iterable
    {
        yield 'no tax' => [10000, 0, 10000];
        yield 'ten percent' => [10000, 10, 11000];
        yield 'half tax' => [10000, 50, 15000];
    }
}
```

- Test Method ជាទូទៅចាប់ផ្ដើមដោយ test។
- assertSame() ប្រៀបធៀបទាំងតម្លៃ និងប្រភេទទិន្នន័យ។
- expectException() បញ្ជាក់ថាកូដត្រូវបោះ Exception ជាក់លាក់។
- Data Provider អនុញ្ញាតឱ្យរត់ Test Method ដដែលជាមួយ Input ច្រើនករណី។

## ៨. រត់ Tests

ដំណើរការ Test Suite ទាំងមូល៖

```bash
composer test
```

ឬដំណើរការ PHPUnit ដោយផ្ទាល់៖

```bash
vendor/bin/phpunit tests
```

លទ្ធផលបង្ហាញចំនួន Tests ដែលបានរត់ និងថាតើវា Passed ឬ Failed។ បើ Test បរាជ័យ PHPUnit បង្ហាញ Expected និង Actual ដើម្បីជួយរកមូលហេតុ។

អាចរត់ Test File តែមួយ៖

```bash
vendor/bin/phpunit tests/Domain/PriceCalculatorTest.php
```

## ៩. រចនាសម្ព័ន្ធ Project

ឧទាហរណ៍រចនាសម្ព័ន្ធដែលបែងចែក Source, Tests និង Composer៖

```text
student-app/
├── composer.json
├── composer.lock
├── src/
│   └── Domain/
│       └── PriceCalculator.php
└── tests/
    └── Domain/
        └── PriceCalculatorTest.php
```

PSR-4 Mapping ក្នុង composer.json ភ្ជាប់ LearningPhp\ ទៅ src/ និង LearningPhp\Tests\ ទៅ tests/។ Test Class និង Source Class មាន Namespace ផ្សេងគ្នា ដើម្បីកុំឱ្យ Tests ចូលក្នុង Autoload របស់ Production។

## ១០. តេស្តប្រភេទផ្សេងៗ

- **Unit Test** ពិនិត្យឯកតាតូចមួយដោយញែកវាចេញពី Database, Network និង File System។
- **Integration Test** ពិនិត្យថាផ្នែកច្រើនធ្វើការជាមួយគ្នាបាន ដូចជា Repository និង Test Database។
- **End-to-End Test** ពិនិត្យលំហូរកម្មវិធីពីចំណុចអ្នកប្រើចូលរហូតដល់លទ្ធផល។

ចាប់ផ្ដើមពី Unit Test សម្រាប់ច្បាប់សំខាន់ៗ។ Integration Test ត្រូវមាន Environment ដែលគ្រប់គ្រងបាន និងមិនប្រើទិន្នន័យ Production។

## ១១. Best Practices

1. Test លទ្ធផលដែលអ្នកប្រើកូដពឹងផ្អែក មិនចាំបាច់ Test Implementation Detail គ្រប់បន្ទាត់។
2. ឱ្យ Test នីមួយៗមានគោលបំណងច្បាស់ និងមិនពឹងលើលំដាប់រត់។
3. ប្រើ Test Names ដែលពន្យល់អាកប្បកិរិយាដែលរំពឹង។
4. ប្រើ Data Provider សម្រាប់ករណីច្រើនដែលមានរចនាសម្ព័ន្ធដូចគ្នា។
5. កុំភ្ជាប់ Unit Test ទៅ Database ឬ Network ពិត; ប្រើ Test Database ឬ Double តាមតម្រូវការ។
6. រត់ Tests ក្រោយកែ Logic និងមុន Commit។
7. រត់ Code Style Tool ជាផ្នែកនៃការផ្ទៀងផ្ទាត់កូដ។
8. Commit Composer Lock File របស់ Application ដើម្បីឱ្យ Test Environment ប្រើ Dependencies ដូចគ្នា។
9. កុំផ្ដល់ Credentials ពិតទៅ Test ដែលដំណើរការក្នុង CI។
10. ជ្រើស PHPUnit Version ដែលត្រូវនឹង PHP Version របស់ Project និងនៅមាន Support។

## ១២. កំហុសដែលជួបញឹកញាប់

- Test គ្មាន Assertion ដូច្នេះមិនបានពិនិត្យលទ្ធផលពិត។
- ប្រើ assertEquals() ពេលត្រូវការប្រៀបធៀបប្រភេទឱ្យតឹងរឹង; ពិចារណា assertSame()។
- Tests ពឹងផ្អែកលើលំដាប់រត់ ឬ State ដែល Test មុនទុកចោល។
- Unit Test ទៅ Database Production ឬសេវាខាងក្រៅ។
- កំណែ PHPUnit មិនត្រូវនឹង PHP Version។
- ភ្លេច Composer Autoload Mapping ឬ Composer Dump Autoload។
- គិតថា Code Style Check ជំនួស Tests; Tools ទាំងពីរពិនិត្យបញ្ហាផ្សេងៗ។
- កែ Test ឱ្យឆ្លងដោយបន្ថយការត្រួតពិនិត្យ ជំនួសឱ្យកែ Bug ពិត។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Test Class

សរសេរ Test សម្រាប់ Method totalWithTax() ដោយពិនិត្យតម្លៃធម្មតា និងអត្រាពន្ធសូន្យ។

### លំហាត់ទី ២៖ Test Exception

សរសេរ Test បញ្ជាក់ថា Method បោះ InvalidArgumentException ពេលអត្រាពន្ធលើស 100។

### លំហាត់ទី ៣៖ Data Provider

បន្ថែម Data Provider ដែលគ្របដណ្ដប់តម្លៃ និងអត្រាពន្ធយ៉ាងតិច 3 ករណី។

### លំហាត់ទី ៤៖ Style Tool

ដំឡើង PHP_CodeSniffer ជា require-dev ហើយរត់ PSR-12 Check លើ src/ និង tests/។

### លំហាត់ទី ៥៖ Composer Scripts

បន្ថែម Scripts test និង style ទៅ composer.json ហើយរត់តាម Composer។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Unit Test និង Integration Test និងហេតុអ្វី Unit Test គួរតែរហ័ស។

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- Unit Test ពិនិត្យឯកតាតូចមួយដាច់ដោយឡែកពី Services ផ្សេង។
- Integration Test ពិនិត្យថា Components ច្រើនធ្វើការជាមួយគ្នាបាន។
- assertSame() ពិនិត្យទាំងតម្លៃ និងប្រភេទ។
- expectException() កំណត់ Exception ដែល Test រំពឹងថានឹងកើត។
- Data Provider ផ្ដល់ Input ច្រើនទៅ Test Method មួយ។
- PSR-12 បង្កើតរចនាបថកូដរួម; PHPUnit ពិនិត្យអាកប្បកិរិយារបស់កម្មវិធី។ ទាំងពីរមិនជំនួសគ្នាទេ។

---

## សេចក្ដីសង្ខេប

- PSR-12 ជា Coding Style រួមសម្រាប់ PHP Projects។
- PHP_CodeSniffer ពិនិត្យរចនាបថ; PHPUnit ពិនិត្យឥរិយាបថកូដ។
- Unit Test ប្រើ Test Case និង Assertions ដើម្បីប្រៀបធៀបលទ្ធផលពិតនឹងលទ្ធផលរំពឹង។
- Data Provider អនុញ្ញាតឱ្យ Test Method មួយទទួល Input ច្រើនករណី។
- ត្រូវជ្រើស PHPUnit Version ដែលគាំទ្រ PHP Version របស់ Project។
- រត់ Tests និង Style Checks មុន Commit ឬ Deploy។

## ឯកសារយោងផ្លូវការ

- [PSR-12 Extended Coding Style](https://www.php-fig.org/psr/psr-12/)
- [PHPUnit Supported Versions](https://phpunit.de/supported-versions.html)
- [PHPUnit Writing Tests](https://docs.phpunit.de/en/13.4/writing-tests-for-phpunit.html)
- [Composer Basic Usage](https://getcomposer.org/doc/01-basic-usage.md)

## មេរៀនបន្ទាប់

អាចបន្តដោយរៀន Design Patterns និង Dependency Injection ដើម្បីរៀបចំ Components ដែលអាចសាកល្បង និងប្ដូរ Implementation បានងាយ។
