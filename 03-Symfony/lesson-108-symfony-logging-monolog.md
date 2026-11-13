# មេរៀនទី ១០៨៖ Symfony Logging និង Monolog

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- Inject LoggerInterface ទៅ service និង controller។
- ជ្រើស log levels និងបន្ថែម context data។
- ប្រើ Monolog handlers សម្រាប់កំណត់គោលដៅ logs។
- ជៀសវាងការកត់ត្រា secrets និងព័ត៌មានផ្ទាល់ខ្លួន។

## ១. ហេតុអ្វីត្រូវប្រើ Logging?

Log ជាកំណត់ត្រាអំពីសកម្មភាព និងកំហុសរបស់កម្មវិធី។ វាជួយពិនិត្យបញ្ហាដែលកើតឡើងនៅ development ឬ production ដោយមិនបង្ហាញព័ត៌មាន debug ទៅអ្នកប្រើ។

PSR-3 កំណត់ interface និង log levels ទូទៅ ដូចជា debug, info, notice, warning, error, critical, alert និង emergency។

## ២. Inject Logger និងសរសេរ Logs

Inject `LoggerInterface` ទៅ service៖

```php
<?php

namespace App\Service;

use Psr\Log\LoggerInterface;

final class PaymentProcessor
{
    public function __construct(
        private LoggerInterface $logger,
    ) {
    }

    public function process(int $orderId): void
    {
        $this->logger->info('កំពុងដំណើរការការទូទាត់ Order {orderId}', [
            'orderId' => $orderId,
        ]);

        try {
            // ហៅ payment provider តាម logic ពិត
        } catch (\Throwable $exception) {
            $this->logger->error('ការទូទាត់ Order {orderId} បរាជ័យ', [
                'orderId' => $orderId,
                'exception' => $exception,
            ]);

            throw $exception;
        }
    }
}
```

ប្រើ placeholder ក្នុង message និងបញ្ជូន values ជា context។ វាងាយស្រួលស្វែងរក និងជួយ logging implementation ដោះស្រាយ escaping។ កុំ log card number, password, access token ឬព័ត៌មានរសើប។

## ៣. ជ្រើស Log Level

- `debug`៖ ព័ត៌មានលម្អិតសម្រាប់អភិវឌ្ឍ និងបំបាត់កំហុស។
- `info`៖ ព្រឹត្តិការណ៍ធម្មតាដែលគួរកត់ត្រា។
- `warning`៖ បញ្ហាដែលមិនបញ្ឈប់ប្រព័ន្ធ ប៉ុន្តែគួរតាមដាន។
- `error`៖ ប្រតិបត្តិការមួយបរាជ័យ។
- `critical` និងខ្ពស់ជាងនេះ៖ បញ្ហាធ្ងន់ដែលត្រូវដោះស្រាយឆាប់រហ័ស។

ជ្រើស level តាមអត្ថន័យ ព្រោះ handler អាច filter ឬផ្ញើ alert តាម level។

## ៤. កន្លែងរក Logs និង Monolog

ក្នុង Symfony environment `dev` logs ជាទូទៅស្ថិតនៅ `var/log/dev.log`។ ក្នុង `prod` project ថ្មីជាច្រើនបញ្ជូន logs ទៅ STDERR ដើម្បីឱ្យ container/platform ប្រមូល។ ពិនិត្យ config ពិតរបស់ project មុនសន្មតទីតាំង។

Symfony អាចប្រើ logger សាមញ្ញបាន ហើយអាចដំឡើង Monolog សម្រាប់ handlers និង routing កម្រិតខ្ពស់៖

```bash
composer require symfony/monolog-bundle
```

Monolog handlers អាចសរសេរទៅ file, STDERR, syslog ឬ logging service ផ្សេង។ Configuration ជាទូទៅស្ថិតក្នុង `config/packages/monolog.yaml` និងអាចខុសគ្នាតាម environment។

## ៥. បន្ថែម Context និង Correlation ID

បន្ថែម identifiers ដែលជួយតាមដាន request ឬ record ដូចជា `orderId`, `userId` (ប្រសិនបើមិនរសើប) ឬ request correlation ID។ ប្រើ context keys មានឈ្មោះច្បាស់។ កុំដាក់ password/token ទៅ context ទោះគ្រាន់តែចង់ debug ក៏ដោយ។

ក្នុង long-running worker ត្រូវគិតពីការសម្អាត context ដើម្បីកុំឱ្យទិន្នន័យ request មួយហូរទៅ message បន្ទាប់។

## ៦. ការគ្រប់គ្រង Log Files

បើសរសេរ logs ទៅ file ត្រូវគិតពី file permissions, disk usage និង rotation។ ប្រើ handler ឬ platform log management ដែលកំណត់ retention និងការចូលប្រើបាន។ កុំដាក់ log files ក្នុង public web directory។

ក្នុង production containerized environment ការបញ្ជូនទៅ STDERR ជារឿយៗធ្វើឱ្យ platform អាចប្រមូល និងបង្វិល logs បានស្រួលជាងការរក្សា file ក្នុង container។

## ៧. សាកល្បង Logging

អាចប្រើ PHPUnit logger mock ឬ test handler ដើម្បីផ្ទៀងផ្ទាត់ថា service កត់ត្រាក្នុង level និង context ត្រឹមត្រូវ។ កុំប្រើ real production logging destination ក្នុង unit test។

## កំហុសដែលជួបញឹកញាប់

- **ប្រើ log level ខុស**៖ error ទាំងអស់ជា critical ធ្វើឱ្យ alert មាន noise។
- **កត់ត្រា secret ឬ PII**៖ លុប ឬ redact values មុនចូល log។
- **Log message ជា concatenated string**៖ ប្រើ placeholders និង context។
- **Logs កើនលឿនពេញ disk**៖ កំណត់ rotation និង retention។
- **សន្មតថា prod logs ទៅ file**៖ ពិនិត្យ Monolog/environment config; ជាញឹកញាប់វាទៅ STDERR។
- **បង្ហាញ exception details ដល់អ្នកប្រើ**៖ ប្រើ generic error response ហើយកត់ត្រាព័ត៌មានសម្រាប់អ្នកថែទាំនៅ server side។

## លំហាត់

1. Inject `LoggerInterface` ទៅ service ហើយកត់ត្រា order ID ដោយ placeholder។
2. កត់ត្រា exception ជា error ហើយបន្តបោះ exception ឡើងវិញ។
3. ដំឡើង Monolog ហើយពិនិត្យ handlers តាម environment។
4. កំណត់ log rotation ឬប្រើ platform logs ក្នុង deployment សាកល្បង។
5. ពិនិត្យ logs របស់អ្នក ហើយដកចេញ token ឬទិន្នន័យរសើប។

## សំណួរត្រួតពិនិត្យ

1. តើ LoggerInterface ផ្ដល់អត្ថប្រយោជន៍អ្វី?
2. តើហេតុអ្វីគួរប្រើ context placeholders?
3. តើ log level info និង error ខុសគ្នាដូចម្ដេច?
4. តើហេតុអ្វីត្រូវ rotate logs?
5. តើព័ត៌មានអ្វីខ្លះមិនគួរកត់ត្រា?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Logging](https://symfony.com/doc/7.4/logging.html)
- [PSR-3 Logger Interface](https://www.php-fig.org/psr/psr-3/)
