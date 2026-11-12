
# មេរៀនទី ២១៖ Inheritance, Abstract Classes និង Interfaces

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ប្រើ Inheritance ដោយ extends ដើម្បីបន្ត Class ដែលមានស្រាប់។
- Override Method ដោយរក្សា Signature ឱ្យស្របគ្នា។
- ប្រើ parent:: ដើម្បីហៅ Method ឬ Constructor របស់ Parent Class។
- យល់ពី Abstract Class និង Abstract Method។
- កំណត់ Interface សម្រាប់កិច្ចសន្យារួម។
- ឱ្យ Class មួយ Implement Interface ច្រើន។
- ជ្រើសរើសប្រើ Interface ឬ Inheritance តាមតម្រូវការ។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ។ មេរៀននេះបន្តពី Classes និង Objects ក្នុងមេរៀនទី ២០។

---

## ១. Inheritance ជាអ្វី?

Inheritance អនុញ្ញាតឱ្យ Child Class បន្ត Properties និង Methods ដែលអាចទទួលបានពី Parent Class។ Child Class អាចប្រើមុខងារដែលមានស្រាប់ ហើយបន្ថែម ឬកែប្រែឥរិយាបថរបស់ខ្លួន។

ប្រើ Inheritance ពេល Class មួយជាប្រភេទពិសេសរបស់ Class មួយទៀត (is-a relationship)។ ឧទាហរណ៍ Dog ជា Animal មួយប្រភេទ។

```php
<?php
class Animal
{
    public function eat(): string
    {
        return 'កំពុងញ៉ាំអាហារ';
    }
}

class Dog extends Animal
{
    public function bark(): string
    {
        return 'Woof!';
    }
}

$dog = new Dog();

echo $dog->eat();
echo "
";
echo $dog->bark();
```

Dog ទទួលបាន Method eat() ពី Animal ហើយមាន Method bark() ផ្ទាល់ខ្លួន។

PHP Class អាច extends Class មួយបានតែមួយ ប៉ុន្តែអាច Implement Interfaces ច្រើន។

## ២. Method Overriding

Child Class អាចសរសេរ Method ឈ្មោះដូច Parent Method ដើម្បីផ្ដល់ឥរិយាបថថ្មី។ នេះហៅថា Method Overriding។

```php
<?php
class Animal
{
    public function speak(): string
    {
        return 'សត្វមួយកំពុងបន្លឺសំឡេង';
    }
}

class Dog extends Animal
{
    public function speak(): string
    {
        return 'Woof!';
    }
}

class Cat extends Animal
{
    public function speak(): string
    {
        return 'Meow!';
    }
}

$animals = [new Dog(), new Cat()];

foreach ($animals as $animal) {
    echo $animal->speak() . "
";
}
```

Method របស់ Child ត្រូវមាន Signature ដែលសមស្របជាមួយ Method របស់ Parent។ កុំប្ដូរ Parameter និង Return Type ដោយធ្វើឱ្យកូដដែលរំពឹង Parent Class មិនអាចប្រើ Child Class បាន។

## ៣. ប្រើ parent::

Child Class អាចហៅ Constructor ឬ Method របស់ Parent តាម parent::៖

```php
<?php
class User
{
    public function __construct(
        protected string $name
    ) {
    }

    public function label(): string
    {
        return 'អ្នកប្រើ៖ ' . $this->name;
    }
}

class Admin extends User
{
    public function __construct(string $name, private string $permission)
    {
        parent::__construct($name);
    }

    public function label(): string
    {
        return parent::label() . ' (Admin)';
    }

    public function permission(): string
    {
        return $this->permission;
    }
}

$admin = new Admin('ដារ៉ា', 'manage-users');
echo $admin->label();
```

បើ Child Class ប្រកាស Constructor ផ្ទាល់ វាមិនហៅ Parent Constructor ដោយស្វ័យប្រវត្តិទេ។ ត្រូវហៅ parent::__construct() ប្រសិនបើ Parent ត្រូវការកំណត់ State។

## ៤. Visibility ក្នុង Inheritance

- public Members អាចប្រើពីគ្រប់ទីកន្លែង។
- protected Members អាចប្រើក្នុង Class និង Child Class។
- private Members ប្រើបានតែក្នុង Class ដែលប្រកាសវា។

```php
<?php
class Person
{
    protected string $name;
    private string $secretCode;

    public function __construct(string $name, string $secretCode)
    {
        $this->name = $name;
        $this->secretCode = $secretCode;
    }
}

class Teacher extends Person
{
    public function greeting(): string
    {
        return 'សួស្តី ' . $this->name;
        // Child Class អាចចូលប្រើ $name ព្រោះវា protected។
        // មិនអាចចូលប្រើ $secretCode ព្រោះវា private។
    }
}
```

ជាទូទៅប្រើ private សម្រាប់ State ខាងក្នុង។ ប្រើ protected តែពេល Child Class ត្រូវការចូលប្រើវាពិតប្រាកដ។

## ៥. Abstract Class

Abstract Class ជា Class គំរូដែលមិនអាចបង្កើត Object ដោយផ្ទាល់។ វាអាចមាន៖

- Properties និង Methods ដែលមាន Implementation រួច។
- Abstract Methods ដែលកំណត់តែ Signature ដោយគ្មាន Body។

Child Class ដែល extends Abstract Class ត្រូវ Implement Abstract Methods ទាំងអស់។

```php
<?php
abstract class Report
{
    public function __construct(
        protected string $title
    ) {
    }

    public function heading(): string
    {
        return 'របាយការណ៍៖ ' . $this->title;
    }

    abstract public function render(): string;
}

class TextReport extends Report
{
    public function render(): string
    {
        return $this->heading() . "
មាតិកាអត្ថបទ";
    }
}

$report = new TextReport('លទ្ធផលសិក្សា');
echo $report->render();
```

កូដខាងក្រោមមិនអាចធ្វើបានទេ ព្រោះ Report ជា Abstract Class៖

```php
$report = new Report('លទ្ធផល'); // កំហុស៖ Abstract Class មិនអាច Instantiate បាន
```

ប្រើ Abstract Class ពេល Child Classes ចែករំលែក State ឬ Implementation មួយចំនួន ប៉ុន្តែត្រូវផ្ដល់អាកប្បកិរិយាផ្សេងគ្នាសម្រាប់ Method មួយចំនួន។

## ៦. Interface

Interface កំណត់កិច្ចសន្យាថា Class ដែល Implement វាត្រូវមាន Methods អ្វីខ្លះ។ Interface បញ្ជាក់ថា Object អាចធ្វើអ្វីបាន ប៉ុន្តែមិនកំណត់ Implementation របស់វាទេ។

```php
<?php
interface Notifier
{
    public function send(string $recipient, string $message): bool;
}

class EmailNotifier implements Notifier
{
    public function send(string $recipient, string $message): bool
    {
        // ក្នុងកម្មវិធីពិត ត្រូវភ្ជាប់ទៅ Email Service។
        echo "ផ្ញើ Email ទៅ {$recipient}: {$message}
";
        return true;
    }
}

class SmsNotifier implements Notifier
{
    public function send(string $recipient, string $message): bool
    {
        // ក្នុងកម្មវិធីពិត ត្រូវភ្ជាប់ទៅ SMS Provider។
        echo "ផ្ញើ SMS ទៅ {$recipient}: {$message}
";
        return true;
    }
}
```

Methods ក្នុង Interface ត្រូវជា public ហើយ Class ដែល Implements ត្រូវផ្ដល់ Implementation ដែលមាន Signature សមស្រប។

## ៧. Polymorphism តាម Interface

Polymorphism អនុញ្ញាតឱ្យកូដប្រើ Objects ផ្សេងៗតាមកិច្ចសន្យារួម ដោយមិនចាំបាច់ស្គាល់ Class ពិតប្រាកដរបស់វា។

```php
<?php
function notifyUser(
    Notifier $notifier,
    string $recipient,
    string $message
): bool {
    return $notifier->send($recipient, $message);
}

$email = new EmailNotifier();
$sms = new SmsNotifier();

notifyUser($email, 'user@example.com', 'គណនីរបស់អ្នករួចរាល់');
notifyUser($sms, '+85512345678', 'លេខកូដរបស់អ្នកគឺ 1234');
```

notifyUser() ទទួល Notifier Interface។ ដូច្នេះអាចប្ដូរ EmailNotifier ទៅ SmsNotifier ដោយមិនចាំបាច់សរសេរ Function ថ្មី។

Interface អាច extends Interface ផ្សេង ហើយ Class មួយអាច implements Interfaces ច្រើន៖

```php
<?php
interface Loggable
{
    public function logMessage(): string;
}

class EmailNotifier implements Notifier, Loggable
{
    public function send(string $recipient, string $message): bool
    {
        return true;
    }

    public function logMessage(): string
    {
        return 'Email Notification ត្រូវបានដំណើរការ';
    }
}
```

កុំដាក់ Constructor ក្នុង Interface ដោយគ្មានហេតុផលច្បាស់ ព្រោះវាអាចកម្រិតភាពបត់បែនរបស់ Class ដែល Implement Interface នោះ។

## ៨. Abstract Class ឬ Interface?

| តម្រូវការ | ជម្រើសសមស្រប |
|---|---|
| ចង់កំណត់កិច្ចសន្យា ដែល Class ផ្សេងៗអនុវត្ត | Interface |
| ចង់ចែករំលែក State និង Code រួម | Abstract Class |
| Class ត្រូវអាចអនុវត្តកិច្ចសន្យាច្រើន | Interface |
| ត្រូវការមេ Class មួយសម្រាប់ Child Classes | Abstract Class |
| ចង់ប្ដូរ Implementation ដោយមិនភ្ជាប់ទៅ Class ជាក់លាក់ | Interface |

កុំប្រើ Inheritance គ្រាន់តែដើម្បីយក Code មួយចំនួនទៅប្រើឡើងវិញ។ បើទំនាក់ទំនងមិនមែន “ជា” ប្រភេទនោះ អាចប្រើ Composition ដោយឱ្យ Object មួយកាន់ Object មួយទៀតជំនួស។

## ៩. Final Class និង Final Method

Keyword final ការពារ Class មិនឱ្យមាន Child Class ឬការពារ Method មិនឱ្យត្រូវ Override៖

```php
<?php
final class TokenGenerator
{
    public function generate(): string
    {
        return bin2hex(random_bytes(16));
    }
}

class SpecialTokenGenerator extends TokenGenerator
{
    // កំហុស៖ មិនអាច extends final Class បានទេ។
}
```

ឧទាហរណ៍ Final Method៖

```php
<?php
class BaseHandler
{
    final public function handle(): void
    {
        // Logic ដែល Child Class មិនគួរ Override។
    }
}
```

ប្រើ final នៅពេលចង់ការពារច្បាប់ ឬ Behavior សំខាន់ពីការប្ដូរដោយ Child Class។ កុំប្រើដោយគ្មានគោលបំណងច្បាស់។

## ១០. Mini Project៖ Notification Service

បង្កើតឯកសារមួយសម្រាប់ Interface និង Classes៖

```php
<?php
declare(strict_types=1);

interface MessageChannel
{
    public function send(string $to, string $message): bool;
}

class EmailChannel implements MessageChannel
{
    public function send(string $to, string $message): bool
    {
        echo "Email ទៅ {$to}: {$message}
";
        return true;
    }
}

class SmsChannel implements MessageChannel
{
    public function send(string $to, string $message): bool
    {
        echo "SMS ទៅ {$to}: {$message}
";
        return true;
    }
}

class NotificationService
{
    public function __construct(
        private MessageChannel $channel
    ) {
    }

    public function notify(string $to, string $message): bool
    {
        return $this->channel->send($to, $message);
    }
}

$emailService = new NotificationService(new EmailChannel());
$smsService = new NotificationService(new SmsChannel());

$emailService->notify('student@example.com', 'ចុះឈ្មោះជោគជ័យ');
$smsService->notify('+85512345678', 'មានការជូនដំណឹងថ្មី');
```

NotificationService ពឹងផ្អែកលើ MessageChannel Interface មិនមែន EmailChannel ឬ SmsChannel ផ្ទាល់ទេ។ ដូច្នេះអាចប្ដូរ Channel ឬសរសេរ Channel ថ្មីបានងាយ។

ឧទាហរណ៍នេះគ្រាន់តែបង្ហាញរចនាសម្ព័ន្ធ OOP។ វាមិនបានផ្ញើ Email ឬ SMS ពិតទេ។

## ១១. Best Practices

1. ប្រើ Inheritance ពេល Child ជា subtype ពិតប្រាកដរបស់ Parent។
2. ប្រើ Interface ដើម្បីបញ្ជាក់កិច្ចសន្យារួមរវាង Implementations ផ្សេងៗ។
3. ប្រើ Abstract Class ដើម្បីចែករំលែក State ឬ Implementation ដែល Child Classes មានរួមគ្នា។
4. រក្សា Method Signature ឱ្យស្របគ្នាពេល Override។
5. ហៅ Parent Constructor ដោយ parent::__construct() បើ Child មាន Constructor ហើយ Parent ត្រូវការកំណត់ State។
6. កុំប្រើ Private Parent Property ដោយរំពឹងថា Child Class អាចចូលប្រើបាន។
7. ពិចារណា Composition មុន Inheritance នៅពេលគ្រាន់តែចង់ប្រើមុខងាររបស់ Object ផ្សេង។
8. កុំបង្កើត Abstract Class ឬ Interface មុនពេលមានតម្រូវការពិតដែលវាដោះស្រាយ។

## ១២. កំហុសដែលជួបញឹកញាប់

- ព្យាយាម Instantiate Abstract Class។
- ភ្លេច Implement Abstract Methods ឬ Interface Methods ទាំងអស់។
- បន្ថយ Visibility របស់ Method ដែល Override ពី public ទៅ protected/private។
- ភ្លេចហៅ Parent Constructor។
- រំពឹងថា Child Class អាចចូលប្រើ Private Property របស់ Parent។
- ប្រើ Inheritance សម្រាប់ទំនាក់ទំនងដែលមិនមែន subtype។
- ប្រកាស Method Parameter Names ខុសពី Interface ហើយបង្កការភាន់ច្រឡំពេលប្រើ Named Arguments។
- គិតថា Interface មាន Implementation រួច; ជាទូទៅវាកំណត់តែកិច្ចសន្យា។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Shape

បង្កើត Abstract Class Shape ដែលមាន Abstract Method area(): float។ បង្កើត Circle និង Rectangle ដែល extends Shape ហើយគណនាផ្ទៃក្រឡារៀងៗខ្លួន។

### លំហាត់ទី ២៖ Logger Interface

បង្កើត Interface Logger ដែលមាន Method log(string $message): void។ Implement ជា FileLogger និង ConsoleLogger។

### លំហាត់ទី ៣៖ Method Override

បង្កើត Parent Class Employee មាន Method role()។ បង្កើត Manager និង Developer ដែល Override Method នោះ។

### លំហាត់ទី ៤៖ Multiple Interfaces

ឱ្យ Class Report អនុវត្ត Interfaces Renderable និង Exportable។ បង្កើត Method ទាំងអស់ដែលត្រូវការ។

### លំហាត់ទី ៥៖ សំណួរយល់ដឹង

ពន្យល់ពេលណាគួរប្រើ Interface, Abstract Class និង Composition។

## ១៤. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```php
<?php
abstract class Shape
{
    abstract public function area(): float;
}

class Circle extends Shape
{
    public function __construct(private float $radius)
    {
        if ($this->radius <= 0) {
            throw new InvalidArgumentException('Radius ត្រូវធំជាងសូន្យ។');
        }
    }

    public function area(): float
    {
        return pi() * $this->radius ** 2;
    }
}

class Rectangle extends Shape
{
    public function __construct(
        private float $width,
        private float $height
    ) {
        if ($this->width <= 0 || $this->height <= 0) {
            throw new InvalidArgumentException('ទំហំត្រូវធំជាងសូន្យ។');
        }
    }

    public function area(): float
    {
        return $this->width * $this->height;
    }
}
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
interface Logger
{
    public function log(string $message): void;
}

class ConsoleLogger implements Logger
{
    public function log(string $message): void
    {
        echo $message . PHP_EOL;
    }
}

class FileLogger implements Logger
{
    public function __construct(private string $path)
    {
    }

    public function log(string $message): void
    {
        file_put_contents($this->path, $message . PHP_EOL, FILE_APPEND | LOCK_EX);
    }
}
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
class Employee
{
    public function role(): string
    {
        return 'បុគ្គលិក';
    }
}

class Manager extends Employee
{
    public function role(): string
    {
        return 'អ្នកគ្រប់គ្រង';
    }
}

class Developer extends Employee
{
    public function role(): string
    {
        return 'អ្នកអភិវឌ្ឍន៍';
    }
}
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
interface Renderable
{
    public function render(): string;
}

interface Exportable
{
    public function export(): string;
}

class Report implements Renderable, Exportable
{
    public function render(): string
    {
        return 'បង្ហាញរបាយការណ៍';
    }

    public function export(): string
    {
        return 'នាំចេញរបាយការណ៍';
    }
}
```

### ចម្លើយលំហាត់ទី ៥

Interface កំណត់កិច្ចសន្យាដែល Implementations ផ្សេងៗត្រូវគោរព។ Abstract Class សមស្របពេលមាន State ឬ Code រួមដែល Child Classes គួរទទួល។ Composition សមស្របពេល Object មួយត្រូវប្រើសមត្ថភាពរបស់ Object មួយទៀត ដោយមិនមែនជាប្រភេទដូចគ្នា។

---

## សេចក្ដីសង្ខេប

- Inheritance ប្រើ extends ដើម្បីបន្ត Behavior ពី Parent Class។
- Child Class អាច Override Method និងហៅ Parent តាម parent::។
- Abstract Class មិនអាចបង្កើត Instance ដោយផ្ទាល់ ហើយអាចបង្ខំ Child Implement Methods។
- Interface កំណត់កិច្ចសន្យា; Class អាច implements Interfaces ច្រើន។
- Polymorphism អនុញ្ញាតឱ្យប្រើ Objects ផ្សេងៗតាម Interface រួម។
- ប្រើ Inheritance សម្រាប់ subtype និងគិតពី Composition ពេលចង់ប្រើសមត្ថភាពពី Object ផ្សេង។

## ឯកសារយោងផ្លូវការ

- [PHP Object Inheritance](https://www.php.net/manual/en/language.oop5.inheritance.php)
- [PHP Class Abstraction](https://www.php.net/manual/en/language.oop5.abstract.php)
- [PHP Object Interfaces](https://www.php.net/manual/en/language.oop5.interfaces.php)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្ស Namespaces, Autoloading និង Composer ដើម្បីរៀបចំ និងផ្ទុក Class Files ដោយស្វ័យប្រវត្តិ។
