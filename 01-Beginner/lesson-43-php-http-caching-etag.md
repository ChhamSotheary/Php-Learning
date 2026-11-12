# មេរៀនទី ៤៣៖ HTTP Caching និង ETag សម្រាប់ PHP API

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Browser Cache និង Shared Cache។
- កំណត់ `Cache-Control` តាមប្រភេទទិន្នន័យ។
- ប្រើ ETag និង `If-None-Match` ដើម្បីឆ្លើយតប `304 Not Modified`។
- ជៀសវាងការរក្សាទុក Response ផ្ទាល់ខ្លួននៅ Public Cache។
- ស្គាល់កំហុស Cache ដែលបង្ហាញទិន្នន័យអ្នកប្រើខុសគ្នា។

## ១. Cache ជួយអ្វីខ្លះ?

Cache រក្សាទុក Response បណ្តោះអាសន្ន ដើម្បីកាត់បន្ថយការផ្ញើទិន្នន័យដដែលៗ និងកាត់បន្ថយ Load លើ Server។ Cache អាចស្ថិតនៅ Browser, Proxy, CDN ឬ Server-side Application។

ចំណុចសំខាន់គឺជ្រើសរើស Policy ត្រឹមត្រូវ៖

- ទិន្នន័យសាធារណៈ និងផ្លាស់ប្តូរយឺត អាច Cache បាន។
- Response ដែលមានព័ត៌មានផ្ទាល់ខ្លួន ត្រូវការការកំណត់ `private` ឬ `no-store` តាមហានិភ័យ។
- Cache Key ត្រូវរួមបញ្ចូលអ្វីៗដែលផ្លាស់ប្តូរ Response; កុំឱ្យទិន្នន័យ User ម្នាក់ត្រូវបម្រើទៅ User ម្នាក់ទៀត។

## ២. Cache-Control Directives

ឧទាហរណ៍សម្រាប់ API សាធារណៈដែលអាច Cache 60 វិនាទី៖

```php
header('Cache-Control: public, max-age=60');
```

សម្រាប់ទិន្នន័យផ្ទាល់ខ្លួន ដែលអាចរក្សាទុកនៅ Browser ប៉ុន្តែមិនអាចចែករំលែកតាម Proxy Cache៖

```php
header('Cache-Control: private, max-age=30');
```

សម្រាប់ព័ត៌មានរសើបដែលមិនគួររក្សាទុកនៅ Cache៖

```php
header('Cache-Control: no-store');
```

`no-cache` មានន័យថា Cache ត្រូវផ្ទៀងផ្ទាត់ជាមួយ Server មុនប្រើ Response ដែលបានរក្សាទុក; វាមិនដូច `no-store` ទេ។ ជ្រើសរើស Directive តាមទិន្នន័យ និង Threat Model។ កុំប្រើ `public` លើ Response ដែលអាស្រ័យលើ Session/User។

## ៣. ETag និង Conditional GET

ETag ជា Validator ដែល Server ផ្ញើមកជាមួយ Representation។ Client អាចផ្ញើ `If-None-Match` នៅ Request បន្ទាប់។ បើ Representation មិនបានផ្លាស់ប្តូរ Server អាចត្រឡប់ `304 Not Modified` ដោយមិនផ្ញើ Body ម្តងទៀត។

ឧទាហរណ៍សម្រាប់ JSON ដែលមិនមាន Personal Data៖

```php
<?php
$body = json_encode(
    ['data' => $publicProducts],
    JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR
);

$etag = '"' . hash('sha256', $body) . '"';
header('ETag: ' . $etag);
header('Cache-Control: public, max-age=60');

$ifNoneMatch = $_SERVER['HTTP_IF_NONE_MATCH'] ?? '';

if (is_string($ifNoneMatch) && trim($ifNoneMatch) === $etag) {
    http_response_code(304);
    exit;
}

header('Content-Type: application/json; charset=utf-8');
echo $body;
```

នេះជាគំរូសាមញ្ញសម្រាប់ ETag តែមួយ។ HTTP អនុញ្ញាត ETag ជាច្រើន និង Weak ETag ផងដែរ; App ដែលត្រូវគាំទ្រទម្រង់ទាំងនោះ គួរប្រើ Framework/Library ឬ Parser ដែលស្របតាម HTTP។ កុំប្រៀបធៀប Header ដោយ String ស្មើធម្មតា ប្រសិនបើត្រូវគាំទ្រ ETag List, `W/` ឬ `*`។

សម្រាប់ `304` កុំផ្ញើ Response Body។ Cache Headers ដែលពាក់ព័ន្ធគួរត្រូវបានរក្សាឱ្យស្របនឹង Response `200`។

## ៤. Cache Response ដែលទាក់ទងនឹង User

ទិន្នន័យដូចជា Profile, Orders និង Private Messages អាចប្រែប្រួលតាម User។ មុន Cache ត្រូវសួរថា៖

- Response មានព័ត៌មានផ្ទាល់ខ្លួន ឬព័ត៌មានដែលត្រូវ Authenticate ឬទេ?
- Cache ស្ថិតនៅ Browser ឬ Shared Proxy/CDN?
- Cache Key រួមបញ្ចូល User/Authorization Context ដោយសុវត្ថិភាពឬទេ?
- ពេល User Sign Out ឬសិទ្ធិផ្លាស់ប្តូរ តើ Cache ត្រូវ Invalidate យ៉ាងដូចម្តេច?

បើមិនប្រាកដ សម្រាប់ Response រសើបប្រើ `Cache-Control: no-store` ជាជម្រើសចាប់ផ្តើម ហើយពិនិត្យ CDN/Reverse Proxy Configuration ផងដែរ។

## ៥. Cache Invalidation

ពេលទិន្នន័យផ្លាស់ប្តូរ Cache ចាស់អាចនៅសល់។ គោលការណ៍ដែលអាចប្រើ៖

- កំណត់ `max-age` ខ្លីសម្រាប់ទិន្នន័យដែលអាចចាស់បន្តិចបាន។
- បង្កើត ETag ថ្មីពី Representation ដែលបាន Update។
- Invalidate Cache Key ពាក់ព័ន្ធនៅ CDN/Application Cache។
- ប្រើ Versioned URL សម្រាប់ Static Asset ដែល Content មិនប្រែប្រួលក្រោម URL ដដែល។
- កុំឱ្យ Cache រក្សាទុក Error ឬ Response ដែលមិនបានកំណត់ Policy ច្បាស់។

ការដាក់ ETag តែមួយមុខមិនបានបញ្ជាក់ថាទិន្នន័យ Fresh គ្មានកំណត់ទេ; Client និង Cache ត្រូវគោរព Freshness Policy ផងដែរ។

## ៦. ការសាកល្បង

ប្រើ Browser Network Panel ឬ `curl` ដើម្បីពិនិត្យ៖

1. Request ដំបូងត្រឡប់ `200`, Body, ETag និង Cache-Control។
2. Request បន្ទាប់ផ្ញើ `If-None-Match` ដែលបានទទួល។
3. បើ Content មិនផ្លាស់ប្តូរ ទទួល `304` គ្មាន Body។
4. ក្រោយ Update Resource, ETag ផ្លាស់ប្តូរ ហើយ Server ត្រឡប់ `200` ជាមួយ Body ថ្មី។
5. ពិនិត្យថា Response ផ្ទាល់ខ្លួនមិនបាន Cache ជា Public។

ឧទាហរណ៍៖

```bash
curl -i https://api.example.com/api/v1/products
curl -i -H 'If-None-Match: "etag-from-first-response"' https://api.example.com/api/v1/products
```

## ៧. កំហុសដែលជួបញឹកញាប់

- ប្រើ `Cache-Control: public` លើ User-Specific Response។
- គិតថា `no-cache` និង `no-store` មានន័យដូចគ្នា។
- បង្កើត ETag ពីទិន្នន័យមិនទាន់បាន Serialize ដូច្នេះ Validator មិនតំណាងឱ្យ Response ពិត។
- ឆ្លើយតប `304` ទាំងដែល Content បានផ្លាស់ប្តូរ។
- មិនពិនិត្យ CDN/Proxy Cache នៅខាងមុខ PHP។
- Cache Authorization Response ដោយមិនមាន Cache Key/Invalidation ត្រឹមត្រូវ។

## ៨. លំហាត់

1. កំណត់ Cache Policy សម្រាប់ Public Product Catalog និង Private User Profile។
2. បន្ថែម ETag ទៅ Public API Response និងឆ្លើយ `304` ពេល Content ដដែល។
3. សាកល្បង Conditional GET ដោយ `curl`។
4. ពិភាក្សាថា Cache Key ត្រូវផ្លាស់ប្តូរតាម User ឬ Locale យ៉ាងណា។
5. រៀបចំផែនការ Invalidation ពេល Admin Update Product។

## ៩. ចម្លើយណែនាំខ្លី

- Public Static/Slow-Changing Data អាចប្រើ `public, max-age=...` តាមតម្រូវការ។
- User-Specific Data ត្រូវជា Private ឬ No-Store តាមភាពរសើប និង Cache Topology។
- ETag/If-None-Match អាចកាត់បន្ថយការផ្ញើ Body ដដែលៗ; `304` មិនមាន Body។
- `no-cache` ទាមទារ Revalidation; `no-store` ប្រាប់ Cache កុំរក្សាទុក Response។
- ត្រូវផ្ទៀងផ្ទាត់ Cache Behavior នៅ Browser, CDN និង Reverse Proxy។

## ឯកសារយោង

- [RFC 9111: HTTP Caching](https://www.rfc-editor.org/rfc/rfc9111.html)
- [MDN: 304 Not Modified](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/304)
- [MDN: ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/ETag)
- [មេរៀនទី ៤១៖ Pagination, Filtering និង Sorting សម្រាប់ API](lesson-41-php-api-pagination-filtering-sorting.md)
