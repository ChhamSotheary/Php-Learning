# មេរៀនទី ១១៖ PHP Sessions និង Cookies

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពី HTTP Request និងហេតុអ្វី Server មិនចងចាំអ្នកប្រើដោយស្វ័យប្រវត្តិ។
- ពន្យល់ពីភាពខុសគ្នារវាង Session និង Cookie។
- ចាប់ផ្តើម និងប្រើ Session ដោយ `session_start()` និង `$_SESSION`។
- បង្កើត និងអាន Cookie ដោយ `setcookie()` និង `$_COOKIE`។
- រៀបចំ Login/Logout Demo ដោយប្រើ Session។
- អនុវត្តការការពារ Session ជាមូលដ្ឋាន។

## ២. HTTP ជា Stateless Protocol

HTTP ត្រូវបានចាត់ទុកថាជា Stateless Protocol។ មានន័យថា Request មួយៗមិនចងចាំ Request មុនដោយស្វ័យប្រវត្តិទេ។

ឧទាហរណ៍៖

1. អ្នកចូលទៅ `login.php` ហើយ Login ជោគជ័យ។
2. បន្ទាប់មកអ្នកចូលទៅ `profile.php`។
3. Server ត្រូវការវិធីដឹងថា Request ថ្មីនេះជារបស់អ្នកប្រើដែលបាន Login មុន។

Session និង Cookie ជួយដោះស្រាយបញ្ហានេះ។

## ៣. តើ Session ជាអ្វី?

Session គឺជាវិធីរក្សាទុកទិន្នន័យរបស់អ្នកប្រើនៅខាង Server រវាង Request ច្រើន។

ឧទាហរណ៍ ទិន្នន័យ Session អាចមាន៖

- User ID
- Username
- Login Status
- តម្លៃផ្សេងៗដែលត្រូវការរក្សាទុកបណ្តោះអាសន្ននៅ Server

ជាទូទៅ Browser រក្សាទុក Session ID ក្នុង Cookie ហើយផ្ញើ Session ID នោះមក Server ក្នុង Request បន្ទាប់។ ទិន្នន័យ Session ពិតប្រាកដត្រូវបានរក្សាទុកនៅ Server។

## ៤. ចាប់ផ្តើម Session ដោយ session_start()

មុនប្រើ `$_SESSION` ត្រូវចាប់ផ្តើម Session៖

```php
<?php
session_start();

$_SESSION['username'] = 'Dara';

echo 'សួស្តី ' . $_SESSION['username'];
```

លទ្ធផល៖

```
សួស្តី Dara
```

**ចំណាំសំខាន់៖** `session_start()` ត្រូវហៅមុនបញ្ចេញ HTML ឬ Output ទៅ Browser ព្រោះ PHP ត្រូវកំណត់ Session Cookie ក្នុង HTTP Header។

ឧទាហរណ៍ត្រឹមត្រូវ៖

```php
<?php
session_start();
?>
<!DOCTYPE html>
<html lang="km">
<body>
    <h1>ទំព័រ Session</h1>
</body>
</html>
```

## ៥. រក្សាទុក និងអាន Session

រក្សាទុកទិន្នន័យ៖

```php
<?php
session_start();

$_SESSION['name'] = 'Sokha';
$_SESSION['role'] = 'student';
$_SESSION['logged_in'] = true;
```

អានទិន្នន័យនៅ File ផ្សេង៖

```php
<?php
session_start();

echo $_SESSION['name'] ?? 'មិនទាន់មានឈ្មោះ';
echo '<br>';
echo $_SESSION['role'] ?? 'មិនទាន់មានតួនាទី';
```

ចំណាំ៖ File នីមួយៗដែលត្រូវប្រើ Session ត្រូវហៅ `session_start()` មុនប្រើ `$_SESSION`។

## ៦. លុប Session Data

លុបតម្លៃមួយ៖

```php
<?php
session_start();

unset($_SESSION['name']);
```

លុបទិន្នន័យ Session ទាំងអស់៖

```php
<?php
session_start();

$_SESSION = [];
session_destroy();
```

`session_destroy()` បំផ្លាញទិន្នន័យ Session នៅ Server ប៉ុន្តែមិនសម្អាត Array `$_SESSION` ក្នុង Memory ដោយស្វ័យប្រវត្តិទេ។ ដូច្នេះការកំណត់ `$_SESSION = [];` ជួយសម្អាតតម្លៃក្នុង Request បច្ចុប្បន្នផងដែរ។

សម្រាប់ Logout ពិតប្រាកដ គួរលុប Session Cookie នៅ Browser ផងដែរ ដោយប្រើ Cookie Parameters ដែលត្រូវនឹង Session Cookie របស់កម្មវិធី។ ឧទាហរណ៍នៅផ្នែកក្រោយ។

## ៧. តើ Cookie ជាអ្វី?

Cookie គឺជាទិន្នន័យតូចៗដែល Website ស្នើឱ្យ Browser រក្សាទុក។ Browser អាចផ្ញើ Cookie ត្រឡប់ទៅ Server ក្នុង Request បន្ទាប់ៗ តាម Domain និង Path ដែលបានកំណត់។

ឧទាហរណ៍ការប្រើ Cookie៖

- រក្សាទុកចំណង់ចំណូលចិត្តភាសា។
- រក្សាទុក Theme (Light/Dark)។
- រក្សាទុកការកំណត់ Website មួយចំនួន។

កុំរក្សាទុក Password ឬ Secret ដោយផ្ទាល់ក្នុង Cookie។

## ៨. បង្កើត Cookie ដោយ setcookie()

`setcookie()` ត្រូវហៅមុន Output ដូច Session ដែរ។

```php
<?php
setcookie(
    'language',
    'km',
    [
        'expires' => time() + 3600,
        'path' => '/',
        'secure' => !empty($_SERVER['HTTPS']),
        'httponly' => true,
        'samesite' => 'Lax'
    ]
);

echo 'Cookie ត្រូវបានកំណត់សម្រាប់ Browser។';
```

ពន្យល់៖

- `language` ជាឈ្មោះ Cookie។
- `km` ជាតម្លៃ Cookie។
- `expires` ជាពេលផុតកំណត់ (Unix Timestamp)។
- `path` កំណត់ Path ដែល Browser ផ្ញើ Cookie។
- `secure` ធ្វើឱ្យ Cookie ផ្ញើតែតាម HTTPS នៅពេលកំណត់ជា true។
- `httponly` មិនអនុញ្ញាត JavaScript អាន Cookie នោះ។
- `samesite` ជួយកាត់បន្ថយហានិភ័យ CSRF តាមរយៈការគ្រប់គ្រងការផ្ញើ Cookie ឆ្លង Site។

សម្រាប់ Production Website គួរប្រើ HTTPS ជានិច្ច និងកំណត់ `secure => true`។ ឧទាហរណ៍ខាងលើប្រើការត្រួតពិនិត្យ HTTPS ដើម្បីអាចសាកល្បងលើ Local HTTP បាន។

## ៩. អាន Cookie ដោយ $_COOKIE

បន្ទាប់ពី Browser ទទួល Cookie ហើយ ផ្ញើ Request ថ្មីមក Server នោះ PHP អាចអានតាម `$_COOKIE`៖

```php
<?php
$language = $_COOKIE['language'] ?? 'en';

echo htmlspecialchars($language, ENT_QUOTES, 'UTF-8');
```

**ចំណាំ៖** បន្ទាប់ពីហៅ `setcookie()` តម្លៃថ្មីមិនចូលក្នុង `$_COOKIE` ភ្លាមៗក្នុង Request ដដែលទេ។ ជាទូទៅ ត្រូវរង់ចាំ Request បន្ទាប់ពី Browser ផ្ញើ Cookie មកវិញ។

## ១០. លុប Cookie

កំណត់ Cookie ឱ្យផុតកំណត់នៅអតីតកាល៖

```php
<?php
setcookie(
    'language',
    '',
    [
        'expires' => time() - 3600,
        'path' => '/',
        'secure' => !empty($_SERVER['HTTPS']),
        'httponly' => true,
        'samesite' => 'Lax'
    ]
);
```

ដើម្បីលុប Cookie ត្រូវប្រើ Path និង Domain ដូចគ្នានឹងពេលបង្កើត Cookie។ ប្រសិនបើ Cookie ដើមមាន Domain ក៏ត្រូវផ្គូផ្គង Domain ដែរ។

## ១១. ភាពខុសគ្នារវាង Session និង Cookie

| ចំណុច | Session | Cookie |
|---|---|---|
| ទីតាំងទិន្នន័យ | ជាទូទៅនៅ Server | នៅ Browser |
| ការផ្ញើត្រឡប់ | តាម Session ID | Browser ផ្ញើ Cookie |
| ទំហំ | អាស្រ័យលើ Server Configuration | តូច និងមានកម្រិតតាម Browser |
| អាយុកាល | អាស្រ័យលើ Session Configuration និងការប្រើប្រាស់ | អាចកំណត់ Expires |
| ឧទាហរណ៍ | Login State | Language Preference |

កុំសន្មតថា Session មានអាយុកាលជាក់លាក់ដូចគ្នានៅគ្រប់ Server ទេ។ វាអាស្រ័យលើ PHP Configuration និងការកំណត់កម្មវិធី។

## ១២. Mini Project៖ Login និង Logout ដោយ Session

គម្រោងនេះជាការសិក្សា ដោយប្រើ Username និង Password សាកល្បងដែលកំណត់ក្នុងកូដ។ វាមិនមែនជា Login System សម្រាប់ Website ពិតទេ។ មេរៀន Database និង Password Hashing នឹងបង្រៀនការផ្ទៀងផ្ទាត់អ្នកប្រើប្រាស់ឱ្យបានត្រឹមត្រូវ។

បង្កើត Folder៖

```
php-session-demo/
│
├── login.php
├── dashboard.php
└── logout.php
```

### ១២.១ login.php

```php
<?php
session_start();

$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';

    // គណនីសាកល្បងសម្រាប់ការរៀនប៉ុណ្ណោះ។
    if ($username === 'student' && $password === 'demo123') {
        session_regenerate_id(true);

        $_SESSION['user'] = [
            'username' => $username,
            'logged_in' => true
        ];

        header('Location: dashboard.php');
        exit;
    }

    $error = 'Username ឬ Password មិនត្រឹមត្រូវ។';
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>Login Demo</title>
</head>
<body>
    <h1>ចូលគណនីសាកល្បង</h1>

    <?php if ($error !== ''): ?>
        <p><?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
    <?php endif; ?>

    <form method="POST">
        <label for="username">Username៖</label>
        <input id="username" name="username" required>

        <label for="password">Password៖</label>
        <input id="password" name="password" type="password" required>

        <button type="submit">Login</button>
    </form>

    <p>គណនីសាកល្បង៖ student / demo123</p>
</body>
</html>
```

### ១២.២ dashboard.php

```php
<?php
session_start();

if (empty($_SESSION['user']['logged_in'])) {
    header('Location: login.php');
    exit;
}

$username = $_SESSION['user']['username'] ?? 'Student';
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>Dashboard</title>
</head>
<body>
    <h1>សូមស្វាគមន៍ <?= htmlspecialchars($username, ENT_QUOTES, 'UTF-8') ?></h1>
    <p>អ្នកបាន Login ជោគជ័យ។</p>

    <a href="logout.php">Logout</a>
</body>
</html>
```

### ១២.៣ logout.php

```php
<?php
session_start();

// សម្អាតទិន្នន័យ Session នៅក្នុង Request នេះ។
$_SESSION = [];

// លុប Session Cookie ប្រសិនបើ Cookies ត្រូវបានប្រើ។
if (ini_get('session.use_cookies')) {
    $params = session_get_cookie_params();

    setcookie(
        session_name(),
        '',
        [
            'expires' => time() - 42000,
            'path' => $params['path'],
            'domain' => $params['domain'],
            'secure' => $params['secure'],
            'httponly' => $params['httponly'],
            'samesite' => $params['samesite'] ?? 'Lax'
        ]
    );
}

session_destroy();

header('Location: login.php');
exit;
```

### ១២.៤ របៀបសាកល្បង

1. បើក Terminal ក្នុង Folder `php-session-demo`។
2. ដំណើរការ៖

```bash
php -S localhost:8000
```

3. បើក `http://localhost:8000/login.php`។
4. សាកល្បង Username `student` និង Password `demo123`។
5. បន្ទាប់ពី Login ជោគជ័យ អ្នកនឹងទៅកាន់ Dashboard។
6. សាកល្បងបើក `dashboard.php` ដោយមិន Login ក្នុង Browser Private Session ថ្មី។
7. ចុច Logout ហើយព្យាយាមបើក Dashboard ម្តងទៀត។

លទ្ធផលរំពឹងទុក៖

- គណនីសាកល្បងត្រឹមត្រូវ៖ ចូល Dashboard។
- គណនីខុស៖ បង្ហាញ Error។
- មិនទាន់ Login៖ បញ្ជូនទៅ Login Page។
- Logout៖ លុប Session និងត្រឡប់ទៅ Login Page។

## ១៣. សុវត្ថិភាព Session និង Cookie

សម្រាប់កម្មវិធីពិត គួរអនុវត្តចំណុចខាងក្រោម៖

1. **ប្រើ HTTPS** ដើម្បីការពារទិន្នន័យពេលផ្ញើតាម Network។
2. **បើក HttpOnly** សម្រាប់ Session Cookie ដើម្បីកាត់បន្ថយការអាន Cookie តាម JavaScript។
3. **កំណត់ Secure** នៅលើ HTTPS Website។
4. **ប្រើ SameSite** ដូចជា `Lax` ជាជំនួយការពារ Cross-Site Request។
5. **Regenerate Session ID ក្រោយ Login** ដោយ `session_regenerate_id(true)` ដើម្បីកាត់បន្ថយ Session Fixation។
6. **កុំរក្សាទុក Password ក្នុង Session ឬ Cookie**។ រក្សាទុក User ID និងព័ត៌មានតិចបំផុតដែលត្រូវការ។
7. **កុំជឿ Cookie ពី Browser** ព្រោះអ្នកប្រើអាចកែតម្លៃ Cookie បាន។
8. **ប្រើ CSRF Token** សម្រាប់ Form ដែលផ្លាស់ប្តូរទិន្នន័យ។ SameSite Cookie ជាជំនួយ មិនមែនជំនួស CSRF Token ទាំងស្រុងទេ។
9. **កំណត់ Session Timeout** និងគ្រប់គ្រងការផុតកំណត់នៅ Server។
10. **កុំបង្ហាញ Session ID** ក្នុង URL ឬ Log ដែលអ្នកដទៃអាចមើលឃើញ។

## ១៤. កំហុសដែលជួបញឹកញាប់

1. **ភ្លេច `session_start()`** — ហៅវាមុនប្រើ `$_SESSION`។
2. **Output មុន `session_start()` ឬ `setcookie()`** — ដាក់ការកំណត់ Header មុន HTML ឬ Echo។
3. **រំពឹងថា Cookie មានតម្លៃថ្មីក្នុង Request ដដែល** — ត្រូវរង់ចាំ Request បន្ទាប់។
4. **មិនពិនិត្យថា User បាន Login ឬអត់** — ការពារទំព័រឯកជននៅ Server។
5. **មិន Regenerate Session ID ក្រោយ Login** — ប្រើ `session_regenerate_id(true)`។
6. **រក្សាទុកព័ត៌មានសម្ងាត់ក្នុង Cookie** — កុំដាក់ Password ឬ Secret នៅទីនោះ។
7. **លុប Cookie ដោយ Path មិនដូចដើម** — ត្រូវផ្គូផ្គង Path និង Domain។

## ១៥. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ User Session

បង្កើត File `profile.php` ដែល៖

- ចាប់ផ្តើម Session។
- រក្សាទុក Username ក្នុង Session។
- បង្ហាញ Username។
- បើមិនមាន Username បង្ហាញសារថាមិនទាន់ Login។

### លំហាត់ទី ២៖ Language Cookie

បង្កើតកម្មវិធីដែលអនុញ្ញាតឱ្យអ្នកប្រើជ្រើសភាសា `km` ឬ `en` ហើយរក្សាទុកជាមួយ Cookie រយៈពេលមួយថ្ងៃ។

### លំហាត់ទី ៣៖ Logout

បង្កើត `logout.php` ដែលសម្អាត Session និងបញ្ជូនអ្នកប្រើទៅ Login Page។

### លំហាត់ទី ៤៖ Protected Page

បង្កើត `admin.php` ដែលអនុញ្ញាតឱ្យចូលបានតែពេល Session មានតម្លៃ `logged_in` ត្រឹមត្រូវ។

## ១៦. Mini Project៖ Student Portal

បង្កើត Student Portal តូចមួយដែលមាន៖

- Login Page
- Dashboard
- Profile Page
- Logout
- Language Preference Cookie

តម្រូវការ៖

1. ប្រើ Session ដើម្បីគ្រប់គ្រង Login State។
2. ប្រើ `session_regenerate_id(true)` ក្រោយ Login។
3. ការពារទំព័រ Dashboard និង Profile នៅ Server។
4. ប្រើ Cookie រក្សាទុកភាសាដែលអ្នកប្រើជ្រើស។
5. Escape ទិន្នន័យមុនបង្ហាញក្នុង HTML។
6. កុំរក្សាទុក Password ក្នុង Cookie។
7. សរសេរចំណាំថា Login Demo មិនទាន់ភ្ជាប់ Database ហើយមិនត្រូវប្រើសម្រាប់ Website ពិតទេ។

## ១៧. សំណួររំលឹកមេរៀន

1. តើ HTTP Stateless មានន័យដូចម្តេច?
2. តើ Session រក្សាទុកទិន្នន័យជាទូទៅនៅទីណា?
3. តើ `session_start()` មានតួនាទីអ្វី?
4. តើ `$_SESSION` និង `$_COOKIE` ខុសគ្នាដូចម្តេច?
5. តើហេតុអ្វី `setcookie()` ត្រូវហៅមុន Output?
6. តើ `session_regenerate_id(true)` ប្រើសម្រាប់អ្វី?
7. តើហេតុអ្វីមិនគួររក្សាទុក Password ក្នុង Cookie?

## ១៨. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

1. Request មួយៗមិនចងចាំ Request មុនដោយស្វ័យប្រវត្តិទេ។
2. នៅ Server។
3. ចាប់ផ្តើម ឬបន្ត Session ដើម្បីអាចប្រើទិន្នន័យ Session។
4. Session Data ស្ថិតនៅ Server ជាទូទៅ; Cookie Data ស្ថិតនៅ Browser។
5. ព្រោះ Cookie ត្រូវកំណត់តាម HTTP Header មុនផ្ញើ Output។
6. បង្កើត Session ID ថ្មី ដើម្បីកាត់បន្ថយហានិភ័យ Session Fixation។
7. Cookie អាចត្រូវបានមើល ឬកែប្រែពី Browser ហើយមិនមែនជាកន្លែងសុវត្ថិភាពសម្រាប់ Password ទេ។

---

## សេចក្តីសង្ខេប

ក្នុងមេរៀននេះ អ្នកបានរៀនពី HTTP Stateless, Session, Cookie, `session_start()`, `$_SESSION`, `setcookie()`, `$_COOKIE`, Login/Logout Demo និងសុវត្ថិភាព Session។

**មេរៀនទី ១២ បន្ទាប់៖ PHP File Handling** — រៀនបើក អាន សរសេរ បន្ថែម និងគ្រប់គ្រង File ដោយ PHP។
