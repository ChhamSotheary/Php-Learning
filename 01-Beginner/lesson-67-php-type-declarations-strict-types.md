# មេរៀនទី ៦៧៖ Type Declarations និង Strict Typing ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ប្រើ Scalar, Nullable និង Union Types ដើម្បីបញ្ជាក់ Input/Output។
- ប្រើ Intersection និង DNF Types នៅ PHP Version ដែលគាំទ្រ។
- យល់ពី `strict_types` និង TypeError។
- ប្រើ `mixed`, `void`, `never` និង `static` តាមអត្ថន័យ។
- រួមបញ្ចូល Native Types និង PHPDoc ដើម្បីពិពណ៌នា Collection/Shape។

> កំណែ PHP ត្រូវការខុសគ្នាតាម Type Feature៖ Union Types PHP 8.0, Intersection Types PHP 8.1, DNF Types PHP 8.2។ ពិនិត្យ PHP Version Constraint របស់ Project មុនប្រើ។

## ១. Type Declaration ជាអ្វី?

Type Declaration អាចកំណត់ប្រភេទ Argument, Return Value និង Properties ដើម្បីឱ្យ PHP ឬ Static Analyzer ចាប់ការប្រើប្រាស់ខុសមុនពេលវាបង្ក Bug៖

```php
<?php
declare(strict_types=1);

function formatPrice(int $cents): string
{
    return number_format($cents / 100, 2);
}
```

Type Declaration មិនជំនួសការផ្ទៀងផ្ទាត់ Input នៅព្រំដែនកម្មវិធីទេ។ ទិន្នន័យពី Form/JSON ត្រូវ Validate មុនបញ្ជូនទៅ Function ដែលរំពឹង Type ជាក់លាក់។

## ២. `strict_types` និង Scalar Types

ដាក់ `declare(strict_types=1);` នៅដើម PHP File មុន Code ផ្សេង។ វាប៉ះពាល់ដល់ Scalar Type Coercion សម្រាប់ Function Call ដែលចេញពី File នោះ។ ក្នុង Weak Mode PHP អាចបម្លែង Scalar ខ្លះដោយស្វ័យប្រវត្តិ; ក្នុង Strict Mode តម្លៃមិនស្របនឹង Type អាចបង្ក `TypeError`។

```php
<?php
declare(strict_types=1);

function add(int $left, int $right): int
{
    return $left + $right;
}

echo add(2, 3); // 5
// add('2', 3); // TypeError នៅ Strict Call Site
```

`strict_types` គឺការគ្រប់គ្រង Type Coercion មិនមែនជា Input Validation ឬ Security Boundary ទេ។ កុំរំពឹងថាវាផ្ទៀងផ្ទាត់ Email, Range, Authorization ឬទម្រង់ Domain ឱ្យអ្នក។

## ៣. Nullable និង Union Types

Nullable Type ទទួលតម្លៃប្រភេទមួយ ឬ `null`៖

```php
function findName(int $id): ?string
{
    // រកឈ្មោះ; ត្រឡប់ null បើរកមិនឃើញ។
    return null;
}
```

Union Type បញ្ជាក់ថាតម្លៃអាចជាប្រភេទណាមួយក្នុងបញ្ជី៖

```php
function normalizeId(int|string $id): int
{
    if (is_int($id)) {
        return $id;
    }

    if (!ctype_digit($id)) {
        throw new InvalidArgumentException('ID must be numeric.');
    }

    return (int) $id;
}
```

ប្រើ Union Type តែពេល Function ពិតជាគាំទ្រប្រភេទទាំងពីរ។ បើមានតម្លៃច្រើនន័យខុសគ្នា ការបង្កើត Value Object អាចធ្វើឱ្យ Contract ច្បាស់ជាង។

## ៤. Intersection និង DNF Types

Intersection Type តម្រូវឱ្យ Object មួយអនុវត្ត Type ទាំងអស់៖

```php
interface CacheAware
{
    public function cacheKey(): string;
}

interface JsonSerializableData
{
    public function toJson(): string;
}

function store(CacheAware&JsonSerializableData $value): void
{
    $key = $value->cacheKey();
    $json = $value->toJson();
}
```

DNF (Disjunctive Normal Form) Types អនុញ្ញាតឱ្យផ្សំ Union និង Intersection ចាប់ពី PHP 8.2៖

```php
function iterate(array|(Iterator&Countable) $items): void
{
    foreach ($items as $item) {
        // ដំណើរការ Item
    }
}
```

អាន DNF Type ជា “Array ឬ Object ដែលជា Iterator និង Countable ទាំងពីរ”។ សរសេរវាឱ្យសាមញ្ញ និងបំបែក Type Object បើ Signature ក្លាយជាពិបាកយល់។

## ៥. Return Types ពិសេស

- `void`៖ Function មិនផ្តល់តម្លៃត្រឡប់សម្រាប់ Caller។
- `never` (PHP 8.1+)៖ Function មិនបញ្ចប់ធម្មតា—វាបោះ Exception, Exit ឬបន្តដោយមិនត្រឡប់។
- `mixed` (PHP 8.0+)៖ ទទួល/ត្រឡប់ប្រភេទណាក៏បាន; ប្រើតែពេលមិនអាចបញ្ជាក់ Type បានច្បាស់។
- `static` (Return Type, PHP 8.0+)៖ បញ្ជាក់ថា Return Value មាន Type ជា Late Static Class។

```php
function fail(string $message): never
{
    throw new RuntimeException($message);
}

function logMessage(string $message): void
{
    error_log($message);
}
```

កុំប្រើ `mixed` ជំនួសការរចនា Type ដែលអាចកំណត់បាន; វាធ្វើឱ្យ Caller ត្រូវពិនិត្យគ្រប់ប្រភេទដោយដៃ។

## ៦. Native Types និង PHPDoc

Native Type អាចបញ្ជាក់ Array ប៉ុន្តែមិនប្រាប់ Type របស់ធាតុខាងក្នុងទេ។ PHPDoc អាចបន្ថែមព័ត៌មាន៖

```php
/**
 * @param list<int> $ids
 * @return array<int, string>
 */
function labelsFor(array $ids): array
{
    // Implementation
    return [];
}
```

ប្រើ PHPStan/Psalm ដើម្បីពិនិត្យ PHPDoc និង Type Flow។ Annotation ត្រូវតែត្រូវនឹងទិន្នន័យពិត; កុំប្រកាស Type មិនពិតដើម្បីឱ្យ Analyzer ឈប់រាយការណ៍។

## ៧. កំហុសដែលជួបញឹកញាប់

- គិតថា `strict_types` ជំនួស Input Validation។
- កំណត់ `mixed` គ្រប់ទីកន្លែងព្រោះមិនចង់សម្រេច Type។
- ប្រើ Union Type ដែលមានតម្លៃន័យខុសគ្នា និងពិបាកប្រើ។
- ប្រើ Syntax PHP 8.2/8.3 នៅលើ Runtime ចាស់។
- PHPDoc Type មិនស៊ីគ្នានឹង Native Type/Runtime Value។
- ភ្លេចថា Strict Scalar Checking ទាក់ទងនឹង File ដែលធ្វើ Call។

## ៨. លំហាត់

1. បើក `strict_types=1` នៅឯកសារ PHP របស់អ្នក និងសាកល្បង Scalar Argument។
2. សរសេរ Function ដែលទទួល `int|string` ហើយ Validate មុនបម្លែង។
3. បង្កើត Interface ពីរ ហើយសរសេរ Function ដែលទទួល Intersection Type។
4. សរសេរ Function `never` ដែលបោះ Exception។
5. បន្ថែម PHPDoc `list<string>` ឬ `array<int, User>` ទៅ Method មួយ ហើយរត់ Static Analyzer។

## ៩. ចម្លើយណែនាំខ្លី

- Type Declaration បញ្ជាក់ Contract របស់ Function/Property; Validate Input នៅព្រំដែនកម្មវិធីនៅតែចាំបាច់។
- `strict_types=1` ប៉ះពាល់ Scalar Coercion នៅ Call Site ក្នុង File ដែលហៅ Function។
- Union មានន័យថា Type មួយក្នុងចំណោមច្រើន; Intersection តម្រូវឱ្យ Type ទាំងអស់។
- ប្រើ `never` សម្រាប់ Function ដែលមិនត្រឡប់ធម្មតា និង `void` សម្រាប់ Function មិនមាន Return Value។
- Native Types បូក PHPDoc/Static Analysis ជួយពិពណ៌នា Collection ឱ្យច្បាស់។

## ឯកសារយោង

- [PHP Manual: Type Declarations](https://www.php.net/manual/en/language.types.declarations.php)
- [PHP Manual: Strict Typing](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.strict)
- [PHP Manual: Union Types](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.composite.union)
- [PHP Manual: Intersection Types](https://www.php.net/manual/en/language.types.declarations.php#language.types.declarations.composite.intersection)
- [មេរៀនទី ៤៨៖ Static Analysis ដោយ PHPStan](lesson-48-php-static-analysis-phpstan.md)
