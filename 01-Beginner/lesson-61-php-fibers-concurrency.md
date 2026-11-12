# មេរៀនទី ៦១៖ PHP Fibers និង Cooperative Concurrency

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពី Fiber និងការផ្អាក/បន្ត Execution ក្នុង PHP។
- ប្រើ `Fiber::start()`, `Fiber::suspend()`, `resume()` និង `getReturn()`។
- បែងចែក Fiber ពី Generator និង Process/Thread Parallelism។
- យល់ពីតួនាទី Event Loop ឬ Async Library ក្នុងការសម្រេច Concurrency។
- ជៀសវាងគិតថា Fiber ធ្វើឱ្យ Blocking I/O ក្លាយជា Async ដោយស្វ័យប្រវត្តិ។

> Fibers មានចាប់ពី PHP 8.1។ ឧទាហរណ៍នេះបង្ហាញការផ្អាក/បន្តដោយដៃ មិនមែន Production Scheduler ទេ។

## ១. Fiber ជាអ្វី?

Fiber ជា Execution Stack ដែលអាចផ្អាកនៅចំណុចណាមួយ ហើយបន្តពេលក្រោយ។ Function នៅក្នុង Fiber អាចហៅ Function ផ្សេងៗជាច្រើនជាន់ រួចហៅ `Fiber::suspend()` ដើម្បីផ្អាក។ ក្រោយមក Caller អាច `resume()` វា។

Fiber ផ្តល់ Primitive សម្រាប់ Cooperative Concurrency ប៉ុន្តែ PHP មិនបង្កើត Event Loop ឬដំណើរការ Code ស្របគ្នាដោយស្វ័យប្រវត្តិទេ។ Scheduler/Library ត្រូវសម្រេចថា Fiber ណាដំណើរការបន្ទាប់។

## ២. Start, Suspend និង Resume

```php
<?php
$fiber = new Fiber(function (): string {
    echo "Fiber: start\n";

    $resumeValue = Fiber::suspend('paused');

    echo 'Fiber resumed with: ' . $resumeValue . "\n";

    return 'completed';
});

$suspendedValue = $fiber->start();
echo 'Main received: ' . $suspendedValue . "\n";

$fiber->resume('continue');
echo 'Fiber return: ' . $fiber->getReturn() . "\n";
```

លំដាប់ដំណើរការ៖

1. `start()` ចាប់ផ្តើម Fiber រហូតដល់វាបញ្ចប់ ឬ Suspend។
2. តម្លៃដែលផ្ញើទៅ `Fiber::suspend($value)` ត្រឡប់ទៅ Caller។
3. `resume($value)` បន្ត Fiber; តម្លៃនោះក្លាយជាលទ្ធផលរបស់ `Fiber::suspend()` ខាងក្នុង Fiber។
4. `getReturn()` ទទួល Return Value ក្រោយ Fiber បានបញ្ចប់។

កុំហៅ `resume()` លើ Fiber ដែលមិនស្ថិតក្នុង Suspended State។ ពិនិត្យ `isStarted()`, `isSuspended()`, `isTerminated()` តាម Lifecycle របស់អ្នក។

## ៣. ផ្អាកនៅក្នុង Function ជាន់ខាងក្នុង

Fiber មាន Full Call Stack ដូច្នេះ Function ខាងក្នុងអាចជាចំណុច Suspend៖

```php
<?php
function waitForScheduler(string $taskName): string
{
    return Fiber::suspend('waiting:' . $taskName);
}

$fiber = new Fiber(function (): void {
    $result = waitForScheduler('download');
    echo 'Scheduler returned: ' . $result . PHP_EOL;
});

echo $fiber->start() . PHP_EOL; // waiting:download
$fiber->resume('ready');
```

Function ដែលអាច Suspend មិនចាំបាច់ត្រឡប់ Generator ដូច Function ដែលប្រើ `yield` ទេ។ ប៉ុន្តែអ្នកហៅវាត្រូវដឹងថា Execution អាចផ្អាក និងបន្តក្រោយមក។

## ៤. Fibers ខុសពី Generators និង Parallelism

- **Generator**៖ ផ្តល់តម្លៃតាមលំដាប់ ដោយ `yield`; សមស្របសម្រាប់ Iterable/Streaming។
- **Fiber**៖ ផ្អាក និងបន្ត Call Stack; សមស្របជាប្រភពបង្កើត Async Runtime/Library។
- **Parallelism**៖ ការងាររត់ពេលតែមួយលើ CPU Core/Process/Thread ច្រើន។ Fiber ក្នុង PHP តែមួយមិនមែនជា Thread ហើយមិនធ្វើ CPU-bound Work ឱ្យរត់ស្របគ្នាដោយស្វ័យប្រវត្តិទេ។

Fiber មួយដែលហៅ Function I/O ដែល Block នឹងនៅតែ Block Thread/Process រហូត Function នោះត្រឡប់ លុះត្រាតែ Async I/O Library/Scheduler អាចចាប់ការរង់ចាំ ហើយប្តូរទៅ Fiber ផ្សេង។

## ៥. Event Loop ជួយអ្វី?

Async Runtime អាច៖

1. ចាប់ផ្តើម Fiber សម្រាប់ Task នីមួយៗ។
2. ពេល Task រង់ចាំ Network/Timer, Suspend Fiber។
3. Scheduler ធ្វើ Task ផ្សេង ឬរង់ចាំ Event ពី Socket។
4. ពេល I/O រួច, Resume Fiber ដែលពាក់ព័ន្ធ។
5. ប្រមូល Result/Error ពេល Task ទាំងអស់បញ្ចប់។

ត្រូវប្រើ Library ដែលរចនាឡើងសម្រាប់ Runtime នោះ។ កុំលាយ Blocking API ជាមួយ Async Flow ដោយមិនយល់ថាវានឹង Block Event Loop ឬអត់។

## ៦. Error Handling និង Lifecycle

Fiber ដែលបោះ Exception អាចបញ្ជូន Exception ទៅ Caller ពេល Start/Resume; Scheduler ត្រូវចាប់ និងដោះស្រាយវាដោយច្បាស់។ Fiber មិនមែនជា Background Job ដែលរស់នៅក្រោយ HTTP Request បញ្ចប់ទេ។ សម្រាប់ការងារដែលត្រូវបន្តក្រោយ Response ប្រើ Queue/Worker ដូចមេរៀនទី ៥៣។

មុនប្រើ Fiber ក្នុង Application សូមកំណត់៖

- តើអ្នកណាជា Scheduler និងគ្រប់គ្រង Lifecycle?
- តើ Fiber អាច Suspend នៅចំណុចណា?
- តើ Exception/Cancel/Timeout ត្រូវដោះស្រាយយ៉ាងដូចម្តេច?
- តើ Dependency ទាំងអស់ Non-blocking ឬមាន Async Adapter?
- តើ Context/Request State អាចបែកចែករវាង Task ដោយសុវត្ថិភាពឬទេ?

## ៧. កំហុសដែលជួបញឹកញាប់

- គិតថា Fiber បង្កើត Thread ឬដំណើរការ CPU កិច្ចការស្របគ្នា។
- ហៅ `Fiber::suspend()` តែគ្មាន Scheduler មក Resume។
- ប្រើ Blocking HTTP/Database Call ហើយរំពឹងថា Fiber ផ្សេងនឹងដំណើរការក្នុងពេលរង់ចាំ។
- Resume Fiber នៅ State មិនត្រឹមត្រូវ។
- ប្រើ Fiber ជំនួស Queue សម្រាប់ Background Job ឆ្លង Request។
- មិនចាប់ Exception ឬគ្មាន Timeout/Cancel Policy។

## ៨. លំហាត់

1. រត់ឧទាហរណ៍ Start/Suspend/Resume ហើយកត់លំដាប់ Output។
2. បង្កើត Fiber ពីរ ដែល Suspend ម្តងម្នាក់ ហើយ Resume ឆ្លាស់គ្នាដោយដៃ។
3. បង្ហាញថា Generator ផ្តល់តម្លៃតាម `yield` ខណៈ Fiber អាចផ្អាក Call Stack។
4. ពិភាក្សាថា Blocking cURL Call ក្នុង Fiber តែមួយមានឥរិយាបថយ៉ាងដូចម្តេច។
5. ប្រៀបធៀប Fiber និង Queue Worker សម្រាប់ Email ផ្ញើក្រោយ HTTP Response។

## ៩. ចម្លើយណែនាំខ្លី

- Fiber ជា Stack ដែលអាច Suspend/Resume; វាមិនមែនជា Thread។
- `start()` ដំណើរការរហូតដល់ Suspend/Finish, `resume()` បន្តពីកន្លែងផ្អាក។
- Scheduler/Event Loop និង Non-blocking I/O Library ត្រូវការសម្រាប់ Async Concurrency មានប្រយោជន៍។
- Generator សម្រាប់បង្កើត Iterable Values; Fiber សម្រាប់ផ្អាក Execution Flow។
- Background Job ដែលត្រូវរស់ឆ្លង HTTP Request គួរប្រើ Queue/Worker។

## ឯកសារយោង

- [PHP Manual: Fibers](https://www.php.net/manual/en/language.fibers.php)
- [PHP Manual: Fiber Class](https://www.php.net/manual/en/class.fiber.php)
- [មេរៀនទី ៥៣៖ Background Jobs និង Queue ក្នុង PHP](lesson-53-php-background-jobs-queues.md)
- [មេរៀនទី ៥៨៖ Generators និង Iterators ក្នុង PHP](lesson-58-php-generators-iterators.md)
