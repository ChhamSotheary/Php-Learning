# មេរៀនទី ៨២៖ Laravel Cache និងការកែលម្អល្បឿន

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពីគោលបំណងរបស់ cache និងអ្វីដែលគួរ cache។
- ប្រើ `Cache::remember()` ដើម្បីកាត់បន្ថយ query ដែលកើតឡើងញឹកញាប់។
- ដក cache key ពេលទិន្នន័យផ្លាស់ប្តូរ។
- ស្គាល់ drivers និង TTL (time to live)។
- ជៀសវាងការបង្ហាញទិន្នន័យហួសសម័យ ឬ private data ដោយ cache key ខុស។

## ១. Cache ជាអ្វី?

Cache រក្សាទុកលទ្ធផលដែលគណនាថ្លៃ ឬទាញពី database ដើម្បីប្រើឡើងវិញក្នុងរយៈពេលកំណត់។ វាអាចកាត់បន្ថយ latency និង database load ប៉ុន្តែត្រូវគ្រប់គ្រងពេលទិន្នន័យប្រែប្រួល។

កុំ cache អ្វីៗទាំងអស់ដោយមិនវាស់វែង។ ចាប់ផ្តើមពីទិន្នន័យដែល៖

- ត្រូវបានស្នើញឹកញាប់។
- មានតម្លៃគណនាឬ query ថ្លៃ។
- អាចទទួលយកបាន ប្រសិនបើវាហួសសម័យក្នុងរយៈពេលខ្លី។

## ២. កំណត់ Cache Store

Laravel អាចប្រើ cache drivers ផ្សេងៗ តាម project configuration ក្នុង `config/cache.php`។ Store សាមញ្ញសម្រាប់ local development អាចរក្សាទុកជា file; production អាចជ្រើស Redis ឬ store ផ្សេងតាម workload និង infrastructure។

កំណត់ default cache store តាម environment/configuration របស់ project។ កុំ assume ថា local និង production ប្រើ store ដូចគ្នា ហើយកុំដាក់តម្លៃ secrets ក្នុង source control។

## ៣. Cache Query Result ដោយ `remember()`

ឧទាហរណ៍ cache បញ្ជី Courses សម្រាប់ 5 នាទី៖

```php
use App\\Models\\Course;
use Illuminate\\Support\\Facades\\Cache;

$courses = Cache::remember(
    'courses:all',
    now()->addMinutes(5),
    fn () => Course::query()
        ->orderBy('title')
        ->get(['id', 'title', 'level'])
);
```

បើ key `courses:all` មិនទាន់មាន ឬផុតកំណត់ Laravel រត់ closure ហើយរក្សាលទ្ធផល។ បើ key មានសុពលភាព វាត្រឡប់តម្លៃពី cache វិញដោយមិនរត់ query closure។

ប្រើ cache key មានឈ្មោះច្បាស់។ បើលទ្ធផលខុសគ្នាតាម language, filter ឬអ្នកប្រើ ត្រូវបញ្ចូល context ដែលចាំបាច់ក្នុង key ឬកំណត់ cache scope ឱ្យសមស្រប។

## ៤. បន្ថែម និងលុប Cache

រក្សាទុកតម្លៃសាមញ្ញដោយ `put()`៖

```php
Cache::put('homepage:notice', 'ចុះឈ្មោះបានហើយ', now()->addHour());
```

អានតម្លៃដោយ default៖

```php
$notice = Cache::get('homepage:notice', 'មិនមានសារថ្មីទេ');
```

ដក key មួយ៖

```php
Cache::forget('homepage:notice');
```

កុំប្រើ `Cache::flush()` ដោយមិនយល់ scope របស់ cache store ព្រោះវាអាចលុប cache entries ទាំងអស់ រួមទាំង entries របស់ app ផ្សេងៗដែល share store។

## ៥. Cache Invalidation ពេល Data ផ្លាស់ប្តូរ

ពេល Course ត្រូវបានបង្កើត ឬកែប្រែ ត្រូវគិតថា key ណាខ្លះក្លាយជាចាស់។ ឧទាហរណ៍៖

```php
$course = Course::create($validated);

Cache::forget('courses:all');
```

សម្រាប់កម្មវិធីធំ អាចរៀបចំ invalidation តាម model events, service ឬ tags (អាស្រ័យលើ driver)។ ត្រូវកំណត់វិធីមួយឱ្យច្បាស់ ដើម្បីកុំឱ្យអ្នកប្រើឃើញ data stale រយៈពេលមិនសមស្រប។

## ៦. Cache និងទិន្នន័យឯកជន

កុំប្រើ key រួមតែមួយសម្រាប់ទិន្នន័យដែលខុសគ្នាតាម user ឬ permission។ ឧទាហរណ៍ dashboard របស់ user ម្នាក់មិនត្រូវ cache ដោយ key `dashboard` ដែល user ទាំងអស់ប្រើដូចគ្នា។ អាចត្រូវបញ្ចូល user ID ទៅ key ឬជៀសវាង cache ទិន្នន័យនោះ។

កុំរក្សា password, access token ឬ secrets ទៅក្នុង cache លុះត្រាតែមាន design សុវត្ថិភាពច្បាស់ និងអាយុកាលសមស្រប។

## ៧. លំហាត់អនុវត្ត

1. ប្រើ `Cache::remember()` សម្រាប់បញ្ជី Course ដែលត្រូវបង្ហាញនៅ homepage។
2. កំណត់ TTL ជា 10 នាទី។
3. ពេល admin បង្កើត Course ថ្មី ដក cache key ដែលពាក់ព័ន្ធ។
4. បង្កើត key សម្រាប់ filtered list ដោយបញ្ចូល filter value ដែលមានសុវត្ថិភាព។
5. ពន្យល់ហេតុអ្វី key រួមសម្រាប់ dashboard ឯកជនអាចបង្ហាញទិន្នន័យ user ម្នាក់ទៅ user ម្នាក់ទៀត។
6. វាស់ query count ឬ response time មុន និងក្រោយ cache ដើម្បីមើលថាមានផលប្រយោជន៍ពិតឬទេ។

## ៨. កំហុសដែលជួបញឹកញាប់

- **មិនកំណត់ TTL**៖ ទិន្នន័យអាចនៅ stale យូរពេក។
- **ភ្លេច invalidate cache**៖ database ថ្មីតែ page នៅបង្ហាញទិន្នន័យចាស់។
- **ប្រើ key មិនគ្រប់ context**៖ filter ឬ user ម្នាក់អាចទទួល result របស់ម្នាក់ទៀត។
- **cache ទិន្នន័យរសើបដោយមិនចាំបាច់**៖ ពង្រីកចំនួនកន្លែងដែលត្រូវការពារ។
- **ប្រើ cache ដើម្បីលាក់ query ដែលមិនបាន optimize**៖ ពិនិត្យ indexes, N+1 និង query plan ផងដែរ។
- **លុប cache store ទាំងមូលដើម្បីកែ bug តូចមួយ**៖ invalidate key ជាក់លាក់ និងយល់ពី shared cache មុន។

## សំណួរត្រួតពិនិត្យ

1. តើ `Cache::remember()` ធ្វើអ្វីពេល key មិនមាន?
2. TTL មានន័យដូចម្ដេច?
3. Cache invalidation គឺជាអ្វី?
4. ហេតុអ្វីត្រូវបញ្ចូល user/filter context ក្នុង cache key?
5. តើហានិភ័យនៃ `Cache::flush()` មានអ្វីខ្លះ?

## ឯកសារយោង

- [Laravel 13 Cache](https://laravel.com/docs/13.x/cache)
- [Laravel 13 Database Performance](https://laravel.com/docs/13.x/database)
