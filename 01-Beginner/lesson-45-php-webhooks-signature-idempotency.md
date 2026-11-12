# មេរៀនទី ៤៥៖ Webhooks និងការផ្ទៀងផ្ទាត់ Signature ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Webhook និងរបៀប Provider ផ្ញើ Event មក Server របស់យើង។
- ផ្ទៀងផ្ទាត់ HMAC Signature លើ Raw Request Body។
- ប្រើ `hash_equals()` ដើម្បីប្រៀបធៀប Signature។
- ការពារ Replay Attack និងដំណើរការ Event ដដែលៗ (Idempotency)។
- បញ្ចូល Event ទៅ Queue និងឆ្លើយតបដោយមិនពន្យារពេលយូរ។

> Signature Header និង Algorithm ខុសគ្នាតាម Provider។ ត្រូវអនុវត្តតាម Documentation ផ្លូវការរបស់ Provider; ឧទាហរណ៍ខាងក្រោមប្រើទម្រង់គំរូ `sha256=<hex>`។

## ១. Webhook ជាអ្វី?

Webhook គឺ HTTP Request ដែលសេវាខាងក្រៅផ្ញើទៅ Endpoint របស់កម្មវិធីយើង ពេលមាន Event ដូចជា Payment Completed ឬ Subscription Changed។

យើងមិនគួរទុកចិត្ត Request ដោយសារតែវាបានមកដល់ Endpoint នោះទេ។ អ្នកវាយប្រហារក៏អាចហៅ URL ដោយផ្ទាល់ដែរ។ ត្រូវផ្ទៀងផ្ទាត់ថា Payload មកពី Provider ដែលទុកចិត្ត និងមិនត្រូវបានកែប្រែ។

## ២. ទទួល Raw Body និងផ្ទៀងផ្ទាត់ HMAC

Signature ជាទូទៅត្រូវគណនាលើ Raw Body ដូចដែលបានទទួល។ កុំ Decode JSON ហើយ Encode ឡើងវិញមុន Verify ព្រោះ Whitespace/Ordering អាចផ្លាស់ប្តូរ Byte ទាំងនោះ។

```php
<?php
$secret = getenv('WEBHOOK_SIGNING_SECRET');
$signatureHeader = $_SERVER['HTTP_X_PROVIDER_SIGNATURE'] ?? null;
$rawBody = file_get_contents('php://input');

if (
    !is_string($secret)
    || $secret === ''
    || !is_string($signatureHeader)
    || !is_string($rawBody)
    || strlen($rawBody) > 1_000_000
) {
    http_response_code(400);
    exit;
}

// គំរូទម្រង់: sha256=<64 hex characters>.
// កែឈ្មោះ Header និងទម្រង់ឱ្យត្រូវនឹង Provider។
if (!preg_match('/^sha256=([a-f0-9]{64})$/i', $signatureHeader, $matches)) {
    http_response_code(401);
    exit;
}

$providedSignature = strtolower($matches[1]);
$expectedSignature = hash_hmac('sha256', $rawBody, $secret);

if (!hash_equals($expectedSignature, $providedSignature)) {
    http_response_code(401);
    exit;
}
```

Secret ត្រូវទទួលពី Secret Store/Environment Configuration និងមិនដាក់ក្នុង Repository ឬ Log។ កំណត់ Request Body Size នៅ Web Server ផងដែរ; Check នៅ PHP គឺស្រទាប់បន្ថែម។

## ៣. Timestamp និង Replay Protection

Signature ត្រឹមត្រូវអាចត្រូវបានចម្លង ហើយផ្ញើម្ដងទៀត (Replay)។ បើ Provider រួមបញ្ចូល Timestamp ក្នុង Signature៖

1. Parse Timestamp ពី Header តាម Format របស់ Provider។
2. ពិនិត្យថាវាស្ថិតក្នុង Window សមស្រប (ឧ. ±5 នាទី)។
3. គណនា HMAC លើ String/Base Message ដែល Provider កំណត់ ដូចជា Timestamp + Raw Body។
4. បន្ទាប់ពី Verify ត្រូវរក្សា Event ID ដែលបានទទួលរួច និងបដិសេធ Duplicate។

កុំគិតថា Timestamp Check ជំនួស Idempotency បាន; Request ដដែលអាចមកម្ដងទៀតក្នុង Window ដែលនៅត្រឹមត្រូវ។ ប្រសិនបើ Provider មិនមាន Timestamp, ប្រើ Event ID និងការការពារ Duplicate តាម Contract ដែលមាន។

## ៤. Idempotency និង Database

Webhook Provider អាច Retry Event ពេលមិនទទួល Response ឬមាន Network Timeout។ ដំណើរការ Event ដដែលៗមិនគួរបង្កើតការទូទាត់ ឬ Update ម្តងហើយម្តងទៀត។ រក្សា Provider Event ID ជា Unique Key៖

```sql
CREATE TABLE processed_webhook_events (
    provider_event_id VARCHAR(191) PRIMARY KEY,
    event_type VARCHAR(100) NOT NULL,
    processed_at DATETIME NOT NULL
);
```

ប្រើ Transaction ដើម្បីបញ្ចូល Event ID និងអនុវត្ត Database Changes ជាអាតូមិក។ បើ Unique Constraint បង្ហាញថា Event បានដំណើរការរួច ឆ្លើយតបជោគជ័យតាម Provider Contract ដើម្បីបញ្ឈប់ Retry។ កុំប្រើ Payload Hash ជំនួស Event ID ប្រសិនបើ Provider មាន Event ID ស្ថិរភាព។

## ៥. ឆ្លើយតបលឿន និងដំណើរការតាម Queue

សម្រាប់ការងារធ្ងន់៖

1. អាន និង Verify Signature។
2. Validate JSON Schema/Required Fields។
3. រក្សា Event ID និង Payload ដែលត្រូវការ ទៅ Database ឬ Queue ដែលបានការពារ។
4. ឆ្លើយ 2xx ទៅ Provider តាមពេលកំណត់របស់វា។
5. Worker ដំណើរការ Event បន្ទាប់ដោយ Idempotent Logic។
6. Log Event ID/Result ដែលចាំបាច់ ដោយមិន Log Secret ឬ Payload ទាំងមូលដោយមិនចាំបាច់។

កុំឆ្លើយ 2xx មុនពេលបានរក្សា Event ឱ្យមានសុវត្ថិភាព ប្រសិនបើបន្ទាប់មក Server អាចបាត់វា។ កំណត់ Retry/Dead Letter Handling នៅ Queue។

## ៦. ត្រួតពិនិត្យ Payload បន្ទាប់ពី Verify

Signature បញ្ជាក់ភាពត្រឹមត្រូវនៃប្រភព/មាតិកា តាម Key ដែលចែករំលែក; វាមិនបញ្ជាក់ថា Event គ្រប់យ៉ាងត្រូវបានអនុញ្ញាតសម្រាប់ Business Logic ទេ។

- Decode JSON ដោយ `JSON_THROW_ON_ERROR`។
- ពិនិត្យ Event Type និង Schema តាម Allowlist។
- Validate ID, Amount, Currency និង State Transition នៅ Server។
- កុំទុកចិត្ត Amount/Status ពី Webhook ដោយមិនប្រៀបធៀបនឹង Record/Provider API តាមតម្រូវការ។
- កុំប្រើ Webhook ជំនួសការពិនិត្យ Authorization សម្រាប់ User Request ផ្សេង។

## ៧. កំហុសដែលជួបញឹកញាប់

- Verify លើ JSON ដែល Decode/Encode ឡើងវិញ ជំនួស Raw Body។
- ប្រើ `===` ជំនួស `hash_equals()` សម្រាប់ Signature។
- មិនពិនិត្យ Body Size ឬ Content Type។
- Log Signing Secret, Signature ឬ Payment Payload ពេញ។
- មិនទប់ Replay/Duplicate Event។
- អនុវត្ត Business Logic មុនផ្ទៀងផ្ទាត់ Signature។
- ទុកឱ្យ Endpoint ចូលបានសាធារណៈដោយគ្មាន Verify ព្រោះគិតថា URL ពិបាកទាយ។

## ៨. លំហាត់

1. បង្កើត Test ដោយ Raw Body និង HMAC Secret សិប្បនិម្មិត។
2. បដិសេធ Signature ខុស និង Payload ធំពេក។
3. បន្ថែម Event ID ជា Unique Key ហើយសាកល្បង Duplicate Event។
4. រចនា Timestamp Verification តាម Signature Format របស់ Provider ដែលអ្នកជ្រើស។
5. បញ្ជូន Event ទៅ Queue បន្ទាប់ពី Verify ហើយឆ្លើយ 2xx តែពេលរក្សា Event បានជោគជ័យ។

## ៩. ចម្លើយណែនាំខ្លី

- គណនា HMAC លើ Raw Body/Message ដែល Provider កំណត់ ហើយប្រៀបធៀបដោយ `hash_equals($expected, $provided)`។
- Timestamp Window ជួយកាត់បន្ថយ Replay; Event ID Unique Constraint ជួយការពារ Duplicate។
- ធ្វើ Signature Verification មុន Parse/អនុវត្ត Business Logic។
- Validate Schema និង State នៅ Server បន្ទាប់ពី Verify។
- រក្សា Event មុនឆ្លើយ 2xx និងដំណើរការការងារធ្ងន់តាម Queue។

## ឯកសារយោង

- [PHP: hash_equals()](https://www.php.net/manual/en/function.hash-equals.php)
- [PHP: hash_hmac()](https://www.php.net/manual/en/function.hash-hmac.php)
- [GitHub Docs: Validating Webhook Deliveries](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries)
- [មេរៀនទី ២៩៖ CSRF, XSS និង SQL Injection](lesson-29-php-csrf-xss-sql-injection.md)
