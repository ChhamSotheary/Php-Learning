# មេរៀនទី ៨៨៖ Laravel Events និង Listeners

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីអត្ថប្រយោជន៍នៃ Event និង Listener។
- បង្កើត Event និង Listener ដោយ Artisan។
- dispatch Event ពេល business event កើតឡើង។
- ឱ្យ Laravel ស្វែងរក Listener តាម convention។
- ដាក់ Listener ឱ្យដំណើរការក្រោយ database commit ឬតាម queue។

## ១. Event និង Listener ជាអ្វី?

**Event** តំណាងឱ្យអ្វីមួយដែលបានកើតឡើងក្នុងកម្មវិធី ដូចជា Course ត្រូវបាន publish។ **Listener** ឆ្លើយតបទៅ Event នោះ ដូចជា ផ្ញើ email ឬកត់ audit record។

Event ជួយកាត់បន្ថយការភ្ជាប់គ្នាខ្លាំងរវាង business flow និង side effects។ Event មួយអាចមាន listeners ច្រើនដែលធ្វើការដោយឯករាជ្យ។

ឧទាហរណ៍៖

```text
Course ត្រូវបាន publish
   ├─ listener ផ្ញើ notification
   ├─ listener បង្កើត audit log
   └─ listener បោស cache ចាស់
```

កុំប្រើ events ដើម្បីលាក់ control flow សំខាន់រហូតអ្នកអានមិនអាចដឹងថាអ្វីកើតឡើង។ ឈ្មោះ event គួរពិពណ៌នាអ្វីដែលបានកើតឡើង ហើយ side effects គួរត្រូវបានតាមដានបាន។

## ២. បង្កើត Event និង Listener

បង្កើត Event៖

```bash
php artisan make:event CoursePublished
```

បង្កើត Listener ភ្ជាប់ជាមួយ Event៖

```bash
php artisan make:listener SendCoursePublishedNotification --event=CoursePublished
```

Event class អាចផ្ទុក Model ដែលពាក់ព័ន្ធ៖

```php
<?php

namespace App\\Events;

use App\\Models\\Course;

class CoursePublished
{
    public function __construct(
        public Course $course,
    ) {}
}
```

Listener ប្រើ `handle()` ដើម្បីធ្វើ side effect៖

```php
<?php

namespace App\\Listeners;

use App\\Events\\CoursePublished;

class SendCoursePublishedNotification
{
    public function handle(CoursePublished $event): void
    {
        $course = $event->course;

        // ជ្រើស recipients ដែលមានសិទ្ធិ និងផ្ញើ notification
    }
}
```

## ៣. Dispatch Event និង Discovery

Laravel ជាទូទៅស្វែងរក Listener ដោយស្វ័យប្រវត្តិពី `app/Listeners` តាម type hint ក្នុង `handle()` ឬ `__invoke()`។

dispatch Event ក្រោយការផ្លាស់ប្តូរបានសម្រេច៖

```php
use App\\Events\\CoursePublished;

CoursePublished::dispatch($course);
```

ឬអាចប្រើ event helper តាម application style៖

```php
event(new CoursePublished($course));
```

បើ Event/Listener មិនត្រូវបានរកឃើញ សូមពិនិត្យ namespace, class name, method type hint និង event cache។ ក្នុង production, `php artisan optimize` ឬ `php artisan event:cache` បង្កើត event listener manifest សម្រាប់ដំណើរការលឿន។

## ៤. Listener ដែលដំណើរការតាម Queue

បើ Listener ផ្ញើ email ឬហៅ external API ដែលយឺត អាច implement `ShouldQueue`៖

```php
use Illuminate\\Contracts\\Queue\\ShouldQueue;

class SendCoursePublishedNotification implements ShouldQueue
{
    public function handle(CoursePublished $event): void
    {
        // ផ្ញើ notification ឬហៅ service
    }
}
```

ត្រូវមាន queue connection និង worker ដំណើរការ។ Queue listener អាច retry ដូច jobs ផ្សេងៗ ដូច្នេះគួររចនា side effects ឱ្យគ្រប់គ្រងការធ្វើឡើងវិញបាន។

## ៥. Database Transactions និង Events

បើ Listener ត្រូវ queue ហើយវាអាស្រ័យលើ record ដែលទើបបង្កើតក្នុង database transaction, worker អាចទទួល Event មុន transaction commit។ ពេលនោះ worker អាចរក record មិនឃើញ ឬមើលឃើញទិន្នន័យចាស់។

ជម្រើសមួយគឺឱ្យ Listener implement `ShouldQueueAfterCommit` ប្រសិនបើ project configuration មិនកំណត់ `after_commit` រួច៖

```php
use Illuminate\\Contracts\\Queue\\ShouldQueueAfterCommit;

class SendCoursePublishedNotification implements ShouldQueueAfterCommit
{
    public function handle(CoursePublished $event): void
    {
        // ដំណើរការក្រោយ transaction commit
    }
}
```

ប្រើតាមតម្រូវការពិត និងពិនិត្យ queue connection configuration របស់ project មុនចម្លង code។

## ៦. លំហាត់អនុវត្ត

1. បង្កើត `CoursePublished` Event ដែលមាន Course។
2. បង្កើត Listener `SendCoursePublishedNotification`។
3. Dispatch Event បន្ទាប់ពី Course ត្រូវបាន publish ជោគជ័យ។
4. បន្ថែម Listener ទីពីរសម្រាប់ audit log ដោយកុំដាក់វាក្នុង listener notification ដដែល។
5. ប្តូរ notification listener ទៅ queued listener។
6. សាកល្បងដំណើរការ worker និងកត់ត្រាថាតើ Event និង Listener ត្រូវបានរកឃើញតាម convention ដែរឬទេ។
7. ពន្យល់បញ្ហាដែលអាចកើតឡើងបើ queued listener រត់មុន database commit។

## ៧. កំហុសដែលជួបញឹកញាប់

- **Event មិនបង្ហាញថាអ្វីបានកើតឡើង**៖ ប្រើឈ្មោះដូច `CoursePublished` ជំនួសឈ្មោះ command ដូច `PublishCourse`។
- **ដាក់ logic ទាំងអស់ក្នុង Listener តែមួយ**៖ បំបែក side effects ដែលមិនពាក់ព័ន្ធ ដើម្បីអាចថែទាំដោយឡែក។
- **ភ្លេច queue worker**៖ Listener ដែល implements `ShouldQueue` មិនដំណើរការដោយគ្មាន worker។
- **Listener មិនត្រូវបាន discover**៖ ពិនិត្យ namespace/type hint និង clear ឬ rebuild event cache ពេលចាំបាច់។
- **dispatch មុន transaction commit**៖ listener អាចអានទិន្នន័យមិនទាន់ commit។
- **មិនគ្រប់គ្រង retry**៖ queued listener អាចរត់ច្រើនដង; ការផ្ញើសារ ឬប្រតិបត្តិការបង់ប្រាក់ត្រូវការការការពារស្ទួន។
- **ប្រើ Event ដើម្បីលាក់ business decision**៖ business flow សំខាន់គួរអានយល់ និងត្រួតពិនិត្យបាន។

## សំណួរត្រួតពិនិត្យ

1. Event តំណាងអ្វី ហើយ Listener មានតួនាទីអ្វី?
2. តើ Laravel រកឃើញ Listener តាម convention ដោយរបៀបណា?
3. តើពេលណាគួរ implement `ShouldQueue`?
4. `ShouldQueueAfterCommit` ជួយបញ្ហាអ្វី?
5. ហេតុអ្វី Listener ដែល retry ត្រូវគិតពី idempotency?

## ឯកសារយោង

- [Laravel 13 Events](https://laravel.com/docs/13.x/events)
- [Laravel 13 Queues](https://laravel.com/docs/13.x/queues)
- [Laravel 13 Database Transactions](https://laravel.com/docs/13.x/database#database-transactions)
