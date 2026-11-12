# មេរៀនទី ៣៦៖ Password Reset ដែលមានសុវត្ថិភាពក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- រចនា Password Reset Flow ដែលមិនបង្ហាញថា Email មានគណនីឬអត់។
- បង្កើត Reset Token ចៃដន្យ និងរក្សាទុកតែ Hash។
- កំណត់ Token ឱ្យផុតកំណត់ និងប្រើបានតែមួយដង។
- ប្តូរ Password ដោយ `password_hash()` និង Transaction។
- បិទ ឬពិនិត្យ Session ចាស់ក្រោយ Password ត្រូវបាន Reset។

> Password Reset គឺជាផ្លូវចូលគណនីមួយ។ ត្រូវការពារវាដូច Login ដែរ៖ Token មិនអាចទាយបាន, Expiry, Rate Limit, HTTPS និងការការពារកុំឱ្យ Token លេចធ្លាយ។

## ១. Request Reset ដោយសារឆ្លើយតបទូទៅ

អ្នកប្រើបញ្ចូល Email របស់ខ្លួន។ មិនថាគណនីមានឬអត់ ត្រូវឆ្លើយតបដោយសារដូចគ្នា ដូចជា៖

> ប្រសិនបើអាសយដ្ឋាន Email នេះមានគណនី យើងនឹងផ្ញើការណែនាំ Reset ទៅ។

វាជួយកាត់បន្ថយការទាយថា Email ណាមានគណនី (Account Enumeration)។ ដាក់ Rate Limit តាម IP និង Account Identifier ហើយពិនិត្យ User-Agent/សញ្ញាប្រើប្រាស់តាមគោលការណ៍ឯកជនភាព។ កុំឱ្យអ្នកវាយប្រហារប្រើ Reset ដើម្បី Lock Account ឬផ្ញើ Email រំខាន។

## ២. បង្កើត Token និងរក្សាទុក Hash

បន្ទាប់ពី Validate Email៖

```php
<?php
$token = bin2hex(random_bytes(32));
$tokenHash = hash('sha256', $token);
$expiresAt = (new DateTimeImmutable('+30 minutes'))
    ->format('Y-m-d H:i:s');
```

បើ User មានគណនី បញ្ចូល `tokenHash`, `user_id`, និង `expires_at` ទៅ Table។ ទោះមិនមានគណនីក៏បង្ហាញសារដដែល។ រក្សាទុកតែ Hash ដើម្បីកុំឱ្យ Database Dump បង្ហាញ Token ដែលអាចប្រើភ្លាមៗ។

```sql
CREATE TABLE password_reset_tokens (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    token_hash CHAR(64) NOT NULL UNIQUE,
    expires_at DATETIME NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_reset_user (user_id)
);
```

អាចលុប/បិទ Token ចាស់របស់ User ពេលបង្កើតថ្មី ដើម្បីកុំឱ្យមាន Link ច្រើននៅរស់។ បង្កើត URL ពី Base URL ដែលបានកំណត់នៅ Server Configuration មិនប្រើ Host Header ដោយផ្ទាល់៖

```php
$resetUrl = rtrim(requiredEnv('APP_BASE_URL'), '/')
    . '/reset-password?token=' . urlencode($token);
```

ផ្ញើ Link តាម Email ដោយ HTTPS។ កុំដាក់ Token ក្នុង Log ឬបញ្ជូនទៅ Analytics/Third-party resource។

## ៣. ទទួល Token និងបង្ហាញ Form

នៅ GET Request៖

1. ទទួល Token ជា String និងពិនិត្យទម្រង់ត្រឹមត្រូវ។
2. Hash Token ហើយស្វែងរក Record ដែលមិនទាន់ Expire។
3. បើត្រឹមត្រូវ បង្ហាញ Password Reset Form និង CSRF Token។
4. កុំប្តូរ Password ដោយស្វ័យប្រវត្តិគ្រាន់តែបើក GET Link; Link Scanner របស់ Email អាចបើក URL មុន User។
5. កុំបង្ហាញព័ត៌មានថា Account ជាក់លាក់ណាត្រូវ Reset។

Form POST ត្រូវមាន CSRF Protection ដូចមេរៀនទី ២៩។ Reset Token និង CSRF Token មានតួនាទីខុសគ្នា៖ Reset Token បញ្ជាក់សិទ្ធិប្តូរ Password តាម Email; CSRF Token ការពារ Form ពី Request ក្លែងក្លាយ។

## ៤. ប្តូរ Password ដោយ Transaction

នៅ POST ត្រូវពិនិត្យ CSRF Token, Password Policy និង Token ម្តងទៀត។ បន្ទាប់មកធ្វើ Update និងប្រើ Token ក្នុង Transaction៖

```php
<?php
$token = $_POST['token'] ?? null;
$newPassword = $_POST['new_password'] ?? null;

if (
    !is_string($token)
    || !preg_match('/^[a-f0-9]{64}$/', $token)
    || !is_string($newPassword)
) {
    http_response_code(400);
    exit('សំណើមិនត្រឹមត្រូវ ឬ Link បានផុតកំណត់។');
}

$passwordHash = password_hash($newPassword, PASSWORD_DEFAULT);
$tokenHash = hash('sha256', $token);

$pdo->beginTransaction();

try {
    $find = $pdo->prepare(
        'SELECT user_id
         FROM password_reset_tokens
         WHERE token_hash = :token_hash
           AND expires_at > UTC_TIMESTAMP()
         FOR UPDATE'
    );
    $find->execute(['token_hash' => $tokenHash]);
    $record = $find->fetch(PDO::FETCH_ASSOC);

    if ($record === false) {
        $pdo->rollBack();
        http_response_code(400);
        exit('សំណើមិនត្រឹមត្រូវ ឬ Link បានផុតកំណត់។');
    }

    $update = $pdo->prepare(
        'UPDATE users
         SET password_hash = :password_hash
         WHERE id = :user_id'
    );
    $update->execute([
        'password_hash' => $passwordHash,
        'user_id' => $record['user_id'],
    ]);

    $delete = $pdo->prepare(
        'DELETE FROM password_reset_tokens WHERE user_id = :user_id'
    );
    $delete->execute(['user_id' => $record['user_id']]);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
```

កូដនេះជាគ្រោងសម្រាប់បង្រៀន; ត្រូវបន្ថែម CSRF Verification, Password Policy, ការចាប់ Error នៅ Boundary និងការធ្វើឱ្យ Session ចាស់អស់សុពលភាពតាមរចនាសម្ព័ន្ធកម្មវិធី។ ត្រូវប្រើ Password Hash Column ដែលមានទំហំគ្រប់គ្រាន់ (ឧ. VARCHAR(255))។

## ៥. ក្រោយប្តូរ Password

- ផ្ញើ Notification Email ថា Password ត្រូវបានប្តូរ ប៉ុន្តែមិនបញ្ចូល Password ឬ Reset Token។
- កុំ Sign In ដោយស្វ័យប្រវត្តិ តាម Link Reset; ឱ្យ User Login ជាថ្មី។
- Revoke Session/Refresh Token ចាស់ ឬបង្កើន Credential Version ដែល Session ត្រូវផ្ទៀងផ្ទាត់។
- លុប Reset Token ទាំងអស់របស់ User នោះ រួមទាំង Token មិនទាន់ប្រើ។
- កត់ Audit Event ដោយមិនដាក់ Password/Token ក្នុង Log។

## ៦. កំហុសដែលជួបញឹកញាប់

- បញ្ជាក់ក្នុង Response ថា Email មានគណនីឬអត់។
- ប្រើ Token ងាយទាយ ឬរក្សាទុក Token ដើមក្នុង Database។
- មិនកំណត់ Expiry ឬអនុញ្ញាត Token ប្រើច្រើនដង។
- Reset Password ពេល GET Request បើក Link។
- មិនមាន Rate Limit សម្រាប់ Request Reset។
- ផ្ញើ Reset URL ដែលបង្កើតពី Host Header មិនទុកចិត្ត។
- មិន Revoke Session ចាស់ក្រោយប្តូរ Password។
- Log Token, Password ឬ URL ពេញដែលមាន Token។

## ៧. លំហាត់

1. សរសេរ Request Reset Endpoint ដែលឆ្លើយតបដោយសារដូចគ្នាសម្រាប់គណនីមាន/គ្មាន។
2. បង្កើត Token ដោយ `random_bytes()`; រក្សាទុកតែ SHA-256 Hash។
3. កំណត់ Expiration, Rate Limit និង URL ពី Base URL ដែលទុកចិត្ត។
4. ប្តូរ Password ដោយ `password_hash()` និង PDO Transaction។
5. រៀបចំការលុប Token និងការបិទ Session ចាស់ក្រោយ Reset។

## ៨. ចម្លើយណែនាំខ្លី

- បង្កើត Reset Token ដោយ `bin2hex(random_bytes(32))` និងរក្សា `hash('sha256', $token)`។
- ឆ្លើយតបដោយសារទូទៅដូចគ្នា មិនថាគណនីមានឬអត់។
- ពិនិត្យ Token និង Expiry នៅ POST ហើយប្រើបានតែមួយដង។
- រក្សា Password ជា Hash ដោយ `password_hash()`; កុំ Encrypt ឬ Log Password។
- ក្រោយ Reset លុប Token ទាំងអស់ និង Revoke Session ចាស់។

## ឯកសារយោង

- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
- [PHP: random_bytes()](https://www.php.net/manual/en/function.random-bytes.php)
- [PHP: password_hash()](https://www.php.net/manual/en/function.password-hash.php)
- [មេរៀនទី ៣៥៖ Email Verification ក្នុង PHP](lesson-35-php-email-verification.md)
