
# មេរៀនទី ២៣៖ Composer Dependencies, Version Constraints និង composer.lock

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់តួនាទី composer.json, composer.lock និង vendor/។
- បែងចែក composer install និង composer update។
- បន្ថែម Dependency ដោយ composer require និង Dependency សម្រាប់ Development ដោយ composer require --dev។
- អាន Version Constraints ដូចជា ^, ~ និង Range។
- ធ្វើឱ្យអ្នកអភិវឌ្ឍន៍ និង Production ប្រើ Package Versions ដូចគ្នា។
- ដោះស្រាយ Composer Update ដោយប្រុងប្រយ័ត្ន។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ និង Composer។ Version របស់ Package អាស្រ័យលើ Constraint និង Composer Repository នៅពេលដំឡើង។

---

## ១. Dependency ជាអ្វី?

Dependency គឺជា Package ឬ Library ដែលកម្មវិធីរបស់យើងត្រូវការដើម្បីដំណើរការ។ ឧទាហរណ៍ កម្មវិធីអាចប្រើ Library សម្រាប់ Logging, Email, Image Processing ឬ Testing។

Composer ជួយ៖

- កត់ត្រា Package ដែលកម្មវិធីត្រូវការ។
- ស្វែងរក Version ដែលស្របគ្នាជាមួយ Dependency ផ្សេងៗ។
- ទាញយក Packages ទៅ Folder vendor/។
- បង្កើត Autoloader សម្រាប់ Class របស់ Project និង Packages។
- រក្សាទុក Version ជាក់លាក់ ដើម្បីឱ្យ Environment ផ្សេងដំឡើងបានដូចគ្នា។

## ២. composer.json, composer.lock និង vendor/

### composer.json

កំណត់ព័ត៌មាន និងច្បាប់របស់ Project ដូចជា៖

- ឈ្មោះ Project។
- Version Constraint របស់ Dependencies។
- PHP និង PHP Extensions ដែលត្រូវការ។
- Autoload Mapping និង Scripts។

ឧទាហរណ៍៖

```json
{
  "name": "learning-php/student-app",
  "type": "project",
  "require": {
    "php": ">=8.0",
    "monolog/monolog": "^3.0"
  },
  "require-dev": {
    "phpunit/phpunit": "^10.0"
  }
}
```

Version ក្នុង composer.json ជា Constraint ដែលប្រាប់ Composer ថា Version ណាខ្លះអាចប្រើបាន មិនមែនចាំបាច់ជាលេខ Version ដែលបានដំឡើងពិតប្រាកដនោះទេ។

### composer.lock

Composer បង្កើត Lock File ដែលកត់ត្រា Version ជាក់លាក់របស់ Package ទាំងអស់ រួមទាំង Dependencies របស់ពួកវា។

- សម្រាប់ Application គួរ Commit composer.lock ទៅ Version Control ដើម្បីឱ្យអ្នកដទៃ និង Production ដំឡើង Version ដូចគ្នា។
- សម្រាប់ Library ដែលអ្នកដទៃនឹងដំឡើងជាផ្នែកមួយនៃ Project របស់ពួកគេ ជាទូទៅ Constraint ក្នុង composer.json ជាអ្វីដែលសំខាន់; Lock File មិនគ្រប់គ្រង Version របស់អ្នកប្រើប្រាស់ Library នោះទេ។

### vendor/

Folder នេះមានកូដ Packages ដែល Composer បានដំឡើង និង Autoloader ដែលបង្កើត។ ជាទូទៅមិន Commit vendor/ ទេ។ អ្នកដទៃអាចបង្កើតវាឡើងវិញដោយប្រើ composer install។

## ៣. composer install និង composer update ខុសគ្នាដូចម្តេច?

### composer install

ប្រើសម្រាប់ដំឡើង Dependencies តាម Lock File៖

```bash
composer install
```

បើមាន composer.lock Composer ប្រើ Version ដែលបាន Lock នោះ។ នេះសមស្របសម្រាប់៖

- Setup Project នៅម៉ាស៊ីនថ្មី។
- CI និង Test Environment។
- Deploy ទៅ Production។

បើគ្មាន Lock File Composer ដោះស្រាយ Constraints ហើយបង្កើត Lock File ថ្មី។

### composer update

ដោះស្រាយឡើងវិញនូវ Constraints ក្នុង composer.json និងសរសេរ Version ដែលបានជ្រើសទៅ composer.lock៖

```bash
composer update
```

វាអាចធ្វើឱ្យ Package មួយចំនួនប្ដូរ Version។ ដូច្នេះ ពិនិត្យ Diff របស់ composer.lock និងរត់ការផ្ទៀងផ្ទាត់កម្មវិធី មុន Commit។

ដើម្បី Update Package មួយ និង Dependencies ដែលវាត្រូវការ៖

```bash
composer update monolog/monolog --with-dependencies
```

Composer អាចកែ Packages ផ្សេងទៀត បើចាំបាច់ដើម្បីបំពេញ Constraint។ បន្ទាប់ពី Update ត្រូវពិនិត្យ Lock File, Release Notes របស់ Package និងផ្ទៀងផ្ទាត់កម្មវិធី។

**កុំប្រើ composer update ជាជំហានធម្មតានៅ Production។** Production គួរដំឡើងតាម Lock File ដោយ composer install។

## ៤. បន្ថែម និងលុប Package

ប្រើ composer require ដើម្បីបន្ថែម Package ទៅ require និងដំឡើងវា៖

```bash
composer require monolog/monolog
```

Composer នឹងកែ composer.json, ដោះស្រាយ Version, កែ composer.lock និងដាក់កូដក្នុង vendor/។

Package ដែលប្រើតែសម្រាប់ Test ឬ Code Quality គួរដាក់ក្នុង require-dev៖

```bash
composer require --dev phpunit/phpunit
```

លុប Package ដោយ៖

```bash
composer remove monolog/monolog
```

ពិនិត្យ Dependencies ដែលបានដំឡើង៖

```bash
composer show
```

កុំកែ composer.json និង composer.lock ដោយដៃ ប្រសិនបើមិនដឹងរចនាសម្ព័ន្ធពេញលេញ។ ប្រើ Composer Command ដើម្បីរក្សាឯកសារទាំងពីរឱ្យស៊ីគ្នា។

## ៥. Version Constraints

Constraint កំណត់ Version ដែល Composer អាចជ្រើសបាន។ Composer ប្រើ SemVer-style Versioning ជាទូទៅ៖

- Major Version អាចមាន Breaking Changes។
- Minor Version បន្ថែមមុខងារដោយរក្សា Compatibility តាមការសន្មត់ SemVer។
- Patch Version ជាទូទៅជាការកែ Bug ឬ Security Fix។

ទម្រង់សំខាន់ៗ៖

| Constraint | អត្ថន័យទូទៅ |
|---|---|
| 3.2.1 | Version ជាក់លាក់ |
| >=3.2 <4.0 | ចន្លោះ Version ដែលកំណត់ |
| 3.2.* | Version ក្នុងស៊េរី 3.2 |
| ~3.2.1 | ចាប់ពី 3.2.1 ដល់មុន 3.3.0 |
| ^3.2.1 | ចាប់ពី 3.2.1 ដល់មុន 4.0.0 |

Caret Constraint ដូចជា ^3.2.1 អនុញ្ញាតការកែប្រែដែលគេរំពឹងថាមិន Breaking តាមច្បាប់ SemVer ដោយមិនឡើងទៅ Major Version បន្ទាប់។ សម្រាប់ Version មុន 1.0, Composer កំណត់ជួរតឹងជាង ព្រោះ API អាចផ្លាស់ប្ដូរច្រើន។

Tilde Constraint ដូចជា ~3.2.1 អនុញ្ញាតឱ្យផ្នែកលេខចុងក្រោយដែលបានសរសេរឡើងទៅ ប៉ុន្តែរក្សាផ្នែកមុនឱ្យថេរ។

Constraint មិនធានាថា Package គ្មាន Bug ឬមានសុវត្ថិភាពទេ។ វាគ្រាន់តែជាក្បួនជ្រើសរើស Version ប៉ុណ្ណោះ។

## ៦. Platform Requirements

Project អាចតម្រូវ PHP Version និង Extensions៖

```json
{
  "require": {
    "php": ">=8.0",
    "ext-pdo": "*",
    "ext-pdo_mysql": "*",
    "ext-mbstring": "*"
  }
}
```

នេះប្រាប់ Composer ឱ្យពិនិត្យថា Environment មានតម្រូវការទាំងនេះ មុនដំឡើង Dependencies។ Extension មួយចំនួនអាចមិនមានក្នុង PHP Installation ដោយលំនាំដើម។

ពិនិត្យ Platform Packages៖

```bash
composer show --platform
```

កុំប្រើ --ignore-platform-reqs ដើម្បីលាក់បញ្ហា Compatibility នៅក្នុង Production ដោយគ្មានការវាយតម្លៃច្បាស់។

## ៧. ប្រើ Package បន្ទាប់ពី Install

បន្ទាប់ពីដំឡើង Package ត្រូវ Load Composer Autoloader នៅ Entry Point៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__) . '/vendor/autoload.php';

use Monolog\Logger;
use Monolog\Handler\StreamHandler;

$logger = new Logger('student-app');
$logger->pushHandler(new StreamHandler(__DIR__ . '/../var/app.log'));

$logger->warning('ឧទាហរណ៍ Log សម្រាប់រៀន');
```

Composer Autoloader ធ្វើឱ្យ Class របស់ Package អាចប្រើបានដោយមិន require File នីមួយៗដោយដៃ។ ឧទាហរណ៍នេះសន្មតថាបានដំឡើង Package ដែលផ្ដល់ Namespace និង Class ទាំងនេះ។

## ៨. Update Autoload Configuration

បន្ទាប់ពីកែ PSR-4 Mapping ក្នុង composer.json អាចបង្កើត Autoloader ឡើងវិញដោយ៖

```bash
composer dump-autoload
```

សម្រាប់ Production អាចប្រើ Autoloader Optimization៖

```bash
composer dump-autoload --optimize
```

Autoloader Optimization បង្កើត Class Map ដើម្បីធ្វើឱ្យការស្វែងរក Class មានប្រសិទ្ធភាពជាងមុនសម្រាប់ Deploy។ ការបន្ថែម Class ថ្មីក្នុង PSR-4 Folder ជាទូទៅមិនត្រូវការ Dump Autoload ម្ដងទៀតក្នុង Development ទេ; កែ Mapping ទើបត្រូវធ្វើ។

## ៩. Mini Project៖ ប្រើ Composer ក្នុង Student App

1. បង្កើត Project Folder និង composer.json។
2. បន្ថែម PSR-4 Mapping តាមមេរៀនទី ២២។
3. បន្ថែម Logging Package ដោយ៖

```bash
composer require monolog/monolog
```

4. បង្កើត public/index.php ដែល Require vendor/autoload.php។
5. បង្កើត Logger និងសរសេរ Log Message មួយ។
6. Commit composer.json និង composer.lock ទៅ Git។
7. លុប vendor/ នៅ Local Test Environment រួចដំណើរការ composer install ដើម្បីបង្កើតវាឡើងវិញ។

បើ Composer មិនអាចដំឡើង Package បាន សូមអាន Error Message ហើយពិនិត្យ PHP Version, Extensions និង Constraint។ កុំប្ដូរទៅ Constraint ទូលំទូលាយដោយមិនយល់ពីផលប៉ះពាល់។

## ១០. ការដំឡើងសម្រាប់ Production

សម្រាប់ Application ជាទូទៅដំឡើងតាម Lock File និងមិនដំឡើង Development Dependencies៖

```bash
composer install --no-dev --prefer-dist --optimize-autoloader
```

- --no-dev មិនដំឡើង Packages ក្នុង require-dev។
- --prefer-dist ស្នើឱ្យ Composer ប្រើ Distribution Archive បើអាច។
- --optimize-autoloader បង្កើនប្រសិទ្ធភាព Autoloader។

ធ្វើ Deploy ពី Source ដែលមាន composer.lock ហើយមិនត្រូវកែ Lock File នៅ Production Server ដោយដៃ។ គួរប្រើ Build/Deploy Process ដែលអាចធ្វើឡើងវិញបាន។

## ១១. Best Practices

1. Commit composer.json និង composer.lock សម្រាប់ Application។
2. ប្រើ composer install នៅម៉ាស៊ីនថ្មី, CI និង Production។
3. ប្រើ composer update ដោយចេតនា ពេលចង់ Update Dependency ហើយពិនិត្យ Diff។
4. ប្រើ Constraint ដែលមានព្រំដែនសមស្រប; ជៀសវាង * ឬ >= ដែលគ្មាន Upper Bound ដោយមិនចាំបាច់។
5. ដាក់ Test Tools ក្នុង require-dev។
6. កុំ Commit vendor/ ជាទូទៅ។
7. កុំ Commit Credentials ឬ Secret ក្នុង composer.json ឬ Repository។
8. ពិនិត្យ PHP និង Extension Requirements។
9. ក្រោយ Update Packages ត្រូវពិនិត្យ Release Notes និងសាកល្បងកម្មវិធី។
10. កុំដំឡើង Package ដោយមិនពិនិត្យ Maintainer, Source និងសមត្ថភាពដែលវាស្នើ។

## ១២. កំហុសដែលជួបញឹកញាប់

- ច្រឡំ composer install និង composer update។
- លុប composer.lock ហើយបង្កើត Version ថ្មីដោយមិនមានគោលបំណង។
- ភ្លេច Commit composer.lock សម្រាប់ Application។
- ប្រើ Version Constraint ទូលំទូលាយពេក ហើយទទួល Breaking Update នៅពេលក្រោយ។
- ដាក់ Test Dependency ក្នុង require ជំនួស require-dev។
- Commit vendor/ ទោះ Project ប្រើ Composer ដើម្បីដំឡើងវាឡើងវិញបាន។
- Update Package តែអត់ពិនិត្យ composer.lock Diff។
- មិនពិនិត្យ PHP Extension ដែល Package ត្រូវការ។
- គិតថា Lock File ជំនួសការថែទាំ Security Updates; ត្រូវ Update Dependencies តាមគោលការណ៍របស់ Project។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ អាន Version Constraint

ពន្យល់អត្ថន័យទូទៅរបស់ ^2.4.1, ~2.4.1 និង 2.4.*។

### លំហាត់ទី ២៖ Require និង Require-dev

ដាក់ Library ដែលកម្មវិធីត្រូវការពេលដំណើរការទៅ require និង Testing Tool ទៅ require-dev។

### លំហាត់ទី ៣៖ Install ឬ Update

Project មាន composer.lock ហើយ។ តើគួរប្រើ Command មួយណាដើម្បី Setup នៅម៉ាស៊ីនថ្មី? ហេតុអ្វី?

### លំហាត់ទី ៤៖ Platform Requirement

បន្ថែម PHP Version និង Extension ដែល Project ត្រូវការ ទៅ composer.json។

### លំហាត់ទី ៥៖ Update Package

ធ្វើ Update Package មួយ ហើយពិនិត្យថា composer.json និង composer.lock បានផ្លាស់ប្ដូរយ៉ាងដូចម្តេច។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ហេតុអ្វី Application គួរ Commit composer.lock និងហេតុអ្វី Production គួរប្រើ composer install។

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- ^2.4.1 ជាទូទៅអនុញ្ញាត Version ចាប់ពី 2.4.1 រហូតមុន 3.0.0។
- ~2.4.1 ជាទូទៅអនុញ្ញាត Version ចាប់ពី 2.4.1 រហូតមុន 2.5.0។
- 2.4.* អនុញ្ញាត Patch Releases ក្នុងស៊េរី 2.4។
- composer install ដំឡើង Version ដែលបានកត់ក្នុង Lock File។
- composer update ដោះស្រាយ Constraints ឡើងវិញ និងអាចកែ Lock File។
- Application Commit Lock File ដើម្បីឱ្យ Environment ផ្សេងទៀតដំឡើង Version ដូចគ្នា និងអាចធ្វើ Deploy ឡើងវិញបាន។

---

## សេចក្ដីសង្ខេប

- composer.json កំណត់ Dependencies និង Version Constraints។
- composer.lock កត់ត្រា Version ជាក់លាក់ដែលបានដោះស្រាយ។
- vendor/ មានកូដ Package និង Autoloader ដែល Composer បង្កើត។
- composer install ប្រើ Lock File; composer update ដោះស្រាយ Version ឡើងវិញ។
- ប្រើ require-dev សម្រាប់ Testing និង Development Tools។
- Commit Lock File សម្រាប់ Application ហើយ Deploy ដោយ composer install។
- ពិនិត្យ Constraint, Platform Requirements និង Package Changes មុនដាក់ឱ្យដំណើរការ។

## ឯកសារយោងផ្លូវការ

- [Composer Basic Usage](https://getcomposer.org/doc/01-basic-usage.md)
- [Composer Versions and Constraints](https://getcomposer.org/doc/articles/versions.md)
- [Composer Command Line](https://getcomposer.org/doc/03-cli.md)
- [Composer Platform Dependencies](https://getcomposer.org/doc/articles/composer-platform-dependencies.md)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្ស PSR Coding Standards, Testing និងការប្រើ PHPUnit ដើម្បីពិនិត្យកម្មវិធីដោយស្វ័យប្រវត្តិ។
