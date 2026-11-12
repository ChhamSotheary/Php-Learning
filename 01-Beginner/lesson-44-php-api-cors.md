# មេរៀនទី ៤៤៖ CORS សម្រាប់ PHP API

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ថា CORS ជាគោលការណ៍របស់ Browser សម្រាប់ Cross-Origin Request។
- កំណត់ Origin Allowlist នៅ Server។
- ឆ្លើយតប Preflight `OPTIONS` ដោយ Headers ត្រឹមត្រូវ។
- កំណត់ Credentialed Request ដោយមិនប្រើ Wildcard Origin។
- យល់ថា CORS មិនមែនជា Authentication, Authorization ឬ CSRF Protection ទេ។

## ១. CORS ជាអ្វី?

Origin គឺគូរួមនៃ Scheme, Host និង Port។ ឧទាហរណ៍ `https://app.example.com` និង `https://api.example.com` មាន Origin ខុសគ្នា។ Browser ប្រើ Cross-Origin Resource Sharing (CORS) ដើម្បីសម្រេចថា JavaScript នៅ Origin មួយអាចអាន Response ពី Server មួយទៀតបានឬអត់។

CORS គ្រប់គ្រងការចូលប្រើ Response ដោយ Browser។ វាមិនរារាំង Client ដូចជា cURL ឬ Server ផ្សេងពីការហៅ API ទេ។ ដូច្នេះ API នៅតែត្រូវ Authenticate និង Authorize រាល់ Request។

## ២. កំណត់ Origin Allowlist

កុំ Echo Origin ណាមួយដែល Client ផ្ញើមក។ ប្រៀបធៀបជាមួយ Allowlist ពេញលេញ៖

```php
<?php
$allowedOrigins = [
    'https://app.example.com',
    'https://admin.example.com',
];

$origin = $_SERVER['HTTP_ORIGIN'] ?? null;

if (is_string($origin) && in_array($origin, $allowedOrigins, true)) {
    header('Access-Control-Allow-Origin: ' . $origin);
    header('Vary: Origin');
}
```

កុំប្រើការត្រួតពិនិត្យបែប `str_contains($origin, 'example.com')` ព្រោះ Origin ក្លែងអាចមាន String នោះនៅក្នុង Host ផ្សេង។ ត្រូវ Allowlist Scheme/Host/Port ជាក់លាក់ និងគិតពី Development Origin ដាច់ដោយឡែក។

បើ Endpoint ជាសាធារណៈ និងមិនប្រើ Cookie/HTTP Credential អាចប្រើ `Access-Control-Allow-Origin: *` តាមការសម្រេចច្បាស់លាស់។ កុំប្រើ Wildcard លើទិន្នន័យដែលមានសិទ្ធិ។

## ៣. Preflight Request

Browser អាចផ្ញើ Preflight `OPTIONS` មុន Request ពិត ដើម្បីសួរថា Method និង Header ត្រូវបានអនុញ្ញាតឬទេ។ ឧទាហរណ៍ Middleware សាមញ្ញ៖

```php
<?php
$allowedOrigins = ['https://app.example.com'];
$allowedMethods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'];
$allowedHeaders = ['Content-Type', 'Authorization', 'X-Request-ID'];

$origin = $_SERVER['HTTP_ORIGIN'] ?? null;
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if (is_string($origin) && in_array($origin, $allowedOrigins, true)) {
    header('Access-Control-Allow-Origin: ' . $origin);
    header('Vary: Origin');
    header('Access-Control-Allow-Methods: ' . implode(', ', $allowedMethods));
    header('Access-Control-Allow-Headers: ' . implode(', ', $allowedHeaders));
    header('Access-Control-Max-Age: 600');
}

if ($method === 'OPTIONS') {
    if (!is_string($origin) || !in_array($origin, $allowedOrigins, true)) {
        http_response_code(403);
        exit;
    }

    http_response_code(204);
    exit;
}
```

ក្នុងកម្មវិធីពិត ត្រូវពិនិត្យ `Access-Control-Request-Method` និង `Access-Control-Request-Headers` ក្នុង Preflight ហើយអនុញ្ញាតតែតម្លៃដែល Allowlist បាន។ កុំឆ្លើយថា Method/Header គ្រប់យ៉ាងអនុញ្ញាត បើ API មិនគាំទ្រវា។ Response Preflight គួរមិនមាន Body។

Preflight គឺជាសំណួររបស់ Browser; វាមិនមែនជា Request ដែលអាចប្រើជំនួស Authorization ឬ Business Logic បានទេ។ Route OPTIONS អាចត្រូវឆ្លើយមុន Authentication Middleware តាមការរៀបចំ Server ប៉ុន្តែ Request ពិតនៅតែត្រូវ Authenticate/Authorize។

## ៤. Cookies និង Credentialed Requests

បើ Browser ត្រូវផ្ញើ Cookie/credential តាម Cross-Origin Request៖

```php
header('Access-Control-Allow-Origin: https://app.example.com');
header('Access-Control-Allow-Credentials: true');
header('Vary: Origin');
```

មិនអាចប្រើ `Access-Control-Allow-Origin: *` ជាមួយ Credentialed Request បានទេ។ Client ក៏ត្រូវកំណត់ `credentials: 'include'` ឬជម្រើសសមស្រប ហើយ Cookie ត្រូវមាន `SameSite`/ `Secure` ត្រឹមត្រូវតាម Browser និង Domain Flow។

CORS ដែលប្រើ Cookie អាចមានផលប៉ះពាល់ជាមួយ CSRF។ ត្រូវប្រើ CSRF Token និង Cookie Policy តាមមេរៀនសុវត្ថិភាពមុនៗ; CORS មិនជំនួស CSRF Protection ទេ។

## ៥. CORS និង Response Errors

បើ App អនុញ្ញាត Origin ត្រឹមត្រូវ ត្រូវបន្ថែម CORS Headers លើ Response Error ដែល Client ត្រូវអានបានផងដែរ ដូចជា 401/403/422។ រៀបចំ Middleware ឱ្យដំណើរការមុន Error Handler ឬកំណត់ Headers នៅ Web Server/Framework ដើម្បីកុំឱ្យ Browser លាក់ Error Response ដែលមានប្រយោជន៍។

កុំឆ្លើយ CORS Header ទៅ Origin មិនបានអនុញ្ញាត។ Browser នឹងមិនអនុញ្ញាត JavaScript អាន Response នោះទេ ប៉ុន្តែ Request អាចបានដល់ Server រួចហើយ ដូច្នេះ Server-side Security នៅតែចាំបាច់។

## ៦. កំហុសដែលជួបញឹកញាប់

- អនុញ្ញាត Origin ដោយ Match Substring ឬ Regex ទូលំទូលាយ។
- ប្រើ `Access-Control-Allow-Origin: *` ជាមួយ Cookie។
- ច្រឡំ CORS ជាមួយ Authentication/Authorization។
- Allow Methods/Headers ទាំងអស់ដោយមិនចាំបាច់។
- មិនបន្ថែម `Vary: Origin` ពេល Response Header ផ្លាស់ប្តូរតាម Origin។
- អនុញ្ញាត Preflight ប៉ុន្តែភ្លេច CORS Header លើ Response ពិត ឬ Error។
- គិតថា CORS អាចការពារ API ពី cURL ឬ Server-to-Server Request។

## ៧. លំហាត់

1. រៀបចំ Origin Allowlist សម្រាប់ Frontend Local និង Production។
2. អនុញ្ញាតតែ GET/POST និង Header Content-Type/Authorization។
3. សរសេរ Preflight `OPTIONS` Response ជាមួយ Status 204។
4. បង្ហាញពីមូលហេតុដែល Wildcard មិនអាចប្រើជាមួយ Credentials។
5. ពន្យល់ពីការការពារផ្សេងទៀតដែល API ត្រូវការ បន្ថែមពី CORS។

## ៨. ចម្លើយណែនាំខ្លី

- Origin Allowlist ត្រូវផ្គូផ្គងពេញលេញ (Scheme, Host, Port)។
- Preflight ត្រូវពិនិត្យ Origin, Requested Method និង Headers។
- Credentialed CORS ត្រូវប្រើ Origin ជាក់លាក់ និង `Access-Control-Allow-Credentials: true`; មិនប្រើ Wildcard។
- CORS ជា Browser Policy; API នៅតែត្រូវ Authenticate, Authorize, Validate និងការពារ CSRF សម្រាប់ Cookie Flow។
- ប្រើ `Vary: Origin` នៅពេល CORS Response អាស្រ័យលើ Origin។

## ឯកសារយោង

- [MDN: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [MDN: CORS Credential Error](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS/Errors/CORSNotSupportingCredentials)
- [មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP](lesson-26-php-json-rest-api.md)
- [មេរៀនទី ២៩៖ CSRF, XSS និង SQL Injection](lesson-29-php-csrf-xss-sql-injection.md)
