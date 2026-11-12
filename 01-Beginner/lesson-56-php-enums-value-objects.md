# មេរៀនទី ៥៦៖ PHP Enums និង Value Objects

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ប្រើ Enum ដើម្បីកំណត់សំណុំតម្លៃដែលអាចមានបានច្បាស់លាស់។
- បម្លែង Backed Enum ទៅ/ពី String ឬ Integer។
- ប្រើ Readonly Class សម្រាប់ Value Object ដែលមិនផ្លាស់ប្តូរ។
- រក្សា Business Rules នៅជាមួយតម្លៃ ដើម្បីកុំឱ្យ Logic បែកខ្ញែក។
- ជ្រើសរើសរបៀប Serialize/Store Value Object នៅ API និង Database។

> Pure/Backed Enum មានចាប់ពី PHP 8.1។ Readonly Class មានចាប់ពី PHP 8.2។ ពិនិត្យ PHP Version របស់ Project មុនប្រើ Syntax ទាំងនេះ។

## ១. Enum ជាអ្វី?

Enum កំណត់សំណុំ Case មានកំណត់ ជំនួសឱ្យការប្រើ String ដែលអាចវាយខុស ឬលេខដែលគ្មានន័យ។ ឧទាហរណ៍ ស្ថានភាព Order អាចមានតែ pending, paid និង cancelled៖

```php
<?php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Cancelled = 'cancelled';
}
```

ប្រើ Enum ក្នុង Function Signature៖

```php
function canBeCancelled(OrderStatus $status): bool
{
    return $status === OrderStatus::Pending;
}
```

បើហៅជាមួយតម្លៃដែលមិនមែនជា OrderStatus នោះ Type Error កើតឡើងមុនពេល Function អនុវត្ត Logic។ វាជួយកាត់បន្ថយតម្លៃស្ថានភាពមិនត្រឹមត្រូវក្នុងកម្មវិធី។

## ២. Backed Enum សម្រាប់ Database និង API

Backed Enum មាន Backing Type តែមួយ គឺ `string` ឬ `int`។ អាចប្រើ `value` ដើម្បីទទួលតម្លៃដែលរក្សាទុកក្នុង Database/JSON៖

```php
<?php
$status = OrderStatus::Paid;

echo $status->value; // "paid"
```

បម្លែងតម្លៃដែលទទួលពី Database ឬ Request៖

```php
<?php
$status = OrderStatus::tryFrom('paid');

if ($status === null) {
    throw new InvalidArgumentException('Order status is invalid.');
}
```

- `OrderStatus::from($value)` បង្កើត Case ឬបោះ `ValueError` បើតម្លៃមិនត្រឹមត្រូវ។
- `OrderStatus::tryFrom($value)` ត្រឡប់ `null` បើមិនមាន Case ដែលត្រូវគ្នា។

ទិន្នន័យក្រៅកម្មវិធីអាចខុស ដូច្នេះ Validate មុនបន្ត។ ពេលប្តូរ Enum Case ដែលបានរក្សាទុកក្នុង Database/API ត្រូវមាន Migration និង Compatibility Plan; កុំលុប Case ដែលប្រើនៅទិន្នន័យចាស់ដោយមិនផ្លាស់ប្តូរវា។

## ៣. Methods និង Rules លើ Enum

អាចដាក់ Behavior ដែលពាក់ព័ន្ធនឹង Case ក្នុង Enum៖

```php
<?php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Cancelled = 'cancelled';

    public function isFinal(): bool
    {
        return match ($this) {
            self::Paid, self::Cancelled => true,
            self::Pending => false,
        };
    }
}
```

Enum មិនមែនជាកន្លែងសម្រាប់ដាក់ Business Logic ទាំងអស់ទេ។ Rule ដែលត្រូវការទិន្នន័យពី Order/User ឬ Database គួររស់នៅក្នុង Domain Service/Entity ដែលមាន Context គ្រប់គ្រាន់។

## ៤. Value Object ជាអ្វី?

Value Object តំណាងឱ្យតម្លៃតាមអត្ថន័យរបស់វា មិនមែនតាម Identity ដូចជា Entity។ ឧទាហរណ៍ Money មិនគួរត្រូវរក្សាទុកជាទសភាគ Float ព្រោះ Binary Floating Point អាចបង្កភាពមិនត្រឹមត្រូវក្នុងការគណនា។

ប្រើចំនួនគត់ជាឯកតាតូចបំផុត (ឧ. cents) និងកូដរូបិយប័ណ្ណ៖

```php
<?php
declare(strict_types=1);

final readonly class Money
{
    public function __construct(
        public int $minorUnits,
        public string $currency
    ) {
        if ($minorUnits < 0) {
            throw new InvalidArgumentException('Amount cannot be negative.');
        }

        if (!preg_match('/^[A-Z]{3}$/', $currency)) {
            throw new InvalidArgumentException('Currency code is invalid.');
        }
    }

    public function add(self $other): self
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Currencies must match.');
        }

        return new self(
            $this->minorUnits + $other->minorUnits,
            $this->currency
        );
    }
}
```

Readonly Class មានចាប់ពី PHP 8.2; Properties ត្រូវបានការពារពីការកែតម្លៃក្រោយ Construction។ ក្នុង PHP 8.1 អាចប្រើ Class ដែលមាន `private readonly` Properties ហើយផ្តល់តែ Getter/Method ដែលមិនកែតម្លៃ។

ត្រូវដឹងពីរបៀបគ្រប់គ្រង Currency Decimal Digits និង Overflow តាមអាជីវកម្ម។ ឧទាហរណ៍ខាងលើមិនមែនជា Money Library ពេញលេញសម្រាប់គ្រប់រូបិយប័ណ្ណទេ។

## ៥. Equality និង Business Meaning

Value Object ពីរអាចមានន័យស្មើគ្នា បើតម្លៃ និង Currency ស្មើគ្នា ទោះជា Instance ខុសគ្នាក៏ដោយ។ PHP Object `===` ប្រៀបធៀប Identity របស់ Object; ដូច្នេះបើត្រូវការ Value Equality បង្កើត Method ច្បាស់៖

```php
public function equals(self $other): bool
{
    return $this->minorUnits === $other->minorUnits
        && $this->currency === $other->currency;
}
```

Value Object គួរត្រូវបានបង្កើតតាម Constructor/Named Constructor ដែល Validate Invariant។ កុំឱ្យ Object អាចមានស្ថានភាពខូចបាន។

## ៦. Serialize ទៅ JSON និងរក្សាទុក

កំណត់ទម្រង់បម្លែងតម្លៃឱ្យច្បាស់៖

```php
<?php
$payload = [
    'status' => $orderStatus->value,
    'total' => [
        'amount_minor' => $money->minorUnits,
        'currency' => $money->currency,
    ],
];

$json = json_encode($payload, JSON_THROW_ON_ERROR);
```

នៅពេលទទួល JSON/Database Value ត្រូវបម្លែងតាម Constructor ឬ Enum `tryFrom()` ដើម្បី Validate ម្តងទៀត។ កុំសន្មតថា Data ត្រឹមត្រូវគ្រាន់តែវាមកពី Database ឬ API ចាស់។

## ៧. កំហុសដែលជួបញឹកញាប់

- ប្រើ String សេរីសម្រាប់សំណុំតម្លៃកំណត់ ហើយមាន Typo/Value ខុស។
- ហៅ `from()` លើ External Input ដោយមិនដោះស្រាយ `ValueError`។
- លុប ឬប្តូរ Backing Value ដែលមានក្នុង Database ដោយមិនធ្វើ Migration។
- ប្រើ Float សម្រាប់ចំនួនប្រាក់ដោយមិនយល់ពី Precision។
- គិតថា Object `===` មានន័យថា Value Equality។
- ដាក់ Business Logic ដែលត្រូវការទិន្នន័យច្រើនក្នុង Enum Case។

## ៨. លំហាត់

1. បង្កើត `PaymentStatus` Backed Enum ដែលមាន Pending, Succeeded និង Failed។
2. ប្រើ `tryFrom()` ដើម្បី Validate Status ពី API Request។
3. បន្ថែម Method `isTerminal()` សម្រាប់ Status ដែលមិនអាចបន្តបាន។
4. បង្កើត Readonly Value Object `EmailAddress` ដែល Normalize/Validate Email ក្នុង Constructor។
5. Serialize Enum និង Value Object ទៅ JSON ដោយកំណត់ទម្រង់ API ឱ្យច្បាស់។

## ៩. ចម្លើយណែនាំខ្លី

- Enum កំណត់ Value Case ដែលអាចមាន; Backed Enum មាន String ឬ Integer Value។
- ប្រើ `tryFrom()` សម្រាប់ទិន្នន័យក្រៅ និងពិនិត្យ `null`។
- Value Object រក្សា Invariant និង Behavior របស់តម្លៃ ដោយគ្មាន Identity ផ្ទាល់ខ្លួន។
- ប្រើ Minor Units/ចំនួនគត់សម្រាប់ប្រាក់តាម Business Rules ជំនួស Float ទូទៅ។
- កំណត់ Serialization/Database Value ជា Contract ហើយគ្រប់គ្រង Migration ពេលប្តូរ។

## ឯកសារយោង

- [PHP Manual: Enumerations](https://www.php.net/manual/en/language.enumerations.php)
- [PHP Manual: Backed Enumerations](https://www.php.net/manual/en/language.enumerations.backed.php)
- [PHP Manual: Readonly Classes](https://www.php.net/manual/en/language.oop5.basic.php#language.oop5.basic.class.readonly)
- [មេរៀនទី ២០៖ Classes, Objects និង Constructor](lesson-20-php-oop-classes-objects-constructors.md)
