# មេរៀនទី ២៩៖ ការការពារ CSRF, XSS និង SQL Injection ក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី CSRF, XSS និង SQL Injection។
- បង្កើត និងផ្ទៀងផ្ទាត់ CSRF Token សម្រាប់ Form។
- បង្ហាញទិន្នន័យដែលមិនទុកចិត្តដោយ Escape តាម HTML Context។
- ប្រើ PDO Prepared Statements ដើម្បីបំបែក SQL និងទិន្នន័យ។
- ដាក់ការការពារទាំងនេះនៅកន្លែងត្រឹមត្រូវក្នុង Request Flow។

> តម្រូវការ៖ PHP 8+, Session និង PDO។ ឧទាហរណ៍ខាងក្រោមជាគំរូសម្រាប់រៀន។ ក្នុងកម្មវិធីពិត គួរប្រើ Framework ដែលមានការការពារដែលបានសាកល្បង និងពិនិត្យ Configuration របស់វា។

---

## ១. CSRF ជាអ្វី?

Cross-Site Request Forgery (CSRF) គឺការបោក Browser របស់អ្នកប្រើដែលបាន Sign In ឱ្យផ្ញើ Request ដែលពួកគេមិនបានចង់ធ្វើ។ Browser អាចភ្ជាប់ Cookie Session ដោយស្វ័យប្រវត្តិ ដូច្នេះ Server ត្រូវបញ្ជាក់ថា Request ផ្លាស់ប្តូរទិន្នន័យមកពី Form/កម្មវិធីរបស់យើង។

សម្រាប់ Form ដែលផ្លាស់ប្តូរ State ដូចជា Update Profile ឬ Delete Record៖

- ប្រើ CSRF Token ដែលបង្កើតដោយ Server និងផ្ទៀងផ្ទាត់ពេលទទួល POST។
- កំណត់ Cookie Session ជា Secure, HttpOnly និង SameSite តាមបរិបទកម្មវិធី។
- កុំប្រើ GET ដើម្បីផ្លាស់ប្តូរទិន្នន័យ។
- ពិនិត្យ Origin/Referer អាចជាការការពារបន្ថែម ប៉ុន្តែមិនគួរជំនួស Token ក្នុងគ្រប់ករណី។

SameSite មិនជំនួស CSRF Token ទេ នៅពេលត្រូវគាំទ្រ Browser ចាស់ ឬ Flow ពិសេស។

## ២. បង្កើត CSRF Token ក្នុង Session

បង្កើត Token ពេលបង្ហាញ Form៖

```php
<?php
session_start();

if (!isset($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$csrfToken = $_SESSION['csrf_token'];
```

ដាក់ Token ជា Hidden Input៖

```php
<form method="post" action="/profile/update">
    <input
        type="hidden"
        name="csrf_token"
        value="<?= htmlspecialchars(
            $csrfToken,
            ENT_QUOTES | ENT_SUBSTITUTE,
            'UTF-8'
        ) ?>"
    >
    <label>
        ឈ្មោះថ្មី
        <input name="display_name" required>
    </label>
    <button type="submit">រក្សាទុក</button>
</form>
```

ត្រូវ Escape តម្លៃ Token ដែរ ព្រោះវាត្រូវបានដាក់ក្នុង HTML Attribute។

## ៣. ផ្ទៀងផ្ទាត់ CSRF Token

ពិនិត្យ Token នៅ Server មុនធ្វើ Update/Delete៖

```php
<?php
session_start();

$submittedToken = $_POST['csrf_token'] ?? null;
$sessionToken = $_SESSION['csrf_token'] ?? null;

if (
    !is_string($submittedToken)
    || !is_string($sessionToken)
    || !hash_equals($sessionToken, $submittedToken)
) {
    http_response_code(403);
    exit('សំណើមិនត្រឹមត្រូវ។ សូម Reload Form ហើយសាកល្បងម្ដងទៀត។');
}

// បន្តផ្ទៀងផ្ទាត់ Input និងអនុវត្តការផ្លាស់ប្តូរទិន្នន័យ។
```

`hash_equals()` ប្រើការប្រៀបធៀបដែលមិនឈប់ឆាប់តាមទីតាំងតួអក្សរខុសគ្នា។ បន្ទាប់ពីអ្នកប្រើ Sign In ឬបង្កើត Session ថ្មី គួរបង្កើត Token ថ្មីតាម Lifecycle របស់ Session។

## ៤. XSS និង HTML Output Escaping

Cross-Site Scripting (XSS) កើតឡើងពេលទិន្នន័យដែលមិនទុកចិត្តត្រូវបាន Browser ចាត់ទុកជាកូដ HTML ឬ JavaScript។ ការពារដោយ Escape នៅពេលបង្ហាញ និងតាម Context ដែលតម្លៃនោះត្រូវដាក់។

សម្រាប់អត្ថបទ HTML ធម្មតា ឬ Attribute ដែលបាន Quote៖

```php
<?php
function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

echo '<p>' . e($comment) . '</p>';
echo '<input value="' . e($displayName) . '">';
```

ចំណុចសំខាន់ៗ៖

- Escape ពេលបង្ហាញ (Output), មិនមែនពេលរក្សាទុកក្នុង Database។
- `htmlspecialchars()` សម្រាប់ HTML Text/Attribute ដែលបាន Quote ប៉ុណ្ណោះ។ វាមិនមែនជាវិធីសុវត្ថិភាពទូទៅសម្រាប់ JavaScript, CSS ឬ URL ទេ។
- បើត្រូវបង្ហាញ HTML ដែលអ្នកប្រើបញ្ចូល ត្រូវប្រើ HTML Sanitizer ដែលបានសាកល្បង និង Allowlist Tags/Attributes; កុំទុកចិត្តលើ Escape តែមួយ។

## ៥. SQL Injection និង PDO Prepared Statements

SQL Injection កើតឡើងពេល Input ត្រូវបានភ្ជាប់ទៅក្នុង SQL String ហើយ Database បកស្រាយ Input នោះជាផ្នែកនៃ Command។

កុំបង្កើត Query ដោយបញ្ចូល Input ផ្ទាល់៖

```php
// មិនមានសុវត្ថិភាព៖ កុំប្រើរបៀបនេះ
$sql = "SELECT id FROM users WHERE email = '$email'";
```

ប្រើ Prepared Statement និង Parameter៖

```php
<?php
$stmt = $pdo->prepare(
    'SELECT id, email, password_hash
     FROM users
     WHERE email = :email
     LIMIT 1'
);
$stmt->execute(['email' => $email]);

$user = $stmt->fetch(PDO::FETCH_ASSOC);
```

Parameter អាចតំណាងឱ្យតម្លៃ (Value) ប៉ុន្តែមិនអាចតំណាងឱ្យឈ្មោះ Column ឬ SQL Keyword ទេ។ បើត្រូវជ្រើស Sort Column ត្រូវប្រើ Allowlist៖

```php
$allowedSorts = ['name', 'created_at'];
$sort = $_GET['sort'] ?? 'name';

if (!is_string($sort) || !in_array($sort, $allowedSorts, true)) {
    $sort = 'name';
}

$sql = 'SELECT id, name FROM products ORDER BY ' . $sort;
```

ក្នុងឧទាហរណ៍នេះ `$sort` មកពី Allowlist របស់ Server មិនមែនជាឈ្មោះដែលយកពី Input ដោយផ្ទាល់ទេ។

## ៦. ដាក់ការការពារតាម Request Flow

សម្រាប់ Form Update Profile៖

1. Browser ស្នើ GET Form; Server បង្កើត Session CSRF Token។
2. Server Escape Token ហើយបញ្ចូលក្នុង Hidden Input។
3. Browser ផ្ញើ POST ជាមួយ Session Cookie និង Token។
4. Server ពិនិត្យថា User បាន Sign In និងមានសិទ្ធិ។
5. Server ផ្ទៀងផ្ទាត់ CSRF Token ដោយ `hash_equals()`។
6. Server Validate Input តាមច្បាប់កម្មវិធី។
7. Server រក្សាទុកដោយ PDO Prepared Statement។
8. នៅពេលបង្ហាញតម្លៃម្ដងទៀត Server Escape វាតាម HTML Context។

CSRF Token បញ្ជាក់ប្រភព Form; វាមិនជំនួស Authentication, Authorization ឬ Input Validation ទេ។

## ៧. កំហុសដែលជួបញឹកញាប់

- មាន CSRF Token តែ endpoint មិនបានផ្ទៀងផ្ទាត់វា។
- គិតថា Input Validation អាចជំនួស Output Escaping សម្រាប់ XSS។
- Escape ទិន្នន័យមុន Save រួច Escape ម្តងទៀតពេលបង្ហាញ ធ្វើឱ្យអក្សរខូច។
- ប្រើ Prepared Statement ប៉ុន្តែបញ្ចូល Column ឬ ORDER BY ពី Input ដោយផ្ទាល់។
- ប្រើ GET ដើម្បី Delete ឬ Update ទិន្នន័យ។
- បង្ហាញ Error ពី Database ដល់អ្នកប្រើ ជំនួសឱ្យសារទូទៅ និង Log នៅ Server។

## ៨. លំហាត់អនុវត្ត

1. បង្កើត Form Update Profile ដែលមាន CSRF Token និងផ្ទៀងផ្ទាត់ Token នៅ POST។
2. បង្ហាញឈ្មោះអ្នកប្រើនៅក្នុង HTML ដោយប្រើ Function `e()`។
3. សរសេរ Query ស្វែងរកផលិតផលតាម Keyword ដោយ PDO Prepared Statement។
4. បង្កើត Sort Allowlist ដែលអនុញ្ញាតតែ `name` និង `created_at`។
5. សរសេរដំណាក់កាល Request Flow ពី GET Form ដល់ការបង្ហាញលទ្ធផល។

## ៩. ចម្លើយណែនាំខ្លី

1. Token គួរត្រូវបានរក្សាទុកក្នុង Session, ដាក់ជា Hidden Input និងផ្ទៀងផ្ទាត់នៅ Server ជាមួយ `hash_equals()`។
2. ប្រើ `htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')` សម្រាប់ HTML Text/Attribute ដែលបាន Quote។
3. ឧទាហរណ៍៖

```php
$stmt = $pdo->prepare(
    'SELECT id, name FROM products WHERE name LIKE :keyword'
);
$stmt->execute(['keyword' => '%' . $keyword . '%']);
$products = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

4. កំណត់ Array នៃឈ្មោះអនុញ្ញាត ហើយបដិសេធ/ប្ដូរទៅ Default បើ Input មិនមានក្នុង Array។
5. Authenticate/authorize → verify CSRF → validate → prepared query → escape output។

## ឯកសារយោង

- [PHP: htmlspecialchars()](https://www.php.net/manual/en/function.htmlspecialchars.php)
- [PHP: hash_equals()](https://www.php.net/manual/en/function.hash-equals.php)
- [PHP: PDO::prepare()](https://www.php.net/manual/en/pdo.prepare.php)
- [OWASP Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Cross-Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
