# មេរៀនទី ៣៤៖ ការផ្ទៀងផ្ទាត់ទិន្នន័យ និង Input ក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Validation, Normalization និង Output Escaping។
- Validate Form Data នៅ Server និងបញ្ជូនកំហុសឱ្យអ្នកប្រើយល់។
- ប្រើ Allowlist សម្រាប់តម្លៃកំណត់ និង Range សម្រាប់លេខ។
- ពិនិត្យ Input ជា Array/Scalar ឱ្យបានត្រឹមត្រូវ។
- ជៀសវាងការយល់ច្រឡំថា Filter/Validation អាចជំនួស CSRF ឬ XSS Protection។

> HTML `required`, `type="email"` និង JavaScript Validation ជួយ UX ប៉ុណ្ណោះ។ Client អាចកែ Request បាន ដូច្នេះ Server ត្រូវ Validate ម្តងទៀត។

## ១. Validation, Normalization និង Escaping

- **Validation**៖ ពិនិត្យថាតម្លៃអាចទទួលយកបានតាមច្បាប់កម្មវិធីឬអត់។
- **Normalization**៖ បម្លែងទម្រង់ទៅជារបៀបស្តង់ដារ ដូចជា Trim Space ខាងចុង/ដើម។ ត្រូវធ្វើដោយប្រុងប្រយ័ត្ន ព្រោះការបម្លែងអាចប្តូរអត្ថន័យ។
- **Escaping**៖ បម្លែងទិន្នន័យនៅពេលបង្ហាញ ដើម្បីកុំឱ្យ Browser/Context ចាត់ទុកវាជា Code។

Validation មិនមែនជា XSS Escaping ទេ។ Input ដែលបាន Validate ក៏ត្រូវ Escape តាម Output Context នៅពេល Render HTML ដែរ។

## ២. ទទួល Form Data ជា String និងពិនិត្យទម្រង់

កុំសន្មតថា `$_POST['email']` តែងតែជា String; Client អាចផ្ញើ Parameter ជា Array បាន៖

```php
<?php
$errors = [];

$emailInput = $_POST['email'] ?? null;
$nameInput = $_POST['name'] ?? null;

if (!is_string($emailInput)) {
    $errors['email'] = 'សូមបញ្ចូល Email ជាអត្ថបទ។';
} else {
    $email = trim($emailInput);

    if ($email === '' || filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
        $errors['email'] = 'សូមបញ្ចូល Email ដែលមានទម្រង់ត្រឹមត្រូវ។';
    }
}

if (!is_string($nameInput)) {
    $errors['name'] = 'សូមបញ្ចូលឈ្មោះ។';
} else {
    $name = trim($nameInput);

    if ($name === '' || mb_strlen($name, 'UTF-8') > 100) {
        $errors['name'] = 'ឈ្មោះត្រូវមានពី 1 ដល់ 100 តួអក្សរ។';
    }
}
```

បើប្រើ `mb_strlen()` ត្រូវបើក PHP Extension `mbstring`។ សម្រាប់ចំនួនតួអក្សរអាចប្រើ `strlen()` តែវាវាស់ Bytes សម្រាប់ UTF-8 មិនមែនតួអក្សរទេ។

## ៣. Validate លេខ និង Range

Input ពី Form ជាទូទៅមកជា String។ ប្រើ `filter_var()` និងពិនិត្យ `false` ឱ្យច្បាស់៖

```php
<?php
$quantityInput = $_POST['quantity'] ?? null;
$quantity = is_string($quantityInput)
    ? filter_var(
        $quantityInput,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1, 'max_range' => 50]]
    )
    : false;

if ($quantity === false) {
    $errors['quantity'] = 'ចំនួនត្រូវជាលេខគត់ពី 1 ដល់ 50។';
}
```

កុំប្រើ `(int)$input` ដោយសន្មតថាតម្លៃណាក៏ក្លាយជាចំនួនត្រឹមត្រូវបានទេ; ឧទាហរណ៍ String មិនមែនលេខអាចបម្លែងទៅ 0 ហើយលាក់ Input ខុស។

## ៤. Allowlist សម្រាប់តម្លៃមានជម្រើស

បើ Form មានជម្រើសតួនាទី ឬប្រភេទដែលកំណត់ទុកជាមុន ត្រូវពិនិត្យតាម Allowlist នៅ Server៖

```php
<?php
$allowedStatuses = ['draft', 'published'];
$statusInput = $_POST['status'] ?? null;

if (
    !is_string($statusInput)
    || !in_array($statusInput, $allowedStatuses, true)
) {
    $errors['status'] = 'ស្ថានភាពដែលបានជ្រើសមិនត្រឹមត្រូវ។';
} else {
    $status = $statusInput;
}
```

កុំទុកចិត្តតម្លៃ Hidden Input ឬ Dropdown ព្រោះអ្នកប្រើអាចកែ Request ដោយខ្លួនឯង។ ការកំណត់ថា User អាច Publish បានឬអត់ គឺជា Authorization Check ដាច់ដោយឡែកពី Validation។

## ៥. ប្រវែង និងប្រភេទទិន្នន័យ

កំណត់ច្បាប់តាម Domain របស់កម្មវិធី៖

- កំណត់ប្រវែងអតិបរមាសម្រាប់ឈ្មោះ, Comment និង Search Query។
- ពិនិត្យ Date/Time តាម Format និងតម្លៃពិត (កាលបរិច្ឆេទខែ/ថ្ងៃអាចមិនមាន)។
- ពិនិត្យ ID ជាចំនួនវិជ្ជមាន ឬទម្រង់ UUID ដែលកម្មវិធីប្រើ។
- ត្រូវកំណត់កម្រិតទំហំ Request នៅ PHP និង Web Server ផងដែរ។
- ប្រើ Database Constraint ជាស្រទាប់បន្ថែមសម្រាប់ច្បាប់ទិន្នន័យសំខាន់ៗ។

កុំប្រើ Regex ស្មុគស្មាញដើម្បីបង្កើតច្បាប់ Email ដោយខ្លួនឯងទាំងស្រុង ប្រសិនបើ Validation ទូទៅគ្រប់គ្រាន់។ Email មាន Format ស្មុគស្មាញ; បើត្រូវប្រាកដថាអាសយដ្ឋានមានម្ចាស់ ត្រូវប្រើ Confirmation Email។

## ៦. បញ្ជូន Error និងបង្ហាញតម្លៃឡើងវិញ

ប្រមូល Error ដោយ Field ដើម្បីបង្ហាញសារច្បាស់ ហើយបង្ហាញតម្លៃចាស់ដោយ Escape៖

```php
<?php
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

if ($errors !== []) {
    http_response_code(422);
    // Render Form និងសារ Error; Escape ទាំង Error/Text និងតម្លៃដែលបង្ហាញ។
}
```

សារកំហុសគួរប្រាប់ច្បាប់ដែលអ្នកប្រើអាចកែបាន។ កុំបង្ហាញ Stack Trace, SQL ឬព័ត៌មានប្រព័ន្ធក្នុង Error Response។ កុំសន្មតថាតម្លៃដែលបាន Trim ឬ Validate រួចមានសុវត្ថិភាពសម្រាប់ HTML/JavaScript ដោយស្វ័យប្រវត្តិ។

## ៧. លំដាប់ពិនិត្យ Form

1. ពិនិត្យ HTTP Method និងទំហំ Request។
2. Authenticate User និងពិនិត្យ Authorization តាម Resource។
3. ផ្ទៀងផ្ទាត់ CSRF Token សម្រាប់ Request ដែលផ្លាស់ប្តូរ State។
4. ពិនិត្យ Type, Format, Allowlist, Range និងប្រវែង Input។
5. ប្រើ PDO Prepared Statements ពេលទាក់ទង Database។
6. បង្ហាញទិន្នន័យដោយ Output Escaping តាម Context។
7. កត់ត្រាតែព័ត៌មានចាំបាច់ ដោយមិន Log Password/Token។

## ៨. កំហុសដែលជួបញឹកញាប់

- Validate តែ Browser ដោយគ្មាន Server-side Check។
- គិតថា `FILTER_SANITIZE_*` ដោះស្រាយ Security បានគ្រប់បញ្ហា។
- បម្លែង Input ជា Integer ដោយ Cast ហើយមិនពិនិត្យទម្រង់ដើម។
- ទទួល Role/Price/Status ពី Client ដោយមិនពិនិត្យ Allowlist និង Authorization។
- បង្ហាញ Input ចាស់ក្នុង HTML ដោយមិន Escape។
- កំណត់ច្បាប់ធំពេក ឬតូចពេកដែលមិនសមនឹងអាជីវកម្ម។

## ៩. លំហាត់អនុវត្ត

1. Validate Email និងឈ្មោះ ដែលទទួលតែ String។
2. Validate ចំនួនទំនិញពី 1 ដល់ 50។
3. Validate Status តាម Allowlist `draft` និង `published`។
4. បង្ហាញ Form Error និងតម្លៃដែលបានវាយ ដោយ Escape។
5. សរសេរច្បាប់ Validation សម្រាប់ Profile Form និងពន្យល់ថាច្បាប់ណាជា Authorization។

## ១០. ចម្លើយណែនាំខ្លី

- ប្រើ `filter_var($email, FILTER_VALIDATE_EMAIL)` ហើយពិនិត្យថាលទ្ធផលមិនមែន `false`។
- ប្រើ `FILTER_VALIDATE_INT` ជាមួយ Min/Max Range សម្រាប់ចំនួនគត់។
- ប្រើ Allowlist និង Strict Comparison `in_array($value, $allowed, true)`។
- Escape តម្លៃនៅពេល Render HTML; កុំច្រឡំ Validation ជាមួយ Escaping។
- តួនាទីដែល User អាចធ្វើបានគឺ Authorization; តម្លៃ Status ដែលអាចមានគឺ Validation។

## ឯកសារយោង

- [PHP: Validation](https://www.php.net/manual/en/filter.examples.validation.php)
- [PHP: filter_var()](https://www.php.net/manual/en/function.filter-var.php)
- [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [មេរៀនទី ២៩៖ CSRF, XSS និង SQL Injection](lesson-29-php-csrf-xss-sql-injection.md)
