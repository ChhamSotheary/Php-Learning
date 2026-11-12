# មេរៀនទី ២៖ Variables និង Data Types ក្នុង PHP

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> ឯកសារ៖ `01-Beginner/lesson-02-variables-and-data-types.md`

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកអាច៖

- ពន្យល់ថា Variable ជាអ្វី និងប្រើវាសម្រាប់អ្វី។
- បង្កើត និងដាក់តម្លៃឱ្យ Variables ក្នុង PHP។
- ស្គាល់ច្បាប់ដាក់ឈ្មោះ Variable។
- ស្គាល់ប្រភេទទិន្នន័យសំខាន់ៗរបស់ PHP។
- ប្រើ `var_dump()` និង `gettype()` ដើម្បីពិនិត្យប្រភេទទិន្នន័យ។
- យល់ពីការបម្លែងប្រភេទទិន្នន័យ (Type Casting) និងការបម្លែងដោយស្វ័យប្រវត្តិ (Type Juggling)។
- បង្ហាញតម្លៃ Variable ក្នុងអេក្រង់ និងភ្ជាប់អត្ថបទជាមួយ Variable។

## ២. Variable គឺជាអ្វី?

**Variable (អថេរ)** គឺជាឈ្មោះដែលយើងប្រើសម្រាប់យោងទៅកាន់តម្លៃទិន្នន័យមួយនៅក្នុងកម្មវិធី។ អ្នកអាចគិតថាវាដូចជាប្រអប់មួយដែលមានស្លាកឈ្មោះ ហើយអាចផ្ទុកតម្លៃដូចជា ឈ្មោះ អាយុ តម្លៃទំនិញ ឬស្ថានភាព។

ឧទាហរណ៍៖

```php
<?php
$name = "Dara";
$age = 20;
$price = 12.50;

echo $name;
echo $age;
echo $price;
```

លទ្ធផល (តម្លៃអាចបង្ហាញជាប់គ្នា ព្រោះយើងមិនទាន់ដាក់ចន្លោះ ឬបន្ទាត់ថ្មី)៖

```text
Dara2012.5
```

ការពន្យល់៖

- `$name` ជា Variable ដែលផ្ទុកអត្ថបទ `"Dara"`។
- `$age` ផ្ទុកចំនួនគត់ `20`។
- `$price` ផ្ទុកចំនួនទសភាគ `12.50`។
- សញ្ញា `=` ជា **Assignment Operator** មានន័យថា ដាក់តម្លៃនៅខាងស្តាំទៅក្នុង Variable ខាងឆ្វេង។

ចំណាំ៖ PHP ប្រើសញ្ញា `$` នៅខាងមុខឈ្មោះ Variable ជានិច្ច។

## ៣. ច្បាប់ដាក់ឈ្មោះ Variable

ច្បាប់សំខាន់ៗ៖

1. ឈ្មោះ Variable ត្រូវចាប់ផ្តើមដោយសញ្ញា `$`។
2. បន្ទាប់ពី `$` ត្រូវចាប់ផ្តើមដោយអក្សរ ឬ underscore `_`។
3. មិនអាចចាប់ផ្តើមដោយលេខបានទេ។
4. អាចមានអក្សរ លេខ និង underscore នៅខាងក្នុងឈ្មោះ។
5. ឈ្មោះ Variable ប្រកាន់អក្សរតូច/ធំ (Case-sensitive)។ ឧទាហរណ៍ `$name` និង `$Name` ជា Variables ពីរផ្សេងគ្នា។
6. មិនត្រូវប្រើចន្លោះក្នុងឈ្មោះ Variable ទេ។

ឧទាហរណ៍ត្រឹមត្រូវ៖

```php
<?php
$name = "Sokha";
$user_name = "Dara";
$age2 = 18;
$_score = 95;

echo $name;
echo $user_name;
echo $age2;
echo $_score;
```

ឧទាហរណ៍មិនត្រឹមត្រូវ៖

```php
<?php
// $2name = "Dara";   // មិនអាចចាប់ផ្តើមដោយលេខ
// $user name = "Dara"; // មិនអាចមានចន្លោះ
```

កុំយកឧទាហរណ៍មិនត្រឹមត្រូវទៅដំណើរការជាកូដពិត ព្រោះវានឹងបង្កើត Syntax Error។

## ៤. ការប្តូរតម្លៃ Variable

តម្លៃរបស់ Variable អាចផ្លាស់ប្តូរបាន៖

```php
<?php
$score = 50;
echo $score . PHP_EOL;

$score = 80;
echo $score . PHP_EOL;
```

លទ្ធផល៖

```text
50
80
```

`PHP_EOL` គឺជា constant ដែលតំណាងឱ្យការបញ្ចប់បន្ទាត់តាមប្រព័ន្ធប្រតិបត្តិការ។ វាមានប្រយោជន៍ពេលបង្ហាញលទ្ធផលក្នុង Terminal។

## ៥. Data Types ក្នុង PHP

**Data Type (ប្រភេទទិន្នន័យ)** ប្រាប់ថាតម្លៃមួយមានលក្ខណៈបែបណា។ PHP មានប្រភេទទិន្នន័យសំខាន់ៗដូចខាងក្រោម៖

### ៥.១ String — អត្ថបទ

String គឺជាអក្សរ ឬអត្ថបទ ដែលសរសេរនៅក្នុងសញ្ញាដកស្រង់តែមួយ ឬពីរ។

```php
<?php
$firstName = "Sotheary";
$country = 'Cambodia';

echo $firstName . PHP_EOL;
echo $country . PHP_EOL;
```

លទ្ធផល៖

```text
Sotheary
Cambodia
```

### ៥.២ Integer — ចំនួនគត់

Integer គឺជាចំនួនគ្មានចំណុចទសភាគ។

```php
<?php
$age = 18;
$quantity = 100;

var_dump($age);
var_dump($quantity);
```

លទ្ធផល៖

```text
int(18)
int(100)
```

### ៥.៣ Float — ចំនួនទសភាគ

Float (ឬ floating-point number) ប្រើសម្រាប់ចំនួនដែលមានចំណុចទសភាគ។

```php
<?php
$price = 19.99;
$temperature = 36.5;

var_dump($price);
var_dump($temperature);
```

លទ្ធផល៖

```text
float(19.99)
float(36.5)
```

ចំណាំ៖ ការគណនាលេខទសភាគក្នុងកុំព្យូទ័រអាចមានភាពខុសគ្នាតូចៗដោយសាររបៀបតំណាងលេខ។ កុំប្រើការប្រៀបធៀប Float ដោយស្មើពិតប្រាកដសម្រាប់ការគណនាហិរញ្ញវត្ថុសំខាន់ៗ។

### ៥.៤ Boolean — តម្លៃពិត ឬមិនពិត

Boolean មានតែពីរតម្លៃ៖ `true` និង `false`។

```php
<?php
$isStudent = true;
$isLoggedIn = false;

var_dump($isStudent);
var_dump($isLoggedIn);
```

លទ្ធផល៖

```text
bool(true)
bool(false)
```

### ៥.៥ Array — សំណុំទិន្នន័យ

Array អាចផ្ទុកតម្លៃច្រើននៅក្នុង Variable តែមួយ។

```php
<?php
$fruits = ["Apple", "Banana", "Mango"];

echo $fruits[0] . PHP_EOL;
echo $fruits[1] . PHP_EOL;
echo $fruits[2] . PHP_EOL;
```

លទ្ធផល៖

```text
Apple
Banana
Mango
```

ចំណាំ៖ Index របស់ Array ប្រភេទនេះចាប់ផ្តើមពី `0` មិនមែន `1` ទេ។

### ៥.៦ NULL — គ្មានតម្លៃ

`null` មានន័យថា Variable មិនមានតម្លៃទេ។

```php
<?php
$middleName = null;

var_dump($middleName);
```

លទ្ធផល៖

```text
NULL
```

### ៥.៧ Object — វត្ថុ

Object ជាប្រភេទទិន្នន័យដែលបង្កើតពី Class។ យើងនឹងសិក្សា Class និង Object ឱ្យស៊ីជម្រៅនៅក្នុងមេរៀន OOP ពេលក្រោយ។ ឧទាហរណ៍ខ្លី៖

```php
<?php
class Student
{
    public string $name = "Dara";
}

$student = new Student();

echo $student->name;
```

លទ្ធផល៖

```text
Dara
```

## ៦. ពិនិត្យប្រភេទទិន្នន័យ

PHP មាន Function ពីរដែលអ្នកគួរចេះ៖

- `var_dump($value)` បង្ហាញប្រភេទ និងតម្លៃ។
- `gettype($value)` ត្រឡប់ឈ្មោះប្រភេទទិន្នន័យជាអត្ថបទ។

```php
<?php
$name = "Dara";
$age = 21;
$height = 1.72;
$isActive = true;
$items = ["Book", "Pen"];
$address = null;

var_dump($name);
echo gettype($age) . PHP_EOL;
echo gettype($height) . PHP_EOL;
echo gettype($isActive) . PHP_EOL;
echo gettype($items) . PHP_EOL;
echo gettype($address) . PHP_EOL;
```

លទ្ធផលប្រហាក់ប្រហែល៖

```text
string(4) "Dara"
integer
double
boolean
array
NULL
```

ចំណាំ៖ `gettype()` អាចបង្ហាញ `double` សម្រាប់ Float។ ក្នុងការសរសេរកូដទូទៅ អ្នកអាចប្រើ `is_string()`, `is_int()`, `is_float()`, `is_bool()`, `is_array()` និង `is_null()` ដើម្បីពិនិត្យប្រភេទបានច្បាស់ជាង។

ឧទាហរណ៍៖

```php
<?php
$age = 18;

if (is_int($age)) {
    echo "Age is an integer.";
}
```

លទ្ធផល៖

```text
Age is an integer.
```

## ៧. ការភ្ជាប់អត្ថបទជាមួយ Variable

PHP អាចបង្ហាញ Variable ក្នុង String បានពីររបៀបសំខាន់ៗ។

### របៀបទី១៖ ប្រើ Dot Operator

សញ្ញា `.` ប្រើសម្រាប់ភ្ជាប់ String ជាមួយតម្លៃផ្សេងៗ។

```php
<?php
$name = "Dara";
$age = 20;

echo "My name is " . $name . " and I am " . $age . " years old.";
```

លទ្ធផល៖

```text
My name is Dara and I am 20 years old.
```

### របៀបទី២៖ ប្រើ Double Quotes

នៅក្នុង Double Quotes (`"..."`) PHP អាចបកស្រាយ Variable បាន។

```php
<?php
$name = "Dara";
$age = 20;

echo "My name is $name and I am $age years old.";
```

លទ្ធផលដូចគ្នា៖

```text
My name is Dara and I am 20 years old.
```

ចំណាំ៖ Single Quotes (`'...'`) មិនបកស្រាយ Variable ដូច Double Quotes ទេ។

```php
<?php
$name = "Dara";

echo 'Hello $name';
echo PHP_EOL;
echo "Hello $name";
```

លទ្ធផល៖

```text
Hello $name
Hello Dara
```

## ៨. Type Casting និង Type Juggling

**Type Casting** គឺការបម្លែងតម្លៃទៅជាប្រភេទទិន្នន័យដែលយើងកំណត់ដោយផ្ទាល់។

```php
<?php
$numberText = "25";

$number = (int) $numberText;

var_dump($number);
```

លទ្ធផល៖

```text
int(25)
```

ឧទាហរណ៍បន្ថែម៖

```php
<?php
$value = 12;

$textValue = (string) $value;
$floatValue = (float) $value;
$boolValue = (bool) $value;

var_dump($textValue);
var_dump($floatValue);
var_dump($boolValue);
```

**Type Juggling** គឺការបម្លែងប្រភេទដោយស្វ័យប្រវត្តិរបស់ PHP ក្នុងបរិបទខ្លះៗ។ ឧទាហរណ៍៖

```php
<?php
$number = "10";
$result = $number + 5;

var_dump($result);
```

លទ្ធផល៖

```text
int(15)
```

PHP បានបកស្រាយ String `"10"` ជាលេខសម្រាប់ការបូក។ ទោះយ៉ាងណា កុំអាស្រ័យលើការបម្លែងស្វ័យប្រវត្តិគ្រប់ករណី។ ពិនិត្យទិន្នន័យ និងបម្លែងប្រភេទឱ្យច្បាស់ នៅពេលត្រូវការ។

## ៩. ការអនុវត្តជាក់ស្តែង៖ បង្កើតកម្មវិធីណែនាំខ្លួន

បង្កើតឯកសារ `profile.php` ហើយសរសេរកូដ៖

```php
<?php
$name = "Sotheary";
$age = 18;
$country = "Cambodia";
$favoriteLanguage = "PHP";
$isLearning = true;

echo "===== MY PROFILE =====" . PHP_EOL;
echo "Name: " . $name . PHP_EOL;
echo "Age: " . $age . PHP_EOL;
echo "Country: " . $country . PHP_EOL;
echo "Favorite language: " . $favoriteLanguage . PHP_EOL;
echo "Currently learning PHP: " . ($isLearning ? "Yes" : "No") . PHP_EOL;
```

លទ្ធផល៖

```text
===== MY PROFILE =====
Name: Sotheary
Age: 18
Country: Cambodia
Favorite language: PHP
Currently learning PHP: Yes
```

ការពន្យល់៖

- បន្ទាត់នីមួយៗបង្កើត Variable ដើម្បីរក្សាទុកព័ត៌មាន។
- `echo` បង្ហាញអត្ថបទ និងតម្លៃ Variable។
- `.` ភ្ជាប់អត្ថបទជាមួយ Variable។
- `PHP_EOL` បំបែកបន្ទាត់។
- `$isLearning ? "Yes" : "No"` គឺជា Ternary Operator ដែលជ្រើសរើសអត្ថបទតាមតម្លៃ Boolean។ យើងនឹងរៀនលម្អិតនៅមេរៀន Conditional Statements។

អ្នកអាចប្តូរតម្លៃ Variables ទៅជាព័ត៌មានផ្ទាល់ខ្លួនសម្រាប់សាកល្បងបាន។

## ១០. របៀបដំណើរការកូដ

1. បើក VS Code ឬ Text Editor ដែលអ្នកប្រើ។
2. បង្កើត Folder សម្រាប់មេរៀន PHP។
3. បង្កើតឯកសារ `profile.php`។
4. ចម្លងកូដខាងលើទៅក្នុងឯកសារ ហើយ Save។
5. បើក Terminal នៅក្នុង Folder នោះ។
6. ដំណើរការ៖

```bash
php profile.php
```

បើ Terminal បង្ហាញព័ត៌មាន Profile មានន័យថាកម្មវិធីដំណើរការបានត្រឹមត្រូវ។

បើឃើញសារ `php is not recognized` ឬ `command not found` នោះ PHP មិនទាន់បានដំឡើង ឬមិនទាន់បានកំណត់ PATH ត្រឹមត្រូវទេ។ សូមត្រឡប់ទៅការរៀបចំបរិស្ថានក្នុងមេរៀនទី១។

## ១១. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ភ្លេចសញ្ញា `$`**៖ សរសេរ `name = "Dara";` ជំនួស `$name = "Dara";`។
2. **ប្រើឈ្មោះខុសអក្សរតូច/ធំ**៖ កំណត់ `$userName` ប៉ុន្តែទៅប្រើ `$username`។
3. **ភ្លេចដាក់សញ្ញា ;** នៅចុង Statement។
4. **ច្រឡំ String និង Variable**៖ `'$name'` បង្ហាញអក្សរ `$name` តាមតម្លៃ Literal ក្នុង Single Quotes។
5. **សន្មត់ថា Array ចាប់ផ្តើមពី 1**៖ Index ដំបូងរបស់ Indexed Array គឺ 0។
6. **បម្លែងទិន្នន័យដោយមិនពិនិត្យ**៖ កុំសន្មត់ថាអត្ថបទគ្រប់ប្រភេទអាចបម្លែងទៅជាលេខបានត្រឹមត្រូវ។

## ១២. លំហាត់អនុវត្ត

### លំហាត់ A — ងាយ

បង្កើត Variables ចំនួន 5៖

- `$firstName` ជា String
- `$age` ជា Integer
- `$height` ជា Float
- `$isStudent` ជា Boolean
- `$favoriteFoods` ជា Array ដែលមានម្ហូប 3 មុខ

បង្ហាញតម្លៃទាំងអស់ដោយប្រើ `echo` ឬ `var_dump()`។

### លំហាត់ B — មធ្យម

បង្កើតកម្មវិធីគណនាតម្លៃទំនិញ៖

- មាន Variable `$productName` សម្រាប់ឈ្មោះទំនិញ។
- មាន Variable `$price` សម្រាប់តម្លៃមួយឯកតា។
- មាន Variable `$quantity` សម្រាប់ចំនួន។
- គណនា `$total = $price * $quantity`។
- បង្ហាញឈ្មោះទំនិញ តម្លៃមួយឯកតា ចំនួន និងតម្លៃសរុប។

### លំហាត់ C — ពិបាកជាងមុន

មានតម្លៃដើមជា String៖

```php
$priceText = "15.75";
$quantityText = "4";
```

បម្លែងតម្លៃទាំងពីរទៅជាលេខសមស្រប គណនាតម្លៃសរុប ហើយប្រើ `var_dump()` ពិនិត្យប្រភេទទិន្នន័យរបស់លទ្ធផល។

### លំហាត់ D — ស្វែងយល់បន្ថែម

សាកល្បងកូដខាងក្រោម ហើយពន្យល់ថាហេតុអ្វីបានជាលទ្ធផលខុសគ្នា៖

```php
<?php
$language = "PHP";

echo 'I learn $language';
echo PHP_EOL;
echo "I learn $language";
```

## ១៣. សំណួររំលឹកមេរៀន

សូមឆ្លើយដោយខ្លួនឯង មុនពេលមើលចម្លើយ៖

1. Variable ក្នុង PHP ត្រូវចាប់ផ្តើមដោយសញ្ញាអ្វី?
2. តើ `$name` និង `$Name` ជា Variable ដូចគ្នាឬទេ?
3. តើ String និង Integer ខុសគ្នាដូចម្តេច?
4. តើ Boolean មានតម្លៃអ្វីខ្លះ?
5. តើ `var_dump()` ប្រើសម្រាប់អ្វី?
6. តើ Array Index ដំបូងជាលេខប៉ុន្មាន?
7. តើ Type Casting ខុសពី Type Juggling ដូចម្តេច?
8. តើប្រើសញ្ញាអ្វីដើម្បីភ្ជាប់ String ក្នុង PHP?

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

<details>
<summary>ចុចទីនេះដើម្បីមើលចម្លើយសំណួររំលឹក</summary>

1. ចាប់ផ្តើមដោយ `$`។
2. មិនដូចគ្នាទេ ព្រោះ PHP ប្រកាន់អក្សរតូច/ធំ។
3. String ផ្ទុកអត្ថបទ; Integer ផ្ទុកចំនួនគត់។
4. `true` និង `false`។
5. បង្ហាញប្រភេទទិន្នន័យ និងតម្លៃ។
6. Index ដំបូងគឺ `0`។
7. Type Casting ជាការបម្លែងដែលអ្នកសរសេរកូដកំណត់ដោយផ្ទាល់; Type Juggling ជាការបម្លែងដោយស្វ័យប្រវត្តិរបស់ PHP ក្នុងបរិបទខ្លះ។
8. ប្រើ Dot Operator `.`។

</details>

<details>
<summary>ចុចទីនេះដើម្បីមើលគំរូចម្លើយលំហាត់ B</summary>

```php
<?php
$productName = "Notebook";
$price = 2.50;
$quantity = 4;

$total = $price * $quantity;

echo "Product: " . $productName . PHP_EOL;
echo "Price: $" . $price . PHP_EOL;
echo "Quantity: " . $quantity . PHP_EOL;
echo "Total: $" . $total . PHP_EOL;
```

លទ្ធផល៖

```text
Product: Notebook
Price: $2.5
Quantity: 4
Total: $10
```

</details>

## ១៥. កិច្ចការផ្ទះ

- ធ្វើលំហាត់ A, B, C និង D។
- បង្កើត Folder `lesson-02` ហើយរក្សាទុកឯកសារកូដរបស់អ្នក។
- សាកល្បងកែតម្លៃ Variables ច្រើនដង ហើយមើលថាលទ្ធផលប្រែប្រួលដូចម្តេច។
- កុំរំលងទៅមេរៀនបន្ទាប់ រហូតដល់អ្នកអាចពន្យល់ពី String, Integer, Float, Boolean, Array និង NULL បាន។

## ១៦. ឯកសារយោងផ្លូវការ

- [PHP Manual — Variables](https://www.php.net/manual/en/language.variables.php)
- [PHP Manual — Variable Basics](https://www.php.net/manual/en/language.variables.basics.php)
- [PHP Manual — Types](https://www.php.net/manual/en/language.types.php)
- [PHP Manual — Type Juggling](https://www.php.net/manual/en/language.types.type-juggling.php)
- [PHP Manual — Type Casting](https://www.php.net/manual/en/language.types.typecasting.php)
- [PHP Manual — Strings](https://www.php.net/manual/en/language.types.string.php)
- [PHP Manual — Arrays](https://www.php.net/manual/en/language.types.array.php)

---

**បញ្ចប់មេរៀនទី ២។** មេរៀនបន្ទាប់៖ Operators ក្នុង PHP — Arithmetic, Assignment, Comparison, Logical និងការអនុវត្តគណនា។
