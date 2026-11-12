# មេរៀនទី ៦៩៖ HTTP Middleware និង PSR-7/PSR-15 ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី HTTP Middleware និងរបៀបដែលវារុំ Request Handler។
- ស្គាល់តួនាទី PSR-7 និង PSR-15។
- សរសេរ Middleware ដែលបន្ថែម Request ID មុន/ក្រោយ Handler។
- យល់ពីលំដាប់ Middleware និងការគ្រប់គ្រង Error។
- បំបែក Cross-Cutting Concerns ចេញពី Business Logic ក្នុង PHP App។

## ១. Middleware ជាអ្វី?

Middleware គឺ Component ដែលចូលរួមដំណើរការ HTTP Request មុន ឬក្រោយ Handler។ ឧទាហរណ៍៖

- Request ID និង Logging Context។
- Authentication/Authorization។
- CORS និង Rate Limiting។
- Error Handling និង Response Headers។
- Routing និង Request Parsing។

Middleware អាចបន្តទៅ Handler បន្ទាប់ ឬត្រឡប់ Response ដោយផ្ទាល់។ ដូច្នេះលំដាប់ Middleware មានឥទ្ធិពលលើ Security និង Response Behavior។

## ២. PSR-7 និង PSR-15

- **PSR-7** កំណត់ Interface សម្រាប់ HTTP Request, Response, URI, Stream និង Uploaded File។
- **PSR-15** កំណត់ Interface សម្រាប់ Server Request Handler និង Middleware។
- **PSR-17** កំណត់ Factory Interface សម្រាប់បង្កើត HTTP Message Object។

PSR ជា Interface Contract; វាមិនផ្តល់ Implementation ឬ Middleware Pipeline ដោយស្វ័យប្រវត្តិទេ។ Project ត្រូវជ្រើស Implementation/Framework ដែលអនុវត្ត PSR ហើយស្គាល់កំណែ Package ដែលបានដំឡើង។

## ៣. Request ID Middleware

ឧទាហរណ៍ Middleware ដែលបង្កើត Request ID នៅ Server, ដាក់វាលើ Request Attribute ហើយបន្ថែម Header ទៅ Response៖

```php
<?php
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\MiddlewareInterface;
use Psr\Http\Server\RequestHandlerInterface;

final class RequestIdMiddleware implements MiddlewareInterface
{
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        $requestId = bin2hex(random_bytes(16));
        $requestWithId = $request->withAttribute('request_id', $requestId);

        $response = $handler->handle($requestWithId);

        return $response->withHeader('X-Request-ID', $requestId);
    }
}
```

PSR-7 Message ជា Immutable Interface ជាទូទៅ៖ `withAttribute()`/`withHeader()` ត្រឡប់ Object ថ្មី។ ត្រូវរក្សា Return Value; កុំសន្មតថា Original Request/Response ត្រូវបានកែប្រែ។

Request ID គួរបង្កើតនៅ Server ឬ Gateway ដែលទុកចិត្ត។ បើទទួល ID ពី Client ត្រូវ Validate ទម្រង់/ប្រវែង ហើយកុំអនុញ្ញាតឱ្យវាបញ្ចូល Newline ឬបំពុល Log។

## ៤. Middleware Stack និងលំដាប់

ឧទាហរណ៍ Stack៖

1. Error Handling Middleware ខាងក្រៅបំផុត ដើម្បីចាប់ Exception ពី Layer ខាងក្រោម។
2. Request ID/Logging Context។
3. Routing។
4. Authentication/Authorization សម្រាប់ Route ដែលត្រូវការ។
5. Body Parsing/Validation តាមសមស្រប។
6. Application Handler។

លំដាប់ពិតអាស្រ័យលើ Framework/Router និង Security Requirements។ CORS Preflight, Authentication និង Error Response ត្រូវបានរៀបចំឱ្យ Browser/Client ទទួល Header ត្រឹមត្រូវ។ Middleware មួយមិនគួររំលង Authorization របស់ Handler ផ្សេងដោយគ្មានចេតនា។

## ៥. Middleware អាចបញ្ឈប់ ឬបន្ត Request

Middleware ដែលត្រូវការបញ្ឈប់ Request អាចត្រឡប់ Response ដោយមិនហៅ `$handler->handle()` ដូចជា Authentication Guard។ PSR-15 ណែនាំឱ្យ Middleware ដែលបង្កើត Response ប្រើ Response Prototype ឬ Factory ដើម្បីកុំឱ្យចងជាប់ Implementation ជាក់លាក់។

ឧទាហរណ៍ Pseudocode៖

```php
if (!$userIsAuthenticated) {
    return $responseFactory
        ->createResponse(401)
        ->withHeader('Content-Type', 'application/json');
}

return $handler->handle($request);
```

នេះជាគំនិតទូទៅ; វិធីបង្កើត JSON Body និង Response Factory អាស្រ័យលើ PSR-7/17 Implementation របស់ Project។

## ៦. Middleware និង Business Logic

ដាក់ Concern ដែលពាក់ព័ន្ធនឹង Request/Response ជាទូទៅនៅ Middleware។ ទុក Domain Rule នៅ Application Service/Handler ដើម្បីឱ្យវាអាចប្រើពី HTTP, CLI, Queue Worker និង Tests ផ្សេងៗបាន។

កុំបង្កើត Middleware ធំមួយដែលគ្រប់គ្រង Authentication, Database, Business Logic និង Response ទាំងអស់។ Middleware គួរមានតួនាទីតូច និងលំដាប់ស្របគ្នា។

## ៧. កំហុសដែលជួបញឹកញាប់

- ភ្លេច Assign Return Value ពី `withHeader()`/`withAttribute()`។
- ដាក់ Error Handler ខាងក្នុង Middleware ដែលអាចបោះ Exception ដោយមិនចាប់។
- Request ID អាចត្រូវ Client ក្លែង ហើយប្រើក្នុង Log ដោយគ្មាន Validation។
- Authentication Middleware ដំណើរការមិនគ្រប់ Route។
- Middleware ធ្វើ Business Logic ច្រើនពេក និងពិបាក Test។
- សន្មតថា PSR ជា Package Implementation ឬ Framework ដោយខ្លួនឯង។

## ៨. លំហាត់

1. បន្ថែម Request ID Middleware ហើយឱ្យ Handler អាន ID តាម Request Attribute។
2. សរសេរ Middleware ដែលឆ្លើយ 401 មុន Handler បើគ្មាន User។
3. រៀបចំ Error Middleware នៅខាងក្រៅបំផុត និងបញ្ជូន Error JSON ទូទៅ។
4. ពិនិត្យលំដាប់ CORS, Routing និង Authentication Middleware។
5. សរសេរ Unit Test ដែល Mock Request Handler ហើយបញ្ជាក់ថា Request ID នៅ Response Header។

## ៩. ចម្លើយណែនាំខ្លី

- Middleware រុំ Handler ដើម្បីអនុវត្តការងាររួមលើ Request/Response។
- PSR-7 កំណត់ HTTP Message Interfaces; PSR-15 កំណត់ Middleware/Handler Contracts។
- PSR-7 `with*` Methods ត្រឡប់ Message ថ្មី; ត្រូវរក្សា Return Value។
- Error Middleware គួរអាចចាប់ Exception ពី Stack ខាងក្នុង។
- Domain Logic គួររស់នៅ Service/Handler មិនចងជាប់ HTTP Middleware។

## ឯកសារយោង

- [PHP-FIG PSR-7: HTTP Message Interfaces](https://www.php-fig.org/psr/psr-7/)
- [PHP-FIG PSR-15: HTTP Server Request Handlers](https://www.php-fig.org/psr/psr-15/)
- [PHP-FIG PSR-17: HTTP Factories](https://www.php-fig.org/psr/psr-17/)
- [មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP](lesson-26-php-json-rest-api.md)
- [មេរៀនទី ៤០៖ Error Handling, Logging និង Debugging](lesson-40-php-error-logging-debugging.md)
