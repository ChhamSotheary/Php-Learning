
# មេរៀនទី ២៨៖ Rate Limiting, Audit Logging និងការការពារ Brute Force

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Rate Limiting និងរបៀបឆ្លើយតបដោយ HTTP 429។
- កំណត់ Rate Limit លើ Login តាម IP និង Account Identifier។
- ប្រើ Retry-After ដើម្បីប្រាប់ Client ពេលអាចសាកល្បងម្ដងទៀត។
- កត់ត្រា Authentication Events ដោយមិនរក្សា Password ឬ Token។
- យល់ពីការប្រុងប្រយ័ត្នចំពោះ Account Lockout និង Proxy Headers។
- ជ្រើសរើសកន្លែងអនុវត្ត Rate Limiting សម្រាប់កម្មវិធីដែលមាន Server ច្រើន។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL/InnoDB, PDO និងចំណេះដឹងពី Authentication/API ក្នុងមេរៀនទី ២៧។ ឧទាហរណ៍ Rate Limiter ក្នុងមេរៀននេះគឺ Fixed Window សម្រាប់រៀន។

---

## ១. Brute Force និង Credential Stuffing

**Brute Force** គឺការសាកល្បង Password ជាច្រើនរហូតទាយត្រូវ។  
**Credential Stuffing** គឺការយក Email និង Password ដែលលេចធ្លាយពីសេវាផ្សេង មកសាកល្បងលើប្រព័ន្ធរបស់យើង។

ការការពារមិនគួរពឹងលើវិធីតែមួយទេ។ ប្រើការរួមបញ្ចូលដូចជា៖

- Password Hashing ដោយ password_hash() និង password_verify()។
- Rate Limiting តាមប្រភព និង Account Identifier។
- Error Message ទូទៅដែលមិនបញ្ជាក់ថា Account មានឬអត់។
- HTTPS សម្រាប់ Login និង API ទាំងមូល។
- Audit Log សម្រាប់ជួយរកលំនាំសាកល្បងខុសធម្មតា។
- MFA ឬការផ្ទៀងផ្ទាត់បន្ថែមសម្រាប់សកម្មភាពមានហានិភ័យខ្ពស់។

## ២. Rate Limiting ជាអ្វី?

Rate Limiting កំណត់ចំនួន Request ដែលអាចធ្វើបានក្នុងរយៈពេលមួយ។ ឧទាហរណ៍៖

- Login មិនលើស 5 ដងក្នុង 1 នាទីតាម Account Identifier។
- Login មិនលើស 20 ដងក្នុង 1 នាទីតាម IP។
- API ទូទៅមិនលើសចំនួនកំណត់តាម User ឬ Token។

បើលើស Limit API អាចឆ្លើយតប HTTP 429 Too Many Requests។ Retry-After អាចប្រាប់ Client ថាត្រូវរង់ចាំប៉ុន្មានវិនាទីមុនសាកល្បងម្ដងទៀត។

Rate Limits ត្រូវកំណត់តាមហានិភ័យ និងចំនួនអ្នកប្រើពិត។ Limit តឹងពេកអាចរារាំងអ្នកប្រើត្រឹមត្រូវ ឬអនុញ្ញាតឱ្យអ្នកវាយប្រហាររារាំង Account របស់អ្នកដទៃ។

## ៣. តារាង Fixed-Window Rate Limit

បង្កើត Table សម្រាប់រក្សា Counter ក្នុងមួយនាទី៖

```sql
CREATE TABLE api_rate_limits (
    rate_key CHAR(64) PRIMARY KEY,
    window_started_at DATETIME NOT NULL,
    request_count INT UNSIGNED NOT NULL DEFAULT 0,
    INDEX idx_rate_limit_window (window_started_at)
) ENGINE=InnoDB;
```

rate_key ជា HMAC Hash របស់ Identifier ដូចជា IP ឬ Email ដែលធ្វើឱ្យ Database មិនរក្សាទុកតម្លៃដើមនោះដោយផ្ទាល់។

បង្កើត Secret សម្រាប់ HMAC នៅ Environment Variable ដូចជា RATE_LIMIT_HMAC_KEY។ Secret នេះត្រូវរក្សាទុកក្រៅ Repository។

## ៤. Fixed-Window Rate Limiter ដោយ PDO

ឧទាហរណ៍នេះបង្កើន Counter ក្នុង Window 60 វិនាទី ហើយត្រឡប់ Retry Time បើ Limit លើស៖

```php
<?php
function enforceRateLimit(
    PDO $pdo,
    string $identifier,
    int $limit
): void {
    $secret = getenv('RATE_LIMIT_HMAC_KEY');

    if (!is_string($secret) || strlen($secret) < 32) {
        error_log('Rate limit HMAC secret is missing or too short.');
        http_response_code(500);
        exit;
    }

    $rateKey = hash_hmac('sha256', $identifier, $secret);

    $upsert = $pdo->prepare(
        'INSERT INTO api_rate_limits
            (rate_key, window_started_at, request_count)
         VALUES
            (:rate_key, UTC_TIMESTAMP(), 1)
         ON DUPLICATE KEY UPDATE
            request_count = IF(
                window_started_at < UTC_TIMESTAMP() - INTERVAL 60 SECOND,
                1,
                request_count + 1
            ),
            window_started_at = IF(
                window_started_at < UTC_TIMESTAMP() - INTERVAL 60 SECOND,
                UTC_TIMESTAMP(),
                window_started_at
            )'
    );
    $upsert->execute(['rate_key' => $rateKey]);

    $read = $pdo->prepare(
        'SELECT
            request_count,
            GREATEST(
                1,
                TIMESTAMPDIFF(
                    SECOND,
                    UTC_TIMESTAMP(),
                    DATE_ADD(window_started_at, INTERVAL 60 SECOND)
                )
            ) AS retry_after
         FROM api_rate_limits
         WHERE rate_key = :rate_key'
    );
    $read->execute(['rate_key' => $rateKey]);
    $bucket = $read->fetch();

    if ($bucket === false) {
        throw new RuntimeException('Rate limit state is unavailable.');
    }

    if ((int) $bucket['request_count'] > $limit) {
        $retryAfter = (int) $bucket['retry_after'];

        header('Retry-After: ' . $retryAfter);
        http_response_code(429);

        echo json_encode([
            'error' => [
                'code' => 'rate_limited',
                'message' => 'សំណើច្រើនពេក។ សូមរង់ចាំមុនសាកល្បងម្ដងទៀត។',
            ],
        ], JSON_UNESCAPED_UNICODE);

        exit;
    }
}
```

នេះជា Fixed Window៖ Counter ចាប់ផ្ដើមឡើងវិញរាល់ 60 វិនាទី។ វាអាចអនុញ្ញាតឱ្យមាន Request ច្រើនជិតព្រំដែន Window ពីរ។ សម្រាប់ Traffic ច្រើន ឬច្បាប់តឹងជាងនេះ គួរប្រើ Sliding Window ឬ Token Bucket ក្នុង Redis/API Gateway ដែលចែករំលែក State បានលឿន និងស្របគ្នារវាង Server ច្រើន។

## ៥. កំណត់ Login Limit តាម IP និង Account

នៅ Login Endpoint អាចអនុវត្ត Rate Limit លើ Identifier ពីរប្រភេទ៖

```php
<?php
$email = strtolower(trim($input->email ?? ''));
$ipAddress = $_SERVER['REMOTE_ADDR'] ?? 'unknown';

enforceRateLimit($pdo, 'login:ip:' . $ipAddress, 20);
enforceRateLimit($pdo, 'login:email:' . $email, 5);

// បន្តទៅ password_verify() បន្ទាប់ពីឆ្លង Rate Limits។
```

- IP Limit ជួយទប់ការសាកល្បងច្រើនពីប្រភពតែមួយ។
- Account Limit ជួយទប់ការទាយលើគណនីតែមួយពី IP ច្រើន។
- អាចដាក់ Limit ដាច់ដោយឡែកសម្រាប់ API Key, User ID ឬ Route ដែលមានហានិភ័យ។

ប្រយ័ត្នពេលប្រើ IP ពី Reverse Proxy។ កុំទុកចិត្ត X-Forwarded-For ដែល Client ផ្ញើដោយផ្ទាល់។ កំណត់បញ្ជី Proxy ដែលទុកចិត្តនៅ Web Server ហើយទទួល IP ដែលបាន Normalize ពីប្រភពនោះប៉ុណ្ណោះ។

## ៦. Account Lockout និងការបដិសេធសេវា

ការចាក់សោ Account ជាអចិន្ត្រៃយ៍ក្រោយការប៉ុនប៉ងខុសអាចឱ្យអ្នកវាយប្រហាររារាំងអ្នកប្រើពិតមិនឱ្យ Login បាន។ ការរចនាដែលប្រុងប្រយ័ត្នជាងនេះ៖

- ប្រើ Rate Limit បណ្ដោះអាសន្នតាម Account និង IP។
- ប្រើការពន្យាពេលកើនឡើងបន្តិចៗ (progressive delay) ដោយមានព្រំដែនអតិបរមា។
- ផ្ដល់ Password Reset និង Recovery Flow ដែលមានសុវត្ថិភាព។
- ប្រើ MFA ឬ Challenge បន្ថែមពេលមានសញ្ញាហានិភ័យ។
- កុំបង្ហាញថា Account មានឬអត់តាម Error Message។
- ត្រួតពិនិត្យសមតុល្យរវាងការការពារ និងការរារាំងអ្នកប្រើត្រឹមត្រូវ។

កុំអនុវត្ត Delay ធំៗក្នុង PHP Worker ដោយ sleep() សម្រាប់ Request ច្រើន ព្រោះវាអាចប្រើ Resource Server អស់។ ប្រើ Rate Limit Store, Queue ឬ Gateway ដែលសមស្រប។

## ៧. Audit Logging

Audit Log ជាកំណត់ត្រាព្រឹត្តិការណ៍សំខាន់ៗដែលជួយស៊ើបអង្កេត និងរកលំនាំមិនធម្មតា។ ឧទាហរណ៍៖

- Login ជោគជ័យ និងបរាជ័យ។
- Token ត្រូវបានបង្កើត ឬ Revoked។
- Authorization បដិសេធសកម្មភាព។
- Rate Limit ត្រូវបានលើស។
- Password Reset ឬការផ្លាស់ប្ដូរ Role។

បង្កើត Table សាមញ្ញ៖

```sql
CREATE TABLE auth_audit_events (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    request_id CHAR(32) NOT NULL,
    user_id BIGINT UNSIGNED NULL,
    event_type VARCHAR(50) NOT NULL,
    subject_hash CHAR(64) NULL,
    ip_hash CHAR(64) NULL,
    succeeded BOOLEAN NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_audit_user_time (user_id, created_at),
    INDEX idx_audit_event_time (event_type, created_at)
) ENGINE=InnoDB;
```

កុំដាក់ Password, Raw Bearer Token, Session ID, API Secret ឬ Request Body ទាំងមូលក្នុង Audit Log។ កំណត់សិទ្ធិចូលមើល Log និងរយៈពេលរក្សាទុកឱ្យសមស្រប។

## ៨. កត់ត្រា Authentication Event

ឧទាហរណ៍ Function កត់ត្រាព័ត៌មានដែលបានកំណត់ជាមុន៖

```php
<?php
function logAuthEvent(
    PDO $pdo,
    string $requestId,
    ?int $userId,
    string $eventType,
    ?string $subject,
    ?string $ipAddress,
    bool $succeeded
): void {
    $allowedEvents = [
        'login_success',
        'login_failure',
        'rate_limited',
        'token_revoked',
        'authorization_denied',
    ];

    if (!in_array($eventType, $allowedEvents, true)) {
        throw new InvalidArgumentException('Audit Event មិនត្រូវបានអនុញ្ញាត។');
    }

    $secret = getenv('AUDIT_HASH_KEY');

    if (!is_string($secret) || strlen($secret) < 32) {
        error_log('Audit HMAC secret is missing or too short.');
        return;
    }

    $subjectHash = $subject === null
        ? null
        : hash_hmac('sha256', strtolower($subject), $secret);

    $ipHash = $ipAddress === null
        ? null
        : hash_hmac('sha256', $ipAddress, $secret);

    try {
        $stmt = $pdo->prepare(
            'INSERT INTO auth_audit_events
                (request_id, user_id, event_type, subject_hash, ip_hash, succeeded)
             VALUES
                (:request_id, :user_id, :event_type, :subject_hash, :ip_hash, :succeeded)'
        );

        $stmt->execute([
            'request_id' => $requestId,
            'user_id' => $userId,
            'event_type' => $eventType,
            'subject_hash' => $subjectHash,
            'ip_hash' => $ipHash,
            'succeeded' => $succeeded ? 1 : 0,
        ]);
    } catch (PDOException $e) {
        // Audit Logging បរាជ័យក៏មិនត្រូវបង្ហាញព័ត៌មានខាងក្នុងទៅ Client។
        error_log('Could not persist auth audit event: ' . $e->getMessage());
    }
}
```

HMAC Hash អាចជួយកុំឱ្យ Email ឬ IP បង្ហាញផ្ទាល់ក្នុង Table ប៉ុន្តែមិនធ្វើឱ្យវាក្លាយជាអនាមិកពេញលេញទេ។ ត្រូវគ្រប់គ្រង Key និង Log ដូចទិន្នន័យដែលអាចមានព័ត៌មានផ្ទាល់ខ្លួន។

បង្កើត Request ID សម្រាប់តាមដានសំណើមួយ៖

```php
$requestId = bin2hex(random_bytes(16));
header('X-Request-Id: ' . $requestId);
```

បើមាន Request ID ពី Reverse Proxy ដែលទុកចិត្ត អាចប្រើវាបន្ទាប់ពី Validate ទម្រង់ និងប្រភព។ កុំទុកចិត្ត Header ដែល Client អាចផ្ញើដោយគ្មានការត្រួតពិនិត្យ។

## ៩. របៀបភ្ជាប់ Rate Limit និង Audit Log ទៅ Login

លំដាប់ការងារដែលអាចប្រើក្នុង Login Endpoint៖

1. បង្កើត Request ID។
2. Validate Content-Type និង JSON Body។
3. Normalize Email និងទាញ Client IP ពីប្រភពដែលទុកចិត្ត។
4. អនុវត្ត Rate Limits តាម IP និង Account Identifier។
5. រក User និង Verify Password។
6. Log Login Failure ឬ Login Success ដោយមិនកត់ត្រា Password។
7. បង្កើត Token ឬ Session បើ Login ជោគជ័យ។
8. ត្រឡប់ Error ទូទៅ ឬ Credential ជោគជ័យ។

ឧទាហរណ៍គំនិត៖

```php
<?php
$requestId = bin2hex(random_bytes(16));
header('X-Request-Id: ' . $requestId);

$input = readJsonObject();
$email = is_string($input->email ?? null)
    ? strtolower(trim($input->email))
    : '';
$ipAddress = trustedClientIp();

enforceRateLimit($pdo, 'login:ip:' . $ipAddress, 20);
enforceRateLimit($pdo, 'login:email:' . $email, 5);

// ផ្ទៀងផ្ទាត់ Password ដោយ password_verify()។

if (!$loginSucceeded) {
    logAuthEvent(
        $pdo,
        $requestId,
        null,
        'login_failure',
        $email,
        $ipAddress,
        false
    );

    jsonError(401, 'invalid_credentials', 'Email ឬ Password មិនត្រឹមត្រូវ។');
}

// ពេលជោគជ័យ កត់ត្រា Event និងបង្កើត Session/Token។
```

Snippet នេះបង្ហាញលំដាប់រួមបញ្ចូលប៉ុណ្ណោះ។ trustedClientIp() ត្រូវអនុវត្តតាម Proxy Configuration របស់ Deployment ពិត មិនគួរយក X-Forwarded-For ដោយផ្ទាល់។

## ១០. Monitoring និងការសម្អាតទិន្នន័យ

- ត្រួតពិនិត្យ Login Failure ច្រើនលើ Account តែមួយ។
- ត្រួតពិនិត្យការប៉ុនប៉ងច្រើនពី IP ឬក្រុមប្រភពផ្សេងៗ។
- ត្រួតពិនិត្យ 429 Responses និង Rate Limit Bypass Patterns។
- កំណត់ Alert សម្រាប់លំនាំខុសធម្មតា តែពិនិត្យ Context មុនសន្និដ្ឋានថាជាការវាយប្រហារ។
- កំណត់រយៈពេលរក្សា Audit Logs និង Rate Limit Buckets។
- លុប Expired Buckets ជាប្រចាំ ឬកំណត់ការផុតកំណត់ដោយ Storage Layer។
- ដាក់ Logs ក្រៅ Public Web Directory និងកំណត់សិទ្ធិអាន/សរសេរ។

## ១១. Best Practices

1. ប្រើ Rate Limit ច្រើន Dimension៖ IP, Account, User/Token និង Endpoint។
2. កុំប្រើ Account Lockout អចិន្ត្រៃយ៍ដែលអ្នកវាយប្រហារអាចបង្ក DoS។
3. ឆ្លើយតប 429 និង Retry-After ពេល Rate Limit លើស។
4. កត់ត្រា Login Success/Failure, Rate Limit និង Authorization Denied។
5. កុំកត់ត្រា Password, Token Raw, Session ID ឬ Secret។
6. ប្រើ HMAC Secret ដែលមាន Entropy គ្រប់គ្រាន់ និងរក្សាក្រៅ Source Code។
7. ប្រើ Shared Store ដូចជា Redis ឬ API Gateway សម្រាប់ Server ច្រើន។
8. កំណត់ Proxy Trust ឱ្យច្បាស់ មុនយក Client IP មកប្រើ។
9. កំណត់ Log Access, Retention និង Alerting។
10. ពិនិត្យ False Positives ដើម្បីកុំឱ្យការការពាររារាំងអ្នកប្រើត្រឹមត្រូវ។

## ១២. កំហុសដែលជួបញឹកញាប់

- Rate Limit តាម IP តែមួយ ហើយមិនគិតពី Account Identifier។
- ទុកចិត្ត X-Forwarded-For ពី Client។
- ប្រើ Memory Counter ក្នុង PHP Worker ដែលមិនចែករំលែករវាង Server/Process។
- ចាក់សោ Account ជារៀងរហូតក្រោយការប៉ុនប៉ងខុស។
- កត់ត្រា Password ឬ Bearer Token ក្នុង Logs។
- រក្សាទុក Email/IP ផ្ទាល់ដោយមិនគិតពីការចូលប្រើ និង Retention។
- អនុញ្ញាតឱ្យ Audit Log បរាជ័យដោយស្ងៀមស្ងាត់ ដោយគ្មាន Monitoring។
- បង្ហាញថា Email មិនមាន ឬ Password ខុសខុសគ្នា។
- កំណត់ Limit ដោយគ្មានការវាស់ Traffic និងផលប៉ះពាល់អ្នកប្រើពិត។
- ដាក់ Rate Limiting តែ Application Server ខណៈ Endpoint អាចត្រូវបានការពារនៅ Gateway ផងដែរ។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Login Rate Limit

កំណត់ Limit ដាច់ដោយឡែកតាម IP និង Email ហើយត្រឡប់ 429 ពេលលើស។

### លំហាត់ទី ២៖ Retry-After

បន្ថែម Retry-After Header ដែលប្រាប់ចំនួនវិនាទីនៅសល់ក្នុង Window។

### លំហាត់ទី ៣៖ Audit Events

កត់ត្រា Login Success, Login Failure, Token Revoked និង Authorization Denied ដោយមិនរក្សា Secret។

### លំហាត់ទី ៤៖ Proxy Trust

សរសេរពន្យល់ថា Web Server ត្រូវកំណត់ Proxy ដែលទុកចិត្តយ៉ាងដូចម្តេច មុនប្រើ X-Forwarded-For។

### លំហាត់ទី ៥៖ Retention

កំណត់គោលការណ៍សម្អាត Rate Limit Buckets និង Audit Events ចាស់ៗ។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Rate Limiting, Account Lockout និង Audit Logging។

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- Rate Limiting កំណត់ចំនួន Request ក្នុងរយៈពេលកំណត់ ហើយអាចត្រឡប់ 429។
- Account Lockout រារាំង Login របស់ Account; ត្រូវប្រុងប្រយ័ត្នព្រោះវាអាចត្រូវប្រើសម្រាប់ Denial of Service។
- Audit Logging កត់ត្រាព្រឹត្តិការណ៍ ដើម្បីត្រួតពិនិត្យ និងស៊ើបអង្កេត; វាមិនរារាំង Request ដោយខ្លួនឯងទេ។
- ការការពារល្អប្រើ Rate Limits ច្រើន Dimension, Error Message ទូទៅ, Password Hashing និង Monitoring។
- កុំរក្សា Password ឬ Token Raw ក្នុង Log។

---

## សេចក្ដីសង្ខេប

- Rate Limiting កាត់បន្ថយការប៉ុនប៉ងស្ទួន និងគួរឆ្លើយតប 429/Retry-After។
- ប្រើ Limit តាម IP និង Account Identifier ព្រមទាំងពិនិត្យផលប៉ះពាល់អ្នកប្រើ។
- កុំជឿ Forwarded IP Header លុះត្រាតែមកពី Proxy ដែលកំណត់ជាទីទុកចិត្ត។
- Audit Log កត់ត្រា Authentication និង Authorization Events ប៉ុន្តែមិនត្រូវមាន Password ឬ Token Raw។
- សម្រាប់ Server ច្រើន ប្រើ Shared Rate Limit Store ឬ API Gateway។
- កំណត់ Retention, Access Control និង Monitoring សម្រាប់ Log/Bucket Data។

## ឯកសារយោង

- [PHP password_verify()](https://www.php.net/manual/en/function.password-verify.php)
- [RFC 6585: Additional HTTP Status Codes (429)](https://www.rfc-editor.org/rfc/rfc6585.html)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្ស File Upload Security ឬ Security Review របស់ PHP Web Application។
