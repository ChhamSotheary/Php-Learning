# មេរៀនទី ១០៧៖ Symfony Cache និង Cache Pools

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ប្រើ Cache Contracts ដើម្បីរក្សាទុកលទ្ធផលដែលគណនាថ្លៃ។
- កំណត់ cache key និង expiration។
- ស្គាល់ `cache.app`, `cache.system` និង custom pools។
- លុប ឬធ្វើឱ្យ cache ផុតសុពលភាពដោយមានគោលបំណង។

## ១. Cache ជាអ្វី?

Cache រក្សាទុកលទ្ធផលមួយរយៈ ដើម្បីកុំឱ្យកម្មវិធីធ្វើ query, HTTP request ឬ computation ដដែលៗគ្រប់ពេល។ Symfony Cache ផ្ដល់ Cache Contracts ដែលសរសេរកូដខ្លី និងការពារ cache stampede ជាលំនាំដើម។

ក្នុង Symfony មាន `cache.system` សម្រាប់ទិន្នន័យដែលកើតចេញពី source code និងផ្លាស់ប្ដូរតាម deployment និង `cache.app` សម្រាប់ទិន្នន័យកម្មវិធីដែលអាចផ្លាស់ប្ដូរពេល runtime។ ជាទូទៅប្រើ `cache.app` សម្រាប់ application data ហើយរក្សា system cache តាមការកំណត់លំនាំដើមរបស់ Symfony។

## ២. ប្រើ Cache Contracts

Inject `CacheInterface` ទៅ service៖

```php
<?php

namespace App\Service;

use Symfony\Contracts\Cache\CacheInterface;
use Symfony\Contracts\Cache\ItemInterface;

final class PopularProducts
{
    public function __construct(
        private CacheInterface $cache,
        private ProductRepository $products,
    ) {
    }

    public function getTopProducts(): array
    {
        return $this->cache->get(
            'catalog.top_products',
            function (ItemInterface $item): array {
                $item->expiresAfter(300);

                return $this->products->findTopProducts();
            },
        );
    }
}
```

បើ key មិនមាន ឬផុតកំណត់ callback នឹងគណនាតម្លៃថ្មី ហើយរក្សាទុកលទ្ធផល។ បើ cache នៅមានសុពលភាព វាត្រឡប់តម្លៃដែលបានរក្សាទុក។ កំណត់ key ឱ្យតំណាងឱ្យ input/filters ដែលប៉ះពាល់ដល់លទ្ធផល; កុំប្រើ key ដូចគ្នាសម្រាប់ query ផ្សេងគ្នា។

## ៣. Expiration និង Invalidation

`expiresAfter(300)` កំណត់អាយុកាល 300 វិនាទី។ ការផុតកំណត់ជួយឱ្យទិន្នន័យមិនចាស់យូរពេក ប៉ុន្តែការផុតកំណត់វែងពេកអាចបង្ហាញតម្លៃចាស់។ ការលុប key ដោយផ្ទាល់៖

```php
$this->cache->delete('catalog.top_products');
```

ហៅការលុប cache ពេលទិន្នន័យពាក់ព័ន្ធត្រូវបានបង្កើត កែប្រែ ឬលុប។ សម្រាប់ key ច្រើនដែលពាក់ព័ន្ធគ្នា អាចរៀបចំ cache pools និង tags តាម Symfony Cache ដើម្បី invalidate ជាក្រុម។

## ៤. Custom Cache Pool

បើចង់បែងចែក application cache តាម domain អាចកំណត់ pool ក្នុង `config/packages/cache.yaml`៖

```yaml
framework:
    cache:
        pools:
            cache.catalog:
                adapter: cache.app
                default_lifetime: 300
```

Custom pool ជួយឱ្យកំណត់ adapter, lifetime ឬ namespace ដាច់ដោយឡែក។ បើ service ត្រូវយក pool ជាក់លាក់ អាច inject `CacheInterface` ដែលកំណត់ autowiring alias សម្រាប់ pool ឬប្រើ DI binding តាម configuration របស់ project។ ពិនិត្យឈ្មោះ service ដោយ `debug:container`។

## ៥. ជ្រើស Cache Adapter

Symfony មាន adapter ដូចជា filesystem, APCu, Redis និង Memcached។ Filesystem ជាលំនាំដើមងាយប្រើសម្រាប់ local app។ Redis អាចចែករំលែក cache រវាង application instances និងរក្សាទុកឆ្លងកាត់ deployment បានល្អជាងក្នុង setup ច្រើនម៉ាស៊ីន ប៉ុន្តែត្រូវរៀបចំ provider និងតាមដានវា។

ជ្រើស backend តាមប្រភេទ data, topology និង operational needs។ កុំរំពឹងថា cache ជា database អចិន្ត្រៃយ៍; តម្លៃក្នុង cache អាចត្រូវបាត់បង់ និងត្រូវអាចគណនាឡើងវិញបាន។

## ៦. ទិន្នន័យឯកជន និង Cache Key

កុំ cache response ឬទិន្នន័យឯកជនដោយប្រើ key សកល ប្រសិនបើលទ្ធផលខុសគ្នាតាមអ្នកប្រើ។ បើចាំបាច់ ត្រូវដាក់ user/tenant/locale/filter identifiers ក្នុង key និងពិចារណាសុវត្ថិភាពនៃ storage។ កុំដាក់ password, access token ឬ secret ក្នុង cache key/values ដោយគ្មានហេតុផលច្បាស់។

ការបំបែក HTTP response cache និង application data cache ក៏សំខាន់ដែរ៖ lesson នេះផ្ដោតលើតម្លៃក្នុងកូដ។ សម្រាប់ cache HTTP response សូមមើល HTTP Cache និង Cache-Control headers។

## ៧. ពិនិត្យ និងសាកល្បង

ពិនិត្យ services និង cache configuration៖

```bash
php bin/console debug:container cache.app
php bin/console debug:config framework cache
```

ក្នុង test អាចប្រើ array adapter ឬ test configuration ដើម្បីកុំឱ្យ test ពឹងផ្អែកលើ cache ចាស់។ សាកល្បងទាំង cache hit, cache miss, expiry និង invalidation។

## កំហុសដែលជួបញឹកញាប់

- **Key មិនបញ្ចូល filters/tenant**៖ អ្នកប្រើអាចទទួល data របស់អ្នកដទៃ ឬលទ្ធផលមិនត្រឹមត្រូវ។
- **មិន invalidate ក្រោយកែ data**៖ cache បង្ហាញតម្លៃចាស់រហូតដល់ expire។
- **ដាក់ lifetime វែងពេក**៖ ព័ត៌មានមិនត្រឹមត្រូវនៅតែបង្ហាញ។
- **ប្រើ cache ដូចជា permanent storage**៖ cache អាចត្រូវលុបគ្រប់ពេល; database ត្រូវជាប្រភពទិន្នន័យពិត។
- **cache data ឯកជនជាមួយ key ទូទៅ**៖ កំណត់ key និង response policy តាមអ្នកប្រើឱ្យត្រឹមត្រូវ។
- **ច្រឡំ cache.app និង cache.system**៖ application data គួរប្រើ cache.app។

## លំហាត់

1. Cache លទ្ធផល `findTopProducts()` រយៈពេល 5 នាទី។
2. បង្កើត cache key ដែលរួមបញ្ចូល category និង page number។
3. លុប key ពេល Product ត្រូវបានកែប្រែ។
4. បង្កើត custom pool `cache.catalog` និងពិនិត្យ config។
5. សរសេរ test ពិនិត្យ cache miss, hit និង invalidation។

## សំណួរត្រួតពិនិត្យ

1. តើ Cache Contracts `get()` ដំណើរការយ៉ាងដូចម្ដេច?
2. តើ `expiresAfter()` កំណត់អ្វី?
3. តើ cache.app ខុសពី cache.system ដូចម្ដេច?
4. តើហេតុអ្វី cache មិនមែនជា permanent storage?
5. តើ key ត្រូវគិតពី tenant/user data ដូចម្ដេច?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Cache](https://symfony.com/doc/7.4/cache.html)
- [Symfony 7.4 HTTP Cache](https://symfony.com/doc/7.4/http_cache.html)
