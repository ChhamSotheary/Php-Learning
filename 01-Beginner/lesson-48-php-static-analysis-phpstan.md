# មេរៀនទី ៤៨៖ Static Analysis ក្នុង PHP ដោយ PHPStan

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Static Analysis និងភាពខុសគ្នាពី Unit Test។
- ដំឡើង PHPStan ជា Development Dependency។
- កំណត់ Analysis Level និង Paths ក្នុង Configuration។
- បន្ថែម Type Hints/PHPDoc ដើម្បីឱ្យកំហុស Type ងាយរកឃើញ។
- ដំណើរការ Static Analysis នៅ Local និង CI។

## ១. Static Analysis ជាអ្វី?

Static Analysis ពិនិត្យ Source Code ដោយមិនចាំបាច់ Run Application ក្នុងគ្រប់ផ្លូវ។ វាអាចរកឃើញ Variable ដែលមិនបានកំណត់, Method មិនមាន, Argument Type ខុស និង Return Type មិនត្រូវ។

- **Static Analysis**៖ វិភាគ Code និង Type ដោយមិន Run Behavior ទាំងអស់។
- **Unit Test**៖ Run Unit ជាក់លាក់ ហើយប្រៀបធៀបលទ្ធផលរំពឹង។
- **Integration Test**៖ Run Component ជាច្រើនរួមគ្នា ដូចជា App ជាមួយ Database។

ឧបករណ៍ទាំងនេះបំពេញគ្នា; PHPStan មិនជំនួស Test ឬ Review ទេ។

## ២. ដំឡើង PHPStan

ដំឡើងជា Composer Development Dependency៖

```bash
composer require --dev phpstan/phpstan
```

ដំណើរការដំបូងដោយកំណត់ថត Source និង Tests៖

```bash
vendor/bin/phpstan analyse src tests --level 5
```

កែ Path ឱ្យសមនឹង Project។ PHPStan មាន Levels ពី 0 ដល់ 10; Level ខ្ពស់ពិនិត្យច្បាប់បន្ថែម។ ចាប់ផ្តើមពី Level ដែលអាចដោះស្រាយបាន ហើយដំឡើង Level ជាបន្តបន្ទាប់។

## ៣. បង្កើត Configuration

បង្កើត `phpstan.neon` នៅ Root៖

```neon
parameters:
    level: 5
    paths:
        - src
        - tests
    tmpDir: var/phpstan
```

បន្ថែម Script ក្នុង `composer.json`៖

```json
{
  "scripts": {
    "analyse": "phpstan analyse"
  }
}
```

បន្ទាប់មកអាចដំណើរការ៖

```bash
composer analyse
```

បើ Configuration មាន `level` រួច កុំបន្ថែម Level ផ្ទុយគ្នានៅ Command ដោយមិនចាំបាច់។ កុំដាក់ Cache/Temporary Output នៅថត Public; បន្ថែមវាទៅ Git Ignore ប្រសិនបើមិនចង់ Commit។

## ៤. បន្ថែម Type Information

ឧទាហរណ៍ Function ដែល Type ច្បាស់៖

```php
<?php
declare(strict_types=1);

/**
 * @param list<int> $pricesInCents
 */
function calculateTotal(array $pricesInCents): int
{
    return array_sum($pricesInCents);
}
```

Type Hint បង្ហាញតម្លៃដែល Function ទទួល/ត្រឡប់។ PHPDoc បន្ថែមព័ត៌មានដែល Native Type មិនទាន់បញ្ជាក់ ដូចជា Shape/Generic របស់ Array។ PHPStan អាចប្រើព័ត៌មានទាំងនេះដើម្បីពិនិត្យ Call Site ផងដែរ។

ពិនិត្យ Input ពី Request មុនបញ្ជូនទៅ Function ដែលទាមទារ Type ជាក់លាក់។ កុំបន្ថែម Type Annotation ដែលមិនពិតគ្រាន់តែឱ្យ Analysis ជោគជ័យ; កែ Logic ឱ្យត្រឹមត្រូវ ឬបង្ហាញ Type ពិត។

## ៥. ដោះស្រាយ Error ពី PHPStan

ពេល PHPStan រាយការណ៍៖

1. អាន File/Line និងប្រភេទ Error។
2. ពិនិត្យថា Bug ពិតឬអត់; កុំបិទ Check មុនយល់បញ្ហា។
3. បន្ថែម Validation/Type Guard ឬកែ Return Type។
4. បើអាស្រ័យលើ Library សូមពិនិត្យ PHPDoc/Stub និង Version របស់ Library។
5. រត់ Analysis ម្តងទៀត ហើយបន្ថែម Test ប្រសិនបើ Behavior ត្រូវបានកែ។

ឧទាហរណ៍ `$_GET['id']` អាចជា String ឬ Array។ កុំប្រកាសថា ID ជា Integer ដោយគ្មាន Validation៖

```php
$idInput = $_GET['id'] ?? null;

if (!is_string($idInput)) {
    throw new InvalidArgumentException('ID must be a string.');
}

$id = filter_var($idInput, FILTER_VALIDATE_INT);

if ($id === false || $id < 1) {
    throw new InvalidArgumentException('ID is invalid.');
}
```

## ៦. Baseline សម្រាប់ Codebase ចាស់

បើ Project ចាស់មាន Error ច្រើន អាចបង្កើត Baseline ដើម្បីរាប់បញ្ចូលបញ្ហាចាស់ ហើយអនុញ្ញាតឱ្យ Error ថ្មីបរាជ័យ CI៖

```bash
vendor/bin/phpstan analyse --generate-baseline
```

Baseline គួរត្រូវបាន Commit និងពិនិត្យដូច Code ផ្សេង។ កុំបង្កើតឡើងវិញរាល់ពេលដោយមិនពិនិត្យ ព្រោះវាអាចលាក់កំហុសថ្មី។ កាត់បន្ថយ Baseline ជាបន្តបន្ទាប់ និងបង្កើន Level ពេល Code មានគុណភាពប្រសើរឡើង។

## ៧. ដាក់ក្នុង CI

បន្ទាប់ពីបន្ថែម Composer Script អាចដាក់ Step ក្នុង GitHub Actions Workflow៖

```yaml
- name: Run static analysis
  run: composer analyse
```

ដាក់ Step នេះក្រោយ Composer Install និងមុន Merge។ កំណត់ Configuration និង PHP Version ឱ្យស្របជាមួយ CI Matrix របស់ Project។

## ៨. កំហុសដែលជួបញឹកញាប់

- បើក Level ខ្ពស់ភ្លាមៗលើ Codebase ចាស់ ហើយមិនមានផែនការដោះស្រាយ។
- បិទ Error ដោយ Ignore Pattern ទូលំទូលាយ។
- បន្ថែម PHPDoc ដែលមិនត្រូវនឹងទិន្នន័យពិត។
- គិតថា Analysis ជោគជ័យមានន័យថា App គ្មាន Runtime Bug។
- បង្កើត Baseline ជំនួសការកែបញ្ហា ហើយមិនកាត់បន្ថយវា។
- មិនដាក់ Static Analysis ក្នុង CI ដូច្នេះ Error ថ្មីចូល Main Branch។

## ៩. លំហាត់

1. ដំឡើង PHPStan និងកំណត់ `src/` និង `tests/`។
2. ដំណើរការ Level 0–5 ហើយកែ Error មួយចំនួន។
3. បន្ថែម Type Hint និង PHPDoc ទៅ Function ដែលទទួល Array។
4. បន្ថែម `composer analyse` Script។
5. បញ្ចូល Static Analysis Step ទៅ GitHub Actions Workflow។

## ១០. ចម្លើយណែនាំខ្លី

- PHPStan ពិនិត្យ Type និង Code Paths ដោយមិន Run Application ទាំងមូល។
- ដំឡើងជា `--dev` Dependency និងកំណត់ Level/Paths ក្នុង `phpstan.neon`។
- ដោះស្រាយបញ្ហាពិតជាមុន; កុំលាក់ដោយ Ignore/Baseline ទូលំទូលាយ។
- Baseline អាចជួយចាប់ផ្តើម Codebase ចាស់ ប៉ុន្តែត្រូវកាត់បន្ថយ និងពិនិត្យ។
- ដាក់ `composer analyse` ក្នុង CI ជាមួយ Unit/Integration Tests។

## ឯកសារយោង

- [PHPStan: Rule Levels](https://phpstan.org/user-guide/rule-levels)
- [PHPStan: Getting Started](https://phpstan.org/user-guide/getting-started)
- [PHPStan: Baseline](https://phpstan.org/user-guide/baseline)
- [មេរៀនទី ៤៧៖ CI/CD ដោយ GitHub Actions](lesson-47-php-github-actions-ci.md)
