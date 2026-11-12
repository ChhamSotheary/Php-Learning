# មេរៀនទី ៨៦៖ Laravel File Storage និងការផ្ទុក Upload

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពី filesystem disks ក្នុង Laravel។
- Validate uploaded files មុនរក្សាទុក។
- រក្សាទុកឯកសារលើ public ឬ private disk។
- បង្កើត URL សម្រាប់ឯកសារសាធារណៈ។
- ការពារឯកសារឯកជន និងជៀសវាងការទុកចិត្តឈ្មោះ file ពី client។

## ១. Laravel Filesystem និង Disks

Laravel ប្រើ filesystem abstraction ដើម្បីធ្វើការជាមួយ local storage, SFTP និង cloud storage ដូចជា S3 តាម API ស្រដៀងគ្នា។ Configuration ស្ថិតក្នុង `config/filesystems.php` ហើយ disk នីមួយៗកំណត់ driver, root, visibility និងព័ត៌មានពាក់ព័ន្ធ។

- **Private/local disk** សម្រាប់ឯកសារដែលមិនគួរចូលដោយ public URL ផ្ទាល់។
- **Public disk** សម្រាប់ឯកសារដែលមានបំណងឱ្យមនុស្សទូទៅមើលបាន ដូចជា avatar សាធារណៈ។

កុំប្រើ public disk សម្រាប់អត្តសញ្ញាណប័ណ្ណ ឯកសារហិរញ្ញវត្ថុ ឬឯកសារដែលត្រូវការសិទ្ធិពិសេស។

## ២. Validate Upload មុនរក្សាទុក

ក្នុង Controller អាច validate file មុន store៖

```php
use Illuminate\\Http\\Request;

public function updateAvatar(Request $request)
{
    $validated = $request->validate([
        'avatar' => ['required', 'image', 'mimes:jpg,jpeg,png,webp', 'max:2048'],
    ]);

    $path = $request->file('avatar')->store(
        'avatars',
        'public',
    );

    // រក្សា $path ក្នុង field avatar_path របស់ user
    $request->user()->update([
        'avatar_path' => $path,
    ]);

    return response()->json([
        'path' => $path,
    ]);
}
```

Rule `max:2048` សម្រាប់ file គិតជា kilobytes ដូច្នេះឧទាហរណ៍នេះកំណត់ទំហំអតិបរមា 2 MB។ កំណត់ប្រភេទ និងទំហំតាមតម្រូវការកម្មវិធី។ កុំជឿលើ file extension ឬ Content-Type ដែល browser ផ្ញើតែប៉ុណ្ណោះ; server-side file validation នៅតែត្រូវការ។

## ៣. ឈ្មោះឯកសារ និង Path

`store()` បង្កើតឈ្មោះឯកសារថ្មីដោយស្វ័យប្រវត្តិ ហើយត្រឡប់ path ដែលអាចរក្សាទុកក្នុង database បាន។ វាជួយកុំឱ្យយក original filename ពី user មកប្រើផ្ទាល់ក្នុង filesystem path។

រក្សា storage path ក្នុង database ជំនួស absolute server path ឬ URL ដែលបង្កើតដោយដៃ។ វាងាយប្ដូរ disk ពី local ទៅ cloud នៅពេលក្រោយ។

កុំទុកចិត្ត path ដែល client ផ្ញើមកពេលលុប ឬបង្ហាញ file។ ទាញ path ពី record ដែលបាន authorize រួច ដើម្បីជៀសវាង path traversal និងការចូលដំណើរការឯកសាររបស់អ្នកដទៃ។

## ៤. បង្កើត URL សម្រាប់ Public Files

បង្កើត symbolic link សម្រាប់ local public disk ដោយ៖

```bash
php artisan storage:link
```

នេះភ្ជាប់ `public/storage` ទៅ `storage/app/public` តាម configuration លំនាំដើម។ បង្កើត URL ដោយ Storage facade៖

```php
use Illuminate\\Support\\Facades\\Storage;

$url = Storage::disk('public')->url($path);
```

URLs ទាំងនេះអាចត្រូវបានចូលដោយអ្នកណាក៏បាន ដូច្នេះកុំបង្ហាញ private files តាម public disk។

## ៥. ទាញយក Private Files ដោយ Authorization

រក្សាឯកសារឯកជននៅ private disk។ Route ត្រូវ authenticate និង authorize មុនបញ្ជូនឯកសារ៖

```php
use Illuminate\\Support\\Facades\\Storage;

public function download(Invoice $invoice)
{
    $this->authorize('view', $invoice);

    return Storage::disk('local')->download(
        $invoice->pdf_path,
        'invoice.pdf',
    );
}
```

ឧទាហរណ៍នេះសន្មតថា `InvoicePolicy` បានកំណត់សិទ្ធិ `view` ហើយ `pdf_path` មកពី database ដែលទុកចិត្ត។ កុំយក filename ឬ path ពី query string ហើយអនុញ្ញាតឱ្យ download ដោយគ្មានការត្រួតពិនិត្យសិទ្ធិ។

## ៦. លុបឯកសារដោយសុវត្ថិភាព

ពេលជំនួស file ចាស់ អាចលុប path ចាស់ក្រោយពិនិត្យថា record ជារបស់ user និង update បានត្រឹមត្រូវ៖

```php
$oldPath = $user->avatar_path;

$newPath = $request->file('avatar')->store('avatars', 'public');

$user->update(['avatar_path' => $newPath]);

if ($oldPath !== null) {
    Storage::disk('public')->delete($oldPath);
}
```

ក្នុងកម្មវិធីពិត ត្រូវគ្រប់គ្រងបរាជ័យកណ្ដាលប្រតិបត្តិការ និង transaction ឱ្យបានល្អ ដើម្បីកុំឱ្យ database ចង្អុលទៅ file ដែលបាត់ ឬ file ថ្មីគ្មាន record យោង។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត form សម្រាប់ upload avatar និង validate image type/size។
2. រក្សាទុក avatar លើ `public` disk ដោយ `store()`។
3. រក្សា path ក្នុង database ហើយបង្ហាញ URL ដោយ `Storage::url()`។
4. បង្កើត private document download route ដែលប្រើ Policy មុន `download()`។
5. សាកល្បងករណី file មិនមែន image, file ធំពេក និង file path ពី user ផ្សេង។
6. ពន្យល់ថាហេតុអ្វី original filename មិនគួរប្រើផ្ទាល់ជាទីតាំងរក្សាទុក។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ប្រើ public disk សម្រាប់ private documents**៖ អ្នកដទៃអាចចូល URL ដោយគ្មាន authorization។
- **រក្សាទុក original filename ជា path**៖ អាចមានឈ្មោះជាន់គ្នា ឬតម្លៃដែលបង្កគ្រោះថ្នាក់។
- **Validate តែ extension នៅ browser**៖ client អាចកែ request បាន; validate នៅ server។
- **មិនកំណត់ file size**៖ អាចបង្កើន storage និង resource consumption។
- **ទុក path ពី request ទៅ `Storage::delete()` ដោយផ្ទាល់**៖ អាចឱ្យ user ប៉ះពាល់ file របស់អ្នកដទៃ។
- **ភ្លេច `storage:link` សម្រាប់ public disk local**៖ URL អាចមិនអាចចូលបាន។
- **រក្សា absolute path ក្នុង database**៖ ប្ដូរ environment ឬ disk ពេលក្រោយពិបាក។
- **បង្ហាញ original file ជាប្រភេទអាចដំណើរការ**៖ ចាត់ចែង download headers និង content types តាមប្រភេទឯកសារ។

## សំណួរត្រួតពិនិត្យ

1. តើ disk ក្នុង Laravel មានតួនាទីអ្វី?
2. Public disk និង private disk ខុសគ្នាដូចម្ដេច?
3. ហេតុអ្វី `store()` ល្អជាងការជឿឈ្មោះ file ពី client?
4. តើត្រូវធ្វើអ្វីមុន private download?
5. តើ `php artisan storage:link` បង្កើតអ្វី?

## ឯកសារយោង

- [Laravel 13 File Storage](https://laravel.com/docs/13.x/filesystem)
- [Laravel 13 File Validation](https://laravel.com/docs/13.x/validation#validating-files)
- [Laravel 13 HTTP Tests: File Uploads](https://laravel.com/docs/13.x/http-tests#testing-file-uploads)
