# មេរៀនទី ១២៖ PHP File Handling (ការគ្រប់គ្រងឯកសារ)

## ១. គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពីមូលហេតុដែលកម្មវិធីត្រូវអាន និងសរសេរ File។
- ប្រើ `file_exists()`, `is_file()`, `is_dir()` និង `filesize()`។
- បើក និងបិទ File ដោយ `fopen()` និង `fclose()`។
- អានទិន្នន័យដោយ `fread()`, `fgets()` និង `file_get_contents()`។
- សរសេរ និងបន្ថែមទិន្នន័យដោយ `fwrite()` និង `file_put_contents()`។
- ប្រើ File Modes ដូចជា `r`, `w`, `a`, `x`។
- បង្កើតកម្មវិធី Note App តូចមួយ។
- យល់ពីសុវត្ថិភាពពេលទទួល File Path ពីអ្នកប្រើ។

## ២. តើ File Handling ជាអ្វី?

File Handling គឺជាការប្រើ PHP ដើម្បីធ្វើការជាមួយឯកសារនៅលើ Server ដូចជា៖

- អានអត្ថបទពី File។
- សរសេរអត្ថបទទៅ File។
- បន្ថែមអត្ថបទទៅចុង File។
- ពិនិត្យថា File មានឬអត់។
- បង្កើត និងលុប File។
- អានបញ្ជីឯកសារនៅក្នុង Folder។

ឧទាហរណ៍ កម្មវិធី Blog អាចរក្សាទុក Log ឬទិន្នន័យសាមញ្ញក្នុង File។ សម្រាប់ទិន្នន័យច្រើន ឬទិន្នន័យដែលត្រូវស្វែងរក និងទាក់ទងគ្នា គួរប្រើ Database ដូចជា MySQL។

## ៣. ពិនិត្យព័ត៌មានអំពី File

### ៣.១ file_exists()

ពិនិត្យថា File ឬ Path មានឬអត់៖

```php
<?php
$path = __DIR__ . '/notes.txt';

if (file_exists($path)) {
    echo 'File មានរួចហើយ។';
} else {
    echo 'មិនទាន់មាន File ទេ។';
}
```

### ៣.២ is_file() និង is_dir()

```php
<?php
$path = __DIR__ . '/notes.txt';

if (is_file($path)) {
    echo 'នេះជា File។';
}

$folder = __DIR__ . '/documents';

if (is_dir($folder)) {
    echo 'នេះជា Directory។';
}
```

### ៣.៣ filesize()

```php
<?php
$path = __DIR__ . '/notes.txt';

if (is_file($path)) {
    echo 'ទំហំ File: ' . filesize($path) . ' bytes';
}
```

ចំណាំ៖ ត្រូវពិនិត្យថា File មាន និងអាចអានបាន មុនប្រើ Function ដែលត្រូវការទំហំ ឬមាតិកា។

## ៤. បើក File ដោយ fopen()

`fopen()` ប្រើសម្រាប់បើក File ហើយត្រឡប់ Resource ឬ `false` ប្រសិនបើបើកមិនបាន។

```php
<?php
$file = fopen(__DIR__ . '/notes.txt', 'r');

if ($file === false) {
    exit('មិនអាចបើក File បានទេ។');
}

fclose($file);
```

កុំភ្លេច `fclose()` បន្ទាប់ពីប្រើ File រួច ដើម្បីបិទ Resource។

## ៥. File Modes សំខាន់ៗ

Mode ជាអក្សរដែលប្រាប់ PHP ថាតើត្រូវបើក File ដើម្បីអាន ឬសរសេរ។

| Mode | អត្ថន័យ |
|---|---|
| `r` | អានតែប៉ុណ្ណោះ; File ត្រូវមានស្រាប់ |
| `r+` | អាន និងសរសេរ; File ត្រូវមានស្រាប់ |
| `w` | សរសេរ; បង្កើត File ថ្មី ឬលុបមាតិកាចាស់ |
| `w+` | អាន និងសរសេរ; បង្កើត ឬកាត់មាតិកា File ឱ្យទទេ |
| `a` | បន្ថែមទៅចុង File; បង្កើតបើមិនទាន់មាន |
| `a+` | អាន និងបន្ថែមទៅចុង File |
| `x` | បង្កើត File ថ្មី; បរាជ័យបើ File មានរួច |
| `x+` | បង្កើត File ថ្មីសម្រាប់អាន និងសរសេរ |

**ប្រយ័ត្ន៖** Mode `w` និង `w+` អាចលុបមាតិកាចាស់ទាំងអស់។ កុំប្រើវាលើ File ដែលមានទិន្នន័យសំខាន់ ដោយមិនបានយល់ច្បាស់។

## ៦. អាន File ដោយ fread()

`fread()` អានចំនួន Bytes ដែលបានកំណត់ពី File។

```php
<?php
$path = __DIR__ . '/notes.txt';
$file = fopen($path, 'r');

if ($file === false) {
    exit('មិនអាចបើក File បានទេ។');
}

$content = fread($file, filesize($path));

fclose($file);

echo htmlspecialchars($content, ENT_QUOTES, 'UTF-8');
```

ឧទាហរណ៍នេះសន្មតថា File មានទំហំធំជាងសូន្យ។ សម្រាប់ File ទទេ ត្រូវដោះស្រាយដោយឡែក។

## ៧. អាន File តាមបន្ទាត់ដោយ fgets()

`fgets()` អានមួយបន្ទាត់ក្នុងមួយដង។

```php
<?php
$path = __DIR__ . '/students.txt';
$file = fopen($path, 'r');

if ($file === false) {
    exit('មិនអាចបើក File បានទេ។');
}

while (($line = fgets($file)) !== false) {
    echo htmlspecialchars($line, ENT_QUOTES, 'UTF-8') . '<br>';
}

fclose($file);
```

វាមានប្រយោជន៍ពេលអាន File ដែលមានច្រើនបន្ទាត់។

## ៨. អាន File ងាយៗដោយ file_get_contents()

ប្រសិនបើអ្នកចង់អានមាតិកា File ទាំងមូល អាចប្រើ៖

```php
<?php
$path = __DIR__ . '/notes.txt';

$content = file_get_contents($path);

if ($content === false) {
    exit('មិនអាចអាន File បានទេ។');
}

echo nl2br(htmlspecialchars($content, ENT_QUOTES, 'UTF-8'));
```

- `file_get_contents()` អាន File ទាំងមូលទៅជា String។
- `nl2br()` បម្លែង Newline ទៅជា HTML Line Break សម្រាប់បង្ហាញ។
- `htmlspecialchars()` Escape អត្ថបទមុនបង្ហាញក្នុង HTML។

សម្រាប់ File ធំខ្លាំង គួរអានជាបន្ទាត់ ឬជាផ្នែកៗ ដើម្បីកុំឱ្យប្រើ Memory ច្រើនពេក។

## ៩. សរសេរ File ដោយ fwrite()

`fwrite()` សរសេរទិន្នន័យទៅ File ដែលបានបើក។

```php
<?php
$path = __DIR__ . '/notes.txt';
$file = fopen($path, 'w');

if ($file === false) {
    exit('មិនអាចបើក File សម្រាប់សរសេរបានទេ។');
}

$text = "រៀន PHP File Handling\n";
fwrite($file, $text);

fclose($file);

echo 'សរសេរ File រួចរាល់។';
```

ដោយប្រើ Mode `w` មាតិកាចាស់នឹងត្រូវកាត់ចោលមុនសរសេរ។

## ១០. បន្ថែមទិន្នន័យដោយ Mode a

ប្រសិនបើអ្នកចង់បន្ថែមអត្ថបទទៅចុង File ដោយមិនលុបមាតិកាចាស់ ប្រើ Mode `a`៖

```php
<?php
$path = __DIR__ . '/notes.txt';
$file = fopen($path, 'a');

if ($file === false) {
    exit('មិនអាចបើក File បានទេ។');
}

fwrite($file, "មេរៀនថ្មីមួយទៀត\n");

fclose($file);
```

## ១១. ប្រើ file_put_contents()

សម្រាប់ការសរសេរងាយៗ អ្នកអាចប្រើ `file_put_contents()` ដោយមិនចាំបាច់បើក និងបិទ File ដោយដៃ។

សរសេរជំនួសមាតិកាចាស់៖

```php
<?php
$result = file_put_contents(
    __DIR__ . '/notes.txt',
    "សួស្តី PHP!\n"
);

if ($result === false) {
    echo 'សរសេរមិនបានទេ។';
} else {
    echo 'បានសរសេរ ' . $result . ' bytes។';
}
```

បន្ថែមទៅចុង File៖

```php
<?php
$result = file_put_contents(
    __DIR__ . '/notes.txt',
    "បន្ថែមមេរៀនថ្មី។\n",
    FILE_APPEND | LOCK_EX
);

if ($result === false) {
    echo 'មិនអាចបន្ថែមទិន្នន័យបានទេ។';
}
```

- `FILE_APPEND` បន្ថែមទៅចុង File។
- `LOCK_EX` ស្នើសុំ Exclusive Lock ខណៈពេលសរសេរ ដើម្បីកាត់បន្ថយការសរសេរប៉ះទង្គិចគ្នា។
- វាមិនមែនជាជំនួស Database Transaction សម្រាប់កម្មវិធីដែលមានអ្នកប្រើច្រើនទេ។

## ១២. បង្កើត Folder និង File

បង្កើត Folder៖

```php
<?php
$folder = __DIR__ . '/storage';

if (!is_dir($folder)) {
    if (!mkdir($folder, 0750, true) && !is_dir($folder)) {
        exit('មិនអាចបង្កើត Folder បានទេ។');
    }
}

echo 'Folder ត្រៀមរួចរាល់។';
```

បង្កើត File ឬសរសេរទៅ File៖

```php
<?php
$path = __DIR__ . '/storage/example.txt';

if (file_put_contents($path, 'Hello PHP') === false) {
    exit('មិនអាចបង្កើត File បានទេ។');
}

echo 'File ត្រូវបានបង្កើត។';
```

សម្រាប់កម្មវិធីពិត ត្រូវរៀបចំ Permission ឱ្យសមស្រប ហើយកុំដាក់ File ដែលមានទិន្នន័យឯកជននៅក្នុង Public Web Root ប្រសិនបើអ្នកប្រើអាចចូលមើលវាដោយផ្ទាល់។

## ១៣. លុប File ដោយ unlink()

`unlink()` លុប File៖

```php
<?php
$path = __DIR__ . '/old-note.txt';

if (is_file($path)) {
    if (unlink($path)) {
        echo 'លុប File រួចរាល់។';
    } else {
        echo 'មិនអាចលុប File បានទេ។';
    }
} else {
    echo 'រកមិនឃើញ File ទេ។';
}
```

**ប្រយ័ត្ន៖** ការលុប File ជាសកម្មភាពដែលអាចបាត់បង់ទិន្នន័យ។ កុំយក Path ពីអ្នកប្រើទៅដាក់ក្នុង `unlink()` ដោយផ្ទាល់។

## ១៤. អានបញ្ជី File ក្នុង Folder

ប្រើ `scandir()`៖

```php
<?php
$folder = __DIR__ . '/storage';
$items = scandir($folder);

if ($items === false) {
    exit('មិនអាចអាន Folder បានទេ។');
}

foreach ($items as $item) {
    if ($item === '.' || $item === '..') {
        continue;
    }

    echo htmlspecialchars($item, ENT_QUOTES, 'UTF-8') . '<br>';
}
```

`.` និង `..` ជា Entry ពិសេសសម្រាប់ Directory បច្ចុប្បន្ន និង Directory មេ។

## ១៥. សុវត្ថិភាពពេលធ្វើការជាមួយ File

ចំណុចទាំងនេះសំខាន់ណាស់៖

1. **កុំទុកចិត្ត File Path ពីអ្នកប្រើ។** អ្នកប្រើអាចផ្ញើ Path ដែលព្យាយាមចូលទៅ File ផ្សេងក្រៅ Folder ដែលអនុញ្ញាត។
2. **កំណត់ទីតាំងឯកសារដែលអនុញ្ញាត។** ប្រើ Directory ថេររបស់កម្មវិធី និងឈ្មោះ File ដែលកម្មវិធីបានកំណត់។
3. **កុំភ្ជាប់ Input ទៅ Path ដោយផ្ទាល់។** កុំសរសេរ `file_get_contents($_GET['file'])`។
4. **ពិនិត្យ Permission។** Web Server ត្រូវមានសិទ្ធិចាំបាច់ប៉ុណ្ណោះ។
5. **Escape មុនបង្ហាញក្នុង HTML។** File Content ក៏អាចមានអត្ថបទមិនទុកចិត្តបានដែរ។
6. **ប្រើ File Lock នៅពេលសមស្រប។** `LOCK_EX` អាចជួយកាត់បន្ថយការសរសេរប៉ះទង្គិចគ្នា។
7. **កុំរក្សាទុក Password ជា Plain Text។**
8. **ប្រើ Database សម្រាប់ទិន្នន័យស្មុគស្មាញ។** File មិនសមស្របសម្រាប់គ្រប់ប្រភេទទិន្នន័យទេ។

## ១៦. Mini Project៖ Simple Note App

កម្មវិធីនេះអនុញ្ញាតឱ្យអ្នកប្រើបញ្ចូល Note ហើយរក្សាទុកទៅ File។ យើងកំណត់ File Path នៅក្នុងកម្មវិធី មិនទទួល Path ពីអ្នកប្រើទេ។

បង្កើត Folder៖

```
php-note-app/
│
├── index.php
└── storage/
    └── notes.txt
```

បង្កើត Folder `storage` ជាមុន ហើយបង្កើត File `index.php`៖

```php
<?php
$storageDir = __DIR__ . '/storage';
$notesFile = $storageDir . '/notes.txt';

if (!is_dir($storageDir)) {
    if (!mkdir($storageDir, 0750, true) && !is_dir($storageDir)) {
        exit('មិនអាចបង្កើត Storage Folder បានទេ។');
    }
}

$message = '';
$error = '';

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $note = trim($_POST['note'] ?? '');

    if ($note === '') {
        $error = 'សូមបញ្ចូល Note ជាមុន។';
    } elseif (mb_strlen($note) > 1000) {
        $error = 'Note មិនអាចលើស 1000 តួអក្សរបានទេ។';
    } else {
        $line = date('Y-m-d H:i:s') . ' | ' . $note . PHP_EOL;

        $saved = file_put_contents(
            $notesFile,
            $line,
            FILE_APPEND | LOCK_EX
        );

        if ($saved === false) {
            $error = 'មិនអាចរក្សាទុក Note បានទេ។';
        } else {
            $message = 'រក្សាទុក Note រួចរាល់។';
        }
    }
}

$notes = [];

if (is_file($notesFile)) {
    $contents = file($notesFile, FILE_IGNORE_NEW_LINES);

    if ($contents !== false) {
        $notes = $contents;
    }
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PHP Note App</title>
</head>
<body>
    <h1>កម្មវិធីកត់ត្រា</h1>

    <?php if ($message !== ''): ?>
        <p><?= htmlspecialchars($message, ENT_QUOTES, 'UTF-8') ?></p>
    <?php endif; ?>

    <?php if ($error !== ''): ?>
        <p><?= htmlspecialchars($error, ENT_QUOTES, 'UTF-8') ?></p>
    <?php endif; ?>

    <form method="POST">
        <label for="note">សរសេរ Note៖</label>
        <br>
        <textarea
            id="note"
            name="note"
            maxlength="1000"
            required
        ></textarea>
        <br>
        <button type="submit">រក្សាទុក</button>
    </form>

    <h2>Note ទាំងអស់</h2>

    <?php if ($notes === []): ?>
        <p>មិនទាន់មាន Note ទេ។</p>
    <?php else: ?>
        <ul>
            <?php foreach (array_reverse($notes) as $note): ?>
                <li><?= htmlspecialchars($note, ENT_QUOTES, 'UTF-8') ?></li>
            <?php endforeach; ?>
        </ul>
    <?php endif; ?>
</body>
</html>
```

### របៀបដំណើរការ

1. បើក Terminal នៅក្នុង Folder `php-note-app`។
2. ដំណើរការ៖

```bash
php -S localhost:8000
```

3. បើក Browser ទៅ `http://localhost:8000/index.php`។
4. បញ្ចូល Note ហើយចុចរក្សាទុក។
5. Refresh ទំព័រ ហើយពិនិត្យថា Note នៅតែមានឬអត់។
6. បើក File `storage/notes.txt` ដើម្បីមើលទិន្នន័យដែលបានរក្សាទុក។

**ចំណាំសុវត្ថិភាព៖** Demo នេះសម្រាប់ Local Learning ប៉ុណ្ណោះ។ ប្រសិនបើដាក់ Online ត្រូវការការការពារ CSRF, ការគ្រប់គ្រងអ្នកប្រើ និងការកំណត់ Storage មិនឱ្យ Public Access។

## ១៧. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Read File

បង្កើត `about.txt` ដែលមានព័ត៌មានអំពីខ្លួនអ្នក ហើយប្រើ `file_get_contents()` ដើម្បីអានវា។

### លំហាត់ទី ២៖ Write File

បង្កើត Form ដែលទទួលឈ្មោះ ហើយរក្សាទុកទៅ `students.txt`។ ត្រូវ Validate ឈ្មោះ និង Escape ពេលបង្ហាញ។

### លំហាត់ទី ៣៖ Append File

សរសេរកម្មវិធីបន្ថែម Log ថ្មីទៅចុង File ដោយមិនលុប Log ចាស់។

### លំហាត់ទី ៤៖ File Information

បង្កើតកម្មវិធីពិនិត្យថា File មានឬអត់ ហើយបង្ហាញទំហំជា Bytes។

### លំហាត់ទី ៥៖ Directory Listing

ប្រើ `scandir()` ដើម្បីបង្ហាញបញ្ជីឈ្មោះ File ក្នុង Folder មួយ។ កុំអនុញ្ញាតឱ្យអ្នកប្រើជ្រើស Path ណាមួយដោយសេរី។

## ១៨. សំណួររំលឹកមេរៀន

1. តើ File Handling មានន័យដូចម្តេច?
2. តើ `fopen()` ប្រើសម្រាប់អ្វី?
3. តើ Mode `r`, `w` និង `a` ខុសគ្នាដូចម្តេច?
4. តើ `fread()` និង `fgets()` ខុសគ្នាដូចម្តេច?
5. តើ `file_get_contents()` ប្រើសម្រាប់អ្វី?
6. តើ `file_put_contents()` ជាមួយ `FILE_APPEND` មានន័យដូចម្តេច?
7. តើហេតុអ្វីមិនគួរយក File Path ពីអ្នកប្រើទៅប្រើដោយផ្ទាល់?
8. តើ `unlink()` ប្រើសម្រាប់អ្វី?

## ១៩. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

1. ការអាន សរសេរ បង្កើត បន្ថែម និងគ្រប់គ្រង File ដោយ PHP។
2. បើក File សម្រាប់អាន ឬសរសេរ តាម Mode ដែលបានកំណត់។
3. `r` អាន; `w` សរសេរដោយកាត់មាតិកាចាស់; `a` បន្ថែមនៅចុង File។
4. `fread()` អានចំនួន Bytes; `fgets()` អានមួយបន្ទាត់។
5. អានមាតិកា File ទាំងមូលទៅជា String។
6. បន្ថែមទិន្នន័យទៅចុង File ដោយមិនលុបមាតិកាចាស់។
7. ព្រោះអាចបង្កហានិភ័យ Path Traversal និងការចូលដល់ File ដែលមិនគួរឱ្យអ្នកប្រើចូល។
8. លុប File។

---

## សេចក្តីសង្ខេប

ក្នុងមេរៀននេះ អ្នកបានរៀនពី File Existence, File Modes, `fopen()`, `fread()`, `fgets()`, `fwrite()`, `file_get_contents()`, `file_put_contents()`, `scandir()` និង `unlink()`។

**មេរៀនទី ១៣ បន្ទាប់៖ PHP Error Handling និង Exceptions** — រៀនគ្រប់គ្រង Error, Warning, Exception, `try/catch/finally` និងបង្កើត Error Handling ឱ្យបានល្អ។
