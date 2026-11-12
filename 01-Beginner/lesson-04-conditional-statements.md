# មេរៀនទី ៤៖ Conditional Statements ក្នុង PHP

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> ឯកសារ៖ `01-Beginner/lesson-04-conditional-statements.md`

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកអាច៖

- ពន្យល់ពី Conditional Statements និងការសម្រេចចិត្តរបស់កម្មវិធី។
- ប្រើ `if`, `else` និង `elseif`។
- សរសេរ Nested If និងយល់ពីការរៀបចំលក្ខខណ្ឌច្រើន។
- ប្រើ `switch` ដើម្បីជ្រើសរើសករណី។
- ប្រើ `match` Expression ក្នុង PHP 8+។
- យល់ពី Truthy និង Falsy Values។
- ជ្រើសរើសរបៀបសរសេរលក្ខខណ្ឌឱ្យសមស្រប និងងាយថែទាំ។

## ២. Conditional Statement គឺជាអ្វី?

**Conditional Statement (សេចក្តីបញ្ជាលក្ខខណ្ឌ)** គឺជារបៀបឱ្យកម្មវិធីសម្រេចថាត្រូវដំណើរការកូដណាមួយ ដោយផ្អែកលើលក្ខខណ្ឌ។

ឧទាហរណ៍ក្នុងជីវិតប្រចាំថ្ងៃ៖

- ប្រសិនបើភ្លៀងធ្លាក់ → យកឆ័ត្រ។
- បើមិនភ្លៀង → ចេញទៅក្រៅធម្មតា។

ក្នុង PHP យើងប្រើ `if` និង `else` ដើម្បីសរសេរគំនិតនេះ។

## ៣. if Statement

`if` ដំណើរការកូដនៅក្នុងប្លុករបស់វា ប្រសិនបើលក្ខខណ្ឌមានតម្លៃ `true`។

ទម្រង់៖

```php
if (condition) {
    // code runs when condition is true
}
```

ឧទាហរណ៍៖

```php
<?php
$age = 20;

if ($age >= 18) {
    echo "You are an adult.";
}
```

លទ្ធផល៖

```text
You are an adult.
```

ការពន្យល់៖

- `$age >= 18` ជាលក្ខខណ្ឌប្រៀបធៀប។
- ព្រោះ 20 ធំជាង ឬស្មើ 18 លទ្ធផលគឺ `true`។
- ដូច្នេះ PHP ដំណើរការ `echo` នៅក្នុងប្លុក `if`។

បើកំណត់ `$age = 15` នោះលក្ខខណ្ឌក្លាយជា `false` ហើយកូដក្នុងប្លុក `if` មិនដំណើរការទេ។

## ៤. if...else Statement

ប្រើ `else` នៅពេលចង់កំណត់កូដជំនួស ប្រសិនបើលក្ខខណ្ឌក្នុង `if` មិនពិត។

ទម្រង់៖

```php
if (condition) {
    // when true
} else {
    // when false
}
```

ឧទាហរណ៍៖

```php
<?php
$score = 42;

if ($score >= 50) {
    echo "Passed";
} else {
    echo "Failed";
}
```

លទ្ធផល៖

```text
Failed
```

សាកល្បងប្តូរ `$score` ទៅជា 75។ លទ្ធផលនឹងក្លាយជា `Passed`។

## ៥. if...elseif...else Statement

ប្រើនៅពេលមានលក្ខខណ្ឌច្រើនដែលត្រូវពិនិត្យតាមលំដាប់។

ឧទាហរណ៍៖ ប្រព័ន្ធដាក់និទ្ទេសតាមពិន្ទុ។

```php
<?php
$score = 85;

if ($score >= 90) {
    echo "Grade A";
} elseif ($score >= 80) {
    echo "Grade B";
} elseif ($score >= 70) {
    echo "Grade C";
} elseif ($score >= 60) {
    echo "Grade D";
} elseif ($score >= 50) {
    echo "Grade E";
} else {
    echo "Grade F";
}
```

លទ្ធផល៖

```text
Grade B
```

ការពន្យល់៖

1. PHP ពិនិត្យ `$score >= 90` មុន។ វាមិនពិត។
2. បន្ទាប់មកពិនិត្យ `$score >= 80`។ វាពិត។
3. PHP ដំណើរការប្លុក `elseif` នោះ។
4. ប្លុកលក្ខខណ្ឌបន្ទាប់មិនត្រូវបានពិនិត្យទៀតទេ។

**ចំណុចសំខាន់៖** រៀបលក្ខខណ្ឌពីជួរខ្ពស់ទៅទាប ដើម្បីជៀសវាងករណីលក្ខខណ្ឌទូលំទូលាយចាប់បានមុន។

ឧទាហរណ៍ខុស៖

```php
<?php
$score = 95;

if ($score >= 50) {
    echo "Passed";
} elseif ($score >= 90) {
    echo "Excellent";
}
```

កូដនេះបង្ហាញ `Passed` ព្រោះលក្ខខណ្ឌដំបូងពិត ហើយ PHP មិនទៅដល់ `elseif` ទេ។

## ៦. Nested If — If នៅក្នុង If

**Nested If** គឺការដាក់ `if` មួយនៅខាងក្នុង `if` មួយទៀត។ វាមានប្រយោជន៍នៅពេលលក្ខខណ្ឌទីពីរត្រូវពិនិត្យ បន្ទាប់ពីលក្ខខណ្ឌទីមួយពិត។

ឧទាហរណ៍៖

```php
<?php
$age = 20;
$hasCard = true;

if ($age >= 18) {
    if ($hasCard) {
        echo "You can enter.";
    } else {
        echo "Please show your card.";
    }
} else {
    echo "You are not eligible.";
}
```

លទ្ធផល៖

```text
You can enter.
```

ការពន្យល់៖

- ពិនិត្យអាយុជាមុន។
- បើអាយុចាប់ពី 18 ឡើងទៅ ទើបពិនិត្យថាមានកាតឬអត់។
- បើអាយុតិចជាង 18 នឹងបង្ហាញសារមិនអាចចូលបាន។

សម្រាប់ករណីសាមញ្ញ អ្នកអាចភ្ជាប់លក្ខខណ្ឌដោយ `&&` ដើម្បីកាត់បន្ថយ Nested If៖

```php
<?php
$age = 20;
$hasCard = true;

if ($age >= 18 && $hasCard) {
    echo "You can enter.";
} else {
    echo "Entry requirements not met.";
}
```

ជ្រើសរើសរបៀបដែលធ្វើឱ្យកូដអានងាយបំផុត។

## ៧. switch Statement

`switch` ប្រើសម្រាប់ជ្រើសរើសកូដតាមតម្លៃមួយដែលមានករណីជាក់លាក់ច្រើន។

ទម្រង់៖

```php
switch ($value) {
    case value1:
        // code
        break;

    case value2:
        // code
        break;

    default:
        // if no case matches
}
```

ឧទាហរណ៍៖

```php
<?php
$day = 3;

switch ($day) {
    case 1:
        echo "Monday";
        break;

    case 2:
        echo "Tuesday";
        break;

    case 3:
        echo "Wednesday";
        break;

    case 4:
        echo "Thursday";
        break;

    case 5:
        echo "Friday";
        break;

    default:
        echo "Weekend or invalid day";
}
```

លទ្ធផល៖

```text
Wednesday
```

ការពន្យល់៖

- `switch ($day)` ជ្រើសរើសតម្លៃដែលត្រូវពិនិត្យ។
- `case 3:` ត្រូវនឹងតម្លៃ 3។
- `break;` បញ្ឈប់ការរត់ចេញពី Switch បន្ទាប់ពី Case ដែលត្រូវគ្នា។
- `default:` ដំណើរការនៅពេលគ្មាន Case ណាត្រូវគ្នា។

### ហេតុអ្វីត្រូវប្រើ break?

បើភ្លេច `break` ក្នុង Switch បែបធម្មតា PHP អាចបន្តរត់ទៅ Case បន្ទាប់ៗទៀត (Fall-through)។

```php
<?php
$color = "red";

switch ($color) {
    case "red":
        echo "Red ";
        // គ្មាន break ដោយចេតនា

    case "blue":
        echo "Blue";
        break;
}
```

លទ្ធផល៖

```text
Red Blue
```

ការបន្តនេះអាចប្រើបានក្នុងករណីជាក់លាក់ ប៉ុន្តែជាធម្មតា ត្រូវដាក់ `break` ដើម្បីជៀសវាងកំហុស។

## ៨. match Expression (PHP 8+)

`match` ត្រូវបានណែនាំក្នុង PHP 8.0។ វាមានលក្ខណៈខ្លះដូច Switch ប៉ុន្តែមានចំណុចខុសគ្នាសំខាន់ៗ៖

- `match` ជា Expression ដូច្នេះវាអាចផ្តល់តម្លៃត្រឡប់។
- វាប្រៀបធៀបដោយតម្លៃ និងប្រភេទយ៉ាងតឹងរឹង (Strict Comparison)។
- មិនត្រូវការ `break` ទេ។
- អាចដាក់ករណីច្រើនក្នុង Arm មួយ។
- ត្រូវមាន Default ឬគ្រប់គ្រងគ្រប់តម្លៃដែលអាចកើតមាន បើមិនដូច្នោះទេអាចមាន `UnhandledMatchError`។

ឧទាហរណ៍៖

```php
<?php
$day = 3;

$dayName = match ($day) {
    1 => "Monday",
    2 => "Tuesday",
    3 => "Wednesday",
    4 => "Thursday",
    5 => "Friday",
    6, 7 => "Weekend",
    default => "Invalid day",
};

echo $dayName;
```

លទ្ធផល៖

```text
Wednesday
```

### ប្រៀបធៀប Switch និង Match

| លក្ខណៈ | switch | match |
|---|---|---|
| ប្រភេទ | Statement | Expression |
| ការប្រៀបធៀប | មិនមែន Strict ដូច match | Strict (`===`) |
| ត្រូវការ break | ជាទូទៅត្រូវការ | មិនត្រូវការ |
| ផ្តល់តម្លៃត្រឡប់ | មិនមែនជាលក្ខណៈ Expression | បាន |
| ករណីមិនត្រូវគ្នា | អាចប្រើ default | អាចប្រើ default |

ប្រើ `match` ពេលចង់បម្លែងតម្លៃមួយទៅតម្លៃមួយទៀតដោយច្បាស់លាស់។ ប្រើ `switch` ពេលត្រូវការដំណើរការ Statement ច្រើនក្នុង Case ឬត្រូវការអាកប្បកិរិយា Switch ជាក់លាក់។

## ៩. Truthy និង Falsy Values

ក្នុង PHP តម្លៃមិនមែន Boolean ក៏អាចប្រើក្នុងលក្ខខណ្ឌបានដែរ។ PHP នឹងបម្លែងវាទៅជា Boolean តាមច្បាប់របស់ខ្លួន។

តម្លៃ Falsy សំខាន់ៗ៖

- Boolean `false`
- Integer `0`
- Float `0.0`
- String ទទេ `""`
- String `"0"`
- Array ទទេ `[]`
- `null`

តម្លៃផ្សេងទៀតជាទូទៅត្រូវបានចាត់ទុកជា Truthy។

ឧទាហរណ៍៖

```php
<?php
$username = "";

if ($username) {
    echo "Username is provided";
} else {
    echo "Username is empty";
}
```

លទ្ធផល៖

```text
Username is empty
```

**ប្រយ័ត្ន៖** String `"0"` ត្រូវបានចាត់ទុកជា Falsy ក្នុង PHP។ បើអ្នកចង់ពិនិត្យថា String មានឬអត់ សូមប្រើការពិនិត្យជាក់លាក់ដូចជា `$value !== ""` ឬ `isset()` តាមតម្រូវការ កុំសន្មត់ថា Boolean Check មានន័យថា “មិនទទេ” គ្រប់ករណី។

## ១០. Alternative Syntax សម្រាប់ Template

PHP អនុញ្ញាតឱ្យសរសេរ Conditional Statements ដោយប្រើ Syntax ជំនួស ដែលអាចងាយអានក្នុងឯកសារដែលលាយ PHP និង HTML។

```php
<?php
$isLoggedIn = true;
?>

<?php if ($isLoggedIn): ?>
    <p>Welcome back!</p>
<?php else: ?>
    <p>Please log in.</p>
<?php endif; ?>
```

នេះមានប្រយោជន៍សម្រាប់ Template ប៉ុន្តែយើងនឹងរៀន PHP និង HTML រួមគ្នាបន្ថែមនៅមេរៀន Web Development។

## ១១. Mini Project៖ ប្រព័ន្ធវាយតម្លៃពិន្ទុ

បង្កើតឯកសារ `grade-system.php`៖

```php
<?php
$studentName = "Dara";
$score = 86;

echo "===== STUDENT RESULT =====" . PHP_EOL;
echo "Name: " . $studentName . PHP_EOL;
echo "Score: " . $score . PHP_EOL;

if ($score < 0 || $score > 100) {
    echo "Invalid score" . PHP_EOL;
} else {
    $grade = match (true) {
        $score >= 90 => "A",
        $score >= 80 => "B",
        $score >= 70 => "C",
        $score >= 60 => "D",
        $score >= 50 => "E",
        default => "F",
    };

    echo "Grade: " . $grade . PHP_EOL;

    if ($score >= 50) {
        echo "Result: Passed" . PHP_EOL;
    } else {
        echo "Result: Failed" . PHP_EOL;
    }
}
```

លទ្ធផល៖

```text
===== STUDENT RESULT =====
Name: Dara
Score: 86
Grade: B
Result: Passed
```

ការពន្យល់៖

1. កំណត់ឈ្មោះសិស្ស និងពិន្ទុ។
2. ពិនិត្យថាពិន្ទុស្ថិតក្នុងចន្លោះ 0 ដល់ 100 ឬអត់។
3. ប្រើ `match (true)` ដើម្បីជ្រើសរើស Grade តាមលក្ខខណ្ឌ។
4. ពិន្ទុចាប់ពី 50 ឡើងទៅគឺ Passed។
5. បើពិន្ទុក្រៅចន្លោះដែលបានកំណត់ បង្ហាញ Invalid Score។

ចំណាំ៖ `match (true)` អាចប្រើសម្រាប់វាយតម្លៃលក្ខខណ្ឌជាបន្តបន្ទាប់។ លក្ខខណ្ឌត្រូវរៀបពីខ្ពស់ទៅទាបដូចឧទាហរណ៍។

## ១២. របៀបដំណើរការកូដ

បើក Terminal នៅក្នុង Folder ដែលមានឯកសារ ហើយដំណើរការ៖

```bash
php grade-system.php
```

សាកល្បងប្តូរ `$score` ទៅជា 45, 72, 95, -5 និង 120 ដើម្បីសាកល្បងគ្រប់ករណី។

## ១៣. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ដាក់ Semicolon ក្រោយលក្ខខណ្ឌខុសកន្លែង**៖ កុំសរសេរ `if ($score >= 50); ` ព្រោះវាធ្វើឱ្យប្លុកក្រោយមិនដំណើរការតាមការរំពឹង។
2. **រៀបលក្ខខណ្ឌមិនត្រឹមត្រូវ**៖ លក្ខខណ្ឌទូលំទូលាយដូចជា `$score >= 50` អាចចាប់បានមុន `$score >= 90`។
3. **ភ្លេច break ក្នុង Switch**៖ អាចធ្វើឱ្យកូដបន្តរត់ Case បន្ទាប់។
4. **ភ្លេច default ក្នុង Match**៖ តម្លៃដែលមិនត្រូវនឹង Arm ណាមួយអាចបង្កើត `UnhandledMatchError`។
5. **ច្រឡំ `=` ជាមួយ `===`**៖ `=` កំណត់តម្លៃ; `===` ប្រៀបធៀប។
6. **សន្មត់ថា String ទទេជាតម្លៃ Falsy តែមួយ**៖ String `"0"` ក៏ជា Falsy ដែរ។
7. **ដាក់លក្ខខណ្ឌស្មុគស្មាញក្នុង Ternary ច្រើនជាន់**៖ ប្រើ `if...elseif...else` ដើម្បីឱ្យងាយអាន។

## ១៤. លំហាត់អនុវត្ត

### លំហាត់ A — ងាយ

បង្កើត Variable `$temperature = 32`។

- បើសីតុណ្ហភាពធំជាង 35 បង្ហាញ `Hot`។
- បើស្មើ 25 ដល់ 35 បង្ហាញ `Warm`។
- បើតិចជាង 25 បង្ហាញ `Cool`។

ប្រើ `if...elseif...else`។

### លំហាត់ B — មធ្យម

បង្កើត Variable `$number = 12`។ ប្រើ `if...else` និង Modulus ដើម្បីពិនិត្យថាជាលេខគូ ឬលេខសេស។

### លំហាត់ C — មធ្យម

បង្កើត Variable `$role = "editor"`។ ប្រើ `switch` ដើម្បីបង្ហាញ៖

- `admin` → `Full access`
- `editor` → `Edit content`
- `viewer` → `Read only`
- ករណីផ្សេងទៀត → `Unknown role`

### លំហាត់ D — មធ្យមទៅពិបាក

ប្រើ `match` ដើម្បីបម្លែងលេខខែទៅជាឈ្មោះរដូវកាល ឬក្រុមខែ៖

- 12, 1, 2 → `Winter`
- 3, 4, 5 → `Spring`
- 6, 7, 8 → `Summer`
- 9, 10, 11 → `Autumn`
- លេខផ្សេងទៀត → `Invalid month`

### លំហាត់ E — Mini Challenge

បង្កើតកម្មវិធី Login Check ដោយប្រើ Variables៖

```php
$username = "student";
$password = "php123";
$isActive = true;
```

ប្រើលក្ខខណ្ឌដើម្បីបង្ហាញ៖

- បើ Username និង Password ត្រូវ ហើយ Account Active → `Login successful`
- បើ Account មិន Active → `Account disabled`
- បើព័ត៌មានមិនត្រូវ → `Invalid credentials`

នេះគ្រាន់តែជាលំហាត់សម្រាប់រៀន Conditional Logic ប៉ុណ្ណោះ មិនមែនជាប្រព័ន្ធ Login សុវត្ថិភាពសម្រាប់ប្រើប្រាស់ពិតទេ។ ប្រព័ន្ធពិតត្រូវរក្សាទុក Password ជា Hash និងផ្ទៀងផ្ទាត់ដោយវិធីសុវត្ថិភាព។

## ១៥. សំណួររំលឹក

សូមសាកឆ្លើយមុនមើលចម្លើយ៖

1. តើ `if` ដំណើរការកូដនៅពេលណា?
2. តើ `elseif` មានតួនាទីអ្វី?
3. ហេតុអ្វីត្រូវរៀបលក្ខខណ្ឌពិន្ទុពីខ្ពស់ទៅទាប?
4. តើ Nested If ជាអ្វី?
5. តើ `break` ក្នុង Switch ប្រើសម្រាប់អ្វី?
6. តើ Match ខុសពី Switch ដូចម្តេច?
7. តើ Truthy និង Falsy មានន័យដូចម្តេច?
8. តើ String `"0"` ជា Truthy ឬ Falsy ក្នុង PHP?

## ១៦. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

<details>
<summary>ចុចទីនេះដើម្បីមើលចម្លើយសំណួររំលឹក</summary>

1. នៅពេលលក្ខខណ្ឌមានតម្លៃ `true`។
2. ពិនិត្យលក្ខខណ្ឌបន្ថែម បើលក្ខខណ្ឌមុនមិនពិត។
3. ដើម្បីកុំឱ្យលក្ខខណ្ឌទូលំទូលាយចាប់បានមុនលក្ខខណ្ឌជាក់លាក់។
4. គឺ `if` មួយនៅក្នុង `if` មួយទៀត។
5. បញ្ឈប់ការបន្តទៅ Case បន្ទាប់ក្នុង Switch។
6. Match ជា Expression, ប្រៀបធៀប Strict, មិនត្រូវការ break និងអាចផ្តល់តម្លៃត្រឡប់។
7. Truthy គឺតម្លៃដែលត្រូវបានបម្លែងទៅ true; Falsy គឺតម្លៃដែលត្រូវបានបម្លែងទៅ false។
8. Falsy។

</details>

<details>
<summary>ចុចទីនេះដើម្បីមើលគំរូចម្លើយលំហាត់ A</summary>

```php
<?php
$temperature = 32;

if ($temperature > 35) {
    echo "Hot";
} elseif ($temperature >= 25) {
    echo "Warm";
} else {
    echo "Cool";
}
```

លទ្ធផល៖

```text
Warm
```

</details>

<details>
<summary>ចុចទីនេះដើម្បីមើលគំរូចម្លើយលំហាត់ C</summary>

```php
<?php
$role = "editor";

switch ($role) {
    case "admin":
        echo "Full access";
        break;

    case "editor":
        echo "Edit content";
        break;

    case "viewer":
        echo "Read only";
        break;

    default:
        echo "Unknown role";
}
```

លទ្ធផល៖

```text
Edit content
```

</details>

## ១៧. កិច្ចការផ្ទះ

- ធ្វើលំហាត់ A ដល់ E។
- រក្សាទុកឯកសារកូដក្នុង Folder `lesson-04`។
- សាកល្បងករណីដែលលក្ខខណ្ឌពិត និងមិនពិត។
- ពន្យល់ដោយពាក្យរបស់អ្នកថា ហេតុអ្វី Match ប្រើ Strict Comparison។
- កុំបន្តទៅមេរៀនបន្ទាប់ រហូតដល់អ្នកអាចសរសេរ `if...else` និង `switch` ដោយខ្លួនឯង។

## ១៨. ឯកសារយោងផ្លូវការ

- [PHP Manual — if](https://www.php.net/manual/en/control-structures.if.php)
- [PHP Manual — switch](https://www.php.net/manual/en/control-structures.switch.php)
- [PHP Manual — match](https://www.php.net/manual/en/control-structures.match.php)
- [PHP Manual — Booleans](https://www.php.net/manual/en/language.types.boolean.php)
- [PHP Manual — Logical Operators](https://www.php.net/manual/en/language.operators.logical.php)

---

**បញ្ចប់មេរៀនទី ៤។** មេរៀនបន្ទាប់៖ Loops ក្នុង PHP — `while`, `do...while`, `for`, `foreach`, `break` និង `continue`។
