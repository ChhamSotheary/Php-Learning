# មេរៀនទី ៥៥៖ វាស់វែង និងបង្កើនល្បឿនកូដ PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- វាស់ពេលវេលាប្រតិបត្តិកូដដោយ `hrtime()`។
- ប្រើ Profiler ដើម្បីរក Bottleneck មុនពេល Optimize។
- កាត់បន្ថយការងារដដែលៗ និងប្រើ Memory ឱ្យសមស្រប។
- យល់ពី OPcache និងតួនាទីរបស់វានៅ Production។
- ប្រៀបធៀបលទ្ធផលដោយលក្ខខណ្ឌស្មើគ្នា និងជៀសវាង Microbenchmark ខុសន័យ។

## ១. កុំ Optimize មុនពេលវាស់

Performance Problem អាចមកពី CPU, Memory, Network, File I/O, External API ឬការងារដដែលៗ។ ការទាយថា Function ណាយឺតហើយកែភ្លាមអាចខ្ជះខ្ជាយពេល និងធ្វើឱ្យ Code ស្មុគស្មាញ។

ដំណើរការដែលមានប្រយោជន៍៖

1. កំណត់ថា Route/Job ណាយឺត និងកំណត់គោលដៅ។
2. វាស់លទ្ធផលក្រោមទិន្នន័យ និង Environment តំណាង។
3. ប្រើ Profiler ដើម្បីរក Function/Call Path ដែលចំណាយពេល ឬ Memory។
4. កែចំណុចសំខាន់បំផុត។
5. វាស់ម្តងទៀត និងរត់ Tests ដើម្បីប្រាកដថា Behavior មិនខូច។

## ២. វាស់ពេលវេលាដោយ `hrtime()`

`hrtime()` ផ្តល់ Monotonic High-Resolution Time ដែលសមស្របសម្រាប់វាស់រយៈពេល៖

```php
<?php
$start = hrtime(true);

$result = calculateSomething();

$elapsedNanoseconds = hrtime(true) - $start;
$elapsedMilliseconds = $elapsedNanoseconds / 1_000_000;

printf("Elapsed: %.3f ms\n", $elapsedMilliseconds);
```

ប្រើ `hrtime()` សម្រាប់ Duration មិនប្រើ `time()` ដែលមានភាពម៉ត់ត្រឹមវិនាទី។ ការវាស់ Function ម្តងអាចមាន Noise ពី JIT, OPcache, OS Scheduler និង Warm-up; វាស់ច្រើនដង និងប្រៀបធៀប Median/Percentiles នៅ Benchmark ដែលមានលក្ខខណ្ឌស្មើគ្នា។

កុំបញ្ចេញ Timing លម្អិតរបស់ Sensitive Operation ទៅ Client ប្រសិនបើអាចជួយទាយព័ត៌មាន។ កត់ Metrics នៅ Server តាម Privacy/Security Policy។

## ៣. វាស់ Memory

អាចពិនិត្យ Memory ដែល PHP ប្រើក្នុង Request/Script៖

```php
<?php
$before = memory_get_usage(true);

$items = loadItems();

$after = memory_get_usage(true);
$peak = memory_get_peak_usage(true);

printf("Current delta: %d bytes\n", $after - $before);
printf("Peak: %d bytes\n", $peak);
```

`memory_get_usage(true)` រាប់ Memory ដែល PHP បានទទួលពី System រួមទាំង Block ទំនេរមួយចំនួន; វាមិនមែនជាការវាស់ Memory របស់ Process ទាំងមូលជានិច្ចទេ។ ប្រើ Profiler/Monitoring សម្រាប់ទិដ្ឋភាពត្រឹមត្រូវនៃ App និង Runtime។

## ៤. ប្រើ Xdebug Profiler នៅ Development

Xdebug Profiler បង្កើត Profile ដែលអាចមើលជាមួយ Cachegrind-compatible Viewer ដើម្បីរក Function ដែលចំណាយពេល ឬ Memory។ វិធីធម្មតា៖

1. ដំឡើង Xdebug នៅ Local/Development Environment។
2. បើក Profiler តែពេលត្រូវការស៊ើបអង្កេត។
3. ប្រើ Profile Output ដើម្បីរក Hot Path។
4. កែ និងវាស់ម្តងទៀត។
5. បិទ Profiling ពេលមិនប្រើ ដើម្បីកុំឱ្យ File ច្រើន/Overhead កើន។

Profile File អាចមានឈ្មោះ Function, Path និងព័ត៌មានរចនាសម្ព័ន្ធ App។ កុំបម្រើវាជាសាធារណៈ ឬទុកក្នុង Production Web Root។ Xdebug អាចបង្កើន Overhead ដូច្នេះកុំប្រើ Profile Result ជាតួលេខ Production ពិតដោយមិនគិតពីផលប៉ះពាល់របស់ Tool។

## ៥. OPcache

PHP បកប្រែ Source Code ទៅ Opcode មុនដំណើរការ។ OPcache រក្សា Opcode ដែលបាន Compile ក្នុង Memory ដើម្បីកុំឱ្យ PHP Parse Source ថ្មីរាល់ Request។

សម្រាប់ Production៖

- ពិនិត្យថា OPcache Extension មាន និងបើកតាម Configuration របស់ PHP Runtime។
- កំណត់ Memory/Validation តាមការណែនាំរបស់ Hosting និង Deployment Model។
- ក្រោយ Deploy Code ថ្មី ត្រូវធានាថា Cache ផ្ទុក Code ថ្មី តាម Reload/Restart ដែល Platform គាំទ្រ។
- កុំបើក OPcache Status Page ជាសាធារណៈ។
- វាស់មុន/ក្រោយបើក ដោយ Load ស្រដៀងគ្នា។

OPcache អាចជួយកាត់បន្ថយការងារ Parse/Compile ប៉ុន្តែមិនធ្វើឱ្យ Algorithm អន់, Request ធំ ឬ External API យឺតក្លាយជាលឿនដោយស្វ័យប្រវត្តិទេ។

## ៦. កាត់បន្ថយការងារ និង Memory

- ជៀសវាងគណនាលទ្ធផលដដែលៗក្នុង Loop; គណនាម្តង ហើយ Reuse។
- ប្រើ Algorithm/Data Structure សមស្របនឹងចំនួនទិន្នន័យ។
- កុំ Load Data ធំទាំងមូល បើអាចដំណើរការជា Chunk/Generator បាន។
- កំណត់ចំនួន Input/Request និងទំហំ Response។
- Cache លទ្ធផលដែលថ្លៃ និងអាចប្រើឡើងវិញបាន ដោយកំណត់ Expiry/Invalidation។
- កុំប្រើ Premature Optimization ដែលធ្វើឱ្យ Code មិនអាចថែទាំបាន។

ឧទាហរណ៍ Generator អាចផ្តល់តម្លៃម្តងមួយ ដោយមិនបង្កើត Array ធំទាំងមូល៖

```php
<?php
function numbersUpTo(int $max): Generator
{
    for ($number = 1; $number <= $max; $number++) {
        yield $number;
    }
}

foreach (numbersUpTo(1_000_000) as $number) {
    // ដំណើរការមួយជំហានៗ
}
```

Generator មិនមានន័យថាលឿនជាងគ្រប់ករណីទេ; វាជួយកាត់បន្ថយការកាន់ទិន្នន័យទាំងមូលក្នុង Memory នៅពេល Consumer អាចប្រើវាជា Stream។

## ៧. កំហុសដែលជួបញឹកញាប់

- Optimize ដោយគ្មាន Profile ឬ Benchmark។
- ប្រៀបធៀបតេស្តដែលមានទិន្នន័យ/Environment ខុសគ្នា។
- ចាប់យកពេលវេលា Function ម្តងហើយសន្និដ្ឋានថាលទ្ធផលត្រឹមត្រូវ។
- បើក Xdebug Profiler នៅ Production ដោយមិនគិត Overhead/ការលេចទិន្នន័យ។
- គិតថា OPcache ជួសជុល Bottleneck គ្រប់ប្រភេទ។
- រក្សាទុក Profile/Cache File ក្នុង Public Web Root។
- កែ Code ឱ្យស្មុគស្មាញដោយមិនវាស់ថា Performance ប្រសើរឡើងឬអត់។

## ៨. លំហាត់

1. វាស់ Function មួយដោយ `hrtime()` ហើយបង្ហាញលទ្ធផលជា Milliseconds។
2. ប្រៀបធៀប Array ធំមួយជាមួយ Generator ដោយកត់ Peak Memory។
3. ប្រើ Xdebug Profiler នៅ Local ដើម្បីរក Hot Path ក្នុង Script។
4. ពិនិត្យ OPcache Status នៅ Development/Production Configuration ដោយមិនបង្ហាញ Status ជាសាធារណៈ។
5. កត់ Baseline និងលទ្ធផលក្រោយ Optimize ដោយរក្សាលក្ខខណ្ឌ Benchmark ដូចគ្នា។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ `hrtime(true)` សម្រាប់វាស់ Duration និងវាស់ច្រើនដង។
- ប្រើ Profiler ដើម្បីរក Bottleneck មុនកែ Code។
- Xdebug Profiler សមស្របសម្រាប់ Development; Profile Files ត្រូវការពារ។
- OPcache រក្សា Compiled Opcode ប៉ុន្តែមិនជំនួសការកែ Algorithm/Network Bottleneck។
- Optimize តែពេលមានភស្តុតាងថាបញ្ហាប៉ះពាល់ ហើយវាស់លទ្ធផលក្រោយកែ។

## ឯកសារយោង

- [PHP: hrtime()](https://www.php.net/manual/en/function.hrtime.php)
- [PHP: OPcache](https://www.php.net/manual/en/book.opcache.php)
- [Xdebug: Profiling](https://xdebug.org/docs/profiler)
- [មេរៀនទី ៤៨៖ Static Analysis ដោយ PHPStan](lesson-48-php-static-analysis-phpstan.md)
