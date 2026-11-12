# មេរៀនទី ៦៥៖ Closures និង Callable ក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖
- បង្កើត Anonymous Function និង Closure។
- បញ្ជូន Function/Method ទៅជា Argument ដោយប្រើ `callable`។
- យល់ពីការចាប់អថេរខាងក្រៅដោយ `use` និង Arrow Function។
- ប្រើ First-Class Callable Syntax ដែលមានចាប់ពី PHP 8.1។
- ជ្រើសរើសរវាង Function ធម្មតា, Closure និង Class ដោយមិនធ្វើឱ្យ Code ស្មុគស្មាញ។

## ១. Closure ជាអ្វី?

Closure គឺ Anonymous Function ដែលអាចរក្សាទុកក្នុង Variable និងបញ្ជូនទៅ Function ផ្សេងបាន៖

```php
<?php
$greet = function (string $name): string {
    return 'សួស្តី ' . $name;
};

echo $greet('អ្នករៀន');
```

Closure មានប្រយោជន៍សម្រាប់ Callback, Strategy តូចៗ និងការចាប់ Context ដែលចាំបាច់។ បើ Logic មានច្រើន ឬត្រូវ Reuse ទូទាំងកម្មវិធី សូមបង្កើត Named Function ឬ Class ដែលមានឈ្មោះច្បាស់។

## ២. បញ្ជូន Callable ទៅ Function

Parameter អាចប្រើ Type `callable` ដើម្បីទទួល Function ឬ Method ដែលអាចហៅបាន៖

```php
<?php
function transformAll(array $values, callable $transform): array
{
    $result = [];

    foreach ($values as $value) {
        $result[] = $transform($value);
    }

    return $result;
}

$labels = transformAll(
    [1, 2, 3],
    fn (int $number): string => 'លេខ ' . $number
);
```

Arrow Function មានចាប់ពី PHP 7.4។ វាជា Syntax ខ្លីសម្រាប់ Expression មួយ ហើយចាប់អថេរខាងក្រៅដោយ Value ដោយស្វ័យប្រវត្តិ។ ប្រសិនបើ Callback ត្រូវការ Logic ច្រើន បំបែកជា Function/Method ដើម្បីងាយអាន និង Test។

## ៣. ចាប់អថេរដោយ `use`

Anonymous Function មិនទទួល Variable ពី Parent Scope ដោយស្វ័យប្រវត្តិទេ; ប្រើ `use`៖

```php
<?php
$prefix = 'PHP: ';

$format = function (string $text) use ($prefix): string {
    return $prefix . $text;
};

echo $format('Closure');
```

ការចាប់ដោយធម្មតាគឺចាប់តម្លៃតាម Value នៅពេល Closure ត្រូវបានបង្កើត។ បើចង់ឱ្យ Closure អាន/កែ Variable ដដែលតាម Reference ប្រើ `&`៖

```php
<?php
$count = 0;

$increment = function () use (&$count): void {
    $count++;
};

$increment();
echo $count; // 1
```

Reference អាចធ្វើឱ្យ State ផ្លាស់ប្តូរពីកន្លែងជាច្រើន និងពិបាក Debug។ ប្រើតែពេលចេតនាច្បាស់។

## ៤. Arrow Function និង Closure

Arrow Function សរសេរបានខ្លី ហើយចាប់ Variable ខាងក្រៅតាម Value ដោយស្វ័យប្រវត្តិ៖

```php
<?php
$taxRate = 0.1;
$withTax = fn (float $amount): float => $amount * (1 + $taxRate);
```

Arrow Function មាន Expression មួយ។ សម្រាប់ Statement ច្រើន, Exception Handling ឬការកែ State ត្រូវប្រើ Anonymous Function ធម្មតា ឬ Method។

## ៥. First-Class Callable Syntax (PHP 8.1+)

ចាប់ពី PHP 8.1 អាចយក Function/Method មកជា Closure ដោយ Syntax `...`៖

```php
<?php
$length = strlen(...);
echo $length('PHP'); // 3

final class PriceFormatter
{
    public function format(int $cents): string
    {
        return number_format($cents / 100, 2);
    }
}

$formatter = new PriceFormatter();
$callback = $formatter->format(...);

echo $callback(1234);
```

First-Class Callable Syntax បង្កើត Closure ក្នុង Scope ដែលបានទទួល Callable ហើយងាយសម្រាប់ Static Analysis ជាង String/Array Callable ចាស់ៗ។ `...` ជាផ្នែកពិតរបស់ Syntax មិនមែនសញ្ញាបង្ហាញថាកូដត្រូវបានកាត់ទេ។

## ៦. ប្រើ `Closure` ឬ `callable`?

- ប្រើ `callable` ជា Parameter Type ពេល Function អាចទទួល Function, Method Callable ឬ Closure។
- ប្រើ `Closure` ជា Type សម្រាប់ Property/Dependency ដែលត្រូវរក្សាទុកជាអចិន្ត្រៃយ៍ក្នុង Object។
- PHP មិនអនុញ្ញាត `callable` ជា Property Type ទេ; បម្លែងទៅ Closure ដោយ `Closure::fromCallable()` ឬ First-Class Callable Syntax តាមករណី។
- សម្រាប់ Service/Dependency ស្មុគស្មាញ ប្រើ Interface/Class ជំនួស Closure ទទេ ដើម្បីឱ្យ Contract ច្បាស់។

```php
<?php
final class CallbackRunner
{
    public function __construct(
        private Closure $callback
    ) {
    }

    public function run(string $value): string
    {
        return ($this->callback)($value);
    }
}

$runner = new CallbackRunner(strtoupper(...));
echo $runner->run('php');
```

## ៧. កំហុសដែលជួបញឹកញាប់

- គិតថា Closure ឃើញ Variable ខាងក្រៅដោយស្វ័យប្រវត្តិ; Anonymous Function ត្រូវប្រើ `use`។
- ប្រើ `use (&$value)` ដោយមិនយល់ថា Reference កំពុងចែករំលែក State។
- ដាក់ Logic ច្រើនក្នុង Arrow Function/Callback ដល់ពិបាកអាន។
- រក្សាទុក String Callable ដែលងាយ Typo ជំនួស First-Class Callable ក្នុង PHP 8.1+។
- ប្រើ Closure សម្រាប់ Service ធំដែលគួរមាន Interface/Named Class។
- ភ្លេច Type Hint និង Return Type នៅ Callback។

## ៨. លំហាត់

1. សរសេរ Function ដែលទទួល `callable` ហើយបម្លែង Array នៃតម្លៃ។
2. បង្កើត Closure ដែលចាប់ Prefix ដោយ `use` និង Arrow Function ដែលចាប់ Value ដោយស្វ័យប្រវត្តិ។
3. ប្រៀបធៀបការចាប់ Variable ដោយ Value និង Reference។
4. យក Object Method មកជា First-Class Callable ក្នុង PHP 8.1+។
5. សម្រេចថា Logic មួយគួរជា Closure ឬ Class ហើយពន្យល់មូលហេតុ។

## ៩. ចម្លើយណែនាំខ្លី

- Closure ជា Function ដែលអាចរក្សាទុក/បញ្ជូនបាន; Anonymous Function ចាប់ Variable ដោយ `use`។
- Arrow Function ចាប់ Variable ខាងក្រៅតាម Value ដោយស្វ័យប្រវត្តិ។
- `callable` សមស្របសម្រាប់ Parameter; Property គួរប្រើ `Closure` ឬ Interface។
- First-Class Callable Syntax `functionName(...)` មានចាប់ពី PHP 8.1។
- បើ Behavior មានទំហំធំ ឬត្រូវថែទាំជាប្រព័ន្ធ ប្រើ Named Function/Class។

## ឯកសារយោង

- [PHP Manual: Anonymous Functions](https://www.php.net/manual/en/functions.anonymous.php)
- [PHP Manual: First-Class Callable Syntax](https://www.php.net/manual/en/functions.first_class_callable_syntax.php)
- [មេរៀនទី ៦៖ PHP Functions](lesson-06-php-functions.md)
- [មេរៀនទី ២៥៖ Dependency Injection និង Design Patterns](lesson-25-php-dependency-injection-design-patterns.md)
