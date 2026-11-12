
# មេរៀនទី ២៧៖ Authentication និង Authorization សម្រាប់ API

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Authentication និង Authorization។
- រក្សាទុក Password ដោយ password_hash() និងពិនិត្យដោយ password_verify()។
- បង្កើត Opaque Bearer Token ដោយ random_bytes()។
- រក្សាទុក Token Hash នៅ Database ជំនួស Raw Token។
- អាន Bearer Token ពី Authorization Header។
- ពិនិត្យសិទ្ធិអ្នកប្រើលើ Resource មួយៗ។
- លុបសិទ្ធិ Token ដោយ Revocation និងប្រើ Expiration។
- យល់ពីការប្រើ HTTPS និងការការពារ Secret ក្នុង API។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL/InnoDB, PDO MySQL និងចំណេះដឹងពី JSON API និង Sessions។ ឧទាហរណ៍នេះសម្រាប់រៀន ហើយមិនមែនជា Authentication Library ពេញលេញទេ។

---

## ១. Authentication និង Authorization ខុសគ្នាដូចម្តេច?

**Authentication** ឆ្លើយសំណួរ៖ «តើអ្នកជានរណា?» ឧទាហរណ៍ ផ្ទៀងផ្ទាត់ Email និង Password ហើយបង្កើត Session ឬ Token។

**Authorization** ឆ្លើយសំណួរ៖ «តើអ្នកមានសិទ្ធិធ្វើអ្វី?» ឧទាហរណ៍ ពិនិត្យថាអ្នកប្រើអាចកែសិស្សដែលជារបស់ខ្លួន ឬមាន Role Admin ដែរឬទេ។

ការចូលប្រព័ន្ធបានមិនមានន័យថាអាចអាន ឬកែ Resource ទាំងអស់បានទេ។ API ត្រូវពិនិត្យទាំងពីរជំហាន។

## ២. Password Hashing

កុំរក្សាទុក Password ជាអត្ថបទធម្មតា ហើយកុំប្រើ MD5 ឬ SHA-256 ដោយផ្ទាល់សម្រាប់ Password។ ប្រើ Password Hashing API របស់ PHP៖

```php
<?php
$passwordHash = password_hash($password, PASSWORD_DEFAULT);
```

PASSWORD_DEFAULT ជ្រើស Algorithm សមស្របតាម PHP Version ហើយ Hash នោះរួមមានព័ត៌មានដែលត្រូវការសម្រាប់ Verify។ រក្សាទុកលទ្ធផលទាំងមូលក្នុង Database Column ប្រវែងគ្រប់គ្រាន់ ដូចជា VARCHAR(255)។

ពេល Login ប្រើ password_verify()៖

```php
<?php
if (password_verify($submittedPassword, $user['password_hash'])) {
    // Password ត្រឹមត្រូវ។
}
```

ក្រោយ Verify ជោគជ័យ អាចពិនិត្យថា Hash គួរធ្វើបច្ចុប្បន្នភាពឬអត់៖

```php
<?php
if (password_needs_rehash($user['password_hash'], PASSWORD_DEFAULT)) {
    $newHash = password_hash($submittedPassword, PASSWORD_DEFAULT);

    // UPDATE users SET password_hash = :password_hash WHERE id = :id
}
```

កុំបង្កើត Salt ដោយដៃ។ password_hash() បង្កើត Salt ដែលសមស្របដោយស្វ័យប្រវត្តិ។

## ៣. បង្កើត Tables សម្រាប់ Users និង API Tokens

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(30) NOT NULL DEFAULT 'user',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;

CREATE TABLE api_tokens (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    token_hash CHAR(64) NOT NULL UNIQUE,
    expires_at DATETIME NOT NULL,
    revoked_at DATETIME NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_api_tokens_user
        FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_api_tokens_user (user_id)
) ENGINE=InnoDB;
```

api_tokens រក្សាទុក Hash របស់ Token មិនមែន Token ពិតទេ។ មានតែអ្នកប្រើ Client ប៉ុណ្ណោះដែលទទួល Raw Token នៅពេលបង្កើត។

## ៤. Login និងបង្កើត Bearer Token

លំហូរទូទៅ៖

1. Client ផ្ញើ Email និង Password តាម HTTPS។
2. Server ស្វែងរក User តាម Email ហើយផ្ទៀងផ្ទាត់ Password Hash។
3. បើត្រឹមត្រូវ Server បង្កើត Token ចៃដន្យ និងរក្សាទុកតែ Hash។
4. Server ផ្ញើ Token មួយដងទៅ Client។
5. Client ផ្ញើ Token តាម Authorization Header នៅ Request បន្ទាប់។

ឧទាហរណ៍ Handler សម្រាប់ Login។ ឧបមាថាមាន jsonResponse(), jsonError() និង readJsonObject() ពីមេរៀនទី ២៦៖

```php
<?php
function login(PDO $pdo): void
{
    $input = readJsonObject();

    $email = $input->email ?? null;
    $password = $input->password ?? null;

    if (
        !is_string($email)
        || !filter_var($email, FILTER_VALIDATE_EMAIL)
        || !is_string($password)
        || $password === ''
        || strlen($password) > 4096
    ) {
        jsonError(422, 'invalid_credentials', 'ព័ត៌មាន Login មិនត្រឹមត្រូវ។');
    }

    $stmt = $pdo->prepare(
        'SELECT id, email, password_hash, role
         FROM users
         WHERE email = :email'
    );
    $stmt->execute(['email' => $email]);

    $user = $stmt->fetch();

    if (
        $user === false
        || !password_verify($password, $user['password_hash'])
    ) {
        // ប្រើសារដូចគ្នា ដើម្បីកុំឱ្យបញ្ជាក់ថា Email មានក្នុងប្រព័ន្ធឬអត់។
        jsonError(401, 'invalid_credentials', 'Email ឬ Password មិនត្រឹមត្រូវ។');
    }

    if (password_needs_rehash($user['password_hash'], PASSWORD_DEFAULT)) {
        $newHash = password_hash($password, PASSWORD_DEFAULT);

        $update = $pdo->prepare(
            'UPDATE users SET password_hash = :password_hash WHERE id = :id'
        );
        $update->execute([
            'password_hash' => $newHash,
            'id' => $user['id'],
        ]);
    }

    $plainToken = bin2hex(random_bytes(32));
    $tokenHash = hash('sha256', $plainToken);
    $expiresAt = gmdate('Y-m-d H:i:s', time() + 3600);

    $saveToken = $pdo->prepare(
        'INSERT INTO api_tokens (user_id, token_hash, expires_at)
         VALUES (:user_id, :token_hash, :expires_at)'
    );
    $saveToken->execute([
        'user_id' => $user['id'],
        'token_hash' => $tokenHash,
        'expires_at' => $expiresAt,
    ]);

    jsonResponse([
        'data' => [
            'token_type' => 'Bearer',
            'access_token' => $plainToken,
            'expires_in' => 3600,
        ],
    ]);
}
```

Token មាន 32 Random Bytes ហើយបម្លែងទៅ Hex ដើម្បីងាយផ្ញើតាម Header។ ដោយសារវាមាន Entropy ខ្ពស់ និងបង្កើតដោយ CSPRNG អាចរក្សា SHA-256 Hash សម្រាប់ Lookup។ Password របស់មនុស្សវិញត្រូវប្រើ password_hash() ព្រោះវាអាចទាយបាន និងត្រូវការចំណាយ CPU ក្នុងការផ្ទៀងផ្ទាត់។

Raw Token ត្រូវបង្ហាញតែម្តងនៅពេល Login។ កុំដាក់ Token ក្នុង Log, URL, Screenshot ឬ Error Message។

## ៥. ផ្ញើ Bearer Token ពី Client

Client ផ្ញើ Token ក្នុង Authorization Header៖

```http
Authorization: Bearer 64-character-hex-token
```

ឧទាហរណ៍៖

```bash
curl -i https://api.example.com/api/profile ^
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

នៅក្នុង HTTP Request Header នេះ Token គឺជា Bearer Credential៖ អ្នកណាដែលកាន់ Token អាចប្រើវាបានរហូតដល់ផុតកំណត់ ឬត្រូវបាន Revoked។ ដូច្នេះត្រូវប្រើ HTTPS ជានិច្ច និងកុំដាក់ Token ក្នុង Query String។

## ៦. Authentication Middleware

បង្កើត Function ដែលទាញ Token ពី Header, Hash វា ហើយរក User ដែល Token មិនទាន់ផុតកំណត់ និងមិនទាន់ត្រូវបាន Revoked៖

```php
<?php
function authenticatedUser(PDO $pdo): array
{
    $authorization = $_SERVER['HTTP_AUTHORIZATION'] ?? '';

    if (
        !is_string($authorization)
        || !preg_match(
            '/^Bearer\s+([a-f0-9]{64})$/i',
            trim($authorization),
            $matches
        )
    ) {
        header('WWW-Authenticate: Bearer');
        jsonError(401, 'unauthorized', 'ត្រូវ Login ដើម្បីបន្ត។');
    }

    $tokenHash = hash('sha256', strtolower($matches[1]));

    $stmt = $pdo->prepare(
        'SELECT
            users.id,
            users.email,
            users.role,
            api_tokens.id AS token_id
         FROM api_tokens
         INNER JOIN users ON users.id = api_tokens.user_id
         WHERE api_tokens.token_hash = :token_hash
           AND api_tokens.revoked_at IS NULL
           AND api_tokens.expires_at > UTC_TIMESTAMP()'
    );
    $stmt->execute(['token_hash' => $tokenHash]);

    $user = $stmt->fetch();

    if ($user === false) {
        header('WWW-Authenticate: Bearer');
        jsonError(401, 'unauthorized', 'Token មិនត្រឹមត្រូវ ឬផុតកំណត់។');
    }

    return $user;
}
```

Web Server ឬ Reverse Proxy ខ្លះអាចមិនបញ្ជូន Authorization Header មក PHP ដោយស្វ័យប្រវត្តិ។ ត្រូវកំណត់ Web Server ឱ្យបញ្ជូន Header នោះត្រឹមត្រូវ ហើយសាកល្បងនៅ Deployment Environment។ កុំទទួល Token ពី URL ជំនួសវិញ។

## ៧. Authorization និង Resource Ownership

ក្រោយ Authentication ត្រូវពិនិត្យសិទ្ធិលើ Resource។ វិធីមួយគឺបញ្ចូល User ID ក្នុង Query ដើម្បីទាញតែ Resource របស់អ្នកប្រើ៖

```php
<?php
$user = authenticatedUser($pdo);
$studentId = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

if ($studentId === false || $studentId === null || $studentId < 1) {
    jsonError(400, 'invalid_id', 'ID មិនត្រឹមត្រូវ។');
}

$stmt = $pdo->prepare(
    'SELECT id, name, email
     FROM students
     WHERE id = :id AND owner_user_id = :user_id'
);
$stmt->execute([
    'id' => $studentId,
    'user_id' => $user['id'],
]);

$student = $stmt->fetch();

if ($student === false) {
    // អាចត្រឡប់ 404 ដើម្បីកុំបង្ហាញថា Resource របស់អ្នកដទៃមានឬអត់។
    jsonError(404, 'not_found', 'រកមិនឃើញ Resource ទេ។');
}

jsonResponse(['data' => $student]);
```

ឧទាហរណ៍នេះសន្មតថា Table students មាន owner_user_id។ បើ Project មិនប្រើ Ownership តាម User ទេ សូមអនុវត្ត Rule តាម Role ឬ Permission របស់ Project។

Role Check សាមញ្ញ៖

```php
<?php
function requireAdmin(array $user): void
{
    if ($user['role'] !== 'admin') {
        jsonError(403, 'forbidden', 'អ្នកមិនមានសិទ្ធិធ្វើសកម្មភាពនេះទេ។');
    }
}
```

កុំយក User ID ឬ Role ពី Request Body មកទុកចិត្ត។ ប្រើព័ត៌មាន User ដែល Server ទាញបានពី Token ដែលបានផ្ទៀងផ្ទាត់។

## ៨. Revocation និង Logout

Token Opaque ដែលមាន Record នៅ Database អាចត្រូវបាន Revoked ដោយកំណត់ revoked_at៖

```php
<?php
$user = authenticatedUser($pdo);

$stmt = $pdo->prepare(
    'UPDATE api_tokens
     SET revoked_at = UTC_TIMESTAMP()
     WHERE id = :token_id AND user_id = :user_id'
);
$stmt->execute([
    'token_id' => $user['token_id'],
    'user_id' => $user['id'],
]);

http_response_code(204);
exit;
```

បន្ទាប់ពី Revocation Request បន្ទាប់ដែលប្រើ Token នោះនឹងទទួល 401។ អាចគ្រប់គ្រង Token ច្រើនសម្រាប់ Device ផ្សេងៗ ដោយមាន Token Record ដាច់ពីគ្នា។

គោលការណ៍គ្រប់គ្រង Token៖

- កំណត់ Expiration ឱ្យសមស្រប។
- អនុញ្ញាតឱ្យ User មើល និងដកសិទ្ធិ Device/Token ចាស់។
- បង្កើត Token ថ្មីពេល Rotate ហើយបិទ Token ចាស់។
- កំណត់ Scopes ឬ Permissions តិចបំផុតដែលត្រូវការ។
- កុំរក្សា Token Raw នៅ Database ឬ Log។

## ៩. Session Cookie ឬ Bearer Token?

| លក្ខណៈ | Session Cookie | Bearer Token |
|---|---|---|
| សមស្របសម្រាប់ | Web App ដែល Server គ្រប់គ្រង Browser | API សម្រាប់ Mobile/Service Client |
| អ្នកផ្ញើ Credential | Browser ផ្ញើ Cookie ដោយស្វ័យប្រវត្តិ | Client ផ្ញើ Authorization Header |
| CSRF | ត្រូវគិតគូរព្រោះ Browser ផ្ញើ Cookie ដោយស្វ័យប្រវត្តិ | ជាទូទៅមិនមាន CSRF ដូច Cookie ប៉ុន្តែ Token អាចត្រូវលួច |
| Revocation | លុប Server Session | ត្រូវរក្សាទុក/គ្រប់គ្រង Token Record ឬប្រើ Token ខ្លី |
| ការផ្ទុក Client | Cookie មាន HttpOnly/Secure/SameSite | Client ត្រូវរក្សា Token ឱ្យសុវត្ថិភាព |

ជ្រើសរើសតាមប្រភេទ Client និងរចនាសម្ព័ន្ធកម្មវិធី។ កុំប្រើ Bearer Token ដោយសារតែវាពេញនិយម ប្រសិនបើ Session Cookie សមស្របជាងសម្រាប់ Web App របស់អ្នក។

## ១០. Bearer Token ឬ JWT?

ឧទាហរណ៍នេះប្រើ Opaque Token ដែល Server រក្សាទុក Hash ក្នុង Database៖

- ងាយ Revoked ដោយកែ Token Record។
- Request ត្រូវ Lookup Token នៅ Server។
- Token មិនបង្ហាញ Claims របស់ User ដល់ Client។

JWT ជា Format Token មួយផ្សេងទៀត ដែលអាចផ្ទៀងផ្ទាត់ដោយហត្ថលេខា។ វាត្រូវការការគ្រប់គ្រង Key, Expiration, Audience និង Revocation Strategy យ៉ាងប្រុងប្រយ័ត្ន។ កុំសរសេរ JWT Format ផ្ទាល់ដោយមិនប្រើ Library ដែលបានពិនិត្យ និងមិនត្រូវសន្មតថា Payload ដែល Encode ជា Base64 មានការសម្ងាត់ទេ។

## ១១. សុវត្ថិភាពសំខាន់ៗ

1. **ប្រើ HTTPS ជានិច្ច។** Bearer Token ជា Credential ដែលអ្នកកាន់អាចប្រើបាន។
2. **Hash Password ដោយ password_hash()។** កុំប្រើ MD5, SHA-1 ឬ SHA-256 ផ្ទាល់សម្រាប់ Password។
3. **បង្កើត Token ដោយ random_bytes()។** កុំប្រើ rand(), uniqid() ឬ Timestamp។
4. **រក្សា Token Hash នៅ Database និង Raw Token តែ Client។**
5. **បន្ថែម Rate Limiting លើ Login។** វាជួយកាត់បន្ថយការទាយ Password។
6. **ប្រើ Error Message ទូទៅសម្រាប់ Login Failure។** កុំបញ្ជាក់ថា Email មានឬអត់។
7. **ពិនិត្យ Authorization លើគ្រប់ Resource Request។** Authentication មិនគ្រប់គ្រាន់ទេ។
8. **កំណត់ Expiration, Revocation និង Scope។**
9. **កុំដាក់ Token ក្នុង URL, Log, Analytics ឬ Browser History។**
10. **CORS មិនមែន Authentication ទេ។** វាជាការគ្រប់គ្រង Browser Origin។

## ១២. កំហុសដែលជួបញឹកញាប់

- រក្សាទុក Password ជាអត្ថបទធម្មតា។
- Hash Password ដោយ SHA-256 ជំនួស password_hash()។
- ផ្ញើ Bearer Token តាម HTTP មិនមែន HTTPS។
- រក្សាទុក Token Raw នៅ Database។
- ទុក Token មិនផុតកំណត់ និងគ្មាន Revocation។
- ពិនិត្យតែថា Token ត្រឹមត្រូវ ប៉ុន្តែមិនពិនិត្យ Ownership ឬ Role។
- ទទួល Role ឬ User ID ពី Client ដោយមិនផ្ទៀងផ្ទាត់។
- ដាក់ Token ក្នុង Query String។
- ប្រើ Error Message ផ្សេងគ្នាសម្រាប់ Email មិនមាន និង Password ខុស។
- គិតថា CORS រារាំង Client ដែលមិនបាន Login។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Password Hash

បង្កើត Password Hash ដោយ password_hash() ហើយ Verify ដោយ password_verify()។

### លំហាត់ទី ២៖ Bearer Token

បង្កើត Token ដោយ random_bytes(32), បម្លែងទៅ Hex និងរក្សាទុកតែ SHA-256 Hash។

### លំហាត់ទី ៣៖ Authentication

បង្កើត Function ដែលអាន Authorization Header, ផ្ទៀងផ្ទាត់ Token និងទាញ User ពី Database។

### លំហាត់ទី ៤៖ Authorization

កែ GET Student Endpoint ឱ្យអ្នកប្រើអាចអានបានតែ Student ដែលជារបស់ខ្លួន ឬ User ដែលមាន Role Admin។

### លំហាត់ទី ៥៖ Revocation

បង្កើត Endpoint សម្រាប់ Revocation Token បច្ចុប្បន្ន ហើយសាកល្បងថា Token នោះប្រើមិនបានទៀត។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Authentication, Authorization, CSRF និង CORS។

## ១៤. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- Authentication ផ្ទៀងផ្ទាត់អត្តសញ្ញាណ។
- Authorization ពិនិត្យសិទ្ធិលើសកម្មភាព ឬ Resource។
- CSRF ជាការបោក Browser ដែលមាន Cookie ឱ្យផ្ញើ Request មិនបានបំណង។
- CORS កំណត់ថា Browser អនុញ្ញាត Web Page ពី Origin ផ្សេងអាន Response ឬអត់។
- password_hash() សម្រាប់ Password របស់មនុស្ស; SHA-256 អាចប្រើ Hash Opaque Token ដែលបង្កើតដោយ Random CSPRNG និងមាន Entropy ខ្ពស់។
- Bearer Token ត្រូវការការការពារ HTTPS ព្រោះអ្នកកាន់ Token អាចប្រើវាបាន។

---

## សេចក្ដីសង្ខេប

- Authentication សួរថាអ្នកជានរណា; Authorization សួរថាអ្នកអាចធ្វើអ្វី។
- រក្សា Password ដោយ password_hash() ហើយ Verify ដោយ password_verify()។
- បង្កើត Opaque Token ដោយ random_bytes(); រក្សា Hash នៅ Server ហើយបង្ហាញ Raw Token តែម្តង។
- ផ្ញើ Token ក្នុង Authorization: Bearer Header តាម HTTPS។
- ពិនិត្យ Ownership/Role សម្រាប់គ្រប់ Resource។
- កំណត់ Expiration, Revocation និង Rate Limiting។
- CORS មិនជំនួស Authentication, Authorization ឬ CSRF Protection ទេ។

## ឯកសារយោង

- [PHP Password Hashing](https://www.php.net/manual/en/book.password.php)
- [PHP random_bytes()](https://www.php.net/manual/en/function.random-bytes.php)
- [RFC 6750: Bearer Token Usage](https://www.rfc-editor.org/rfc/rfc6750.html)
- [PSR-11 Container Interface](https://www.php-fig.org/psr/psr-11/)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្ស API Rate Limiting, Audit Logging និងការការពារ Brute Force សម្រាប់ Login Endpoint។
