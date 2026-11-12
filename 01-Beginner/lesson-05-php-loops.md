# មេរៀនទី ៥៖ PHP Loops (រង្វិលជុំ)

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> គោលបំណង៖ រៀនប្រើរង្វិលជុំ ដើម្បីអនុវត្តកូដដដែលៗដោយមិនចាំបាច់សរសេរឡើងវិញច្រើនដង។

---

## ១. តើ Loop គឺជាអ្វី?

**Loop (រង្វិលជុំ)** គឺជារចនាសម្ព័ន្ធដែលអនុញ្ញាតឱ្យកម្មវិធីអនុវត្តកូដមួយ ឬច្រើនបន្ទាត់ម្តងហើយម្តងទៀត ដរាបណាលក្ខខណ្ឌនៅតែត្រឹមត្រូវ ឬរហូតដល់ទិន្នន័យត្រូវបានអានអស់។

ឧទាហរណ៍៖ ប្រសិនបើអ្នកចង់បង្ហាញលេខពី 1 ដល់ 100 អ្នកមិនចាំបាច់សរសេរ `echo` 100 ដងទេ។ អ្នកអាចប្រើ Loop មួយ។

PHP មាន Loop សំខាន់ៗ៖

1. `while`
2. `do...while`
3. `for`
4. `foreach`

ក៏មាន `break` និង `continue` សម្រាប់គ្រប់គ្រងដំណើរការ Loop ផងដែរ។

## ២. while Loop

`while` ពិនិត្យលក្ខខណ្ឌជាមុន។ ប្រសិនបើលក្ខខណ្ឌមានតម្លៃ `true` វានឹងដំណើរការកូដក្នុង Loop។ វាបន្តរហូតដល់លក្ខខណ្ឌក្លាយជា `false`។

ទម្រង់៖

```php
while (condition) {
    // code to repeat
}
```

ឧទាហរណ៍៖

```php
<?php
$count = 1;

while ($count <= 5) {
    echo $count . PHP_EOL;
    $count++;
}
```

លទ្ធផល៖

```text
1
2
3
4
5
```

ពន្យល់៖
- `$count = 1` កំណត់តម្លៃចាប់ផ្តើម។
- `$count <= 5` ជាលក្ខខណ្ឌបន្ត។
- `echo` បង្ហាញលេខបច្ចុប្បន្ន។
- `$count++` បន្ថែមតម្លៃ 1 រាល់ជុំ។

**ចំណាំសំខាន់៖** កុំភ្លេចធ្វើឱ្យលក្ខខណ្ឌអាចក្លាយជា `false`។ បើមិនដូច្នោះទេ អ្នកអាចបង្កើត Infinite Loop (រង្វិលជុំមិនចេះចប់)។

## ៣. do...while Loop

`do...while` ដំណើរការកូដក្នុងប្លុកម្តងសិន រួចទើបពិនិត្យលក្ខខណ្ឌ។ ដូច្នេះវាធានាថាកូដក្នុង Loop ដំណើរការយ៉ាងហោចណាស់ម្តង។

ទម្រង់៖

```php
do {
    // code to repeat
} while (condition);
```

ឧទាហរណ៍៖

```php
<?php
$number = 1;

do {
    echo "Number: " . $number . PHP_EOL;
    $number++;
} while ($number <= 3);
```

លទ្ធផល៖

```text
Number: 1
Number: 2
Number: 3
```

ប្រៀបធៀប៖ ប្រសិនបើលក្ខខណ្ឌដំបូងជា `false` នោះ `while` អាចមិនដំណើរការសោះ ប៉ុន្តែ `do...while` នឹងដំណើរការម្តងជាមុន។

```php
<?php
$value = 10;

while ($value < 5) {
    echo "while runs" . PHP_EOL;
}

do {
    echo "do...while runs" . PHP_EOL;
} while ($value < 5);
```

លទ្ធផល៖

```text
do...while runs
```

## ៤. for Loop

ប្រើ `for` នៅពេលអ្នកដឹងច្បាស់ ឬប្រហែលដឹងចំនួនជុំដែលត្រូវដំណើរការ។

ទម្រង់៖

```php
for (initialization; condition; update) {
    // code to repeat
}
```

មានបីផ្នែក៖
1. **Initialization**៖ កំណត់តម្លៃចាប់ផ្តើម។
2. **Condition**៖ ពិនិត្យថាតើត្រូវបន្តឬអត់។
3. **Update**៖ កែប្រែតម្លៃបន្ទាប់ពីដំណើរការមួយជុំ។

ឧទាហរណ៍៖

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    echo "Round " . $i . PHP_EOL;
}
```

លទ្ធផល៖

```text
Round 1
Round 2
Round 3
Round 4
Round 5
```

រាប់ថយក្រោយ៖

```php
<?php
for ($i = 5; $i >= 1; $i--) {
    echo $i . PHP_EOL;
}
echo "Finished!" . PHP_EOL;
```

លទ្ធផល៖

```text
5
4
3
2
1
Finished!
```

## ៥. foreach Loop

`foreach` ត្រូវបានប្រើជាពិសេសសម្រាប់អានតម្លៃក្នុង Array។ វាងាយស្រួល ព្រោះមិនចាំបាច់គ្រប់គ្រង index ដោយដៃ។

ទម្រង់៖

```php
foreach ($array as $value) {
    // use value
}
```

ឧទាហរណ៍៖

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

foreach ($fruits as $fruit) {
    echo $fruit . PHP_EOL;
}
```

លទ្ធផល៖

```text
Apple
Banana
Orange
```

### ៥.១ អានទាំង Key និង Value

សម្រាប់ Associative Array អ្នកអាចប្រើ `key => value`៖

```php
<?php
$student = [
    "name" => "Dara",
    "age" => 18,
    "major" => "PHP"
];

foreach ($student as $key => $value) {
    echo $key . ": " . $value . PHP_EOL;
}
```

លទ្ធផល៖

```text
name: Dara
age: 18
major: PHP
```

### ៥.២ កែប្រែតម្លៃ Array ដោយ Reference

បើអ្នកចង់កែប្រែតម្លៃដើមរបស់ Array ក្នុង Loop អ្នកអាចប្រើ `&`៖

```php
<?php
$numbers = [1, 2, 3];

foreach ($numbers as &$number) {
    $number *= 2;
}
unset($number); // ផ្តាច់ reference បន្ទាប់ពី Loop

print_r($numbers);
```

លទ្ធផល៖

```text
Array
(
    [0] => 2
    [1] => 4
    [2] => 6
)
```

ប្រើ reference តែនៅពេលចាំបាច់។ ការប្រើ `unset($number)` ក្រោយ Loop ជួយជៀសវាងការប្រើ reference ចាស់ដោយចៃដន្យ។

## ៦. break

`break` បញ្ឈប់ Loop ភ្លាមៗ ហើយចេញពី Loop នោះ។

```php
<?php
for ($i = 1; $i <= 10; $i++) {
    if ($i === 6) {
        break;
    }

    echo $i . PHP_EOL;
}
```

លទ្ធផល៖

```text
1
2
3
4
5
```

## ៧. continue

`continue` រំលងកូដដែលនៅសល់ក្នុងជុំបច្ចុប្បន្ន ហើយទៅជុំបន្ទាប់។

ឧទាហរណ៍ រំលងលេខ 3៖

```php
<?php
for ($i = 1; $i <= 5; $i++) {
    if ($i === 3) {
        continue;
    }

    echo $i . PHP_EOL;
}
```

លទ្ធផល៖

```text
1
2
4
5
```

**ភាពខុសគ្នា៖**
- `break`៖ បញ្ឈប់ Loop ទាំងមូល។
- `continue`៖ រំលងតែជុំបច្ចុប្បន្ន។

## ៨. Nested Loop (Loop ក្នុង Loop)

អ្នកអាចដាក់ Loop មួយនៅក្នុង Loop មួយទៀត។ វាមានប្រយោជន៍សម្រាប់បង្ហាញតារាង ឬទិន្នន័យពីរវិមាត្រ។

```php
<?php
for ($row = 1; $row <= 3; $row++) {
    for ($column = 1; $column <= 3; $column++) {
        echo "* ";
    }

    echo PHP_EOL;
}
```

លទ្ធផល៖

```text
* * *
* * *
* * *
```

Loop ខាងក្រៅគ្រប់គ្រងជួរដេក (row) ហើយ Loop ខាងក្នុងគ្រប់គ្រងជួរឈរ (column)។

## ៩. Mini Project៖ តារាងគុណ (Multiplication Table)

គោលដៅ៖ បង្កើតកម្មវិធីបង្ហាញតារាងគុណពី 1 ដល់ 5។

```php
<?php
for ($number = 1; $number <= 5; $number++) {
    echo "Multiplication table of {$number}" . PHP_EOL;

    for ($i = 1; $i <= 10; $i++) {
        $result = $number * $i;
        echo "{$number} x {$i} = {$result}" . PHP_EOL;
    }

    echo "--------------------" . PHP_EOL;
}
```

ពន្យល់៖
- Loop ខាងក្រៅកំណត់លេខតារាងគុណពី 1 ដល់ 5។
- Loop ខាងក្នុងគណនាពី 1 ដល់ 10។
- `$result` រក្សាទុកលទ្ធផលគុណ។
- បន្ទាត់សញ្ញា `--------------------` ជួយបំបែកតារាងនីមួយៗ។

សាកល្បងប្តូរ `$number <= 5` ទៅជា `$number <= 12` ដើម្បីបង្កើតតារាងគុណ 1 ដល់ 12។

## ១០. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ភ្លេច Update Counter**៖ ក្នុង `while` ភ្លេចសរសេរ `$i++` ធ្វើឱ្យ Loop អាចមិនចេះចប់។
2. **លក្ខខណ្ឌខុស**៖ ប្រើ `$i < 5` ជំនួស `$i <= 5` ធ្វើឱ្យបាត់លេខ 5។
3. **ប្រើសញ្ញា Semicolon ខុសទីតាំង**៖ `do...while` ត្រូវមាន `;` ក្រោយ `while (...)`។
4. **ប្រើ Loop មិនសមស្រប**៖ ប្រសិនបើកំពុងអាន Array ជាទូទៅ `foreach` ងាយស្រួលជាងការគ្រប់គ្រង index ដោយដៃ។
5. **ច្រឡំ break និង continue**៖ ចងចាំថា break ចេញពី Loop ចំណែក continue រំលងជុំបច្ចុប្បន្ន។

## ១១. លំហាត់អនុវត្ត

សូមសរសេរកូដដោយខ្លួនឯង មុនពេលមើលចម្លើយនៅផ្នែកខាងក្រោម។

### លំហាត់ទី ១៖ រាប់លេខ
ប្រើ `for` បង្ហាញលេខពី 1 ដល់ 20។

### លំហាត់ទី ២៖ លេខគូ
ប្រើ `while` បង្ហាញលេខគូពី 2 ដល់ 20។

### លំហាត់ទី ៣៖ បូកលេខ
ប្រើ Loop ដើម្បីគណនាផលបូកលេខពី 1 ដល់ 100។ លទ្ធផលគួរតែជា `5050`។

### លំហាត់ទី ៤៖ អាន Array
បង្កើត Array ឈ្មោះ `$languages` ដែលមាន `PHP`, `JavaScript`, `Python` ហើយប្រើ `foreach` បង្ហាញភាសានីមួយៗ។

### លំហាត់ទី ៥៖ រំលងលេខ
ប្រើ `for` បង្ហាញលេខពី 1 ដល់ 10 ប៉ុន្តែរំលងលេខដែលចែកដាច់នឹង 3។

### លំហាត់ទី ៦៖ Mini Project
បង្កើតកម្មវិធីបង្ហាញផ្កាយជាជ្រុងការ៉េ 5 ជួរដេក និង 5 ជួរឈរ ដោយប្រើ Nested Loop។

## ១២. សំណួរត្រួតពិនិត្យចំណេះដឹង

1. តើ Loop មានប្រយោជន៍អ្វី?
2. តើ `while` និង `do...while` ខុសគ្នាដូចម្តេច?
3. តើផ្នែកទាំងបីក្នុង `for` មានអ្វីខ្លះ?
4. តើ Loop មួយណាដែលសមស្របសម្រាប់អាន Array?
5. តើ `break` និង `continue` ខុសគ្នាយ៉ាងដូចម្តេច?
6. តើ Infinite Loop កើតឡើងដោយសារអ្វី?

---

## ១៣. ចម្លើយគំរូ (មើលក្រោយពេលសាកល្បងរួច)

### ចម្លើយលំហាត់ទី ១

```php
<?php
for ($i = 1; $i <= 20; $i++) {
    echo $i . PHP_EOL;
}
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
$i = 2;

while ($i <= 20) {
    echo $i . PHP_EOL;
    $i += 2;
}
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
$sum = 0;

for ($i = 1; $i <= 100; $i++) {
    $sum += $i;
}

echo "Sum = " . $sum . PHP_EOL;
```

លទ្ធផល៖ `Sum = 5050`

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$languages = ["PHP", "JavaScript", "Python"];

foreach ($languages as $language) {
    echo $language . PHP_EOL;
}
```

### ចម្លើយលំហាត់ទី ៥

```php
<?php
for ($i = 1; $i <= 10; $i++) {
    if ($i % 3 === 0) {
        continue;
    }

    echo $i . PHP_EOL;
}
```

### ចម្លើយលំហាត់ទី ៦

```php
<?php
for ($row = 1; $row <= 5; $row++) {
    for ($column = 1; $column <= 5; $column++) {
        echo "* ";
    }

    echo PHP_EOL;
}
```

លទ្ធផល៖

```text
* * * * *
* * * * *
* * * * *
* * * * *
* * * * *
```

### ចម្លើយសំណួរត្រួតពិនិត្យ

1. Loop ជួយដំណើរការកូដដដែលៗ ដោយកាត់បន្ថយការសរសេរកូដស្ទួន។
2. `while` ពិនិត្យលក្ខខណ្ឌមុន; `do...while` ដំណើរការម្តងសិន រួចពិនិត្យលក្ខខណ្ឌ។
3. Initialization, Condition និង Update។
4. `foreach`។
5. `break` ចេញពី Loop ទាំងមូល; `continue` រំលងតែជុំបច្ចុប្បន្ន។
6. វាអាចកើតឡើងនៅពេលលក្ខខណ្ឌមិនដែលក្លាយជា `false` ឧទាហរណ៍ ភ្លេចកែប្រែ Counter។

## ១៤. របៀបដំណើរការកូដ

1. បង្កើតឯកសារ `lesson05.php`។
2. ចម្លងកូដឧទាហរណ៍ទៅក្នុងឯកសារ។
3. បើក Terminal ឬ Command Prompt នៅទីតាំងឯកសារ។
4. ដំណើរការ៖

```bash
php lesson05.php
```

បើអ្នកប្រើ VS Code អ្នកអាចបើក Terminal តាមម៉ឺនុយ **Terminal → New Terminal**។ ត្រូវប្រាកដថា PHP បានដំឡើង និងអាចប្រើពាក្យបញ្ជា `php` បាន។

## សង្ខេបមេរៀន

ក្នុងមេរៀននេះ អ្នកបានរៀន៖
- `while`, `do...while`, `for`, `foreach`
- អាន Array ជាមួយ `foreach`
- ប្រើ `break` និង `continue`
- Nested Loop
- បង្កើត Multiplication Table

**ការងារមុនទៅមេរៀនបន្ទាប់៖** សាកល្បងលំហាត់ទាំង ៦ ហើយព្យាយាមសរសេរកូដឡើងវិញដោយមិនមើលឧទាហរណ៍។
