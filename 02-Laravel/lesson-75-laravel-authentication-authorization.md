# មេរៀនទី ៧៥៖ Laravel Authentication និង Authorization

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក Authentication និង Authorization។
- ការពារ Route ដោយ middleware `auth`។
- ស្គាល់តួនាទីរបស់ Laravel starter kits និង authentication system។
- បង្កើត Policy ដើម្បីគ្រប់គ្រងសិទ្ធិលើ Model។
- យល់ពីការត្រួតពិនិត្យសិទ្ធិនៅ server និងការឆ្លើយតប 403/404។

## ១. Authentication និង Authorization ខុសគ្នាដូចម្ដេច?

- **Authentication (ផ្ទៀងផ្ទាត់អត្តសញ្ញាណ)** សួរថា៖ “តើអ្នកជានរណា?” ឧទាហរណ៍ ពិនិត្យ login session ឬ API token។
- **Authorization (ពិនិត្យសិទ្ធិ)** សួរថា៖ “តើអ្នកមានសិទ្ធិធ្វើសកម្មភាពនេះទេ?” ឧទាហរណ៍ អនុញ្ញាតឱ្យម្ចាស់អត្ថបទកែអត្ថបទរបស់ខ្លួន។

អ្នកប្រើអាចបាន authenticate រួច ប៉ុន្តែមិនមានសិទ្ធិលើ resource ជាក់លាក់។ កុំយកការចូលប្រើបានទៅស្មើនឹងសិទ្ធិគ្រប់គ្រងទិន្នន័យទាំងអស់។

## ២. រៀបចំ Authentication

Laravel starter kits ផ្តល់ scaffolding សម្រាប់ authentication flow ដូចជា login, registration និង password reset។ សម្រាប់កម្មវិធីថ្មី សូមពិនិត្យ starter kit ដែលសមស្របក្នុងឯកសារ Laravel 13 ហើយអនុវត្ត setup របស់វា។ មេរៀននេះមិនសរសេរ login/password handling ពីដៃទេ ព្រោះការប្រើឧបករណ៍ដែលបានថែទាំកាត់បន្ថយកំហុសសុវត្ថិភាព។

Laravel authentication ប្រើ guards និង providers ដើម្បីសម្រេចថាតើអ្នកប្រើត្រូវ authenticate ដោយរបៀបណា និងទាញ user ពីណា។ Web app ជាទូទៅប្រើ session guard; API token authentication ត្រូវជ្រើសយន្តការដែលសមស្របតាមប្រភេទ client ហើយនឹងរៀនលម្អិតក្នុងមេរៀន API។

កុំរក្សាទុកពាក្យសម្ងាត់ជាអត្ថបទធម្មតា ឬបង្កើត token ផ្ទាល់ខ្លួនដោយគ្មានការគ្រប់គ្រង hashing, expiry និង revocation។ ប្រើ Laravel auth scaffolding និង API package ដែលឯកសារផ្លូវការណែនាំសម្រាប់ use case នោះ។

## ៣. ការពារ Route ដោយ `auth` Middleware

ក្រោយ setup authentication អាចការពារ route ដែលត្រូវការអ្នកប្រើបានចូល៖

```php
use App\\Http\\Controllers\\ProfileController;
use Illuminate\\Support\\Facades\\Route;

Route::get('/profile', [ProfileController::class, 'show'])
    ->middleware('auth');
```

អ្នកអាចការពារ route ជាក្រុម៖

```php
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', DashboardController::class);
    Route::get('/settings', [SettingsController::class, 'index']);
});
```

`auth` ពិនិត្យអត្តសញ្ញាណអ្នកប្រើ។ វាមិនប្រាប់ថាអ្នកប្រើមានសិទ្ធិកែ Model នីមួយៗឬអត់ទេ; សម្រាប់ចំណុចនោះប្រើ Policy ឬ Gate។

## ៤. បង្កើត Policy

Policy រៀបចំសិទ្ធិតាម Model ឬ resource។ បង្កើត PostPolicy៖

```bash
php artisan make:policy PostPolicy --model=Post
```

Laravel អាចរកឃើញ Policy ដោយ convention ប្រសិនបើឈ្មោះ និងទីតាំងតាមលំនាំ។ ឧទាហរណ៍ method `update`៖

```php
<?php

namespace App\\Policies;

use App\\Models\\Post;
use App\\Models\\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

នេះគ្រាន់តែឧទាហរណ៍សាមញ្ញសម្រាប់ ownership។ កម្មវិធីពិតអាចមាន role, team membership ឬលក្ខខណ្ឌផ្សេងទៀត។ ត្រូវអនុវត្ត policy តាមច្បាប់អាជីវកម្ម និងកុំទុកចិត្ត user ID ដែល client ផ្ញើមក។

## ៥. អនុវត្ត Policy ក្នុង Controller

អាចប្រើ `authorize()` ក្នុង Controller ដែលមាន Laravel base Controller trait ឬប្រើ Gate facade។ ឧទាហរណ៍៖

```php
use App\\Models\\Post;
use Illuminate\\Http\\Request;

public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);

    $validated = $request->validate([
        'title' => ['required', 'string', 'max:200'],
        'body' => ['required', 'string'],
    ]);

    $post->update($validated);

    return response()->json($post);
}
```

បើមិនអនុញ្ញាត Laravel បម្លែង authorization exception ទៅ HTTP 403 Forbidden។ ចំពោះ resource ដែលមិនមាន ឬមិនគួរបង្ហាញអត្ថិភាពរបស់វា អាចរចនាការឆ្លើយតប 404 តាមតម្រូវការសុវត្ថិភាពនៃកម្មវិធី។

Policy អាចអនុវត្តតាម middleware ផងដែរ ដូចជា `can:update,post`។ សូមប្រើវិធីដែលធ្វើឱ្យ authorization check នៅកន្លែងសមស្រប និងងាយពិនិត្យ។

## ៦. Gates សម្រាប់សិទ្ធិទូទៅ

Gate សមស្របសម្រាប់សកម្មភាពដែលមិនភ្ជាប់ទៅ Model ជាក់លាក់ ដូចជា dashboard សម្រាប់ administrator។ Gate ជា closure ដែលកំណត់ថា user អាចធ្វើអ្វីមួយឬអត់។ សម្រាប់សិទ្ធិលើ Post, Course ឬ resource ប្រើ Policy ជាទូទៅងាយថែទាំជាង។

ចំណុចសំខាន់៖ កុំយកការលាក់ប៊ូតុងក្នុង UI ជាការការពារសិទ្ធិ។ ត្រូវ authorize នៅ server រាល់ request ដែលអាចអាន ឬកែទិន្នន័យការពារ។

## ៧. លំហាត់អនុវត្ត

1. ការពារ `/dashboard` ដោយ middleware `auth`។
2. បង្កើត `CoursePolicy` ដើម្បីអនុញ្ញាតឱ្យម្ចាស់ Course កែវគ្គសិក្សារបស់ខ្លួន។
3. ហៅ authorization check មុនពេល update Course។
4. សាកល្បងជាមួយអ្នកប្រើដែលមិនបាន login និងអ្នកប្រើដែលបាន login ប៉ុន្តែមិនមែនជាម្ចាស់ Course។
5. កត់ត្រាភាពខុសគ្នារវាង authentication error និង authorization error។

## ៨. កំហុសដែលជួបញឹកញាប់

- **គិតថា login មានន័យថា admin**៖ authenticate មិនស្មើ authorize។
- **ពឹងលើ UI ដើម្បីការពារ**៖ Route និង Controller នៅ server ត្រូវធ្វើ permission checks។
- **ប្រើ user ID ពី request ដើម្បីបញ្ជាម្ចាស់**៖ ប្រើ user ដែល authenticate រួច និងទិន្នន័យ resource ពី server។
- **ភ្លេច authorization លើ object ID ក្នុង URL**៖ ត្រួតពិនិត្យ ownership/permission លើ resource ដែលបាន route-model-bind។
- **សរសេរ login system ឬ password storage ដោយខ្លួនឯង**៖ ប្រើ Laravel authentication features និង starter kit ដែលបានថែទាំ។
- **យក validation ជំនួស Policy**៖ Validation ពិនិត្យ input; Policy ពិនិត្យសិទ្ធិ។

## សំណួរត្រួតពិនិត្យ

1. Authentication ឆ្លើយសំណួរអ្វី ហើយ Authorization ឆ្លើយសំណួរអ្វី?
2. តើ middleware `auth` ការពារអ្វី?
3. ពេលណាគួរប្រើ Policy ជំនួស Gate?
4. តើ Laravel ឆ្លើយតបយ៉ាងដូចម្ដេចពេល Policy បដិសេធ?
5. ហេតុអ្វីលាក់ប៊ូតុងក្នុង UI មិនគ្រប់គ្រាន់សម្រាប់សុវត្ថិភាព?

## ឯកសារយោង

- [Laravel 13 Authentication](https://laravel.com/docs/13.x/authentication)
- [Laravel 13 Authorization](https://laravel.com/docs/13.x/authorization)
- [Laravel 13 Starter Kits](https://laravel.com/docs/13.x/starter-kits)
