# មេរៀនទី ១៣៖ PHP Error Handling និង Exceptions

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Error, Warning, Notice និង Exception។
- ប្រើ `try`, `catch`, `finally`, និង `throw`។
- បង្កើត Custom Exception សម្រាប់ករណីពិសេស។
- ចាប់កំហុសដែលអាចកើតឡើងដោយសុវត្ថិភាព។
- កត់ត្រាកំហុស (logging) ដោយមិនបង្ហាញព័ត៌មានសម្ងាត់ដល់អ្នកប្រើ។
- រៀបចំ Error Handling សម្រាប់ Development និង Production។

> តម្រូវការ៖ ឧទាហរណ៍ក្នុងមេរៀននេះប្រើ PHP 8.0 ឡើងទៅ។ អ្នកអាចពិនិត្យកំណែ PHP ដោយប្រើ `php -v`។

---

## ១. តើ Error Handling ជាអ្វី?

**Error Handling** គឺជាវិធីដែលកម្មវិធីរកឃើញ ដោះស្រាយ និងកត់ត្រាបញ្ហាដែលកើតឡើងពេលដំណើរការ។ កម្មវិធីល្អមិនត្រឹមតែដំណើរការពេលអ្វីៗត្រឹមត្រូវទេ ប៉ុន្តែត្រូវមានផែនការពេលមានបញ្ហាផងដែរ។

ឧទាហរណ៍៖

- អ្នកប្រើបញ្ចូលអក្សរជំនួសលេខ។
- កម្មវិធីមិនអាចបើកឯកសារបាន។
- Database មិនអាចភ្ជាប់បាន។
- ការគណនាមានតម្លៃមិនត្រឹមត្រូវ។
- កូដហៅ Function ដែលមិនមាន។

គោលដៅគឺកុំឱ្យកម្មវិធីបិទដោយមិនបានគ្រប់គ្រង និងកុំបង្ហាញព័ត៌មានបច្ចេកទេសសម្ងាត់ទៅអ្នកប្រើ។

## ២. ប្រភេទកំហុសក្នុង PHP

### ២.១ Error

Error ជាបញ្ហាធ្ងន់ធ្ងរដែល PHP មិនអាចបន្តប្រតិបត្តិការតាមធម្មតាបាន។ ក្នុង PHP 7+ ប្រភេទជាច្រើនស្ថិតក្រោម `Error` ហើយអាចចាប់បានតាម `Throwable`។

ឧទាហរណ៍៖

```php
<?php
function greet(): void
{
    echo "Hello!";
}

greet("Sotheary"); // ArgumentCountError
```

Function ខាងលើមិនទទួល argument ទេ ប៉ុន្តែយើងបានផ្ញើមួយ។

### ២.២ Warning និង Notice

Warning និង Notice ជា PHP errors ដែលជាទូទៅមិនមែនជា Exception ទេ។ ឧទាហរណ៍៖ ការបើកឯកសារដែលមិនមានអាចបង្ក Warning។

```php
<?php
$file = fopen("missing.txt", "r");
```

កុំសន្មតថា `try/catch` នឹងចាប់ Warning ទាំងអស់ដោយស្វ័យប្រវត្តិ។ ត្រូវពិនិត្យលទ្ធផល Function និងប្រើ Error Handler តែនៅពេលមានហេតុផលច្បាស់លាស់។

### ២.៣ Exception

Exception គឺជាស្ថានភាពពិសេសដែលកូដអាចបោះចេញដោយប្រើ `throw` ហើយយើងអាចចាប់វាដោយ `catch`។

```php
<?php
throw new Exception("មានបញ្ហាមួយកើតឡើង");
```

កូដខាងលើបោះ Exception។ បើគ្មានកន្លែងចាប់វា កម្មវិធីអាចបញ្ចប់ដោយបង្ហាញ Uncaught Exception។

## ៣. ប្រើ try និង catch

- `try`៖ ដាក់កូដដែលអាចបង្ក Exception។
- `catch`៖ ចាប់ និងដោះស្រាយ Exception។
- `finally`៖ កូដដែលត្រូវដំណើរការនៅចុងក្រោយ មិនថាមាន Exception ឬអត់ទេ។

```php
<?php
try {
    $number = 10;
    $result = $number / 2;

    echo "លទ្ធផល៖ " . $result;
} catch (Throwable $e) {
    echo "មានបញ្ហាកើតឡើង។";
} finally {
    echo "\nបញ្ចប់ការគណនា។";
}
```

លទ្ធផល៖

```text
លទ្ធផល៖ 5
បញ្ចប់ការគណនា។
```

ឧទាហរណ៍បង្ក Exception ដោយចេតនា៖

```php
<?php
try {
    $age = 15;

    if ($age < 18) {
        throw new Exception("ត្រូវមានអាយុចាប់ពី 18 ឆ្នាំឡើងទៅ");
    }

    echo "អាចបន្តបាន";
} catch (Exception $e) {
    echo "សារ៖ " . $e->getMessage();
}
```

លទ្ធផល៖

```text
សារ៖ ត្រូវមានអាយុចាប់ពី 18 ឆ្នាំឡើងទៅ
```

## ៤. ប្រើ throw ដើម្បីបោះ Exception

យើងប្រើ `throw` ពេលលក្ខខណ្ឌណាមួយមិនអនុញ្ញាតឱ្យបន្ត។

```php
<?php
function divide(float $a, float $b): float
{
    if ($b === 0.0) {
        throw new InvalidArgumentException("មិនអាចចែកនឹងសូន្យបានទេ");
    }

    return $a / $b;
}

try {
    echo divide(20, 0);
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}
```

លទ្ធផល៖

```text
មិនអាចចែកនឹងសូន្យបានទេ
```

ចំណាំ៖ PHP ក៏មាន DivisionByZeroError សម្រាប់ប្រតិបត្តិការចែកនឹងសូន្យមួយចំនួនដែរ។ ក្នុងឧទាហរណ៍នេះ យើងពិនិត្យមុន ដើម្បីផ្តល់សារដែលងាយយល់។

## ៥. ចាប់ Exception ច្រើនប្រភេទ

អាចដាក់ `catch` ច្រើន ដើម្បីដោះស្រាយកំហុសតាមប្រភេទ៖

```php
<?php
try {
    $value = "abc";

    if (!is_numeric($value)) {
        throw new InvalidArgumentException("តម្លៃត្រូវតែជាលេខ");
    }

    echo (float) $value * 2;
} catch (InvalidArgumentException $e) {
    echo "ទិន្នន័យមិនត្រឹមត្រូវ៖ " . $e->getMessage();
} catch (Throwable $e) {
    echo "មានបញ្ហាបច្ចេកទេសមិនរំពឹងទុក។";
}
```

ដាក់ Catch ដែលជាក់លាក់ជាងមុននៅខាងលើ និង Catch ទូលំទូលាយនៅខាងក្រោម។

## ៦. Exception និង Throwable

ក្នុង PHP 7+៖

- `Throwable` ជា Interface មូលដ្ឋានសម្រាប់អ្វីៗដែលអាច `throw` បាន។
- `Exception` ជា Base Class សម្រាប់ Exceptions។
- `Error` ជា Base Class សម្រាប់ PHP engine errors ដូចជា `TypeError` និង `ValueError`។

ដូច្នេះ `catch (Throwable $e)` អាចចាប់ទាំង Exception និង Error។ ប៉ុន្តែកុំប្រើវាដើម្បីលាក់កំហុសទាំងអស់ដោយគ្មាន Logging ឬដំណោះស្រាយត្រឹមត្រូវ។

```php
<?php
function setAge(int $age): int
{
    return $age;
}

try {
    setAge("មិនមែនជាលេខ");
} catch (TypeError $e) {
    echo "ប្រភេទទិន្នន័យមិនត្រឹមត្រូវ។";
}
```

## ៧. បង្កើត Custom Exception

Custom Exception គឺជា Class ដែលយើងបង្កើតដោយខ្លួនឯង ដើម្បីបង្ហាញប្រភេទបញ្ហាជាក់លាក់របស់កម្មវិធី។

```php
<?php
class InsufficientBalanceException extends Exception
{
}

function withdraw(float $balance, float $amount): float
{
    if ($amount <= 0) {
        throw new InvalidArgumentException("ចំនួនដកត្រូវធំជាងសូន្យ");
    }

    if ($amount > $balance) {
        throw new InsufficientBalanceException("សមតុល្យមិនគ្រប់គ្រាន់");
    }

    return $balance - $amount;
}

try {
    echo withdraw(100, 150);
} catch (InsufficientBalanceException $e) {
    echo $e->getMessage();
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}
```

លទ្ធផល៖

```text
សមតុល្យមិនគ្រប់គ្រាន់
```

## ៨. ការកំណត់ Error Reporting

ពេលអភិវឌ្ឍកម្មវិធី យើងចង់ឃើញ Error ដើម្បីកែសម្រួល។ PHP មានការកំណត់ដូចជា៖

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', '1'); // Development ប៉ុណ្ណោះ
```

**សំខាន់៖** កុំបើក `display_errors` នៅលើ Production Server ព្រោះវាអាចបង្ហាញ File Paths, SQL details ឬព័ត៌មានសម្ងាត់។ នៅ Production គួរបិទការបង្ហាញកំហុស និងកត់ត្រាទៅ Log។

ឧទាហរណ៍សម្រាប់ Production៖

```php
<?php
error_reporting(E_ALL);
ini_set('display_errors', '0');
ini_set('log_errors', '1');
```

ការកំណត់នៅក្នុង PHP configuration របស់ Server ក៏សំខាន់ដែរ។ កុំសន្មតថា `ini_set()` អាចប្ដូរគ្រប់ Setting នៅគ្រប់ Hosting។

## ៩. កត់ត្រាកំហុសដោយ error_log()

`error_log()` អាចសរសេរសារ Error ទៅកាន់ PHP Error Log ដែលបានកំណត់លើ Server។

```php
<?php
try {
    throw new RuntimeException("Database connection failed");
} catch (Throwable $e) {
    error_log(
        "Application error: " . $e->getMessage()
    );

    echo "សូមព្យាយាមម្ដងទៀតនៅពេលក្រោយ។";
}
```

អ្នកប្រើឃើញតែសារទូទៅ ខណៈព័ត៌មានលម្អិតត្រូវបានកត់ត្រាសម្រាប់អ្នកអភិវឌ្ឍន៍។

កុំកត់ត្រា Password, Access Token, Session ID ឬព័ត៌មានឯកជនរបស់អ្នកប្រើ។ ត្រូវកំណត់សិទ្ធិចូលមើល Log ឱ្យបានត្រឹមត្រូវ។

## ១០. set_exception_handler និង set_error_handler

### ១០.១ Global Exception Handler

ប្រើសម្រាប់ចាប់ Uncaught Exception នៅកម្រិតកម្មវិធី៖

```php
<?php
set_exception_handler(function (Throwable $e): void {
    error_log($e);

    http_response_code(500);
    echo "មានបញ្ហាក្នុងប្រព័ន្ធ។ សូមព្យាយាមម្ដងទៀត។";
});

throw new RuntimeException("Unexpected application failure");
```

Handler នេះមិនមែនជំនួស `try/catch` នៅកន្លែងដែលត្រូវដោះស្រាយកំហុសដោយជាក់លាក់ទេ។ វាជាខ្សែការពារចុងក្រោយសម្រាប់ Exception ដែលមិនត្រូវបានចាប់។

### ១០.២ Custom Error Handler

អាចប្រើ `set_error_handler()` ដើម្បីគ្រប់គ្រង PHP errors មួយចំនួន៖

```php
<?php
set_error_handler(function (
    int $severity,
    string $message,
    string $file,
    int $line
): bool {
    error_log("$message in $file on line $line");

    // true មានន័យថា Handler បានគ្រប់គ្រង Error នេះហើយ
    return true;
});

trigger_error("ឧទាហរណ៍ Warning", E_USER_WARNING);
```

ចំណាំ៖ Error Handler មិនចាប់ Error គ្រប់ប្រភេទទាំងអស់ទេ។ កុំបម្លែង Warning ទាំងអស់ទៅជា Exception ដោយមិនយល់ពីផលប៉ះពាល់។ សម្រាប់កម្មវិធីធំ គួរប្រើ Framework ឬ Logging library ដែលមានរចនាសម្ព័ន្ធច្បាស់លាស់។

## ១១. Mini Project៖ កម្មវិធីគណនាដែលមាន Error Handling

បង្កើតឯកសារ `calculator.php`៖

```php
<?php
declare(strict_types=1);

function calculate(float $a, float $b, string $operator): float
{
    return match ($operator) {
        '+' => $a + $b,
        '-' => $a - $b,
        '*' => $a * $b,
        '/' => $b == 0.0
            ? throw new InvalidArgumentException("មិនអាចចែកនឹងសូន្យបានទេ")
            : $a / $b,
        default => throw new InvalidArgumentException("ប្រមាណវិធីមិនត្រឹមត្រូវ"),
    };
}

try {
    $a = filter_input(INPUT_GET, 'a', FILTER_VALIDATE_FLOAT);
    $b = filter_input(INPUT_GET, 'b', FILTER_VALIDATE_FLOAT);
    $operator = $_GET['op'] ?? '+';

    if ($a === false || $a === null || $b === false || $b === null) {
        throw new InvalidArgumentException("សូមបញ្ចូលលេខ a និង b ឱ្យបានត្រឹមត្រូវ");
    }

    if (!is_string($operator)) {
        throw new InvalidArgumentException("ប្រមាណវិធីមិនត្រឹមត្រូវ");
    }

    $result = calculate((float) $a, (float) $b, $operator);

    echo "លទ្ធផល៖ " . htmlspecialchars((string) $result, ENT_QUOTES, 'UTF-8');
} catch (InvalidArgumentException $e) {
    http_response_code(400);
    echo "កំហុសបញ្ចូលទិន្នន័យ៖ "
        . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8');
} catch (Throwable $e) {
    error_log($e);
    http_response_code(500);
    echo "មានបញ្ហាក្នុងប្រព័ន្ធ។";
}
```

សាកល្បងតាម URL នៅ Local Server៖

- `http://localhost/calculator.php?a=10&b=5&op=+`
- `http://localhost/calculator.php?a=10&b=0&op=/`
- `http://localhost/calculator.php?a=abc&b=5&op=*`
- `http://localhost/calculator.php?a=10&b=5&op=%`

ចំណាំ៖ នៅក្នុង URL សញ្ញា `+` អាចត្រូវបានបកស្រាយជាចន្លោះ។ សម្រាប់ភាពងាយស្រួល អាចប្រើ `%2B` ជំនួស `+` ក្នុង URL ដំបូង។

## ១២. Best Practices សម្រាប់ Error Handling

1. ចាប់ Exception តាមប្រភេទដែលអ្នកអាចដោះស្រាយបាន។
2. កុំប្រើ `catch (Throwable $e)` ហើយមិនធ្វើអ្វីទាំងអស់។
3. កុំបង្ហាញ Stack Trace ឬព័ត៌មាន Database ដល់អ្នកប្រើនៅ Production។
4. ប្រើ `error_log()` ឬ Logging System ដើម្បីរក្សាព័ត៌មានសម្រាប់ Debug។
5. ប្រើ HTTP Status Code ត្រឹមត្រូវ ដូចជា 400 សម្រាប់ Input មិនត្រឹមត្រូវ និង 500 សម្រាប់ Server Error។
6. Validate ទិន្នន័យពីអ្នកប្រើ មុនពេលយកទៅប្រើ។
7. ប្រើ Custom Exception នៅពេលវាធ្វើឱ្យកូដច្បាស់ និងងាយថែទាំ។
8. កុំចាប់ Exception ដើម្បីលាក់បញ្ហា។ បើមិនអាចដោះស្រាយបាន ត្រូវ Log ហើយបញ្ជូនវាទៅកន្លែងដែលអាចគ្រប់គ្រងបាន។
9. កុំកត់ត្រា Password, Token ឬទិន្នន័យសម្ងាត់។

## ១៣. កំហុសដែលអ្នកចាប់ផ្ដើមរៀនជួបញឹកញាប់

- **គិតថា try/catch ចាប់ Warning ទាំងអស់៖** មិនពិតទេ។ Warning មិនមែនជា Exception ដោយស្វ័យប្រវត្តិ។
- **ដាក់ Catch ទូលំទូលាយមុន Catch ជាក់លាក់៖** Catch ជាក់លាក់អាចមិនទទួលបានកំហុសតាមដែលរំពឹង។
- **បង្ហាញ `$e->getMessage()` ទៅអ្នកប្រើ Production៖** អាចបង្ហាញព័ត៌មានខាងក្នុង។ បង្ហាញសារទូទៅ ហើយ Log ព័ត៌មានលម្អិត។
- **ប្រើ `finally` ជំនួស Catch៖** `finally` សម្រាប់ការងារបញ្ចប់/សម្អាត មិនមែនសម្រាប់ដោះស្រាយ Exception ទេ។
- **ចាប់ Exception ហើយមិនធ្វើអ្វី៖** វាធ្វើឱ្យបញ្ហាពិបាកស្វែងរក។

## ១៤. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ ចាប់ Exception
សរសេរកូដដែលបោះ `Exception` ជាមួយសារ `Something went wrong` ហើយចាប់វាដើម្បីបង្ហាញសារ។

### លំហាត់ទី ២៖ ពិនិត្យលេខ
បង្កើត Function `squareRoot(float $number)`។ បើលេខតូចជាងសូន្យ ត្រូវបោះ `InvalidArgumentException`។ បើមិនដូច្នោះទេ ត្រឡប់ឫសការេដោយ `sqrt()`។

### លំហាត់ទី ៣៖ Custom Exception
បង្កើត Class `InvalidEmailException` ដែល extends `Exception`។ បង្កើត Function ពិនិត្យ Email ដោយ `filter_var($email, FILTER_VALIDATE_EMAIL)`។ បើមិនត្រឹមត្រូវ ត្រូវបោះ Custom Exception។

### លំហាត់ទី ៤៖ Logging
កែ Mini Project ឱ្យកំហុសមិនរំពឹងទុកត្រូវបានកត់ត្រាដោយ `error_log()` ហើយអ្នកប្រើឃើញតែសារទូទៅ។

### លំហាត់ទី ៥៖ សំណួរពន្យល់
ពន្យល់ភាពខុសគ្នារវាង `Exception`, `Error`, និង `Throwable` ដោយប្រើពាក្យផ្ទាល់ខ្លួន។

---

## ១៥. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```php
<?php
try {
    throw new Exception("Something went wrong");
} catch (Exception $e) {
    echo $e->getMessage();
}
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
function squareRoot(float $number): float
{
    if ($number < 0) {
        throw new InvalidArgumentException("លេខមិនអាចតូចជាងសូន្យបានទេ");
    }

    return sqrt($number);
}

try {
    echo squareRoot(25);
} catch (InvalidArgumentException $e) {
    echo $e->getMessage();
}
```

លទ្ធផល៖ `5`

### ចម្លើយលំហាត់ទី ៣

```php
<?php
class InvalidEmailException extends Exception
{
}

function validateEmail(string $email): void
{
    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        throw new InvalidEmailException("Email មិនត្រឹមត្រូវ");
    }
}

try {
    validateEmail("hello-example.com");
    echo "Email ត្រឹមត្រូវ";
} catch (InvalidEmailException $e) {
    echo $e->getMessage();
}
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
try {
    // កូដដែលអាចបង្កបញ្ហា
    throw new RuntimeException("Internal detail");
} catch (Throwable $e) {
    error_log($e);
    echo "មានបញ្ហា។ សូមព្យាយាមម្ដងទៀត។";
}
```

### ចម្លើយលំហាត់ទី ៥

- `Exception`៖ Base Class សម្រាប់ Exception ដែលកម្មវិធីអាចបោះ និងចាប់បាន។
- `Error`៖ Base Class សម្រាប់កំហុសកម្រិត PHP Engine ដូចជា TypeError។
- `Throwable`៖ Interface ដែលទាំង Exception និង Error អនុវត្ត ដូច្នេះអាចប្រើវាក្នុង Catch ដើម្បីចាប់ទាំងពីរ។

---

## សេចក្ដីសង្ខេប

- `try/catch` ប្រើសម្រាប់ចាប់ និងដោះស្រាយ Exception។
- `finally` ប្រើសម្រាប់កូដដែលត្រូវដំណើរការនៅចុងក្រោយ។
- `throw` ប្រើសម្រាប់បោះ Exception។
- Custom Exception ធ្វើឱ្យកំហុសជាក់លាក់ងាយយល់។
- Development អាចបង្ហាញ Error ប៉ុន្តែ Production គួរលាក់ព័ត៌មានលម្អិត និងប្រើ Logging។
- `Throwable` អាចចាប់ទាំង `Exception` និង `Error`។

## មេរៀនបន្ទាប់

**មេរៀនទី ១៤៖ PHP Date and Time** — រៀនអំពី `date()`, `time()`, `strtotime()`, `DateTime`, `DateTimeImmutable`, Timezone និងការគណនាថ្ងៃខែម៉ោង។
