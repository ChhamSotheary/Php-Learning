# មេរៀនទី ១១៧៖ Portfolio Project ៧ — បង្កើត PHP Composer Package

## គោលបំណង

បង្កើត PHP library តូចមួយសម្រាប់ validate និង normalize email address ហើយរៀបចំវាជា Composer package ដែល project ផ្សេងអាចដំឡើងប្រើបាន។ មេរៀននេះអនុវត្ត PHP OOP, namespaces, PSR-4 autoloading, Composer metadata, semantic versioning, tests និងការចែកចាយ package។

គោលដៅមិនមែនសរសេរ email validator ដែលជំនួសការផ្ញើ verification email ទេ។ Syntax validation មិនអាចបញ្ជាក់ថាអាសយដ្ឋានមានម្ចាស់ប្រើបានឡើយ; ការផ្ទៀងផ្ទាត់ ownership ត្រូវធ្វើដោយផ្ញើ link ឬ code ទៅអាសយដ្ឋាននោះ។

## ១. កំណត់ API របស់ Library

ស្នើ public API តូច និងច្បាស់៖

- EmailAddress::parse(string $input): EmailAddress
- EmailAddress::value(): string
- EmailAddress::domain(): string
- InvalidEmailAddress exception សម្រាប់ input មិនត្រឹមត្រូវ

កំណត់ behavior ជាមុន៖ trim whitespace ខាងក្រៅ បដិសេធ string ទទេ និងបដិសេធ format ដែល FILTER_VALIDATE_EMAIL មិនទទួលស្គាល់។ កុំបម្លែង local part ទៅ lowercase ដោយស្វ័យប្រវត្តិ ព្រោះអត្ថន័យអាចអាស្រ័យលើប្រព័ន្ធ mail; តែ domain name មិនប្រកាន់អក្សរតូចធំ។ សរសេរ README បង្ហាញការសម្រេចចិត្តទាំងនេះ។

## ២. រចនាសម្ព័ន្ធ Package

~~~text
email-address/
├── src/
│   ├── EmailAddress.php
│   └── InvalidEmailAddress.php
├── tests/
│   └── EmailAddressTest.php
├── .github/
│   └── workflows/
│       └── tests.yml
├── composer.json
├── .gitignore
├── .gitattributes
├── LICENSE
└── README.md
~~~

Package ត្រូវមាន source code, tests, license និងឯកសារប្រើប្រាស់។ កុំដាក់ credentials ឬ private test data ក្នុង repository។ បន្ថែម tests និង workflow ដើម្បីរកឃើញបញ្ហាមុនបញ្ចេញ version ថ្មី។

## ៣. composer.json និង PSR-4

ឧទាហរណ៍ metadata៖

~~~json
{
  "name": "chhamsotheary/email-address",
  "description": "A small immutable PHP email address value object",
  "type": "library",
  "license": "MIT",
  "require": {
    "php": "^8.2"
  },
  "autoload": {
    "psr-4": {
      "ChhamSotheary\\EmailAddress\\": "src/"
    }
  },
  "autoload-dev": {
    "psr-4": {
      "ChhamSotheary\\EmailAddress\\Tests\\": "tests/"
    }
  }
}
~~~

សូមជ្រើស PHP constraint តាម version ដែល code ពិតប្រាកដគាំទ្រ ហើយកំណត់ PHP extensions ដែល library ត្រូវការ ប្រសិនបើមាន។ Prefix PSR-4 ត្រូវចប់ដោយ backslash ហើយ namespace/subnamespace ត្រូវផ្គូផ្គង path និង case របស់ filename។ Composer ប្រើ mapping នេះដើម្បីរក class ដោយមិនចាំបាច់ include file ដោយដៃ។

បន្ទាប់ពីកែ autoload mapping អាចបង្កើត Composer autoloader ដោយ៖

~~~bash
composer dump-autoload
~~~

## ៤. អនុវត្ត Value Object

ឧទាហរណ៍ class៖

~~~php
<?php

declare(strict_types=1);

namespace ChhamSotheary\EmailAddress;

final readonly class EmailAddress
{
    private function __construct(
        private string $value,
    ) {
    }

    public static function parse(string $input): self
    {
        $candidate = trim($input);

        if ($candidate === '' || filter_var($candidate, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidEmailAddress('The email address is invalid.');
        }

        return new self($candidate);
    }

    public function value(): string
    {
        return $this->value;
    }

    public function domain(): string
    {
        return substr(strrchr($this->value, '@') ?: '', 1);
    }
}
~~~

Exception class អាចមានទម្រង់៖

~~~php
<?php

declare(strict_types=1);

namespace ChhamSotheary\EmailAddress;

use InvalidArgumentException;

final class InvalidEmailAddress extends InvalidArgumentException
{
}
~~~

ប្រើ immutable object ដើម្បីឱ្យ email ដែលបាន parse មិនផ្លាស់ប្តូរបន្ទាប់ពីបង្កើត។ ពិនិត្យកម្រិត PHP ដែលត្រូវការ ព្រោះ readonly class ត្រូវការកំណែ PHP សមស្រប។ ប្រសិនបើ package គាំទ្រ PHP ចាស់ជាងនេះ សូមប្រើ class ធម្មតា និងរក្សា fields ជា private ដោយគ្មាន mutator។

## ៥. ប្រើ Package ពី Consumer Project

ក្នុង project សាកល្បងដែលមាន Composer៖

~~~bash
composer require chhamsotheary/email-address
~~~

បន្ទាប់មកប្រើ class៖

~~~php
<?php

require __DIR__ . '/vendor/autoload.php';

use ChhamSotheary\EmailAddress\EmailAddress;
use ChhamSotheary\EmailAddress\InvalidEmailAddress;

try {
    $email = EmailAddress::parse(' learner@example.com ');
    echo $email->value();
    echo $email->domain();
} catch (InvalidEmailAddress $exception) {
    echo 'Please enter a valid email address.';
}
~~~

កុំបង្ហាញ exception message ខាងក្នុងទៅ user នៅ production ប្រសិនបើវាអាចបង្ហាញព័ត៌មានដែលមិនចង់ចែករំលែក។ Consumer application អាចបម្លែង exception ទៅ validation message របស់ UI/API។

## ៦. Tests និង Compatibility

សរសេរ unit tests សម្រាប់៖

- តម្លៃ email ដែលត្រឹមត្រូវ។
- ការកាត់ whitespace ខាងក្រៅ។
- Input ទទេ និង format មិនត្រឹមត្រូវ។
- Domain extraction។
- Input ដែលមានអក្សរធំ/តូច ដោយផ្ទៀងផ្ទាត់ behavior ដែល README សន្យា។
- Class autoload បានពី Composer ដោយគ្មាន manual include។

បែងចែក dependencies សម្រាប់អ្នកប្រើ package និង developer៖ dependencies ដែល library ត្រូវការពេល runtime ដាក់ក្នុង require; PHPUnit និង tools សម្រាប់អ្នកអភិវឌ្ឍដាក់ក្នុង require-dev។ កុំដាក់ library tests ឬ development tools ក្នុង runtime API របស់ package។

កំណត់ lower bound របស់ PHP និង dependencies ឱ្យសមនឹង code។ CI គួរដំណើរការ tests លើ PHP versions ដែល package សន្យាគាំទ្រ។ បើ dependency constraints អនុញ្ញាត អាចមាន job មួយសាកល្បង lowest dependencies ផងដែរ។

## ៧. ការចេញផ្សាយ Version

អនុវត្ត semantic versioning៖

- MAJOR សម្រាប់ការផ្លាស់ប្តូរ public API ដែលបំបែក compatibility។
- MINOR សម្រាប់មុខងារថ្មីដែលរក្សា compatibility។
- PATCH សម្រាប់ bug fixes ដែលរក្សា compatibility។

មុន release៖

1. រត់ test suite និង static analysis ដែល project បានកំណត់។
2. ពិនិត្យ composer.json, README, LICENSE និង changelog។
3. បង្កើត Git tag ដូចជា v1.0.0។
4. បង្កើត GitHub release ដែលភ្ជាប់ changelog។
5. បញ្ជូន VCS repository ទៅ Packagist តាម account ដែលមានសិទ្ធិ និងរៀបចំ auto-update hook តាមឯកសារផ្លូវការ។
6. សាកដំឡើង package ពី consumer project មួយ មុនប្រកាសថា release រួចរាល់។

កុំចេញ stable version ប្រសិនបើ API មិនទាន់ស្ថិរភាព។ កុំលុប ឬផ្លាស់ប្ដូរ tag ដែលអ្នកប្រើប្រាស់បានដំឡើងរួចដោយគ្មានផែនការដោះស្រាយ។

## ៨. Documentation និង Package Quality

README ត្រូវមាន៖

- តម្រូវការ PHP និង extensions។
- Composer installation command។
- ឧទាហរណ៍ API ដែលដំណើរការបាន។
- Exceptions ដែលអាចកើតឡើង។
- គោលការណ៍ normalization និងដែនកំណត់របស់ validator។
- Test command និង contribution instructions។
- License និង supported PHP versions។

បន្ថែម .gitattributes export-ignore សម្រាប់ឯកសារ development ដែលមិនចាំបាច់ក្នុង distribution archive ប្រសិនបើចង់បាន package archive ស្រាល។ កុំ exclude LICENSE ឬឯកសារដែល consumer ត្រូវការអាន។

## ៩. Definition of Done

- Package មាន composer.json ត្រឹមត្រូវ និង PSR-4 autoload mapping។
- Public API និង behavior មាន tests និង documentation។
- Invalid input ត្រូវបានតំណាងដោយ exception ដែលអាចចាប់បាន។
- PHP/dependency constraints ស្របនឹង code និង CI matrix។
- Consumer project អាចដំឡើង package និង load class តាម Composer។
- Version/tag និង release notes ពន្យល់ពីការផ្លាស់ប្តូរ។
- README ពន្យល់ដែនកំណត់ថា syntax validation មិនមែន ownership verification។
- Repository មិនមាន secrets ឬ development files ដែលមិនចាំបាច់ក្នុង release archive។

## លំហាត់បន្ថែម

1. បន្ថែម interface សម្រាប់ email validator ហើយរក្សា API ឱ្យងាយពង្រីក។
2. បន្ថែម Unicode domain handling ដោយពិនិត្យ extension និង policy ឱ្យច្បាស់។
3. បន្ថែម static analysis និង code style check ក្នុង CI។
4. រៀបចំ changelog និង compatibility policy។
5. បង្កើត demo project ដែលប្រើ package ដោយ Composer path repository សម្រាប់ development មុន publish។

## សំណួរត្រួតពិនិត្យ

1. តើ PSR-4 mapping ភ្ជាប់ namespace ទៅ file path ដូចម្តេច?
2. ហេតុអ្វី runtime dependency ខុសពី require-dev?
3. តើ readonly value object ផ្ដល់អត្ថប្រយោជន៍អ្វី?
4. តើ semantic versioning កំណត់ MAJOR, MINOR និង PATCH ដូចម្តេច?
5. ហេតុអ្វី email syntax validation មិនអាចបញ្ជាក់ថាអ្នកប្រើគ្រប់គ្រង mailbox នោះ?

## ឯកសារយោង

- [Composer Libraries](https://getcomposer.org/doc/02-libraries.md)
- [Composer Schema: autoload និង package metadata](https://getcomposer.org/doc/04-schema.md)
- [Composer Basic Usage](https://getcomposer.org/doc/01-basic-usage.md)
- [PHP-FIG PSR-4 Autoloader](https://www.php-fig.org/psr/psr-4/)
- [PHP filter_var](https://www.php.net/manual/en/function.filter-var.php)
