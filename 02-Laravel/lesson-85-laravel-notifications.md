# មេរៀនទី ៨៥៖ Laravel Notifications

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពី Notification និង delivery channels។
- បង្កើត Notification class និងជ្រើស channel តាម `via()`។
- ផ្ញើ notification ទៅ User។
- រក្សាទុក database notifications និងបង្ហាញ unread items។
- ដាក់ notification ទៅ queue និងសាកល្បងវាដោយ fake។

## ១. Notification ជាអ្វី?

Notification ជាសារខ្លីដែលប្រាប់អ្នកប្រើអំពីព្រឹត្តិការណ៍ក្នុងកម្មវិធី ដូចជា Course ត្រូវបានបោះពុម្ព ឬការទូទាត់បានសម្រេច។ Laravel អាចបញ្ជូន notification តាម channels ដូចជា mail និង database ហើយមាន community channels ផ្សេងៗទៀត។

Notification មិនដូច Mailable ទេ៖ Mailable តំណាងឱ្យ email ជាក់លាក់ ខណៈ Notification អាចបង្កើត message សម្រាប់ channel ច្រើន។

## ២. បង្កើត Notification Class

បង្កើត class៖

```bash
php artisan make:notification CoursePublishedNotification
```

Notification class មាន `via()` សម្រាប់ជ្រើស channels និង method ដូចជា `toMail()` ឬ `toDatabase()` សម្រាប់បង្កើត message តាម channel។

ឧទាហរណ៍ notification ទៅ email៖

```php
<?php

namespace App\\Notifications;

use App\\Models\\Course;
use Illuminate\\Bus\\Queueable;
use Illuminate\\Notifications\\Messages\\MailMessage;
use Illuminate\\Notifications\\Notification;

class CoursePublishedNotification extends Notification
{
    use Queueable;

    public function __construct(
        public Course $course,
    ) {}

    public function via(object $notifiable): array
    {
        return ['mail', 'database'];
    }

    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
            ->subject('វគ្គសិក្សាថ្មីត្រូវបានផ្សព្វផ្សាយ')
            ->greeting('សួស្តី!')
            ->line("វគ្គ {$this->course->title} អាចចូលរៀនបានហើយ។");
    }

    public function toDatabase(object $notifiable): array
    {
        return [
            'course_id' => $this->course->id,
            'title' => $this->course->title,
            'message' => 'វគ្គសិក្សាថ្មីអាចចូលរៀនបានហើយ។',
        ];
    }
}
```

`via()` ត្រឡប់ channel names។ `toMail()` បង្កើត mail message ហើយ `toDatabase()` ត្រឡប់ array ដែល Laravel រក្សាជា JSON data ក្នុង database notification record។

## ៣. ផ្ញើ Notification

User model របស់ Laravel ជាទូទៅប្រើ `Notifiable` trait។ ផ្ញើ notification ដោយ៖

```php
$user->notify(new CoursePublishedNotification($course));
```

អាចប្រើ Notification facade សម្រាប់ recipients ច្រើន៖

```php
use Illuminate\\Support\\Facades\\Notification;

Notification::send($users, new CoursePublishedNotification($course));
```

កំណត់ recipients តាម authorization និង business rules។ កុំផ្ញើ private Course details ទៅ user ដែលគ្មានសិទ្ធិ។

## ៤. Database Notification Channel

បង្កើត migration សម្រាប់ notifications table៖

```bash
php artisan make:notifications-table
php artisan migrate
```

ទាញ notifications របស់ User៖

```php
$notifications = $user->notifications;
$unread = $user->unreadNotifications;
```

សម្គាល់ notification ថាអានរួច៖

```php
$notification->markAsRead();
```

លុប notification ឬសម្គាល់ទាំងអស់ថាអានរួច គួរត្រូវធ្វើតាមសិទ្ធិរបស់ user ដែលកំពុងចូល។ Notification record អាចមាន resource identifiers ប៉ុន្តែមុនបង្ហាញ resource ត្រូវ authorize វាម្តងទៀត។

## ៥. Queue Notifications

បើ channel ណាមួយចំណាយពេល អាចឱ្យ Notification implements `ShouldQueue` ហើយ Laravel នឹង queue delivery៖

```php
use Illuminate\\Contracts\\Queue\\ShouldQueue;

class CoursePublishedNotification extends Notification implements ShouldQueue
{
    // ...
}
```

ត្រូវមាន queue connection និង worker ដំណើរការ។ ពិនិត្យការរក្សា notification/channel payload ឱ្យមានតែទិន្នន័យចាំបាច់ ហើយប្រុងប្រយ័ត្នចំពោះ transaction timing និង retry side effects។

## ៦. សាកល្បង Notifications

Fake notification facade ដើម្បីកុំផ្ញើ mail ពិតក្នុង test៖

```php
use App\\Notifications\\CoursePublishedNotification;
use Illuminate\\Support\\Facades\\Notification;

Notification::fake();

// ហៅ application code ដែលគួរផ្ញើ notification...

Notification::assertSentTo(
    $user,
    CoursePublishedNotification::class,
);
```

សម្រាប់ notification ដែល queue អាចប្រើ assertion ពាក់ព័ន្ធដូចជា `assertQueued` តាម Laravel version និង testing behavior របស់ project។ បញ្ជាក់ថា test ពិនិត្យ recipient, class និងទិន្នន័យសំខាន់ៗ មិនមែនត្រឹមតែថាមាន notification មួយទេ។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត `CoursePublishedNotification` ដែលប្រើ mail និង database channels។
2. កំណត់ `toDatabase()` ឱ្យរក្សា course ID និង message ខ្លី។
3. ផ្ញើ notification ទៅអ្នកប្រើដែលបាន subscribe វគ្គ។
4. បង្កើតទំព័រ ឬ API សម្រាប់បង្ហាញ `unreadNotifications`។
5. បង្កើត action សម្រាប់ mark-as-read ដោយពិនិត្យថា notification ជារបស់ user ដែល authenticate។
6. សរសេរ test ដោយ `Notification::fake()`។
7. Queue notification ហើយពិនិត្យ worker និង failed jobs នៅ local environment។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ច្រឡំ Notification ជាមួយ Mailable**៖ Notification អាចចែក message ទៅច្រើន channels។
- **មិនបានបង្កើត notifications table**៖ database channel មិនអាចរក្សា records បាន។
- **ដាក់ព័ត៌មានរសើបក្នុង notification data**៖ រក្សាតែទិន្នន័យចាំបាច់ និង authorize មុនបង្ហាញ។
- **ភ្លេចចាត់ចែង unread/read state**៖ UI អាចបង្ហាញសារចាស់ជាថ្មីជានិច្ច។
- **Queue ដោយគ្មាន worker**៖ message អាចនៅ pending មិនទៅដល់ user។
- **សម្គាល់ notification របស់អ្នកដទៃថាអានរួច**៖ ត្រូវ scope query តាម authenticated user's notifications។
- **បញ្ជូនដដែលពេល retry**៖ ពិនិត្យថា notification side effects អាចធ្វើឡើងវិញដោយសុវត្ថិភាព។

## សំណួរត្រួតពិនិត្យ

1. `via()` មានតួនាទីអ្វី?
2. Mailable និង Notification ខុសគ្នាដូចម្ដេច?
3. តើ command ណាបង្កើត database notifications migration?
4. ហេតុអ្វីត្រូវ authorize មុនបង្ហាញ notification payload?
5. តើ `Notification::fake()` ជួយ test ដូចម្ដេច?

## ឯកសារយោង

- [Laravel 13 Notifications](https://laravel.com/docs/13.x/notifications)
- [Laravel 13 Mail](https://laravel.com/docs/13.x/mail)
- [Laravel 13 Queues](https://laravel.com/docs/13.x/queues)
