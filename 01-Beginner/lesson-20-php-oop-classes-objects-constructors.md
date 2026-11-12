
# មេរៀនទី ២០៖ OOP ក្នុង PHP — Classes, Objects និង Constructor

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ភាពខុសគ្នារវាង Class និង Object។
- កំណត់ Properties និង Methods ក្នុង Class។
- ប្រើ $this ដើម្បីយោងទៅ Object បច្ចុប្បន្ន។
- ប្រើ Constructor និង Constructor Property Promotion។
- ប្រើ public, protected និង private។
- ប្រើ Encapsulation ដើម្បីការពារ State របស់ Object។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ។ Constructor Property Promotion មានតាំងពី PHP 8.0។

---

## ១. OOP ជាអ្វី?

OOP (Object-Oriented Programming) ជារបៀបរៀបចំកម្មវិធីជុំវិញ Object ដែលមានទាំងទិន្នន័យ និងសកម្មភាព។

ឧទាហរណ៍ Object គណនីធនាគារ អាចមានទិន្នន័យដូចជា ឈ្មោះម្ចាស់ និងសមតុល្យ ហើយមានសកម្មភាពដូចជា ដាក់ប្រាក់ និងដកប្រាក់។ OOP ជួយដាក់ State និងច្បាប់របស់វានៅកន្លែងតែមួយ។

## ២. Class និង Object

Class ជាគំរូ។ Object ជា Instance ដែលបង្កើតពីគំរូនោះ។

```php
<?php
class Student
{
    public string $name;
    public string $email;

    public function introduce(): string
    {
        return "ខ្ញុំឈ្មោះ {$this->name}";
    }
}

$student = new Student();
$student->name = 'ដារ៉ា';
$student->email = 'dara@example.com';

echo $student->introduce();
```

- class Student ប្រកាស Class។
- new Student() បង្កើត Object។
- $student->name ចូលទៅ Property របស់ Object។
- $this សំដៅទៅ Object បច្ចុប្បន្ននៅក្នុង Method។

## ៣. Properties និង Methods

Property រក្សាទុកទិន្នន័យ។ Method កំណត់សកម្មភាពរបស់ Object។

```php
<?php
class Product
{
    public string $name = 'មិនទាន់ដាក់ឈ្មោះ';
    public int $priceInCents = 0;

    public function formattedPrice(): string
    {
        return number_format($this->priceInCents / 100, 2);
    }
}

$product = new Product();
$product->name = 'សៀវភៅ PHP';
$product->priceInCents = 1250;

echo $product->name . ' — $' . $product->formattedPrice();
```

ឧទាហរណ៍នេះរក្សាតម្លៃជាសេន (Integer) ដើម្បីកុំឱ្យបញ្ហា Floating Point ប៉ះពាល់ដល់ការគណនាប្រាក់។

## ៤. Constructor

Constructor ជា Method ពិសេសឈ្មោះ __construct() ដែល PHP ហៅពេលបង្កើត Object។ វាសមស្របសម្រាប់កំណត់តម្លៃដំបូង និងធានាថា Object មានទិន្នន័យចាំបាច់។

```php
<?php
class Student
{
    public string $name;
    public string $email;

    public function __construct(string $name, string $email)
    {
        $this->name = $name;
        $this->email = $email;
    }
}

$student = new Student('ដារ៉ា', 'dara@example.com');
echo $student->name;
```

Constructor អាចមាន Default Value៖

```php
<?php
class Course
{
    public string $title;
    public int $capacity;

    public function __construct(string $title, int $capacity = 20)
    {
        $this->title = $title;
        $this->capacity = $capacity;
    }
}

$phpCourse = new Course('PHP Basics');
$mysqlCourse = new Course('MySQL Basics', 15);
```

## ៥. Typed Properties និង Type Declarations

Typed Property បញ្ជាក់ប្រភេទទិន្នន័យដែល Property ត្រូវផ្ទុក៖

```php
public string $name;
public int $age;
public bool $active;
```

Method អាចកំណត់ប្រភេទ Argument និង Return Value៖

```php
public function setAge(int $age): void
{
    $this->age = $age;
}
```

void មានន័យថា Method មិនត្រឡប់តម្លៃទេ។ Typed Property ដែលមិនទាន់បានកំណត់តម្លៃ មិនអាចអានបានឡើយ។ ត្រូវផ្ដល់ Default Value ឬកំណត់តម្លៃក្នុង Constructor។

## ៦. Visibility និង Encapsulation

Visibility កំណត់ថាកូដណាអាចប្រើ Property ឬ Method៖

- public៖ អាចចូលប្រើពីគ្រប់ទីកន្លែង។
- protected៖ អាចប្រើក្នុង Class និង Class កូន។
- private៖ អាចប្រើតែក្នុង Class ដែលបានប្រកាសវា។

Encapsulation គឺការលាក់ State ខាងក្នុង ហើយអនុញ្ញាតឱ្យកែប្រែតាម Method ដែលអាច Validate ច្បាប់បាន។

```php
<?php
class Student
{
    private string $name;
    private string $email;

    public function __construct(string $name, string $email)
    {
        $this->changeName($name);
        $this->changeEmail($email);
    }

    public function name(): string
    {
        return $this->name;
    }

    public function changeName(string $name): void
    {
        $name = trim($name);

        if ($name === '' || mb_strlen($name) > 100) {
            throw new InvalidArgumentException('ឈ្មោះមិនត្រឹមត្រូវ។');
        }

        $this->name = $name;
    }

    public function changeEmail(string $email): void
    {
        $email = trim($email);

        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Email មិនត្រឹមត្រូវ។');
        }

        $this->email = $email;
    }
}

$student = new Student('ដារ៉ា', 'dara@example.com');
$student->changeName('សុភា');
echo $student->name();
```

កូដក្រៅ Class មិនអាចកំណត់ឈ្មោះទៅ String ទទេដោយផ្ទាល់បានទេ ព្រោះ Property ជា private។ វាត្រូវប្រើ Method ដែលពិនិត្យច្បាប់ជាមុន។

## ៧. Constructor Property Promotion

ចាប់ពី PHP 8.0 អាចប្រកាស Argument ជា Property ក្នុង Constructor ដោយផ្ទាល់៖

```php
<?php
class Course
{
    public function __construct(
        private string $title,
        private int $capacity
    ) {
    }

    public function title(): string
    {
        return $this->title;
    }

    public function capacity(): int
    {
        return $this->capacity;
    }
}

$course = new Course('PHP Basics', 25);
echo $course->title();
```

ទម្រង់នេះជំនួសការប្រកាស Property ដាច់ដោយឡែក និងការកំណត់តម្លៃក្នុង Constructor។ វាសមស្របនៅពេល Constructor គ្រាន់តែរក្សាទុកតម្លៃ។ បើត្រូវ Validate តម្លៃ សូមបន្ថែម Logic ក្នុង Constructor ឬហៅ Method ដែលមានច្បាប់ច្បាស់។

## ៨. Object State

Object នីមួយៗមាន State ផ្ទាល់ខ្លួន៖

```php
<?php
class Counter
{
    private int $value = 0;

    public function increment(): void
    {
        $this->value++;
    }

    public function value(): int
    {
        return $this->value;
    }
}

$first = new Counter();
$second = new Counter();

$first->increment();

echo $first->value();  // 1
echo $second->value(); // 0
```

ការកែប្រែ $first មិនប៉ះពាល់ $second ទេ ព្រោះវាជា Object ពីរផ្សេងគ្នា។

## ៩. Mini Project៖ គណនីសាកល្បង

បង្កើតឯកសារ BankAccount.php៖

```php
<?php
declare(strict_types=1);

class BankAccount
{
    private int $balanceInCents = 0;

    public function __construct(
        private string $owner,
        int $initialBalanceInCents = 0
    ) {
        $owner = trim($owner);

        if ($owner === '') {
            throw new InvalidArgumentException('ត្រូវមានឈ្មោះម្ចាស់គណនី។');
        }

        if ($initialBalanceInCents < 0) {
            throw new InvalidArgumentException('សមតុល្យដំបូងមិនអាចអវិជ្ជមានបានទេ។');
        }

        $this->owner = $owner;
        $this->balanceInCents = $initialBalanceInCents;
    }

    public function owner(): string
    {
        return $this->owner;
    }

    public function deposit(int $amountInCents): void
    {
        if ($amountInCents < 1) {
            throw new InvalidArgumentException('ចំនួនដាក់ត្រូវធំជាងសូន្យ។');
        }

        $this->balanceInCents += $amountInCents;
    }

    public function withdraw(int $amountInCents): void
    {
        if ($amountInCents < 1) {
            throw new InvalidArgumentException('ចំនួនដកត្រូវធំជាងសូន្យ។');
        }

        if ($amountInCents > $this->balanceInCents) {
            throw new RuntimeException('សមតុល្យមិនគ្រប់គ្រាន់។');
        }

        $this->balanceInCents -= $amountInCents;
    }

    public function formattedBalance(): string
    {
        return number_format($this->balanceInCents / 100, 2);
    }
}
```

បង្កើត demo.php៖

```php
<?php
require __DIR__ . '/BankAccount.php';

try {
    $account = new BankAccount('ដារ៉ា', 5000);
    $account->deposit(2500);
    $account->withdraw(1200);

    echo htmlspecialchars($account->owner(), ENT_QUOTES, 'UTF-8');
    echo ' — សមតុល្យ៖ $' . $account->formattedBalance();
} catch (Throwable $e) {
    echo htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8');
}
```

លទ្ធផល៖ 63.00

សាកល្បងដកលើសសមតុល្យ ដាក់ប្រាក់ជាចំនួនអវិជ្ជមាន និងបង្កើតគណនីដោយគ្មានឈ្មោះ។ Object គួរបដិសេធសកម្មភាពទាំងនោះ។ គម្រោងនេះសម្រាប់រៀនប៉ុណ្ណោះ មិនរក្សាទុកទិន្នន័យ និងមិនមែនជាប្រព័ន្ធហិរញ្ញវត្ថុពិតទេ។

## ១០. Best Practices

1. ប្រើឈ្មោះ Class ជានាម និងឈ្មោះ Method ជាកិរិយាសព្ទ។
2. ចាប់ផ្ដើមពី private ហើយបើកតែ Method ដែលត្រូវប្រើខាងក្រៅ។
3. Validate ទិន្នន័យពេលបង្កើត ឬកែប្រែ Object។
4. ប្រើ Typed Properties និង Return Types។
5. កុំបង្កើត Getter និង Setter ដោយស្វ័យប្រវត្តិសម្រាប់គ្រប់ Property។
6. រក្សា Method ឱ្យមានតួនាទីច្បាស់។
7. កុំបង្កើត Class សម្រាប់គ្រប់ Function; ប្រើ Object ពេលវាជួយឱ្យ State និងច្បាប់ច្បាស់ឡើង។
8. ប្រើ __construct() ជានិច្ច ជំនួស Constructor រចនាបថចាស់ដែលដាក់ឈ្មោះដូច Class។

## ១១. កំហុសដែលជួបញឹកញាប់

- ភ្លេច new ពេលបង្កើត Object។
- ច្រឡំ -> ដែលប្រើលើ Object និង :: ដែលប្រើលើ Class Member។
- ភ្លេច $this ក្នុង Instance Method។
- អាន Typed Property មុនកំណត់តម្លៃ។
- បើក Property ទាំងអស់ជា public ហើយអនុញ្ញាតឱ្យកូដខាងក្រៅបំបែកច្បាប់របស់ Object។
- គិតថា Type Declaration ជំនួស Business Validation; int មិនធានាថាតម្លៃធំជាងសូន្យទេ។
- ប្រើ Float សម្រាប់គណនាប្រាក់ដោយមិនគិតពីភាពជាក់លាក់។

## ១២. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Book Class

បង្កើត Class Book មាន Properties ឯកជន title និង author។ Constructor ទទួលតម្លៃទាំងពីរ ហើយ Method description() ត្រឡប់ប្រយោគពណ៌នាសៀវភៅ។

### លំហាត់ទី ២៖ Validate Constructor

កែ Book ឱ្យបោះ InvalidArgumentException ប្រសិនបើ Title ឬ Author ទទេ។

### លំហាត់ទី ៣៖ Product

បង្កើត Product មានឈ្មោះ និង Price ជាសេន។ មិនអនុញ្ញាតឈ្មោះទទេ ឬតម្លៃអវិជ្ជមាន។

### លំហាត់ទី ៤៖ Counter

បង្កើត Counter ពីរ បង្កើនតែមួយ ហើយពិនិត្យថា State របស់មួយទៀតនៅដដែល។

### លំហាត់ទី ៥៖ Constructor Promotion

បង្កើត Course ដោយប្រើ Constructor Property Promotion សម្រាប់ Title និង Capacity ហើយបន្ថែម Method អាន Title។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ Class, Object, Property, Method, Constructor និង Encapsulation ដោយឧទាហរណ៍ផ្ទាល់ខ្លួន។

## ១៣. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១ និង ២

```php
<?php
class Book
{
    public function __construct(
        private string $title,
        private string $author
    ) {
        if (trim($this->title) === '' || trim($this->author) === '') {
            throw new InvalidArgumentException('ត្រូវបញ្ចូល Title និង Author។');
        }
    }

    public function description(): string
    {
        return "{$this->title} និពន្ធដោយ {$this->author}";
    }
}

$book = new Book('រៀន PHP', 'ដារ៉ា');
echo $book->description();
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
class Product
{
    public function __construct(
        private string $name,
        private int $priceInCents
    ) {
        if (trim($this->name) === '' || $this->priceInCents < 0) {
            throw new InvalidArgumentException('ព័ត៌មាន Product មិនត្រឹមត្រូវ។');
        }
    }

    public function price(): string
    {
        return number_format($this->priceInCents / 100, 2);
    }
}
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$first = new Counter();
$second = new Counter();

$first->increment();

echo $first->value();  // 1
echo $second->value(); // 0
```

### ចម្លើយលំហាត់ទី ៥

```php
<?php
class Course
{
    public function __construct(
        private string $title,
        private int $capacity
    ) {
    }

    public function title(): string
    {
        return $this->title;
    }
}
```

### ចម្លើយលំហាត់ទី ៦

Class ជាគំរូ; Object ជា Instance ពីគំរូ។ Property រក្សាទិន្នន័យ; Method កំណត់សកម្មភាព។ Constructor រៀបចំទិន្នន័យដំបូងពេលបង្កើត Object។ Encapsulation លាក់ State ហើយអនុញ្ញាតឱ្យកែប្រែតាម Method ដែលអនុវត្តច្បាប់។

---

## សេចក្ដីសង្ខេប

- Class ជាគំរូ; Object ជា Instance ដែលមាន State ផ្ទាល់។
- Property រក្សាទិន្នន័យ; Method កំណត់សកម្មភាព។
- Constructor __construct() រៀបចំ Object ពេលបង្កើត។
- Typed Properties និង Type Declarations ធ្វើឱ្យប្រភេទទិន្នន័យច្បាស់។
- Visibility និង Encapsulation ការពារច្បាប់របស់ Object។
- Constructor Property Promotion មានតាំងពី PHP 8.0។
- OOP ជួយរៀបចំកម្មវិធី ប៉ុន្តែមិនចាំបាច់បង្កើត Class សម្រាប់គ្រប់ Function ទេ។

## ឯកសារយោងផ្លូវការ

- [PHP Classes and Objects: The Basics](https://www.php.net/manual/en/language.oop5.basic.php)
- [PHP Constructors and Destructors](https://www.php.net/manual/en/language.oop5.decon.php)
- [PHP Properties](https://www.php.net/manual/en/language.oop5.properties.php)

## មេរៀនបន្ទាប់

អាចបន្ត OOP ដោយរៀន Inheritance, Method Overriding, Abstract Classes និង Interfaces។
