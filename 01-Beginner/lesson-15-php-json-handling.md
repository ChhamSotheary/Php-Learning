# មេរៀនទី ១៥៖ PHP JSON Handling (ការគ្រប់គ្រង JSON)

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពី JSON និងការប្រើវាសម្រាប់ផ្លាស់ប្ដូរទិន្នន័យ។
- បម្លែង PHP Array និង Object ទៅ JSON ដោយប្រើ \`json_encode()\`។
- បម្លែង JSON ទៅ PHP Value ដោយប្រើ \`json_decode()\`។
- ជ្រើសរើសរវាង PHP Object និង Associative Array ពេល Decode។
- គ្រប់គ្រង JSON Error ដោយប្រើ \`JSON_THROW_ON_ERROR\`។
- ទទួល និងផ្ញើ JSON តាម HTTP API។
- Validate ទិន្នន័យ JSON និងគិតពីសុវត្ថិភាព។

> តម្រូវការ៖ ឧទាហរណ៍ប្រើ PHP 8.0 ឡើងទៅ។ \`JSON_THROW_ON_ERROR\` មានតាំងពី PHP 7.3 ឡើងទៅ។

---

## ១. JSON ជាអ្វី?

JSON (JavaScript Object Notation) ជាទម្រង់អត្ថបទសម្រាប់តំណាងទិន្នន័យ។ វាត្រូវបានប្រើជាញឹកញាប់ដើម្បីផ្ញើទិន្នន័យរវាង Browser, Server និង API។

ឧទាហរណ៍ JSON៖

\`\`\`json
{
  "name": "Sokha",
  "age": 20,
  "is_student": true,
  "courses": ["PHP", "MySQL"],
  "address": null
}
\`\`\`

ប្រភេទតម្លៃ JSON ដែលមាន៖

- String៖ \`"hello"\`
- Number៖ \`25\` ឬ \`3.14\`
- Boolean៖ \`true\` ឬ \`false\`
- Null៖ \`null\`
- Array៖ \`["PHP", "HTML"]\`
- Object៖ \`{"name": "Sokha"}\`

JSON មិនគាំទ្រ Comment ឬ Comma នៅចុងធាតុចុងក្រោយទេ។

## ២. បម្លែង PHP ទៅ JSON ដោយ json_encode()

ប្រើ \`json_encode()\` ដើម្បីបម្លែង PHP Value ទៅជា JSON String៖

\`\`\`php
<?php
$user = [
    'name' => 'Sokha',
    'age' => 20,
    'is_student' => true,
    'courses' => ['PHP', 'MySQL'],
];

$json = json_encode($user);

if ($json === false) {
    exit('មិនអាចបម្លែងទៅ JSON បានទេ។');
}

echo $json;
\`\`\`

លទ្ធផល៖

\`\`\`json
{"name":"Sokha","age":20,"is_student":true,"courses":["PHP","MySQL"]}
\`\`\`

ជាទូទៅ៖

- PHP Associative Array ត្រូវបានបម្លែងទៅជា JSON Object។
- PHP Indexed Array ត្រូវបានបម្លែងទៅជា JSON Array។
- PHP Object ត្រូវបានបម្លែងតាម Properties ដែលអាចមើលឃើញបាន (visible properties)។

ឧទាហរណ៍ Indexed Array៖

\`\`\`php
<?php
$colors = ['red', 'green', 'blue'];

echo json_encode($colors);
\`\`\`

លទ្ធផល៖

\`\`\`json
["red","green","blue"]
\`\`\`

## ៣. ជម្រើសសំខាន់ៗសម្រាប់ json_encode()

### ៣.១ បង្ហាញ JSON ឱ្យអានងាយ

ប្រើ \`JSON_PRETTY_PRINT\`៖

\`\`\`php
<?php
$data = [
    'name' => 'Sokha',
    'skills' => ['PHP', 'JSON'],
];

echo json_encode($data, JSON_PRETTY_PRINT);
\`\`\`

### ៣.២ រក្សាទុកអក្សរ Unicode

PHP អាច Escape អក្សរ Unicode ជា \`\\uXXXX\`។ ប្រើ \`JSON_UNESCAPED_UNICODE\` ដើម្បីរក្សាអក្សរខ្មែរ និងអក្សរផ្សេងៗឱ្យអានបាន៖

\`\`\`php
<?php
$data = ['message' => 'សួស្តី'];

echo json_encode($data, JSON_UNESCAPED_UNICODE);
\`\`\`

### ៣.៣ រក្សាទុក Slash

ប្រើ \`JSON_UNESCAPED_SLASHES\` ប្រសិនបើចង់ឱ្យ URL មិនបង្ហាញ Slash ជា Escape៖

\`\`\`php
<?php
$data = ['website' => 'https://example.com'];

echo json_encode($data, JSON_UNESCAPED_SLASHES);
\`\`\`

អាចផ្សំ Flags ដោយប្រើសញ្ញា \`|\`៖

\`\`\`php
<?php
$options = JSON_PRETTY_PRINT
    | JSON_UNESCAPED_UNICODE
    | JSON_UNESCAPED_SLASHES;

echo json_encode($data, $options);
\`\`\`

## ៤. បម្លែង JSON ទៅ PHP ដោយ json_decode()

\`json_decode()\` បម្លែង JSON String ទៅ PHP Value។ តាមលំនាំដើម JSON Object ត្រូវបានបម្លែងទៅជា PHP Object៖

\`\`\`php
<?php
$json = '{"name":"Sokha","age":20}';

$user = json_decode($json);

echo $user->name;
echo $user->age;
\`\`\`

ដាក់ \`true\` ជា Argument ទីពីរ ដើម្បីបម្លែង JSON Object ទៅ Associative Array៖

\`\`\`php
<?php
$json = '{"name":"Sokha","age":20}';

$user = json_decode($json, true);

echo $user['name'];
echo $user['age'];
\`\`\`

ជ្រើសរើសទម្រង់មួយដែលស្របគ្នានៅក្នុងកម្មវិធី៖

- ប្រើ Object៖ អានតម្លៃដូចជា \`$user->name\`។
- ប្រើ Associative Array៖ អានតម្លៃដូចជា \`$user['name']\`។

ចំពោះ JSON Array វានឹងក្លាយជា PHP Array៖

\`\`\`php
<?php
$json = '["PHP", "MySQL"]';
$courses = json_decode($json, true);

echo $courses[0];
\`\`\`

## ៥. គ្រប់គ្រង JSON Error

JSON អាចមិនត្រឹមត្រូវ ដូចជា ខ្វះសញ្ញាបិទ Quote ឬមាន Comma លើស។ ប្រើ \`JSON_THROW_ON_ERROR\` ដើម្បីឱ្យ PHP បោះ \`JsonException\` នៅពេល Encode ឬ Decode បរាជ័យ៖

\`\`\`php
<?php
$json = '{"name": "Sokha",}';

try {
    $data = json_decode(
        $json,
        true,
        512,
        JSON_THROW_ON_ERROR
    );

    var_dump($data);
} catch (JsonException $e) {
    echo 'JSON មិនត្រឹមត្រូវ៖ '
        . htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8');
}
\`\`\`

លើកមុនៗ អាចពិនិត្យ \`json_last_error()\` និង \`json_last_error_msg()\` ក្រោយពេលហៅ Encode/Decode។ ប៉ុន្តែសម្រាប់កូដថ្មី \`JSON_THROW_ON_ERROR\` ជួយឱ្យដោះស្រាយកំហុសដោយ \`try/catch\` បានច្បាស់។

\`\`\`php
<?php
$json = json_encode($data);

if ($json === false) {
    error_log(json_last_error_msg());
    exit('មិនអាចបង្កើតទិន្នន័យបានទេ។');
}
\`\`\`

កុំសន្មតថា \`json_encode()\` តែងតែជោគជ័យ។ ឧទាហរណ៍ String ដែលមិនមែនជា UTF-8 ត្រឹមត្រូវអាចធ្វើឱ្យ Encode បរាជ័យ។

## ៦. ទទួល JSON ពី HTTP Request

នៅពេល Client ផ្ញើ JSON មក PHP តាម HTTP POST ជាទូទៅ Body មិនស្ថិតក្នុង \`$_POST\` ទេ។ ប្រើ \`php://input\` ដើម្បីអាន Request Body៖

\`\`\`php
<?php
$rawBody = file_get_contents('php://input');

if ($rawBody === false || $rawBody === '') {
    http_response_code(400);
    exit('សំណើគ្មានទិន្នន័យ។');
}

try {
    $data = json_decode(
        $rawBody,
        true,
        512,
        JSON_THROW_ON_ERROR
    );
} catch (JsonException $e) {
    http_response_code(400);
    exit('ទម្រង់ JSON មិនត្រឹមត្រូវ។');
}

var_dump($data);
\`\`\`

បន្ទាប់ពី Decode ត្រូវ Validate ទិន្នន័យ៖

\`\`\`php
<?php
if (!is_array($data)) {
    http_response_code(400);
    exit('ទិន្នន័យត្រូវតែជា JSON Object។');
}

$name = $data['name'] ?? null;

if (!is_string($name) || trim($name) === '') {
    http_response_code(422);
    exit('ត្រូវការឈ្មោះដែលត្រឹមត្រូវ។');
}
\`\`\`

ការដែល JSON Syntax ត្រឹមត្រូវ មិនមានន័យថាទិន្នន័យទាំងអស់មានតម្លៃត្រឹមត្រូវ ឬគួរឱ្យទុកចិត្តទេ។ ត្រូវ Validate ប្រភេទ និងលក្ខខណ្ឌរបស់ Field នីមួយៗ។

## ៧. ផ្ញើ JSON ពី PHP ជា API Response

API ដែលផ្ញើ JSON គួរកំណត់ HTTP Header ត្រឹមត្រូវ៖

\`\`\`php
<?php
header('Content-Type: application/json; charset=utf-8');

$response = [
    'success' => true,
    'message' => 'ទទួលបានសំណើដោយជោគជ័យ',
    'data' => [
        'id' => 101,
        'name' => 'Sokha',
    ],
];

try {
    echo json_encode(
        $response,
        JSON_THROW_ON_ERROR
            | JSON_UNESCAPED_UNICODE
            | JSON_UNESCAPED_SLASHES
    );
} catch (JsonException $e) {
    error_log($e->getMessage());
    http_response_code(500);

    echo '{"success":false,"message":"មានបញ្ហាក្នុងប្រព័ន្ធ។"}';
}
\`\`\`

ចំណុចសំខាន់ៗ៖

- កំណត់ \`Content-Type: application/json\` មុនបញ្ចេញ Body។
- ប្រើ HTTP Status Code សមស្រប ដូចជា 200, 400, 404, 422 ឬ 500។
- កុំបញ្ចូល HTML ឬ Warning របស់ PHP ក្នុង JSON Response។
- កុំបង្ហាញ Stack Trace ឬព័ត៌មានសម្ងាត់ទៅ Client។

## ៨. Mini Project៖ JSON API សម្រាប់បញ្ជីកិច្ចការ

បង្កើត Folder និងឯកសារ៖

\`\`\`text
php-json-api/
└── index.php
\`\`\`

បង្កើត \`index.php\`៖

\`\`\`php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=utf-8');

function sendJson(int $statusCode, array $payload): never
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
        error_log($e->getMessage());
        http_response_code(500);
        echo '{"success":false,"message":"មានបញ្ហាក្នុងប្រព័ន្ធ។"}';
    }

    exit;
}

$tasks = [
    ['id' => 1, 'title' => 'រៀន PHP', 'completed' => false],
    ['id' => 2, 'title' => 'ហាត់ JSON', 'completed' => true],
];

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($method === 'GET') {
    sendJson(200, [
        'success' => true,
        'data' => $tasks,
    ]);
}

if ($method === 'POST') {
    $rawBody = file_get_contents('php://input');

    if ($rawBody === false || $rawBody === '') {
        sendJson(400, [
            'success' => false,
            'message' => 'សំណើគ្មានទិន្នន័យ។',
        ]);
    }

    try {
        $input = json_decode(
            $rawBody,
            true,
            512,
            JSON_THROW_ON_ERROR
        );
    } catch (JsonException $e) {
        sendJson(400, [
            'success' => false,
            'message' => 'ទម្រង់ JSON មិនត្រឹមត្រូវ។',
        ]);
    }

    if (!is_array($input)) {
        sendJson(400, [
            'success' => false,
            'message' => 'ទិន្នន័យត្រូវតែជា JSON Object។',
        ]);
    }

    $title = $input['title'] ?? null;

    if (!is_string($title) || trim($title) === '') {
        sendJson(422, [
            'success' => false,
            'message' => 'ត្រូវបញ្ចូលចំណងជើងកិច្ចការ។',
        ]);
    }

    $newTask = [
        'id' => 3,
        'title' => trim($title),
        'completed' => false,
    ];

    sendJson(201, [
        'success' => true,
        'message' => 'បង្កើតកិច្ចការរួចរាល់។',
        'data' => $newTask,
    ]);
}

header('Allow: GET, POST');

sendJson(405, [
    'success' => false,
    'message' => 'Method នេះមិនត្រូវបានអនុញ្ញាតទេ។',
]);
\`\`\`

> គម្រោងនេះបង្ហាញរចនាសម្ព័ន្ធ API ប៉ុណ្ណោះ។ ទិន្នន័យមិនត្រូវបានរក្សាទុករវាង Request ទេ ព្រោះ Array \`$tasks\` ត្រូវបានបង្កើតឡើងវិញរាល់ពេល។ កម្មវិធីពិតគួរប្រើ Database និង Authentication តាមតម្រូវការ។

ដំណើរការ៖

1. បើក Terminal ក្នុង Folder \`php-json-api\`។
2. ដំណើរការ៖

\`\`\`bash
php -S localhost:8000
\`\`\`

3. សាកល្បង GET ដោយបើក \`http://localhost:8000\` នៅ Browser។
4. សាកល្បង POST ដោយប្រើ cURL៖

\`\`\`bash
curl -X POST http://localhost:8000 ^
  -H "Content-Type: application/json" ^
  -d "{\"title\":\"អានមេរៀន JSON\"}"
\`\`\`

នៅ macOS ឬ Linux ប្រើ Backslash ជំនួសសញ្ញា \`^\` សម្រាប់បំបែក Command ច្រើនបន្ទាត់ ឬដាក់ Command ទាំងមូលក្នុងមួយបន្ទាត់។

លទ្ធផល POST គួរមាន Status Code 201 និង JSON ដែលមានកិច្ចការថ្មី។

## ៩. សុវត្ថិភាព និង Best Practices

1. **Validate ទិន្នន័យក្រោយ Decode។** ពិនិត្យ Field, ប្រភេទទិន្នន័យ, ប្រវែង និងតម្លៃដែលអនុញ្ញាត។
2. **កំណត់ទំហំ Request។** កុំអនុញ្ញាត Body ធំពេក; គ្រប់គ្រងទាំង Server Configuration និង Application។
3. **កុំទុកចិត្ត Content-Type តែមួយមុខ។** ពិនិត្យទិន្នន័យពិតផងដែរ។
4. **កុំបញ្ចូលទិន្នន័យ JSON ទៅ SQL ដោយផ្ទាល់។** ប្រើ PDO Prepared Statements។
5. **Escape តាមបរិបទពេលបង្ហាញ។** JSON encoding មិនជំនួស HTML Escaping ទេ។
6. **កុំផ្ញើព័ត៌មានសម្ងាត់។** មិនត្រូវបញ្ចេញ Password Hash, Access Token ឬព័ត៌មានឯកជនដោយគ្មានហេតុផល។
7. **គ្រប់គ្រង Error ដោយសុវត្ថិភាព។** កត់ត្រាព័ត៌មានបច្ចេកទេសក្នុង Log; ផ្ញើសារទូទៅ និង Status Code ទៅ Client។
8. **កុំសរសេរ JSON ដោយភ្ជាប់ String ដោយដៃ។** ប្រើ \`json_encode()\` ដើម្បី Escape តួអក្សរឱ្យបានត្រឹមត្រូវ។
9. **ប្រើ HTTPS សម្រាប់ API ពិតប្រាកដ។** វាការពារទិន្នន័យពេលធ្វើដំណើររវាង Client និង Server។

## ១០. កំហុសដែលអ្នកចាប់ផ្ដើមរៀនជួបញឹកញាប់

- **ភ្លេច \`true\` ក្នុង \`json_decode()\` ហើយប្រើ Array Syntax៖** លទ្ធផលលំនាំដើមជា Object។
- **មិនចាប់ JSON Error៖** JSON ខូចអាចធ្វើឱ្យលទ្ធផល Decode មិនដូចការរំពឹង។
- **គិតថា JSON ត្រឹមត្រូវមានន័យថា Input មានសុវត្ថិភាព៖** ត្រូវ Validate តម្លៃទាំងអស់នៅ Server។
- **អាន JSON Request ពី \`$_POST\`៖** JSON Body ជាទូទៅត្រូវអានពី \`php://input\`។
- **ភ្លេច Content-Type Response៖** Client ប្រហែលមិនដឹងថា Body ជា JSON។
- **បញ្ចូល Warning ឬ HTML ក្នុង Response៖** វាធ្វើឱ្យ JSON មិនអាច Parse បាន។
- **បង្ហាញ Exception Message ឬ Stack Trace ទៅ Client៖** អាចបញ្ចេញព័ត៌មានខាងក្នុង។
- **ប្ដូរ Indexed Array ទៅ Associative Array ដោយមិនប្រុងប្រយ័ត្ន៖** JSON អាចក្លាយពី Array ទៅ Object។

## ១១. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Encode ទិន្នន័យ

បង្កើត Associative Array មានឈ្មោះ អាយុ និងជំនាញជាភាសាខ្មែរ។ បម្លែងទៅ JSON ដោយរក្សាអក្សរ Unicode ឱ្យអានបាន។

### លំហាត់ទី ២៖ Decode JSON

បម្លែង JSON String ខាងក្រោមទៅ Associative Array ហើយបង្ហាញឈ្មោះ និង Email៖

\`\`\`json
{"name":"Dara","email":"dara@example.com"}
\`\`\`

### លំហាត់ទី ៣៖ ចាប់ JSON Exception

សាកល្បង Decode JSON ដែលមិនត្រឹមត្រូវដោយប្រើ \`JSON_THROW_ON_ERROR\` ហើយចាប់ \`JsonException\`។

### លំហាត់ទី ៤៖ Validate API Input

ទទួល JSON Object ដែលមាន Field \`title\`។ ត្រូវពិនិត្យថា៖

- JSON Syntax ត្រឹមត្រូវ។
- Root Value ជា Object។
- \`title\` ជា String មិនទទេ។
- ចំណងជើងមិនលើស 200 តួអក្សរ។

### លំហាត់ទី ៥៖ API Response

បង្កើត Response ដែលមាន \`success\`, \`message\`, និង \`data\`។ កំណត់ Content-Type និង Status Code ឱ្យសមស្រប។

## ១២. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

\`\`\`php
<?php
$student = [
    'name' => 'ដារ៉ា',
    'age' => 19,
    'skills' => ['PHP', 'HTML'],
];

echo json_encode(
    $student,
    JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE
);
\`\`\`

### ចម្លើយលំហាត់ទី ២

\`\`\`php
<?php
$json = '{"name":"Dara","email":"dara@example.com"}';

try {
    $user = json_decode($json, true, 512, JSON_THROW_ON_ERROR);

    echo $user['name'] . "\n";
    echo $user['email'];
} catch (JsonException $e) {
    echo 'JSON មិនត្រឹមត្រូវ។';
}
\`\`\`

### ចម្លើយលំហាត់ទី ៣

\`\`\`php
<?php
$json = '{"name":"Dara",}';

try {
    $data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    echo 'មិនអាចអាន JSON បានទេ។';
}
\`\`\`

### ចម្លើយលំហាត់ទី ៤

\`\`\`php
<?php
$rawBody = file_get_contents('php://input');

try {
    $data = json_decode($rawBody ?: '', true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $e) {
    http_response_code(400);
    exit('JSON មិនត្រឹមត្រូវ។');
}

$title = is_array($data) ? ($data['title'] ?? null) : null;

if (
    !is_string($title)
    || trim($title) === ''
    || mb_strlen($title) > 200
) {
    http_response_code(422);
    exit('ចំណងជើងមិនត្រឹមត្រូវ។');
}

echo 'ទទួលបានចំណងជើងត្រឹមត្រូវ។';
\`\`\`

### ចម្លើយលំហាត់ទី ៥

\`\`\`php
<?php
header('Content-Type: application/json; charset=utf-8');
http_response_code(200);

echo json_encode(
    [
        'success' => true,
        'message' => 'ជោគជ័យ',
        'data' => ['id' => 1],
    ],
    JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE
);
\`\`\`

---

## សេចក្ដីសង្ខេប

- JSON ជាទម្រង់អត្ថបទសម្រាប់ផ្លាស់ប្ដូរទិន្នន័យ។
- \`json_encode()\` បម្លែង PHP Value ទៅ JSON String។
- \`json_decode()\` បម្លែង JSON String ទៅ PHP Object ឬ Array។
- \`JSON_THROW_ON_ERROR\` ជួយគ្រប់គ្រង JSON Error តាម \`try/catch\`។
- JSON Request Body អានតាម \`php://input\`។
- API Response គួរកំណត់ Content-Type, Status Code និង Validate ទិន្នន័យ។
- JSON មិនធ្វើឱ្យទិន្នន័យមានសុវត្ថិភាពដោយស្វ័យប្រវត្តិទេ; ត្រូវ Validate និងការពារទិន្នន័យឯកជន។

## មេរៀនបន្ទាប់

សូមបន្តទៅមេរៀនបន្ទាប់ក្នុងផែនទីសិក្សា ដើម្បីយកចំណេះដឹង PHP មូលដ្ឋានទៅប្រើក្នុងការភ្ជាប់ Database និងបង្កើត API ពេញលេញ។
