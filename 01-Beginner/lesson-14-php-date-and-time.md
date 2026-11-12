# មេរៀនទី ១៤៖ PHP Date and Time (កាលបរិច្ឆេទ និងពេលវេលា)

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពី Unix Timestamp និង Timezone។
- ប្រើ `date()`, `time()`, `strtotime()` និង `mktime()`។
- បង្កើត និងកែប្រែកាលបរិច្ឆេទដោយប្រើ `DateTime`។
- ប្រើ `DateTimeImmutable` ដើម្បីជៀសវាងការកែប្រែ Object ដើមដោយចៃដន្យ។
- ប្រៀបធៀបកាលបរិច្ឆេទ និងគណនារយៈពេល។
- រៀបចំការបញ្ចូល និងបង្ហាញកាលបរិច្ឆេទឱ្យត្រឹមត្រូវ។

> តម្រូវការ៖ ឧទាហរណ៍ប្រើ PHP 8.0 ឡើងទៅ។ សម្រាប់គម្រោងថ្មី គួរយល់ពី `DateTimeImmutable` និង DateTime API ជាជាងប្រើតែ Function ចាស់ៗ។

---

## ១. ហេតុអ្វីត្រូវរៀន Date and Time?

កម្មវិធីជាច្រើនត្រូវការគ្រប់គ្រងកាលបរិច្ឆេទ និងពេលវេលា៖

- បង្ហាញថ្ងៃខែឆ្នាំដែលអ្នកប្រើចុះឈ្មោះ។
- រក្សាទុកថ្ងៃបង្កើត Post ឬ Article។
- កំណត់ថ្ងៃផុតកំណត់ (Due Date)។
- គណនាអាយុ ឬរយៈពេល។
- កំណត់ពេលណាត់ជួប។
- បង្ហាញពេលវេលាតាម Timezone របស់អ្នកប្រើ។

PHP មាន Function និង Class ជាច្រើនសម្រាប់ការងារទាំងនេះ។

## ២. Timezone (តំបន់ពេលវេលា)

Timezone គឺជាតំបន់ដែលប្រើពេលវេលាស្តង់ដារដូចគ្នា។ ប្រទេសកម្ពុជាប្រើ Timezone `Asia/Phnom_Penh` ដែលមាន UTC+7។

កំណត់ Timezone សម្រាប់ Script៖

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

echo date('Y-m-d H:i:s');
```

លទ្ធផលនឹងអាស្រ័យលើពេលវេលាពិតនៅពេលដំណើរការ។

ពិនិត្យ Timezone ដែលបានកំណត់៖

```php
<?php
echo date_default_timezone_get();
```

លទ្ធផល៖

```text
Asia/Phnom_Penh
```

អ្នកអាចបង្កើត Timezone Object ដោយមិនចាំបាច់ប្ដូរ Timezone ទូទៅរបស់ Script៖

```php
<?php
$timezone = new DateTimeZone('Asia/Phnom_Penh');

$now = new DateTimeImmutable('now', $timezone);

echo $now->format('Y-m-d H:i:s');
```

**Best Practice៖** សម្រាប់កម្មវិធីដែលមានអ្នកប្រើច្រើនប្រទេស គួររក្សាទុកពេលវេលាជា UTC នៅក្នុង Database ហើយបម្លែងទៅ Timezone របស់អ្នកប្រើនៅពេលបង្ហាញ។

## ៣. Unix Timestamp និង time()

Unix Timestamp គឺជាចំនួនវិនាទីដែលបានរាប់ចាប់ពី Unix Epoch គឺថ្ងៃទី ១ ខែមករា ឆ្នាំ ១៩៧០ ម៉ោង 00:00:00 UTC។

ប្រើ `time()` ដើម្បីទទួល Timestamp បច្ចុប្បន្ន៖

```php
<?php
$timestamp = time();

echo $timestamp;
```

លទ្ធផលជាចំនួនគត់ ហើយវាផ្លាស់ប្ដូរតាមពេលវេលាដែលដំណើរការ។

បម្លែង Timestamp ទៅជាកាលបរិច្ឆេទ៖

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

$timestamp = time();

echo date('Y-m-d H:i:s', $timestamp);
```

បង្កើត Timestamp ពីកាលបរិច្ឆេទដែលបានកំណត់៖

```php
<?php
$timestamp = strtotime('2026-12-25 09:30:00');

echo $timestamp;
```

ចំណាំ៖ `strtotime()` អាចត្រឡប់ `false` ប្រសិនបើមិនអាចបកស្រាយអត្ថបទបាន។ កុំសន្មតថាវាតែងតែទទួលបាន Timestamp ត្រឹមត្រូវ។

## ៤. date() — បង្ហាញកាលបរិច្ឆេទ

Syntax៖

```php
date(string $format, ?int $timestamp = null): string
```

ឧទាហរណ៍៖

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

echo date('Y-m-d');
echo "\n";
echo date('d/m/Y');
echo "\n";
echo date('H:i:s');
echo "\n";
echo date('l');
```

Format Characters ដែលប្រើញឹកញាប់៖

| Format | អត្ថន័យ | ឧទាហរណ៍ |
|---|---|---|
| `Y` | ឆ្នាំ ៤ ខ្ទង់ | 2026 |
| `y` | ឆ្នាំ ២ ខ្ទង់ | 26 |
| `m` | ខែ ២ ខ្ទង់ | 11 |
| `n` | ខែគ្មានសូន្យខាងមុខ | 11 |
| `d` | ថ្ងៃ ២ ខ្ទង់ | 12 |
| `j` | ថ្ងៃគ្មានសូន្យខាងមុខ | 12 |
| `H` | ម៉ោង 24-hour | 20 |
| `h` | ម៉ោង 12-hour | 08 |
| `i` | នាទី | 05 |
| `s` | វិនាទី | 09 |
| `A` | AM ឬ PM | PM |
| `l` | ឈ្មោះថ្ងៃពេញ | Thursday |
| `D` | ឈ្មោះថ្ងៃខ្លី | Thu |
| `F` | ឈ្មោះខែពេញ | November |
| `M` | ឈ្មោះខែខ្លី | Nov |

ឧទាហរណ៍ Format ផ្សេងៗ៖

```php
<?php
echo date('l, F j, Y');
echo "\n";
echo date('d-m-Y h:i A');
```

សញ្ញាដែលមិនមែនជា Format Character អាចដាក់ជាអក្សរធម្មតាបាន។ ប្រសិនបើចង់បង្ហាញ Format Character ជាអក្សរធម្មតា ត្រូវ Escape ដោយ Backslash។

```php
<?php
echo date('Y-m-d');       // 2026-11-12
echo "\n";
echo date('H:i:s');       // ម៉ោងបច្ចុប្បន្ន
echo "\n";
echo date('l, d F Y');    // ថ្ងៃ ខែ ឆ្នាំ
```

## ៥. strtotime() — បកស្រាយអត្ថបទកាលបរិច្ឆេទ

`strtotime()` បម្លែងអត្ថបទដែលពិពណ៌នាពេលវេលាទៅជា Unix Timestamp។

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

echo date('Y-m-d', strtotime('tomorrow'));
echo "\n";
echo date('Y-m-d', strtotime('next Monday'));
echo "\n";
echo date('Y-m-d', strtotime('+7 days'));
echo "\n";
echo date('Y-m-d', strtotime('-1 month'));
```

អាចប្រើអត្ថបទដូចជា៖

- `today`
- `tomorrow`
- `yesterday`
- `+3 days`
- `-2 weeks`
- `+1 month`
- `next Monday`

ប្រយ័ត្ន៖ ការបន្ថែមមួយខែទៅថ្ងៃចុងខែអាចមានលទ្ធផលដែលមិនដូចការរំពឹង ព្រោះចំនួនថ្ងៃក្នុងមួយខែមិនស្មើគ្នា។ សម្រាប់ Logic សំខាន់ៗ គួរប្រើ DateTime និងកំណត់ច្បាប់ច្បាស់លាស់។

## ៦. mktime() — បង្កើត Timestamp

`mktime()` បង្កើត Timestamp ពីម៉ោង នាទី វិនាទី ខែ ថ្ងៃ និងឆ្នាំ។

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

$timestamp = mktime(9, 30, 0, 12, 25, 2026);

echo date('Y-m-d H:i:s', $timestamp);
```

លទ្ធផល៖

```text
2026-12-25 09:30:00
```

សម្រាប់កូដថ្មី ជាទូទៅ `DateTimeImmutable` ងាយអាន និងគ្រប់គ្រងជាងការប្រើ `mktime()`។

## ៧. DateTime — បង្កើតកាលបរិច្ឆេទជា Object

`DateTime` ជា Class សម្រាប់បង្កើត កែប្រែ និងបង្ហាញកាលបរិច្ឆេទ។

```php
<?php
$date = new DateTime('2026-11-12');

echo $date->format('Y-m-d');
```

កំណត់ម៉ោង និង Timezone៖

```php
<?php
$timezone = new DateTimeZone('Asia/Phnom_Penh');

$date = new DateTime('2026-11-12 14:30:00', $timezone);

echo $date->format('Y-m-d H:i:s');
echo "\n";
echo $date->getTimezone()->getName();
```

កែប្រែកាលបរិច្ឆេទដោយ `modify()`៖

```php
<?php
$date = new DateTime('2026-11-12');

$date->modify('+7 days');

echo $date->format('Y-m-d');
```

លទ្ធផល៖

```text
2026-11-19
```

ចំណាំ៖ `DateTime` ជា Mutable Object។ មានន័យថា Method ដូចជា `modify()` កែប្រែ Object ដើមផ្ទាល់។

## ៨. DateTimeImmutable — ជម្រើសល្អសម្រាប់ការងារច្រើន

`DateTimeImmutable` មិនកែប្រែ Object ដើមទេ។ Method ដូចជា `modify()` ត្រឡប់ Object ថ្មីមកវិញ។

```php
<?php
$date1 = new DateTimeImmutable('2026-11-12');

$date2 = $date1->modify('+7 days');

echo $date1->format('Y-m-d');
echo "\n";
echo $date2->format('Y-m-d');
```

លទ្ធផល៖

```text
2026-11-12
2026-11-19
```

ចំណុចសំខាន់៖ ត្រូវរក្សាទុក Object ថ្មីពី Method Call៖

```php
<?php
$date = new DateTimeImmutable('2026-11-12');

// ត្រឹមត្រូវ
$date = $date->modify('+1 day');
```

បើអ្នកហៅ `$date->modify('+1 day');` ដោយមិនរក្សាទុកលទ្ធផល នោះ `$date` ដើមនៅដដែល។

## ៩. DateInterval — គណនារយៈពេល

`DateInterval` តំណាងឱ្យចន្លោះពេល។ អាចប្រើវាជាមួយ Method `add()` និង `sub()`។

```php
<?php
$date = new DateTimeImmutable('2026-11-12');

$interval = new DateInterval('P10D');

$newDate = $date->add($interval);

echo $newDate->format('Y-m-d');
```

លទ្ធផល៖

```text
2026-11-22
```

ទម្រង់ Interval ដែលប្រើញឹកញាប់៖

| Code | អត្ថន័យ |
|---|---|
| `P1D` | ១ ថ្ងៃ |
| `P7D` | ៧ ថ្ងៃ |
| `P2W` | ២ សប្ដាហ៍ |
| `P1M` | ១ ខែ |
| `P1Y` | ១ ឆ្នាំ |
| `PT2H` | ២ ម៉ោង |
| `PT30M` | ៣០ នាទី |
| `PT45S` | ៤៥ វិនាទី |
| `P1DT2H` | ១ ថ្ងៃ និង ២ ម៉ោង |

អក្សរ `P` ចាប់ផ្ដើមផ្នែក Date ហើយ `T` ចាប់ផ្ដើមផ្នែក Time។

## ១០. DateTime::diff() — គណនាភាពខុសគ្នារវាងកាលបរិច្ឆេទ

`diff()` ត្រឡប់ `DateInterval` មួយ។

```php
<?php
$start = new DateTimeImmutable('2026-11-12');
$end = new DateTimeImmutable('2026-12-25');

$diff = $start->diff($end);

echo $diff->days . " days";
```

លទ្ធផល៖

```text
43 days
```

ពិនិត្យថាកាលបរិច្ឆេទណាមុន ឬក្រោយ៖

```php
<?php
$today = new DateTimeImmutable('2026-11-12');
$deadline = new DateTimeImmutable('2026-11-20');

if ($deadline > $today) {
    echo "នៅសល់ពេលមុនថ្ងៃកំណត់";
} elseif ($deadline < $today) {
    echo "ហួសថ្ងៃកំណត់";
} else {
    echo "ថ្ងៃនេះជាថ្ងៃកំណត់";
}
```

ចំណាំ៖ `$diff->days` ជាចំនួនថ្ងៃសរុបនៅពេល PHP អាចគណនាបាន។ ចំណែក `y`, `m`, និង `d` បង្ហាញឆ្នាំ ខែ និងថ្ងៃតាមផ្នែក។

## ១១. បម្លែង Timezone

អាចបម្លែង DateTime Object ទៅ Timezone ផ្សេងដោយ `setTimezone()`។

```php
<?php
$utc = new DateTimeZone('UTC');
$cambodia = new DateTimeZone('Asia/Phnom_Penh');

$date = new DateTimeImmutable('2026-11-12 12:00:00', $utc);

echo $date->format('Y-m-d H:i:s T');
echo "\n";

$localDate = $date->setTimezone($cambodia);

echo $localDate->format('Y-m-d H:i:s T');
```

លទ្ធផល៖

```text
2026-11-12 12:00:00 UTC
2026-11-12 19:00:00 +07
```

ពេលវេលាតំណាងឱ្យ Instant ដដែល ប៉ុន្តែរបៀបបង្ហាញខុសគ្នាតាម Timezone។

## ១២. បកស្រាយកាលបរិច្ឆេទពី Form ឱ្យត្រឹមត្រូវ

នៅពេលទទួលកាលបរិច្ឆេទពី Form កុំជឿលើ Input ដោយផ្ទាល់។ ត្រូវពិនិត្យទម្រង់ និង Validity។

ឧទាហរណ៍៖ Form ផ្ញើថ្ងៃខែឆ្នាំជាទម្រង់ `Y-m-d`។

```php
<?php
$dateInput = $_POST['date'] ?? '';

if (!is_string($dateInput)) {
    exit('Input មិនត្រឹមត្រូវ');
}

$date = DateTimeImmutable::createFromFormat(
    '!Y-m-d',
    $dateInput,
    new DateTimeZone('Asia/Phnom_Penh')
);

$errors = DateTimeImmutable::getLastErrors();

if (
    $date === false ||
    ($errors !== false && ($errors['warning_count'] > 0 || $errors['error_count'] > 0)) ||
    $date->format('Y-m-d') !== $dateInput
) {
    echo "សូមបញ្ចូលកាលបរិច្ឆេទត្រឹមត្រូវ ដូចជា 2026-12-25";
} else {
    echo "កាលបរិច្ឆេទត្រឹមត្រូវ៖ " . $date->format('d/m/Y');
}
```

ចំណាំ៖ នៅ PHP 8.2 ឡើងទៅ `getLastErrors()` អាចត្រឡប់ `false` ប្រសិនបើគ្មាន Warning ឬ Error។ ហេតុនេះឧទាហរណ៍ខាងលើពិនិត្យករណីនេះផងដែរ។

## ១៣. Mini Project៖ គណនាចំនួនថ្ងៃនៅសល់ដល់ថ្ងៃកំណត់

បង្កើតឯកសារ `deadline.php`៖

```php
<?php
declare(strict_types=1);

date_default_timezone_set('Asia/Phnom_Penh');

$today = new DateTimeImmutable('today');

$deadlineInput = $_GET['deadline'] ?? '';

if (!is_string($deadlineInput) || $deadlineInput === '') {
    echo 'សូមបញ្ចូលថ្ងៃកំណត់ តាមទម្រង់ YYYY-MM-DD';
    exit;
}

$deadline = DateTimeImmutable::createFromFormat(
    '!Y-m-d',
    $deadlineInput,
    new DateTimeZone('Asia/Phnom_Penh')
);

$errors = DateTimeImmutable::getLastErrors();

if (
    $deadline === false ||
    ($errors !== false && ($errors['warning_count'] > 0 || $errors['error_count'] > 0)) ||
    $deadline->format('Y-m-d') !== $deadlineInput
) {
    http_response_code(400);
    echo 'ថ្ងៃកំណត់មិនត្រឹមត្រូវ';
    exit;
}

$difference = $today->diff($deadline);
$days = $difference->days;

if ($deadline < $today) {
    echo "ថ្ងៃកំណត់នេះបានកន្លងផុតហើយចំនួន {$days} ថ្ងៃ។";
} elseif ($deadline == $today) {
    echo "ថ្ងៃនេះជាថ្ងៃកំណត់!";
} else {
    echo "នៅសល់ {$days} ថ្ងៃដល់ថ្ងៃកំណត់។";
}
```

សាកល្បងនៅ Local Server៖

- `http://localhost/deadline.php?deadline=2026-12-25`
- `http://localhost/deadline.php?deadline=2026-11-01`
- `http://localhost/deadline.php?deadline=2026-11-12`
- `http://localhost/deadline.php?deadline=2026-02-30`

លទ្ធផលអាស្រ័យលើថ្ងៃបច្ចុប្បន្ននៅពេលកម្មវិធីដំណើរការ។

### កិច្ចការបន្ថែមសម្រាប់ Project

1. បង្កើត HTML Form ដែលមាន `<input type="date">`។
2. ប្រើ POST ជំនួស GET ដើម្បីផ្ញើកាលបរិច្ឆេទ។
3. បង្ហាញថ្ងៃកំណត់ជាទម្រង់ `d/m/Y`។
4. បង្ហាញសារផ្សេងគ្នាសម្រាប់ថ្ងៃកន្លងផុត ថ្ងៃនេះ និងថ្ងៃអនាគត។
5. បន្ថែមការការពារ Input មិនត្រឹមត្រូវ។

## ១៤. កំហុសដែលអ្នកចាប់ផ្ដើមរៀនជួបញឹកញាប់

- **មិនបានកំណត់ Timezone៖** ពេលវេលាដែលបង្ហាញអាចខុសពីតំបន់របស់អ្នក។
- **ច្រឡំ `m` និង `i`៖** `m` គឺខែ ចំណែក `i` គឺនាទី។
- **ច្រឡំ `d` និង `D`៖** `d` គឺថ្ងៃជាលេខ ចំណែក `D` គឺឈ្មោះថ្ងៃខ្លី។
- **មិនពិនិត្យលទ្ធផល `strtotime()`៖** វាអាចត្រឡប់ `false`។
- **មិនរក្សាទុកលទ្ធផល DateTimeImmutable៖** Object ដើមមិនត្រូវបានកែប្រែទេ។
- **សន្មតថា ១ ខែស្មើ ៣០ ថ្ងៃ៖** ខែខ្លះមាន ២៨, ២៩ ឬ ៣១ ថ្ងៃ។
- **ប្រើ Timezone មិនស្របគ្នា៖** អាចធ្វើឱ្យថ្ងៃកំណត់ ឬការណាត់ជួបខុសពេល។
- **ទុកចិត្តលើ Form Input៖** ត្រូវ Validate នៅ Server ជានិច្ច។

## ១៥. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ បង្ហាញថ្ងៃបច្ចុប្បន្ន
ប្រើ `date()` ដើម្បីបង្ហាញថ្ងៃ ខែ ឆ្នាំ និងម៉ោងបច្ចុប្បន្នតាម Timezone កម្ពុជា។

### លំហាត់ទី ២៖ បន្ថែមថ្ងៃ
បង្កើត `DateTimeImmutable` សម្រាប់ថ្ងៃ 2026-11-12 ហើយបន្ថែម 30 ថ្ងៃ។ បង្ហាញលទ្ធផលជាទម្រង់ `Y-m-d`។

### លំហាត់ទី ៣៖ គណនាចំនួនថ្ងៃ
បង្កើតថ្ងៃចាប់ផ្ដើម 2026-01-01 និងថ្ងៃបញ្ចប់ 2026-12-31។ ប្រើ `diff()` ដើម្បីរកចំនួនថ្ងៃសរុប។

### លំហាត់ទី ៤៖ បម្លែង Timezone
បង្កើតពេលវេលា 2026-11-12 12:00:00 នៅ UTC ហើយបម្លែងទៅ `Asia/Phnom_Penh`។

### លំហាត់ទី ៥៖ Validate Date
សរសេរកូដទទួលកាលបរិច្ឆេទទម្រង់ `Y-m-d` ហើយពិនិត្យថាវាជាកាលបរិច្ឆេទពិតប្រាកដឬអត់។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង
ពន្យល់ភាពខុសគ្នារវាង `DateTime` និង `DateTimeImmutable`។

---

## ១៦. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```php
<?php
date_default_timezone_set('Asia/Phnom_Penh');

echo date('l, d F Y H:i:s');
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
$date = new DateTimeImmutable('2026-11-12');

$newDate = $date->modify('+30 days');

echo $newDate->format('Y-m-d');
```

លទ្ធផល៖

```text
2026-12-12
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
$start = new DateTimeImmutable('2026-01-01');
$end = new DateTimeImmutable('2026-12-31');

$diff = $start->diff($end);

echo $diff->days;
```

លទ្ធផល៖

```text
364
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$utc = new DateTimeZone('UTC');
$khmerTimezone = new DateTimeZone('Asia/Phnom_Penh');

$date = new DateTimeImmutable('2026-11-12 12:00:00', $utc);
$local = $date->setTimezone($khmerTimezone);

echo $local->format('Y-m-d H:i:s T');
```

លទ្ធផល៖

```text
2026-11-12 19:00:00 +07
```

### ចម្លើយលំហាត់ទី ៥

```php
<?php
$input = '2026-02-30';

$date = DateTimeImmutable::createFromFormat(
    '!Y-m-d',
    $input
);

$errors = DateTimeImmutable::getLastErrors();

$isValid = $date !== false
    && ($errors === false || (
        $errors['warning_count'] === 0
        && $errors['error_count'] === 0
    ))
    && $date->format('Y-m-d') === $input;

echo $isValid ? 'ត្រឹមត្រូវ' : 'មិនត្រឹមត្រូវ';
```

លទ្ធផល៖

```text
មិនត្រឹមត្រូវ
```

### ចម្លើយលំហាត់ទី ៦

- `DateTime` ជា Mutable Object។ ការហៅ Method កែប្រែ Object ដើម។
- `DateTimeImmutable` ជា Immutable Object។ Method បង្កើត Object ថ្មី ហើយទុក Object ដើមឱ្យនៅដដែល។

---

## សេចក្ដីសង្ខេប

- `date()` ប្រើសម្រាប់ Format និងបង្ហាញកាលបរិច្ឆេទ។
- `time()` ទទួល Unix Timestamp បច្ចុប្បន្ន។
- `strtotime()` បម្លែងអត្ថបទទៅ Timestamp។
- `DateTime` និង `DateTimeImmutable` ផ្ដល់វិធីគ្រប់គ្រងកាលបរិច្ឆេទតាម Object។
- `DateInterval` ប្រើសម្រាប់បន្ថែម ឬដករយៈពេល។
- `diff()` ប្រើសម្រាប់គណនាភាពខុសគ្នារវាងកាលបរិច្ឆេទ។
- ត្រូវគ្រប់គ្រង Timezone និង Validate កាលបរិច្ឆេទពីអ្នកប្រើជានិច្ច។

## មេរៀនបន្ទាប់

**មេរៀនទី ១៥៖ PHP JSON Handling** — រៀនអំពី JSON, `json_encode()`, `json_decode()`, ការគ្រប់គ្រង JSON Error និងការបង្កើតទិន្នន័យសម្រាប់ API។
