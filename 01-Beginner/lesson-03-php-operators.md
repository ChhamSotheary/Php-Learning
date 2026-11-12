# មេរៀនទី ៣៖ PHP Operators

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> ឯកសារ៖ `01-Beginner/lesson-03-php-operators.md`

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកអាច៖

- ពន្យល់ថា Operator និង Operand ជាអ្វី។
- ប្រើ Arithmetic Operators សម្រាប់គណនា។
- ប្រើ Assignment Operators ដើម្បីកំណត់ និងកែប្រែតម្លៃ។
- ប្រៀបធៀបទិន្នន័យដោយ Comparison Operators។
- ប្រើ Logical Operators ដើម្បីភ្ជាប់លក្ខខណ្ឌ។
- យល់ពី Increment និង Decrement។
- ប្រើ String Operators, Null Coalescing និង Ternary Operator។
- យល់ពី Operator Precedence និងការប្រើវង់ក្រចក។

## ២. Operator គឺជាអ្វី?

**Operator** គឺជានិមិត្តសញ្ញា ឬពាក្យគន្លឹះដែលប្រើដើម្បីធ្វើប្រតិបត្តិការលើតម្លៃ ឬ Variables។ តម្លៃដែល Operator ធ្វើការលើវា ហៅថា **Operand**។

ឧទាហរណ៍៖

```php
<?php
$result = 10 + 5;
echo $result;
```

- `+` គឺជា Operator។
- `10` និង `5` គឺជា Operands។
- `15` គឺជាលទ្ធផល។

## ៣. Arithmetic Operators — ប្រមាណវិធីគណិតវិទ្យា

ប្រើសម្រាប់គណនាលេខ។

| Operator | អត្ថន័យ | ឧទាហរណ៍ | លទ្ធផល |
|---|---|---:|---:|
| `+` | បូក | `10 + 3` | `13` |
| `-` | ដក | `10 - 3` | `7` |
| `*` | គុណ | `10 * 3` | `30` |
| `/` | ចែក | `10 / 2` | `5` |
| `%` | សំណល់ពីការចែក | `10 % 3` | `1` |
| `**` | ស្វ័យគុណ | `2 ** 3` | `8` |

ឧទាហរណ៍កូដ៖

```php
<?php
$a = 10;
$b = 3;

echo "Addition: " . ($a + $b) . PHP_EOL;
echo "Subtraction: " . ($a - $b) . PHP_EOL;
echo "Multiplication: " . ($a * $b) . PHP_EOL;
echo "Division: " . ($a / $b) . PHP_EOL;
echo "Modulus: " . ($a % $b) . PHP_EOL;
echo "Exponentiation: " . ($a ** $b) . PHP_EOL;
```

លទ្ធផល៖

```text
Addition: 13
Subtraction: 7
Multiplication: 30
Division: 3.3333333333333
Modulus: 1
Exponentiation: 1000
```

ចំណាំ៖ កុំចែកនឹងសូន្យ (`0`) ព្រោះវាមិនមែនជាការចែកដែលអាចធ្វើបានតាមគណិតវិទ្យា ហើយ PHP នឹងបង្កើត Error/DivisionByZeroError តាមប្រតិបត្តិការ។

### ការប្រើ Modulus ដើម្បីពិនិត្យលេខគូ និងលេខសេស

`%` ផ្តល់សំណល់ពីការចែក។ បើចែកលេខដោយ 2 ហើយសំណល់ស្មើ 0 នោះជាលេខគូ។

```php
<?php
$number = 14;

if ($number % 2 === 0) {
    echo "Even number";
} else {
    echo "Odd number";
}
```

លទ្ធផល៖

```text
Even number
```

យើងនឹងរៀន `if...else` ឱ្យលម្អិតក្នុងមេរៀន Conditional Statements។

## ៤. Assignment Operators — ប្រមាណវិធីកំណត់តម្លៃ

Assignment Operator ប្រើសម្រាប់ដាក់តម្លៃទៅក្នុង Variable។ `=` មិនមែនមានន័យថា “ស្មើ” ក្នុងការប្រៀបធៀបទេ ប៉ុន្តែមានន័យថា “កំណត់តម្លៃ”។

```php
<?php
$score = 10;
$score += 5; // ដូច $score = $score + 5

echo $score . PHP_EOL;

$score -= 3; // ដូច $score = $score - 3
echo $score . PHP_EOL;

$score *= 2; // ដូច $score = $score * 2
echo $score . PHP_EOL;

$score /= 4; // ដូច $score = $score / 4
echo $score . PHP_EOL;
```

លទ្ធផល៖

```text
15
12
24
6
```

Assignment Operators ដែលគួរស្គាល់៖

| Operator | ស្មើនឹង |
|---|---|
| `$x += $y` | `$x = $x + $y` |
| `$x -= $y` | `$x = $x - $y` |
| `$x *= $y` | `$x = $x * $y` |
| `$x /= $y` | `$x = $x / $y` |
| `$x %= $y` | `$x = $x % $y` |
| `$text .= $more` | ភ្ជាប់ String ទៅចុង Variable |

ឧទាហរណ៍ String Assignment៖

```php
<?php
$message = "Hello";
$message .= " PHP";
$message .= " Developer";

echo $message;
```

លទ្ធផល៖

```text
Hello PHP Developer
```

## ៥. Comparison Operators — ប្រមាណវិធីប្រៀបធៀប

Comparison Operators ប្រើសម្រាប់ប្រៀបធៀបតម្លៃ។ លទ្ធផលជាទូទៅគឺ Boolean៖ `true` ឬ `false`។

| Operator | អត្ថន័យ |
|---|---|
| `==` | ស្មើគ្នាតាមតម្លៃ បន្ទាប់ពី PHP អាចបម្លែងប្រភេទ |
| `===` | ស្មើទាំងតម្លៃ និងប្រភេទទិន្នន័យ |
| `!=` ឬ `<>` | មិនស្មើតាមតម្លៃ |
| `!==` | មិនស្មើតម្លៃ ឬប្រភេទ |
| `>` | ធំជាង |
| `<` | តូចជាង |
| `>=` | ធំជាង ឬស្មើ |
| `<=` | តូចជាង ឬស្មើ |
| `<=>` | Spaceship Operator |

ឧទាហរណ៍សំខាន់៖

```php
<?php
$a = 5;
$b = "5";

var_dump($a == $b);
var_dump($a === $b);
var_dump($a != $b);
var_dump($a !== $b);
```

លទ្ធផល៖

```text
bool(true)
bool(false)
bool(false)
bool(true)
```

ការពន្យល់៖

- `$a == $b` បាន `true` ព្រោះតម្លៃទាំងពីរអាចប្រៀបធៀបថាស្មើគ្នា បន្ទាប់ពី Type Juggling។
- `$a === $b` បាន `false` ព្រោះ `$a` ជា Integer តែ `$b` ជា String។
- ជាទម្លាប់ល្អ ប្រើ `===` និង `!==` នៅពេលអ្នកចង់ប្រៀបធៀបទាំងតម្លៃ និងប្រភេទឱ្យច្បាស់។

### Spaceship Operator

`<=>` ប្រៀបធៀបតម្លៃពីរ៖

- ត្រឡប់ `-1` ប្រសិនបើខាងឆ្វេងតូចជាងខាងស្តាំ។
- ត្រឡប់ `0` ប្រសិនបើស្មើគ្នា។
- ត្រឡប់ `1` ប្រសិនបើខាងឆ្វេងធំជាងខាងស្តាំ។

```php
<?php
var_dump(3 <=> 8);
var_dump(5 <=> 5);
var_dump(9 <=> 2);
```

លទ្ធផល៖

```text
int(-1)
int(0)
int(1)
```

## ៦. Logical Operators — ប្រមាណវិធីតក្កវិជ្ជា

Logical Operators ប្រើដើម្បីភ្ជាប់ ឬបញ្ច្រាសលក្ខខណ្ឌ។

### ៦.១ AND (`&&`)

លទ្ធផលជា `true` តែពេលលក្ខខណ្ឌទាំងពីរពិត។

```php
<?php
$age = 20;
$hasTicket = true;

var_dump($age >= 18 && $hasTicket === true);
```

លទ្ធផល៖

```text
bool(true)
```

### ៦.២ OR (`||`)

លទ្ធផលជា `true` ប្រសិនបើមានលក្ខខណ្ឌយ៉ាងហោចណាស់មួយពិត។

```php
<?php
$hasStudentCard = false;
$hasLibraryCard = true;

var_dump($hasStudentCard || $hasLibraryCard);
```

លទ្ធផល៖

```text
bool(true)
```

### ៦.៣ NOT (`!`)

បញ្ច្រាស Boolean៖ `true` ទៅជា `false` និង `false` ទៅជា `true`។

```php
<?php
$isClosed = false;

var_dump(!$isClosed);
```

លទ្ធផល៖

```text
bool(true)
```

### តារាងសង្ខេប

| A | B | A && B | A || B |
|---|---|---|---|
| true | true | true | true |
| true | false | false | true |
| false | true | false | true |
| false | false | false | false |

**ចំណាំអំពី Short-circuit Evaluation៖** ក្នុង `A && B` ប្រសិនបើ A ជា `false` PHP មិនចាំបាច់វាយតម្លៃ B ទេ។ ក្នុង `A || B` ប្រសិនបើ A ជា `true` PHP មិនចាំបាច់វាយតម្លៃ B ទេ។ ដូច្នេះ កុំដាក់កូដដែលមានផលប៉ះពាល់សំខាន់ៗនៅក្នុងផ្នែកដែលអាចមិនត្រូវបានដំណើរការ។

## ៧. Increment និង Decrement

- Increment (`++`) បន្ថែមតម្លៃ 1។
- Decrement (`--`) ដកតម្លៃ 1។

មានពីររបៀប៖

- Prefix: `++$x` បន្ថែមមុនពេលយកតម្លៃទៅប្រើ។
- Postfix: `$x++` យកតម្លៃបច្ចុប្បន្នទៅប្រើសិន រួចទើបបន្ថែម។

```php
<?php
$x = 5;

echo $x++ . PHP_EOL; // បង្ហាញ 5 រួច $x ក្លាយជា 6
echo $x . PHP_EOL;   // បង្ហាញ 6

echo ++$x . PHP_EOL; // បន្ថែមទៅ 7 មុន រួចបង្ហាញ 7
echo $x . PHP_EOL;   // បង្ហាញ 7
```

លទ្ធផល៖

```text
5
6
7
7
```

ឧទាហរណ៍ Decrement៖

```php
<?php
$count = 3;

$count--;
echo $count . PHP_EOL;

--$count;
echo $count . PHP_EOL;
```

លទ្ធផល៖

```text
2
1
```

សម្រាប់កូដដែលងាយអាន ជាពិសេសពេលទើបរៀន គួរបំបែកការបង្កើនតម្លៃ និងការបង្ហាញតម្លៃទៅជាបន្ទាត់ផ្សេងៗ។

## ៨. String Operators

PHP មាន Operator សម្រាប់ភ្ជាប់ String៖

- `.` ភ្ជាប់ String។
- `.=` ភ្ជាប់ String ទៅចុង Variable ហើយរក្សាទុកលទ្ធផលក្នុង Variable ដើម។

```php
<?php
$first = "Learn";
$second = " PHP";

$sentence = $first . $second;
echo $sentence . PHP_EOL;

$sentence .= " step by step";
echo $sentence;
```

លទ្ធផល៖

```text
Learn PHP
Learn PHP step by step
```

## ៩. Null Coalescing Operator (`??`)

ប្រើសម្រាប់ផ្តល់តម្លៃជំនួស ប្រសិនបើ Variable មិនមាន ឬមានតម្លៃ `null`។

```php
<?php
$username = null;

$displayName = $username ?? "Guest";

echo $displayName;
```

លទ្ធផល៖

```text
Guest
```

ឧទាហរណ៍មួយទៀត៖

```php
<?php
$settings = [
    "theme" => "dark"
];

$theme = $settings["theme"] ?? "light";
$language = $settings["language"] ?? "km";

echo $theme . PHP_EOL;
echo $language . PHP_EOL;
```

លទ្ធផល៖

```text
dark
km
```

Null Coalescing មានប្រយោជន៍ពេលអានទិន្នន័យដែលអាចមិនមាន។ វាមិនដូចការពិនិត្យថាតម្លៃជា `false` ឬ `0` ទេ៖ `??` ពិនិត្យការមិនមាន ឬ `null`។

## ១០. Ternary Operator (`?:`)

Ternary Operator ជាវិធីសរសេរ Conditional Expression ខ្លីៗ។

ទម្រង់៖

```php
$variable = condition ? value_if_true : value_if_false;
```

ឧទាហរណ៍៖

```php
<?php
$score = 75;

$result = $score >= 50 ? "Passed" : "Failed";

echo $result;
```

លទ្ធផល៖

```text
Passed
```

ប្រើវាសម្រាប់លក្ខខណ្ឌខ្លីៗដែលអាចអានយល់បាន។ បើមានលក្ខខណ្ឌច្រើន ឬស្មុគស្មាញ គួរប្រើ `if...else` ដើម្បីឱ្យកូដច្បាស់។

## ១១. Operator Precedence — លំដាប់អាទិភាព

ពេលមាន Operator ច្រើនក្នុង Expression តែមួយ PHP អនុវត្តតាមលំដាប់អាទិភាពរបស់វា។

ឧទាហរណ៍៖

```php
<?php
$result = 2 + 3 * 4;
echo $result . PHP_EOL;

$result2 = (2 + 3) * 4;
echo $result2 . PHP_EOL;
```

លទ្ធផល៖

```text
14
20
```

ការពន្យល់៖

- ក្នុង Expression ដំបូង គុណមុនបូក៖ `2 + (3 * 4) = 14`។
- ក្នុង Expression ទីពីរ វង់ក្រចកធ្វើឱ្យបូកមុន៖ `(2 + 3) * 4 = 20`។

**គន្លឹះ៖** ប្រើវង់ក្រចក `()` នៅពេល Expression អាចបកស្រាយបានច្រើនរបៀប។ វាជួយឱ្យអ្នកអានកូដយល់ច្បាស់ និងកាត់បន្ថយកំហុស។

## ១២. Mini Project៖ កម្មវិធីគណនាវិក្កយបត្រ

បង្កើតឯកសារ `bill.php`៖

```php
<?php
$productName = "Keyboard";
$price = 25.50;
$quantity = 2;
$discountPercent = 10;

$subtotal = $price * $quantity;
$discount = $subtotal * ($discountPercent / 100);
$total = $subtotal - $discount;

echo "===== CUSTOMER BILL =====" . PHP_EOL;
echo "Product: " . $productName . PHP_EOL;
echo "Price: $" . $price . PHP_EOL;
echo "Quantity: " . $quantity . PHP_EOL;
echo "Subtotal: $" . $subtotal . PHP_EOL;
echo "Discount: " . $discountPercent . "%" . PHP_EOL;
echo "Discount amount: $" . $discount . PHP_EOL;
echo "Total: $" . $total . PHP_EOL;

$isLargeOrder = $quantity >= 2;
echo "Large order: " . ($isLargeOrder ? "Yes" : "No") . PHP_EOL;
```

លទ្ធផល៖

```text
===== CUSTOMER BILL =====
Product: Keyboard
Price: $25.5
Quantity: 2
Subtotal: $51
Discount: 10%
Discount amount: $5.1
Total: $45.9
Large order: Yes
```

ការពន្យល់៖

1. `$subtotal` គឺតម្លៃមួយឯកតាគុណនឹងចំនួន។
2. `$discount` គឺ Subtotal គុណនឹងភាគរយបញ្ចុះតម្លៃចែកនឹង 100។
3. `$total` គឺ Subtotal ដកចំនួនបញ្ចុះតម្លៃ។
4. `$quantity >= 2` ជា Comparison Expression ដែលផ្តល់ Boolean។
5. Ternary Operator បង្ហាញ `Yes` ឬ `No` តាមលទ្ធផល។

ចំណាំ៖ គម្រោងនេះបង្ហាញគោលការណ៍គណនាមូលដ្ឋានប៉ុណ្ណោះ។ ប្រព័ន្ធហិរញ្ញវត្ថុពិតអាចត្រូវការរបៀបគ្រប់គ្រងចំនួនទសភាគ និងការបង្គត់លេខឱ្យបានត្រឹមត្រូវជាងនេះ។

## ១៣. របៀបដំណើរការ

បើក Terminal នៅក្នុង Folder ដែលមាន `bill.php` ហើយរត់៖

```bash
php bill.php
```

បើអ្នកឃើញ Customer Bill និងតម្លៃសរុបដូចលទ្ធផលខាងលើ នោះកម្មវិធីដំណើរការបានត្រឹមត្រូវ។

## ១៤. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ច្រឡំ `=` និង `==`**៖ `=` កំណត់តម្លៃ; `==` ប្រៀបធៀបតម្លៃ។
2. **ច្រឡំ `==` និង `===`**៖ `===` ពិនិត្យទាំងតម្លៃ និងប្រភេទ។
3. **ប្រើ `&&` ជំនួស `||` ឬផ្ទុយមកវិញ**៖ AND ត្រូវការលក្ខខណ្ឌទាំងអស់ពិត; OR ត្រូវការយ៉ាងហោចណាស់មួយពិត។
4. **ភ្លេចលំដាប់គណនា**៖ ប្រើវង់ក្រចកដើម្បីបញ្ជាក់លំដាប់។
5. **មិនពិនិត្យការចែកនឹងសូន្យ**៖ ត្រូវធានាថា Divisor មិនមែនសូន្យមុនគណនា។
6. **ប្រើ Ternary ច្រើនជាន់ពេក**៖ កូដអាចពិបាកអាន។ ប្រើ `if...else` សម្រាប់ករណីស្មុគស្មាញ។
7. **ច្រឡំ `??` ជាមួយការពិនិត្យ Boolean**៖ Null Coalescing ផ្តោតលើ `null` ឬមិនមានតម្លៃ មិនមែនលើតម្លៃ False ទាំងអស់ទេ។

## ១៥. លំហាត់អនុវត្ត

### លំហាត់ A — ងាយ

កំណត់ Variables៖

```php
$a = 20;
$b = 6;
```

សរសេរកូដបង្ហាញលទ្ធផលនៃ៖

- បូក
- ដក
- គុណ
- ចែក
- សំណល់ពីការចែក
- ស្វ័យគុណ `$a ** 2`

### លំហាត់ B — មធ្យម

បង្កើត Variables៖

```php
$math = 80;
$english = 70;
$computer = 90;
```

គណនាពិន្ទុមធ្យម ហើយប្រើ Ternary Operator ដើម្បីកំណត់៖

- Average ចាប់ពី 50 ឡើងទៅ៖ `Passed`
- តិចជាង 50៖ `Failed`

### លំហាត់ C — មធ្យមទៅពិបាក

មានកូដ៖

```php
$age = 19;
$hasPermission = true;
```

បង្កើត Expression ដែលលទ្ធផលជា `true` ប្រសិនបើអាយុចាប់ពី 18 ឡើងទៅ និងមាន Permission។ បន្ទាប់មកសាកល្បងប្តូរតម្លៃទាំងពីរ។

### លំហាត់ D — ពិបាកជាងមុន

បង្កើតកម្មវិធីគណនាតម្លៃសរុបរបស់ទំនិញ 3 មុខ។ សម្រាប់មុខនីមួយៗ មានតម្លៃ និងចំនួន។ គណនា Subtotal នីមួយៗ បូកជាតម្លៃសរុប ហើយប្រើ Comparison Operator ពិនិត្យថាតើតម្លៃសរុបលើស 100 ដុល្លារឬទេ។

### លំហាត់ E — សាកល្បងការយល់ដឹង

ព្យាករលទ្ធផលមុនរត់កូដ៖

```php
<?php
$x = 4;

echo $x++ . PHP_EOL;
echo ++$x . PHP_EOL;
echo $x . PHP_EOL;

var_dump(5 == "5");
var_dump(5 === "5");
```

សរសេរចម្លើយរបស់អ្នក រួចដំណើរការកូដដើម្បីពិនិត្យ។

## ១៦. សំណួររំលឹក

សូមឆ្លើយសំណួរទាំងនេះមុនបើកចម្លើយ៖

1. Operator និង Operand ខុសគ្នាដូចម្តេច?
2. តើ `%` ប្រើសម្រាប់អ្វី?
3. តើ `+=` មានន័យដូចម្តេច?
4. តើខុសគ្នាដូចម្តេចរវាង `==` និង `===`?
5. តើ `&&` និង `||` ខុសគ្នាដូចម្តេច?
6. តើ Prefix Increment និង Postfix Increment ខុសគ្នាដូចម្តេច?
7. តើ `??` ប្រើនៅពេលណា?
8. ហេតុអ្វីយើងគួរប្រើវង់ក្រចកក្នុង Expression?

## ១៧. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

<details>
<summary>ចុចទីនេះដើម្បីមើលចម្លើយសំណួររំលឹក</summary>

1. Operator ជានិមិត្តសញ្ញា/ពាក្យសម្រាប់ប្រតិបត្តិការ; Operand ជាតម្លៃដែល Operator ធ្វើការលើ។
2. គណនាសំណល់ពីការចែក។
3. បូកតម្លៃថ្មីទៅក្នុង Variable ដើម ហើយរក្សាទុកលទ្ធផល។
4. `==` ប្រៀបធៀបតម្លៃដោយអាចមាន Type Juggling; `===` ប្រៀបធៀបទាំងតម្លៃ និងប្រភេទ។
5. AND ត្រូវការលក្ខខណ្ឌទាំងអស់ពិត; OR ត្រូវការយ៉ាងហោចណាស់មួយពិត។
6. Prefix បង្កើនមុនយកតម្លៃទៅប្រើ; Postfix យកតម្លៃបច្ចុប្បន្នទៅប្រើសិន រួចទើបបង្កើន។
7. ប្រើផ្តល់តម្លៃជំនួសពេលតម្លៃមិនមាន ឬជា `null`។
8. ដើម្បីកំណត់លំដាប់ប្រតិបត្តិការ និងធ្វើឱ្យ Expression ងាយយល់។

</details>

<details>
<summary>ចុចទីនេះដើម្បីមើលគំរូចម្លើយលំហាត់ B</summary>

```php
<?php
$math = 80;
$english = 70;
$computer = 90;

$average = ($math + $english + $computer) / 3;
$result = $average >= 50 ? "Passed" : "Failed";

echo "Average: " . $average . PHP_EOL;
echo "Result: " . $result . PHP_EOL;
```

លទ្ធផល៖

```text
Average: 80
Result: Passed
```

</details>

<details>
<summary>ចុចទីនេះដើម្បីមើលចម្លើយលំហាត់ E</summary>

លទ្ធផល៖

```text
4
6
6
bool(true)
bool(false)
```

ការពន្យល់៖ `$x++` បង្ហាញ 4 រួចបង្កើនទៅ 5។ បន្ទាប់មក `++$x` បង្កើនទៅ 6 មុនបង្ហាញ។ ដូច្នេះតម្លៃចុងក្រោយគឺ 6។ `5 == "5"` ពិតតាមតម្លៃ; `5 === "5"` មិនពិតព្រោះប្រភេទខុសគ្នា។

</details>

## ១៨. កិច្ចការផ្ទះ

- ធ្វើលំហាត់ A ដល់ E។
- បង្កើត Folder `lesson-03` ដើម្បីរក្សាទុកឯកសារកូដ។
- កែប្រែ Mini Project ឱ្យមានទំនិញច្រើនមុខ។
- ព្យាយាមពន្យល់ពីភាពខុសគ្នារវាង `=`, `==` និង `===` ដោយមិនមើលមេរៀន។

## ១៩. ឯកសារយោងផ្លូវការ

- [PHP Manual — Arithmetic Operators](https://www.php.net/manual/en/language.operators.arithmetic.php)
- [PHP Manual — Assignment Operators](https://www.php.net/manual/en/language.operators.assignment.php)
- [PHP Manual — Comparison Operators](https://www.php.net/manual/en/language.operators.comparison.php)
- [PHP Manual — Logical Operators](https://www.php.net/manual/en/language.operators.logical.php)
- [PHP Manual — Incrementing/Decrementing](https://www.php.net/manual/en/language.operators.increment.php)
- [PHP Manual — String Operators](https://www.php.net/manual/en/language.operators.string.php)
- [PHP Manual — Null Coalescing Operator](https://www.php.net/manual/en/language.operators.comparison.php)
- [PHP Manual — Operator Precedence](https://www.php.net/manual/en/language.operators.precedence.php)

---

**បញ្ចប់មេរៀនទី ៣។** មេរៀនបន្ទាប់៖ Conditional Statements — `if`, `else`, `elseif`, `switch` និង `match`។
