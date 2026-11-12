# មេរៀនទី ១០៖ PHP Include និង Require

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពីមូលហេតុដែលគួរបំបែក PHP Code ទៅជា File ច្រើន។
- ប្រើ `include` និង `require` ដើម្បីបញ្ចូល File មួយទៅក្នុង File មួយទៀត។
- យល់ពីភាពខុសគ្នារវាង `include`, `require`, `include_once` និង `require_once`។
- រៀបចំ Header, Footer, Navigation និង Configuration ជា File ដាច់ដោយឡែក។
- ប្រើ Path ដោយត្រឹមត្រូវជាមួយ `__DIR__`។
- រៀបចំ Project តូចមួយដែលអាចថែទាំ និងកែប្រែបានងាយស្រួល។

## ២. ហេតុអ្វីត្រូវប្រើ Include និង Require?

ស្រមៃថា Website របស់អ្នកមាន ១០ ទំព័រ ហើយទំព័រទាំងអស់មាន Header និង Footer ដូចគ្នា។ ប្រសិនបើអ្នកចម្លង Header និង Footer ទៅគ្រប់ File នោះពេលចង់កែ Logo ឬ Menu អ្នកត្រូវកែ ១០ កន្លែង។

វិធីល្អជាងគេគឺដាក់ Header ក្នុង File មួយ និង Footer ក្នុង File មួយ ហើយបញ្ចូលវាទៅទំព័រដែលត្រូវការ។

ឧទាហរណ៍រចនាសម្ព័ន្ធ៖

```
my-website/
│
├── index.php
├── about.php
├── contact.php
│
└── includes/
    ├── header.php
    ├── navigation.php
    └── footer.php
```

អត្ថប្រយោជន៍៖

- កាត់បន្ថយកូដស្ទួន។
- កែប្រែបាននៅកន្លែងតែមួយ។
- ងាយស្រួលរៀបចំ និងថែទាំ Project។
- អាចប្រើ Component ដដែលឡើងវិញនៅទំព័រផ្សេងៗ។

## ៣. មុខងារ include

`include` បញ្ចូល និងដំណើរការមាតិកា PHP ពី File ផ្សេងទៀត។

បង្កើត File `message.php`៖

```php
<?php
echo "សួស្តី! នេះជាសារមកពី message.php";
```

បង្កើត File `index.php`៖

```php
<?php
echo "ចាប់ផ្តើមកម្មវិធី<br>";

include 'message.php';

echo "<br>បញ្ចប់កម្មវិធី";
```

លទ្ធផលរំពឹងទុក៖

```
ចាប់ផ្តើមកម្មវិធី
សួស្តី! នេះជាសារមកពី message.php
បញ្ចប់កម្មវិធី
```

ចំណាំ៖ File ដែលបាន Include អាចមាន PHP, HTML, CSS ឬអត្ថបទធម្មតា។ PHP នឹងដំណើរការ PHP Code នៅក្នុង File នោះ។

## ៤. មុខងារ require

`require` ក៏បញ្ចូល File ដូច `include` ដែរ ប៉ុន្តែប្រសិនបើ File មិនមាន ឬមិនអាចបញ្ចូលបាន វាបង្កើត Error កម្រិតធ្ងន់ (Error/Throwable) ហើយការដំណើរការបច្ចុប្បន្ននឹងមិនបន្តតាមធម្មតាទៀតទេ។

ឧទាហរណ៍៖

```php
<?php
require 'config.php';

echo "កម្មវិធីអាចប្រើ Configuration បាន។";
```

ប្រើ `require` សម្រាប់ File សំខាន់ដែលកម្មវិធីមិនអាចដំណើរការបានដោយគ្មានវា ដូចជា Configuration ឬ File ដែលកំណត់ Function សំខាន់ៗ។

## ៥. ភាពខុសគ្នារវាង Include និង Require

| មុខងារ | ពេល File មិនមាន |
|---|---|
| `include` | បង្ហាញ Warning ហើយ Script អាចបន្ត |
| `require` | បង្កើត Error/Throwable ហើយការដំណើរការមិនបន្តតាមធម្មតា |
| `include_once` | ដូច include ប៉ុន្តែបញ្ចូល File ម្តងប៉ុណ្ណោះ |
| `require_once` | ដូច require ប៉ុន្តែបញ្ចូល File ម្តងប៉ុណ្ណោះ |

ជ្រើសរើសឱ្យសមស្រប៖

- `include`៖ សម្រាប់ផ្នែកដែលអាចខ្វះបាន ហើយទំព័រនៅតែអាចដំណើរការ។
- `require`៖ សម្រាប់ Dependency សំខាន់។
- `include_once`៖ ពេលត្រូវការបញ្ចូល File ម្តងប៉ុណ្ណោះ។
- `require_once`៖ ពេល File សំខាន់ និងមិនចង់ឱ្យវាត្រូវបញ្ចូលស្ទួន។

## ៦. Include Once និង Require Once

ពេល File មួយត្រូវបាន Include ច្រើនដង វាអាចបង្កបញ្ហា ដូចជា Function ត្រូវបានប្រកាសស្ទួន។

ឧទាហរណ៍៖

File `functions.php`៖

```php
<?php
function sayHello(): string
{
    return "សួស្តី PHP!";
}
```

File `index.php`៖

```php
<?php
require_once __DIR__ . '/functions.php';
require_once __DIR__ . '/functions.php';

echo sayHello();
```

លទ្ធផល៖

```
សួស្តី PHP!
```

PHP មិនបញ្ចូល File ដដែលជាលើកទីពីរទេ ដូច្នេះមិនមានការប្រកាស Function ស្ទួនឡើយ។

## ៧. ការប្រើ __DIR__ សម្រាប់ File Path

`__DIR__` ជា Magic Constant ដែលផ្តល់ទីតាំង Directory ពេញលេញរបស់ File PHP បច្ចុប្បន្ន។

ឧទាហរណ៍៖

```php
<?php
require_once __DIR__ . '/includes/header.php';
```

ប្រសិនបើ File បច្ចុប្បន្នស្ថិតនៅ៖

```
C:/xampp/htdocs/my-website/index.php
```

នោះ `__DIR__` នឹងមានតម្លៃ៖

```
C:/xampp/htdocs/my-website
```

អត្ថប្រយោជន៍៖

- មិនសូវពឹងផ្អែកលើ Current Working Directory។
- ងាយស្រួលផ្លាស់ទី Project។
- កាត់បន្ថយបញ្ហា File Path មិនត្រឹមត្រូវ។

ឧទាហរណ៍ Path ទៅ File ក្នុង Folder `includes`៖

```php
require_once __DIR__ . '/includes/header.php';
```

Path ទៅ Folder មេ៖

```php
require_once __DIR__ . '/../config/database.php';
```

## ៨. រៀបចំ Header និង Footer

បង្កើត Project ដូចខាងក្រោម៖

```
php-layout/
│
├── index.php
├── about.php
│
└── includes/
    ├── header.php
    ├── navigation.php
    └── footer.php
```

### ៨.១ File includes/header.php

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHP Website</title>
</head>
<body>
    <header>
        <h1>គេហទំព័ររៀន PHP</h1>
    </header>
```

### ៨.២ File includes/navigation.php

```php
<nav>
    <a href="/index.php">ទំព័រដើម</a> |
    <a href="/about.php">អំពីយើង</a>
</nav>
```

### ៨.៣ File includes/footer.php

```php
    <footer>
        <p>រក្សាសិទ្ធិ © 2026 PHP Learning</p>
    </footer>
</body>
</html>
```

### ៨.៤ File index.php

```php
<?php
require_once __DIR__ . '/includes/header.php';
require_once __DIR__ . '/includes/navigation.php';
?>

<main>
    <h2>សូមស្វាគមន៍មកកាន់ទំព័រដើម</h2>
    <p>យើងកំពុងរៀន PHP Include និង Require។</p>
</main>

<?php
require_once __DIR__ . '/includes/footer.php';
```

### ៨.៥ File about.php

```php
<?php
require_once __DIR__ . '/includes/header.php';
require_once __DIR__ . '/includes/navigation.php';
?>

<main>
    <h2>អំពីយើង</h2>
    <p>នេះជាទំព័រណែនាំអំពី Website របស់យើង។</p>
</main>

<?php
require_once __DIR__ . '/includes/footer.php';
```

ឥឡូវនេះទំព័រទាំងពីរប្រើ Header, Navigation និង Footer ដូចគ្នា។ បើអ្នកចង់កែ Header អ្នកកែតែ File `includes/header.php` មួយប៉ុណ្ណោះ។

## ៩. បង្កើត Configuration File

Configuration File ជាទីតាំងរក្សាការកំណត់រួមរបស់កម្មវិធី។

ឧទាហរណ៍ File `config.php`៖

```php
<?php
declare(strict_types=1);

define('APP_NAME', 'PHP Learning');
define('APP_VERSION', '1.0.0');
```

ប្រើវានៅ `index.php`៖

```php
<?php
require_once __DIR__ . '/config.php';

echo APP_NAME;
echo '<br>';
echo APP_VERSION;
```

លទ្ធផល៖

```
PHP Learning
1.0.0
```

**ចំណាំសុវត្ថិភាព៖** កុំដាក់ Password ឬ Secret Key ក្នុង File ដែលអាចចូលមើលតាម Web បាន។ ក្នុង Project ពិត គួររក្សា Secret នៅក្រៅ Public Web Root ឬប្រើ Environment Variables ហើយកុំ Commit Secret ទៅ GitHub។

## ១០. Variable Scope ពេល Include File

File ដែលត្រូវបាន Include ដំណើរការនៅក្នុង Scope របស់បន្ទាត់ដែល Include វា។ ដូច្នេះ Variable ដែលមាននៅ Scope នោះអាចប្រើបានក្នុង File ដែលបាន Include។

ឧទាហរណ៍៖

File `profile.php`៖

```php
<?php
echo 'ឈ្មោះ៖ ' . htmlspecialchars($username, ENT_QUOTES, 'UTF-8');
```

File `index.php`៖

```php
<?php
$username = 'Dara';

require __DIR__ . '/profile.php';
```

លទ្ធផល៖

```
ឈ្មោះ៖ Dara
```

កុំប្រើ Variable Global ច្រើនពេកដោយមិនចាំបាច់។ សម្រាប់កម្មវិធីធំ គួររៀបចំ Function ឬ Class ឱ្យមានរចនាសម្ព័ន្ធច្បាស់លាស់។

## ១១. កំហុសដែលជួបញឹកញាប់

1. **File Path ខុស** — ប្រើ `__DIR__` ដើម្បីកំណត់ទីតាំងឱ្យច្បាស់។
2. **ប្រើ include ជំនួស require សម្រាប់ File សំខាន់** — ប្រើ `require` ឬ `require_once` សម្រាប់ Dependency ដែលចាំបាច់។
3. **បញ្ចូល File ដដែលច្រើនដង** — ប្រើ `include_once` ឬ `require_once` នៅពេលសមស្រប។
4. **ឈ្មោះ File អក្សរតូចធំមិនត្រូវគ្នា** — នៅ Server មួយចំនួន ឈ្មោះ `Header.php` និង `header.php` ខុសគ្នា។
5. **ដាក់ Configuration សម្ងាត់ក្នុង Repository សាធារណៈ** — រក្សា Secret នៅក្រៅ Git និង Public Web Root។
6. **ច្រឡំថា Include ជាការបញ្ជូនទៅទំព័រផ្សេង** — Include បញ្ចូល File ក្នុងការដំណើរការ Request ដដែល មិនមែន Redirect Browser ទេ។

## ១២. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Reusable Header

បង្កើត File `header.php` ដែលមាន៖
- HTML Document Start
- Meta Charset
- Website Title
- Heading

បន្ទាប់មក Include វានៅក្នុង `index.php` និង `about.php`។

### លំហាត់ទី ២៖ Footer

បង្កើត `footer.php` ដែលមានព័ត៌មានរក្សាសិទ្ធិ ហើយប្រើនៅទំព័រទាំងពីរ។

### លំហាត់ទី ៣៖ Function File

បង្កើត `functions.php` ដែលមាន Function៖

```php
function calculateTotal(float $price, int $quantity): float
{
    return $price * $quantity;
}
```

ប្រើ `require_once` ដើម្បីបញ្ចូល Function ហើយគណនាតម្លៃទំនិញ ៣ ឯកតា។

### លំហាត់ទី ៤៖ Path Practice

បង្កើត Folder `includes` និង `config`។ សាកល្បងប្រើ `__DIR__` ដើម្បីបញ្ចូល File ពី Folder ទាំងពីរ។

## ១៣. Mini Project៖ Multi-page PHP Website

បង្កើត Website ដែលមានទំព័រ៖

- Home
- About
- Contact

រចនាសម្ព័ន្ធ៖

```
my-php-site/
│
├── index.php
├── about.php
├── contact.php
├── config.php
│
└── includes/
    ├── header.php
    ├── navigation.php
    ├── footer.php
    └── functions.php
```

តម្រូវការ៖

1. គ្រប់ទំព័រត្រូវប្រើ Header និង Footer ដូចគ្នា។
2. Navigation ត្រូវអាចចូលទៅទំព័រទាំងបី។
3. ប្រើ `require_once` សម្រាប់ File សំខាន់ៗ។
4. ប្រើ `__DIR__` សម្រាប់ File Path។
5. ដាក់ឈ្មោះ Website ក្នុង `config.php`។
6. បង្កើត Function មួយក្នុង `functions.php` ហើយប្រើវានៅទំព័រយ៉ាងហោចណាស់មួយ។
7. កុំដាក់ Secret ឬ Password ក្នុង Repository។

## ១៤. សំណួររំលឹកមេរៀន

1. តើហេតុអ្វីយើងត្រូវបំបែក PHP Code ទៅជា File ច្រើន?
2. តើ `include` និង `require` ខុសគ្នាដូចម្តេច?
3. តើ `require_once` មានអត្ថប្រយោជន៍អ្វី?
4. តើ `__DIR__` ផ្តល់តម្លៃអ្វី?
5. តើ File ដែល Include អាចប្រើ Variable ពី File មេបានទេ?
6. តើ Include ខុសពី Redirect ដូចម្តេច?

## ១៥. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

1. ដើម្បីកាត់បន្ថយកូដស្ទួន ងាយកែប្រែ និងថែទាំ។
2. `include` បង្ហាញ Warning ប្រសិនបើ File មិនមាន ហើយ Script អាចបន្ត; `require` បង្កើត Error/Throwable ហើយមិនបន្តតាមធម្មតា។
3. វាជួយមិនឱ្យ File ដដែលត្រូវបានបញ្ចូលច្រើនដង។
4. វាផ្តល់ទីតាំង Directory ពេញលេញរបស់ File PHP បច្ចុប្បន្ន។
5. បាន ប្រសិនបើ Variable នោះស្ថិតនៅ Scope ដែល Include កើតឡើង។
6. Include បញ្ចូល និងដំណើរការ File នៅក្នុង Request ដដែល; Redirect បញ្ជូន Browser ទៅ URL ផ្សេង។

---

## សេចក្តីសង្ខេប

ក្នុងមេរៀននេះ អ្នកបានរៀនពី `include`, `require`, `include_once`, `require_once`, `__DIR__`, Variable Scope និងការរៀបចំ Header/Footer ជា File ដាច់ដោយឡែក។

**មេរៀនទី ១១ បន្ទាប់៖ PHP Sessions និង Cookies** — រៀនពីរបៀបរក្សាទុកស្ថានភាពអ្នកប្រើប្រាស់រវាង Request ដូចជា Login Session និង Cookie។
