# មេរៀនទី ៥៣៖ Background Jobs និង Queue ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីមូលហេតុដែលការងារយូរគួរដំណើរការក្រៅ HTTP Request។
- រៀបចំ Flow Producer → Queue → Worker។
- ប្រើ Message/Handler Pattern ដើម្បីបំបែកការងារ។
- រចនា Job ឱ្យ Idempotent និងមាន Retry/Failure Handling។
- ដំណើរការ Worker និងតាមដាន Queue ក្នុង Production។

## ១. Background Job ជាអ្វី?

Background Job គឺការងារដែល App បញ្ជូនទៅ Worker ដំណើរការនៅពេលក្រោយ ជំនួសឱ្យធ្វើក្នុង HTTP Request ដែល User កំពុងរង់ចាំ។

ឧទាហរណ៍៖

- ផ្ញើ Email Verification ឬ Receipt។
- ដំណើរការ Webhook Event។
- បង្កើត Report/Export ធំ។
- កែរូបភាព ឬ Import CSV។
- ហៅ External API ដែលអាចយឺត។

Flow ទូទៅ៖

1. Web Request Validate និងរក្សាទិន្នន័យសំខាន់។
2. Producer បង្កើត Message ហើយដាក់ក្នុង Queue។
3. Worker ទាញ Message មកដំណើរការ Handler។
4. ជោគជ័យ៖ Message ត្រូវបាន Ack/ដកចេញពី Queue។
5. បរាជ័យបណ្ដោះអាសន្ន៖ Retry តាម Policy។
6. បរាជ័យជាប់៖ ផ្លាស់ទៅ Failure Queue/Dead Letter ដើម្បីពិនិត្យ។

## ២. Message គួរមានតែព័ត៌មានចាំបាច់

ឧទាហរណ៍ Message សម្រាប់ផ្ញើ Email៖

```php
<?php
final readonly class SendVerificationEmail
{
    public function __construct(
        public int $userId,
        public string $verificationTokenId,
    ) {
    }
}
```

គួរផ្ញើ ID ឬ Reference ទៅទិន្នន័យដែលចាំបាច់ ជំនួសឱ្យចម្លង Password, API Secret ឬទិន្នន័យរសើបទាំងមូលទៅ Queue។ Queue Message អាចត្រូវរក្សាទុកក្នុង Database/Redis/Broker និងអាចស្ថិតនៅទីនោះរយៈពេលមួយ។

Handler គួរទាញទិន្នន័យថ្មីពី Database តាម ID ហើយពិនិត្យថា Record នៅតែមាន និងស្ថិតក្នុង State ដែលអនុញ្ញាត។

## ៣. ឧទាហរណ៍ Message Handler ជាមួយ Symfony Messenger

Symfony Messenger គាំទ្រការដំណើរការ Message ភ្លាមៗ ឬបញ្ជូនទៅ Transport សម្រាប់ Worker ដំណើរការក្រោយមក។ ឧទាហរណ៍ Message និង Handler៖

```php
<?php
namespace App\Message;

final readonly class GenerateReport
{
    public function __construct(
        public int $reportId,
        public int $requestedByUserId,
    ) {
    }
}
```

```php
<?php
namespace App\MessageHandler;

use App\Message\GenerateReport;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final class GenerateReportHandler
{
    public function __invoke(GenerateReport $message): void
    {
        // Load report តាម ID ពិនិត្យសិទ្ធិ/State និងបង្កើត File។
        // រក្សា Result នៅ Private Storage ហើយ Update Status។
    }
}
```

ដើម្បីឱ្យ Message ដំណើរការជា Async ត្រូវកំណត់ Transport និង Route Message ទៅ Transport នោះក្នុង Symfony Configuration។ បន្ទាប់មក Worker អាចដំណើរការដោយ៖

```bash
php bin/console messenger:consume async
```

ឈ្មោះ Transport `async` និង Package/DSN ត្រូវកំណត់តាម Broker ដែលជ្រើស (Doctrine, Redis, AMQP ជាដើម)។ កុំដាក់ Credential របស់ Broker ក្នុង Git; ប្រើ Secret/Environment Configuration។

## ៤. At-Least-Once Delivery និង Idempotency

Queue ជាច្រើនធានា **At-Least-Once Delivery**៖ Message អាចត្រូវដំណើរការឡើងវិញ បើ Worker បានអនុវត្តការងាររួច តែបរាជ័យមុន Ack។ ដូច្នេះ Handler ត្រូវរៀបចំឱ្យ Idempotent៖

- ប្រើ Job/Operation ID ជា Unique Key ក្នុង Database។
- ពិនិត្យថា Job បានបញ្ចប់រួចឬនៅ មុនធ្វើ Side Effect ម្តងទៀត។
- រក្សា Business Update និង Job Status ក្នុង Transaction តាមដែលអាចធ្វើបាន។
- សម្រាប់ Payment/External API ប្រើ Provider Idempotency Key។
- កុំសន្មតថា Message មួយនឹងមកដល់តែម្តង។

ឧទាហរណ៍ Job បង្កើត Report អាច Save ទៅ Path តាម Report ID ហើយ Update Status តែម្ដង; Email/Payment ត្រូវមាន Mechanism បន្ថែមដើម្បីមិនផ្ញើ/គិតប្រាក់ស្ទួន។

## ៥. Retry, Delay និង Failure Queue

កំណត់ Retry តាមប្រភេទ Error៖

- Network Timeout ឬ 503 បណ្ដោះអាសន្ន៖ អាច Retry ជាមួយ Delay/Backoff។
- Input មិនត្រឹមត្រូវ ឬ Record មិនមាន៖ ជាទូទៅកុំ Retry ដដែលៗ។
- Provider Rate Limit៖ គោរព `Retry-After` ប្រសិនបើមាន។
- Retry អស់ចំនួន៖ ផ្ញើទៅ Failure/Dead Letter Queue និង Alert អ្នកថែទាំ។

ត្រូវកំណត់ចំនួន Retry, Backoff និង Maximum Processing Time។ Failure Queue មិនមែនជាកន្លែងលុបចោលបញ្ហាទេ; ត្រូវមាន Process ស៊ើបអង្កេត, ជួសជុល និង Redeliver ដោយប្រុងប្រយ័ត្ន។

## ៦. Worker Lifecycle និងការត្រួតពិនិត្យ

Worker ជា Long-Running Process ដូច្នេះត្រូវគិតពី៖

- Supervisor/Container Orchestrator សម្រាប់ Restart Worker បើវាបិទ។
- Graceful Shutdown ដើម្បីបញ្ចប់ Message បច្ចុប្បន្នមុន Stop។
- Restart Worker ក្រោយ Deploy ដើម្បី Load Code ថ្មី។
- កំណត់ Memory/Time Limit ឬចំនួន Message មុន Worker Restart។
- Monitor Queue Depth, Oldest Message Age, Failure Count និង Processing Duration។
- កុំឱ្យ Worker មាន Permission/Secret ច្រើនជាងតម្រូវការ។

កុំដំណើរការ Queue Worker តាម HTTP Request របស់អ្នកប្រើរាល់ដង; ប្រើ Process Manager/Platform Worker Service។

## ៧. Transaction និងការដាក់ Message ទៅ Queue

មាន Race Condition អាចកើតឡើង បើ Database Commit ជោគជ័យ តែការបញ្ជូន Message ទៅ Queue បរាជ័យ ឬផ្ទុយទៅវិញ។ សម្រាប់ប្រព័ន្ធដែលត្រូវការភាពទុកចិត្តខ្ពស់ ពិចារណា **Transactional Outbox**៖

1. ក្នុង Transaction តែមួយ រក្សា Business Data និង Outbox Event ក្នុង Database។
2. Separate Publisher អាន Outbox ហើយបញ្ជូនទៅ Broker។
3. បន្ទាប់ពី Publish ជោគជ័យ សម្គាល់ Outbox Row ថាបានបញ្ជូន។
4. Consumer នៅតែត្រូវ Idempotent ព្រោះការបញ្ជូនអាចស្ទួន។

## ៨. កំហុសដែលជួបញឹកញាប់

- ធ្វើ Email/Report ធំនៅក្នុង HTTP Request រហូត Timeout។
- ផ្ញើ Secret ឬទិន្នន័យរសើបមិនចាំបាច់ក្នុង Message Payload។
- គិតថា Message ដំណើរការតែម្តង ហើយបង្ក Side Effect ស្ទួន។
- Retry Error ទាំងអស់គ្មានកំណត់។
- មិនពិនិត្យ Failure Queue និង Queue Backlog។
- Deploy Code ថ្មីតែភ្លេច Restart Worker ចាស់។
- Commit Broker Credential ទៅ Repository។

## ៩. លំហាត់

1. រចនា Message សម្រាប់ Generate Report ដែលមានតែ ID ចាំបាច់។
2. សរសេរ Handler ដែលធ្វើការងារ Idempotent និង Update Job Status។
3. កំណត់ Retry Policy សម្រាប់ Network Error និង Validation Error។
4. គូរ Flow ពី Web Request → Queue → Worker → Failure Queue។
5. ពន្យល់ពី Race Condition រវាង DB Commit និង Publish Message ហើយស្នើ Outbox Pattern។

## ១០. ចម្លើយណែនាំខ្លី

- Queue ជួយបំបែកការងារយូរចេញពី HTTP Request និងអនុញ្ញាតឱ្យ Worker ដំណើរការដាច់ដោយឡែក។
- Message គួរផ្ទុកតែ Reference/ID ចាំបាច់; កុំដាក់ Password/Secret។
- រៀបចំ Handler ឱ្យ Idempotent ព្រោះ Message អាចមកច្រើនដង។
- Retry តែ Error បណ្ដោះអាសន្ន ដោយកំណត់ចំនួន និង Backoff។
- សម្រាប់ទិន្នន័យសំខាន់ដែលត្រូវ Publish ទៅ Queue ពិចារណា Transactional Outbox។

## ឯកសារយោង

- [Symfony Messenger: Queued Message Handling](https://symfony.com/doc/current/messenger.html)
- [មេរៀនទី ៤៥៖ Webhooks និង Idempotency](lesson-45-php-webhooks-signature-idempotency.md)
- [មេរៀនទី ៥២៖ Retry, Backoff និង Circuit Breaker](lesson-52-php-api-retries-backoff-circuit-breaker.md)
