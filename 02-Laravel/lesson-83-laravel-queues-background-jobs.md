# មេរៀនទី ៨៣៖ Laravel Queues និង Background Jobs

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីមូលហេតុដាក់ការងារយូរទៅ queue។
- បង្កើត Job ដែល implement `ShouldQueue`។
- Dispatch Job ហើយដំណើរការ queue worker។
- កំណត់ retry/backoff និងពិនិត្យ failed jobs។
- សរសេរ job ដែលអាច retry ដោយមិនបង្កើត side effects ស្ទួន។

## ១. ហេតុអ្វីប្រើ Queue?

ការងារដូចជា ផ្ញើ email, បង្កើត report ធំ, បម្លែងរូបភាព ឬហៅ external service អាចចំណាយពេល។ បើធ្វើក្នុង HTTP request ដោយផ្ទាល់ អ្នកប្រើអាចរង់ចាំយូរ ឬ request timeout។

Queue អនុញ្ញាតឱ្យ request បញ្ជូនការងារទៅ worker ហើយត្រឡប់ response ឱ្យអ្នកប្រើបានឆាប់។ Worker ដំណើរការ job នៅ background។

## ២. កំណត់ Queue Connection

Laravel កំណត់ queue connections ក្នុង `config/queue.php` ហើយជ្រើស connection តាម `QUEUE_CONNECTION`។ អាចប្រើ database queue, Redis ឬ driver ផ្សេងដែល project បានដំឡើង។ ពិនិត្យ configuration និង migrations របស់ project មុនប្រើ។

សម្រាប់ database queue ប្រសិនបើ project មិនទាន់មាន jobs table migration អាចប្រើ៖

```bash
php artisan make:queue-table
php artisan migrate
```

កុំសន្មតថា production និង local ប្រើ driver ដូចគ្នា។ កំណត់ connection និង worker ឱ្យស្របនឹង infra ដែល deploy ពិត។

## ៣. បង្កើត Job

បង្កើត job៖

```bash
php artisan make:job SendCoursePublishedNotice
```

Job ដែល implement `ShouldQueue` នឹងត្រូវ queue ជំនួសឱ្យដំណើរការភ្លាមៗ៖

```php
<?php

namespace App\\Jobs;

use App\\Models\\Course;
use Illuminate\\Contracts\\Queue\\ShouldQueue;
use Illuminate\\Foundation\\Queue\\Queueable;
use Illuminate\\Bus\\Dispatchable;
use Illuminate\\Queue\\InteractsWithQueue;
use Illuminate\\Queue\\SerializesModels;

class SendCoursePublishedNotice implements ShouldQueue
{
    use Dispatchable;
    use InteractsWithQueue;
    use Queueable;
    use SerializesModels;

    public function __construct(
        public Course $course,
    ) {}

    public function handle(): void
    {
        // ផ្ញើ notification ឬហៅ service ដែលបានចាក់ចូលតាម dependency injection
    }
}
```

អត្ថបទ body ក្នុង `handle()` ជាកន្លែងធ្វើការងារ។ កុំរក្សា connection/socket ឬ secret ដែលផុតសុពលភាពនៅក្នុង serialized job payload។ បញ្ជូន identifiers ឬ Model ដែលចាំបាច់ប៉ុណ្ណោះ។

## ៤. Dispatch Job

បន្ទាប់ពី Course ត្រូវបាន publish៖

```php
SendCoursePublishedNotice::dispatch($course);
```

អាចជ្រើស queue name ឬ delay ប្រសិនបើ connection គាំទ្រ៖

```php
SendCoursePublishedNotice::dispatch($course)
    ->onQueue('notifications');
```

បើ database transaction កំពុងបង្កើត Course និង dispatch job ពេលតែមួយ សូមពិនិត្យការបញ្ជូន job ក្រោយ transaction commit ដើម្បីជៀសវាង worker រក Model មិនឃើញមុនទិន្នន័យត្រូវបាន commit។

## ៥. ដំណើរការ Worker និង Retry

ចាប់ផ្តើម worker ក្នុង terminal សម្រាប់ local development៖

```bash
php artisan queue:work
```

អាចកំណត់ចំនួន attempts និង backoff៖

```bash
php artisan queue:work --tries=3 --backoff=5
```

Worker ជា long-running process។ នៅ production ត្រូវគ្រប់គ្រងដោយ process manager ឬ platform ដែលអាច restart វាបាន។ ក្រោយ deploy កូដថ្មី ត្រូវ reload/restart workers ដើម្បីឱ្យវាផ្ទុក code ថ្មី។

កំណត់ timeout និង `retry_after` ឱ្យស្របគ្នា។ Timeout របស់ worker គួរខ្លីជាង `retry_after` ដើម្បីកាត់បន្ថយហានិភ័យឱ្យ job ដដែលរត់ស្របគ្នាពីរដង។

## ៦. Failed Jobs និង Idempotency

បើ job បរាជ័យលើសចំនួន attempts វាអាចត្រូវកត់ក្នុង failed jobs storage (បើបានកំណត់)។ ពិនិត្យ និង retry៖

```bash
php artisan queue:failed
php artisan queue:retry all
```

Retry មានន័យថា `handle()` អាចដំណើរការម្តងហើយម្តងទៀត។ ដូច្នេះរចនា Job ឱ្យ **idempotent**៖ ការដំណើរការដដែលច្រើនដងមិនគួរបង្កើតការទូទាត់ ឬ side effects ស្ទួន។ ឧទាហរណ៍ រក្សា provider event ID ឬ job key មួយដែលអាចប្រើដើម្បីពិនិត្យថាការងារនោះបានធ្វើរួចឬនៅ។

កុំបិទ error ដោយ retry គ្មានដែនកំណត់។ កំណត់ attempts, backoff និងការតាមដាន failed jobs ឱ្យសមស្រប។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត `SendCoursePublishedNotice` ដែល implement `ShouldQueue`។
2. Dispatch Job ពេល Course ត្រូវបាន publish។
3. កំណត់ database ឬ Redis queue connection ក្នុង local environment។
4. ដំណើរការ `php artisan queue:work` ហើយសង្កេត worker output។
5. បន្ថែម retry/backoff និងពិនិត្យ failed jobs។
6. រចនាការការពារកុំឱ្យ Job ផ្ញើ notification ស្ទួនពេល retry។
7. ពន្យល់ភាពខុសគ្នារវាង synchronous និង queued job។

## ៨. កំហុសដែលជួបញឹកញាប់

- **មិនដំណើរការ worker**៖ dispatch បាន ប៉ុន្តែ job នៅក្នុង queue មិនត្រូវបាន process។
- **ភ្លេច configure queue connection/table**៖ ពិនិត្យ `QUEUE_CONNECTION`, config និង migration របស់ driver។
- **Job មិន idempotent**៖ retry អាចបង្កើត email ឬការទូទាត់ស្ទួន។
- **Serialize ទិន្នន័យមិនចាំបាច់**៖ payload ធំ និងមានព័ត៌មានរសើបអាចបង្កហានិភ័យ។
- **កំណត់ timeout លើស retry_after**៖ អាចឱ្យ job ដដែលត្រូវ retry ខណៈ attempt មុននៅកំពុងដំណើរការ។
- **គិតថា dispatch មានន័យថាបានជោគជ័យ**៖ ត្រូវតាមដាន worker និង failed jobs។
- **មិន restart worker ក្រោយ deploy**៖ long-running workers អាចបន្តប្រើ code ចាស់។

## សំណួរត្រួតពិនិត្យ

1. តើ Queue ជួយ web request យ៉ាងដូចម្ដេច?
2. `ShouldQueue` ប្រាប់ Laravel អ្វី?
3. ហេតុអ្វី Job ត្រូវអាច retry ដោយមានសុវត្ថិភាព?
4. Worker មានតួនាទីអ្វី?
5. ហេតុអ្វី worker timeout គួរខ្លីជាង `retry_after`?

## ឯកសារយោង

- [Laravel 13 Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 Database Queues](https://laravel.com/docs/13.x/queues#database)
- [Laravel 13 Deployment](https://laravel.com/docs/13.x/deployment)
