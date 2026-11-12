# មេរៀនទី ៦៖ PHP Functions (អនុគមន៍)

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> គោលបំណង៖ រៀនបង្កើត និងប្រើ Function ដើម្បីរៀបចំកូដឱ្យមានរបៀប កាត់បន្ថយកូដស្ទួន និងងាយស្រួលថែទាំ។

---

## ១. តើ Function គឺជាអ្វី?

**Function (អនុគមន៍)** គឺជាប្លុកកូដដែលមានឈ្មោះ ហើយអាចហៅឱ្យដំណើរការបាននៅពេលដែលយើងត្រូវការ។ Function អាចទទួលទិន្នន័យចូល (Parameters/Arguments) ហើយអាចបញ្ជូនលទ្ធផលត្រឡប់មកវិញ (Return Value)។

ឧទាហរណ៍៖ បើអ្នកត្រូវគណនាផលបូកពីរលេខជាច្រើនកន្លែង អ្នកអាចសរសេរ Function ម្តង ហើយហៅប្រើបានច្រើនដង។

អត្ថប្រយោជន៍៖
- កាត់បន្ថយកូដស្ទួន (Code Reuse)។
- ធ្វើឱ្យកូដងាយអាន និងរៀបចំ (Organization)។
- ងាយស្រួលសាកល្បង និងកែប្រែ (Testing and Maintenance)។
- បំបែកកម្មវិធីធំៗជាផ្នែកតូចៗ។

## ២. បង្កើត Function ដំបូង

ប្រើពាក្យគន្លឹះ `function` ដើម្បីប្រកាស Function។

```php
<?php
function sayHello() {
    echo "Hello, PHP!" . PHP_EOL;
}

sayHello();
sayHello();
```

លទ្ធផល៖

```text
Hello, PHP!
Hello, PHP!
```

ពន្យល់៖
- `function` ប្រកាស Function ថ្មី។
- `sayHello` ជាឈ្មោះ Function។
- `()` ជាកន្លែងដាក់ Parameters ប្រសិនបើមាន។
- `{}` កំណត់ប្លុកកូដរបស់ Function។
- `sayHello();` ហៅ Function ឱ្យដំណើរការ។

ការប្រកាស Function មិនមានន័យថាកូដខាងក្នុងដំណើរការភ្លាមៗទេ។ វាដំណើរការនៅពេលមានការហៅ Function។

## ៣. Parameters និង Arguments

**Parameter** គឺជាអថេរដែលកំណត់នៅក្នុងការប្រកាស Function។  
**Argument** គឺជាតម្លៃពិតដែលយើងផ្ញើទៅ Function នៅពេលហៅវា។

```php
<?php
function greet($name) {
    echo "សួស្តី " . $name . PHP_EOL;
}

greet("Dara");
greet("Sokha");
```

លទ្ធផល៖

```text
សួស្តី Dara
សួស្តី Sokha
```

នៅទីនេះ `$name` ជា Parameter ចំណែក `"Dara"` និង `"Sokha"` ជា Arguments។

### ៣.១ Function មាន Parameters ច្រើន

```php
<?php
function introduce($name, $age, $city) {
    echo "Name: {$name}" . PHP_EOL;
    echo "Age: {$age}" . PHP_EOL;
    echo "City: {$city}" . PHP_EOL;
}

introduce("Dara", 20, "Phnom Penh");
```

លទ្ធផល៖

```text
Name: Dara
Age: 20
City: Phnom Penh
```

លំដាប់ Arguments ត្រូវតែត្រូវគ្នានឹងលំដាប់ Parameters លុះត្រាតែអ្នកប្រើ Named Arguments ដែលនឹងរៀននៅខាងក្រោម។

## ៤. Return Value (តម្លៃត្រឡប់)

`return` បញ្ជូនលទ្ធផលពី Function ត្រឡប់ទៅកន្លែងដែលបានហៅវា។ វាខុសពី `echo` ដែលគ្រាន់តែបង្ហាញអត្ថបទ។

```php
<?php
function add($a, $b) {
    return $a + $b;
}

$result = add(10, 5);

echo $result . PHP_EOL;
```

លទ្ធផល៖

```text
15
```

ចំណុចសំខាន់៖
- `echo` បង្ហាញទិន្នន័យទៅ Output។
- `return` បញ្ជូនតម្លៃត្រឡប់ទៅអ្នកហៅ Function។
- បន្ទាប់ពី `return` ដំណើរការ Function នោះបញ្ចប់ភ្លាមៗ។

ឧទាហរណ៍៖

```php
<?php
function checkNumber($number) {
    if ($number < 0) {
        return "Negative";
    }

    return "Zero or Positive";
}

echo checkNumber(-4) . PHP_EOL;
echo checkNumber(8) . PHP_EOL;
```

លទ្ធផល៖

```text
Negative
Zero or Positive
```

## ៥. Type Declarations (កំណត់ប្រភេទទិន្នន័យ)

PHP អនុញ្ញាតឱ្យកំណត់ប្រភេទទិន្នន័យរបស់ Parameters និង Return Value ដើម្បីធ្វើឱ្យ Function ច្បាស់លាស់ជាងមុន។

```php
<?php
function multiply(int $a, int $b): int {
    return $a * $b;
}

echo multiply(4, 5) . PHP_EOL;
```

លទ្ធផល៖

```text
20
```

ពន្យល់៖
- `int $a` និង `int $b` បញ្ជាក់ថា Parameters រំពឹងទុកជាចំនួនគត់។
- `: int` បញ្ជាក់ថា Function ត្រូវបញ្ជូនចំនួនគត់ត្រឡប់។
- ប្រភេទទិន្នន័យផ្សេងទៀតមាន `string`, `float`, `bool`, `array`, `object` និង `mixed`។

### ៥.១ strict_types

តាមលំនាំដើម PHP អាចបម្លែងប្រភេទតម្លៃខ្លះនៅក្នុងករណីដែលអនុញ្ញាត។ អ្នកអាចបើក Strict Typing ដោយដាក់បន្ទាត់នេះនៅដើមឯកសារ៖

```php
<?php
declare(strict_types=1);

function square(int $number): int {
    return $number * $number;
}

echo square(5) . PHP_EOL;
```

លទ្ធផល៖

```text
25
```

ជាមួយ `strict_types=1` ការហៅ Function ដោយផ្ញើប្រភេទមិនត្រូវគ្នាអាចបង្កើត `TypeError`។ ត្រូវដាក់ `declare(strict_types=1);` នៅដើមឯកសារ បន្ទាប់ពីស្លាកបើក PHP។

## ៦. Default Parameters (តម្លៃលំនាំដើម)

អ្នកអាចកំណត់តម្លៃលំនាំដើមសម្រាប់ Parameter។ ប្រសិនបើអ្នកមិនផ្ញើ Argument នោះ PHP នឹងប្រើតម្លៃលំនាំដើម។

```php
<?php
function welcome($name = "អ្នកសិក្សា") {
    return "សូមស្វាគមន៍ " . $name;
}

echo welcome() . PHP_EOL;
echo welcome("Dara") . PHP_EOL;
```

លទ្ធផល៖

```text
សូមស្វាគមន៍ អ្នកសិក្សា
សូមស្វាគមន៍ Dara
```

ជាទូទៅ Parameters ដែលមាន Default Value គួរដាក់នៅខាងក្រោយ Parameters ដែលត្រូវការតម្លៃ។

## ៧. Named Arguments

ចាប់ពី PHP 8 អ្នកអាចបញ្ជាក់ឈ្មោះ Parameter នៅពេលហៅ Function។ វាជួយឱ្យកូដងាយអាន និងអាចប្តូរលំដាប់បាន។

```php
<?php
function createUser($name, $role, $active = true) {
    echo "Name: {$name}" . PHP_EOL;
    echo "Role: {$role}" . PHP_EOL;
    echo "Active: " . ($active ? "Yes" : "No") . PHP_EOL;
}

createUser(
    role: "Admin",
    name: "Dara",
    active: true
);
```

លទ្ធផល៖

```text
Name: Dara
Role: Admin
Active: Yes
```

ឈ្មោះដែលប្រើក្នុង Named Arguments ត្រូវតែត្រូវនឹងឈ្មោះ Parameter ក្នុង Function។

## ៨. Variadic Functions (ទទួល Arguments ច្រើន)

ប្រើ `...` មុន Parameter ដើម្បីឱ្យ Function ទទួល Arguments ច្រើនចំនួនមិនកំណត់។ តម្លៃទាំងនោះត្រូវបានប្រមូលជាទម្រង់ Array។

```php
<?php
function sumAll(int ...$numbers): int {
    $total = 0;

    foreach ($numbers as $number) {
        $total += $number;
    }

    return $total;
}

echo sumAll(1, 2, 3) . PHP_EOL;
echo sumAll(10, 20, 30, 40) . PHP_EOL;
```

លទ្ធផល៖

```text
6
100
```

## ៩. Variable Scope (វិសាលភាពរបស់អថេរ)

Scope គឺជាកន្លែងដែលអថេរអាចប្រើបាន។

### ៩.១ Local Variable

អថេរដែលប្រកាសនៅក្នុង Function ជាទូទៅអាចប្រើបានតែក្នុង Function នោះ។

```php
<?php
function showMessage() {
    $message = "Hello from function";
    echo $message . PHP_EOL;
}

showMessage();
```

### ៩.២ Global Variable

អថេរដែលប្រកាសនៅខាងក្រៅ Function មិនអាចយកមកប្រើក្នុង Function ដោយផ្ទាល់បានទេ។ អ្នកអាចប្រើ `global` ប៉ុន្តែសម្រាប់ការរចនាកូដល្អ គួរផ្ញើតម្លៃតាម Parameter ជាញឹកញាប់។

```php
<?php
$siteName = "My PHP Website";

function showSiteName() {
    global $siteName;
    echo $siteName . PHP_EOL;
}

showSiteName();
```

វិធីដែលងាយសាកល្បង និងមានភាពច្បាស់លាស់ជាង៖

```php
<?php
$siteName = "My PHP Website";

function printSiteName($name) {
    echo $name . PHP_EOL;
}

printSiteName($siteName);
```

### ៩.៣ Static Variable ក្នុង Function

អថេរ `static` ក្នុង Function រក្សាតម្លៃរបស់វាទុករវាងការហៅ Function ម្តងៗ។

```php
<?php
function counter() {
    static $count = 0;
    $count++;

    echo $count . PHP_EOL;
}

counter();
counter();
counter();
```

លទ្ធផល៖

```text
1
2
3
```

## ១០. Anonymous Functions និង Arrow Functions

### ១០.១ Anonymous Function

Anonymous Function គឺជា Function ដែលមិនមានឈ្មោះ ហើយអាចរក្សាទុកក្នុងអថេរ។

```php
<?php
$greet = function ($name) {
    return "Hello, " . $name;
};

echo $greet("Dara") . PHP_EOL;
```

លទ្ធផល៖

```text
Hello, Dara
```

បើចង់ប្រើអថេរខាងក្រៅក្នុង Anonymous Function អាចប្រើ `use`៖

```php
<?php
$prefix = "Welcome";

$greet = function ($name) use ($prefix) {
    return $prefix . ", " . $name;
};

echo $greet("Sokha") . PHP_EOL;
```

### ១០.២ Arrow Function

Arrow Function មានទម្រង់ខ្លីជាង ហើយចាប់យកអថេរខាងក្រៅដោយស្វ័យប្រវត្តិ តាមតម្លៃ។

```php
<?php
$numbers = [1, 2, 3, 4];

$squares = array_map(
    fn($number) => $number * $number,
    $numbers
);

print_r($squares);
```

លទ្ធផល៖

```text
Array
(
    [0] => 1
    [1] => 4
    [2] => 9
    [3] => 16
)
```

Arrow Function សមស្របសម្រាប់ Expression ខ្លីៗ។ ប្រសិនបើមានដំណើរការច្រើនជំហាន សូមប្រើ Function ធម្មតាដើម្បីឱ្យកូដងាយអាន។

## ១១. Recursive Function (អនុគមន៍ហៅខ្លួនឯង)

Recursive Function គឺ Function ដែលហៅខ្លួនឯង។ វាត្រូវមាន **Base Case** ដើម្បីបញ្ឈប់ការហៅបន្តបន្ទាប់។

ឧទាហរណ៍ គណនា Factorial៖

```php
<?php
function factorial(int $number): int {
    if ($number <= 1) {
        return 1; // Base Case
    }

    return $number * factorial($number - 1);
}

echo factorial(5) . PHP_EOL;
```

លទ្ធផល៖

```text
120
```

ពន្យល់៖ `5! = 5 × 4 × 3 × 2 × 1 = 120`។ Base Case ការពារកុំឱ្យ Function ហៅខ្លួនឯងដោយគ្មានទីបញ្ចប់។ សម្រាប់បញ្ហាដែលអាចប្រើ Loop បានសាមញ្ញជាង គួរពិចារណា Loop ផងដែរ។

## ១២. Built-in Functions (Function ដែល PHP មានស្រាប់)

PHP ផ្តល់ Function ជាច្រើនស្រាប់។ ឧទាហរណ៍៖

```php
<?php
$name = "  Learn PHP  ";

echo strlen($name) . PHP_EOL;          // ប្រវែង String ជា bytes
echo trim($name) . PHP_EOL;            // ដកចន្លោះខាងមុខ និងខាងក្រោយ
echo strtoupper("hello") . PHP_EOL;    // បម្លែងទៅអក្សរធំ
echo count([10, 20, 30]) . PHP_EOL;    // រាប់ចំនួនធាតុក្នុង Array
echo max(4, 9, 2) . PHP_EOL;           // រកតម្លៃធំបំផុត
```

លទ្ធផល៖

```text
13
Learn PHP
HELLO
3
9
```

ចំណាំ៖ `strlen()` រាប់ប្រវែងជា bytes មិនមែនចំនួនតួអក្សរដែលមើលឃើញទេ។ សម្រាប់អត្ថបទ UTF-8 ដូចជាភាសាខ្មែរ គួរប្រើ `mb_strlen($text, 'UTF-8')` ប្រសិនបើបានដំឡើង និងបើក PHP extension `mbstring`។

## ១៣. Mini Project៖ កម្មវិធីគណនាពិន្ទុសិស្ស

គោលដៅ៖ បង្កើត Function សម្រាប់គណនាមធ្យមភាគ និងកំណត់និទ្ទេស។

```php
<?php
declare(strict_types=1);

function calculateAverage(array $scores): float {
    if (count($scores) === 0) {
        return 0.0;
    }

    return array_sum($scores) / count($scores);
}

function getGrade(float $average): string {
    if ($average >= 90) {
        return "A";
    } elseif ($average >= 80) {
        return "B";
    } elseif ($average >= 70) {
        return "C";
    } elseif ($average >= 60) {
        return "D";
    }

    return "F";
}

function showStudentResult(string $name, array $scores): void {
    $average = calculateAverage($scores);
    $grade = getGrade($average);

    echo "Student: {$name}" . PHP_EOL;
    echo "Average: " . number_format($average, 2) . PHP_EOL;
    echo "Grade: {$grade}" . PHP_EOL;
}

$studentScores = [85, 90, 78, 92];

showStudentResult("Dara", $studentScores);
```

លទ្ធផល៖

```text
Student: Dara
Average: 86.25
Grade: B
```

ការពន្យល់៖
- `calculateAverage()` គណនាមធ្យមភាគ និងពិនិត្យករណី Array ទទេ។
- `getGrade()` បម្លែងពិន្ទុមធ្យមទៅជានិទ្ទេស។
- `showStudentResult()` ហៅ Function ពីរផ្សេងទៀត ហើយបង្ហាញលទ្ធផល។
- `: float`, `: string`, `: void` ជា Return Type។ `void` មានន័យថា Function មិនបញ្ជូនតម្លៃត្រឡប់ទេ។

## ១៤. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ភ្លេចហៅ Function**៖ ប្រកាស Function រួច ប៉ុន្តែមិនសរសេរ `functionName();`។
2. **ច្រឡំ Parameter និង Argument**៖ Parameter នៅក្នុងការប្រកាស; Argument ជាតម្លៃដែលផ្ញើពេលហៅ។
3. **ប្រើ echo ជំនួស return**៖ ប្រសិនបើចង់យកលទ្ធផលទៅគណនាបន្ត គួរប្រើ `return`។
4. **មិនត្រូវ Type Declaration**៖ ផ្ញើ String ទៅ Parameter ដែលកំណត់ជា `int` អាចបង្កើត TypeError ជាពិសេសពេល Strict Typing។
5. **ភ្លេច Base Case**៖ Recursive Function អាចបន្តហៅខ្លួនឯងដោយគ្មានទីបញ្ចប់។
6. **ប្រើ Global ច្រើនពេក**៖ ការផ្ញើទិន្នន័យតាម Parameters ជួយឱ្យ Function ងាយសាកល្បង និងយល់។

## ១៥. លំហាត់អនុវត្ត

សូមសាកល្បងដោយខ្លួនឯងមុនពេលមើលចម្លើយនៅផ្នែកខាងក្រោម។

### លំហាត់ទី ១៖ Hello Function
បង្កើត Function ឈ្មោះ `sayHello` ដែលបង្ហាញ `Hello PHP` ហើយហៅវា 3 ដង។

### លំហាត់ទី ២៖ បូកលេខ
បង្កើត Function `add` ដែលទទួលចំនួនពីរ និង Return ផលបូក។ សាកល្បងជាមួយ 12 និង 8។

### លំហាត់ទី ៣៖ ពិនិត្យលេខគូ
បង្កើត Function `isEven` ដែលទទួលចំនួនគត់ និង Return `true` ប្រសិនបើជាលេខគូ បើមិនដូច្នោះទេ Return `false`។

### លំហាត់ទី ៤៖ Default Parameter
បង្កើត Function `greet` ដែលទទួលឈ្មោះ ហើយមាន Default Value ជា `Guest`។

### លំហាត់ទី ៥៖ Variadic Function
បង្កើត Function `sumAll` ដែលអាចទទួលលេខបានច្រើន ហើយ Return ផលបូក។

### លំហាត់ទី ៦៖ Mini Project
បង្កើត Function `calculateArea` សម្រាប់គណនាផ្ទៃក្រឡាចតុកោណកែង ដោយទទួលប្រវែង និងទទឹង ហើយ Return លទ្ធផល។ កំណត់ Type Declaration ឱ្យបានត្រឹមត្រូវ។

## ១៦. សំណួរត្រួតពិនិត្យចំណេះដឹង

1. តើ Function មានអត្ថប្រយោជន៍អ្វី?
2. តើ Parameter និង Argument ខុសគ្នាដូចម្តេច?
3. តើ `echo` និង `return` ខុសគ្នាដូចម្តេច?
4. តើ Return Type `: void` មានន័យដូចម្តេច?
5. តើ Default Parameter ប្រើសម្រាប់អ្វី?
6. តើ Recursive Function ត្រូវការអ្វីដើម្បីបញ្ឈប់ការហៅខ្លួនឯង?
7. តើ Arrow Function ខុសពី Anonymous Function ធម្មតាយ៉ាងដូចម្តេច?

---

## ១៧. ចម្លើយគំរូ (មើលក្រោយពេលសាកល្បងរួច)

### ចម្លើយលំហាត់ទី ១

```php
<?php
function sayHello(): void {
    echo "Hello PHP" . PHP_EOL;
}

sayHello();
sayHello();
sayHello();
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
function add(int $a, int $b): int {
    return $a + $b;
}

echo add(12, 8) . PHP_EOL;
```

លទ្ធផល៖ `20`

### ចម្លើយលំហាត់ទី ៣

```php
<?php
function isEven(int $number): bool {
    return $number % 2 === 0;
}

var_dump(isEven(4));
var_dump(isEven(7));
```

លទ្ធផល៖

```text
bool(true)
bool(false)
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
function greet(string $name = "Guest"): string {
    return "Hello, " . $name;
}

echo greet() . PHP_EOL;
echo greet("Dara") . PHP_EOL;
```

### ចម្លើយលំហាត់ទី ៥

```php
<?php
function sumAll(int ...$numbers): int {
    return array_sum($numbers);
}

echo sumAll(2, 4, 6, 8) . PHP_EOL;
```

លទ្ធផល៖ `20`

### ចម្លើយលំហាត់ទី ៦

```php
<?php
declare(strict_types=1);

function calculateArea(float $length, float $width): float {
    return $length * $width;
}

echo calculateArea(10.5, 4.0) . PHP_EOL;
```

លទ្ធផល៖ `42`

### ចម្លើយសំណួរត្រួតពិនិត្យ

1. ជួយរៀបចំកូដ កាត់បន្ថយកូដស្ទួន និងអាចប្រើឡើងវិញ។
2. Parameter ជាអថេរក្នុងការប្រកាស Function; Argument ជាតម្លៃដែលផ្ញើពេលហៅ Function។
3. `echo` បង្ហាញលទ្ធផល; `return` បញ្ជូនតម្លៃទៅកន្លែងហៅ។
4. Function មិនបញ្ជូនតម្លៃត្រឡប់។
5. ដើម្បីប្រើតម្លៃជំនួស នៅពេលមិនបានផ្ញើ Argument។
6. Base Case។
7. Arrow Function មានទម្រង់ខ្លី និងចាប់យកអថេរខាងក្រៅដោយស្វ័យប្រវត្តិតាមតម្លៃ។

## ១៨. របៀបដំណើរការកូដ

1. បង្កើតឯកសារ `lesson06.php`។
2. ចម្លងកូដឧទាហរណ៍ទៅក្នុងឯកសារ។
3. បើក Terminal ឬ Command Prompt នៅទីតាំងឯកសារ។
4. ដំណើរការ៖

```bash
php lesson06.php
```

ត្រូវប្រាកដថា PHP បានដំឡើង ហើយពាក្យបញ្ជា `php` អាចប្រើបានក្នុង Terminal។

## សង្ខេបមេរៀន

ក្នុងមេរៀននេះ អ្នកបានរៀន៖
- ប្រកាស និងហៅ Function។
- Parameters, Arguments និង Return Values។
- Type Declarations និង Strict Typing។
- Default និង Named Arguments។
- Variadic Functions។
- Variable Scope និង Static Variables។
- Anonymous, Arrow និង Recursive Functions។
- Built-in Functions និង Mini Project។

**ការងារមុនទៅមេរៀនបន្ទាប់៖** អនុវត្តលំហាត់ទាំង ៦ ហើយសាកល្បងបង្កើត Function ផ្ទាល់ខ្លួនយ៉ាងតិច ៣។
