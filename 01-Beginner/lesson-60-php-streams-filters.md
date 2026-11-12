# មេរៀនទី ៦០៖ PHP Streams និង Stream Filters

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Stream ជាប្រភព/ទិសដៅទិន្នន័យដែលអាចអាន ឬសរសេរជាបន្តបន្ទាប់។
- ប្រើ `php://temp` និង `php://memory` សម្រាប់ទិន្នន័យបណ្តោះអាសន្ន។
- បន្ថែម Stream Filter ដើម្បីបម្លែងទិន្នន័យពេលអាន/សរសេរ។
- ស្គាល់ Wrapper ដូចជា `file://`, `php://input` និង `http://`។
- គ្រប់គ្រង Resource និងការពារ Path/URL មិនទុកចិត្ត។

## ១. Stream ជាអ្វី?

Stream គឺ Interface សម្រាប់អាន/សរសេរទិន្នន័យពីប្រភពផ្សេងៗតាមលំដាប់។ ប្រភពអាចជា File, Memory, PHP Input/Output ឬ Network Wrapper។ Function ដូចជា `fopen()`, `fread()`, `fwrite()`, `fgets()` និង `stream_get_contents()` អាចធ្វើការជាមួយ Stream Resource។

ការអានជាបន្តបន្ទាប់មានប្រយោជន៍សម្រាប់ឯកសារធំ ព្រោះអាចដំណើរការមួយផ្នែកៗ ជំនួសឱ្យផ្ទុកទាំងមូលក្នុង Memory។

## ២. Memory និង Temp Stream

`php://memory` រក្សាទិន្នន័យនៅ Memory ទាំងស្រុង។ `php://temp` រក្សាទុកនៅ Memory ដំបូង ហើយអាចប្ដូរទៅ Temporary File ពេលទិន្នន័យលើសកម្រិត៖

```php
<?php
$stream = fopen('php://temp', 'w+');

if ($stream === false) {
    throw new RuntimeException('Unable to create temporary stream.');
}

try {
    fwrite($stream, "PHP\nStreams\n");
    rewind($stream);

    while (($line = fgets($stream)) !== false) {
        echo strtoupper($line);
    }
} finally {
    fclose($stream);
}
```

ប្រើ `try/finally` ដើម្បីបិទ Resource ទោះមាន Exception ក៏ដោយ។ សម្រាប់ទិន្នន័យ Sensitive សូមចងចាំថា Temporary Stream អាចប្រើ Temporary File នៅលើ Disk; គិតពី Permission និង Storage Environment។

## ៣. អាន File ជាបន្ទាត់ៗ

```php
<?php
$path = __DIR__ . '/data.txt';
$stream = fopen($path, 'rb');

if ($stream === false) {
    throw new RuntimeException('Unable to open file.');
}

try {
    while (($line = fgets($stream)) !== false) {
        $line = rtrim($line, "\r\n");
        // Validate និងដំណើរការ Line នេះភ្លាមៗ។
    }
} finally {
    fclose($stream);
}
```

បើ File Path មកពី User ត្រូវកុំភ្ជាប់ទៅ Path ដោយផ្ទាល់។ ប្រើ ID/Allowlist និងកំណត់ Base Directory ដែល Server គ្រប់គ្រង; ពិនិត្យថា Path ដែល Resolve ហើយនៅក្នុង Directory ដែលអនុញ្ញាត។ កុំទុកចិត្តតែការលុប String `../` ដោយមិនគិតពី Encoding/Symlink។

## ៤. Stream Filters

Stream Filter បម្លែងទិន្នន័យក្នុងពេលអាន ឬសរសេរ៖

```php
<?php
$stream = fopen('php://temp', 'w+');

if ($stream === false) {
    throw new RuntimeException('Unable to create stream.');
}

try {
    $filter = stream_filter_append(
        $stream,
        'convert.base64-encode',
        STREAM_FILTER_WRITE
    );

    if ($filter === false) {
        throw new RuntimeException('Unable to attach stream filter.');
    }

    fwrite($stream, 'PHP stream filter');
    rewind($stream);

    echo stream_get_contents($stream);
} finally {
    fclose($stream);
}
```

Filter អាចប្រើសម្រាប់ការបម្លែង Encoding ឬ Compression តាមតម្រូវការ។ កុំច្រឡំ Encoding/Transformation ជាមួយ Encryption ឬ Security Validation; Base64 គ្រាន់តែ Encode ទិន្នន័យ មិនមែនអ៊ិនគ្រីបទេ។

## ៥. Wrapper និង `php://input`

PHP មាន Stream Wrappers ជាច្រើន៖

- `file://` សម្រាប់ File System។
- `php://memory` និង `php://temp` សម្រាប់ Stream បណ្តោះអាសន្ន។
- `php://stdin`, `php://stdout`, `php://stderr` សម្រាប់ CLI I/O។
- `php://input` សម្រាប់អាន Raw Request Body ក្នុង HTTP Context។
- `http://` និង `https://` អាចប្រើតាម Stream Context សម្រាប់ Network Request។

ឧទាហរណ៍អាន JSON Raw Body៖

```php
<?php
$rawBody = file_get_contents('php://input');

if (!is_string($rawBody) || strlen($rawBody) > 1_000_000) {
    http_response_code(413);
    exit;
}

try {
    $payload = json_decode($rawBody, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException) {
    http_response_code(400);
    exit('JSON មិនត្រឹមត្រូវ។');
}
```

កំណត់ Request Size នៅ Web Server/Proxy ផងដែរ។ កុំអាន Body ដោយគ្មាន Limit នៅ Endpoint ដែលអាចទទួល Request ធំ។

## ៦. URL Wrappers និងសុវត្ថិភាព

URL Wrappers អាចឱ្យ PHP អាន/សរសេរទៅ Network ប្រសិនបើ Configuration អនុញ្ញាត។ ការប្រើ URL ដែល User ផ្ញើមកជាមួយ `fopen()`/`file_get_contents()` អាចបង្ក SSRF ឬចូលទៅ Resource ខាងក្នុង។

- ប្រើ Host/URL ដែលកំណត់ដោយ Server ឬ Allowlist។
- Validate Scheme, Host និង Port; កុំអនុញ្ញាត Protocol មិនចាំបាច់។
- គ្រប់គ្រង Redirect និង Timeout។
- កុំបិទ TLS Verification។
- កំណត់ `allow_url_fopen` តាមតម្រូវការ និង Deployment Policy។
- សម្រាប់ HTTP Client ដែលត្រូវការការគ្រប់គ្រងច្បាស់ សូមប្រើ cURL/Client Library ដែលបានកំណត់ Timeout និង TLS ត្រឹមត្រូវ។

## ៧. កំហុសដែលជួបញឹកញាប់

- ហៅ `fopen()` ហើយមិនពិនិត្យ `false`។
- មិនបិទ Stream Resource ក្រោយប្រើ។
- អានឯកសារធំទាំងមូលដោយ `file_get_contents()` ដោយមិនគិត Memory។
- ជឿថា Base64 គឺ Encryption។
- ប្រើ URL/Path ពី User ដោយផ្ទាល់ ហើយបង្ក SSRF/Path Traversal។
- មិនកំណត់ Size Limit លើ Input/Output Stream។

## ៨. លំហាត់

1. បង្កើត `php://temp` Stream ហើយសរសេរ/អានអត្ថបទឡើងវិញ។
2. អាន File ធំម្តងមួយ Line និងរាប់ចំនួន Line ដោយមិនរក្សាទុក Array ទាំងអស់។
3. បន្ថែម Base64 Encode Filter ហើយពន្យល់ថាវាមិនមែនជា Encryption។
4. អាន JSON Request Body ពី `php://input` និងដាក់ Size Limit។
5. ពន្យល់ពីហានិភ័យនៃការប្រើ URL ពី User ជា Stream Wrapper។

## ៩. ចម្លើយណែនាំខ្លី

- Stream អនុញ្ញាតឱ្យប្រើ I/O ជាបន្តបន្ទាប់តាម Interface ស្របគ្នា។
- `php://temp` អាចប្រើ Memory ហើយអាចផ្លាស់ទៅ Temp File ពេលលើសកម្រិត។
- Stream Filter បម្លែងទិន្នន័យកំពុងឆ្លងកាត់ Stream; វាមិនជំនួស Encryption/Validation ទេ។
- ប្រើ `try/finally` ដើម្បីបិទ Resource និងកំណត់ Size/Timeout។
- ការប្រើ URL ពី User អាចនាំទៅ SSRF; ប្រើ Allowlist និងការគ្រប់គ្រង Network។

## ឯកសារយោង

- [PHP Manual: Supported Protocols and Wrappers](https://www.php.net/manual/en/wrappers.php)
- [PHP Manual: Stream Filters](https://www.php.net/manual/en/filters.php)
- [PHP Manual: stream_filter_append()](https://www.php.net/manual/en/function.stream-filter-append.php)
- [មេរៀនទី ៥៨៖ Generators និង Iterators](lesson-58-php-generators-iterators.md)
- [មេរៀនទី ៥១៖ ហៅ External API ដោយ PHP cURL](lesson-51-php-curl-external-apis.md)
