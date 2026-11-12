# មេរៀនទី ១៖ ស្វែងយល់អំពី PHP និងរៀបចំបរិស្ថាន

**កម្រិត៖** Beginner  
**ផ្នែក៖** PHP Fundamentals  
**គោលដៅ៖** អាចបង្កើត និងដំណើរការកម្មវិធី PHP ដំបូងបាន។

## ១. គោលបំណង

ក្រោយរៀនចប់ អ្នកគួរតែអាច៖

1. ពន្យល់បានថា PHP ជាអ្វី និងប្រើសម្រាប់អ្វី។
2. យល់ពីភាពខុសគ្នារវាង PHP និង HTML។
3. រៀបចំ PHP នៅលើកុំព្យូទ័រ។
4. ពិនិត្យ PHP Version តាម Terminal។
5. បង្កើតឯកសារ `.php` និងប្រើ `echo`។
6. ដំណើរការ PHP តាម Terminal និង Browser។

## ២. តម្រូវការមុនចាប់ផ្ដើម

- កុំព្យូទ័រ Windows, Linux ឬ macOS។
- Visual Studio Code ឬ Text Editor ផ្សេង។
- PHP CLI ដែលមាន Version នៅក្នុងបញ្ជី Supported Versions។
- Terminal / Command Prompt។
- Web Browser ដូចជា Chrome, Firefox ឬ Edge។

> ព័ត៌មាន Version និងការគាំទ្រអាចផ្លាស់ប្ដូរ។ សូមពិនិត្យ [PHP Supported Versions](https://www.php.net/supported-versions.php) និងទាញយកពី [PHP Downloads](https://www.php.net/downloads.php) មុនដំឡើង។

## ៣. តើ PHP ជាអ្វី?

PHP មានឈ្មោះពេញថា **PHP: Hypertext Preprocessor**។ វាជា Programming Language ដែលប្រើជាចម្បងសម្រាប់បង្កើត Web Applications និងដំណើរការកូដនៅ Server។ វាក៏អាចដំណើរការតាម Command Line បានដែរ។

PHP អាចប្រើសម្រាប់៖

- បង្កើត Website និង Web Applications។
- ទទួល និងដំណើរការទិន្នន័យពី Form។
- ភ្ជាប់ និងធ្វើការជាមួយ Database។
- បង្កើត Login និង Register។
- បង្កើត REST API។
- បង្កើត Backend សម្រាប់កម្មវិធីផ្សេងៗ។

### PHP ដំណើរការជាមួយ Browser យ៉ាងដូចម្ដេច?

1. អ្នកប្រើបើក Website តាម Browser។
2. Browser ផ្ញើ HTTP Request ទៅ Web Server។
3. Server ដំណើរការ PHP ហើយអាចទាក់ទង Database ប្រសិនបើត្រូវការ។
4. Server ផ្ញើ Response ត្រឡប់ទៅ Browser ជាទូទៅជា HTML។
5. Browser បង្ហាញលទ្ធផល។

ចំណុចសំខាន់៖ Browser ជាទូទៅទទួលបាន HTML Response មិនមែន PHP Source Code ដើមទេ។

## ៤. ភាពខុសគ្នារវាង PHP និង HTML

| PHP | HTML |
|---|---|
| Programming Language | Markup Language |
| អាចគណនា និងដំណើរការលក្ខខណ្ឌ | រៀបចំរចនាសម្ព័ន្ធ Web Page |
| ដំណើរការនៅ Server ឬ CLI | Browser ប្រើសម្រាប់បង្ហាញទំព័រ |
| ជាទូទៅប្រើឯកសារ `.php` | ជាទូទៅប្រើឯកសារ `.html` |

ឧទាហរណ៍ HTML៖

```html
<h1>សួស្តី</h1>
<p>ខ្ញុំកំពុងរៀន Web Development</p>
```

ឧទាហរណ៍ PHP៖

```php
<?php
echo "សួស្តី";
```

PHP និង HTML អាចសរសេររួមគ្នាក្នុងឯកសារ `.php` តែមួយបាន។

## ៥. រៀបចំបរិស្ថាន PHP

### Windows

1. ចូល [PHP for Windows](https://windows.php.net/download/)។
2. ជ្រើសកញ្ចប់ PHP ដែលសមស្របនឹងប្រព័ន្ធរបស់អ្នក។
3. Extract ទៅ Folder ឧទាហរណ៍ `C:\\php`។
4. បន្ថែម Folder នោះទៅក្នុង Environment Variable `Path`។
5. បិទ Terminal ចាស់ ហើយបើក Terminal ថ្មី។
6. វាយពាក្យបញ្ជា៖

```powershell
php -v
```

បើប្រើ XAMPP អាចប្រើ PHP ដែលភ្ជាប់មកជាមួយបាន ប៉ុន្តែត្រូវពិនិត្យ Version ជាមុន។

### Ubuntu / Debian

```bash
sudo apt update
sudo apt install php-cli
php -v
```

ចំណាំ៖ Version ក្នុង Linux Repository អាចខុសពី Version ថ្មីបំផុតនៅ PHP Website។ ពិនិត្យ Version និងការគាំទ្រមុនប្រើ។

### macOS (Homebrew)

បើបានដំឡើង Homebrew រួច៖

```bash
brew install php
php -v
```

ឯកសារយោង៖ [PHP Installation Manual](https://www.php.net/manual/en/install.php)

## ៦. ពិនិត្យថា PHP ដំណើរការឬអត់

បើក Terminal ហើយវាយ៖

```bash
php -v
```

ដើម្បីសាកល្បងដំណើរការកូដតូចមួយ៖

```bash
php -r "echo 'PHP is working!';"
```

លទ្ធផលរំពឹងទុក៖

```text
PHP is working!
```

## ៧. បង្កើតកម្មវិធី PHP ដំបូង

បង្កើត Folder ឈ្មោះ `PHP-Course` ហើយនៅក្នុងនោះបង្កើតឯកសារ `hello.php`។

ដាក់កូដ៖

```php
<?php

// My first PHP program

echo "Hello World!\n";
echo "សួស្តី! ខ្ញុំកំពុងរៀន PHP។\n";
echo "នេះជាកម្មវិធី PHP ដំបូងរបស់ខ្ញុំ។\n";
```

ដំណើរការតាម Terminal៖

```bash
php hello.php
```

លទ្ធផលរំពឹងទុក៖

```text
Hello World!
សួស្តី! ខ្ញុំកំពុងរៀន PHP។
នេះជាកម្មវិធី PHP ដំបូងរបស់ខ្ញុំ។
```

### ពន្យល់កូដ

- `<?php` ចាប់ផ្ដើមផ្នែក PHP។
- `//` ចាប់ផ្ដើម Comment មួយបន្ទាត់។
- `echo` បញ្ចេញអត្ថបទ ឬលទ្ធផល។
- អត្ថបទក្នុងសញ្ញា Quote គឺជា String។
- `\n` បង្កើតបន្ទាត់ថ្មីនៅក្នុង Terminal។
- `;` បញ្ចប់ Statement។

## ៨. បញ្ចូល PHP ក្នុង HTML និងបើកតាម Browser

បង្កើតឯកសារ `index.php`៖

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My First PHP Website</title>
</head>
<body>
    <h1>សូមស្វាគមន៍មកកាន់វគ្គ PHP</h1>

    <p>
        <?php
            echo "សួស្តី! នេះគឺជា PHP នៅក្នុង HTML។";
        ?>
    </p>
</body>
</html>
```

ពន្យល់៖

- `<!DOCTYPE html>` បញ្ជាក់ HTML5។
- `<html lang="km">` កំណត់ភាសាទំព័រជាខ្មែរ។
- `<meta charset="UTF-8">` កំណត់ Character Encoding។
- `<h1>` បង្កើតចំណងជើង។
- `<?php ... ?>` ជាផ្នែក PHP។
- `echo` បញ្ចេញអត្ថបទទៅក្នុង HTML Response។

បើក Terminal នៅក្នុង Folder គម្រោង ហើយដំណើរការ៖

```bash
php -S 127.0.0.1:8000
```

បើក Browser ទៅកាន់៖

`http://127.0.0.1:8000/index.php`

ដើម្បីបញ្ឈប់ Server ចុច `Ctrl + C` ក្នុង Terminal។

> **សុវត្ថិភាព៖** PHP Built-in Web Server សម្រាប់ Development និង Testing ប៉ុណ្ណោះ។ កុំប្រើវាជា Production Server ឬបើកទៅ Public Network។

## ៩. ពាក្យបច្ចេកទេស

| ពាក្យ | អត្ថន័យ |
|---|---|
| Programming Language | ភាសាសម្រាប់សរសេរកម្មវិធី |
| Server | ប្រព័ន្ធផ្ដល់សេវាកម្ម |
| Client | អ្នកស្នើសុំសេវាកម្ម |
| Backend | ផ្នែកដំណើរការខាងក្រោយ |
| Frontend | ផ្នែកដែលអ្នកប្រើមើលឃើញ |
| PHP Runtime | បរិស្ថានដំណើរការកូដ PHP |
| CLI | Command Line Interface |
| Browser | កម្មវិធីបើក Website |
| Statement | សេចក្ដីបញ្ជាក្នុងកម្មវិធី |
| Comment | អត្ថបទពន្យល់ក្នុងកូដ |
| Syntax | ច្បាប់សរសេរកូដ |
| Extension | ផ្នែកបន្ថែមឈ្មោះឯកសារ |

## ១០. កំហុសទូទៅ

| កំហុស | មូលហេតុ / ដំណោះស្រាយ |
|---|---|
| `php is not recognized` | ពិនិត្យ PHP PATH ហើយបើក Terminal ថ្មី។ |
| `Could not open input file` | ពិនិត្យ Folder បច្ចុប្បន្ន និងឈ្មោះឯកសារ។ |
| `Parse error` | ពិនិត្យ Syntax, Quote និង Semicolon។ |
| Browser បង្ហាញ PHP Code | កុំបើក File ដោយផ្ទាល់។ ប្រើ PHP Server។ |
| អក្សរខ្មែរបង្ហាញខុស | រក្សាទុកឯកសារជា UTF-8 និងពិនិត្យ Meta Charset។ |
| Port 8000 កំពុងប្រើ | បិទកម្មវិធីដែលកាន់ Port ឬសាក Port 8001។ |

## ១១. លំហាត់អនុវត្ត

សូមសាកល្បងដោយខ្លួនឯងមុនមើលចម្លើយនៅផ្នែកទី ១៣។

### លំហាត់ទី ១ — ងាយ

បង្កើត `exercise1.php` ហើយប្រើ `echo` បង្ហាញ៖

```text
My name is [Your Name]
I am learning PHP
This is Lesson 1
```

### លំហាត់ទី ២ — ងាយទៅមធ្យម

ប្រើ `echo` បង្ហាញពាក្យ PHP, HTML, CSS, JavaScript និង MySQL ដោយមួយបន្ទាត់ក្នុងមួយពាក្យ។

### លំហាត់ទី ៣ — មធ្យម

បង្កើត `about.php` ដែលមាន Comment មួយ ចំណងជើង About Me និងប្រើ `echo` យ៉ាងហោចណាស់ពីរដង។

### លំហាត់ទី ៤ — មធ្យមទៅពិបាក

បង្កើត Web Page ដោយប្រើ HTML និង PHP រួមគ្នា។ ត្រូវមានចំណងជើងជាភាសាខ្មែរ Paragraph សារស្វាគមន៍ និង UTF-8។

### លំហាត់ទី ៥ — ប្រកួតប្រជែង

បង្កើតរចនាសម្ព័ន្ធ៖

```text
my-first-php-project/
├── index.php
├── about.php
└── README.txt
```

- `index.php` បង្ហាញទំព័រស្វាគមន៍។
- `about.php` បង្ហាញព័ត៌មានអំពីគម្រោង។
- `README.txt` ពន្យល់ពីរបៀបដំណើរការ។

## ១២. Mini Project៖ My First PHP Welcome Website

បង្កើត Folder `welcome-website` និងឯកសារ `index.php`៖

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHP Welcome Website</title>
</head>
<body>
    <h1>សូមស្វាគមន៍!</h1>
    <h2>My First PHP Website</h2>

    <p>
        <?php
            echo "ខ្ញុំកំពុងរៀន PHP ពីកម្រិត Beginner។";
        ?>
    </p>

    <p>
        <?php
            echo "ខ្ញុំនឹងបន្តរៀនរហូតដល់ Advanced។";
        ?>
    </p>
</body>
</html>
```

សាកល្បងដោយដំណើរការ៖

```bash
php -S 127.0.0.1:8000
```

បើក `http://127.0.0.1:8000`។ គម្រោងជោគជ័យនៅពេល Browser បង្ហាញចំណងជើង និងអត្ថបទខ្មែរត្រឹមត្រូវ។

## ១៣. ចម្លើយលំហាត់ (ពិនិត្យក្រោយសាកល្បង)

### ចម្លើយលំហាត់ទី ១

```php
<?php
echo "My name is Sotheary\n";
echo "I am learning PHP\n";
echo "This is Lesson 1\n";
```

### ចម្លើយលំហាត់ទី ២

```php
<?php
echo "PHP\n";
echo "HTML\n";
echo "CSS\n";
echo "JavaScript\n";
echo "MySQL\n";
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
// About me information

echo "About Me\n";
echo "I am learning Web Development.\n";
echo "I am starting with PHP.\n";
```

### ចម្លើយលំហាត់ទី ៤

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>PHP Website</title>
</head>
<body>
    <h1>គេហទំព័ររបស់ខ្ញុំ</h1>
    <p><?php echo "សូមស្វាគមន៍មកកាន់ Website របស់ខ្ញុំ!"; ?></p>
</body>
</html>
```

### ចម្លើយលំហាត់ទី ៥

`index.php`៖

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>My Project</title>
</head>
<body>
    <h1>My First PHP Project</h1>
    <p><?php echo "សូមស្វាគមន៍!"; ?></p>
    <a href="about.php">About Project</a>
</body>
</html>
```

`about.php`៖

```php
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <title>About</title>
</head>
<body>
    <h1>About This Project</h1>
    <p><?php echo "គម្រោងនេះបង្កើតឡើងដើម្បីហាត់ PHP។"; ?></p>
    <a href="index.php">Home</a>
</body>
</html>
```

`README.txt`៖

```text
My First PHP Project

1. Install PHP.
2. Open terminal in this folder.
3. Run: php -S 127.0.0.1:8000
4. Open http://127.0.0.1:8000
```

## ១៤. សំណួរត្រួតពិនិត្យ

1. តើ PHP ជាអ្វី?
2. តើ PHP និង HTML ខុសគ្នាដូចម្ដេច?
3. តើឯកសារ PHP ជាទូទៅប្រើ Extension អ្វី?
4. តើ `echo` មានតួនាទីអ្វី?
5. តើ `//` ប្រើសម្រាប់អ្វី?
6. តើ Command ណាដំណើរការ File PHP តាម Terminal?
7. ហេតុអ្វីបានជាមិនគួរបើក File PHP ដោយផ្ទាល់ក្នុង Browser?
8. តើ Command ណាចាប់ផ្ដើម PHP Development Server?

## ១៥. សង្ខេប និងលក្ខខណ្ឌមុនទៅមេរៀនបន្ទាប់

មុនទៅមេរៀនទី ២ សូមប្រាកដថាអ្នកអាច៖

- ពិនិត្យ PHP Version ដោយ `php -v`។
- បង្កើត និងដំណើរការ `hello.php`។
- ប្រើ `echo` និង Comment។
- បញ្ចូល PHP ក្នុង HTML។
- បើក `index.php` តាម Local Development Server។

## ១៦. មេរៀនបន្ទាប់

**មេរៀនទី ២៖ Variables និង Data Types ក្នុង PHP**

ប្រធានបទនឹងរួមមាន៖ Variables, Naming Rules, String, Integer, Float, Boolean, Array, Null, `var_dump()` និង `gettype()` ព្រមទាំងលំហាត់ និង Mini Project។

## ឯកសារយោងផ្លូវការ

- [PHP Supported Versions](https://www.php.net/supported-versions.php)
- [PHP Manual](https://www.php.net/manual/en/)
- [PHP Installation](https://www.php.net/manual/en/install.php)
- [PHP Basic Syntax](https://www.php.net/manual/en/language.basic-syntax.php)
- [PHP echo](https://www.php.net/manual/en/function.echo.php)
- [PHP Built-in Web Server](https://www.php.net/manual/en/features.commandline.webserver.php)
