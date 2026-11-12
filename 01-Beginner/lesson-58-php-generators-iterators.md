# មេរៀនទី ៥៨៖ Generators និង Iterators ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Array, Iterator និង Generator។
- ប្រើ `yield` ដើម្បីផ្តល់តម្លៃមួយៗពេល `foreach` ដំណើរការ។
- អានឯកសារ/ទិន្នន័យជាបន្តបន្ទាប់ ដោយមិនបង្កើត Array ធំទាំងមូល។
- ប្រើ `yield from` និង `IteratorAggregate`។
- យល់ពីដែនកំណត់៖ Generator ជាទូទៅមិនអនុញ្ញាត Random Access ហើយអាចប្រើបានតែមួយដំណើរការ។

## ១. Array និង Generator ខុសគ្នាយ៉ាងដូចម្តេច?

Array រក្សាតម្លៃទាំងអស់ក្នុង Memory មុនពេល Loop ប្រើវា។ Generator ផ្តល់តម្លៃតាមលំដាប់ពេល Consumer ស្នើតម្លៃបន្ទាប់ ដូច្នេះវាអាចសន្សំ Memory បើទិន្នន័យមិនត្រូវប្រមូលទុកទាំងអស់។

Generator មិនធ្វើឱ្យគ្រប់កម្មវិធីលឿនជាងមុនទេ៖

- បើ Consumer ប្រមូលតម្លៃទាំងអស់ទៅ Array ដដែល Memory ក៏អាចកើនវិញ។
- បើប្រភពទិន្នន័យខាងក្រោយ Load ទាំងអស់ជាមុន Generator មិនអាចសន្សំ Memory នៅប្រភពនោះបានទេ។
- Generator មិនសមស្របបើត្រូវការចូលទៅ Index ណាមួយដោយចៃដន្យ។

## ២. Generator ដំបូងដោយ `yield`

```php
<?php
function evenNumbersUpTo(int $max): Generator
{
    for ($number = 0; $number <= $max; $number += 2) {
        yield $number;
    }
}

foreach (evenNumbersUpTo(10) as $number) {
    echo $number . PHP_EOL;
}
```

Function ដែលមាន `yield` ត្រឡប់ Generator Object មិនមែនត្រឡប់ Array ទេ។ ការគណនាក្នុង Function ចាប់ផ្តើមពេល Consumer ចាប់ផ្តើម Iterate ហើយបន្តតាមលំដាប់។

## ៣. បង្កើត Key និង Value

អាចកំណត់ Key ដូចជា Associative Array៖

```php
<?php
function userNames(iterable $users): Generator
{
    foreach ($users as $user) {
        yield $user['id'] => $user['name'];
    }
}

foreach (userNames($users) as $id => $name) {
    echo $id . ': ' . $name . PHP_EOL;
}
```

កំណត់ Key ឱ្យមានន័យ និងជៀសវាង Key ស្ទួន ប្រសិនបើ Consumer រំពឹងថាជា Map; Generator អាចមាន Key ដដែលបាន ប៉ុន្តែ Consumer មួយចំនួនអាចបាត់តម្លៃមុន។

## ៤. អាន CSV ជាបន្ទាត់ៗ

ឧទាហរណ៍ Generator បើកឯកសារ អានមួយ Record ម្តង និងបិទ File នៅពេលបញ្ចប់/មាន Exception៖

```php
<?php
function csvRows(string $path): Generator
{
    $handle = fopen($path, 'rb');

    if ($handle === false) {
        throw new RuntimeException('Unable to open CSV file.');
    }

    try {
        while (($row = fgetcsv($handle, 0, ',', '"', '')) !== false) {
            yield $row;
        }
    } finally {
        fclose($handle);
    }
}

foreach (csvRows(__DIR__ . '/data.csv') as $row) {
    // Validate Row ហើយដំណើរការភ្លាមៗ
}
```

`fgetcsv()` Parse CSV តាម Rules របស់វា; កុំបំបែក CSV ដោយ `explode(',')` ព្រោះ Field អាចមាន Comma/Quote។ ត្រូវ Validate Row/Column Count និងទំហំឯកសារ។ Path គួរត្រូវបានកំណត់ដោយ Server មិនយកពី Input ដោយផ្ទាល់ ដើម្បីកាត់បន្ថយ Path Traversal។

## ៥. ប្រើ `yield from`

`yield from` ផ្ទេរតម្លៃពី Iterable ផ្សេងទៅ Consumer៖

```php
<?php
function allLines(array $files): Generator
{
    foreach ($files as $file) {
        yield from csvRows($file);
    }
}
```

វាអាចធ្វើឱ្យ Code សម្រាប់បញ្ចូល Generator មួយទៅមួយទៀតសាមញ្ញ។ ត្រូវគ្រប់គ្រង Error/Resource នៅ Generator ដែលបានបញ្ចូលផងដែរ។

## ៦. IteratorAggregate សម្រាប់ Collection

បើ Class មួយចង់អនុញ្ញាត `foreach` លើ Items របស់ខ្លួន អាច Implements `IteratorAggregate`៖

```php
<?php
final class ProductCollection implements IteratorAggregate
{
    /** @param list<Product> $products */
    public function __construct(
        private array $products
    ) {
    }

    public function getIterator(): Traversable
    {
        yield from $this->products;
    }
}
```

`IteratorAggregate::getIterator()` អាចត្រឡប់ Traversable ដូចជា ArrayIterator ឬ Generator។ ប្រើវាដើម្បីបង្ហាញ Collection ជា Iterable ដោយមិនបង្ហាញរបៀបផ្ទុកខាងក្នុង។

## ៧. Generator មាន Lifecycle

Generator ជាទូទៅត្រូវ Iterate ម្តង។ បើត្រូវការប្រើទិន្នន័យច្រើនដង អាចបង្កើត Generator ថ្មីពី Factory Function ម្តងទៀត ប្រសិនបើប្រភពអាចអានឡើងវិញបាន។ កុំសន្មតថា Generator មាន Count/Index ឬអាច Rewind ដូច Array ទេ។

ប្រសិនបើ Generator បើក File/Resource ត្រូវប្រើ `try/finally` ដើម្បីធានា Cleanup។ Consumer ដែលបញ្ឈប់ Loop មុនចប់ក៏ត្រូវគិតពីការបិទ Resource តាម Lifecycle របស់កម្មវិធី។

## ៨. កំហុសដែលជួបញឹកញាប់

- បម្លែង Generator ទៅ Array ភ្លាមៗ ហើយបាត់ Memory Advantage។
- រំពឹងថា Generator អាចប្រើ Index ដូច Array។
- ប្រើ Generator សម្រាប់ Data តូច និងធ្វើឱ្យ Code ស្មុគស្មាញដោយមិនចាំបាច់។
- មិនបិទ File/Database Cursor នៅពេល Generator បញ្ចប់។
- ទុកចិត្តលើ Input Row ដោយមិន Validate។
- ប្រើ `explode()` ដើម្បី Parse CSV ដែលមាន Quote/Comma ខាងក្នុង។

## ៩. លំហាត់

1. សរសេរ Generator ផ្តល់លេខចាប់ពី 1 ដល់ N។
2. បង្កើត CSV Generator ដែលអាន File មួយ Record ម្តង។
3. រាប់ចំនួន Row ដោយ Loop ដោយមិនរក្សា Row ទាំងអស់ក្នុង Array។
4. បង្កើត Class Collection ដែល Implements `IteratorAggregate`។
5. សរសេរឧទាហរណ៍ដែល Generator មិនសន្សំ Memory ព្រោះ Consumer ប្រមូលតម្លៃទាំងអស់ទៅ Array។

## ១០. ចម្លើយណែនាំខ្លី

- `yield` ផ្តល់តម្លៃមួយម្តងក្នុងពេល Iterate។
- Generator ជួយសន្សំ Memory បើ Source និង Consumer ដំណើរការជា Stream។
- `yield from` ផ្ទេរតម្លៃពី Iterable មួយទៀត។
- ប្រើ `IteratorAggregate` ដើម្បីឱ្យ Collection អាច Iterate បាន។
- Generator មិនមែនជា Array ទេ៖ វាមិនសមស្របសម្រាប់ Random Access ហើយជាទូទៅ Iterate ម្តង។

## ឯកសារយោង

- [PHP Manual: Generators Overview](https://www.php.net/manual/en/language.generators.overview.php)
- [PHP Manual: Generator Syntax](https://www.php.net/manual/en/language.generators.syntax.php)
- [PHP Manual: IteratorAggregate](https://www.php.net/manual/en/class.iteratoraggregate.php)
- [មេរៀនទី ៥៥៖ វាស់វែង Performance និង Memory ក្នុង PHP](lesson-55-php-performance-profiling-opcache.md)
