# មេរៀនទី ៨៖ PHP Strings (ការគ្រប់គ្រងអត្ថបទ)

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> គោលបំណង៖ យល់ពី String, ការភ្ជាប់អត្ថបទ, ស្វែងរក, កែប្រែ, បំបែក, រៀបចំទម្រង់ និងដោះស្រាយអត្ថបទ UTF-8 ជាពិសេសភាសាខ្មែរ។

---

## ១. តើ String គឺជាអ្វី?

**String** គឺជាប្រភេទទិន្នន័យសម្រាប់រក្សាទុកអត្ថបទ ដូចជា ឈ្មោះ ប្រយោគ អាសយដ្ឋាន និងសារផ្សេងៗ។

PHP អនុញ្ញាតឱ្យសរសេរ String ដោយប្រើ៖
- Single quotes: `'Hello'`
- Double quotes: `"Hello"`
- Heredoc និង Nowdoc សម្រាប់អត្ថបទច្រើនបន្ទាត់។

```php
<?php
$name = "Dara";
$language = 'PHP';

echo $name . PHP_EOL;
echo $language . PHP_EOL;
```

លទ្ធផល៖

```text
Dara
PHP
```

### ១.១ Single Quotes និង Double Quotes

Double quotes អាចបកស្រាយ Variable និង Escape Sequences មួយចំនួន។ Single quotes មិនបកស្រាយ Variable ទេ លើកលែងតែ Escape សម្រាប់សញ្ញា quote និង Backslash។

```php
<?php
$name = "Sokha";

echo 'Hello $name' . PHP_EOL;
echo "Hello $name" . PHP_EOL;
echo "Line 1\nLine 2" . PHP_EOL;
```

លទ្ធផល៖

```text
Hello $name
Hello Sokha
Line 1
Line 2
```

Escape Sequences ដែលប្រើញឹកញាប់ក្នុង Double Quotes៖
- `\n` បន្ទាត់ថ្មី (newline)។
- `\t` Tab។
- `\r` Carriage return។
- `\$` សញ្ញា Dollar ដោយមិនចាប់ផ្តើម Variable។
- `\\"` សញ្ញា Double Quote។
- `\\` Backslash។

## ២. ភ្ជាប់ String (Concatenation)

ប្រើ Dot Operator `.` ដើម្បីភ្ជាប់ String ពីរ ឬច្រើន។

```php
<?php
$firstName = "Dara";
$lastName = "Chhim";

$fullName = $firstName . " " . $lastName;

echo $fullName . PHP_EOL;
```

លទ្ធផល៖ `Dara Chhim`

ប្រើ `.=` ដើម្បីបន្ថែមអត្ថបទទៅ String ដែលមានស្រាប់៖

```php
<?php
$message = "Hello";
$message .= " PHP";
$message .= " Developer";

echo $message . PHP_EOL;
```

លទ្ធផល៖ `Hello PHP Developer`

## ៣. String Interpolation

ក្នុង Double Quotes អ្នកអាចបញ្ចូល Variable ទៅក្នុង String ដោយផ្ទាល់។

```php
<?php
$name = "Malis";
$age = 18;

echo "My name is {$name} and I am {$age} years old." . PHP_EOL;
```

ការប្រើ `{$variable}` ធ្វើឱ្យព្រំដែន Variable ច្បាស់លាស់ ជាពិសេសពេលភ្ជាប់ជាមួយអក្សរផ្សេងទៀត។

## ៤. String Length និង UTF-8

ចំណុចនេះសំខាន់ណាស់សម្រាប់ភាសាខ្មែរ៖

- `strlen($text)` រាប់ប្រវែងជា **bytes**។
- `mb_strlen($text, 'UTF-8')` រាប់ចំនួនតួអក្សរតាម Multibyte Encoding។
- តួអក្សរខ្មែរមួយអាចប្រើច្រើន bytes ដូច្នេះលទ្ធផល `strlen()` និង `mb_strlen()` អាចខុសគ្នា។

```php
<?php
$text = "សួស្តី";

echo "Bytes: " . strlen($text) . PHP_EOL;

if (function_exists("mb_strlen")) {
    echo "Characters: " . mb_strlen($text, "UTF-8") . PHP_EOL;
} else {
    echo "mbstring extension is not enabled." . PHP_EOL;
}
```

ចំណាំ៖ ត្រូវដំឡើង និងបើក PHP Extension `mbstring` ដើម្បីប្រើ `mb_strlen()`។ ចំនួនតួអក្សរដែលមនុស្សមើលឃើញអាចស្មុគស្មាញជាងចំនួន Unicode code points ក្នុងករណីមាន Combining Marks ឬ Emoji ផ្សំគ្នា។

## ៥. String Functions សំខាន់ៗ

### ៥.១ trim(), ltrim(), rtrim()

ដក Whitespace ឬតួអក្សរដែលបានកំណត់ចេញពីចុង String៖

```php
<?php
$text = "   Learn PHP   ";

echo "[" . trim($text) . "]" . PHP_EOL;
echo "[" . ltrim($text) . "]" . PHP_EOL;
echo "[" . rtrim($text) . "]" . PHP_EOL;
```

- `trim()` ដកពីទាំងសងខាង។
- `ltrim()` ដកពីខាងឆ្វេង។
- `rtrim()` ដកពីខាងស្តាំ។

### ៥.២ strtoupper(), strtolower(), ucfirst(), ucwords()

មុខងារទាំងនេះប្រើសម្រាប់អក្សរ Latin៖

```php
<?php
$text = "hello php developer";

echo strtoupper($text) . PHP_EOL;
echo strtolower("HELLO PHP") . PHP_EOL;
echo ucfirst($text) . PHP_EOL;
echo ucwords($text) . PHP_EOL;
```

លទ្ធផល៖

```text
HELLO PHP DEVELOPER
hello php
Hello php developer
Hello Php Developer
```

ចំណាំ៖ Function ទាំងនេះមិនមែនជាវិធីបម្លែងអក្សរធំតូចដែលគាំទ្រ Unicode គ្រប់ភាសាទេ។ សម្រាប់ Unicode អាចប្រើ `mb_strtoupper()` និង `mb_strtolower()` ប្រសិនបើមាន `mbstring`។

### ៥.៣ str_replace() — ជំនួសអត្ថបទ

```php
<?php
$text = "I like Java. Java is popular.";

$result = str_replace("Java", "PHP", $text);

echo $result . PHP_EOL;
```

លទ្ធផល៖ `I like PHP. PHP is popular.`

អ្នកអាចផ្ញើ Array សម្រាប់ស្វែងរក និងជំនួសតម្លៃច្រើន៖

```php
<?php
$text = "red blue green";

$result = str_replace(
    ["red", "blue"],
    ["ក្រហម", "ខៀវ"],
    $text
);

echo $result . PHP_EOL;
```

### ៥.៤ str_ireplace() — ជំនួសដោយមិនប្រកាន់អក្សរធំតូច

```php
<?php
$text = "PHP is great. php is useful.";

echo str_ireplace("php", "Laravel", $text) . PHP_EOL;
```

លទ្ធផល៖ `Laravel is great. Laravel is useful.`

### ៥.៥ strpos() និង stripos() — ស្វែងរកទីតាំង

`strpos()` ស្វែងរកទីតាំងដំបូងដែល Substring លេចឡើង។ វាត្រឡប់ Position ឬ `false` ប្រសិនបើមិនរកឃើញ។

```php
<?php
$text = "Learn PHP programming";

$position = strpos($text, "PHP");

if ($position !== false) {
    echo "Found at position: " . $position . PHP_EOL;
} else {
    echo "Not found" . PHP_EOL;
}
```

លទ្ធផល៖ `Found at position: 6`

**ប្រយ័ត្ន៖** ត្រូវប្រៀបធៀបជាមួយ `!== false` មិនមែន `if ($position)` ទេ ព្រោះអត្ថបទអាចរកឃើញនៅទីតាំង 0។

`stripos()` ដូច `strpos()` ប៉ុន្តែមិនប្រកាន់អក្សរធំតូចសម្រាប់ការស្វែងរកធម្មតា។

### ៥.៦ substr() — កាត់ String

```php
<?php
$text = "Hello PHP Developer";

echo substr($text, 6, 3) . PHP_EOL;
echo substr($text, 6) . PHP_EOL;
```

លទ្ធផល៖

```text
PHP
PHP Developer
```

`substr()` រាប់ជា Bytes។ សម្រាប់អត្ថបទ UTF-8 ដែលមានភាសាខ្មែរ ប្រើ `mb_substr()` ជាមួយ `mbstring` ដើម្បីកាត់តាមតួអក្សរ។

```php
<?php
$text = "សួស្តីពិភពលោក";

if (function_exists("mb_substr")) {
    echo mb_substr($text, 0, 5, "UTF-8") . PHP_EOL;
}
```

### ៥.៧ str_contains(), str_starts_with(), str_ends_with()

Function ទាំងនេះមានចាប់ពី PHP 8៖

```php
<?php
$text = "Learn PHP today";

var_dump(str_contains($text, "PHP"));
var_dump(str_starts_with($text, "Learn"));
var_dump(str_ends_with($text, "today"));
```

លទ្ធផល៖

```text
bool(true)
bool(true)
bool(true)
```

Function ទាំងនេះប្រកាន់អក្សរធំតូច (case-sensitive)។

### ៥.៨ strrev() — បញ្ច្រាស String

```php
<?php
echo strrev("ABCDE") . PHP_EOL;
```

លទ្ធផល៖ `EDCBA`

ចំណាំ៖ `strrev()` ដំណើរការតាម Bytes ហើយមិនសមស្របសម្រាប់បញ្ច្រាសអត្ថបទ UTF-8 ដូចជាភាសាខ្មែរ។ កុំប្រើវាលើអត្ថបទខ្មែរ ប្រសិនបើចង់រក្សាតួអក្សរឱ្យត្រឹមត្រូវ។

## ៦. បំបែក និងបញ្ចូល String

### ៦.១ explode()

បំបែក String ទៅជា Array ដោយប្រើ Separator៖

```php
<?php
$csv = "Dara,Sokha,Malis";

$names = explode(",", $csv);

print_r($names);
```

លទ្ធផល៖ Array មាន Dara, Sokha និង Malis។

អាចកំណត់ចំនួនផ្នែកអតិបរមាដោយ Argument ទីបី៖

```php
<?php
$path = "home/user/documents/file.txt";

$parts = explode("/", $path, 3);

print_r($parts);
```

### ៦.២ implode()

បញ្ចូល Array ទៅជា String ដោយប្រើ Separator៖

```php
<?php
$skills = ["HTML", "CSS", "PHP"];

echo implode(" | ", $skills) . PHP_EOL;
```

លទ្ធផល៖ `HTML | CSS | PHP`

## ៧. String Formatting

### ៧.១ sprintf()

`sprintf()` បង្កើត String តាម Format ហើយ Return String ដោយមិនបង្ហាញវាភ្លាមៗ។

```php
<?php
$name = "Dara";
$score = 95;

$message = sprintf("Student %s scored %d points.", $name, $score);

echo $message . PHP_EOL;
```

លទ្ធផល៖ `Student Dara scored 95 points.`

Format Specifiers ដែលប្រើញឹកញាប់៖
- `%s` String។
- `%d` Integer។
- `%f` Float។
- `%.2f` Float មាន 2 ខ្ទង់ក្រោយក្បៀស។

### ៧.២ printf()

`printf()` បង្ហាញអត្ថបទតាម Format ដោយផ្ទាល់៖

```php
<?php
$product = "Keyboard";
$price = 25.5;

printf("%s costs $%.2f\n", $product, $price);
```

លទ្ធផល៖ `Keyboard costs $25.50`

### ៧.៣ number_format()

ប្រើសម្រាប់បង្ហាញលេខដែលមានសញ្ញាបំបែកខ្ទង់៖

```php
<?php
$price = 1234567.5;

echo number_format($price, 2) . PHP_EOL;
echo number_format($price, 2, ".", ",") . PHP_EOL;
```

លទ្ធផល៖

```text
1,234,567.50
1,234,567.50
```

## ៨. Heredoc និង Nowdoc

ប្រើ Heredoc និង Nowdoc សម្រាប់ String ច្រើនបន្ទាត់។

### ៨.១ Heredoc

Heredoc បកស្រាយ Variable ដូច Double Quotes៖

```php
<?php
$name = "Dara";

$message = <<<TEXT
Hello, {$name}!
Welcome to PHP.
TEXT;

echo $message . PHP_EOL;
```

### ៨.២ Nowdoc

Nowdoc មិនបកស្រាយ Variable ដូច Single Quotes៖

```php
<?php
$name = "Dara";

$message = <<<'TEXT'
Hello, $name!
Welcome to PHP.
TEXT;

echo $message . PHP_EOL;
```

លទ្ធផល៖ `Hello, $name!` និងបន្ទាត់បន្ទាប់ `Welcome to PHP.`

ចំណាំ៖ សញ្ញាបិទ Identifier ត្រូវដាក់តាម Syntax របស់ PHP ហើយកុំដាក់អត្ថបទបន្ថែមលើបន្ទាត់បិទ។

## ៩. String និង Security

ការបង្ហាញទិន្នន័យដែលអ្នកប្រើប្រាស់បញ្ចូលក្នុង HTML ត្រូវ Escape ឱ្យត្រឹមត្រូវ ដើម្បីកាត់បន្ថយហានិភ័យ HTML Injection និង Cross-Site Scripting (XSS)។

ឧទាហរណ៍៖

```php
<?php
$userInput = '<script>alert("Hello")</script>';

$safeOutput = htmlspecialchars(
    $userInput,
    ENT_QUOTES | ENT_SUBSTITUTE,
    "UTF-8"
);

echo $safeOutput;
```

នៅពេលបង្ហាញក្នុង Browser Tag នឹងត្រូវបម្លែងទៅជា Text មិនមែនដំណើរការជា Script ទេ។

ចំណុចសំខាន់៖
- ប្រើ `htmlspecialchars()` សម្រាប់បញ្ចូលអត្ថបទទៅក្នុង HTML Text ឬ Attribute ដែលសមស្រប។
- កុំយល់ថា Function នេះអាចការពារគ្រប់ Context ទាំងអស់។ JavaScript, CSS និង URL ត្រូវការការដោះស្រាយខុសគ្នា។
- កុំបញ្ចូល User Input ទៅក្នុង SQL ដោយភ្ជាប់ String។ ប្រើ PDO Prepared Statements ដែលនឹងរៀនក្នុងមេរៀន Database។
- ការផ្ទៀងផ្ទាត់ទិន្នន័យ (Validation) និងការបង្ហាញដោយសុវត្ថិភាព (Output Escaping) គឺជារឿងពីរផ្សេងគ្នា។

## ១០. Mini Project៖ កម្មវិធីសង្ខេប Profile

គោលដៅ៖ ទទួលព័ត៌មានជាអថេរ បង្កើត Profile Text និងបង្ហាញក្នុង HTML ដោយ Escape ទិន្នន័យ។

បង្កើតឯកសារ `profile.php`៖

```php
<?php
declare(strict_types=1);

$name = "Dara";
$city = "Phnom Penh";
$skills = ["PHP", "MySQL", "HTML"];
$bio = "ខ្ញុំកំពុងរៀន PHP និង Web Development។";

// សម្អាត Whitespace ខាងចុង និងខាងមុខ
$name = trim($name);
$city = trim($city);

// បញ្ចូល Skills ជា String
$skillsText = implode(", ", $skills);

// បង្កើតសេចក្តីសង្ខេប
$profileText = sprintf(
    "My name is %s. I live in %s. My skills: %s.",
    $name,
    $city,
    $skillsText
);

// Escape ពេលបញ្ចូលទៅក្នុង HTML
function e(string $value): string {
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        "UTF-8"
    );
}
?>

<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>Student Profile</title>
</head>
<body>
    <h1>Student Profile</h1>

    <p><?= e($profileText) ?></p>
    <p><?= e($bio) ?></p>

    <h2>Skills</h2>
    <ul>
        <?php foreach ($skills as $skill): ?>
            <li><?= e($skill) ?></li>
        <?php endforeach; ?>
    </ul>
</body>
</html>
```

លទ្ធផលដែលរំពឹងទុកក្នុង Browser៖
- ចំណងជើង Student Profile។
- សេចក្តីសង្ខេបឈ្មោះ និងទីក្រុង។
- Bio ជាភាសាខ្មែរ។
- បញ្ជី Skills ចំនួន ៣។

ចំណុចដែលបានអនុវត្ត៖
- `trim()` សម្អាត Whitespace។
- `implode()` បញ្ចូល Array ទៅជា String។
- `sprintf()` រៀបចំអត្ថបទតាម Format។
- `htmlspecialchars()` Escape Output សម្រាប់ HTML។
- `foreach` បង្ហាញ Skills ជាបញ្ជី។

## ១១. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ច្រឡំ Single Quotes និង Double Quotes**៖ Variable មិនត្រូវបានបកស្រាយក្នុង Single Quotes។
2. **ប្រើ `strlen()` រាប់តួអក្សរខ្មែរ**៖ វារាប់ Bytes។ ប្រើ `mb_strlen()` សម្រាប់ Multibyte Text។
3. **ប្រើ `substr()` កាត់អត្ថបទខ្មែរ**៖ វាអាចកាត់ចំកណ្តាល Bytes។ ប្រើ `mb_substr()`។
4. **ពិនិត្យ `strpos()` ដោយមិនប្រើ Strict Comparison**៖ ប្រើ `!== false` ព្រោះ Position 0 ក៏ជាលទ្ធផលត្រឹមត្រូវ។
5. **ភ្លេច Separator ក្នុង `explode()` និង `implode()`**៖ ត្រូវកំណត់ឱ្យត្រូវតាមទិន្នន័យ។
6. **បង្ហាញ User Input ដោយមិន Escape**៖ អាចបង្កហានិភ័យ XSS នៅក្នុង HTML។
7. **យល់ថា `strtoupper()` គាំទ្រអក្សរខ្មែរទាំងអស់**៖ វាមិនមែនជា Unicode Case Conversion សម្រាប់គ្រប់ភាសាទេ។

## ១២. លំហាត់អនុវត្ត

សាកល្បងសរសេរកូដដោយខ្លួនឯង មុនពេលមើលចម្លើយនៅខាងក្រោម។

### លំហាត់ទី ១៖ String Concatenation
បង្កើតអថេរ `$firstName` និង `$lastName` ហើយភ្ជាប់ជាឈ្មោះពេញដោយប្រើ Dot Operator។

### លំហាត់ទី ២៖ String Length
ប្រើ `strlen()` រាប់ Bytes នៃ `"Hello PHP"`។ បន្ទាប់មកពន្យល់ថាហេតុអ្វីអត្ថបទខ្មែរអាចមានលទ្ធផលធំជាងចំនួនតួអក្សរ។

### លំហាត់ទី ៣៖ Replace
ប្តូរពាក្យ `Java` ទៅជា `PHP` ក្នុងប្រយោគ `I learn Java every day`។

### លំហាត់ទី ៤៖ Search
ពិនិត្យថា String `"I love PHP"` មានពាក្យ `"PHP"` ឬអត់ ដោយប្រើ `str_contains()`។

### លំហាត់ទី ៥៖ Split និង Join
បំបែក `"HTML,CSS,PHP"` ទៅជា Array ដោយប្រើ `explode()` ហើយបញ្ចូលវិញដោយប្រើ `implode(" | ", ...)`។

### លំហាត់ទី ៦៖ UTF-8
បង្កើត String ភាសាខ្មែរ ហើយពិនិត្យថា Extension `mbstring` មានឬអត់។ ប្រសិនបើមាន សាកល្បង `mb_strlen()` និង `mb_substr()`។

### លំហាត់ទី ៧៖ Mini Project
បង្កើតកម្មវិធីសង្ខេបព័ត៌មានសៀវភៅ ដែលមាន title, author និង description។ ប្រើ `trim()`, `sprintf()` និង `htmlspecialchars()` នៅពេលបង្ហាញក្នុង HTML។

## ១៣. សំណួរត្រួតពិនិត្យចំណេះដឹង

1. តើ String ជាអ្វី?
2. តើ Single Quotes និង Double Quotes ខុសគ្នាដូចម្តេច?
3. តើ Dot Operator `.` ប្រើសម្រាប់អ្វី?
4. តើ `strlen()` និង `mb_strlen()` ខុសគ្នាដូចម្តេច?
5. តើ `strpos()` អាចត្រឡប់តម្លៃអ្វីខ្លះ?
6. ហេតុអ្វីត្រូវប្រើ `!== false` ពេលពិនិត្យលទ្ធផល `strpos()`?
7. តើ `explode()` និង `implode()` ខុសគ្នាដូចម្តេច?
8. តើ `htmlspecialchars()` ជួយការពារអ្វី?

---

## ១៤. ចម្លើយគំរូ (មើលក្រោយពេលសាកល្បងរួច)

### ចម្លើយលំហាត់ទី ១

```php
<?php
$firstName = "Dara";
$lastName = "Chhim";

$fullName = $firstName . " " . $lastName;

echo $fullName . PHP_EOL;
```

លទ្ធផល៖ `Dara Chhim`

### ចម្លើយលំហាត់ទី ២

```php
<?php
$text = "Hello PHP";

echo strlen($text) . PHP_EOL;
```

លទ្ធផល៖ `9`

ភាសាខ្មែរប្រើ UTF-8 ដែលតួអក្សរជាច្រើនប្រើច្រើន Bytes។ ដូច្នេះ `strlen()` រាប់ Bytes មិនមែនចំនួនតួអក្សរដែលមើលឃើញទេ។

### ចម្លើយលំហាត់ទី ៣

```php
<?php
$text = "I learn Java every day";

echo str_replace("Java", "PHP", $text) . PHP_EOL;
```

លទ្ធផល៖ `I learn PHP every day`

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$text = "I love PHP";

var_dump(str_contains($text, "PHP"));
```

លទ្ធផល៖ `bool(true)`

### ចម្លើយលំហាត់ទី ៥

```php
<?php
$text = "HTML,CSS,PHP";

$items = explode(",", $text);

echo implode(" | ", $items) . PHP_EOL;
```

លទ្ធផល៖ `HTML | CSS | PHP`

### ចម្លើយលំហាត់ទី ៦

```php
<?php
$text = "ខ្ញុំរៀន PHP";

if (function_exists("mb_strlen")) {
    echo mb_strlen($text, "UTF-8") . PHP_EOL;
    echo mb_substr($text, 0, 5, "UTF-8") . PHP_EOL;
} else {
    echo "Please enable the mbstring extension." . PHP_EOL;
}
```

ចំនួនតួអក្សរអាស្រ័យលើ String និងរបៀបរាប់ Unicode។ ត្រូវសាកល្បងលទ្ធផលលើ PHP Environment របស់អ្នក។

### ចម្លើយលំហាត់ទី ៧

```php
<?php
$title = "  Learning PHP  ";
$author = "Dara";
$description = "មេរៀន PHP សម្រាប់អ្នកចាប់ផ្តើម។";

$title = trim($title);

function e(string $value): string {
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        "UTF-8"
    );
}

$summary = sprintf(
    "Book: %s | Author: %s",
    $title,
    $author
);
?>

<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>Book Summary</title>
</head>
<body>
    <h1><?= e($title) ?></h1>
    <p><?= e($summary) ?></p>
    <p><?= e($description) ?></p>
</body>
</html>
```

### ចម្លើយសំណួរត្រួតពិនិត្យ

1. String ជាទិន្នន័យសម្រាប់រក្សាទុកអត្ថបទ។
2. Double Quotes បកស្រាយ Variable និង Escape Sequences មួយចំនួន; Single Quotes មិនបកស្រាយ Variable។
3. ភ្ជាប់ String។
4. `strlen()` រាប់ Bytes; `mb_strlen()` រាប់ Multibyte Characters តាម Encoding។
5. Position ជាចំនួនគត់ ឬ `false` ប្រសិនបើរកមិនឃើញ។
6. ព្រោះ Position 0 ជាលទ្ធផលត្រឹមត្រូវ ហើយមានន័យថារកឃើញនៅដើម String។
7. `explode()` បំបែក String ទៅ Array; `implode()` បញ្ចូល Array ទៅ String។
8. វា Escape តួអក្សរពិសេសសម្រាប់ HTML ដើម្បីកាត់បន្ថយហានិភ័យ HTML Injection និង XSS ក្នុង Context ដែលសមស្រប។

## ១៥. របៀបដំណើរការកូដ

### សាកល្បងជា CLI

1. បង្កើតឯកសារ `lesson08.php`។
2. ដាក់កូដ PHP ឧទាហរណ៍ក្នុងឯកសារ។
3. បើក Terminal នៅទីតាំងឯកសារ។
4. ដំណើរការ៖

```bash
php lesson08.php
```

### សាកល្បង Mini Project ក្នុង Browser

1. រក្សាទុកឯកសារ Mini Project ជា `profile.php` ក្នុង Folder ដែល Web Server អាចចូលដំណើរការ។
2. ចូលទៅ Folder នោះក្នុង Terminal។
3. ដំណើរការ PHP Built-in Development Server៖

```bash
php -S localhost:8000
```

4. បើក Browser ទៅកាន់ `http://localhost:8000/profile.php`។
5. សម្រាប់ការអភិវឌ្ឍន៍ប៉ុណ្ណោះ៖ PHP Built-in Server មិនគួរប្រើជា Production Server សម្រាប់ Website សាធារណៈទេ។

## សង្ខេបមេរៀន

ក្នុងមេរៀននេះ អ្នកបានរៀន៖
- String Syntax, Quotes និង Concatenation។
- String Interpolation និង Formatting។
- String Functions សម្រាប់ Trim, Replace, Search និង Substring។
- បំបែក និងបញ្ចូល String ជាមួយ Array។
- Heredoc និង Nowdoc។
- UTF-8 និងការប្រើ `mbstring` សម្រាប់អត្ថបទខ្មែរ។
- Escape HTML Output ដើម្បីកាត់បន្ថយហានិភ័យ XSS។
- បង្កើត Mini Project សង្ខេប Profile។

**ការងារមុនទៅមេរៀនបន្ទាប់៖** អនុវត្តលំហាត់ទាំង ៧ ហើយសាកល្បង String Functions លើអត្ថបទខ្មែរ និងអង់គ្លេស។
