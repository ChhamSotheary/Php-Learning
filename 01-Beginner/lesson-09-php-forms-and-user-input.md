# មេរៀនទី ៩៖ PHP Forms និង User Input

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពីរបៀប HTML Form ផ្ញើទិន្នន័យទៅ PHP។
- ប្រៀបធៀប HTTP GET និង POST។
- ប្រើ PHP Superglobals ដូចជា `$_GET`, `$_POST`, `$_SERVER`។
- ទទួល និងត្រួតពិនិត្យទិន្នន័យពី Form។
- បង្ហាញទិន្នន័យដោយសុវត្ថិភាព ដោយប្រើ `htmlspecialchars()`។
- យល់ពី CSRF និងវិធីការពារជាមូលដ្ឋាន។
- បង្កើតកម្មវិធី Contact Form តូចមួយ។

## ២. តើ Form ជាអ្វី?

Form គឺជាផ្នែកមួយនៃ HTML ដែលអនុញ្ញាតឱ្យអ្នកប្រើបញ្ចូលព័ត៌មាន ដូចជា ឈ្មោះ អ៊ីមែល ពាក្យសម្ងាត់ ឬសារមួយ ហើយផ្ញើទៅ Server។

ឧទាហរណ៍៖

```html
<form action="process.php" method="POST">
    <label for="name">ឈ្មោះ៖</label>
    <input type="text" id="name" name="name">

    <button type="submit">ផ្ញើ</button>
</form>
```

ចំណុចសំខាន់៖

- `action` កំណត់ URL ឬ PHP file ដែលទទួលទិន្នន័យ។
- `method` កំណត់វិធីផ្ញើទិន្នន័យ។
- `name` ជាឈ្មោះ key ដែល PHP ប្រើដើម្បីអានតម្លៃ។
- `type="submit"` បង្កើតប៊ូតុងផ្ញើ Form។

**ចំណាំ៖** `id` ប្រើសម្រាប់ភ្ជាប់ Label ឬ JavaScript/CSS។ PHP ទទួលតម្លៃតាម `name` មិនមែន `id` ទេ។

## ៣. GET និង POST ខុសគ្នាដូចម្តេច?

### GET

GET ផ្ញើទិន្នន័យតាម URL ជា query string។

```html
<form action="search.php" method="GET">
    <input type="text" name="keyword">
    <button type="submit">ស្វែងរក</button>
</form>
```

បើវាយពាក្យ PHP អាចទទួល URL ប្រហែល៖

```
search.php?keyword=PHP
```

អានទិន្នន័យ៖

```php
<?php
$keyword = $_GET['keyword'] ?? '';
echo htmlspecialchars($keyword, ENT_QUOTES, 'UTF-8');
```

ប្រើ GET សម្រាប់ការស្វែងរក តម្រងទិន្នន័យ និងទំព័រដែលអាចចែករំលែក URL បាន។ កុំប្រើ GET ដើម្បីផ្ញើពាក្យសម្ងាត់ ឬទិន្នន័យឯកជន ព្រោះវាអាចបង្ហាញក្នុង URL និងប្រវត្តិ Browser។

### POST

POST ផ្ញើទិន្នន័យក្នុង request body ជំនួសឱ្យបង្ហាញក្នុង URL។

```html
<form action="register.php" method="POST">
    <label for="email">អ៊ីមែល៖</label>
    <input type="email" id="email" name="email">

    <button type="submit">ចុះឈ្មោះ</button>
</form>
```

អានទិន្នន័យ៖

```php
<?php
$email = $_POST['email'] ?? '';
echo htmlspecialchars($email, ENT_QUOTES, 'UTF-8');
```

POST សមស្របសម្រាប់ការបង្កើត ឬកែប្រែទិន្នន័យ។ ប៉ុន្តែ POST មិនធ្វើឱ្យទិន្នន័យមានសុវត្ថិភាពដោយស្វ័យប្រវត្តិទេ។ សម្រាប់ទិន្នន័យឯកជន ត្រូវប្រើ HTTPS និងការការពារផ្សេងៗទៀត។

| ចំណុច | GET | POST |
|---|---|---|
| ទីតាំងទិន្នន័យ | URL | Request body |
| អាចរក្សាទុកក្នុង Bookmark | ជាទូទៅបាន | មិនមែនគោលបំណងរបស់វា |
| ការប្រើប្រាស់ធម្មតា | Search/Filter | Submit/Create/Update |
| លាក់ពី URL | ទេ | បាទ/ចាស |
| សុវត្ថិភាពដោយស្វ័យប្រវត្តិ | គ្មាន | គ្មាន |

## ៤. PHP Superglobals

Superglobals គឺជា Variables ពិសេសដែល PHP អាចប្រើបានគ្រប់ Scope។

### ៤.១ `$_GET`

ទទួលទិន្នន័យដែលផ្ញើដោយ GET។

```php
<?php
$name = $_GET['name'] ?? 'អ្នកប្រើប្រាស់';
echo 'សួស្តី ' . htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
```

### ៤.២ `$_POST`

ទទួលទិន្នន័យដែលផ្ញើដោយ POST។

```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = $_POST['name'] ?? '';
    echo htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
}
```

### ៤.៣ `$_SERVER`

ផ្ទុកព័ត៌មានអំពី Request និង Server។

```php
<?php
echo $_SERVER['REQUEST_METHOD'];
```

តម្លៃអាចជា `GET` ឬ `POST`។ កុំសន្មតថា Header ឬតម្លៃពី `$_SERVER` ទាំងអស់គួរឱ្យទុកចិត្ត។

### ៤.៤ `$_REQUEST`

អាចរួមបញ្ចូលតម្លៃពី GET, POST និង Cookie អាស្រ័យលើការកំណត់ PHP។ ជាទូទៅ គួរប្រើ `$_GET` ឬ `$_POST` ដោយច្បាស់លាស់ ដើម្បីដឹងថាទិន្នន័យមកពីណា។

### ៤.៥ `$_FILES`

ប្រើសម្រាប់ទទួលព័ត៌មានឯកសារដែល Upload តាម Form។ ត្រូវមាន `enctype="multipart/form-data"`។ ការទទួល File Upload ត្រូវពិនិត្យប្រភេទឯកសារ ទំហំ និងឈ្មោះឯកសារនៅ Server មិនត្រូវជឿលើឈ្មោះ ឬ MIME type ដែល Browser ផ្ញើមកតែមួយមុខទេ។

## ៥. ពិនិត្យថា Form ត្រូវបាន Submit ឬអត់

```php
<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    echo 'Form ត្រូវបាន Submit ដោយ POST។';
} else {
    echo 'សូមបំពេញ Form ជាមុន។';
}
```

វិធីនេះជួយឱ្យ PHP ដឹងថាគួរដំណើរការទិន្នន័យពេលណា។

## ៦. Validation និង Sanitization

**Validation** គឺការត្រួតពិនិត្យថាទិន្នន័យត្រឹមត្រូវតាមលក្ខខណ្ឌឬអត់។ ឧទាហរណ៍ អ៊ីមែលមានទម្រង់ត្រឹមត្រូវឬទេ។

**Sanitization** គឺការសម្អាត ឬបម្លែងទិន្នន័យ។ វាមិនអាចជំនួស Validation បានទេ។

ឧទាហរណ៍៖

```php
<?php
$email = trim($_POST['email'] ?? '');

if ($email === '') {
    echo 'សូមបញ្ចូលអ៊ីមែល។';
} elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    echo 'ទម្រង់អ៊ីមែលមិនត្រឹមត្រូវ។';
} else {
    echo 'អ៊ីមែលត្រឹមត្រូវ។';
}
```

- `trim()` ដកចន្លោះនៅដើម និងចុង String។
- `filter_var()` ជាមួយ `FILTER_VALIDATE_EMAIL` ពិនិត្យទម្រង់អ៊ីមែល។
- ការត្រួតពិនិត្យត្រូវធ្វើនៅ Server ទោះបីមាន HTML validation ក៏ដោយ។

## ៧. បង្ហាញទិន្នន័យដោយសុវត្ថិភាព

កុំបង្ហាញ Input របស់អ្នកប្រើដោយផ្ទាល់ទៅ HTML។ ប្រសិនបើអ្នកប្រើបញ្ចូលអក្សរដែលមាន HTML ឬ JavaScript វាអាចបង្កហានិភ័យ XSS ប្រសិនបើបង្ហាញដោយមិន Escape។

ប្រើ៖

```php
<?php
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

$name = $_POST['name'] ?? '';
echo '<p>សួស្តី ' . e($name) . '</p>';
```

`htmlspecialchars()` ជួយ Escape តួអក្សរពិសេសសម្រាប់បរិបទ HTML។ វាមិនមែនជាវិធីការពារគ្រប់បរិបទទាំងអស់ទេ។ កុំយកទិន្នន័យទៅដាក់ក្នុង JavaScript ឬ SQL ដោយគិតថា HTML escaping គ្រប់គ្រាន់។

## ៨. កម្មវិធីអនុវត្ត៖ Contact Form

បង្កើត Folder មួយឈ្មោះ `php-form-demo` ហើយបង្កើតឯកសារ `contact.php`៖

```php
<?php
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

$name = '';
$email = '';
$message = '';
$errors = [];
$success = false;

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = trim($_POST['name'] ?? '');
    $email = trim($_POST['email'] ?? '');
    $message = trim($_POST['message'] ?? '');

    if ($name === '') {
        $errors[] = 'សូមបញ្ចូលឈ្មោះ។';
    } elseif (mb_strlen($name) > 100) {
        $errors[] = 'ឈ្មោះវែងពេក។';
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = 'សូមបញ្ចូលអ៊ីមែលឱ្យបានត្រឹមត្រូវ។';
    }

    if ($message === '') {
        $errors[] = 'សូមបញ្ចូលសារ។';
    } elseif (mb_strlen($message) > 2000) {
        $errors[] = 'សារវែងពេក។';
    }

    if ($errors === []) {
        // ក្នុងមេរៀននេះ យើងបង្ហាញសារជោគជ័យប៉ុណ្ណោះ។
        // មេរៀន Database នឹងបង្រៀនការរក្សាទុកទិន្នន័យ។
        $success = true;
    }
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Contact Form</title>
</head>
<body>
    <h1>ទាក់ទងមកយើង</h1>

    <?php if ($success): ?>
        <p>សូមអរគុណ <?= e($name) ?>! Form របស់អ្នកត្រឹមត្រូវហើយ។</p>
    <?php endif; ?>

    <?php if ($errors !== []): ?>
        <ul>
            <?php foreach ($errors as $error): ?>
                <li><?= e($error) ?></li>
            <?php endforeach; ?>
        </ul>
    <?php endif; ?>

    <form method="POST" action="">
        <div>
            <label for="name">ឈ្មោះ៖</label>
            <input
                type="text"
                id="name"
                name="name"
                maxlength="100"
                value="<?= e($name) ?>"
                required
            >
        </div>

        <div>
            <label for="email">អ៊ីមែល៖</label>
            <input
                type="email"
                id="email"
                name="email"
                value="<?= e($email) ?>"
                required
            >
        </div>

        <div>
            <label for="message">សារ៖</label>
            <textarea
                id="message"
                name="message"
                maxlength="2000"
                required
            ><?= e($message) ?></textarea>
        </div>

        <button type="submit">ផ្ញើសារ</button>
    </form>
</body>
</html>
```

### របៀបដំណើរការ

1. បើក Terminal ឬ Command Prompt។
2. ចូលទៅក្នុង Folder `php-form-demo`។
3. ដំណើរការ៖

```bash
php -S localhost:8000
```

4. បើក Browser ទៅកាន់ `http://localhost:8000/contact.php`។
5. សាកល្បង Submit Form ដោយទុកប្រអប់ទទេ និងបញ្ចូលព័ត៌មានត្រឹមត្រូវ។

**លទ្ធផលរំពឹងទុក៖**

- បើឈ្មោះទទេ នឹងបង្ហាញសារ «សូមបញ្ចូលឈ្មោះ»។
- បើអ៊ីមែលខុសទម្រង់ នឹងបង្ហាញសារកំហុស។
- បើទិន្នន័យត្រឹមត្រូវ នឹងបង្ហាញសារអរគុណ។

## ៩. សុវត្ថិភាព Form ដែលគួរចងចាំ

### ៩.១ កុំជឿទិន្នន័យពី Browser

អ្នកប្រើអាចកែ HTML ឬផ្ញើ Request ដោយផ្ទាល់បាន។ ដូច្នេះ Validation នៅ Server គឺចាំបាច់។

### ៩.២ CSRF ជាអ្វី?

CSRF (Cross-Site Request Forgery) គឺជាការបោកឱ្យ Browser របស់អ្នកប្រើផ្ញើ Request ដែលពួកគេមិនមានបំណងធ្វើ ទៅកាន់ Website ដែលពួកគេបាន Login រួច។

សម្រាប់ Form ដែលបង្កើត ឬកែប្រែទិន្នន័យ គួរប្រើ CSRF Token។ ឧទាហរណ៍មូលដ្ឋាន៖

```php
<?php
session_start();

if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$token = $_SESSION['csrf_token'];
?>

<form method="POST">
    <input
        type="hidden"
        name="csrf_token"
        value="<?= htmlspecialchars($token, ENT_QUOTES, 'UTF-8') ?>"
    >
    <button type="submit">បញ្ជូន</button>
</form>
```

នៅផ្នែកទទួល POST ត្រូវផ្ទៀងផ្ទាត់ Token មុនធ្វើសកម្មភាព៖

```php
<?php
session_start();

$submittedToken = $_POST['csrf_token'] ?? '';
$sessionToken = $_SESSION['csrf_token'] ?? '';

if (
    !is_string($submittedToken) ||
    !is_string($sessionToken) ||
    $sessionToken === '' ||
    !hash_equals($sessionToken, $submittedToken)
) {
    http_response_code(403);
    exit('សំណើមិនត្រឹមត្រូវ។');
}

// បន្តដំណើរការ Form បន្ទាប់ពី Token ត្រឹមត្រូវ។
```

ឧទាហរណ៍នេះបង្ហាញគោលការណ៍ Token ប៉ុណ្ណោះ។ ក្នុងកម្មវិធីពិត ត្រូវរៀបចំ Session, Cookie និងការការពារផ្សេងៗឱ្យបានត្រឹមត្រូវ។

### ៩.៣ ការប្រុងប្រយ័ត្នផ្សេងៗ

- ប្រើ HTTPS សម្រាប់ Website ពិតប្រាកដ។
- កុំបង្ហាញ Error លម្អិតរបស់ Server ទៅអ្នកប្រើធម្មតា។
- កុំរក្សាទុក Password ជា Plain Text។ មេរៀនក្រោយនឹងរៀន `password_hash()` និង `password_verify()`។
- ពេលភ្ជាប់ Database ត្រូវប្រើ PDO Prepared Statements មិនត្រូវភ្ជាប់ SQL ដោយ String Concatenation។
- កំណត់ប្រវែងទិន្នន័យ និងប្រភេទទិន្នន័យនៅ Server។

## ១០. កំហុសដែលអ្នកចាប់ផ្តើមរៀនជួបញឹកញាប់

1. **ប្រើ `$_POST['name']` ដោយមិនពិនិត្យថាមានឬអត់** — ប្រើ `$_POST['name'] ?? ''`។
2. **ភ្លេច `name` ក្នុង Input** — PHP ត្រូវការ `name` ដើម្បីទទួលតម្លៃ។
3. **Form ប្រើ POST តែ PHP អាន `$_GET`** — ត្រូវប្រើឱ្យត្រូវនឹង Method។
4. **ជឿលើ HTML `required` តែមួយ** — ត្រូវ Validate នៅ Server ផងដែរ។
5. **បង្ហាញ Input ដោយផ្ទាល់** — ប្រើ `htmlspecialchars()` នៅពេលបង្ហាញក្នុង HTML។
6. **គិតថា POST មានសុវត្ថិភាពជាងគ្រប់ករណី** — POST មិនជំនួស HTTPS, Validation ឬ CSRF Protection ទេ។

## ១១. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Simple Greeting Form

បង្កើត Form មានប្រអប់ឈ្មោះមួយ។ ពេល Submit ត្រូវបង្ហាញ៖

`សួស្តី [ឈ្មោះរបស់អ្នក]`

លក្ខខណ្ឌ៖
- ប្រើ POST។
- បើឈ្មោះទទេ បង្ហាញសារកំហុស។
- ប្រើ `htmlspecialchars()` ពេលបង្ហាញឈ្មោះ។

### លំហាត់ទី ២៖ Email Validation

បង្កើត Form មានប្រអប់ Email។ ប្រើ `filter_var()` ដើម្បីពិនិត្យទម្រង់អ៊ីមែល។

### លំហាត់ទី ៣៖ Student Registration

បង្កើត Form ដែលមាន៖
- Full Name
- Email
- Age
- Gender
- Course

ត្រូវ Validate ថា៖
- ឈ្មោះមិនទទេ។
- អ៊ីមែលមានទម្រង់ត្រឹមត្រូវ។
- អាយុជាចំនួនគត់ និងនៅចន្លោះ 13 ដល់ 100។
- Course ត្រូវជាជម្រើសដែលបានកំណត់ក្នុងកម្មវិធី មិនមែនតម្លៃណាក៏បាន។

## ១២. Mini Project៖ Student Registration Form

បង្កើតកម្មវិធីចុះឈ្មោះសិស្សដោយប្រើ PHP និង HTML Form។

**តម្រូវការ៖**

- មានឈ្មោះ អ៊ីមែល អាយុ ភេទ និងវគ្គសិក្សា។
- ប្រើ POST ដើម្បីផ្ញើទិន្នន័យ។
- Validate ទិន្នន័យទាំងអស់នៅ Server។
- បង្ហាញ Error Message នៅពេលទិន្នន័យមិនត្រឹមត្រូវ។
- បង្ហាញព័ត៌មានសង្ខេបនៅពេលទិន្នន័យត្រឹមត្រូវ។
- Escape តម្លៃទាំងអស់មុនបង្ហាញក្នុង HTML។
- ក្នុងមេរៀននេះ មិនទាន់ត្រូវការរក្សាទុក Database ទេ។

## ១៣. សំណួររំលឹកមេរៀន

1. តើ HTML Form មានតួនាទីអ្វី?
2. តើ GET និង POST ខុសគ្នាដូចម្តេច?
3. តើ `$_POST` ប្រើសម្រាប់អ្វី?
4. តើ `$_SERVER['REQUEST_METHOD']` អាចជួយយើងដូចម្តេច?
5. តើ Validation ខុសពី Sanitization ដូចម្តេច?
6. ហេតុអ្វីត្រូវប្រើ `htmlspecialchars()`?
7. តើ CSRF Token មានតួនាទីអ្វី?

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

1. Form ប្រមូលទិន្នន័យពីអ្នកប្រើ ហើយផ្ញើទៅ Server។
2. GET ផ្ញើតាម URL; POST ផ្ញើតាម Request body។
3. `$_POST` អានតម្លៃដែលបានផ្ញើដោយ HTTP POST។
4. វាប្រាប់ថា Request បច្ចុប្បន្នប្រើ Method អ្វី។
5. Validation ពិនិត្យថាទិន្នន័យត្រឹមត្រូវឬអត់; Sanitization សម្អាត ឬបម្លែងទិន្នន័យ។
6. ដើម្បី Escape តួអក្សរពិសេសនៅពេលបង្ហាញក្នុង HTML និងកាត់បន្ថយហានិភ័យ XSS។
7. CSRF Token ជួយផ្ទៀងផ្ទាត់ថា Form Request មាន Token ត្រឹមត្រូវដែលភ្ជាប់នឹង Session របស់អ្នកប្រើ។

---

## សេចក្តីសង្ខេប

ក្នុងមេរៀននេះ អ្នកបានរៀនពី HTML Form, GET, POST, PHP Superglobals, Validation, Output Escaping និង CSRF Token។ ជំហានបន្ទាប់គឺរៀន **PHP Include និង Require** ដើម្បីរៀបចំកូដជា File ច្រើន និងប្រើកូដឡើងវិញបានងាយស្រួល។
