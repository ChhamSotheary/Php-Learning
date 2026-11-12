# មេរៀនទី ៣៥៖ ផ្ញើ Email និងផ្ទៀងផ្ទាត់អាសយដ្ឋានក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពីជំហានទូទៅក្នុងការផ្ញើ Email ពី Web App។
- ប្រើ Mailer Library តាម SMTP ដោយមិនសរសេរ Header ដោយដៃ។
- បង្កើត Verification Token ចៃដន្យ ហើយរក្សាទុកតែ Hash របស់វា។
- កំណត់ Expiration និងប្រើ Token តែមួយដង។
- ជៀសវាងការបង្ហាញ Token ឬ SMTP Credential ក្នុង Log។

> PHP `mail()` មិនធានាថា Email ត្រូវបានបញ្ជូនដល់ Inbox ទេ ហើយការតំឡើង MTA/SMTP គឺជាការទទួលខុសត្រូវរបស់ Server។ ក្នុងកម្មវិធីពិត ជាទូទៅប្រើ Mailer Library និង SMTP Provider ដែលបានកំណត់ត្រឹមត្រូវ។

## ១. Email Flow សម្រាប់ការផ្ទៀងផ្ទាត់

Flow ទូទៅ៖

1. User បង្កើតគណនី ហើយ Server Validate Email និងព័ត៌មានផ្សេង។
2. Server បង្កើត Random Token, រក្សាទុក Hash របស់ Token និង Expiry នៅ Database។
3. Server ផ្ញើ Link ដែលមាន Token ទៅ Email អ្នកប្រើ។
4. User បើក Link; Server Hash Token ដែលទទួលបាន ហើយប្រៀបធៀបនឹង Hash នៅ Database។
5. បើ Token ត្រឹមត្រូវ និងមិនទាន់ផុតកំណត់ Server កំណត់ Email ជា Verified និងប្រើ Token អស់ហើយ។

ការផ្ទៀងផ្ទាត់បង្ហាញថា អ្នកទទួលអាចចូលប្រើ Inbox នោះនៅពេលនោះ។ វាមិនធានាថា Email ជារបស់បុគ្គលជាក់លាក់ ឬថាអត្តសញ្ញាណពិតត្រឹមត្រូវទេ។

## ២. បង្កើត Token និងទុកតែ Hash

ប្រើ `random_bytes()` សម្រាប់ Token ដែលពិបាកទាយ៖

```php
<?php
$token = bin2hex(random_bytes(32));
$tokenHash = hash('sha256', $token);
$expiresAt = (new DateTimeImmutable('+30 minutes'))
    ->format('Y-m-d H:i:s');
```

រក្សាទុក `tokenHash`, `user_id`, និង `expires_at` ក្នុង Database ដោយ Prepared Statement។ កុំរក្សា Token ដើមក្នុង Database ព្រោះអ្នកណាដែលបានអាន Table អាចប្រើ Link បាន។

ឧទាហរណ៍ Schema៖

```sql
CREATE TABLE email_verification_tokens (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    token_hash CHAR(64) NOT NULL UNIQUE,
    expires_at DATETIME NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email_verification_user (user_id)
);
```

បង្កើត Link ដោយប្រើ Origin ដែលបានកំណត់នៅ Configuration របស់ Server មិនយក Host Header ដែល Client ផ្ញើមកទៅបង្កើត URL ដោយផ្ទាល់៖

```php
$baseUrl = requiredEnv('APP_BASE_URL'); // ឧ. https://example.com
$verificationUrl = rtrim($baseUrl, '/')
    . '/verify-email?token=' . urlencode($token);
```

## ៣. ផ្ញើ Email តាម SMTP Library

ដំឡើង PHPMailer តាម Composer ក្នុង Project៖

```bash
composer require phpmailer/phpmailer
```

ឧទាហរណ៍គំនិត (តម្លៃ SMTP មកពី Configuration; កុំ Commit Credential)៖

```php
<?php
use PHPMailer\PHPMailer\PHPMailer;

$mail = new PHPMailer(true);
$mail->isSMTP();
$mail->Host = requiredEnv('MAIL_HOST');
$mail->SMTPAuth = true;
$mail->Username = requiredEnv('MAIL_USERNAME');
$mail->Password = requiredEnv('MAIL_PASSWORD');
$mail->SMTPSecure = PHPMailer::ENCRYPTION_STARTTLS;
$mail->Port = 587;

$mail->setFrom(requiredEnv('MAIL_FROM_ADDRESS'), 'Learning PHP');
$mail->addAddress($email);
$mail->Subject = 'ផ្ទៀងផ្ទាត់អាសយដ្ឋាន Email';
$mail->isHTML(false);
$mail->Body = "សូមបើក Link នេះដើម្បីផ្ទៀងផ្ទាត់ Email៖\n"
    . $verificationUrl;

$mail->send();
```

ប្រើ SMTP Provider ដែលទាមទារ TLS និងកំណត់ From Address ដែលអនុញ្ញាត។ ដោះស្រាយ Error ដោយកុំបញ្ចេញ SMTP Password ឬព័ត៌មាន Server ទៅអ្នកប្រើ។ កុំបញ្ចូល Input ដែលមិនទុកចិត្តទៅ Email Header ដូចជា Subject/From ដោយមិន Validate។

## ៤. ផ្ទៀងផ្ទាត់ Token និងប្រើតែមួយដង

ទទួល Token ជា String, ពិនិត្យទម្រង់ ហើយ Hash មុនស្វែងរក៖

```php
<?php
$token = $_GET['token'] ?? null;

if (!is_string($token) || !preg_match('/^[a-f0-9]{64}$/', $token)) {
    http_response_code(400);
    exit('Link ផ្ទៀងផ្ទាត់មិនត្រឹមត្រូវ ឬបានផុតកំណត់។');
}

$tokenHash = hash('sha256', $token);

// ចាប់ផ្តើម Transaction ដើម្បី Update User និងលុប Token ជាអាតូមិក។
$pdo->beginTransaction();

$stmt = $pdo->prepare(
    'SELECT user_id
     FROM email_verification_tokens
     WHERE token_hash = :token_hash
       AND expires_at > UTC_TIMESTAMP()
     FOR UPDATE'
);
$stmt->execute(['token_hash' => $tokenHash]);
$record = $stmt->fetch(PDO::FETCH_ASSOC);

if ($record === false) {
    $pdo->rollBack();
    http_response_code(400);
    exit('Link ផ្ទៀងផ្ទាត់មិនត្រឹមត្រូវ ឬបានផុតកំណត់។');
}

$update = $pdo->prepare(
    'UPDATE users SET email_verified_at = UTC_TIMESTAMP() WHERE id = :id'
);
$update->execute(['id' => $record['user_id']]);

$delete = $pdo->prepare(
    'DELETE FROM email_verification_tokens WHERE token_hash = :token_hash'
);
$delete->execute(['token_hash' => $tokenHash]);

$pdo->commit();
```

ត្រូវរៀបចំ Exception Handling ដើម្បី Rollback Transaction បើ Query បរាជ័យ។ នៅ Database ដែលប្រើ មុខងារ `FOR UPDATE` ត្រូវមាន Transaction និង Engine ដែលគាំទ្រ Locking ដូចជា InnoDB។

## ៥. ការពារ Token ក្នុង URL និង Log

Verification Token មានសិទ្ធិបញ្ជាក់ Email ដូច្នេះ៖

- ប្រើ HTTPS; កុំផ្ញើ Token លើ HTTP។
- កុំ Log Query String ពេញលេញដែលមាន Token។
- កំណត់ Token ឱ្យមាន Expiry ខ្លី និងប្រើបានតែមួយដង។
- កុំដាក់ Third-party Analytics/Resource លើទំព័រដែលបង្ហាញ Token។
- អាចប្តូរទៅទំព័រ URL ស្អាតក្រោយទទួល Token ដោយប្រុងប្រយ័ត្ន និងមិន Log វា។
- កំណត់ចំនួន Request ផ្ញើ Verification Email ឡើងវិញ ដើម្បីបន្ថយ Spam/Abuse។
- បង្ហាញសារទូទៅ និងកុំបញ្ជាក់លើសពីចាំបាច់ថា Account មានឬអត់។

## ៦. កំហុសដែលជួបញឹកញាប់

- ប្រើ `rand()` ឬ `uniqid()` ជា Token សុវត្ថិភាព។
- រក្សា Token ដើមក្នុង Database។
- មិនកំណត់ Expiration ឬទុក Token ប្រើបានច្រើនដង។
- សង់ Verification URL ពី Host Header ដែលអាចក្លែងបាន។
- Log URL ពេញ ឬបង្ហាញ SMTP Exception/Password។
- គិតថា `mail()` ត្រឡប់ true មានន័យថា Email ដល់ Inbox។
- មិនកំណត់ Rate Limit លើការផ្ញើ Email ឡើងវិញ។

## ៧. លំហាត់

1. បង្កើត Verification Token ដោយ `random_bytes(32)` ហើយរក្សាទុក Hash និង Expiry។
2. ផ្ញើ Email តាម SMTP Library ដោយប្រើ Configuration ខាងក្រៅ Source Code។
3. Validate Token, ពិនិត្យ Expiration និងប្រើ Database Transaction។
4. លុប Token បន្ទាប់ពី Verify ជោគជ័យ។
5. រចនាការការពារ Resend Verification ដោយ Rate Limit និងសារឆ្លើយតបដែលមិនបង្ហាញព័ត៌មានគណនី។

## ៨. ចម្លើយណែនាំខ្លី

- បង្កើត Token ដោយ CSPRNG៖ `bin2hex(random_bytes(32))`។
- រក្សាទុក `hash('sha256', $token)` មិនរក្សា Token ដើម។
- ផ្ទៀងផ្ទាត់ Expiry និង Update User/Delete Token ក្នុង Transaction។
- SMTP Credential គួរមកពី Secret Store/Environment ហើយមិនត្រូវ Commit ឬ Log។
- Rate Limit ការផ្ញើ Email និងកុំបង្ហាញ Credential ឬ Token ក្នុង Error។

## ឯកសារយោង

- [PHP: random_bytes()](https://www.php.net/manual/en/function.random-bytes.php)
- [PHP: mail()](https://www.php.net/manual/en/function.mail.php)
- [PHPMailer: GitHub Repository](https://github.com/PHPMailer/PHPMailer)
- [OWASP Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
