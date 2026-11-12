# មេរៀនទី ៣១៖ សុវត្ថិភាព Session និង Cookie ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីរបៀបដែល Session ID ភ្ជាប់ Browser ទៅនឹងទិន្នន័យ Session នៅ Server។
- កំណត់ Cookie ជាមួយ `Secure`, `HttpOnly`, `SameSite` និង Path សមស្រប។
- ការពារ Session Fixation ដោយបង្កើត Session ID ថ្មីពេល Sign In។
- បញ្ចប់ Session នៅ Server និងលុប Cookie ពេល Sign Out។
- យល់ថា Session ID គឺជា Secret ដូចជា Credential មួយ។

> តម្រូវការ៖ PHP 8+ និង HTTPS សម្រាប់ Website ដែលមាន Login។

## ១. Session ដំណើរការយ៉ាងដូចម្តេច?

Server រក្សាទិន្នន័យ Session ដូចជា User ID។ Browser ទទួល Cookie ដែលមាន Session ID ហើយផ្ញើវាមកវិញក្នុង Request បន្ទាប់ៗ។ Session ID មិនគួរដាក់ក្នុង URL ឬបង្ហាញក្នុង HTML ទេ ព្រោះអ្នកណាដែលបានវាអាចក្លែងជា User បាន។

កុំដាក់ Password, Access Token ឬ Secret មិនចាំបាច់ក្នុង Session។ រក្សាទុកតែព័ត៌មានដែលចាំបាច់ ហើយកំណត់ Session Storage ឱ្យត្រូវនឹងកម្មវិធី/Server របស់អ្នក។

## ២. កំណត់ Cookie Attributes មុន `session_start()`

កំណត់ជម្រើសមុនចាប់ផ្តើម Session និងមុនមាន Output៖

```php
<?php
$httpsEnabled = true; // កំណត់តាម HTTPS ពិតរបស់ Deployment

session_set_cookie_params([
    'lifetime' => 0,       // Cookie បិទពេលបិទ Browser
    'path' => '/',
    'domain' => '',        // កុំកំណត់ Domain ធំជាងចាំបាច់
    'secure' => $httpsEnabled,
    'httponly' => true,
    'samesite' => 'Lax',
]);

ini_set('session.use_only_cookies', '1');
ini_set('session.use_strict_mode', '1');

session_start();
```

ក្នុង Production ដែលប្រើ HTTPS គ្រប់ទីកន្លែង ត្រូវធានាថា `$httpsEnabled` ពិតជាត្រូវជា `true`។ កុំទុកជម្រើសនេះឱ្យ Client គ្រប់គ្រង។ `SameSite=Lax` ជាជម្រើសទូទៅមួយ; Flow ពិសេសដូចជា Cross-site Login អាចត្រូវការការកំណត់ផ្សេង និងការការពារបន្ថែម។ SameSite មិនមែនជាការជំនួស CSRF Token ទេ។

## ៣. បង្កើត Session ID ថ្មីពេល Sign In

កុំរក្សា Session ID ដដែលពីមុនពេល User Sign In។ បន្ទាប់ពីផ្ទៀងផ្ទាត់ Password ជោគជ័យ៖

```php
<?php
// ក្រោយពេលផ្ទៀងផ្ទាត់ Credential ជោគជ័យ
session_regenerate_id(true);

$_SESSION['user_id'] = $user['id'];
$_SESSION['authenticated_at'] = time();
```

ការបង្កើត ID ថ្មីជួយការពារ Session Fixation។ ត្រូវកំណត់ User ID ពី Record ដែលផ្ទៀងផ្ទាត់នៅ Server មិនមែនទទួល `user_id` ពី Form ឬ Query String ទេ។

ពេល User Sign Out ក៏អាចបង្កើត ID ថ្មី ឬបញ្ចប់ Session ទាំងស្រុងតាម Flow របស់កម្មវិធី។

## ៤. Sign Out និងលុប Session Cookie

លុបទិន្នន័យនៅ Server ហើយផ្ញើ Cookie ផុតកំណត់ដែលមាន Path/Domain ដូចនឹង Cookie ដើម៖

```php
<?php
session_start();

$_SESSION = [];

$params = session_get_cookie_params();

if (ini_get('session.use_cookies')) {
    setcookie(session_name(), '', [
        'expires' => time() - 42000,
        'path' => $params['path'],
        'domain' => $params['domain'],
        'secure' => $params['secure'],
        'httponly' => $params['httponly'],
        'samesite' => $params['samesite'] ?? 'Lax',
    ]);
}

session_destroy();
```

Sign Out គួរធ្វើតាម POST និងការពារ CSRF ព្រោះវាជា Action ផ្លាស់ប្តូរ Authentication State។ លុប Cookie ក្នុង Browser តែមួយមុខមិនចាំបាច់ធ្វើឱ្យ Session នៅ Server បាត់ទេ; ត្រូវបញ្ចប់ Session នៅ Server ផងដែរ។

## ៥. ការផុតកំណត់ និង Lifecycle

PHP Session GC គឺជាការសម្អាតតាម Configuration ហើយមិនមែនជាការធានាថា Session ផុតកំណត់ត្រឹមពេលជាក់លាក់ទេ។ បើកម្មវិធីត្រូវការកំណត់ Idle Timeout ឬ Absolute Lifetime ត្រូវអនុវត្ត និងពិនិត្យនៅ Server រាល់ Request៖

```php
<?php
$idleLimit = 30 * 60;       // 30 នាទី
$absoluteLimit = 8 * 60 * 60; // 8 ម៉ោង
$now = time();

$lastActivity = $_SESSION['last_activity'] ?? $now;
$createdAt = $_SESSION['created_at'] ?? $now;

if (
    !is_int($lastActivity)
    || !is_int($createdAt)
    || $now - $lastActivity > $idleLimit
    || $now - $createdAt > $absoluteLimit
) {
    $_SESSION = [];
    session_destroy();
    http_response_code(401);
    exit('Session បានផុតកំណត់។ សូម Sign In ម្តងទៀត។');
}

$_SESSION['created_at'] = $createdAt;
$_SESSION['last_activity'] = $now;
```

នៅក្នុងកម្មវិធីពិត បន្ទាប់ពី Expire ត្រូវលុប Session Cookie ផងដែរ ដូចឧទាហរណ៍ Sign Out។ កុំរំពឹងថា GC របស់ PHP នឹងអនុវត្ត Idle/Absolute Timeout របស់កម្មវិធីជំនួសអ្នក។

## ៦. ចំណុចសុវត្ថិភាពបន្ថែម

- ប្រើ HTTPS ទាំងមូល ហើយបើក Secure Cookie ក្នុង Production។
- កំណត់ `session.use_strict_mode=1` និងប្រើ Cookie-only Session ID។
- ប្តូរ Session ID ក្រោយ Login ឬពេលផ្លាស់ប្តូរសិទ្ធិសំខាន់។
- កុំ Log Session ID ឬបញ្ចូលវាក្នុង URL, HTML, Error Message។
- ការពារ POST ដែលផ្លាស់ប្តូរ State ដោយ CSRF Token។
- កំណត់ Session Storage ឱ្យសុវត្ថិភាព; កុំឱ្យអ្នកប្រើផ្សេងអាចអាន/សរសេរ Session Files។
- ប្រសិនបើមាន Server ច្រើន ត្រូវប្រើ Shared Session Store ដែលបានការពារ និងកំណត់ Expiration ឱ្យសមស្រប។
- ក្រោយផ្លាស់ប្តូរ Password, Sign Out ពីគ្រប់ឧបករណ៍ ឬមានសញ្ញាបាត់បង់ Session ត្រូវមានវិធី Revoke Sessions។

## ៧. កំហុសដែលជួបញឹកញាប់

- ហៅ `session_start()` ក្រោយផ្ញើ HTML ឬមាន Whitespace Output។
- ប្រើ Session ID ដដែលមុន និងក្រោយ Sign In។
- បើក `HttpOnly` ប៉ុន្តែភ្លេច HTTPS និង `Secure` នៅ Production។
- គិតថា `SameSite` អាចជំនួស CSRF Token គ្រប់ករណី។
- លុប Cookie តែ Browser ដោយមិនបញ្ចប់ Session នៅ Server។
- ប្រើ Cookie Lifetime វែងជំនួស Remember Me Flow ដែលមាន Token សុវត្ថិភាព និងអាច Revoke បាន។

## ៨. លំហាត់

1. កំណត់ Session Cookie Options សម្រាប់ Local Development និង Production HTTPS។
2. បន្ទាប់ពី Login ជោគជ័យ បង្កើត Session ID ថ្មី ហើយរក្សាទុក User ID។
3. បង្កើត Sign Out Endpoint ដោយ POST, CSRF Check, លុប Cookie និងបញ្ចប់ Session។
4. បន្ថែម Idle Timeout 30 នាទី និង Absolute Lifetime 8 ម៉ោង។
5. ពន្យល់ពីមូលហេតុដែល Session ID មិនគួរដាក់ក្នុង URL។

## ៩. ចម្លើយណែនាំខ្លី

- កំណត់ Cookie Options មុន `session_start()`។
- Production HTTPS: `Secure=true`, `HttpOnly=true`, និង `SameSite` សមស្របនឹង Flow។
- ប្រើ `session_regenerate_id(true)` ក្រោយផ្ទៀងផ្ទាត់ Login។
- Sign Out ត្រូវបញ្ចប់ Session នៅ Server និង Expire Cookie ដែលមាន Path/Domain ដូច Cookie ដើម។
- Session ID ជា Bearer Secret; អ្នកកាន់វាអាចប្រើ Session បាន ប្រសិនបើគ្មានការការពារផ្សេង។

## ឯកសារយោង

- [PHP: Securing Session INI Settings](https://www.php.net/manual/en/session.security.ini.php)
- [PHP: session_set_cookie_params()](https://www.php.net/manual/en/function.session-set-cookie-params.php)
- [PHP: session_regenerate_id()](https://www.php.net/manual/en/function.session-regenerate-id.php)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
