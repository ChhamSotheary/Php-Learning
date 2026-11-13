# មេរៀនទី ១០៣៖ Symfony Messenger និង Background Queues

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក message, handler, bus និង transport។
- បង្កើត message class និង handler។
- កំណត់ message ឱ្យដំណើរការតាម queue។
- ដំណើរការ worker និងពិនិត្យ failed messages ជាមូលដ្ឋាន។

## ១. Messenger ជាអ្វី?

Symfony Messenger ផ្ដល់ message bus ដើម្បីបញ្ជូន message ទៅ handler។ តាមលំនាំដើម handler អាចដំណើរការភ្លាមៗក្នុង request។ បើចង់ពន្យារពេល អាច route message ទៅ transport ដែលជាទូទៅជា queue ហើយ worker ទាញមកដំណើរការក្រោយ។

- **Message** ផ្ទុកព័ត៌មានដែល handler ត្រូវការ។
- **Handler** អនុវត្ត business logic សម្រាប់ message។
- **Transport** ផ្ញើ និងទទួល message (ឧ. Doctrine, Redis, RabbitMQ)។
- **Worker** ដំណើរការនៅ background ដើម្បីទាញ message ពី transport។

## ២. ដំឡើង Messenger

```bash
composer require symfony/messenger
```

អាចប្រើ Doctrine transport ដើម្បីទុក message ក្នុង database ដូចគ្នា។ ពិនិត្យ Symfony Messenger docs សម្រាប់ package/transport ដែលត្រូវការនឹង version របស់អ្នក។ បើ project មិនទាន់មាន Doctrine ត្រូវដំឡើង និងរៀបចំវាមុន។

## ៣. បង្កើត Message និង Handler

បង្កើត `src/Message/SendOrderConfirmation.php`៖

```php
<?php

namespace App\Message;

final class SendOrderConfirmation
{
    public function __construct(
        private int $orderId,
    ) {
    }

    public function getOrderId(): int
    {
        return $this->orderId;
    }
}
```

ផ្ញើ ID ជំនួស Doctrine Entity object ដើម្បីកុំឱ្យ message ជាប់នឹង lifecycle របស់ EntityManager និងឱ្យ message ងាយ serialize។ បង្កើត handler `src/MessageHandler/SendOrderConfirmationHandler.php`៖

```php
<?php

namespace App\MessageHandler;

use App\Message\SendOrderConfirmation;
use Symfony\Component\Messenger\Attribute\AsMessageHandler;

#[AsMessageHandler]
final class SendOrderConfirmationHandler
{
    public function __invoke(SendOrderConfirmation $message): void
    {
        $orderId = $message->getOrderId();

        // ទាញ Order ឡើងវិញ ហើយផ្ញើ email តាម Mailer service។
    }
}
```

Handler អាច inject service ដូចជា repository និង mailer តាម constructor។ កូដផ្ញើ email នៅទីនេះជាគ្រោង; ត្រូវបន្ថែម business logic ពិត និងកំណត់ការដោះស្រាយករណី Order មិនមាន។

## ៤. Dispatch Message

Inject `MessageBusInterface` ទៅ service ឬ Controller៖

```php
use App\Message\SendOrderConfirmation;
use Symfony\Component\Messenger\MessageBusInterface;

public function completeOrder(
    int $orderId,
    MessageBusInterface $bus,
): void {
    // រក្សាទុក Order ឱ្យបានជោគជ័យមុន dispatch។
    $bus->dispatch(new SendOrderConfirmation($orderId));
}
```

បើមិនបាន route message ទៅ asynchronous transport ទេ handler អាចដំណើរការភ្លាមៗ។ ដូច្នេះគ្រាន់តែ dispatch message មិនមានន័យថាវាកំពុងដំណើរការជា background ទេ។

## ៥. កំណត់ Doctrine Transport និង Routing

Flex recipe អាចបន្ថែម configuration ជាគំរូក្នុង `.env`។ កំណត់ transport DSN ក្នុង `.env.local` សម្រាប់ម៉ាស៊ីន development៖

```dotenv
MESSENGER_TRANSPORT_DSN=doctrine://default
```

ក្នុង `config/packages/messenger.yaml` កំណត់ transport និង routing៖

```yaml
framework:
    messenger:
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'

        routing:
            'App\Message\SendOrderConfirmation': async
```

ឥឡូវ dispatch message ប្រភេទនេះនឹងដាក់ចូល transport `async` ជំនួសឱ្យការហៅ handler ក្នុង request។ បើប្រើ Doctrine transport ជាលើកដំបូង ត្រូវបង្កើតតារាង queue តាមការណែនាំរបស់ package/config ដែលបានដំឡើង។

## ៦. ដំណើរការ Worker

ចាប់ផ្ដើម worker ក្នុង terminal៖

```bash
php bin/console messenger:consume async -vv
```

បន្ទាប់មកបង្កើត Order ក្នុងកម្មវិធី ហើយមើល worker ទទួល និងដំណើរការ message។ Worker គួរដំណើរការជា process ដែលត្រូវបានគ្រប់គ្រង (ឧ. Supervisor ឬ systemd) នៅ production ហើយត្រូវ restart បន្ទាប់ពី deploy តាមវិធីដែលសមស្រប។

## ៧. Retry និង Failed Messages

Handler អាចបរាជ័យដោយសារ network ឬ service ខាងក្រៅ។ កំណត់ retry strategy និង failure transport តាមតម្រូវការក្នុង `messenger.yaml`។ ឧទាហរណ៍៖

```yaml
framework:
    messenger:
        failure_transport: failed

        transports:
            async:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                retry_strategy:
                    max_retries: 3
                    delay: 1000
                    multiplier: 2
            failed: 'doctrine://default?queue_name=failed'
```

Message ដែលបរាជ័យបន្ទាប់ពី retry អាចពិនិត្យ និងដំណើរការឡើងវិញដោយ console commands។ សូមពិនិត្យ commands ដែល version របស់ package ផ្ដល់ឱ្យ៖

```bash
php bin/console messenger:failed:show
php bin/console messenger:failed:retry
```

សម្រាប់ command ដែលបង្ហាញបញ្ជី ឬជ្រើស message មួយចំនួន សូមប្រើ `--help` មុនដំណើរការ។

## ៨. Idempotency និងការប្រុងប្រយ័ត្ន

Message អាចត្រូវបានដំណើរការឡើងវិញ ពេល worker ឬ connection មានបញ្ហា។ Handler ដែលធ្វើសកម្មភាពមិនអាចធ្វើម្ដងទៀតដោយសុវត្ថិភាព ដូចជា charge កាត ឬបញ្ជូន email អាចបង្កើតផលស្ទួន។ រចនា handler ឱ្យ **idempotent**៖ ប្រើ unique operation key, ពិនិត្យស្ថានភាព Order ឬរក្សាកំណត់ត្រាថាសកម្មភាពបានធ្វើរួច។

កុំរំពឹងថា message queue ជំនួស database transaction។ គិតពីអ្វីកើតឡើងបើ database commit រួច តែ dispatch ទៅ queue បរាជ័យ ឬផ្ទុយទៅវិញ។ ប្រព័ន្ធសំខាន់អាចត្រូវការការរចនា transactional outbox។

## កំហុសដែលជួបញឹកញាប់

- **សន្មតថា dispatch មានន័យថា async**៖ ត្រូវមាន transport និង routing rule។
- **មិនដំណើរការ worker**៖ message នៅសល់ក្នុង queue មិនទាន់ត្រូវ handle។
- **Worker ប្រើ environment/config ចាស់**៖ restart worker តាម deployment procedure។
- **Handler មិនអាច serialize message**៖ ផ្ញើទិន្នន័យសាមញ្ញ និង identifier ជំនួស resource/object មិនសមស្រប។
- **Message បរាជ័យហើយមិនតាមដាន**៖ configure failure transport, logs និងការជូនដំណឹង។
- **Handler មានផលប៉ះពាល់ស្ទួនពេល retry**៖ បន្ថែម idempotency guard។

## លំហាត់

1. បង្កើត message `SendOrderConfirmation` ដែលផ្ទុក Order ID។
2. បង្កើត handler ហើយ log ID ដើម្បីឃើញ worker បានទទួល message។
3. កំណត់ Doctrine transport និង route message ទៅ `async`។
4. ដំណើរការ worker ហើយបញ្ជាក់ថា message មិនត្រូវ handle មុន worker consume។
5. ពិពណ៌នាថា handler របស់អ្នកអាចធានា idempotency យ៉ាងដូចម្ដេច។

## សំណួរត្រួតពិនិត្យ

1. តើ Message, Handler និង Transport មានតួនាទីខុសគ្នាដូចម្ដេច?
2. តើត្រូវធ្វើអ្វីដើម្បីឱ្យ message ដំណើរការជា asynchronous?
3. តើ Retry អាចបង្កើតបញ្ហាអ្វីចំពោះ handler?
4. តើ Worker មានតួនាទីអ្វី?
5. តើ failure transport ជួយអ្វី?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Messenger](https://symfony.com/doc/7.4/messenger.html)
- [Symfony 7.4 Doctrine Transport](https://symfony.com/doc/7.4/messenger.html#doctrine-transport)
