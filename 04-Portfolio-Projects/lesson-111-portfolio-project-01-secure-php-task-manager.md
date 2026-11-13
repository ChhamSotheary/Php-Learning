# មេរៀនទី ១១១៖ Portfolio Project ១ — Secure PHP Task Manager

## គោលបំណងគម្រោង

ក្នុងគម្រោងនេះ អ្នកនឹងបង្កើត Task Manager ដោយប្រើ PHP និង PDO ដែលអ្នកប្រើអាច៖

- ចុះឈ្មោះ និង login ដោយ password hashing។
- បង្កើត, មើល, កែប្រែ និងលុប tasks របស់ខ្លួន។
- ស្វែងរក និង filter tasks តាម status។
- ប្រើ CSRF protection, prepared statements និង output escaping។
- រៀបចំ README និងរូបថតអេក្រង់សម្រាប់ Portfolio។

គម្រោងនេះអនុវត្តមេរៀន PHP, MySQL/PDO, Sessions, Forms, Validation និង Security ដែលបានរៀនពីមុន។

## ១. តម្រូវការមុខងារ

### Authentication

- User ចុះឈ្មោះដោយឈ្មោះបង្ហាញ, email និង password។
- Email ត្រូវ unique និងត្រូវ validate។
- Password ត្រូវរក្សាទុកជា hash ដោយ `password_hash()`។
- Login ផ្ទៀងផ្ទាត់ដោយ `password_verify()`។
- Logout ត្រូវបញ្ចប់ session ហើយបញ្ជូនទៅ login page។

### Task Management

- User ដែល login អាចបង្កើត task ដែលមាន title, description, status និង due date។
- Status មានតម្លៃតែ `todo`, `in_progress`, `done`។
- អ្នកប្រើអាចកែ/លុបតែ task ដែលខ្លួនជាម្ចាស់។
- បញ្ជី task អាច filter តាម status និងស្វែងរកតាម title។
- ទិន្នន័យដែលបង្ហាញត្រូវ escape ដើម្បីកាត់បន្ថយ XSS។

## ២. Database Schema

បង្កើត database និង tables ឧទាហរណ៍៖

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(120) NOT NULL,
    email VARCHAR(190) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tasks (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    title VARCHAR(160) NOT NULL,
    description TEXT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'todo',
    due_date DATE NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT fk_tasks_user
        FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE,
    CONSTRAINT chk_tasks_status
        CHECK (status IN ('todo', 'in_progress', 'done')),
    INDEX idx_tasks_user_status (user_id, status),
    INDEX idx_tasks_user_created (user_id, created_at)
);
```

DBMS មួយចំនួនអាចមានខុសគ្នាលើការអនុវត្ត CHECK constraints។ រក្សា validation នៅក្នុង PHP ផង ហើយសាកល្បង schema ជាមួយ database version ដែលអ្នកប្រើពិត។

## ៣. រៀបចំ Project Structure

ប្រើ structure តូចមួយដែលបែងចែក public entry point, source code និង templates៖

```text
task-manager/
├── public/
│   ├── index.php
│   └── assets/
├── src/
│   ├── Database.php
│   ├── Auth.php
│   ├── Csrf.php
│   ├── TaskRepository.php
│   └── Validation.php
├── templates/
│   ├── auth/
│   └── tasks/
├── migrations/
│   └── 001_create_tables.sql
├── .env.example
├── .gitignore
└── README.md
```

កំណត់ web server document root ទៅ `public/` ដើម្បីកុំឱ្យ browser អាចចូល `src/`, `migrations/` ឬ configuration files ដោយផ្ទាល់។

## ៤. ភ្ជាប់ PDO ដោយ Prepared Statements

អាន database credentials ពី environment variables មិនមែន hard-code ក្នុង repository៖

```php
<?php

$dsn = sprintf(
    'mysql:host=%s;dbname=%s;charset=utf8mb4',
    getenv('DB_HOST'),
    getenv('DB_NAME'),
);

$pdo = new PDO($dsn, getenv('DB_USER'), getenv('DB_PASSWORD'), [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    PDO::ATTR_EMULATE_PREPARES => false,
]);
```

នៅ query ដែលប្រើ user input ត្រូវប្រើ prepared statements៖

```php
$stmt = $pdo->prepare(
    'SELECT id, title, status FROM tasks WHERE user_id = :user_id ORDER BY created_at DESC'
);
$stmt->execute(['user_id' => $currentUserId]);

$tasks = $stmt->fetchAll();
```

`user_id = :user_id` ជួយឱ្យ query ទាញតែ task របស់ user ដែលបាន login។ ត្រូវអនុវត្ត owner check នេះនៅលើ read, update និង delete ទាំងអស់។

## ៥. Password និង Session

ពេលចុះឈ្មោះ៖

```php
$passwordHash = password_hash($plainPassword, PASSWORD_DEFAULT);
```

ពេល login៖

```php
if (password_verify($plainPassword, $user['password_hash'])) {
    session_regenerate_id(true);
    $_SESSION['user_id'] = (int) $user['id'];
}
```

កំណត់ session cookie flags ឱ្យសមស្របមុនចាប់ផ្ដើម session៖ `HttpOnly`, `Secure` លើ HTTPS និង `SameSite`។ នៅក្នុង app ពិត ត្រូវកំណត់ cookie path/domain តាម deployment ហើយបញ្ចប់ session ពេល logout។

## ៦. CSRF និង Forms

បង្កើត CSRF token នៅ session ហើយដាក់ក្នុង hidden input សម្រាប់គ្រប់ form ដែលប្ដូរទិន្នន័យ។ ពេលទទួល POST request ត្រូវផ្ទៀងផ្ទាត់ token មុនធ្វើការបង្កើត/កែ/លុប។

ប្រើ POST សម្រាប់ mutations; កុំលុប task តាម GET link។ បន្ថែម method override ឬ route handling តាមដែលអ្នករចនា ប៉ុន្តែ server ត្រូវពិនិត្យសិទ្ធិជានិច្ច។

## ៧. Validation និង Output Escaping

Validate data នៅ server៖

- Title មិនទទេ និងមិនលើស 160 តួអក្សរ។
- Description មិនលើសកំណត់ដែលអ្នកកំណត់។
- Status ត្រូវស្ថិតក្នុង allowlist។
- Due date ត្រូវជាទម្រង់ YYYY-MM-DD ត្រឹមត្រូវ។
- ID ត្រូវជា integer វិជ្ជមាន។

ពេលបញ្ចូលទិន្នន័យទៅ HTML ត្រូវ escape៖

```php
function e(string $value): string
{
    return htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
```

ហៅ `e($task['title'])` នៅក្នុង template។ កុំបង្ហាញ user input ដោយផ្ទាល់ក្នុង HTML។

## ៨. គ្រប់គ្រងកំហុស និង Transactions

ចាប់ exceptions នៅ boundary របស់កម្មវិធី ដើម្បីកត់ត្រាបញ្ហា និងបង្ហាញសារទូទៅដល់អ្នកប្រើ។ កុំបង្ហាញ SQL error, stack trace ឬ credentials ក្នុង response។

ប្រសិនបើ operation មានការផ្លាស់ប្ដូរច្រើន rows ដែលត្រូវជោគជ័យជាមួយគ្នា ប្រើ PDO transaction៖

```php
$pdo->beginTransaction();

try {
    // execute related database changes
    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
```

កុំទទួល exception ហើយបន្តដោយមិន rollback ឬកត់ត្រា។

## ៩. Definition of Done

គម្រោងរួចរាល់សម្រាប់ Portfolio នៅពេល៖

- Registration, login, logout និង CRUD ដំណើរការតាម acceptance criteria។
- User A មិនអាចមើល ឬកែ task របស់ User B ទោះប្ដូរ ID ក្នុង URL ក៏ដោយ។
- គ្រប់ form មាន CSRF protection ហើយ validation ធ្វើនៅ server។
- Queries ដែលប្រើ user input ប្រើ prepared statements។
- HTML outputs ត្រូវបាន escape។
- Secrets មិនស្ថិតក្នុង Git history និង `.gitignore` មាន local env file។
- README មាន prerequisites, install/run steps, DB migration, features និង screenshots។
- Git history មាន commits តូចៗដែលពិពណ៌នាការងារច្បាស់។

## ១០. របៀបបង្ហាញក្នុង Portfolio

README គួរមាន៖

- សេចក្ដីសង្ខេបនៃបញ្ហា និងអ្នកប្រើគោលដៅ។
- Feature list និង stack (PHP, PDO, MySQL)។
- របៀបដំឡើង និងបង្កើត database ពី clean checkout។
- រូបថត login, task list, filter និង form validation។
- សេចក្ដីពន្យល់អំពី ownership check, CSRF, password hashing និង prepared statements។
- Known limitations និង next steps។

កុំបង្ហោះ demo ដែលមាន test credentials ឬទិន្នន័យផ្ទាល់ខ្លួនពិត។

## លំហាត់ពង្រីក

1. បន្ថែម pagination ហើយកំណត់ចំនួន row ក្នុងមួយទំព័រ។
2. បន្ថែម priority និង filter ដោយ prepared query parameters។
3. សរសេរ tests សម្រាប់ Auth, validation និង ownership checks។
4. បង្កើត seed data សម្រាប់ local demo ដោយមិនប្រើ production data។
5. ដាក់ app ទៅ staging ហើយកត់ត្រាដំណើរការ deployment ក្នុង README។

## សំណួរត្រួតពិនិត្យ

1. តើ prepared statement ការពារបញ្ហាអ្វី?
2. តើហេតុអ្វីគ្រប់ task query ត្រូវពិនិត្យ user_id?
3. តើ password hash ខុសពី encryption ដូចម្ដេច?
4. តើ CSRF token ត្រូវផ្ទៀងផ្ទាត់នៅពេលណា?
5. តើអ្វីខ្លះបញ្ជាក់ថាគម្រោងអាចទទួលការពិនិត្យ Portfolio បាន?

## ឯកសារយោង

- [PHP PDO Manual](https://www.php.net/manual/en/book.pdo.php)
- [PHP Password Hashing](https://www.php.net/manual/en/book.password.php)
- [PHP Session Security](https://www.php.net/manual/en/session.security.ini.php)
- [PHP htmlspecialchars](https://www.php.net/manual/en/function.htmlspecialchars.php)
