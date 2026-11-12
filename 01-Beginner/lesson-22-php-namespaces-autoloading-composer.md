
# មេរៀនទី ២២៖ Namespaces, Autoloading និង Composer

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់មូលហេតុដែលកម្មវិធីត្រូវប្រើ Namespace។
- ប្រកាស និងប្រើ Class ក្នុង Namespace ផ្សេងៗ។
- ប្រើ use ដើម្បី Import Class ឱ្យអានងាយ។
- យល់ពី Autoloading និងមូលហេតុដែលវាជំនួស require ច្រើនៗ។
- កំណត់ PSR-4 Autoloading ក្នុង composer.json។
- ប្រើ Composer Autoloader ក្នុងកម្មវិធី។
- រៀបចំ Folder និង Class Files តាមឈ្មោះ Namespace។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ និង Composer 2.x។ ពិនិត្យការដំឡើងដោយប្រើ php -v និង composer --version។

---

## ១. ហេតុអ្វីត្រូវប្រើ Namespace?

កម្មវិធីធំអាចមាន Class ច្រើន ហើយឈ្មោះសាមញ្ញអាចប៉ះទង្គិចគ្នា។ ឧទាហរណ៍ កម្មវិធីអាចមាន Class User សម្រាប់អ្នកប្រើ និង Class User មួយទៀតពី Library។

Namespace ជួយដាក់ឈ្មោះ Class, Interface, Function និង Constant ក្នុងក្រុមដែលមានឈ្មោះពេញលេញខុសៗគ្នា។

ឈ្មោះពេញលេញរបស់ Class រួមមាន Namespace និង Class Name៖

```text
LearningPhp\Domain\Student
LearningPhp\Http\StudentController
```

ទោះមានឈ្មោះខ្លី Student ដូចគ្នាក៏ដោយ Namespace ខុសគ្នាធ្វើឱ្យវាមិនប៉ះទង្គិចគ្នា។

## ២. ប្រកាស Namespace

Namespace ត្រូវដាក់នៅដើម PHP File ក្រោយ <?php និង declare បើមាន៖

```php
<?php
declare(strict_types=1);

namespace LearningPhp\Domain;

class Student
{
    public function __construct(
        private string $name
    ) {
    }

    public function name(): string
    {
        return $this->name;
    }
}
```

បើ File មាន Namespace នោះ Class និង Function ក្នុង File នោះស្ថិតក្រោម Namespace ដូចគ្នា លុះត្រាតែប្រកាសផ្សេង។

## ៣. ប្រើ Fully Qualified Name

អាចប្រើ Class ដោយសរសេរឈ្មោះពេញ៖

```php
<?php
$student = new \LearningPhp\Domain\Student('ដារ៉ា');

echo $student->name();
```

សញ្ញា Backslash នៅដើមមានន័យថាចាប់ផ្ដើមស្វែងរកពី Global Namespace។ វាជួយកុំឱ្យ PHP ស្វែងរកឈ្មោះនៅក្នុង Namespace បច្ចុប្បន្នជាមុន។

## ៤. ប្រើ use ដើម្បី Import Class

បើប្រើឈ្មោះពេញជាញឹកញាប់ អាច Import Class ដោយ use៖

```php
<?php
declare(strict_types=1);

namespace LearningPhp\Application;

use LearningPhp\Domain\Student;

$student = new Student('ដារ៉ា');

echo $student->name();
```

use គ្រាន់តែបង្កើត Alias សម្រាប់ឈ្មោះក្នុង File បច្ចុប្បន្ន មិនមែនបញ្ចូល File ឬបង្កើត Object ដោយស្វ័យប្រវត្តិទេ។ Autoloader ជាអ្នករក និងផ្ទុក Class File នៅពេល Class ត្រូវបានប្រើ។

អាចកំណត់ Alias ផ្សេង ដើម្បីជៀសវាងឈ្មោះប៉ះទង្គិច៖

```php
<?php
use Vendor\Package\User as PackageUser;
use LearningPhp\Domain\User as AppUser;

$first = new PackageUser();
$second = new AppUser();
```

## ៥. Namespace សម្រាប់ Function និង Constant

Namespace អាចប្រើជាមួយ Function និង Constant ផងដែរ៖

```php
<?php
namespace LearningPhp\Support;

const APP_VERSION = '1.0.0';

function normalizeName(string $name): string
{
    return trim($name);
}
```

Import Function និង Constant ដោយបញ្ជាក់ប្រភេទ៖

```php
<?php
namespace LearningPhp\Application;

use function LearningPhp\Support\normalizeName;
use const LearningPhp\Support\APP_VERSION;

echo normalizeName(' ដារ៉ា ');
echo APP_VERSION;
```

## ៦. Autoloading ជាអ្វី?

បើមិនប្រើ Autoloading អ្នកអាចត្រូវសរសេរ require ឬ include សម្រាប់ Class File នីមួយៗ៖

```php
require __DIR__ . '/src/Domain/Student.php';
require __DIR__ . '/src/Repository/StudentRepository.php';
require __DIR__ . '/src/Service/StudentService.php';
```

ពេល Project មាន Class ច្រើន វាពិបាកថែទាំ និងងាយភ្លេច File មួយ។ Autoloading ជាវិធីឱ្យ PHP ផ្ទុក Class File ដោយស្វ័យប្រវត្តិ នៅពេលកូដប្រើ Class នោះ។

Composer ផ្ដល់ Autoloader ដែលគាំទ្រ PSR-4។ PSR-4 ផ្គូផ្គង Namespace Prefix ទៅ Folder ហើយផ្គូផ្គងផ្នែកនៅសល់នៃ Namespace ទៅ Subfolder និងឈ្មោះ File។

## ៧. កំណត់ PSR-4 ក្នុង Composer

បង្កើត Folder របស់ Project៖

```text
student-app/
├── composer.json
├── public/
│   └── index.php
└── src/
    ├── Domain/
    │   └── Student.php
    └── Service/
        └── StudentDirectory.php
```

បង្កើត composer.json នៅ Root៖

```json
{
  "name": "learning-php/student-app",
  "description": "ឧទាហរណ៍ Namespaces និង Composer Autoloading",
  "type": "project",
  "autoload": {
    "psr-4": {
      "LearningPhp\\": "src/"
    }
  },
  "require": {
    "php": ">=8.0"
  }
}
```

ការផ្គូផ្គងមានន័យថា៖

| Namespace/Class | File Path |
|---|---|
| LearningPhp\Domain\Student | src/Domain/Student.php |
| LearningPhp\Service\StudentDirectory | src/Service/StudentDirectory.php |

ត្រូវរក្សាអក្សរធំតូចរបស់ Namespace, Folder និង File ឱ្យដូចគ្នា ជាពិសេសនៅ Linux ដែលឈ្មោះ Path មាន Case-sensitive។

ក្រោយបង្កើត ឬកែ Autoload Configuration ដំណើរការ៖

```bash
composer dump-autoload
```

ពាក្យបញ្ជានេះបង្កើត ឬធ្វើបច្ចុប្បន្នភាព Composer Autoloader នៅក្នុង vendor/។

## ៨. បង្កើត Class Files

ឯកសារ src/Domain/Student.php៖

```php
<?php
declare(strict_types=1);

namespace LearningPhp\Domain;

final class Student
{
    public function __construct(
        private int $id,
        private string $name,
        private string $email
    ) {
        if ($this->id < 1) {
            throw new \InvalidArgumentException('ID មិនត្រឹមត្រូវ។');
        }

        if (trim($this->name) === '') {
            throw new \InvalidArgumentException('ឈ្មោះមិនអាចទទេបានទេ។');
        }

        if (!filter_var($this->email, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException('Email មិនត្រឹមត្រូវ។');
        }
    }

    public function id(): int
    {
        return $this->id;
    }

    public function name(): string
    {
        return $this->name;
    }

    public function email(): string
    {
        return $this->email;
    }
}
```

ឯកសារ src/Service/StudentDirectory.php៖

```php
<?php
declare(strict_types=1);

namespace LearningPhp\Service;

use LearningPhp\Domain\Student;

final class StudentDirectory
{
    /** @var list<Student> */
    private array $students = [];

    public function add(Student $student): void
    {
        $this->students[] = $student;
    }

    /** @return list<Student> */
    public function all(): array
    {
        return $this->students;
    }
}
```

StudentDirectory ប្រើ use ដើម្បី Import Class Student។ Type Hint Student បញ្ជាក់ថា Method add() ទទួលតែ Student Object ប៉ុណ្ណោះ។

## ៩. ប្រើ Composer Autoloader

ឯកសារ public/index.php៖

```php
<?php
declare(strict_types=1);

use LearningPhp\Domain\Student;
use LearningPhp\Service\StudentDirectory;

require dirname(__DIR__) . '/vendor/autoload.php';

$directory = new StudentDirectory();

$directory->add(
    new Student(1, 'ដារ៉ា', 'dara@example.com')
);

foreach ($directory->all() as $student) {
    echo htmlspecialchars($student->name(), ENT_QUOTES, 'UTF-8');
    echo ' — ';
    echo htmlspecialchars($student->email(), ENT_QUOTES, 'UTF-8');
    echo '<br>';
}
```

require vendor/autoload.php ត្រូវធ្វើម្តងនៅ Entry Point របស់កម្មវិធី។ បន្ទាប់មក Composer នឹងផ្ទុក Class តាម PSR-4 ដោយស្វ័យប្រវត្តិ។

ចំណាំអំពី PHP Syntax៖ Namespace Import ដូចជា use ត្រូវដាក់នៅកម្រិត File មិនមែនក្នុង Block ឬ Function។ ក្នុងឧទាហរណ៍ពិត ដាក់ use Statements ភ្លាមៗក្រោយ namespace declaration បើ File មាន Namespace។

## ១០. Composer Commands សំខាន់ៗ

- composer init បង្កើត composer.json តាមសំណួរណែនាំ។
- composer install ដំឡើង Dependencies ដែលបានកំណត់។
- composer require vendor/package បន្ថែម Package មួយ។
- composer dump-autoload បង្កើត Autoloader ឡើងវិញក្រោយកែ Autoload Mapping។
- composer show បង្ហាញ Packages ដែលបានដំឡើង។

សម្រាប់ Project ដែលមាន composer.json រួចហើយ ប្រើ composer install ដើម្បីដំឡើង Dependencies។ កុំ Commit vendor/ ជាទូទៅ; Commit composer.json និង composer.lock ដើម្បីឱ្យ Environment ផ្សេងដំឡើង Version ដូចគ្នាបាន។

## ១១. Namespace Resolution និងឈ្មោះ PHP ដើម

ក្នុង Namespace មួយ ឈ្មោះ Class មិនមាន Backslash ដើមនឹងត្រូវស្វែងរកនៅ Namespace នោះជាមុន។ ឈ្មោះ Class ដើមរបស់ PHP ដូចជា DateTime អាចយោងជាមួយ Backslash ឬ Import ដោយ use៖

```php
<?php
namespace LearningPhp\Service;

use DateTimeImmutable;
use InvalidArgumentException;

$date = new DateTimeImmutable('now');

throw new InvalidArgumentException('Input មិនត្រឹមត្រូវ។');
```

ប្រើ use ឱ្យច្បាស់ ឬ Fully Qualified Name ដើម្បីជៀសវាងឈ្មោះ Class ប៉ះទង្គិច។

## ១២. Mini Project៖ Student Directory ដោយ Autoloading

1. បង្កើត Folder និង Files តាមផ្នែកទី ៧។
2. បង្កើត composer.json ហើយដំណើរការ composer dump-autoload។
3. បន្ថែម Class Student និង StudentDirectory តាមផ្នែកទី ៨។
4. ដាក់ Entry Point នៅ public/index.php។
5. ដំណើរការ PHP Built-in Server ពី Root របស់ Project៖

```bash
php -S localhost:8000 -t public
```

6. បើក http://localhost:8000។
7. បង្កើត Student Object មួយទៀត ហើយបន្ថែមទៅ Directory។
8. ប្ដូរឈ្មោះ Folder មួយជាបណ្ដោះអាសន្ន រួចសាកល្បងថា Autoloader រកឃើញបញ្ហាដែរឬទេ; បន្ទាប់មកដាក់ឈ្មោះត្រឡប់ដូចដើម។

Composer ត្រូវដំឡើងនៅលើម៉ាស៊ីន ហើយត្រូវមាន PHP ក្នុង PATH។ បើ Command composer មិនដំណើរការ សូមពិនិត្យការដំឡើង Composer និង PATH របស់ Terminal។

## ១៣. កំហុសដែលជួបញឹកញាប់

- ភ្លេច Backslash ពេលសរសេរ Namespace ឬ Fully Qualified Name។
- ដាក់ Namespace Declaration ក្រោយកូដដែលមិនអនុញ្ញាតឱ្យដាក់មុនវា។
- ឈ្មោះ File ឬ Folder មិនត្រូវនឹង Namespace និង Class Name តាម PSR-4។
- ភ្លេច include vendor/autoload.php។
- កែ composer.json តែភ្លេចដំណើរការ composer dump-autoload។
- សរសេរ Namespace Prefix មិនមាន Backslash បិទ ឬ Mapping ទៅ Folder ខុស។
- គិតថា use ផ្ទុក File; use គ្រាន់តែ Import ឈ្មោះ។
- Commit Database Password ឬ Secret ក្នុង composer.json ឬ Source Code។
- ច្រឡំ require ជាមួយ require-dev; Dependencies សម្រាប់ Runtime ត្រូវដាក់ក្នុង require។

## ១៤. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Namespaced Class

បង្កើត Class Book នៅ Namespace LearningPhp\Domain ហើយបង្កើត Object ពី File ផ្សេងដោយ Fully Qualified Name។

### លំហាត់ទី ២៖ Import Class

កែឧទាហរណ៍មុនឱ្យប្រើ use ដើម្បី Import Book។

### លំហាត់ទី ៣៖ PSR-4 Mapping

បន្ថែម Namespace Prefix LearningPhp\Repository\ ដែល Map ទៅ src/Repository/។ បង្កើត Class BookRepository នៅ Path ដែលត្រឹមត្រូវ។

### លំហាត់ទី ៤៖ Autoloading

កែ composer.json ដោយបន្ថែម Class File ថ្មី រួចដំណើរការ composer dump-autoload និងប្រើ Class ដោយមិន require File របស់វាដោយផ្ទាល់។

### លំហាត់ទី ៥៖ Alias

បង្កើត Class User ពី Namespace ផ្សេងពីរដែលមានឈ្មោះដូចគ្នា។ Import មួយដោយ Alias ហើយប្រើទាំងពីរ។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Namespace, use, Autoloading និង require vendor/autoload.php។

## ១៥. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- Namespace ដាក់ឈ្មោះ Class និងកូដជាក្រុម ដើម្បីកាត់បន្ថយឈ្មោះប៉ះទង្គិច។
- use បង្កើតឈ្មោះខ្លី ឬ Alias សម្រាប់ Class/Function/Constant ក្នុង File បច្ចុប្បន្ន។
- Autoloading រក និងផ្ទុក Class File នៅពេលកូដប្រើ Class។
- vendor/autoload.php ចុះឈ្មោះ Composer Autoloader និងការកំណត់ PSR-4។
- PSR-4 Mapping ភ្ជាប់ Namespace Prefix ទៅ Folder ហើយ Namespace ដែលនៅសល់កំណត់ Subfolder និង File Name។

ឧទាហរណ៍ Mapping៖

```json
{
  "autoload": {
    "psr-4": {
      "LearningPhp\\": "src/"
    }
  }
}
```

Class LearningPhp\Repository\BookRepository ត្រូវដាក់ក្នុង src/Repository/BookRepository.php។

---

## សេចក្ដីសង្ខេប

- Namespace កាត់បន្ថយការប៉ះទង្គិចឈ្មោះ និងរៀបចំ Class តាមផ្នែក។
- use Import Class, Function ឬ Constant; វាមិនផ្ទុក File ដោយខ្លួនឯងទេ។
- Autoloading ផ្ទុក Class File តាមតម្រូវការ។
- Composer គាំទ្រ PSR-4 និងបង្កើត vendor/autoload.php។
- កំណត់ Namespace Prefix ឱ្យ Map ទៅ Folder ហើយរក្សាឈ្មោះ File/Folder ឱ្យត្រូវគ្នា។
- ប្រើ composer install សម្រាប់ Dependencies និង composer dump-autoload ក្រោយកែ Autoload Mapping។

## ឯកសារយោងផ្លូវការ

- [PHP Namespaces](https://www.php.net/manual/en/language.namespaces.php)
- [Composer Autoload Schema](https://getcomposer.org/doc/04-schema.md#autoload)
- [Composer Basic Usage](https://getcomposer.org/doc/01-basic-usage.md)

## មេរៀនបន្ទាប់

អាចបន្តដោយរៀន Composer Dependencies, Version Constraints និងការគ្រប់គ្រង composer.lock។
