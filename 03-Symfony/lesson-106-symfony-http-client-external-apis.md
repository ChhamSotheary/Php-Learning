# មេរៀនទី ១០៦៖ Symfony HTTP Client សម្រាប់ External API

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ផ្ញើ HTTP request ទៅ API ដោយ Symfony HttpClient។
- ដំណើរការ JSON response និង HTTP errors។
- កំណត់ timeout, headers និង authentication។
- រក្សា API token ក្នុង environment variable និងបង្ការការហៅ URL ដែលមិនទុកចិត្ត។

## ១. ដំឡើង និង Inject HTTP Client

ក្នុង Symfony project ដំឡើង component៖

```bash
composer require symfony/http-client
```

Inject interface ទៅ service៖

```php
<?php

namespace App\Service;

use Symfony\Contracts\HttpClient\HttpClientInterface;

final class CatalogApiClient
{
    public function __construct(
        private HttpClientInterface $httpClient,
    ) {
    }
}
```

Symfony នឹងផ្ដល់ HTTP client service តាម autowiring។

## ២. ផ្ញើ Request និងអាន JSON

បន្ថែម method ស្វែងរក product ពី API៖

```php
public function getProduct(int $id): array
{
    $response = $this->httpClient->request(
        'GET',
        'https://api.example.test/products/'.$id,
        [
            'headers' => [
                'Accept' => 'application/json',
            ],
            'timeout' => 5,
        ],
    );

    return $response->toArray();
}
```

`request()` បង្កើត request ហើយ `toArray()` decode JSON response ទៅ PHP array។ សម្រាប់ API ដែលកំណត់ផ្ទាល់ អ្នកអាចប្រើ client មាន base URI ដើម្បីកុំឱ្យសរសេរ host ដដែលៗក្នុងគ្រប់ method។

## ៣. ផ្ញើ POST JSON

ប្រើ `json` option ដើម្បី encode payload និងកំណត់ JSON content type៖

```php
$response = $this->httpClient->request(
    'POST',
    'https://api.example.test/products',
    [
        'json' => [
            'name' => 'សៀវភៅ PHP',
            'priceInCents' => 1800,
        ],
        'timeout' => 5,
    ],
);

$data = $response->toArray();
```

កុំទទួល payload ពី user ហើយបញ្ជូនបន្តទៅ API ដោយមិន validate។ កំណត់ fields ដែលអនុញ្ញាត និងបម្លែងប្រភេទទិន្នន័យឱ្យត្រឹមត្រូវ។

## ៤. ដោះស្រាយ HTTP និង Network Errors

HTTP Client អាចបោះ exception នៅពេលអាន response ដែលមាន error status ឬពេល network បរាជ័យ។ ប្រើ exception types សមស្រប៖

```php
use Symfony\Contracts\HttpClient\Exception\HttpExceptionInterface;
use Symfony\Contracts\HttpClient\Exception\TransportExceptionInterface;

try {
    $response = $this->httpClient->request(
        'GET',
        'https://api.example.test/products/123',
        ['timeout' => 5],
    );

    return $response->toArray();
} catch (HttpExceptionInterface $exception) {
    // API បានឆ្លើយតបជាមួយ HTTP error status
    // កត់ត្រា status/context ដោយមិន log secrets ឬ private payload
} catch (TransportExceptionInterface $exception) {
    // timeout, DNS, TLS ឬបញ្ហា network
}
```

ការចាត់ថ្នាក់ exception អាស្រ័យលើពេលដែល response ត្រូវបានអាន។ ចាត់ចែង error នៅកន្លែងដែលសមស្រប ហើយកុំលាក់បញ្ហាទាំងអស់ដោយត្រឡប់ array ទទេ ព្រោះវាធ្វើឱ្យ debugging ពិបាក។

## ៥. ការផ្ទៀងផ្ទាត់ API និងការពារ Secrets

សម្រាប់ API ដែលទាមទារ token សូមកំណត់ token ក្នុង environment៖

```dotenv
CATALOG_API_TOKEN=replace_with_local_secret
```

បញ្ចូលតម្លៃតាម config/service options ឬ scoped client របស់ Symfony។ កុំ hard-code token ក្នុង PHP, commit ទៅ repository ឬបង្ហាញក្នុង logs។ ក្នុង production ប្រើ secret manager ឬ environment injection ដែល platform ផ្ដល់។

បើត្រូវបន្ថែម header ដោយផ្ទាល់ ឧទាហរណ៍៖

```php
'headers' => [
    'Authorization' => 'Bearer '.$token,
    'Accept' => 'application/json',
],
```

កុំប្រើ API token ដដែលជាមួយ host ដែលមិនទុកចិត្ត។ Scoped client ជួយកំណត់ base URI និង headers សម្រាប់ domain ដែលត្រូវគ្នាប៉ុណ្ណោះ។

## ៦. SSRF និង URL ដែលអ្នកប្រើផ្ដល់

កុំឱ្យ user បញ្ចូល URL សេរីដែល server ត្រូវទៅ fetch ដោយផ្ទាល់។ វាអាចធ្វើឱ្យ server ទៅកាន់ internal IP, metadata service ឬ host ដែលមិនគួរចូល។ កំណត់ allowlist domain/scheme, validate URL និងអនុវត្ត SSRF protections របស់ Symfony HttpClient នៅពេល use case ត្រូវការ។

## ៧. សាកល្បង និង Debug

ប្រើ MockHttpClient ឬ test tools របស់ Symfony ដើម្បីសាកល្បង request ដោយមិនចាំបាច់ភ្ជាប់ API ពិត។ ពិនិត្យ status, headers និង response parsing។ កុំដាក់ API token ពិតក្នុង test fixture។

## កំហុសដែលជួបញឹកញាប់

- **JSON មិនអាច decode**៖ ពិនិត្យ response content-type/body និងប្រើ `toArray()` សម្រាប់ JSON។
- **Request ចាំយូរ**៖ កំណត់ timeout សមស្រប។
- **មិនបែងចែក HTTP error និង network error**៖ ដោះស្រាយ exception types ផ្សេងៗ។
- **Log Authorization header**៖ កុំ log credentials/tokens។
- **ប្រើ URL ពី user input ដោយផ្ទាល់**៖ បង្កើត SSRF risk; ប្រើ allowlist និង validation។
- **បញ្ជូន token ទៅគ្រប់ domain**៖ ប្រើ scoped client និងកំណត់ host ឱ្យច្បាស់។

## លំហាត់

1. បង្កើត `CatalogApiClient` ដែលទាញ product JSON ដោយ GET។
2. បន្ថែម POST JSON request និង validate fields មុនបញ្ជូន។
3. កំណត់ timeout និងដោះស្រាយ HTTP errors និង transport errors។
4. ដាក់ token ក្នុង environment variable ហើយពិនិត្យថាមិនមានក្នុង source code/logs។
5. សរសេរ unit test ដោយ mock HTTP response មួយ success និងមួយ error។

## សំណួរត្រួតពិនិត្យ

1. តើ `toArray()` ធ្វើអ្វី?
2. តើ `json` option ខុសពី `body` ដូចម្ដេច?
3. តើ timeout ជួយការពារអ្វី?
4. តើ SSRF ជាអ្វី?
5. តើហេតុអ្វីត្រូវប្រើ scoped client សម្រាប់ API token?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 HTTP Client](https://symfony.com/doc/7.4/http_client.html)
- [Symfony 7.4 HTTP Client Testing](https://symfony.com/doc/7.4/http_client.html#testing)
