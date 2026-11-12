
# មេរៀនទី ៣០៖ សុវត្ថិភាព Upload ឯកសារនៅក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- អានរចនាសម្ព័ន្ធ `$_FILES` និងពិនិត្យ Upload Error។
- កំណត់ដែនកំណត់ទំហំ និងប្រភេទឯកសារដោយ Allowlist។
- បង្កើតឈ្មោះឯកសារថ្មីដោយ Random Bytes។
- ផ្ទុកឯកសារនៅក្រៅ Web Root និងបម្រើឯកសារដោយមានការអនុញ្ញាត។
- ជៀសវាងការទុកចិត្តលើឈ្មោះឯកសារ និង MIME Type ដែល Browser ផ្ញើមក។

> ឧទាហរណ៍នេះសម្រាប់រូបភាព JPEG/PNG។ ការត្រួតពិនិត្យ File មួយមុខមិនអាចធានាថាឯកសារមានសុវត្ថិភាពទាំងស្រុងទេ។

## ១. ហានិភ័យនៃ File Upload

ឯកសារដែលអ្នកប្រើផ្ញើមកគឺទិន្នន័យមិនទុកចិត្ត។ បញ្ហាដែលអាចកើតមាន៖

- ឯកសារធំពេកធ្វើឱ្យប្រើ Storage ឬ Bandwidth ច្រើន។
- ឯកសារក្លែងប្រភេទអាចឆ្លងកាត់ការពិនិត្យតាម Extension ឬ Browser MIME Type។
- ឈ្មោះឯកសារមាន Path ដូចជា `../../` អាចនាំឱ្យសរសេរខុសទីតាំង ប្រសិនបើយកទៅប្រើផ្ទាល់។
- ឯកសារដែលដាក់ក្នុង Web Root អាចត្រូវបានបើកដោយផ្ទាល់ ឬអាចត្រូវ Server ប្រតិបត្តិ ប្រសិនបើ Configuration ខុស។
- អ្នកប្រើអាច Upload ច្រើនដងដើម្បីប្រើ Storage អស់។

ដូច្នេះ ត្រូវកំណត់ទំហំ និងចំនួន, ប្រើ Allowlist តាមតម្រូវការពិត, បង្កើតឈ្មោះ Server-side និងរក្សាឯកសារក្រៅ Web Root។

## ២. HTML Form

Form ត្រូវមាន `enctype="multipart/form-data"`៖

```html
<form method="post" action="/upload" enctype="multipart/form-data">
    <label for="photo">ជ្រើសរូបភាព JPEG ឬ PNG</label>
    <input id="photo" type="file" name="photo" accept=".jpg,.jpeg,.png" required>
    <button type="submit">Upload</button>
</form>
```

`accept` គ្រាន់តែជួយ UI ជ្រើសឯកសារ មិនមែនជាការត្រួតពិនិត្យសុវត្ថិភាពនៅ Server ទេ។

## ៣. ពិនិត្យ Upload Error និងទំហំ

PHP ដាក់ព័ត៌មានឯកសារក្នុង `$_FILES`។ ពិនិត្យ Error Code មុនប្រើ Temporary File៖

```php
<?php
$file = $_FILES['photo'] ?? null;

if (!is_array($file) || !isset($file['error'], $file['tmp_name'], $file['size'])) {
    http_response_code(400);
    exit('សំណើ Upload មិនត្រឹមត្រូវ។');
}

if ($file['error'] !== UPLOAD_ERR_OK) {
    http_response_code(400);
    exit('Upload មិនបានជោគជ័យ។ សូមពិនិត្យទំហំ និងសាកល្បងម្ដងទៀត។');
}

$maxBytes = 2 * 1024 * 1024; // 2 MiB
if (!is_int($file['size']) || $file['size'] <= 0 || $file['size'] > $maxBytes) {
    http_response_code(413);
    exit('ឯកសារត្រូវមានទំហំក្រោម 2 MiB។');
}

if (!is_uploaded_file($file['tmp_name'])) {
    http_response_code(400);
    exit('ឯកសារ Upload មិនត្រឹមត្រូវ។');
}
```

ត្រូវកំណត់ `upload_max_filesize` និង `post_max_size` នៅ PHP Configuration ឱ្យសមនឹង Limit របស់កម្មវិធី។ កំណត់ Limit នៅ Web Server/Reverse Proxy ផងដែរ ដើម្បីកុំឱ្យ Request ធំមកដល់ PHP។

## ៤. Allowlist ប្រភេទឯកសារ និងឈ្មោះថ្មី

កុំទុកចិត្តលើ `$_FILES['photo']['type']` ដែល Client ផ្ញើមក។ អាចប្រើ Fileinfo ដើម្បីពិនិត្យ MIME ដែលស្គាល់តាមមាតិកា ហើយផ្ទៀងផ្ទាត់ Extension ដែលអនុញ្ញាត៖

```php
<?php
$allowed = [
    'image/jpeg' => 'jpg',
    'image/png' => 'png',
];

$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($file['tmp_name']);

if (!is_string($mime) || !isset($allowed[$mime])) {
    http_response_code(415);
    exit('អនុញ្ញាតតែរូបភាព JPEG ឬ PNG ប៉ុណ្ណោះ។');
}

// ឈ្មោះ Client មិនត្រូវប្រើជាឈ្មោះផ្ទុកទេ។
$storedName = bin2hex(random_bytes(16)) . '.' . $allowed[$mime];
```

MIME sniffing ក៏មិនអាចបញ្ជាក់ថាឯកសារមិនមាន Payload ព្យាបាទ ឬមិនក្លែងបន្លំបានដែរ។ សម្រាប់រូបភាពដែលត្រូវបង្ហាញជាសាធារណៈ អាច Decode និង Encode ឡើងវិញដោយ Image Library ដែលទុកចិត្តបាន។ កម្មវិធីដែលមានហានិភ័យខ្ពស់អាចត្រូវការ Antivirus ឬ Content Disarm and Reconstruction។

## ៥. ផ្លាស់ទីទៅក្រៅ Web Root

ឧទាហរណ៍៖ Web Root នៅ `/var/www/app/public`, ហើយ Upload Directory នៅ `/var/www/app-storage/uploads`៖

```php
<?php
$uploadDirectory = '/var/www/app-storage/uploads';

// Directory ត្រូវបង្កើតដោយ Deployment និងមាន Permission សមស្រប។
if (!is_dir($uploadDirectory) || !is_writable($uploadDirectory)) {
    throw new RuntimeException('Upload storage is not available.');
}

$destination = $uploadDirectory . DIRECTORY_SEPARATOR . $storedName;

if (!move_uploaded_file($file['tmp_name'], $destination)) {
    http_response_code(500);
    exit('មិនអាចរក្សាទុកឯកសារបានទេ។');
}
```

កុំបើក Permission ជាសាធារណៈទូលំទូលាយ។ គណនីរត់ PHP គួរមានសិទ្ធិត្រឹមតែអ្វីដែលត្រូវការ។ កុំដាក់ Upload Directory នៅក្រោម Public Web Root ប្រសិនបើមិនចាំបាច់។

## ៦. បម្រើឯកសារដោយពិនិត្យសិទ្ធិ

រក្សា Metadata ដូចជា ID, ឈ្មោះ Random, MIME ដែលបានកំណត់ដោយ Server, ទំហំ និង Owner ID ក្នុង Database។ ពេលអ្នកប្រើស្នើ Download៖

1. ស្វែងរកឯកសារតាម ID ដែលមានទម្រង់ត្រឹមត្រូវ។
2. ពិនិត្យថា User មានសិទ្ធិមើលឬទាញយកឯកសារនោះ។
3. បង្កើត Path ពីឈ្មោះ Server-side ដែលរកបានក្នុង Database ប៉ុណ្ណោះ។
4. ផ្ញើ MIME ដែលបាន Allowlist, `Content-Length` និង `X-Content-Type-Options: nosniff`។
5. ប្រើ `Content-Disposition: attachment` សម្រាប់ឯកសារដែលមិនចង់បង្ហាញក្នុង Browser។

កុំយក Path ឬឈ្មោះឯកសារពី Query String ទៅបើកដោយផ្ទាល់។ ឯកសារមាន Content ប្រភេទ HTML/SVG ឬឯកសារដែលមិនទុកចិត្ត គួរបម្រើជា Attachment ឬពី Origin ដាច់ដោយឡែក។

## ៧. កំហុសដែលជួបញឹកញាប់

- ពិនិត្យ Extension តែប៉ុណ្ណោះ។
- ទុកចិត្តលើ MIME Type ដែល Client ផ្ញើមក។
- ប្រើឈ្មោះឯកសារដើមជាឈ្មោះនៅលើ Disk។
- ផ្ទុក File នៅ Public Directory ហើយគិតថា URL ពិបាកទាយគឺការការពារសិទ្ធិ។
- ភ្លេចពិនិត្យ `$_FILES['...']['error']` និង Upload Limit នៅ Server។
- ទទួលយក File ទំហំធំដោយមិនកំណត់ Quota ឬ Rate Limit។

## ៨. លំហាត់អនុវត្ត

1. កែ Form ឱ្យអាចផ្ញើរូបភាព JPEG/PNG។
2. បដិសេធឯកសារដែល Upload Error ឬលើស 2 MiB។
3. ប្រើ Fileinfo និង Allowlist ដើម្បីរក Extension ពី MIME ដែលទទួលស្គាល់។
4. ផ្ទុកដោយឈ្មោះ Random នៅក្រៅ Web Root។
5. រចនា Endpoint Download ដែលពិនិត្យ Owner/Permission មុនបញ្ជូន File។

## ៩. ចម្លើយណែនាំខ្លី

- ត្រូវប្រើ `multipart/form-data` សម្រាប់ Form Upload។
- `accept` នៅ HTML មិនជំនួស Server-side Validation ទេ។
- ប្រើ Allowlist និង Fileinfo; មិនទុកចិត្ត Client filename/MIME។
- បង្កើតឈ្មោះដោយ `bin2hex(random_bytes(16))` និងកំណត់ Extension ពី Allowlist។
- ផ្ទុកក្រៅ Web Root ហើយ Download តាម Handler ដែលពិនិត្យសិទ្ធិ។

## ឯកសារយោង

- [PHP Manual: Handling File Uploads](https://www.php.net/manual/en/features.file-upload.php)
- [PHP Manual: is_uploaded_file()](https://www.php.net/manual/en/function.is-uploaded-file.php)
- [PHP Manual: move_uploaded_file()](https://www.php.net/manual/en/function.move-uploaded-file.php)
- [OWASP File Upload Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
