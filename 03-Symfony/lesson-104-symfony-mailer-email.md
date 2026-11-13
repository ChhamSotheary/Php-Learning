# មេរៀនទី ១០៤៖ Symfony Mailer និងការផ្ញើ Email

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- រៀបចំ Mailer transport តាម environment variable។
- បង្កើត និងផ្ញើ Email ដោយ Symfony Mailer។
- បង្ហាញ HTML email ដោយ Twig template។
- សាកល្បង email ក្នុង development និងយល់ពី async sending។

## ១. ដំឡើង Mailer

ក្នុង Symfony project ដែលប្រើ Flex៖

```bash
composer require symfony/mailer
```

Package នេះផ្ដល់ Mailer សម្រាប់រៀបចំ និងបញ្ជូន email។ ក្នុង development ប្រើ email catcher ឬ provider sandbox ដើម្បីកុំឱ្យផ្ញើសារសាកល្បងទៅអ្នកទទួលពិត។

## ២. រៀបចំ Transport និង Credentials

Mailer ប្រើ DSN ដើម្បីដឹងថាត្រូវភ្ជាប់ទៅ SMTP server ឬ provider ណា។ កំណត់ DSN ក្នុង `.env.local` ជាមួយតម្លៃរបស់អ្នក៖

```dotenv
MAILER_DSN=smtp://user:password@smtp.example.test:587
```

Symfony recipe ជាទូទៅកំណត់ config ក្នុង `config/packages/mailer.yaml`៖

```yaml
framework:
    mailer:
        dsn: '%env(MAILER_DSN)%'
```

ជំនួស host/user/password ទៅតាម SMTP provider។ កុំ commit credentials ពិតទៅ Git; រក្សាទុកក្នុង `.env.local` សម្រាប់ local និង secret manager/environment variables នៅ deployment។ តួអក្សរពិសេសនៅក្នុង DSN credentials ត្រូវ URL-encode។

## ៣. បង្កើត និងផ្ញើ Email

Inject `MailerInterface` ទៅ service ឬ controller៖

```php
<?php

namespace App\Service;

use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Mime\Email;

final class WelcomeMailer
{
    public function __construct(
        private MailerInterface $mailer,
    ) {
    }

    public function send(string $recipient, string $name): void
    {
        $email = (new Email())
            ->from('no-reply@example.test')
            ->to($recipient)
            ->subject('សូមស្វាគមន៍')
            ->text('សួស្តី '.$name.' សូមស្វាគមន៍មកកាន់វគ្គសិក្សា PHP។');

        $this->mailer->send($email);
    }
}
```

`from()`, `to()`, `subject()` និង `text()` កំណត់អ្នកផ្ញើ អ្នកទទួល ចំណងជើង និង body ជា text។ ក្នុង app ពិត ត្រូវផ្ទៀងផ្ទាត់អាសយដ្ឋានអ្នកទទួល និងប្រើ sender domain ដែល provider អនុញ្ញាត។

## ៤. បង្កើត HTML Email ដោយ Twig

សម្រាប់ HTML email អាចប្រើ `TemplatedEmail`។ បើមិនទាន់មាន Twig integration សូមដំឡើង package ដែលសមស្របតាម project៖

```bash
composer require symfony/twig-bundle
```

ឧទាហរណ៍៖

```php
use Symfony\Bridge\Twig\Mime\TemplatedEmail;

$email = (new TemplatedEmail())
    ->from('no-reply@example.test')
    ->to($recipient)
    ->subject('សូមស្វាគមន៍')
    ->htmlTemplate('emails/welcome.html.twig')
    ->context([
        'name' => $name,
    ]);

$mailer->send($email);
```

Template `templates/emails/welcome.html.twig`៖

```twig
<h1>សួស្តី {{ name }}</h1>
<p>សូមស្វាគមន៍មកកាន់វគ្គសិក្សា PHP។</p>
```

Twig escape values ដោយស្វ័យប្រវត្តិ។ កុំប្រើ `raw` លើទិន្នន័យដែលអ្នកប្រើផ្ដល់ លុះត្រាតែបានសម្អាត HTML តាមវិធីដែលទុកចិត្តបាន។

## ៥. សាកល្បង Email

សាកល្បងដោយប្រើ local email catcher ឬ sandbox provider។ កុំប្រើអាសយដ្ឋាន និងព័ត៌មានសម្ងាត់ពិតរបស់អ្នកក្នុងឯកសារ source។ ពិនិត្យ Symfony Profiler នៅ development ដើម្បីមើល email ដែលបានបង្កើត និង errors។

បើ provider ត្រូវការ credentials សូមដាក់ DSN ក្នុង `.env.local` ឬ secret store ដែលត្រូវនឹង environment។ `MAILER_DSN=null://null` អាចបិទការបញ្ជូនសម្រាប់ environment ដែលមិនចង់ផ្ញើ email ប៉ុន្តែសូមពិនិត្យ syntax និង config របស់ Symfony version របស់អ្នក។

## ៦. ផ្ញើ Email ជា Background

ការផ្ញើ SMTP/API អាចធ្វើឱ្យ HTTP request រង់ចាំ។ អាចប្រើ Symfony Messenger ដើម្បីដាក់ email ទៅ queue ហើយផ្ញើតាម worker។ Mailer អាចភ្ជាប់ជាមួយ Messenger តាម configuration; សូមពិនិត្យ Symfony Mailer និង Messenger docs ព្រោះ message serialization/attachment អាចប៉ះពាល់ដល់ queue។

អត្ថប្រយោជន៍គឺ response ទៅអ្នកប្រើលឿនជាង ប៉ុន្តែត្រូវដំណើរការ worker និងតាមដាន failed messages។

## កំហុសដែលជួបញឹកញាប់

- **Connection/authentication failed**៖ ពិនិត្យ DSN, port, username, password និង provider config។
- **Credentials មិនត្រូវបានអាន**៖ ពិនិត្យ `MAILER_DSN` ក្នុង environment ដែលកំពុងប្រើ។
- **Email មិនបង្ហាញ HTML**៖ ប្រើ `html()` ឬ `htmlTemplate()` ត្រឹមត្រូវ។
- **Email ស្ទួន**៖ ពិនិត្យថាតើ code ត្រូវបានហៅម្ដងឬច្រើន និងគិតពី retry/queue behavior។
- **ដាក់ secret ក្នុង Git**៖ ផ្លាស់ទៅ `.env.local` ឬ secret manager ហើយ rotate credential ប្រសិនបើវាបាន commit រួច។
- **ផ្ញើ email ចេញពី test/dev**៖ ប្រើ email catcher ឬ sandbox មុនប្រើ transport production។

## លំហាត់

1. បង្កើត service `WelcomeMailer` ដែលទទួល `MailerInterface` តាម constructor។
2. បង្កើត text email និង HTML email template ដែលបង្ហាញឈ្មោះអ្នកទទួល។
3. រៀបចំ email catcher/sandbox ហើយផ្ញើសារសាកល្បង។
4. ប្ដូរឱ្យ email ត្រូវ dispatch ជា background តាម Messenger និងសាកល្បង failed message។
5. ពន្យល់ពីកន្លែងរក្សាទុក MAILER_DSN សម្រាប់ local និង production។

## សំណួរត្រួតពិនិត្យ

1. តើ Mailer DSN កំណត់អ្វី?
2. តើ `MailerInterface` ជួយអ្វីក្នុង service?
3. ហេតុអ្វីមិនគួរ commit SMTP credentials?
4. តើ Messenger ជួយដល់ការផ្ញើ email យ៉ាងដូចម្ដេច?
5. តើហេតុអ្វីត្រូវមាន email catcher ក្នុង development?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Mailer](https://symfony.com/doc/7.4/mailer.html)
- [Symfony 7.4 Messenger](https://symfony.com/doc/7.4/messenger.html)
- [Symfony 7.4 Twig](https://symfony.com/doc/7.4/templates.html)
