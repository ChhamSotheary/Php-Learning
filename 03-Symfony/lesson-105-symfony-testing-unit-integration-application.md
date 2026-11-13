# មេរៀនទី ១០៥៖ Symfony Testing — Unit និង Application Tests

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ស្គាល់ភាពខុសគ្នារវាង Unit, Integration និង Application tests។
- សរសេរ Unit test សម្រាប់ service។
- សរសេរ HTTP/Application test ដោយ WebTestCase។
- ដាក់ test database ដាច់ពី development និង production។

## ១. ប្រភេទ Test

- **Unit test** ផ្ទៀងផ្ទាត់ method ឬ class តូចៗ ដោយមិនចាប់ផ្ដើម application ទាំងមូល។
- **Integration test** ផ្ទៀងផ្ទាត់ថា services ច្រើនធ្វើការជាមួយគ្នា តាម Symfony kernel/container។
- **Application test** ផ្ញើ HTTP request ទៅកម្មវិធី ហើយពិនិត្យ status, response និង page behavior។

ចាប់ផ្ដើមពី test តូចៗសម្រាប់ business rules ហើយបន្ថែម Application tests សម្រាប់លំហូរដែលអ្នកប្រើពិតជាប្រើ។

## ២. ដំឡើង Test Pack

ក្នុង Symfony project៖

```bash
composer require --dev symfony/test-pack
```

Flex recipe ជាទូទៅរៀបចំ PHPUnit config និង `tests/` folder។ មើលថា test runner ដំណើរការដោយ៖

```bash
php bin/phpunit
```

## ៣. សរសេរ Unit Test

បើមាន service `PriceCalculator` ពីមេរៀន Services៖

```php
<?php

namespace App\Tests\Service;

use App\Service\PriceCalculator;
use PHPUnit\Framework\TestCase;

final class PriceCalculatorTest extends TestCase
{
    public function testItSumsPricesInCents(): void
    {
        $calculator = new PriceCalculator();

        self::assertSame(2000, $calculator->total([1200, 350, 450]));
    }
}
```

Unit test មិនចាំបាច់ boot Symfony kernel ទេ។ វារត់តែ PHP code ដែលកំពុងសាកល្បង ហើយការប្រើ `assertSame()` ពិនិត្យទាំងតម្លៃ និងប្រភេទ។

ដំណើរការ test file មួយ៖

```bash
php bin/phpunit tests/Service/PriceCalculatorTest.php
```

## ៤. Integration Test ជាមួយ Kernel

ប្រើ `KernelTestCase` នៅពេលត្រូវការយក service ពី Symfony container ឬសាកល្បង integration៖

```php
<?php

namespace App\Tests\Service;

use App\Service\PriceCalculator;
use Symfony\Bundle\FrameworkBundle\Test\KernelTestCase;

final class PriceCalculatorIntegrationTest extends KernelTestCase
{
    public function testServiceCanBeLoaded(): void
    {
        self::bootKernel();

        $calculator = static::getContainer()->get(PriceCalculator::class);

        self::assertInstanceOf(PriceCalculator::class, $calculator);
        self::assertSame(2000, $calculator->total([1200, 350, 450]));
    }
}
```

`static::getContainer()` ផ្ដល់ test container ដែលអនុញ្ញាតឱ្យ test ទាញ services សម្រាប់ការសាកល្បង។

## ៥. Application Test ជាមួយ WebTestCase

បង្កើត test ដែលសាកល្បង HTTP route៖

```php
<?php

namespace App\Tests\Controller;

use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

final class HomeControllerTest extends WebTestCase
{
    public function testHomePageLoads(): void
    {
        $client = static::createClient();
        $client->request('GET', '/');

        self::assertResponseIsSuccessful();
    }
}
```

`WebTestCase` ផ្ដល់ test client ដើម្បីផ្ញើ request និងពិនិត្យ response។ កែ path និង assertion ឱ្យត្រូវនឹង route ដែលកម្មវិធីរបស់អ្នកមាន។ Test application ជួយរកកំហុសក្នុងការភ្ជាប់ route, controller, template និង services។

## ៦. ប្រើ Database សម្រាប់ Test

បើ test អាន/កែទិន្នន័យ database ត្រូវកំណត់ database ផ្សេងសម្រាប់ environment `test`។ ឧទាហរណ៍ក្នុង `.env.test.local`៖

```dotenv
DATABASE_URL="mysql://test_user:test_password@127.0.0.1:3306/app_db_test?serverVersion=8.0.37"
```

មុនប្រើ ត្រូវពិនិត្យឱ្យប្រាកដថា database នេះមិនមែនជា production ឬ development database ដែលមានទិន្នន័យសំខាន់។ អាចបង្កើត test database និង schema តាម migrations៖

```bash
php bin/console --env=test doctrine:database:create
php bin/console --env=test doctrine:migrations:migrate --no-interaction
```

`.env.local` មិនត្រូវបានអានក្នុង test environment; ប្រើ `.env.test.local` សម្រាប់តម្លៃជាក់លាក់ក្នុងម៉ាស៊ីនរបស់អ្នក។ កុំ commit credentials ពិតទៅ Git។

## ៧. រក្សា Test ឱ្យឯករាជ្យ

Test មួយមិនគួរផ្លាស់ប្ដូរលទ្ធផលរបស់ test ផ្សេង។ ប្រើ database សាកល្បង, fixtures និង cleanup/reset strategy ដែលសមស្រប។ កុំអាស្រ័យលើលំដាប់ដែល PHPUnit រត់ tests។

មុនពេលប្រើ fixtures ឬ bundle សម្រាប់ transaction rollback សូមដំឡើង package និងអនុវត្ត setup តាម official docs របស់វា។ កុំរត់ command ដែល reset ឬលុបទិន្នន័យ លើ database ដែលមានទិន្នន័យត្រូវរក្សាទុក។

## ៨. រត់ Test Suite

រត់ tests ទាំងអស់៖

```bash
php bin/phpunit
```

រត់តែ folder ឬ file៖

```bash
php bin/phpunit tests/Controller
php bin/phpunit tests/Controller/HomeControllerTest.php
```

ពេល test បរាជ័យ អាន assertion/error message ហើយកំណត់ថាបញ្ហានៅក្នុង expected result, code ឬ test setup។ កុំប្ដូរ assertion ដើម្បីឱ្យ test pass ប្រសិនបើវាមិនឆ្លុះបញ្ចាំង behavior ដែលត្រូវការ។

## កំហុសដែលជួបញឹកញាប់

- **ដាក់ test class ខុសទីតាំង/namespace**៖ ពិនិត្យ PHPUnit autoload configuration និងឈ្មោះ file។
- **Unit test boot kernel ដោយមិនចាំបាច់**៖ វាធ្វើឱ្យ test យឺត និងពិបាកបំបែក unit។
- **Test ប្រើ production database**៖ កំណត់ `DATABASE_URL` សម្រាប់ test ហើយពិនិត្យ environment មុន run។
- **Tests ពឹងផ្អែកលើលំដាប់**៖ reset state ឬប្រើ data setup ក្នុង test នីមួយៗ។
- **Test route ខុស**៖ ពិនិត្យឈ្មោះ និង path តាម `php bin/console debug:router`។
- **លុប database ដើម្បី reset ដោយមិនពិនិត្យ environment**៖ ធ្វើតែជាមួយ test database ដែលអាចបោះចោលបាន។

## លំហាត់

1. បង្កើត Unit test សម្រាប់ `PriceCalculator::total()` ដោយ test array មានតម្លៃច្រើន និង array ទទេ។
2. បង្កើត WebTestCase សម្រាប់ទំព័រ Twig ដែលអ្នកបានបង្កើត។
3. សរសេរ test ពិនិត្យ 404 សម្រាប់ route ដែលស្វែងរក entity មិនឃើញ។
4. បង្កើត test database ដាច់ដោយឡែក ហើយបញ្ជាក់ថា PHPUnit ប្រើ environment `test`។
5. ពន្យល់ថា test មួយណាគួរប្រើសម្រាប់ logic របស់ service និងមួយណាសម្រាប់ HTTP route។

## សំណួរត្រួតពិនិត្យ

1. តើ Unit, Integration និង Application tests ខុសគ្នាដូចម្ដេច?
2. តើពេលណាត្រូវប្រើ `KernelTestCase`?
3. តើ `WebTestCase` ជួយអ្វី?
4. ហេតុអ្វី database សម្រាប់ test ត្រូវដាច់ដោយឡែក?
5. តើ test ឯករាជ្យមានន័យដូចម្ដេច?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Testing](https://symfony.com/doc/7.4/testing.html)
- [PHPUnit Documentation](https://docs.phpunit.de/)
