# មេរៀនទី ៨៤៖ Laravel Mailables និងការផ្ញើ Email

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- រៀបចំ mail transport តាម environment។
- បង្កើត Mailable class ដើម្បីរៀបចំ email។
- បញ្ជូន data ទៅ email view។
- ផ្ញើ ឬ queue email។
- សាកល្បង mail behavior ដោយមិនផ្ញើ email ពិត។

## ១. Mail Configuration

Laravel mail configuration ស្ថិតក្នុង `config/mail.php`។ Mailer ជ្រើស transport ដូចជា SMTP ឬ provider ផ្សេងដែល project បានកំណត់។ តម្លៃជាក់លាក់ដូចជា mail host, username និង password គួររក្សាក្នុង environment variables ឬ secret manager; កុំ commit credentials ទៅ Git។

សម្រាប់ local development អាចប្រើ mail testing service ឬ mail catcher ដើម្បីពិនិត្យ email ដោយមិនផ្ញើទៅអ្នកប្រើពិត។ កុំប្រើ real customer email addresses សម្រាប់សាកល្បង។

## ២. បង្កើត Mailable

ក្នុង Laravel, email ប្រភេទនីមួយៗត្រូវបានតំណាងដោយ Mailable class។ បង្កើតវាដោយ៖

```bash
php artisan make:mail CoursePublished
```

Mailable class នៅ `app/Mail/CoursePublished.php`។ កំណត់ទិន្នន័យដែលចាំបាច់ និង view ដែលប្រើសម្រាប់បង្កើត email៖

```php
<?php

namespace App\\Mail;

use App\\Models\\Course;
use Illuminate\\Bus\\Queueable;
use Illuminate\\Mail\\Mailable;
use Illuminate\\Queue\\SerializesModels;

class CoursePublished extends Mailable
{
    use Queueable;
    use SerializesModels;

    public function __construct(
        public Course $course,
    ) {}

    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'វគ្គសិក្សាថ្មីត្រូវបានផ្សព្វផ្សាយ',
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'mail.courses.published',
        );
    }
}
```

បើប្រើ return types ក្នុងឧទាហរណ៍ សូម import `Illuminate\\Mail\\Mailables\\Content` និង `Envelope`៖

```php
use Illuminate\\Mail\\Mailables\\Content;
use Illuminate\\Mail\\Mailables\\Envelope;
```

បង្កើត Blade view `resources/views/mail/courses/published.blade.php`៖

```blade
<h1>វគ្គសិក្សាថ្មី៖ {{ $course->title }}</h1>
<p>{{ $course->description }}</p>
```

Blade escaped output ដោយស្វ័យប្រវត្តិ។ កុំប្រើ unescaped output លើ user-provided course text។

## ៣. ផ្ញើ Email

ប្រើ Mail facade ដើម្បីផ្ញើ៖

```php
use App\\Mail\\CoursePublished;
use Illuminate\\Support\\Facades\\Mail;

Mail::to($recipient)->send(
    new CoursePublished($course)
);
```

`$recipient` គួរត្រូវបានទាញពី user ឬ business record ដែលបានផ្ទៀងផ្ទាត់ មិនមែនទុកចិត្ត email address ដែល client អាចកែដោយគ្មាន authorization ទេ។

## ៤. Queue Email

ការផ្ញើ email អាចចំណាយពេល ដូច្នេះអាចដាក់ Mailable ទៅ queue៖

```php
Mail::to($recipient)->queue(
    new CoursePublished($course)
);
```

ត្រូវមាន queue connection និង worker កំពុងដំណើរការ។ Mailable ដែល implement `ShouldQueue` នឹងត្រូវ queue ទោះបីប្រើ `send()` ក៏ដោយ។ ប្រសិនបើ email អាស្រ័យលើ Model ដែលទើបបង្កើតក្នុង database transaction សូមដាក់វាឱ្យ dispatch ក្រោយ transaction commit ដើម្បីកុំឱ្យ worker អានទិន្នន័យមុនពេល commit។

## ៥. សាកល្បង Mail

ក្នុង test អាច fake mailer ដើម្បីមិនផ្ញើ email ពិត៖

```php
use App\\Mail\\CoursePublished;
use Illuminate\\Support\\Facades\\Mail;

Mail::fake();

// ហៅ code path ដែលគួរផ្ញើ email...

Mail::assertSent(CoursePublished::class);
```

បើ Mailable ត្រូវ queue សូមប្រើ `Mail::assertQueued()` ជំនួស `assertSent()`។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត `CoursePublished` Mailable ដែលទទួល Course។
2. បង្កើត Blade email view ដែលបង្ហាញ title និង description ដោយ escaped output។
3. ផ្ញើ Mailable ទៅ address សាកល្បងដោយ Mail facade។
4. ប្តូរឱ្យ email ត្រូវបាន queue ហើយសាកល្បងវាជាមួយ worker local។
5. សរសេរ test ដោយ `Mail::fake()` និង assert ថា Mailable ត្រូវបានផ្ញើ ឬ queue។
6. ពន្យល់ថាហេតុអ្វីមិនត្រូវ commit SMTP password។

## ៧. កំហុសដែលជួបញឹកញាប់

- **ដាក់ mail credentials ក្នុង source code**៖ ប្រើ environment configuration និង secrets management។
- **សាកល្បងផ្ញើទៅអតិថិជនពិត**៖ ប្រើ mail catcher ឬ sandbox provider។
- **គិតថា `queue()` ផ្ញើភ្លាមៗ**៖ worker ត្រូវដំណើរការ ហើយ job អាច fail។
- **Queue email មុន transaction commit**៖ worker អាចមិនឃើញ Model/changes ថ្មី។
- **បញ្ចូល user content ដោយ unescaped Blade**៖ បង្កហានិភ័យ HTML injection ឬ phishing-like content។
- **ប្រើ `assertSent()` សម្រាប់ queued mail**៖ ប្រើ `assertQueued()` តាម behavior។

## សំណួរត្រួតពិនិត្យ

1. Mailable class មានតួនាទីអ្វី?
2. តើ mail credentials គួររក្សាទុកនៅឯណា?
3. `send()` និង `queue()` ខុសគ្នាដូចម្ដេច?
4. ហេតុអ្វីត្រូវ queue mail ក្រោយ database transaction commit?
5. តើ test អាចបញ្ជាក់ mail behavior ដោយមិនផ្ញើ email ពិតយ៉ាងដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 Mail](https://laravel.com/docs/13.x/mail)
- [Laravel 13 Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 Mail Testing](https://laravel.com/docs/13.x/mail#testing)
