# មេរៀនទី ៤០៖ Error Handling, Logging និង Debugging ក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Development Error Display និង Production Error Logging។
- ប្រើ Exception Handling ដើម្បីគ្រប់គ្រងកំហុសដោយមិនបង្ហាញព័ត៌មានខាងក្នុង។
- បង្កើត Request ID ដើម្បីភ្ជាប់ Error Response ទៅ Log។
- កត់ Log ដោយមិនបញ្ចេញ Password, Token ឬទិន្នន័យរសើប។
- ប្រើ Log Level និងទុក Log នៅទីតាំងដែលមាន Access Control។

## ១. Error Handling និង Debugging

Error ជាសញ្ញាថាកម្មវិធីជួបបញ្ហា។ នៅ Development យើងត្រូវការព័ត៌មានលម្អិតដើម្បី Debug; នៅ Production អ្នកប្រើគួរទទួលសារទូទៅ ខណៈព័ត៌មានលម្អិតត្រូវរក្សាក្នុង Log ដែលបានការពារ។

- **Display Error**៖ បង្ហាញកំហុសទៅ Browser/CLI។
- **Log Error**៖ រក្សាព័ត៌មានកំហុសនៅ Log សម្រាប់អ្នកថែទាំ។
- **Exception**៖ វត្ថុដែលបញ្ជូនលក្ខខណ្ឌខុសពី Code ទៅកន្លែងដែលអាចដោះស្រាយបាន។

កុំបង្ហាញ Stack Trace, SQL, File Path ឬ Environment Variables ដល់អ្នកប្រើនៅ Production។

## ២. កំណត់ PHP Error Settings

នៅ Production គួរកំណត់ក្នុង `php.ini` ឬ Deployment Configuration៖

```ini
error_reporting = E_ALL
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/my-php-app/php-error.log
```

ត្រូវធានាថា Log Directory អាចសរសេរបានដោយ PHP Process តែអ្នកគ្មានសិទ្ធិមិនអាចអានបាន។ ការកំណត់ពិតអាស្រ័យលើ PHP SAPI/Container/Hosting របស់អ្នក។ នៅ Development អាចបើក Display Error ដើម្បីស្វែងរកបញ្ហា ប៉ុន្តែកុំយក Configuration នោះទៅ Production។

## ៣. ចាប់ Exception នៅ Application Boundary

ឧទាហរណ៍សាមញ្ញសម្រាប់ JSON API៖

```php
<?php
$requestId = bin2hex(random_bytes(16));

try {
    $result = handleRequest();
    header('Content-Type: application/json; charset=utf-8');
    echo json_encode(['data' => $result], JSON_THROW_ON_ERROR);
} catch (Throwable $exception) {
    error_log(sprintf(
        'request_id=%s exception_class=%s message=%s',
        $requestId,
        get_class($exception),
        $exception->getMessage()
    ));

    http_response_code(500);
    header('Content-Type: application/json; charset=utf-8');
    header('X-Request-ID: ' . $requestId);
    echo json_encode([
        'error' => [
            'code' => 'internal_error',
            'message' => 'មានបញ្ហាក្នុងប្រព័ន្ធ។ សូមសាកល្បងម្ដងទៀត។',
            'request_id' => $requestId,
        ],
    ], JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);
}
```

ក្នុង App ពិត គួរមាន Error Handler/Middleware កណ្ដាល និងកំណត់ HTTP Status សមស្របសម្រាប់ Validation, Authentication, Not Found និង Server Error។ កុំ Catch Throwable គ្រប់ទីកន្លែងហើយលាក់កំហុស; ដោះស្រាយតែនៅកន្លែងដែលអាចធ្វើសកម្មភាពត្រឹមត្រូវ ហើយកុំចាប់ Error រួចបន្តដោយ State ខូច។

## ៤. Request ID និង Log Context

Request ID ជួយស្វែងរក Log ដែលពាក់ព័ន្ធនឹង Request មួយ។ បង្កើតដោយ Server ឬទទួលពី Trusted Gateway បន្ទាប់ពី Validate ទម្រង់។ កុំយក Header ពី Client ទៅ Log ដោយមិនកំណត់ប្រវែង/ទម្រង់ ព្រោះអាចបញ្ចូល Log Injection។

Context ដែលអាចមានប្រយោជន៍៖

- Request ID និង Route Template (ឧ. `/users/{id}`)។
- Error Class និងសកម្មភាពដែលបរាជ័យ។
- Status Code, Timestamp និង Service Name។
- User ID ដែលបាន Authenticate ប្រសិនបើចាំបាច់ និងគោរព Privacy Policy។

ជៀសវាង Log Request Body ទាំងមូល ឬ Header ទាំងអស់ដោយស្វ័យប្រវត្តិ។

## ៥. អ្វីដែលមិនគួរ Log

កុំ Log៖

- Password, Password Reset Token, API Token ឬ Session ID។
- Authorization Header, Cookie ឬ SMTP/Database Credential។
- លេខកាត/ព័ត៌មានផ្ទាល់ខ្លួនដែលមិនចាំបាច់។
- SQL Parameter ឬ Full Connection String ដែលមាន Secret។
- Request URL ពេញ ប្រសិនបើ Query String មាន Token/Secret។

បើត្រូវ Log Identifier ដែលមានភាពរសើប ត្រូវកំណត់តាមតម្រូវការ, Mask/Hash តាមគោលបំណង និងមានការកំណត់ Retention/Access។

## ៦. Log Levels និងការត្រួតពិនិត្យ

ប្រើ Log Level ដែលមានន័យច្បាស់៖

- **DEBUG**៖ ព័ត៌មានលម្អិតសម្រាប់ Development; ជាទូទៅមិនបើកច្រើននៅ Production។
- **INFO**៖ ព្រឹត្តិការណ៍ធម្មតាដែលមានប្រយោជន៍។
- **WARNING**៖ ស្ថានភាពមិនធម្មតា ប៉ុន្តែកម្មវិធីអាចបន្តបាន។
- **ERROR**៖ Operation បរាជ័យ ឬត្រូវការស៊ើបអង្កេត។
- **CRITICAL**៖ សេវាសំខាន់មិនអាចដំណើរការ។

ប្រើ Logger Library/PSR-3 Interface សម្រាប់ App ធំ ដើម្បីចែក Handler និង Context ឱ្យស្របគ្នា។ ត្រូវមាន Rotation, Retention និង Alert តាមសមត្ថភាពប្រព័ន្ធ; កុំទុក Log ឱ្យពេញ Disk ដោយគ្មានការគ្រប់គ្រង។

## ៧. កំហុសដែលជួបញឹកញាប់

- `display_errors=On` នៅ Production។
- បង្ហាញ Exception Message ឬ Stack Trace ដល់ Browser។
- Catch Exception ទទេហើយមិន Log ឬមិនបញ្ជូន Error។
- Log Request ទាំងមូលដែលមាន Cookie/Authorization។
- Log ទៅ Public Web Directory។
- Log User Input ដោយមិនកំណត់ប្រវែង/គ្រប់គ្រង Newline ដែលអាចធ្វើឱ្យ Log ត្រូវក្លែង។
- ប្រើព័ត៌មានលម្អិតក្នុង Log ប៉ុន្តែមិនមាន Access Control/Retention។

## ៨. លំហាត់

1. បិទ Error Display និងបើក Error Logging សម្រាប់ Production Configuration។
2. បង្កើត Request ID មួយសម្រាប់ Request ហើយដាក់វាក្នុង Error Response។
3. បង្កើត JSON Error Handler ដែល Log Details ប៉ុន្តែបញ្ជូនសារទូទៅទៅ Client។
4. ពិនិត្យ Log របស់កម្មវិធី ហើយសម្គាល់ Password/Token/Cookie ដែលមិនគួររក្សាទុក។
5. កំណត់ Log Level និង Retention Policy សម្រាប់ Development និង Production។

## ៩. ចម្លើយណែនាំខ្លី

- Production: `display_errors=Off`, `log_errors=On`, និង Error Log នៅទីតាំងមាន Access Control។
- Client ទទួលសារ Error ទូទៅ និង Request ID; Log មានព័ត៌មានគ្រប់គ្រាន់សម្រាប់ Debug។
- កុំ Log Password, Token, Session ID, Cookie ឬ Authorization Header។
- ប្រើ Handler កណ្ដាល និង Log Level ច្បាស់; កុំលាក់ Exception ដោយគ្មានដំណោះស្រាយ។
- កំណត់ Log Rotation/Retention ដើម្បីគ្រប់គ្រងទំហំ និង Privacy។

## ឯកសារយោង

- [PHP: Runtime Configuration for Errors and Logging](https://www.php.net/manual/en/errorfunc.configuration.php)
- [PHP: error_log()](https://www.php.net/manual/en/function.error-log.php)
- [OWASP Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
- [មេរៀនទី ១៣៖ Error Handling និង Exceptions](lesson-13-php-error-handling-and-exceptions.md)
