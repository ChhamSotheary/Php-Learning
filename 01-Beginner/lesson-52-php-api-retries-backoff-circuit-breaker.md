# មេរៀនទី ៥២៖ Retry, Backoff និង Circuit Breaker សម្រាប់ PHP API Client

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែកកំហុសបណ្ដោះអាសន្ន និងកំហុសដែលមិនគួរ Retry។
- Retry Request ដោយកំណត់ចំនួន និងពេលរង់ចាំ។
- ប្រើ Exponential Backoff ជាមួយ Jitter និងគោរព `Retry-After`។
- ជៀសវាង Retry Operation ដែលអាចបង្កើត Side Effect ស្ទួន។
- ពន្យល់ពី Circuit Breaker និងរបៀបប្រើវាក្នុងប្រព័ន្ធដែលហៅសេវាច្រើន។

## ១. ហេតុអ្វីមិនគួរ Retry គ្រប់ Error?

External API អាច Timeout ឬឆ្លើយ 503 ជាបណ្ដោះអាសន្ន។ Retry អាចជួយឱ្យ Request ជោគជ័យពេលក្រោយ ប៉ុន្តែ Retry ច្រើនពេកអាចធ្វើឱ្យសេវាខូចកាន់តែធ្ងន់ និងបង្កើន Load។

កុំ Retry ដោយស្វ័យប្រវត្តិចំពោះ៖

- 400/401/403/404 ដែលជាកំហុស Request/Permission ទូទៅ។
- Validation Error ឬទិន្នន័យខុស។
- Operation ដែលមិនប្រាកដថាអាចធ្វើម្ដងទៀតដោយមិនបង្ក Side Effect។
- Request ដែលអស់ Deadline/ពេលវេលារបស់ Caller រួច។

ករណីដែលអាច Retry តាម Policy៖ Network Error បណ្ដោះអាសន្ន, 429 និង 502/503/504។ ស្ថានភាព 500 ត្រូវសម្រេចតាម Contract របស់ API និងប្រភេទ Operation។

## ២. Idempotency និងសុវត្ថិភាព Retry

HTTP Method ខ្លះដូចជា GET/PUT/DELETE មាន Semantics Idempotent តាម HTTP Specification ប៉ុន្តែ App/API ត្រូវអនុវត្តឱ្យស្របនឹងន័យនោះ។ POST ជាទូទៅមិនអាចសន្មតថា Retry មានសុវត្ថិភាពទេ។

សម្រាប់ការបង្កើត Payment/Order៖

- ប្រើ Idempotency Key ដែល Client បង្កើតដោយ Random Value។
- Provider ត្រូវរក្សា Key និងលទ្ធផល ដើម្បីការពារការបង្កើត Operation ស្ទួន។
- ប្រើ Key ដដែលសម្រាប់ Retry ដដែល; កុំបង្កើត Key ថ្មីរាល់ Attempt។
- កុំចាត់ Timeout ថា Operation មិនបានកើតឡើងទេ; Server អាចបានធ្វើរួច តែ Response បាត់។

Retry Non-idempotent Request តែពេល API មាន Idempotency Mechanism ឬមានវិធីពិនិត្យថា Request ដើមមិនបានអនុវត្ត។

## ៣. Exponential Backoff និង Jitter

Backoff បង្កើនពេលរង់ចាំរាល់ Attempt។ Jitter បន្ថែមភាពចៃដន្យ ដើម្បីកុំឱ្យ Client ច្រើន Retry ពេលតែមួយ៖

```php
<?php
function retryDelayMs(int $attempt, int $baseMs = 200, int $capMs = 5000): int
{
    $exponent = min(max($attempt - 1, 0), 10);
    $upperBound = min($capMs, $baseMs * (2 ** $exponent));

    // Full jitter: រើសពេលរង់ចាំពី 0 ដល់ Upper Bound។
    return random_int(0, $upperBound);
}
```

កំណត់ Attempt Count តូច (ឧ. 3 ដង) និង Total Deadline។ Retry គួរប្រើពេលវេលាចំណាយសរុបតិចជាង Timeout របស់ Request ខាងលើ។ កុំឱ្យ Worker/Web Request ដេករង់ចាំយូរដោយគ្មានកំណត់។

## ៤. គោរព Retry-After

HTTP 429 និងសេវាខ្លះអាចផ្ញើ `Retry-After` ជាវិនាទី ឬ HTTP Date។ បើមាន ត្រូវគោរពតាម Provider Policy ហើយកុំ Retry មុនពេលដែល Server បានណែនាំ។ ត្រូវកំណត់អតិបរមារង់ចាំ ដើម្បីកុំឱ្យ Worker ជាប់យូរ; បើលើស Deadline សូមដាក់ Job ទៅ Queue សម្រាប់ពេលក្រោយ។

```php
<?php
$retryAfter = $responseHeaders['retry-after'] ?? null;

// Parse វិនាទី ឬ HTTP-Date ដោយអនុវត្ត Header Parser ដែលត្រឹមត្រូវ។
// បន្ទាប់មកកំណត់ Delay មិនឱ្យលើស Retry Policy/Deadline របស់កម្មវិធី។
```

Header អាចមិនមាន ឬមិនត្រឹមត្រូវ។ ប្រើ Backoff Policy របស់ Client ជាជម្រើស ហើយកត់ត្រា Retry Attempt ដោយមិន Log Secret។

## ៥. Retry Loop សម្រាប់ Operation ដែលអាច Retry បាន

គំរូ Pseudocode៖

```php
<?php
$maxAttempts = 3;

for ($attempt = 1; $attempt <= $maxAttempts; $attempt++) {
    try {
        $response = callPartnerApi();

        if (isSuccessful($response)) {
            return $response;
        }

        if (!isTransientStatus($response->status)) {
            throw new RuntimeException('Partner API rejected the request.');
        }
    } catch (TransientNetworkException $exception) {
        if ($attempt === $maxAttempts) {
            throw $exception;
        }
    }

    if ($attempt < $maxAttempts) {
        usleep(retryDelayMs($attempt) * 1000);
    }
}

throw new RuntimeException('Partner API retry limit reached.');
```

នេះជា Pseudocode មិនមែន Class ដែលអាចដំណើរការភ្លាមទេ៖ ត្រូវកំណត់ការចាប់ Network Error និង Status Code តាម Client Library និង API Contract។ កុំ Catch Error គ្រប់ប្រភេទហើយ Retry; ការកំណត់ Retry ត្រូវមាន Allowlist Error ដែលច្បាស់។

សម្រាប់ Web Request ដែលអ្នកប្រើកំពុងរង់ចាំ គួររក្សា Retry តិច និង Timeout សរុបខ្លី។ សម្រាប់ការងារមិនបន្ទាន់ ប្រើ Queue/Background Worker ដើម្បី Retry ដោយមាន Delay។

## ៦. Circuit Breaker

Circuit Breaker ជួយបញ្ឈប់ការហៅទៅ Dependency ដែលបរាជ័យជាបន្តបន្ទាប់៖

- **Closed**៖ ហៅសេវាធម្មតា និងរាប់ Failure។
- **Open**៖ បរាជ័យច្រើនលើស Threshold ក្នុង Window; ឈប់ហៅមួយរយៈ ហើយត្រឡប់ Fallback/Error ភ្លាម។
- **Half-open**៖ ក្រោយ Cooldown អនុញ្ញាត Probe Request តិចតួច; បើជោគជ័យបិទ Circuit, បើបរាជ័យបើកវិញ។

ប្រើ Circuit Breaker ជាមួយ Timeout, Retry Budget និង Monitoring។ Circuit Breaker មិនជំនួស Retry ឬ Error Handling ទេ; វាការពារកុំឱ្យការហៅបន្តបង្ក Load លើ Dependency ដែលកំពុងមានបញ្ហា។

## ៧. ត្រួតពិនិត្យ Retry

កត់ Metrics ដូចជា៖

- ចំនួន Request និង Error តាម Provider/Endpoint។
- Retry Attempt Count និង Retry Exhausted Count។
- Latency រួមទាំងពេលវេលា Retry។
- Circuit State និងចំនួន Fallback។

កុំ Log API Key, Request Authorization Header, Payment Data ឬ Response Body ទាំងមូល។ ដាក់ Request/Correlation ID ដើម្បីតាមដាន Call រវាងសេវា។

## ៨. កំហុសដែលជួបញឹកញាប់

- Retry POST/Payment ដោយគ្មាន Idempotency Key។
- Retry លើ Validation/Authentication Error ដែលមិនអាចជួសជុលដោយរង់ចាំ។
- Retry គ្មានចំនួនកំណត់ ឬគ្មាន Total Deadline។
- Client ទាំងអស់ Retry ក្នុង Delay ដូចគ្នា ដោយគ្មាន Jitter។
- មិនគោរព `Retry-After` ហើយធ្វើឱ្យ Rate Limit កាន់តែខ្លាំង។
- បង្កើន Timeout/Retry រួមគ្នារហូតធ្វើឱ្យ Request ខាងលើរង់ចាំលើស Deadline។
- គិតថា Timeout មានន័យថា Operation នៅ Server មិនបានអនុវត្ត។

## ៩. លំហាត់

1. កំណត់ថា Error ប្រភេទណាអាច Retry សម្រាប់ GET Product API។
2. រៀបចំ Backoff មាន Jitter, Attempt Limit និង Total Deadline។
3. រចនា Payment Request ដែលប្រើ Idempotency Key ដដែលសម្រាប់ Retry។
4. បន្ថែម Retry-After Handling សម្រាប់ HTTP 429។
5. គូររដ្ឋ Circuit Breaker និងកំណត់ Threshold/Cooldown។

## ១០. ចម្លើយណែនាំខ្លី

- Retry តែ Error បណ្ដោះអាសន្នដែលបានកំណត់ច្បាស់ និងមាន Attempt/Deadline Limit។
- ប្រើ Exponential Backoff ជាមួយ Jitter និងគោរព `Retry-After`។
- កុំ Retry Non-idempotent Request បើគ្មាន Idempotency Key/វិធីដឹងថា Operation មិនបានកើតឡើង។
- Circuit Breaker បើកពេល Dependency បរាជ័យជាប់ៗគ្នា ហើយសាកល្បងវិញក្រោយ Cooldown។
- Timeout មិនបញ្ជាក់ថា Server មិនបានអនុវត្ត Operation នោះទេ។

## ឯកសារយោង

- [RFC 9110: HTTP Semantics — Idempotent Methods](https://www.rfc-editor.org/rfc/rfc9110.html)
- [RFC 6585: 429 Too Many Requests](https://www.rfc-editor.org/rfc/rfc6585.html)
- [មេរៀនទី ៥១៖ ហៅ External API ដោយ PHP cURL](lesson-51-php-curl-external-apis.md)
- [មេរៀនទី ៤៥៖ Webhooks និង Idempotency](lesson-45-php-webhooks-signature-idempotency.md)
