# មេរៀនទី ១៩៖ រៀបចំកម្មវិធី CRUD ដោយ PHP និង PDO

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- រៀបចំ PHP Project ជា Files ដែលមានតួនាទីច្បាស់។
- បែងចែក Database Access, Request Handling និង HTML View។
- បង្កើត Form សម្រាប់ Create, Read, Update និង Delete។
- ប្រើ POST/Redirect/GET ដើម្បីជៀសវាង Submit ទិន្នន័យដដែលម្ដងទៀត។
- បន្ថែម CSRF Token ទៅ Form ដែលផ្លាស់ប្ដូរទិន្នន័យ។
- បន្តប្រើ PDO Prepared Statements, Server-side Validation និង HTML Escaping។
- រៀបចំ Public Directory ដើម្បីកុំឱ្យអ្នកចូលមើលឯកសារខាងក្នុងដោយផ្ទាល់។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL, PDO MySQL និងចំណេះដឹងពី Form, Session និង PDO នៅមេរៀនមុនៗ។

---

## ១. CRUD ជាអ្វី?

CRUD ជាឈ្មោះខ្លីសម្រាប់ប្រតិបត្តិការទិន្នន័យមូលដ្ឋាន៖

| អក្សរ | អត្ថន័យ | SQL |
|---|---|---|
| Create | បង្កើតទិន្នន័យថ្មី | INSERT |
| Read | អានទិន្នន័យ | SELECT |
| Update | កែប្រែទិន្នន័យ | UPDATE |
| Delete | លុបទិន្នន័យ | DELETE |

កម្មវិធីគ្រប់គ្រងសិស្សអាចឱ្យអ្នកប្រើមើលបញ្ជីសិស្ស បន្ថែមសិស្ស កែឈ្មោះ/Email និងលុបកំណត់ត្រា។

## ២. រៀបចំរចនាសម្ព័ន្ធ Project

បង្កើតរចនាសម្ព័ន្ធដូចខាងក្រោម៖

```text
student-crud/
├── public/
│   ├── index.php
│   ├── create.php
│   ├── edit.php
│   └── delete.php
└── src/
    ├── bootstrap.php
    └── student_repository.php
```

- `public/` មានឯកសារដែល Browser អាចស្នើបាន។
- `src/` មាន Database Connection និង Logic ដែលមិនគួរបើកមើលដោយផ្ទាល់។
- Web Server គួរកំណត់ Document Root ទៅ `public/`។ កុំបង្ហោះ Root ទាំងមូលជាថតសាធារណៈ ប្រសិនបើអាចកំណត់បាន។

ការបែងចែកនេះធ្វើឱ្យងាយស្វែងរកកូដ និងកាត់បន្ថយការលាយបញ្ចូល SQL, Form Handling និង HTML នៅកន្លែងតែមួយ។

## ៣. Bootstrap៖ Session និង Database

បង្កើត `src/bootstrap.php`៖

```php
<?php
declare(strict_types=1);

$httpsEnabled = isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off';

ini_set('session.use_strict_mode', '1');

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => $httpsEnabled,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();

if (!isset($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

function csrfToken(): string
{
    return $_SESSION['csrf_token'];
}

function verifyCsrfToken(mixed $submittedToken): bool
{
    return is_string($submittedToken)
        && isset($_SESSION['csrf_token'])
        && is_string($_SESSION['csrf_token'])
        && hash_equals($_SESSION['csrf_token'], $submittedToken);
}

function escapeHtml(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}

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

require_once __DIR__ . '/student_repository.php';
```

ចំណុចសំខាន់ៗ៖

- ហៅ `session_start()` មុនបញ្ចេញ HTML ឬ Output។
- Session Cookie មាន `HttpOnly` និង `SameSite=Lax`; `Secure` ត្រូវបើកនៅលើ HTTPS។
- កំណត់ Database Credentials ជា Environment Variables មិនសរសេរពាក្យសម្ងាត់ពិតក្នុង Repository។
- Token ពី `random_bytes()` ប្រើសម្រាប់ CSRF Protection។
- `hash_equals()` ប្រៀបធៀប Token ដោយសុវត្ថិភាពជាងប្រៀបធៀប String ធម្មតា។
- Reverse Proxy អាចធ្វើឱ្យការស្គាល់ HTTPS ត្រូវការកំណត់បន្ថែមនៅ Web Server; កុំទុកចិត្ត Header ពី Client ដោយផ្ទាល់។

## ៤. Repository៖ ដាក់ Database Queries នៅកន្លែងតែមួយ

បង្កើត `src/student_repository.php`៖

```php
<?php
declare(strict_types=1);

function findAllStudents(PDO $pdo): array
{
    $stmt = $pdo->query(
        'SELECT id, name, email, created_at
         FROM students
         ORDER BY id DESC'
    );

    return $stmt->fetchAll();
}

function findStudentById(PDO $pdo, int $id): array|false
{
    $stmt = $pdo->prepare(
        'SELECT id, name, email
         FROM students
         WHERE id = :id'
    );
    $stmt->execute(['id' => $id]);

    return $stmt->fetch();
}

function createStudent(PDO $pdo, string $name, string $email): void
{
    $stmt = $pdo->prepare(
        'INSERT INTO students (name, email)
         VALUES (:name, :email)'
    );

    $stmt->execute([
        'name' => $name,
        'email' => $email,
    ]);
}

function updateStudent(
    PDO $pdo,
    int $id,
    string $name,
    string $email
): bool {
    $stmt = $pdo->prepare(
        'UPDATE students
         SET name = :name, email = :email
         WHERE id = :id'
    );

    $stmt->execute([
        'id' => $id,
        'name' => $name,
        'email' => $email,
    ]);

    return $stmt->rowCount() > 0;
}

function deleteStudent(PDO $pdo, int $id): bool
{
    $stmt = $pdo->prepare(
        'DELETE FROM students WHERE id = :id'
    );
    $stmt->execute(['id' => $id]);

    return $stmt->rowCount() > 0;
}
```

Repository Function ទទួល PDO និងតម្លៃដែលបាន Validate រួច។ វាប្រើ Prepared Statements សម្រាប់ Query ដែលមាន Input។

## ៥. Read៖ បង្ហាញបញ្ជីសិស្ស

បង្កើត `public/index.php`៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__) . '/src/bootstrap.php';

$students = findAllStudents($pdo);
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

    <p><a href="create.php">បន្ថែមសិស្ស</a></p>

    <?php if ($students === []): ?>
        <p>មិនទាន់មានទិន្នន័យទេ។</p>
    <?php else: ?>
        <table>
            <thead>
                <tr>
                    <th>ឈ្មោះ</th>
                    <th>Email</th>
                    <th>សកម្មភាព</th>
                </tr>
            </thead>
            <tbody>
                <?php foreach ($students as $student): ?>
                    <tr>
                        <td><?= escapeHtml($student['name']) ?></td>
                        <td><?= escapeHtml($student['email']) ?></td>
                        <td>
                            <a href="edit.php?id=<?= (int) $student['id'] ?>">
                                កែប្រែ
                            </a>

                            <form method="POST" action="delete.php">
                                <input
                                    type="hidden"
                                    name="csrf_token"
                                    value="<?= escapeHtml(csrfToken()) ?>"
                                >
                                <input
                                    type="hidden"
                                    name="id"
                                    value="<?= (int) $student['id'] ?>"
                                >
                                <button type="submit">លុប</button>
                            </form>
                        </td>
                    </tr>
                <?php endforeach; ?>
            </tbody>
        </table>
    <?php endif; ?>
</body>
</html>
```

ទិន្នន័យពី Database ត្រូវ Escape មុនបង្ហាញ។ ID ត្រូវបានបម្លែងទៅ Integer មុនដាក់ក្នុង HTML Attribute។

## ៦. Create៖ បង្កើតកំណត់ត្រាថ្មី

បង្កើត `public/create.php`៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__) . '/src/bootstrap.php';

$errors = [];
$name = '';
$email = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!verifyCsrfToken($_POST['csrf_token'] ?? null)) {
        http_response_code(403);
        exit('សំណើមិនត្រឹមត្រូវ។ សូមផ្ទុកទំព័រឡើងវិញ។');
    }

    $name = trim($_POST['name'] ?? '');
    $email = trim($_POST['email'] ?? '');

    if ($name === '' || mb_strlen($name) > 100) {
        $errors[] = 'ឈ្មោះត្រូវមានពី 1 ដល់ 100 តួអក្សរ។';
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = 'សូមបញ្ចូល Email ដែលត្រឹមត្រូវ។';
    }

    if ($errors === []) {
        try {
            createStudent($pdo, $name, $email);

            header('Location: index.php', true, 303);
            exit;
        } catch (PDOException $e) {
            error_log('Create student failed: ' . $e->getMessage());
            $errors[] = 'មិនអាចរក្សាទុកបានទេ។ Email ប្រហែលត្រូវបានប្រើរួចហើយ។';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>បន្ថែមសិស្ស</title>
</head>
<body>
    <h1>បន្ថែមសិស្ស</h1>

    <?php foreach ($errors as $error): ?>
        <p><?= escapeHtml($error) ?></p>
    <?php endforeach; ?>

    <form method="POST">
        <input
            type="hidden"
            name="csrf_token"
            value="<?= escapeHtml(csrfToken()) ?>"
        >

        <label for="name">ឈ្មោះ៖</label>
        <input
            id="name"
            name="name"
            maxlength="100"
            value="<?= escapeHtml($name) ?>"
            required
        >

        <label for="email">Email៖</label>
        <input
            id="email"
            name="email"
            type="email"
            value="<?= escapeHtml($email) ?>"
            required
        >

        <button type="submit">រក្សាទុក</button>
    </form>
</body>
</html>
```

## ៧. Update៖ កែប្រែកំណត់ត្រា

បង្កើត `public/edit.php`។ ត្រូវ Validate ID ទាំងពេល GET ដើម្បីបង្ហាញ Form និងពេល POST ដើម្បីរក្សាទុក៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__) . '/src/bootstrap.php';

$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    http_response_code(400);
    exit('លេខសម្គាល់មិនត្រឹមត្រូវ។');
}

$student = findStudentById($pdo, $id);

if ($student === false) {
    http_response_code(404);
    exit('រកមិនឃើញសិស្សទេ។');
}

$errors = [];
$name = $student['name'];
$email = $student['email'];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (!verifyCsrfToken($_POST['csrf_token'] ?? null)) {
        http_response_code(403);
        exit('សំណើមិនត្រឹមត្រូវ។ សូមផ្ទុកទំព័រឡើងវិញ។');
    }

    $name = trim($_POST['name'] ?? '');
    $email = trim($_POST['email'] ?? '');

    if ($name === '' || mb_strlen($name) > 100) {
        $errors[] = 'ឈ្មោះត្រូវមានពី 1 ដល់ 100 តួអក្សរ។';
    }

    if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors[] = 'សូមបញ្ចូល Email ដែលត្រឹមត្រូវ។';
    }

    if ($errors === []) {
        try {
            updateStudent($pdo, $id, $name, $email);

            header('Location: index.php', true, 303);
            exit;
        } catch (PDOException $e) {
            error_log('Update student failed: ' . $e->getMessage());
            $errors[] = 'មិនអាចកែប្រែទិន្នន័យបានទេ។';
        }
    }
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>កែប្រែសិស្ស</title>
</head>
<body>
    <h1>កែប្រែសិស្ស</h1>

    <?php foreach ($errors as $error): ?>
        <p><?= escapeHtml($error) ?></p>
    <?php endforeach; ?>

    <form method="POST">
        <input
            type="hidden"
            name="csrf_token"
            value="<?= escapeHtml(csrfToken()) ?>"
        >

        <label for="name">ឈ្មោះ៖</label>
        <input
            id="name"
            name="name"
            maxlength="100"
            value="<?= escapeHtml($name) ?>"
            required
        >

        <label for="email">Email៖</label>
        <input
            id="email"
            name="email"
            type="email"
            value="<?= escapeHtml($email) ?>"
            required
        >

        <button type="submit">រក្សាទុកការកែប្រែ</button>
    </form>
</body>
</html>
```

## ៨. Delete៖ លុបកំណត់ត្រា

ការលុបត្រូវប្រើ POST និង CSRF Token។ កុំលុបទិន្នន័យដោយ GET Link ព្រោះ Browser, Crawler ឬ CSRF Request អាចបើក Link ដោយមិនមានបំណងច្បាស់។

បង្កើត `public/delete.php`៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__) . '/src/bootstrap.php';

if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    header('Allow: POST');
    http_response_code(405);
    exit('Method មិនត្រូវបានអនុញ្ញាត។');
}

if (!verifyCsrfToken($_POST['csrf_token'] ?? null)) {
    http_response_code(403);
    exit('សំណើមិនត្រឹមត្រូវ។');
}

$id = filter_input(INPUT_POST, 'id', FILTER_VALIDATE_INT);

if ($id === false || $id === null || $id < 1) {
    http_response_code(400);
    exit('លេខសម្គាល់មិនត្រឹមត្រូវ។');
}

deleteStudent($pdo, $id);

header('Location: index.php', true, 303);
exit;
```

## ៩. POST/Redirect/GET

ក្រោយ Form POST ជោគជ័យ កូដបញ្ជូន Browser ទៅទំព័រ GET ដោយប្រើ៖

```php
header('Location: index.php', true, 303);
exit;
```

នេះហៅថា Post/Redirect/Get (PRG)៖

1. Browser ផ្ញើ POST ដើម្បីបង្កើត ឬកែប្រែ។
2. Server ដំណើរការសំណើ រួចឆ្លើយតប Redirect។
3. Browser ស្នើទំព័រលទ្ធផលដោយ GET។

វាជួយឱ្យ Refresh ទំព័រលទ្ធផលមិនផ្ញើ Form ដដែលម្ដងទៀត។ Redirect មិនជំនួសការការពារ Request ស្ទួននៅកម្មវិធីទាំងអស់ទេ; ប្រតិបត្តិការសំខាន់ៗអាចត្រូវការ Idempotency បន្ថែម។

## ១០. CSRF និងការការពារ Form

CSRF (Cross-Site Request Forgery) គឺការបោក Browser ដែលបាន Login រួចឱ្យផ្ញើ Request ដោយអ្នកប្រើមិនបានបំណង។ Form ដែលបង្កើត កែប្រែ ឬលុបទិន្នន័យគួរមាន CSRF Token៖

1. Server បង្កើត Token ពិបាកទាយ និងរក្សាទុកក្នុង Session។
2. Form ផ្ញើ Token មកវិញក្នុង POST Body។
3. Server ប្រៀបធៀប Token ពី Request ជាមួយ Token ក្នុង Session។
4. បដិសេធ Request បើ Token ខុស ឬបាត់។

`SameSite` Cookie ជាជំនួយការពារ ប៉ុន្តែមិនជំនួស CSRF Token សម្រាប់ Form ទាំងអស់ទេ។ ត្រូវប្រើ HTTPS នៅ Production និងការពារ Session Cookie តាមការកំណត់របស់ Server។

## ១១. សាកល្បង Project

1. បង្កើត Database និង Table `students` តាមមេរៀនទី ១៦។
2. កំណត់ `DB_HOST`, `DB_NAME`, `DB_USER`, និង `DB_PASSWORD` ក្នុងបរិស្ថាន PHP។
3. បើក Terminal នៅក្នុង Folder `student-crud`។
4. ដំណើរការ៖

```bash
php -S localhost:8000 -t public
```

5. បើក `http://localhost:8000/index.php`។
6. បន្ថែម Student, កែប្រែ Student និងលុប Student។
7. សាកល្បងទិន្នន័យខុស និង Request ដែលគ្មាន CSRF Token។

PHP Built-in Server សម្រាប់ Local Development ប៉ុណ្ណោះ។ សម្រាប់ Hosting ពិត ត្រូវកំណត់ Web Server Document Root ទៅ Folder `public/` និងប្រើ HTTPS។

## ១២. Best Practices

1. **បំបែកកូដតាមតួនាទី។** Connection និង Query នៅក្រៅ Template។
2. **Validate នៅ Server។** HTML `required` មិនជំនួស Validation ទេ។
3. **ប្រើ Prepared Statements។** Query ដែលមាន Input ត្រូវប្រើ Parameter។
4. **Escape មុនបញ្ចូលក្នុង HTML។** ប្រើ Escape តាម Context។
5. **ការពារ State-changing Form ដោយ CSRF Token។**
6. **ប្រើ POST សម្រាប់ Create, Update និង Delete។**
7. **ប្រើ PRG ក្រោយ POST ជោគជ័យ។**
8. **លាក់ Files ខាងក្នុងក្រៅ Public Web Root។**
9. **រក្សា Credentials ក្នុង Environment Variables ឬ Secret Manager។**
10. **ប្រើ Authentication និង Authorization ក្នុងកម្មវិធីពិត។** CSRF Token មិនបញ្ជាក់ថាអ្នកប្រើមានសិទ្ធិកែ ឬលុប Record នោះទេ។

## ១៣. កំហុសដែលជួបញឹកញាប់

- **ដាក់ `database.php` ក្នុង Public Directory៖** អាចបង្កហានិភ័យបញ្ចេញ Configuration។
- **ទុកចិត្ត HTML Validation៖** Request អាចផ្ញើដោយមិនប្រើ Browser Form។
- **ប្រើ GET ដើម្បីលុបទិន្នន័យ៖** សកម្មភាពផ្លាស់ប្ដូរទិន្នន័យត្រូវការបច្ចេកទេស POST និង CSRF Protection។
- **ភ្លេច `exit` ក្រោយ Redirect៖** កូដបន្ទាប់អាចបន្តដំណើរការ។
- **បង្ហាញ PDOException ដល់អ្នកប្រើ៖** Log ព័ត៌មានលម្អិត ហើយបង្ហាញសារទូទៅ។
- **មិន Escape Database Output៖** ទិន្នន័យដែលបានរក្សាទុកក៏អាចមាន HTML មិនទុកចិត្តបាន។
- **គិតថា CSRF Token ផ្តល់សិទ្ធិអ្នកប្រើ៖** ត្រូវពិនិត្យ Authentication និង Authorization ដាច់ដោយឡែក។
- **កែ Session Cookie បន្ទាប់ពី Output៖** Header និង Cookie ត្រូវកំណត់មុន Output។

## ១៤. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ បន្ថែមតារាង Product

បង្កើត Repository និងទំព័របញ្ជី Product ដែលមានឈ្មោះ និងតម្លៃ។

### លំហាត់ទី ២៖ Create Form

បង្កើត Form បន្ថែម Product។ ត្រូវ Validate ឈ្មោះ និងតម្លៃនៅ Server ហើយប្រើ Prepared Statement។

### លំហាត់ទី ៣៖ Update Form

បង្កើតទំព័រកែប្រែ Product តាម ID។ ត្រូវពិនិត្យថា ID មានសុពលភាព និង Product មានក្នុង Database។

### លំហាត់ទី ៤៖ Delete Form

បង្កើត Delete Form ដែលប្រើ POST និង CSRF Token។ បន្ទាប់ពីលុប ត្រូវ Redirect ទៅបញ្ជី។

### លំហាត់ទី ៥៖ PRG

កែ Form Create ឱ្យ Redirect ទៅបញ្ជីក្រោយជោគជ័យ។ Refresh ទំព័របញ្ជី ហើយពិនិត្យថាមិនបង្កើត Record ស្ទួន។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Validation, Prepared Statements, HTML Escaping, CSRF Protection និង Authorization។

## ១៥. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- **Validation** ពិនិត្យថាទិន្នន័យតាមច្បាប់កម្មវិធីឬអត់។
- **Prepared Statements** បំបែក SQL Command ចេញពី Input ដើម្បីការពារ SQL Injection។
- **HTML Escaping** បម្លែងតួអក្សរពិសេសមុនបង្ហាញក្នុង HTML ដើម្បីកាត់បន្ថយ XSS។
- **CSRF Protection** បញ្ជាក់ថា Form Request មកពី Session/Page ដែលកម្មវិធីបានបង្កើត។
- **Authorization** ពិនិត្យថាអ្នកប្រើមានសិទ្ធិធ្វើសកម្មភាពលើ Resource នោះឬអត់។

ចំណុចទាំងនេះការពារបញ្ហាខុសៗគ្នា ដូច្នេះមិនអាចយកមួយជំនួសមួយទៀតបានទេ។

---

## សេចក្ដីសង្ខេប

- CRUD គឺ Create, Read, Update និង Delete។
- បំបែក Public Entry Points, Bootstrap និង Repository Functions។
- Validate Input, ប្រើ Prepared Statements និង Escape Output។
- ប្រើ CSRF Token សម្រាប់ Form ដែលផ្លាស់ប្ដូរទិន្នន័យ។
- ប្រើ POST/Redirect/GET ដើម្បីកុំឱ្យ Browser Submit Form ដដែលពេល Refresh។
- កំណត់ Public Web Root ទៅ `public/` និងរក្សា Secret ក្រៅ Source Code។
- CSRF មិនជំនួស Authentication ឬ Authorization ទេ។

## ឯកសារយោងផ្លូវការ

- [PHP Sessions និងការការពារ Session Settings](https://www.php.net/manual/en/session.security.ini.php)
- [PHP session_regenerate_id()](https://www.php.net/manual/en/function.session-regenerate-id.php)
- [PHP header() និង Redirect](https://www.php.net/manual/en/function.header.php)

## មេរៀនបន្ទាប់

អាចបន្តទៅ OOP ដោយរៀន Classes, Objects, Properties, Methods និង Constructor ដើម្បីរៀបចំ PHP Code ជា Components ដែលអាចប្រើឡើងវិញបាន។
