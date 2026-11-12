# មេរៀនទី ៤៧៖ CI/CD សម្រាប់ PHP ដោយ GitHub Actions

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Continuous Integration (CI) និង Continuous Delivery/Deployment (CD)។
- បង្កើត GitHub Actions Workflow សម្រាប់ PHP Project។
- Validate Composer Files, ដំឡើង Dependency និងរត់ Test ដោយស្វ័យប្រវត្តិ។
- ប្រើ Permission តូចបំផុត និងកុំដាក់ Secret ក្នុង Workflow។
- កែ Workflow ឱ្យសមនឹង PHP Version និង Extensions របស់ Project។

## ១. CI/CD ជាអ្វី?

**Continuous Integration (CI)** រត់ការពិនិត្យស្វ័យប្រវត្តិពេលមាន Push ឬ Pull Request ដើម្បីរកបញ្ហាមុន Merge។  
**Continuous Delivery/Deployment (CD)** រៀបចំ Artifact ឬ Deploy ទៅបរិស្ថានបន្ទាប់ បន្ទាប់ពី Check ជោគជ័យ និងតាម Approval/Policy របស់ក្រុម។

មេរៀននេះផ្តោតលើ CI: Validate Composer, Install Dependency និង Test។ Deployment ត្រូវការការកំណត់ Environment/Secret និងការអនុម័តតាម Platform ដាច់ដោយឡែក។

## ២. ដាក់ Workflow ក្នុង Repository

GitHub Actions Workflow ជា YAML នៅថត `.github/workflows/`។ បង្កើតឯកសារ `.github/workflows/php.yml`៖

```yaml
name: PHP CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      fail-fast: false
      matrix:
        php-version: ['8.2', '8.3']

    steps:
      - name: Check out source
        uses: actions/checkout@v4

      - name: Set up PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php-version }}
          tools: composer:v2
          coverage: none

      - name: Validate Composer files
        run: composer validate --strict

      - name: Install dependencies
        run: composer install --prefer-dist --no-progress --no-interaction

      - name: Run test suite
        run: composer test
```

កំណត់ PHP Version Matrix ឱ្យស្របនឹង `composer.json` និង Version ដែល App គាំទ្រ។ បើ Project មិនមាន `composer test` Script សូមបន្ថែមវា ឬប្រើ Command របស់ Test Runner របស់ Project ដូចជា `vendor/bin/phpunit`។

## ៣. Composer Script និង Lock File

ឧទាហរណ៍ `composer.json` មាន Test Script៖

```json
{
  "scripts": {
    "test": "phpunit"
  }
}
```

Commit `composer.lock` ដើម្បីឱ្យ CI ដំឡើង Dependency Version ដដែល។ CI ប្រើ `composer install` ដើម្បីគោរព Lock File; ការរត់ `composer update` ក្នុង CI អាចធ្វើឱ្យ Dependency ផ្លាស់ប្តូរមិនបានគ្រប់គ្រង។

បើ Project មាន PHP Extensions ត្រូវការបន្ថែមវានៅ Setup Step ឧទាហរណ៍ `extensions: mbstring, pdo_mysql` ហើយបញ្ជាក់ Extension តម្រូវការនៅ `composer.json` ផងដែរ។

## ៤. ដំណើរការ Test Database

បើ Integration Test ត្រូវការ Database៖

1. បន្ថែម Database Service Container ក្នុង Workflow ឬប្រើ Test DB ដែល Platform កំណត់។
2. បង្កើត Database ដាច់ដោយឡែកសម្រាប់ Job នោះ។
3. ដាក់ Test Credential ក្នុង Environment សម្រាប់ Step/Test ប៉ុណ្ណោះ។
4. ដំណើរការ Migration និង Integration Suite។
5. កុំប្រើ Production Credential ឬ Production Data។

Environment Variable សម្រាប់ Test អាចកំណត់ក្នុង Workflow ជា Dummy Value សម្រាប់ Container Local CI។ Secret សម្រាប់ External Service មិនគួរត្រូវបានបញ្ជូនទៅ Pull Request ពី Fork ដែលមិនទុកចិត្ត។ បង្កើត Job/Workflow ដាច់ដោយឡែកសម្រាប់ Test ដែលត្រូវការ Secret និងគោរព GitHub Security Policy។

## ៥. ការគ្រប់គ្រង Permission និង Actions

- កំណត់ Workflow `permissions` ត្រឹមតែអ្វីដែលត្រូវការ; CI ដែលអាន Source ជាទូទៅត្រូវការ `contents: read`។
- ពិនិត្យ Source និង Version របស់ Third-party Action ដែលប្រើ។
- អាច Pin Action ទៅ Commit SHA សម្រាប់ការគ្រប់គ្រង Supply Chain តឹងជាង Tag។
- កុំបោះពុម្ព Secret ក្នុង Log ឬ Command Output។
- កុំឱ្យ Workflow ពី Pull Request ដែលមិនទុកចិត្តមាន Permission សរសេរ Repository ឬចូល Secret ដោយមិនចាំបាច់។
- កំណត់ Branch Protection ឱ្យត្រូវការ CI Checks មុន Merge។

## ៦. បន្ថែម Static Checks

ក្រោយ Test ដំណើរការ អាចបន្ថែម៖

- PHP Syntax Check ឬ Linter។
- PHPStan/Psalm សម្រាប់ Static Analysis។
- Formatter Check ដូចជា PHP_CodeSniffer។
- Dependency Audit តាម Tool/Policy របស់ Project។
- Test Matrix តាម PHP Version និង Database ដែលគាំទ្រ។

បន្ថែម Check ម្តងមួយៗ ហើយធានាថា Developer អាចរត់ Command ដូចគ្នានៅ Local ដើម្បី Debug Failure បាន។

## ៧. ពិនិត្យលទ្ធផល Workflow

ពេល Push ឬបើក Pull Request៖

1. ចូល Repository → Actions។
2. បើក Run ថ្មី ដើម្បីមើល Job និង Step។
3. ពិនិត្យ Log នៃ Step ដែលបរាជ័យ; កុំបង្ហោះ Log ដែលមាន Secret សាធារណៈ។
4. កែបញ្ហា ហើយ Push ម្តងទៀត។
5. បើ Check ជោគជ័យ ត្រូវការពារ Branch ឱ្យ CI Check ជា Required មុន Merge។

CI បញ្ជាក់តែ Command ដែល Workflow បានរត់។ វាមិនធានាថាគ្មាន Vulnerability ឬ Test Coverage គ្រប់ករណីទេ។

## ៨. កំហុសដែលជួបញឹកញាប់

- ភ្លេច Commit `composer.lock` ធ្វើឱ្យ Dependency មិនស្ថិតស្ថេរ។
- Workflow ដំឡើង Dependency ប៉ុន្តែមិនរត់ Test។
- PHP Matrix Version មិនត្រូវនឹង Version Constraint របស់ Project។
- Integration Test ប្រើ Production Database/Secret។
- ដាក់ Secret ក្នុង YAML ឬបង្ហាញវាក្នុង Log។
- អនុញ្ញាត Workflow Pull Request ពី Fork ចូល Secret ដោយមិនពិនិត្យ Permission។
- CI ជោគជ័យ ប៉ុន្តែ Branch Protection មិនតម្រូវឱ្យ CI ជោគជ័យមុន Merge។

## ៩. លំហាត់

1. បង្កើត `.github/workflows/php.yml` តាមឧទាហរណ៍។
2. បន្ថែម `composer test` Script និង Test មួយក្នុង Project។
3. កំណត់ PHP Matrix តាម Version ដែល Project គាំទ្រ។
4. បន្ថែម Extension ដែល Project ត្រូវការទៅ Setup Step។
5. បើក Branch Protection ឱ្យតម្រូវ CI ជោគជ័យមុន Merge។
6. ពិនិត្យថា Workflow មិនបញ្ជូន Secret ទៅ Pull Request ពី Fork។

## ១០. ចម្លើយណែនាំខ្លី

- Workflow ដាក់នៅ `.github/workflows/*.yml`។
- ប្រើ `composer validate --strict`, `composer install` និង Test Command។
- Commit `composer.lock` ហើយប្រើ PHP Version ដែលស្របនឹង Project។
- កំណត់ Permission តូចបំផុត និងកុំបង្ហាញ Secret ក្នុង Log។
- Required Status Check/Branch Protection ធ្វើឱ្យ CI ជាផ្នែកនៃ Merge Process។

## ឯកសារយោង

- [GitHub Actions Starter Workflow: PHP with Composer](https://github.com/actions/starter-workflows/blob/main/ci/php.yml)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [shivammathur/setup-php Action](https://github.com/shivammathur/setup-php)
- [Composer CLI Documentation](https://getcomposer.org/doc/03-cli.md)
- [មេរៀនទី ៣៨៖ Unit Testing ដោយ PHPUnit](lesson-38-phpunit-unit-testing.md)
- [មេរៀនទី ៤៦៖ Deploy PHP Application ទៅ Production](lesson-46-php-production-deployment.md)
