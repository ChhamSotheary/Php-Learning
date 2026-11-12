# មេរៀនទី ៥១៖ ហៅ External API ដោយ PHP cURL

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ប្រើ PHP cURL ដើម្បីផ្ញើ HTTP Request ទៅ API ផ្សេង។
- កំណត់ Connection Timeout និង Request Timeout។
- ពិនិត្យ cURL Error, HTTP Status និង JSON Response ដាច់ពីគ្នា។
- រក្សា TLS Certificate Verification ឱ្យបើក។
- កាត់បន្ថយហានិភ័យ SSRF និងការលេចធ្លាយ API Credential។

> តម្រូវការ៖ PHP 8+ និង Extension `curl` ត្រូវបានបើក។ ឧទាហរណ៍ប្រើ GET និង JSON API។

## ១. cURL និង External API

cURL ជា PHP Extension សម្រាប់ទាក់ទងជាមួយ URL/HTTP។ ពេលហៅ API ខាងក្រៅ កម្មវិធីត្រូវគិតពី៖

- DNS/Connection អាចយឺត ឬបរាជ័យ។
- API អាចឆ្លើយ HTTP Error ដូចជា 401, 429 ឬ 500។
- Response Body អាចមិនមែនជា JSON ត្រឹមត្រូវ។
- Certificate TLS ត្រូវតែផ្ទៀងផ្ទាត់។
- URL ខាងក្រៅអាចបង្កហានិភ័យ SSRF ប្រសិនបើ User អាចកំណត់បាន។

## ២. GET Request ដោយ Timeout និង TLS Verification

```php
<?php
function getJson(string $url): array
{
    $handle = curl_init($url);

    if ($handle === false) {
        throw new RuntimeException('មិនអាចបង្កើត HTTP Client បានទេ។');
    }

    curl_setopt_array($handle, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_CONNECTTIMEOUT => 3,
        CURLOPT_TIMEOUT => 10,
        CURLOPT_FOLLOWLOCATION => false,
        CURLOPT_SSL_VERIFYPEER => true,
        CURLOPT_SSL_VERIFYHOST => 2,
        CURLOPT_HTTPHEADER => [
            'Accept: application/json',
        ],
    ]);

    try {
        $body = curl_exec($handle);

        if ($body === false) {
            $error = curl_error($handle);
            throw new RuntimeException('External API Request failed: ' . $error);
        }

        $status = (int) curl_getinfo($handle, CURLINFO_HTTP_CODE);
        $contentType = curl_getinfo($handle, CURLINFO_CONTENT_TYPE);

        if ($status < 200 || $status >= 300) {
            throw new RuntimeException('External API returned HTTP ' . $status);
        }

        if (!is_string($contentType) || !str_contains($contentType, 'application/json')) {
            throw new RuntimeException('External API returned an unexpected content type.');
        }

        $decoded = json_decode($body, true, 512, JSON_THROW_ON_ERROR);

        if (!is_array($decoded)) {
            throw new RuntimeException('External API JSON response has an unexpected shape.');
        }

        return $decoded;
    } finally {
        curl_close($handle);
    }
}
```

បើ Request បរាជ័យ សូម Log Error នៅ Server តាម Policy ដោយកុំបង្ហាញ URL មាន Credential ឬ Response រសើបទៅ User។ កុំបិទ `CURLOPT_SSL_VERIFYPEER` ឬ `CURLOPT_SSL_VERIFYHOST` ដើម្បីឱ្យ Request “ដំណើរការ”។

## ៣. ពិនិត្យ HTTP Status ខុសពី cURL Error

cURL Error មានន័យថាមានបញ្ហាកម្រិត Transport/Client ដូចជា Timeout, DNS ឬ TLS។ HTTP Status ដូចជា 404/500 មានន័យថា Server បានឆ្លើយតបតាម HTTP ប៉ុន្តែ Result អាចជាកំហុសតាម API Contract។

ប្រើ Status Code តាម API Documentation៖

- 2xx៖ Request ជោគជ័យតាមកម្រិត HTTP។
- 3xx៖ Redirect; កំណត់ថាតើអាចអនុញ្ញាត Redirect ឬត្រូវបដិសេធ។
- 4xx៖ Request/Authentication/Rate Limit បញ្ហា។
- 5xx៖ API ខាងក្រៅជួបបញ្ហា។

កុំ Retry គ្រប់ Request ដោយស្វ័យប្រវត្តិ។ Retry តែ Error ដែលសមស្រប, កំណត់ចំនួន/Backoff និងធានាថា Operation មាន Idempotency ប្រសិនបើអាចបង្កើត Side Effect។

## ៤. POST JSON និង API Key

កំណត់ API Base URL និង Credential ពី Configuration ដែលទុកចិត្តបាន។ កុំទទួល URL ពី Query String/Form មក Request ដោយផ្ទាល់៖

```php
<?php
$baseUrl = 'https://api.example.com'; // Configuration ដែលគ្រប់គ្រងដោយ Server
$endpoint = $baseUrl . '/v1/orders';

$payload = json_encode(
    ['product_id' => 123, 'quantity' => 2],
    JSON_THROW_ON_ERROR
);

$apiKey = getenv('PARTNER_API_KEY');
if (!is_string($apiKey) || $apiKey === '') {
    throw new RuntimeException('API credential is not configured.');
}

$handle = curl_init($endpoint);

curl_setopt_array($handle, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => $payload,
    CURLOPT_CONNECTTIMEOUT => 3,
    CURLOPT_TIMEOUT => 10,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_SSL_VERIFYPEER => true,
    CURLOPT_SSL_VERIFYHOST => 2,
    CURLOPT_HTTPHEADER => [
        'Accept: application/json',
        'Content-Type: application/json',
        'Authorization: Bearer ' . $apiKey,
    ],
]);

try {
    $body = curl_exec($handle);
    if ($body === false) {
        throw new RuntimeException('Partner API transport error.');
    }

    $status = (int) curl_getinfo($handle, CURLINFO_HTTP_CODE);
    // ត្រូវពិនិត្យ $status និង Parse Response តាម API Contract។
} finally {
    curl_close($handle);
}
```

កុំដាក់ API Key ក្នុង URL Query String ឬ Log Request Header។ សម្រាប់ Error Handling ពិត គួរប្រើ Client Class ដែលរួមបញ្ចូលការត្រួតពិនិត្យ Response ដូច GET ឧទាហរណ៍ខាងលើ។

## ៥. ការពារ SSRF

Server-Side Request Forgery (SSRF) អាចកើតឡើងពេល Server ហៅ URL ដែលអ្នកប្រើកំណត់ ហើយអាចចូលទៅ Internal Service/Metadata Endpoint។

- ប្រើ Hostname/API Base URL ដែលបាន Allowlist ក្នុង Configuration។
- កុំយក URL ពី User ទៅ cURL ដោយផ្ទាល់។
- បិទ Redirect ឬត្រួតពិនិត្យ Redirect Target រាល់ដំណាក់កាល។
- កំណត់ Egress Network Policy ដើម្បីកុំឱ្យ App ចូល Internal Network មិនចាំបាច់។
- Validate Scheme/Host/Port និងទប់ IP Private/Loopback ប្រសិនបើ Use Case ត្រូវឱ្យ User ផ្តល់ URL។
- កុំទុកចិត្តតែ DNS Check មួយដង ប្រសិនបើអាចមាន DNS Rebinding/Redirect។

## ៦. Response Size និង Logging

External Service អាចផ្ញើ Response ធំពេក។ កំណត់ Max Response Size នៅ Client/Proxy និងកំណត់ Timeout; សម្រាប់ Response Stream ធំ ប្រើ Write Callback ដែលរាប់ Bytes ហើយបញ្ឈប់ពេលលើស Limit។

Log ព័ត៌មានដែលជួយ Debug ដូចជា Provider Name, HTTP Status, Duration និង Request ID។ កុំ Log API Key, Authorization Header, Full Payload ឬទិន្នន័យផ្ទាល់ខ្លួនដោយមិនចាំបាច់។

## ៧. កំហុសដែលជួបញឹកញាប់

- កំណត់ Timeout គ្មាន ធ្វើឱ្យ PHP Worker ជាប់រង់ចាំយូរ។
- បិទ TLS Certificate Verification។
- គិតថា HTTP 500 ជា cURL Transport Error ឬពិនិត្យតែ `curl_exec()` ប៉ុណ្ណោះ។
- Follow Redirect ដោយស្វ័យប្រវត្តិទៅ Host មិនទុកចិត្ត។
- ប្រើ URL ពី User ដែលបង្ក SSRF។
- Log Bearer Token ឬដាក់ API Key ក្នុង Query String។
- Retry POST ដែលបង្កើត Order ដោយគ្មាន Idempotency Key។

## ៨. លំហាត់

1. សរសេរ GET Client ដែលកំណត់ Connect Timeout និង Total Timeout។
2. ពិនិត្យ cURL Error, HTTP Status និង JSON Decode Error ដាច់ដោយឡែក។
3. បន្ថែម API Key ពី Environment Variable ទៅ Authorization Header។
4. រចនា Retry Policy សម្រាប់ 429/503 ដោយ Backoff និង Retry Limit។
5. វាយតម្លៃ SSRF Risk ប្រសិនបើ Feature ត្រូវទាញយក URL ពី User។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ `CURLOPT_CONNECTTIMEOUT` និង `CURLOPT_TIMEOUT`។
- រក្សា TLS Verification បើក ហើយពិនិត្យ HTTP Status បន្ថែមពី cURL Error។
- Parse JSON ដោយ Error Handling ដូចជា `JSON_THROW_ON_ERROR`។
- API Key មកពី Secret Store/Environment និងផ្ញើតាម Header; កុំ Log វា។
- Allowlist Host និងគ្រប់គ្រង Redirect ដើម្បីកាត់បន្ថយ SSRF។

## ឯកសារយោង

- [PHP cURL Functions](https://www.php.net/manual/en/book.curl.php)
- [PHP: curl_setopt()](https://www.php.net/manual/en/function.curl-setopt.php)
- [PHP: curl_getinfo()](https://www.php.net/manual/en/function.curl-getinfo.php)
- [មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP](lesson-26-php-json-rest-api.md)
- [មេរៀនទី ៤៩៖ សុវត្ថិភាព Dependencies និង Composer Audit](lesson-49-composer-dependency-security-audit.md)
