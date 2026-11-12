# មេរៀនទី ៥៩៖ សរសេរ PHP CLI Scripts

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បង្កើត PHP Script ដែលរត់ពី Terminal។
- អាន Option ដោយ `getopt()` និង Argument តាម `$argv`។
- ប្រើ STDIN, STDOUT និង STDERR។
- ត្រឡប់ Exit Code ដើម្បីឱ្យ Shell/CI ដឹងថា Command ជោគជ័យឬអត់។
- រៀបចំ Error Message និង Help Text សម្រាប់ Command Line។

> CLI គឺ Command Line Interface។ ឧទាហរណ៍នេះប្រើ PHP CLI SAPI; វាមិនមែនជា HTTP Request ទេ។

## ១. បង្កើត Script និងរត់វា

បង្កើតឯកសារ `bin/greet.php`៖

```php
<?php
declare(strict_types=1);

$name = $argv[1] ?? 'អ្នករៀន';

if (!is_string($name) || trim($name) === '') {
    fwrite(STDERR, "សូមបញ្ចូលឈ្មោះ។\n");
    exit(2);
}

fwrite(STDOUT, 'សួស្តី ' . $name . '!' . PHP_EOL);
exit(0);
```

រត់នៅ Terminal៖

```bash
php bin/greet.php Sotheary
```

- `$argc` ជាចំនួន Argument ដែលបានផ្ញើ។
- `$argv` ជា Array នៃ Argument; `$argv[0]` ជាឈ្មោះ Script។
- `STDOUT` សម្រាប់ Output ធម្មតា; `STDERR` សម្រាប់ Error/Diagnostic។
- Exit Code `0` ជាទូទៅមានន័យថាជោគជ័យ; លេខមិនមែនសូន្យបង្ហាញថាមានបញ្ហា។

## ២. Parse Options ដោយ `getopt()`

ឧទាហរណ៍ Command មាន `--limit`, `--format` និង `--help`៖

```php
<?php
declare(strict_types=1);

$options = getopt('', ['limit:', 'format:', 'help']);

if ($options === false || isset($options['help'])) {
    fwrite(STDOUT, "ប្រើ៖ php bin/list.php --limit=20 --format=json\n");
    exit(0);
}

$limitRaw = $options['limit'] ?? '20';
$limit = filter_var(
    $limitRaw,
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 100]]
);

if ($limit === false) {
    fwrite(STDERR, "--limit ត្រូវជាលេខពី 1 ដល់ 100។\n");
    exit(2);
}

$format = $options['format'] ?? 'text';
if (!is_string($format) || !in_array($format, ['text', 'json'], true)) {
    fwrite(STDERR, "--format អាចជា text ឬ json។\n");
    exit(2);
}

// ហៅ Application Service ដើម្បីទាញ/រៀបចំទិន្នន័យ។
fwrite(STDOUT, "Limit: {$limit}; format: {$format}" . PHP_EOL);
```

នៅពេលប្រើ `getopt()`, កំណត់ Option Names និងថាតើត្រូវការ Value ឬអត់។ បន្ទាប់មក Validate Type/Range ដូច Input ផ្សេងទៀត។ ឥរិយាបថ Parse Option និង Positional Argument មានភាពជាក់លាក់តាម PHP; សម្រាប់ CLI ស្មុគស្មាញ អាចប្រើ Console Component របស់ Framework។

## ៣. អាន Input ពី STDIN

CLI អាចទទួល Input ពី Pipe ដោយ `STDIN`៖

```php
<?php
$line = fgets(STDIN);

if ($line === false) {
    fwrite(STDERR, "មិនទទួលបាន Input។\n");
    exit(1);
}

$value = trim($line);
fwrite(STDOUT, "បានទទួល " . strlen($value) . " bytes។\n");
```

អាចប្រើ៖

```bash
echo "hello" | php bin/read-input.php
```

បើអានឯកសារធំតាម Pipe ត្រូវដំណើរការម្តងមួយបន្ទាត់/Chunk ជំនួសការអានទាំងមូលចូល Memory។ ចងចាំថា `strlen()` វាស់ Bytes មិនមែនចំនួនតួអក្សរ UTF-8 ទេ។

## ៤. បំបែក CLI ពី Business Logic

កុំដាក់ Business Logic ទាំងអស់ក្នុង Script។ ឱ្យ CLI ជា Thin Entry Point៖

1. Parse និង Validate Options។
2. បង្កើត Dependency/Load Configuration។
3. ហៅ Service ដែលអាចត្រូវបានប្រើដោយ Web App/Test ផងដែរ។
4. បង្ហាញ Result ឬ Error ទៅ STDOUT/STDERR។
5. ត្រឡប់ Exit Code សមស្រប។

នេះធ្វើឱ្យ Logic អាច Unit Test បាន ដោយមិនចាំបាច់ចាប់យក Terminal Input។

## ៥. Exit Codes និង Shell Automation

ជ្រើសរើស Exit Code ឱ្យមានន័យ និងឯកសារបញ្ជាក់៖

- `0`: Command ជោគជ័យ។
- `1`: កំហុសទូទៅ/Unexpected Error។
- `2`: Argument ឬ Option មិនត្រឹមត្រូវ។

ប្រព័ន្ធផ្សេងអាចកំណត់ Code ផ្ទាល់ខ្លួន ប៉ុន្តែត្រូវរក្សាឱ្យជាប់លាប់។ CI, Cron និង Shell Script អាចប្រើ Exit Code ដើម្បីសម្រេចបន្ត ឬបញ្ឈប់។

## ៦. សុវត្ថិភាព CLI

- Validate Option/Argument ទាំងអស់; CLI Input ក៏អាចមកពី User/Automation ដែលមិនទុកចិត្ត។
- កុំបោះពុម្ព Secret/API Key ទៅ Terminal Log/CI Log។
- កុំភ្ជាប់ User Input ទៅ Shell Command ដោយ String Concatenation។
- ប្រើ File Path ដែលកំណត់/Allowlist និងពិនិត្យ Permission។
- កំណត់ Resource Limit សម្រាប់ Command ដែលអាចដំណើរការយូរ។
- កុំប្រើ Production Configuration ដោយចៃដន្យក្នុង Local Command។

## ៧. កំហុសដែលជួបញឹកញាប់

- បញ្ជូន Error ទៅ STDOUT ធ្វើឱ្យ Pipe/JSON Output ខូច។
- មិន Validate Option ហើយទទួលតម្លៃក្រៅ Range។
- រត់ Script ជា Root ដោយមិនចាំបាច់។
- បញ្ចេញ Secret ទៅ CI/Terminal Logs។
- ដាក់ Logic ក្នុង Script មួយធំដែលមិនអាច Test បាន។
- មិនកំណត់ Exit Code ទើប Cron/CI មិនដឹងថា Command បរាជ័យ។

## ៨. លំហាត់

1. បង្កើត `bin/report.php` ដែលទទួល `--format=text|json`។
2. បន្ថែម `--limit=1..100` និង Validate ជាចំនួនគត់។
3. សរសេរ Error ទៅ STDERR និង Return Exit Code 2 ពេល Option ខុស។
4. អាន CSV ពី STDIN ដោយប្រើ Generator ពីមេរៀនទី ៥៨។
5. បំបែក Report Logic ទៅ Service Class ហើយសរសេរ Unit Test។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ `$argv` សម្រាប់ Argument និង `getopt()` សម្រាប់ Options។
- Validate Input ដូច Request ផ្សេងៗ ហើយប្រើ Exit Code មានន័យ។
- ផ្ញើ Result ទៅ STDOUT និង Diagnostic/Error ទៅ STDERR។
- បំបែក Business Logic ពី CLI Entry Point ដើម្បី Test បាន។
- កុំ Log Secret ឬបញ្ជូន Input ទៅ Shell Command ដោយមិនមានការការពារ។

## ឯកសារយោង

- [PHP Manual: Command Line Usage](https://www.php.net/manual/en/features.commandline.php)
- [PHP Manual: getopt()](https://www.php.net/manual/en/function.getopt.php)
- [PHP Manual: Command Line I/O Streams](https://www.php.net/manual/en/features.commandline.io-streams.php)
- [មេរៀនទី ៥៨៖ Generators និង Iterators](lesson-58-php-generators-iterators.md)
- [មេរៀនទី ២៥៖ Dependency Injection និង Design Patterns](lesson-25-php-dependency-injection-design-patterns.md)
