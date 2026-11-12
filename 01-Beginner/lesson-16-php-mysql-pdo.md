# មេរៀនទី ១៦៖ PHP និង MySQL ដោយប្រើ PDO

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីតួនាទី Database, Table, Row និង Column។
- បង្កើត Database និង Table សាមញ្ញក្នុង MySQL។
- ភ្ជាប់ PHP ទៅ MySQL ដោយប្រើ PDO។
- ប្រើ Prepared Statements ដើម្បីដំណើរការ SQL ដោយសុវត្ថិភាព។
- បន្ថែម អាន កែប្រែ និងលុបទិន្នន័យ (CRUD)។
- គ្រប់គ្រងកំហុស Database ដោយមិនបង្ហាញព័ត៌មានសម្ងាត់។
- រក្សាទុកព័ត៌មានភ្ជាប់ Database ឱ្យមានសុវត្ថិភាព។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL និង PHP PDO MySQL driver (`pdo_mysql`)។ ពិនិត្យ Extension ដោយប្រើ `php -m`។

---

## ១. Database ជាអ្វី?

Database ជាកន្លែងរក្សាទុកទិន្នន័យឱ្យមានរចនាសម្ព័ន្ធ ដើម្បីអាចស្វែងរក និងកែប្រែបាន។ ក្នុង MySQL៖

- **Database** ជាផ្ទុករួមសម្រាប់តារាង។
- **Table** ជារចនាសម្ព័ន្ធទុកទិន្នន័យប្រភេទដូចគ្នា។
- **Row** ជាកំណត់ត្រាមួយ។
- **Column** ជាលក្ខណៈមួយរបស់កំណត់ត្រា។

ឧទាហរណ៍ Table `students` អាចមាន Column `id`, `name`, និង `email`។ Row នីមួយៗតំណាងឱ្យសិស្សម្នាក់។

PHP អាចភ្ជាប់ទៅ Database ជាច្រើនប្រភេទតាម PDO។ ដើម្បីប្រើ MySQL ត្រូវមាន Driver PDO_MYSQL ដំឡើង និងបើកដំណើរការ។

## ២. បង្កើត Database និង Table

ដំណើរការ SQL ខាងក្រោមក្នុង MySQL client ឬ phpMyAdmin៖

```sql
CREATE DATABASE learning_php
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

USE learning_php;

CREATE TABLE students (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- `utf8mb4` គាំទ្រអក្សរខ្មែរ និង Unicode ផ្សេងៗ។
- `AUTO_INCREMENT` បង្កើន `id` ដោយស្វ័យប្រវត្តិ។
- `PRIMARY KEY` កំណត់អត្តសញ្ញាណ Row មួយៗ។
- `NOT NULL` បញ្ជាក់ថា Column ត្រូវមានតម្លៃ។
- `UNIQUE` មិនអនុញ្ញាតឱ្យ Email ស្ទួន។

## ៣. ភ្ជាប់ Database ដោយ PDO

បង្កើត `database.php`៖

```php
<?php
declare(strict_types=1);

$host = getenv('DB_HOST') ?: '127.0.0.1';
$dbname = getenv('DB_NAME') ?: 'learning_php';
$username = getenv('DB_USER') ?: 'app_user';
$password = getenv('DB_PASSWORD') ?: '';

$dsn = "mysql:host={$host};dbname={$dbname};charset=utf8mb4";

try {
    $pdo = new PDO($dsn, $username, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]);
} catch (PDOException $e) {
    error_log('Database connection failed: ' . $e->getMessage());
    http_response_code(500);
    exit('មិនអាចភ្ជាប់ Database បានទេ។');
}
```

ពន្យល់៖

- DSN បញ្ជាក់ Driver និងព័ត៌មានទីតាំង Database។ សម្រាប់ MySQL ទម្រង់ចាប់ផ្ដើមដោយ `mysql:`។
- `charset=utf8mb4` កំណត់ Encoding នៃការភ្ជាប់។
- `PDO::ERRMODE_EXCEPTION` ធ្វើឱ្យ PDO បោះ `PDOException` ពេលមានបញ្ហា។
- `PDO::FETCH_ASSOC` ធ្វើឱ្យ Row ដែលទាញមកមាន Key តាមឈ្មោះ Column។
- `getenv()` អាន Environment Variables ដើម្បីមិនសរសេរ Password ក្នុងកូដ។

នៅ PHP 8.0 ឡើងទៅ PDO ប្រើ Exception Mode ជាលំនាំដើម ប៉ុន្តែការកំណត់វាច្បាស់ធ្វើឱ្យគោលបំណងកូដងាយយល់។

**កុំដាក់ Database Password ពិតក្នុង GitHub។** កំណត់ Environment Variables នៅ Local Server និង Hosting។ ឧទាហរណ៍ខាងលើអាចប្រើ User សាកល្បងបាន តែត្រូវបង្កើត MySQL User ដែលមានសិទ្ធិតិចតួចតាមតម្រូវការ។

## ៤. អានទិន្នន័យដោយ SELECT

ភ្ជាប់ឯកសារ `database.php` រួចស្នើទិន្នន័យ៖

```php
<?php
require __DIR__ . '/database.php';

$stmt = $pdo->query(
    'SELECT id, name, email, created_at FROM students ORDER BY id DESC'
);

$students = $stmt->fetchAll();

foreach ($students as $student) {
    echo htmlspecialchars($student['name'], ENT_QUOTES, 'UTF-8');
    echo ' — ';
    echo htmlspecialchars($student['email'], ENT_QUOTES, 'UTF-8');
    echo '<br>';
}
```

លទ្ធផលពី Database គួរត្រូវបាន Escape មុនបង្ហាញក្នុង HTML។ នៅទីនេះប្រើ `htmlspecialchars()` ដើម្បីកាត់បន្ថយហានិភ័យ XSS។

## ៥. Prepared Statements

កុំភ្ជាប់ Input របស់អ្នកប្រើទៅក្នុង SQL ដោយ String Concatenation។ វាអាចបង្ក SQL Injection។ ប្រើ Prepared Statement និង Placeholder៖

```php
<?php
require __DIR__ . '/database.php';

$email = $_GET['email'] ?? '';

$stmt = $pdo->prepare(
    'SELECT id, name, email FROM students WHERE email = :email'
);

$stmt->execute(['email' => $email]);

$student = $stmt->fetch();

if ($student === false) {
    echo 'រកមិនឃើញសិស្សទេ។';
} else {
    echo htmlspecialchars($student['name'], ENT_QUOTES, 'UTF-8');
}
```

PDO បញ្ជូន SQL Template និងតម្លៃប៉ារ៉ាម៉ែត្រដាច់ពីគ្នា។ តម្លៃដែលអ្នកប្រើផ្ញើមកនឹងមិនត្រូវចាត់ទុកជាផ្នែកនៃ SQL Command ទេ។

អាចប្រើ Placeholder តាមឈ្មោះដូច `:email` ឬ Placeholder តាមលំដាប់ `?`៖

```php
$stmt = $pdo->prepare(
    'SELECT id, name FROM students WHERE email = ?'
);
$stmt->execute([$email]);
```

Placeholder អាចតំណាងឱ្យតម្លៃ ប៉ុន្តែមិនអាចជំនួស Table Name, Column Name ឬ SQL Keyword បានទេ។ ប្រសិនបើ Sort Field ត្រូវជ្រើសដោយអ្នកប្រើ ត្រូវផ្គូផ្គងជាមួយបញ្ជីឈ្មោះដែលកូដអនុញ្ញាតជាមុន។

## ៦. បន្ថែមទិន្នន័យដោយ INSERT

```php
<?php
require __DIR__ . '/database.php';

$name = trim($_POST['name'] ?? '');
$email = trim($_POST['email'] ?? '');

if ($name === '' || mb_strlen($name) > 100) {
    http_response_code(422);
    exit('ឈ្មោះមិនត្រឹមត្រូវ។');
}

if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
    http_response_code(422);
    exit('Email មិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare(
    'INSERT INTO students (name, email) VALUES (:name, :email)'
);

try {
    $stmt->execute([
        'name' => $name,
        'email' => $email,
    ]);

    echo 'បានបង្កើតសិស្សលេខ ' . $pdo->lastInsertId();
} catch (PDOException $e) {
    error_log('Insert student failed: ' . $e->getMessage());
    http_response_code(500);
    echo 'មិនអាចរក្សាទុកទិន្នន័យបានទេ។';
}
```

ការប្រើ Prepared Statement ការពារ SQL Injection ប៉ុន្តែមិនជំនួស Validation ទេ។ ត្រូវពិនិត្យទិន្នន័យនៅ Server ដដែល។

## ៧. កែប្រែទិន្នន័យដោយ UPDATE

```php
<?php
require __DIR__ . '/database.php';

$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);
$name = trim($_POST['name'] ?? '');

if ($id === false || $id === null || $id < 1 || $name === '') {
    http_response_code(422);
    exit('ទិន្នន័យមិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare(
    'UPDATE students SET name = :name WHERE id = :id'
);

$stmt->execute([
    'name' => $name,
    'id' => $id,
]);

if ($stmt->rowCount() === 0) {
    echo 'មិនមាន Row ត្រូវបានផ្លាស់ប្ដូរ។';
} else {
    echo 'កែប្រែទិន្នន័យរួចរាល់។';
}
```

ចំណាំ៖ `rowCount()` សម្រាប់ UPDATE អាចប្រាប់ចំនួន Row ដែលត្រូវបានផ្លាស់ប្ដូរ។ វាមិនចាំបាច់ស្មើនឹងចំនួន Row ដែលត្រូវនឹងលក្ខខណ្ឌនៅគ្រប់ Database ឬគ្រប់ករណីទេ។

## ៨. លុបទិន្នន័យដោយ DELETE

```php
<?php
require __DIR__ . '/database.php';

$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    http_response_code(422);
    exit('លេខសម្គាល់មិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare('DELETE FROM students WHERE id = :id');
$stmt->execute(['id' => $id]);

echo $stmt->rowCount() > 0
    ? 'លុបទិន្នន័យរួចរាល់។'
    : 'រកមិនឃើញទិន្នន័យដែលត្រូវលុបទេ។';
```

សកម្មភាពបង្កើត កែប្រែ និងលុបគួរប្រើ POST, PUT, PATCH ឬ DELETE តាមរចនាបថកម្មវិធី។ កុំធ្វើការលុបទិន្នន័យដោយ GET Link ធម្មតា។

## ៩. Mini Project៖ Student Directory

បង្កើត Folder៖

```text
student-directory/
├── database.php
└── index.php
```

បង្កើត Table តាម SQL នៅផ្នែកទី ២។ កំណត់ Environment Variables សម្រាប់ Database ហើយបង្កើត `index.php`៖

```php
<?php
declare(strict_types=1);

require __DIR__ . '/database.php';

$error = '';
$name = '';
$email = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $name = trim($_POST['name'] ?? '');
    $email = trim($_POST['email'] ?? '');

    if ($name === '' || mb_strlen($name) > 100) {
        $error = 'សូមបញ្ចូលឈ្មោះដែលមានប្រវែងមិនលើស 100 តួអក្សរ។';
    } elseif (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $error = 'សូមបញ្ចូល Email ដែលត្រឹមត្រូវ។';
    } else {
        try {
            $stmt = $pdo->prepare(
                'INSERT INTO students (name, email) VALUES (:name, :email)'
            );

            $stmt->execute([
                'name' => $name,
                'email' => $email,
            ]);

            header('Location: index.php');
            exit;
        } catch (PDOException $e) {
            error_log('Create student failed: ' . $e->getMessage());
            $error = 'មិនអាចរក្សាទុកបានទេ។ Email ប្រហែលត្រូវបានប្រើរួចហើយ។';
        }
    }
}

$stmt = $pdo->query(
    'SELECT id, name, email, created_at FROM students ORDER BY id DESC'
);
$students = $stmt->fetchAll();
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>បញ្ជីសិស្ស</title>
</head>
<body>
    <h1>បញ្ជីសិស្ស</h1>

    <?php if ($error !== ''): ?>
        <p><?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
    <?php endif; ?>

    <form method="POST">
        <label for="name">ឈ្មោះ៖</label>
        <input
            id="name"
            name="name"
            maxlength="100"
            value="<?= htmlspecialchars($name, ENT_QUOTES, 'UTF-8') ?>"
            required
        >

        <label for="email">Email៖</label>
        <input
            id="email"
            name="email"
            type="email"
            value="<?= htmlspecialchars($email, ENT_QUOTES, 'UTF-8') ?>"
            required
        >

        <button type="submit">បន្ថែមសិស្ស</button>
    </form>

    <h2>សិស្សទាំងអស់</h2>

    <?php if ($students === []): ?>
        <p>មិនទាន់មានទិន្នន័យទេ។</p>
    <?php else: ?>
        <ul>
            <?php foreach ($students as $student): ?>
                <li>
                    <?= htmlspecialchars($student['name'], ENT_QUOTES, 'UTF-8') ?>
                    —
                    <?= htmlspecialchars($student['email'], ENT_QUOTES, 'UTF-8') ?>
                </li>
            <?php endforeach; ?>
        </ul>
    <?php endif; ?>
</body>
</html>
```

សាកល្បង៖

1. បើក Web Server និង MySQL។
2. បើក Terminal ក្នុង Folder `student-directory`។
3. ដំណើរការ `php -S localhost:8000`។
4. បើក `http://localhost:8000/index.php`។
5. បញ្ចូលឈ្មោះ និង Email រួចពិនិត្យថាតារាងបង្ហាញ Row ថ្មី។

គម្រោងនេះបង្ហាញ Create និង Read។ អ្នកអាចបន្តបន្ថែម Form សម្រាប់ Update និង Delete ដោយប្រើ Prepared Statements ដូចផ្នែកខាងលើ។

## ១០. កំហុសដែលជួបញឹកញាប់

- **ភ្លេចបើក `pdo_mysql`៖** PHP អាចរាយការណ៍ថា Driver មិនមាន។
- **DSN ខុស ឬ Database មិនទាន់បង្កើត៖** ពិនិត្យ Host, Port, Database Name និងសិទ្ធិ User។
- **ប្រើ Charset មិនត្រឹមត្រូវ៖** កំណត់ `utf8mb4` សម្រាប់ Database និង DSN។
- **ភ្ជាប់ Input ទៅ SQL ដោយ String Concatenation៖** ប្រើ Prepared Statements។
- **យក Placeholder ទៅដាក់ជំនួស Table Name៖** Placeholder គាំទ្រតែតម្លៃទិន្នន័យ។
- **គិតថា Prepared Statement ជំនួស Validation៖** ត្រូវ Validate ទិន្នន័យដោយឡែក។
- **បង្ហាញ PDOException ទៅអ្នកប្រើ៖** Log ព័ត៌មានលម្អិត ហើយបង្ហាញសារទូទៅ។
- **Commit Database Password ទៅ GitHub៖** ប្រើ Environment Variables ឬ Secret Manager។
- **មិន Escape ទិន្នន័យមុនបង្ហាញ HTML៖** ទិន្នន័យពី Database ក៏ត្រូវចាត់ទុកថាមិនទុកចិត្តបានដែរ។

## ១១. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ បង្កើត Table ផ្សេង

បង្កើត Table `books` ដែលមាន `id`, `title`, `author` និង `created_at`។

### លំហាត់ទី ២៖ ស្វែងរកតាម Email

បង្កើត Form ស្វែងរកសិស្សតាម Email ដោយប្រើ Prepared Statement។

### លំហាត់ទី ៣៖ បន្ថែមសិស្ស

បង្កើត Form ទទួល Name និង Email។ Validate ទាំងពីរនៅ Server ហើយ Insert ទៅ Database។

### លំហាត់ទី ៤៖ កែប្រែឈ្មោះ

បង្កើត Form ទទួល Student ID និងឈ្មោះថ្មី។ ប្រើ UPDATE ជាមួយ Placeholder។

### លំហាត់ទី ៥៖ លុបសិស្ស

បង្កើត Form ប្រើ POST ដើម្បីលុបសិស្សតាម ID។ ពិនិត្យ ID មុនដំណើរការ។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ថា Prepared Statement ជួយទប់ស្កាត់ SQL Injection ដូចម្ដេច ហើយហេតុអ្វីវាមិនជំនួស Input Validation។

## ១២. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```sql
CREATE TABLE books (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(150) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
$email = trim($_GET['email'] ?? '');

$stmt = $pdo->prepare(
    'SELECT id, name, email FROM students WHERE email = :email'
);
$stmt->execute(['email' => $email]);

$student = $stmt->fetch();
```

ចំណាំ៖ ក្នុងកម្មវិធីពិត ត្រូវពិនិត្យថា Email មានទម្រង់ត្រឹមត្រូវ និងបង្ហាញលទ្ធផលដោយ Escape។

### ចម្លើយលំហាត់ទី ៣

```php
<?php
$name = trim($_POST['name'] ?? '');
$email = trim($_POST['email'] ?? '');

if ($name === '' || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
    http_response_code(422);
    exit('ទិន្នន័យមិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare(
    'INSERT INTO students (name, email) VALUES (:name, :email)'
);

$stmt->execute([
    'name' => $name,
    'email' => $email,
]);
```

### ចម្លើយលំហាត់ទី ៤

```php
<?php
$stmt = $pdo->prepare(
    'UPDATE students SET name = :name WHERE id = :id'
);

$stmt->execute([
    'name' => $name,
    'id' => $id,
]);
```

### ចម្លើយលំហាត់ទី ៥

```php
<?php
$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    http_response_code(422);
    exit('ID មិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare('DELETE FROM students WHERE id = :id');
$stmt->execute(['id' => $id]);
```

### ចម្លើយលំហាត់ទី ៦

Prepared Statement បំបែក SQL Command ចេញពីទិន្នន័យដែលអ្នកប្រើផ្ដល់ឱ្យ។ ដូច្នេះ Input មិនអាចប្ដូររចនាសម្ព័ន្ធ SQL បានដោយងាយ។ ប៉ុន្តែវាមិនពិនិត្យថាទិន្នន័យត្រូវតាមច្បាប់កម្មវិធីឬអត់ទេ; ឧទាហរណ៍ Email នៅតែត្រូវ Validate ថាមានទម្រង់ត្រឹមត្រូវ។

---

## សេចក្ដីសង្ខេប

- PDO ផ្ដល់ Interface ស្របគ្នាសម្រាប់ភ្ជាប់ទៅ Database ហើយ MySQL ត្រូវការ Driver `pdo_mysql`។
- DSN កំណត់ Driver, Host, Database Name និង Charset។
- ប្រើ Prepared Statements សម្រាប់ SQL ដែលមាន Input ពីអ្នកប្រើ។
- CRUD មាន Create, Read, Update និង Delete។
- Validate ទិន្នន័យនៅ Server និង Escape នៅពេលបង្ហាញ HTML។
- រក្សា Database Credentials ក្រៅ Source Code និងកុំ Commit Secret ទៅ Repository។
- Log កំហុសសម្រាប់អ្នកអភិវឌ្ឍន៍ ប៉ុន្តែបង្ហាញតែសារទូទៅទៅអ្នកប្រើ។

## មេរៀនបន្ទាប់

មេរៀនបន្ទាប់អាចបន្តលើ Database ដោយផ្ដោតលើ Schema Design, Relationships, JOIN និង Pagination មុនពេលយកវាទៅបង្កើតកម្មវិធីពេញលេញ។
