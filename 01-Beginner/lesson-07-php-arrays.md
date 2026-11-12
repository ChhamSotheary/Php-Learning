# មេរៀនទី ៧៖ PHP Arrays (អារេ)

> កម្រិត៖ Beginner  
> ផ្នែក៖ PHP Fundamentals  
> គោលបំណង៖ រៀនរក្សាទុកទិន្នន័យច្រើនក្នុងអថេរតែមួយ អាន បន្ថែម កែប្រែ លុប និងប្រើ Array Functions។

---

## ១. តើ Array គឺជាអ្វី?

**Array (អារេ)** គឺជាប្រភេទទិន្នន័យដែលអាចរក្សាទុកតម្លៃច្រើនក្នុងអថេរតែមួយ។ ឧទាហរណ៍ អ្នកអាចរក្សាទុកឈ្មោះសិស្ស ៥ នាក់ក្នុង Array មួយ ជំនួសឱ្យបង្កើតអថេរចំនួន ៥។

PHP មាន Array សំខាន់ៗ ៣ ប្រភេទ៖

1. **Indexed Array**៖ ប្រើលេខ Index ជា Key។
2. **Associative Array**៖ ប្រើឈ្មោះ Key ជា String។
3. **Multidimensional Array**៖ Array ដែលមាន Array ផ្សេងទៀតនៅខាងក្នុង។

PHP Array អាចផ្ទុកតម្លៃប្រភេទផ្សេងៗគ្នាបាន ដូចជា String, Integer, Float, Boolean និង Array ផ្សេងទៀត។

## ២. Indexed Array

Indexed Array ប្រើលេខជាគន្លឹះ (Key)។ Index របស់ Array ដែលបង្កើតដោយស្វ័យប្រវត្តិ ជាទូទៅចាប់ផ្តើមពី 0។

### ២.១ បង្កើត Array

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

print_r($fruits);
```

លទ្ធផល៖

```text
Array
(
    [0] => Apple
    [1] => Banana
    [2] => Orange
)
```

ក៏អាចប្រើ `array()` បានដែរ៖

```php
<?php
$numbers = array(10, 20, 30);
```

ទម្រង់ `[]` គឺជាទម្រង់ខ្លី និងពេញនិយម។

### ២.២ ចូលប្រើធាតុដោយ Index

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

echo $fruits[0] . PHP_EOL;
echo $fruits[1] . PHP_EOL;
echo $fruits[2] . PHP_EOL;
```

លទ្ធផល៖

```text
Apple
Banana
Orange
```

Index ដំបូងគឺ 0 មិនមែន 1 ទេ។ ការចូលប្រើ Index ដែលមិនមាន អាចបង្កើត Warning ឬលទ្ធផលមិនដូចការរំពឹងទុក ដូច្នេះគួរពិនិត្យមុន។

```php
<?php
$fruits = ["Apple", "Banana"];

if (isset($fruits[2])) {
    echo $fruits[2];
} else {
    echo "មិនមានធាតុនៅ Index 2";
}
```

### ២.៣ កែប្រែធាតុ

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

$fruits[1] = "Mango";

print_r($fruits);
```

លទ្ធផល៖

```text
Array
(
    [0] => Apple
    [1] => Mango
    [2] => Orange
)
```

### ២.៤ បន្ថែមធាតុ

ប្រើ `[]` ដើម្បីបន្ថែមធាតុថ្មីទៅចុង Array៖

```php
<?php
$fruits = ["Apple", "Banana"];

$fruits[] = "Mango";
$fruits[] = "Orange";

print_r($fruits);
```

លទ្ធផល៖

```text
Array
(
    [0] => Apple
    [1] => Banana
    [2] => Mango
    [3] => Orange
)
```

អ្នកក៏អាចប្រើ `array_push()` ដើម្បីបន្ថែមធាតុមួយ ឬច្រើន៖

```php
<?php
$numbers = [1, 2];

array_push($numbers, 3, 4);

print_r($numbers);
```

### ២.៥ លុបធាតុ

ប្រើ `unset()` ដើម្បីលុបធាតុតាម Key៖

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

unset($fruits[1]);

print_r($fruits);
```

លទ្ធផលនឹងមាន Key 0 និង 2។ `unset()` មិនរៀបចំ Index ឡើងវិញដោយស្វ័យប្រវត្តិទេ។

ប្រសិនបើចង់រៀបចំ Index ជាលេខជាប់គ្នាឡើងវិញ អាចប្រើ `array_values()`៖

```php
<?php
$fruits = ["Apple", "Banana", "Orange"];

unset($fruits[1]);
$fruits = array_values($fruits);

print_r($fruits);
```

លទ្ធផល៖

```text
Array
(
    [0] => Apple
    [1] => Orange
)
```

## ៣. Associative Array

Associative Array ប្រើ Key ជាអក្សរ ឬ String ដើម្បីកំណត់អត្តសញ្ញាណតម្លៃ។ វាសមស្របសម្រាប់ទិន្នន័យដែលមានឈ្មោះវាល ដូចជា Student ឬ User។

```php
<?php
$student = [
    "name" => "Dara",
    "age" => 20,
    "major" => "PHP"
];

echo $student["name"] . PHP_EOL;
echo $student["age"] . PHP_EOL;
echo $student["major"] . PHP_EOL;
```

លទ្ធផល៖

```text
Dara
20
PHP
```

### ៣.១ បន្ថែម និងកែប្រែ Key

```php
<?php
$student = [
    "name" => "Dara",
    "age" => 20
];

$student["city"] = "Phnom Penh"; // បន្ថែម Key ថ្មី
$student["age"] = 21;            // កែប្រែ Key ចាស់

print_r($student);
```

### ៣.២ ពិនិត្យថា Key មានឬអត់

ប្រើ `array_key_exists()` ដើម្បីពិនិត្យថា Key មាននៅក្នុង Array ឬអត់។ វាអាចត្រឡប់ `true` ទោះតម្លៃរបស់ Key នោះជា `null` ក៏ដោយ។

```php
<?php
$user = [
    "name" => "Sokha",
    "email" => null
];

var_dump(array_key_exists("email", $user));
var_dump(isset($user["email"]));
```

លទ្ធផល៖

```text
bool(true)
bool(false)
```

ភាពខុសគ្នា៖
- `array_key_exists()` ពិនិត្យថា Key មានឬអត់។
- `isset()` ត្រឡប់ false ប្រសិនបើ Key មិនមាន ឬតម្លៃជា `null`។

## ៤. Multidimensional Array

Multidimensional Array គឺ Array ដែលមាន Array ផ្សេងទៀតជាធាតុខាងក្នុង។ វាមានប្រយោជន៍សម្រាប់ទិន្នន័យជាក្រុម ដូចជា បញ្ជីសិស្សដែលម្នាក់ៗមានឈ្មោះ និងពិន្ទុ។

```php
<?php
$students = [
    [
        "name" => "Dara",
        "score" => 90
    ],
    [
        "name" => "Sokha",
        "score" => 85
    ],
    [
        "name" => "Malis",
        "score" => 95
    ]
];

echo $students[0]["name"] . PHP_EOL;
echo $students[1]["score"] . PHP_EOL;
```

លទ្ធផល៖

```text
Dara
85
```

ដើម្បីបង្ហាញសិស្សទាំងអស់ ប្រើ Nested `foreach`៖

```php
<?php
foreach ($students as $student) {
    echo "Name: " . $student["name"] . PHP_EOL;
    echo "Score: " . $student["score"] . PHP_EOL;
    echo "--------" . PHP_EOL;
}
```

## ៥. Loop ជាមួយ Array

`foreach` ជាវិធីងាយស្រួលបំផុតសម្រាប់អានធាតុទាំងអស់ក្នុង Array។

### ៥.១ អានតម្លៃ

```php
<?php
$colors = ["Red", "Green", "Blue"];

foreach ($colors as $color) {
    echo $color . PHP_EOL;
}
```

### ៥.២ អាន Key និង Value

```php
<?php
$prices = [
    "Book" => 5,
    "Pen" => 2,
    "Bag" => 15
];

foreach ($prices as $product => $price) {
    echo "{$product}: ${$price}" . PHP_EOL;
}
```

លទ្ធផល៖

```text
Book: $5
Pen: $2
Bag: $15
```

## ៦. Array Functions ដែលប្រើញឹកញាប់

PHP មាន Built-in Array Functions ជាច្រើន។ ខាងក្រោមជាមុខងារសំខាន់ៗដែលគួរចេះ។

### ៦.១ count() — រាប់ចំនួនធាតុ

```php
<?php
$items = ["Pen", "Book", "Bag"];

echo count($items) . PHP_EOL;
```

លទ្ធផល៖ `3`

### ៦.២ in_array() — ពិនិត្យតម្លៃមានក្នុង Array ឬអត់

```php
<?php
$roles = ["admin", "editor", "user"];

var_dump(in_array("admin", $roles, true));
var_dump(in_array("guest", $roles, true));
```

លទ្ធផល៖

```text
bool(true)
bool(false)
```

Argument ទីបី `true` បើក Strict Checking ដើម្បីឱ្យពិនិត្យប្រភេទទិន្នន័យផងដែរ។

### ៦.៣ array_merge() — បញ្ចូល Array

```php
<?php
$first = ["Red", "Green"];
$second = ["Blue", "Yellow"];

$allColors = array_merge($first, $second);

print_r($allColors);
```

### ៦.៤ array_unique() — ដកតម្លៃស្ទួន

```php
<?php
$numbers = [1, 2, 2, 3, 3, 3];

$uniqueNumbers = array_unique($numbers);

print_r($uniqueNumbers);
```

មុខងារនេះរក្សា Key ដើម។ ប្រសិនបើចង់បាន Index ជាប់គ្នា អាចប្រើ៖

```php
$uniqueNumbers = array_values(array_unique($numbers));
```

### ៦.៥ sort() និង rsort()

`sort()` រៀបតម្លៃតាមលំដាប់កើន និងរៀបចំ Key ជាលេខឡើងវិញ។ `rsort()` រៀបតាមលំដាប់ថយ។

```php
<?php
$numbers = [5, 2, 9, 1];

sort($numbers);
print_r($numbers);

rsort($numbers);
print_r($numbers);
```

### ៦.៦ asort() និង ksort()

សម្រាប់ Associative Array៖
- `asort()` រៀបតាម Value ហើយរក្សា Key។
- `ksort()` រៀបតាម Key។

```php
<?php
$scores = [
    "Dara" => 80,
    "Malis" => 95,
    "Sokha" => 85
];

asort($scores);
print_r($scores);

ksort($scores);
print_r($scores);
```

មុខងាររៀបលំដាប់ Array ទាំងនេះកែប្រែ Array ដើម (in-place)។

### ៦.៧ array_map() — បម្លែងធាតុមួយៗ

```php
<?php
$numbers = [1, 2, 3, 4];

$doubled = array_map(
    fn($number) => $number * 2,
    $numbers
);

print_r($doubled);
```

លទ្ធផល៖ `[2, 4, 6, 8]` (បង្ហាញជាទម្រង់ Array)។

### ៦.៨ array_filter() — ជ្រើសធាតុតាមលក្ខខណ្ឌ

```php
<?php
$numbers = [1, 2, 3, 4, 5, 6];

$evenNumbers = array_filter(
    $numbers,
    fn($number) => $number % 2 === 0
);

print_r($evenNumbers);
```

`array_filter()` រក្សា Key ដើម។ បើចង់រៀប Index ឡើងវិញ ប្រើ `array_values()`។

### ៦.៩ array_reduce() — បង្រួម Array ទៅជាតម្លៃតែមួយ

```php
<?php
$numbers = [10, 20, 30];

$total = array_reduce(
    $numbers,
    fn($carry, $number) => $carry + $number,
    0
);

echo $total . PHP_EOL;
```

លទ្ធផល៖ `60`

`$carry` រក្សាលទ្ធផលបណ្តោះអាសន្ន ហើយ `$number` ជាធាតុបច្ចុប្បន្ន។

### ៦.១០ array_column() — យក Column ពី Array ខាងក្នុង

```php
<?php
$users = [
    ["id" => 1, "name" => "Dara"],
    ["id" => 2, "name" => "Sokha"],
    ["id" => 3, "name" => "Malis"]
];

$names = array_column($users, "name");

print_r($names);
```

លទ្ធផល៖ Array ដែលមានឈ្មោះ Dara, Sokha និង Malis។

### ៦.១១ implode() និង explode()

- `implode()` បម្លែង Array ទៅជា String។
- `explode()` បំបែក String ទៅជា Array តាម Separator។

```php
<?php
$colors = ["Red", "Green", "Blue"];

$text = implode(", ", $colors);
echo $text . PHP_EOL;

$parts = explode(", ", "PHP, JavaScript, Python");
print_r($parts);
```

លទ្ធផលដំបូង៖

```text
Red, Green, Blue
```

## ៧. Destructuring Array

អ្នកអាចដកតម្លៃពី Array ទៅក្នុងអថេរច្រើនក្នុងពេលតែមួយ ដោយប្រើ `list()` ឬទម្រង់ `[]`។

```php
<?php
$person = ["Dara", 20, "Cambodia"];

[$name, $age, $country] = $person;

echo $name . PHP_EOL;
echo $age . PHP_EOL;
echo $country . PHP_EOL;
```

លទ្ធផល៖

```text
Dara
20
Cambodia
```

សម្រាប់ Associative Array អ្នកអាចប្រើ Key៖

```php
<?php
$user = [
    "name" => "Malis",
    "email" => "malis@example.com"
];

["name" => $name, "email" => $email] = $user;

echo $name . PHP_EOL;
echo $email . PHP_EOL;
```

## ៨. Spread Operator (...) ក្នុង Array

Spread Operator អាចយកធាតុពី Array មួយទៅដាក់ក្នុង Array ថ្មី។

```php
<?php
$frontend = ["HTML", "CSS"];
$backend = ["PHP", "MySQL"];

$skills = [...$frontend, ...$backend];

print_r($skills);
```

លទ្ធផល៖ Array មាន HTML, CSS, PHP និង MySQL។

Spread Operator ក្នុង Array អាចប្រើជាមួយ Numeric Keys ហើយក៏អាចប្រើជាមួយ String Keys ក្នុង PHP 8.1 ឡើងទៅ។ ប្រសិនបើ String Key ដូចគ្នា តម្លៃក្រោយអាចជំនួសតម្លៃមុន។

```php
<?php
$defaults = ["theme" => "light", "language" => "km"];
$settings = [...$defaults, "theme" => "dark"];

print_r($settings);
```

## ៩. Array Keys និងការពិនិត្យប្រភេទ Array

PHP Array ជា Ordered Map មានន័យថាវារក្សាទុកគូ Key និង Value តាមលំដាប់បញ្ចូល។ Key អាចជា Integer ឬ String។ Key មួយមិនអាចមានតម្លៃពីរដាច់ដោយឡែកក្នុង Array តែមួយបានទេ; ការកំណត់ Key ដដែលម្តងទៀតនឹងជំនួស Value ចាស់។

```php
<?php
$data = [
    "name" => "Dara",
    "name" => "Sokha"
];

echo $data["name"] . PHP_EOL;
```

លទ្ធផល៖ `Sokha`

PHP 8.1 ឡើងទៅមាន `array_is_list()` សម្រាប់ពិនិត្យថា Array ជា List ដែលមាន Key ជាលេខចាប់ពី 0 ជាប់គ្នាឬអត់៖

```php
<?php
var_dump(array_is_list(["A", "B", "C"]));
var_dump(array_is_list([1 => "A", 2 => "B"]));
```

លទ្ធផល៖

```text
bool(true)
bool(false)
```

## ១០. Mini Project៖ កម្មវិធីគ្រប់គ្រងបញ្ជីសិស្ស

គោលដៅ៖ រក្សាទុកទិន្នន័យសិស្ស បង្ហាញបញ្ជី គណនាមធ្យមភាគ និងស្វែងរកសិស្សតាមឈ្មោះ។

```php
<?php
declare(strict_types=1);

$students = [
    [
        "id" => 1,
        "name" => "Dara",
        "scores" => [80, 90, 85]
    ],
    [
        "id" => 2,
        "name" => "Sokha",
        "scores" => [70, 75, 80]
    ],
    [
        "id" => 3,
        "name" => "Malis",
        "scores" => [95, 90, 100]
    ]
];

function calculateAverage(array $scores): float {
    if (count($scores) === 0) {
        return 0.0;
    }

    return array_sum($scores) / count($scores);
}

function findStudentByName(array $students, string $searchName): ?array {
    foreach ($students as $student) {
        if (strcasecmp($student["name"], $searchName) === 0) {
            return $student;
        }
    }

    return null;
}

// បង្ហាញបញ្ជីសិស្ស
echo "=== Student List ===" . PHP_EOL;

foreach ($students as $student) {
    $average = calculateAverage($student["scores"]);

    echo "ID: " . $student["id"] . PHP_EOL;
    echo "Name: " . $student["name"] . PHP_EOL;
    echo "Average: " . number_format($average, 2) . PHP_EOL;
    echo "-------------------" . PHP_EOL;
}

// ស្វែងរកសិស្ស
$foundStudent = findStudentByName($students, "Sokha");

echo "=== Search Result ===" . PHP_EOL;

if ($foundStudent !== null) {
    echo "Found: " . $foundStudent["name"] . PHP_EOL;
    echo "Average: " .
        number_format(calculateAverage($foundStudent["scores"]), 2) .
        PHP_EOL;
} else {
    echo "Student not found" . PHP_EOL;
}
```

លទ្ធផល៖

```text
=== Student List ===
ID: 1
Name: Dara
Average: 85.00
-------------------
ID: 2
Name: Sokha
Average: 75.00
-------------------
ID: 3
Name: Malis
Average: 95.00
-------------------
=== Search Result ===
Found: Sokha
Average: 75.00
```

ចំណុចដែលបានអនុវត្ត៖
- Array ច្រើនជាន់សម្រាប់រក្សាទុកសិស្ស។
- `foreach` សម្រាប់បង្ហាញទិន្នន័យ។
- `array_sum()` និង `count()` សម្រាប់គណនាមធ្យមភាគ។
- Function `findStudentByName()` សម្រាប់ស្វែងរក។
- Return Type `?array` មានន័យថា Function អាចត្រឡប់ Array ឬ `null`។

## ១១. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ភ្លេចថា Index ចាប់ផ្តើមពី 0**៖ ធាតុទីមួយគឺ `$array[0]`។
2. **ប្រើ Key មិនត្រឹមត្រូវ**៖ Associative Array ត្រូវប្រើ Key ដែលមានស្រាប់។
3. **សន្មតថា unset() រៀប Index ឡើងវិញ**៖ វាលុបធាតុ ប៉ុន្តែមិនរៀប Numeric Keys ឡើងវិញទេ។
4. **ច្រឡំ isset() និង array_key_exists()**៖ `isset()` ត្រឡប់ false ប្រសិនបើ Value ជា null។
5. **ភ្លេចថា array_filter() និង array_unique() អាចរក្សា Key ចាស់**៖ ប្រើ `array_values()` ប្រសិនបើត្រូវការ Index ជាប់គ្នា។
6. **ប្រើ Array ដែលមិនមានធាតុ**៖ ពិនិត្យ `count()` ឬប្រើ Default Value មុនគណនា។
7. **ច្រឡំ sort() និង asort()**៖ `sort()` រៀប Numeric Keys ឡើងវិញ; `asort()` រក្សា Key។

## ១២. លំហាត់អនុវត្ត

សូមសាកល្បងសរសេរកូដដោយខ្លួនឯង មុនពេលមើលចម្លើយនៅផ្នែកខាងក្រោម។

### លំហាត់ទី ១៖ Indexed Array
បង្កើត Array ឈ្មោះ `$animals` ដែលមាន Cat, Dog និង Bird។ បង្ហាញធាតុទីពីរ។

### លំហាត់ទី ២៖ បន្ថែម និងលុប
បង្កើត Array មានលេខ 10, 20, 30។ បន្ថែម 40 ហើយលុបធាតុដែលមានតម្លៃ 20។ បន្ទាប់មករៀប Index ឡើងវិញ។

### លំហាត់ទី ៣៖ Associative Array
បង្កើត Array សម្រាប់ Product ដែលមាន name, price និង quantity។ បង្ហាញព័ត៌មានទាំងអស់។

### លំហាត់ទី ៤៖ Filter លេខគូ
ប្រើ `array_filter()` ដើម្បីជ្រើសលេខគូពី `[1, 2, 3, 4, 5, 6, 7, 8]`។

### លំហាត់ទី ៥៖ គណនាផលបូក
ប្រើ `array_reduce()` ឬ `array_sum()` ដើម្បីគណនាផលបូកនៃ `[5, 10, 15, 20]`។

### លំហាត់ទី ៦៖ ស្វែងរកសិស្ស
បង្កើត Array សិស្ស ៣ នាក់ដែលមាន name និង score។ ប្រើ `foreach` ស្វែងរកសិស្សដែលមានពិន្ទុចាប់ពី 80 ឡើងទៅ។

### លំហាត់ទី ៧៖ Mini Project
បង្កើតកម្មវិធី Product Cart ដែលមាន Product យ៉ាងតិច ៣។ Product នីមួយៗមាន name, price និង quantity។ គណនាតម្លៃសរុបនៃទំនិញទាំងអស់។

## ១៣. សំណួរត្រួតពិនិត្យចំណេះដឹង

1. តើ Array មានអត្ថប្រយោជន៍អ្វី?
2. តើ Indexed Array ខុសពី Associative Array ដូចម្តេច?
3. តើ Multidimensional Array ជាអ្វី?
4. តើ `unset()` ធ្វើអ្វី?
5. តើ `isset()` និង `array_key_exists()` ខុសគ្នាដូចម្តេច?
6. តើ `array_map()` និង `array_filter()` មានតួនាទីអ្វី?
7. តើ `array_reduce()` ប្រើសម្រាប់អ្វី?
8. តើ `implode()` និង `explode()` ខុសគ្នាដូចម្តេច?

---

## ១៤. ចម្លើយគំរូ (មើលក្រោយពេលសាកល្បងរួច)

### ចម្លើយលំហាត់ទី ១

```php
<?php
$animals = ["Cat", "Dog", "Bird"];

echo $animals[1] . PHP_EOL;
```

លទ្ធផល៖ `Dog`

### ចម្លើយលំហាត់ទី ២

```php
<?php
$numbers = [10, 20, 30];

$numbers[] = 40;

unset($numbers[1]);

$numbers = array_values($numbers);

print_r($numbers);
```

លទ្ធផល៖ `[10, 30, 40]`

### ចម្លើយលំហាត់ទី ៣

```php
<?php
$product = [
    "name" => "Keyboard",
    "price" => 25.50,
    "quantity" => 2
];

foreach ($product as $key => $value) {
    echo $key . ": " . $value . PHP_EOL;
}
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$numbers = [1, 2, 3, 4, 5, 6, 7, 8];

$even = array_values(array_filter(
    $numbers,
    fn($number) => $number % 2 === 0
));

print_r($even);
```

លទ្ធផល៖ `[2, 4, 6, 8]`

### ចម្លើយលំហាត់ទី ៥

```php
<?php
$numbers = [5, 10, 15, 20];

echo array_sum($numbers) . PHP_EOL;
```

លទ្ធផល៖ `50`

### ចម្លើយលំហាត់ទី ៦

```php
<?php
$students = [
    ["name" => "Dara", "score" => 85],
    ["name" => "Sokha", "score" => 70],
    ["name" => "Malis", "score" => 95]
];

foreach ($students as $student) {
    if ($student["score"] >= 80) {
        echo $student["name"] . ": " .
            $student["score"] . PHP_EOL;
    }
}
```

លទ្ធផល៖

```text
Dara: 85
Malis: 95
```

### ចម្លើយលំហាត់ទី ៧

```php
<?php
$cart = [
    ["name" => "Keyboard", "price" => 25, "quantity" => 1],
    ["name" => "Mouse", "price" => 10, "quantity" => 2],
    ["name" => "Monitor", "price" => 120, "quantity" => 1]
];

$total = 0;

foreach ($cart as $item) {
    $subtotal = $item["price"] * $item["quantity"];
    $total += $subtotal;

    echo $item["name"] . ": $" .
        number_format($subtotal, 2) . PHP_EOL;
}

echo "Total: $" . number_format($total, 2) . PHP_EOL;
```

លទ្ធផល៖

```text
Keyboard: $25.00
Mouse: $20.00
Monitor: $120.00
Total: $165.00
```

### ចម្លើយសំណួរត្រួតពិនិត្យ

1. រក្សាទុក និងគ្រប់គ្រងតម្លៃច្រើនក្នុងអថេរតែមួយ។
2. Indexed Array ប្រើ Numeric Keys; Associative Array ប្រើ String Keys។
3. Array ដែលមាន Array ផ្សេងទៀតនៅខាងក្នុង។
4. លុបធាតុ ឬអថេរដែលបានកំណត់។
5. `isset()` ពិនិត្យថា Key មាន និង Value មិនមែន null; `array_key_exists()` ពិនិត្យតែថា Key មាន។
6. `array_map()` បម្លែងធាតុ; `array_filter()` ជ្រើសធាតុតាមលក្ខខណ្ឌ។
7. បង្រួមធាតុ Array ទៅជាលទ្ធផលតែមួយ ដូចជាផលបូក។
8. `implode()` បម្លែង Array ទៅ String; `explode()` បំបែក String ទៅ Array។

## ១៥. របៀបដំណើរការកូដ

1. បង្កើតឯកសារ `lesson07.php`។
2. ចម្លងកូដឧទាហរណ៍ទៅក្នុងឯកសារ។
3. បើក Terminal ឬ Command Prompt នៅទីតាំងឯកសារ។
4. ដំណើរការ៖

```bash
php lesson07.php
```

ត្រូវប្រាកដថា PHP បានដំឡើង និងអាចប្រើពាក្យបញ្ជា `php` ក្នុង Terminal បាន។

## សង្ខេបមេរៀន

ក្នុងមេរៀននេះ អ្នកបានរៀន៖
- Indexed, Associative និង Multidimensional Arrays។
- បន្ថែម កែប្រែ និងលុបធាតុ។
- អាន Array ដោយប្រើ `foreach`។
- Array Functions ដូចជា `count()`, `array_map()`, `array_filter()`, `array_reduce()`, `array_column()`។
- Destructuring និង Spread Operator។
- បង្កើតកម្មវិធីគ្រប់គ្រងបញ្ជីសិស្ស និង Product Cart។

**ការងារមុនទៅមេរៀនបន្ទាប់៖** អនុវត្តលំហាត់ទាំង ៧ ហើយសាកល្បងបង្កើត Array ផ្ទាល់ខ្លួនដែលមានទិន្នន័យសិស្ស ឬផលិតផល។
