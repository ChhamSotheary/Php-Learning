
# មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Resource និង HTTP Methods ក្នុង REST-style API។
- បង្កើត JSON Response ជាមួយ Content-Type និង HTTP Status Code។
- អាន JSON Request Body តាម php://input។
- Validate Input និងឆ្លើយតបដោយ Error Format ស្របគ្នា។
- អនុវត្ត GET, POST, PUT និង DELETE លើ Resource សិស្ស។
- ប្រើ Prepared Statements និង Escape ព័ត៌មាន Error។
- សាកល្បង API ដោយប្រើ curl។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL, PDO MySQL និង Database Table students តាមមេរៀនទី ១៦។ ឧទាហរណ៍នេះសម្រាប់រៀនក្នុង Local Environment ប៉ុណ្ណោះ។

---

## ១. REST-style API ជាអ្វី?

API អនុញ្ញាតឱ្យកម្មវិធីមួយស្នើទិន្នន័យ ឬសកម្មភាពពីកម្មវិធីមួយទៀតតាម HTTP។

REST-style API រៀបចំ URL ជុំវិញ Resource ហើយប្រើ HTTP Method ដើម្បីបង្ហាញប្រភេទប្រតិបត្តិការ៖

| Method | សកម្មភាព | ឧទាហរណ៍ |
|---|---|---|
| GET | អាន Resource | ទាញបញ្ជីសិស្ស |
| POST | បង្កើត Resource | បង្កើតសិស្សថ្មី |
| PUT | ជំនួស/កែ Resource | កែព័ត៌មានសិស្ស |
| DELETE | លុប Resource | លុបសិស្ស |

ឧទាហរណ៍ URL ជាទម្រង់ Resource៖

- GET /api/students — ទាញបញ្ជីសិស្ស
- GET /api/students/12 — ទាញសិស្សលេខ 12
- POST /api/students — បង្កើតសិស្ស
- PUT /api/students/12 — កែសិស្សលេខ 12
- DELETE /api/students/12 — លុបសិស្សលេខ 12

ក្នុងឧទាហរណ៍ PHP Built-in Server ខាងក្រោម យើងប្រើ File api/students.php និង Query Parameter id ដើម្បីរក្សាឧទាហរណ៍ឱ្យសាមញ្ញ។ Framework Router អាចបម្លែង URL ទៅជា Resource Path ស្អាតជាងនេះបាន។

## ២. HTTP Status Codes សំខាន់ៗ

Status Code ជាលេខ 3 ខ្ទង់ដែលប្រាប់ Client អំពីលទ្ធផល Request៖

| Code | ការប្រើប្រាស់ |
|---|---|
| 200 OK | Request ជោគជ័យ និងមាន Response |
| 201 Created | Resource ថ្មីត្រូវបានបង្កើត |
| 204 No Content | Request ជោគជ័យ តែគ្មាន Response Body |
| 400 Bad Request | JSON ឬ Request Syntax មិនត្រឹមត្រូវ |
| 404 Not Found | Resource មិនមាន |
| 405 Method Not Allowed | Method នេះមិនត្រូវបានគាំទ្រនៅ Endpoint |
| 409 Conflict | ទិន្នន័យប៉ះទង្គិច ដូចជា Email ស្ទួន |
| 415 Unsupported Media Type | Content-Type មិនមែន JSON តាមតម្រូវការ |
| 422 Unprocessable Content | JSON Syntax ត្រឹមត្រូវ ប៉ុន្តែ Input មិនត្រឹមត្រូវ |
| 500 Internal Server Error | បញ្ហាមិនរំពឹងទុកនៅ Server |

Status Code គួរត្រូវនឹងលទ្ធផល ហើយ Client មិនគួរត្រូវទាយពី Error Message ដើម្បីដឹងថា Request ជោគជ័យឬអត់។

## ៣. រចនាសម្ព័ន្ធ JSON Response

ប្រើទម្រង់ស្របគ្នាសម្រាប់ Response៖

ជោគជ័យ៖

```json
{
  "data": {
    "id": 12,
    "name": "ដារ៉ា",
    "email": "dara@example.com"
  }
}
```

កំហុស៖

```json
{
  "error": {
    "code": "validation_failed",
    "message": "Input មិនត្រឹមត្រូវ។",
    "details": {
      "email": "Email មិនត្រឹមត្រូវ។"
    }
  }
}
```

កុំបញ្ចូល Stack Trace, SQL Query, Database Password ឬ File Path ក្នុង API Error Response។ ព័ត៌មានលម្អិតគួរទៅ Server Log។

## ៤. បង្កើត JSON Helper Functions

បង្កើត File api/students.php។ នៅខាងលើ កំណត់ Content-Type និង Helper សម្រាប់ឆ្លើយតប JSON៖

```php
<?php
declare(strict_types=1);

require dirname(__DIR__, 2) . '/src/bootstrap.php';

header('Content-Type: application/json; charset=utf-8');

function jsonResponse(array $payload, int $statusCode = 200): void
{
    http_response_code($statusCode);

    try {
        echo json_encode(
            $payload,
            JSON_THROW_ON_ERROR
                | JSON_UNESCAPED_UNICODE
                | JSON_UNESCAPED_SLASHES
        );
    } catch (JsonException $e) {
        error_log('JSON response encoding failed: ' . $e->getMessage());
        http_response_code(500);
        echo '{"error":{"code":"server_error","message":"មានបញ្ហាក្នុងប្រព័ន្ធ។"}}';
    }

    exit;
}

function jsonError(
    int $statusCode,
    string $code,
    string $message,
    array $details = []
): void {
    $error = [
        'code' => $code,
        'message' => $message,
    ];

    if ($details !== []) {
        $error['details'] = $details;
    }

    jsonResponse(['error' => $error], $statusCode);
}
```

កែ Path របស់ bootstrap.php ឱ្យត្រូវនឹងរចនាសម្ព័ន្ធ Project របស់អ្នក។ Helper jsonResponse() ចេញពី Script ក្រោយផ្ញើ Response ដើម្បីកុំឱ្យកូដបន្តបញ្ចូល Output បន្ថែម។

## ៥. អាន និង Validate JSON Request Body

JSON Body មិនស្ថិតក្នុង $_POST ទេ។ ត្រូវអានពី php://input៖

```php
<?php
function readJsonObject(): stdClass
{
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';

    if (!str_starts_with(strtolower($contentType), 'application/json')) {
        jsonError(
            415,
            'unsupported_media_type',
            'Content-Type ត្រូវតែជា application/json។'
        );
    }

    $rawBody = file_get_contents('php://input');

    if ($rawBody === false || $rawBody === '') {
        jsonError(400, 'empty_body', 'Request Body មិនអាចទទេបានទេ។');
    }

    if (strlen($rawBody) > 1_000_000) {
        jsonError(413, 'payload_too_large', 'Request Body ធំពេក។');
    }

    try {
        $data = json_decode(
            $rawBody,
            false,
            512,
            JSON_THROW_ON_ERROR
        );
    } catch (JsonException $e) {
        jsonError(400, 'invalid_json', 'ទម្រង់ JSON មិនត្រឹមត្រូវ។');
    }

    if (!$data instanceof stdClass) {
        jsonError(400, 'invalid_json_shape', 'Root JSON ត្រូវតែជា Object។');
    }

    return $data;
}
```

ការកំណត់ទំហំ Body ក្នុងកូដជួយបន្ថែមការការពារ ប៉ុន្តែ Web Server និង PHP Configuration ក៏គួរមាន Request Size Limit ផងដែរ។

Validate Field ជាក់លាក់៖

```php
<?php
function validateStudentInput(stdClass $input): array
{
    $errors = [];

    $name = $input->name ?? null;
    $email = $input->email ?? null;

    if (!is_string($name) || trim($name) === '' || mb_strlen($name) > 100) {
        $errors['name'] = 'ឈ្មោះត្រូវមានពី 1 ដល់ 100 តួអក្សរ។';
    }

    if (!is_string($email) || !filter_var($email, FILTER_VALIDATE_EMAIL)) {
        $errors['email'] = 'Email មិនត្រឹមត្រូវ។';
    }

    if ($errors !== []) {
        jsonError(
            422,
            'validation_failed',
            'Input មិនត្រឹមត្រូវ។',
            $errors
        );
    }

    return [
        'name' => trim($name),
        'email' => trim($email),
    ];
}
```

JSON Syntax ត្រឹមត្រូវមិនមានន័យថា Input ត្រឹមត្រូវតាមច្បាប់កម្មវិធីទេ។ Validate ប្រភេទទិន្នន័យ, ប្រវែង និងលក្ខខណ្ឌអាជីវកម្មនៅ Server ជានិច្ច។

## ៦. ស្វែងរក Resource ID

ឧទាហរណ៍នេះយក id ពី Query String៖

```php
<?php
function requestId(): int
{
    $id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

    if ($id === false || $id === null || $id < 1) {
        jsonError(400, 'invalid_id', 'ID មិនត្រឹមត្រូវ។');
    }

    return $id;
}
```

តម្លៃពី URL មិនគួរជឿទុកចិត្តដោយស្វ័យប្រវត្តិទេ។ ត្រូវ Validate មុនយកទៅប្រើក្នុង Query ឬ Business Logic។

## ៧. រៀបចំ Request Method Routing

ទទួល Method របស់ Request ហើយបញ្ជូនទៅ Handler ត្រឹមត្រូវ៖

```php
<?php
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

try {
    switch ($method) {
        case 'GET':
            handleGet($pdo);
            break;

        case 'POST':
            handlePost($pdo);
            break;

        case 'PUT':
            handlePut($pdo);
            break;

        case 'DELETE':
            handleDelete($pdo);
            break;

        default:
            header('Allow: GET, POST, PUT, DELETE');
            jsonError(
                405,
                'method_not_allowed',
                'Method នេះមិនត្រូវបានអនុញ្ញាតទេ។'
            );
    }
} catch (PDOException $e) {
    if ($e->getCode() === '23000') {
        jsonError(409, 'data_conflict', 'ទិន្នន័យប៉ះទង្គិច ឬត្រូវបានប្រើរួចហើយ។');
    }

    error_log('Student API database error: ' . $e->getMessage());
    jsonError(500, 'server_error', 'មានបញ្ហាក្នុងប្រព័ន្ធ។');
}
```

ដាក់ Helper Functions និង Handler Definitions មុន Routing Code ឬដាក់ក្នុង File ផ្សេង រួច Require វា។

## ៨. Handler សម្រាប់ GET

បើគ្មាន id ទាញបញ្ជី; បើមាន id ទាញ Student ម្នាក់៖

```php
<?php
function handleGet(PDO $pdo): void
{
    if (!isset($_GET['id'])) {
        $stmt = $pdo->query(
            'SELECT id, name, email, created_at
             FROM students
             ORDER BY id DESC'
        );

        jsonResponse(['data' => $stmt->fetchAll()]);
    }

    $id = requestId();

    $stmt = $pdo->prepare(
        'SELECT id, name, email, created_at
         FROM students
         WHERE id = :id'
    );
    $stmt->execute(['id' => $id]);

    $student = $stmt->fetch();

    if ($student === false) {
        jsonError(404, 'not_found', 'រកមិនឃើញសិស្សទេ។');
    }

    jsonResponse(['data' => $student]);
}
```

ការទាញ List ក្នុង Project ពិតគួរបន្ថែម Pagination និងកំណត់ទំហំ Page ដើម្បីកុំឱ្យ Response ធំពេក។

## ៩. Handler សម្រាប់ POST

POST បង្កើត Student ថ្មី៖

```php
<?php
function handlePost(PDO $pdo): void
{
    $input = validateStudentInput(readJsonObject());

    $stmt = $pdo->prepare(
        'INSERT INTO students (name, email)
         VALUES (:name, :email)'
    );

    try {
        $stmt->execute($input);
    } catch (PDOException $e) {
        if ($e->getCode() === '23000') {
            jsonError(409, 'email_conflict', 'Email នេះត្រូវបានប្រើរួចហើយ។');
        }

        throw $e;
    }

    $id = (int) $pdo->lastInsertId();

    header('Location: ?id=' . $id);

    jsonResponse([
        'data' => [
            'id' => $id,
            'name' => $input['name'],
            'email' => $input['email'],
        ],
    ], 201);
}
```

201 Created បញ្ជាក់ថា Resource ថ្មីត្រូវបានបង្កើត។ Location Header អាចប្រាប់ Client ពី URL របស់ Resource ថ្មី។

## ១០. Handler សម្រាប់ PUT និង DELETE

PUT កែព័ត៌មាន Student ដែលមានស្រាប់៖

```php
<?php
function handlePut(PDO $pdo): void
{
    $id = requestId();
    $input = validateStudentInput(readJsonObject());

    $exists = $pdo->prepare(
        'SELECT id FROM students WHERE id = :id'
    );
    $exists->execute(['id' => $id]);

    if ($exists->fetchColumn() === false) {
        jsonError(404, 'not_found', 'រកមិនឃើញសិស្សទេ។');
    }

    $stmt = $pdo->prepare(
        'UPDATE students
         SET name = :name, email = :email
         WHERE id = :id'
    );
    $stmt->execute([
        'id' => $id,
        'name' => $input['name'],
        'email' => $input['email'],
    ]);

    jsonResponse([
        'data' => [
            'id' => $id,
            'name' => $input['name'],
            'email' => $input['email'],
        ],
    ]);
}
```

DELETE លុប Student ហើយឆ្លើយតប 204 ដោយគ្មាន Body៖

```php
<?php
function handleDelete(PDO $pdo): void
{
    $id = requestId();

    $stmt = $pdo->prepare(
        'DELETE FROM students WHERE id = :id'
    );
    $stmt->execute(['id' => $id]);

    if ($stmt->rowCount() === 0) {
        jsonError(404, 'not_found', 'រកមិនឃើញសិស្សទេ។');
    }

    http_response_code(204);
    exit;
}
```

204 No Content មានន័យថា Request ជោគជ័យ ហើយ Response មិនមាន Body។ កុំបញ្ចូល JSON Body ក្រោយ Status 204។

## ១១. សាកល្បង API ដោយ curl

ដំណើរការ PHP Server៖

```bash
php -S localhost:8000 -t public
```

ទាញបញ្ជីសិស្ស៖

```bash
curl -i http://localhost:8000/api/students.php
```

បង្កើត Student៖

```bash
curl -i -X POST http://localhost:8000/api/students.php ^
  -H "Content-Type: application/json" ^
  -d "{"name":"ដារ៉ា","email":"dara@example.com"}"
```

ទាញ Student តាម ID៖

```bash
curl -i "http://localhost:8000/api/students.php?id=1"
```

កែ Student៖

```bash
curl -i -X PUT "http://localhost:8000/api/students.php?id=1" ^
  -H "Content-Type: application/json" ^
  -d "{"name":"សុភា","email":"sophea@example.com"}"
```

លុប Student៖

```bash
curl -i -X DELETE "http://localhost:8000/api/students.php?id=1"
```

សញ្ញា ^ សម្រាប់បំបែក Command ច្រើនបន្ទាត់នៅ Windows Command Prompt។ នៅ macOS/Linux ប្រើ Backslash ឬដាក់ Command ជាបន្ទាត់តែមួយ។ នៅ PowerShell អាចប្រើ Backtick ជា Line Continuation ឬដាក់ Command ជាបន្ទាត់តែមួយ។

## ១២. CORS និង Authentication

CORS ជាក្បួន Browser គ្រប់គ្រងការអនុញ្ញាតឱ្យ Web Page ពី Origin ផ្សេងអាន Response។ CORS មិនមែនជា Authentication និងមិនរារាំង Client ផ្សេងៗដូចជា curl មិនឱ្យហៅ API ទេ។

API ពិតប្រាកដដែលត្រូវការអ្នកប្រើប្រាស់ គួរមាន៖

- Authentication ដើម្បីដឹងថា Request ជារបស់អ្នកណា។
- Authorization ដើម្បីពិនិត្យថាអ្នកនោះមានសិទ្ធិលើ Resource ឬអត់។
- HTTPS ដើម្បីការពារទិន្នន័យពេលផ្ញើ។
- Rate Limiting និង Log តាមតម្រូវការ។
- Validation និង Prepared Statements នៅ Server។

ឧទាហរណ៍ក្នុងមេរៀននេះមិនមាន Authentication ឬ Authorization ទេ។ កុំដាក់ Endpoint នេះ Online ដោយគ្មានការការពារសមស្រប។

## ១៣. Best Practices

1. ប្រើ Content-Type application/json សម្រាប់ JSON Response។
2. ពិនិត្យ Content-Type និង Request Body នៅពេលទទួល JSON។
3. កំណត់ទំហំ Request និង Validate Field ទាំងអស់។
4. ប្រើ Status Code ត្រឹមត្រូវ និង Error Format ស្របគ្នា។
5. ប្រើ Prepared Statements សម្រាប់ SQL ដែលមាន Input។
6. កុំបញ្ចេញ Stack Trace, SQL Error ឬ Secret ទៅ Client។
7. ប្រើ Allowlist សម្រាប់ Method និងតម្លៃដែលអនុញ្ញាត។
8. បន្ថែម Pagination សម្រាប់ List Endpoint។
9. កំណត់ Authentication, Authorization, HTTPS និង Rate Limits សម្រាប់ Production API។
10. ពិចារណាថា PUT ជំនួស Representation ពេញលេញ ខណៈ PATCH សម្រាប់ការកែប្រែផ្នែកខ្លះ។

## ១៤. កំហុសដែលជួបញឹកញាប់

- អាន JSON Request Body ពី $_POST ជំនួស php://input។
- ត្រឡប់ Status 200 សម្រាប់គ្រប់លទ្ធផល។
- ផ្ញើ 200 ជាមួយ Error Message ហើយរំពឹងឱ្យ Client ទាយ។
- បង្ហាញ PDOException ឬ Stack Trace ក្នុង Response។
- មិន Validate JSON Root Value និង Field Types។
- ប្រើ GET ដើម្បីផ្លាស់ប្ដូរទិន្នន័យ។
- មិនកំណត់ Allow Header ពេលត្រឡប់ 405។
- ផ្ញើ JSON Body ជាមួយ 204 No Content។
- គិតថា CORS ការពារ API ពីការហៅដោយ Client ផ្សេង។
- មិនដាក់ Authentication ឬ Authorization លើ Resource ឯកជន។

## ១៥. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ GET One

បន្ថែម GET Handler ដែលទាញ Student ម្នាក់តាម ID ហើយត្រឡប់ 404 បើមិនមាន។

### លំហាត់ទី ២៖ Validate POST

សាកល្បង POST ដោយ JSON ខូច, Email មិនត្រឹមត្រូវ និង Name ទទេ។ ពិនិត្យ Status Code និង Error Shape។

### លំហាត់ទី ៣៖ Update

បង្កើត PUT Request ដែល Validate Body និងពិនិត្យ Resource មានមុនកែប្រែ។

### លំហាត់ទី ៤៖ Delete

បង្កើត DELETE Request ដែលត្រឡប់ 404 ពេល ID មិនមាន និង 204 ពេលលុបជោគជ័យ។

### លំហាត់ទី ៥៖ Pagination

បន្ថែម page និង per_page ទៅ GET List។ Validate តម្លៃ និងកំណត់ Maximum Page Size។

### លំហាត់ទី ៦៖ Security Review

រាយបញ្ជីចំណុចដែលត្រូវបន្ថែមមុនដាក់ API នេះទៅ Production ដោយផ្អែកលើ Authentication, Authorization, HTTPS, Rate Limiting និង Error Logging។

## ១៦. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- GET អាន Resource; POST បង្កើត; PUT កែ Representation; DELETE លុប។
- JSON Body ត្រូវអានតាម php://input ហើយ Decode ជាមួយ JSON_THROW_ON_ERROR។
- 400 សម្រាប់ Request មិនត្រឹមត្រូវ; 422 សម្រាប់ Content ដែល Parse បានតែបំពាន Validation; 404 សម្រាប់ Resource មិនមាន; 500 សម្រាប់ Error Server មិនរំពឹងទុក។
- Prepared Statements ការពារ SQL Injection ប៉ុន្តែមិនជំនួស Input Validation ទេ។
- CORS ជាការកំណត់ Browser; វាមិនបញ្ជាក់អត្តសញ្ញាណ ឬសិទ្ធិអ្នកប្រើទេ។

---

## សេចក្ដីសង្ខេប

- REST-style API ប្រើ Resource URLs និង HTTP Methods ដើម្បីតំណាងសកម្មភាព។
- JSON Request Body អានពី php://input។
- Response គួរមាន Content-Type, Status Code និង Error Shape ស្របគ្នា។
- Validate Input, ប្រើ Prepared Statements និងលាក់ Error លម្អិតពី Client។
- Authentication និង Authorization ជាការងារដាច់ដោយឡែកពី CORS។
- ឧទាហរណ៍នេះសម្រាប់រៀន; Production API ត្រូវការការការពារ និងការរៀបចំបន្ថែម។

## ឯកសារយោង

- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)
- [PHP http_response_code()](https://www.php.net/manual/en/function.http-response-code.php)
- [PHP filter_input()](https://www.php.net/manual/en/function.filter-input.php)
- [PHP JSON Functions](https://www.php.net/manual/en/ref.json.php)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្ស Authentication និង Authorization សម្រាប់ API រួមមាន Password Hashing, Login Flow, Session ឬ Token និងការការពារ Endpoint។
