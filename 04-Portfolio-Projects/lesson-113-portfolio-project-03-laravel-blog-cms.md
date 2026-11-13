# មេរៀនទី ១១៣៖ Portfolio Project ៣ — Laravel Blog CMS

## គោលបំណងគម្រោង

បង្កើត Blog CMS ដោយ Laravel ដែលមានអ្នកនិពន្ធច្រើននាក់។ អ្នកនិពន្ធអាចបង្កើត draft និងកែតែអត្ថបទរបស់ខ្លួន។ Editor អាចពិនិត្យ និង publish អត្ថបទ។ អ្នកអានអាចមើលអត្ថបទដែលបាន publish ស្វែងរក ច្រោះតាម category និងដាក់ comment។

គម្រោងនេះអនុវត្ត Laravel routing, Eloquent relationships, validation, authorization policies, Blade, pagination, database seeders និង feature tests។

## ១. តម្រូវការមុខងារ

- អ្នកប្រើប្រាស់អាចចូលគណនី និងចេញពីគណនី។
- អ្នកនិពន្ធអាចបង្កើត កែ និងលុប draft ផ្ទាល់ខ្លួន។
- Editor អាចពិនិត្យ និង publish/unpublish អត្ថបទ។
- អ្នកអានមើលបានតែអត្ថបទដែលមានស្ថានភាព published។
- អ្នកអានស្វែងរកតាម title និងច្រោះតាម category។
- អ្នកប្រើដែលបាន login អាចដាក់ comment លើអត្ថបទដែលបាន publish។
- អ្នកគ្រប់គ្រងអាចលុប comment ដែលបំពានគោលការណ៍។
- បង្ហាញកំហុស validation ជាភាសាដែលងាយយល់ និងរក្សាទិន្នន័យ input ដែលមិនរសើប។

## ២. រៀបចំ Project និង Authentication

បង្កើត Laravel project តាមឯកសារដំឡើងផ្លូវការ៖

~~~bash
composer create-project laravel/laravel blog-cms
cd blog-cms
php artisan serve
~~~

រៀបចំ database ក្នុង .env ហើយបង្កើត authentication ដោយប្រើ starter kit ដែល Laravel គាំទ្រ ឬប្រព័ន្ធ login ដែលបានរៀនក្នុងមេរៀនមុន។ កុំ commit .env ព្រោះវាអាចមាន password និង secret។ ប្រើ .env.example សម្រាប់បង្ហាញឈ្មោះ configuration ដែលត្រូវការ ដោយមិនដាក់តម្លៃសម្ងាត់ពិត។

## ៣. Database Design

អនុវត្ត schema ដោយ migrations។ គំរូទំនាក់ទំនង៖

- users មាន posts ច្រើន។
- posts មាន comments ច្រើន។
- posts និង categories ជា many-to-many តាម post_category។
- post មាន slug unique សម្រាប់ URL និង status ជា draft ឬ published។

Fields គំរូសម្រាប់ posts៖

~~~text
id
user_id
title
slug
excerpt (nullable)
body
status (draft | published)
published_at (nullable)
created_at
updated_at
~~~

Fields សំខាន់ៗផ្សេងទៀត៖

- categories: id, name, slug
- post_category: post_id, category_id (កំណត់ unique លើគូនេះ)
- comments: id, post_id, user_id, body, approved_at (nullable)

ប្រើ foreign keys និង index លើ slug, status, published_at និង foreign key fields ដែលប្រើញឹកញាប់។ សម្រាប់ការបង្ហាញ public ត្រូវយកតែ published posts ប៉ុណ្ណោះ។

## ៤. Eloquent Relationships

ក្នុង Post model កំណត់ relationships ដូចជា៖

~~~php
public function author(): BelongsTo
{
    return $this->belongsTo(User::class, 'user_id');
}

public function categories(): BelongsToMany
{
    return $this->belongsToMany(Category::class);
}

public function comments(): HasMany
{
    return $this->hasMany(Comment::class);
}
~~~

ក្នុង User model បង្កើត relationship សម្រាប់ posts និង comments។ បន្ថែម type imports តាម model របស់អ្នក។ ពេលបង្ហាញបញ្ជី post ដែលត្រូវបង្ហាញ author និង categories សូមប្រើ eager loading ដូចជា Post::with(['author', 'categories']) ដើម្បីកាត់បន្ថយ query ច្រើនពេក។

## ៥. Routes និង Controllers

បែងចែក route សាធារណៈ និង route ដែលទាមទារ authentication៖

~~~php
Route::get('/posts', [PostController::class, 'index'])->name('posts.index');
Route::get('/posts/{post:slug}', [PostController::class, 'show'])->name('posts.show');

Route::middleware('auth')->group(function () {
    Route::resource('dashboard/posts', DashboardPostController::class)
        ->except(['show']);
});
~~~

ការគ្រប់គ្រង status និង publish action គួរមាន route/action ច្បាស់លាស់ និងត្រូវបានការពារដោយ policy។ កុំឱ្យ public route បង្ហាញ draft ដោយគ្រាន់តែអ្នកណាម្នាក់ស្គាល់ slug។ ប្រើ route model binding តាម slug សម្រាប់ URL អានងាយ ប៉ុន្តែត្រូវ query តែ published post នៅ public controller។

## ៦. Validation និងការរក្សាទិន្នន័យ

ប្រើ Form Request សម្រាប់ create/update។ ច្បាប់គំរូ៖

~~~php
public function rules(): array
{
    return [
        'title' => ['required', 'string', 'max:180'],
        'slug' => ['required', 'string', 'max:200', 'alpha_dash'],
        'excerpt' => ['nullable', 'string', 'max:500'],
        'body' => ['required', 'string'],
        'categories' => ['sometimes', 'array'],
        'categories.*' => ['integer', 'exists:categories,id'],
    ];
}
~~~

បន្ថែម unique validation សម្រាប់ slug ហើយពេល update ត្រូវ ignore record បច្ចុប្បន្នដោយប្រើ Rule::unique(...)->ignore($post) តាម Form Request ដែលបាន authorize។ កុំយក user_id ពី form; កំណត់ author ពី authenticated user នៅ server។

សម្រាប់ comment ត្រូវ validate body និងទំហំអតិបរមា។ បង្ហាញ body ដោយ escaping ក្នុង Blade ដូចជា {{ $comment->body }}។ កុំបង្ហាញ comment ជា raw HTML លុះត្រាតែមានការសម្អាត HTML ដោយវិធីសុវត្ថិភាពជាក់លាក់។

## ៧. Authorization ជាមួយ Policy

បង្កើត policy សម្រាប់ Post។ គោលការណ៍ដែលត្រូវអនុវត្ត៖

- Author អាចកែ ឬលុប post របស់ខ្លួន ប្រសិនបើវាមិនទាន់ publish ឬប្រសិនបើកំណត់ថាអនុញ្ញាត។
- Editor/admin អាចពិនិត្យ និងកែ post របស់អ្នកដទៃ។
- មានតែ Editor/admin ទើបអាច publish ឬ unpublish។
- Public អាចមើលបានតែ published post។

ប្រើ policy នៅ controller ដោយ authorize action មុនធ្វើការផ្លាស់ប្តូរ។ ការលាក់ប៊ូតុងក្នុង UI មិនមែនជាការការពារគ្រប់គ្រាន់ទេ; server ត្រូវពិនិត្យសិទ្ធិគ្រប់ request។ តួនាទី editor/admin ត្រូវមានប្រភពទិន្នន័យ និងការគ្រប់គ្រងច្បាស់លាស់ មិនត្រូវទុកឱ្យ user កែ role តាម request body បានឡើយ។

## ៨. Blade Views និង UX

បង្កើត layout មួយសម្រាប់ navigation, flash messages និង validation errors។ Views អាចរួមមាន៖

~~~text
resources/views/
├── layouts/app.blade.php
├── posts/index.blade.php
├── posts/show.blade.php
└── dashboard/posts/
    ├── index.blade.php
    ├── create.blade.php
    └── edit.blade.php
~~~

ទំព័របញ្ជី public គួរមាន search, category filter និង pagination។ Form គួររក្សាតម្លៃចាស់ដោយប្រើ old() នៅពេល validation បរាជ័យ។ សកម្មភាពលុប/បោះពុម្ពផ្សាយត្រូវប្រើ POST/PATCH/DELETE តាមតម្រូវការ ហើយ form ត្រូវមាន CSRF protection ដែល Laravel ផ្ដល់ឱ្យ។

## ៩. Feature Tests

បង្កើត tests សម្រាប់លំហូរសំខាន់ៗ៖

- Guest មើល published post បាន ប៉ុន្តែមើល draft មិនបាន។
- Author អាចបង្កើត post ហើយ user ផ្សេងមិនអាចកែ draft នោះបាន។
- Editor អាច publish post ហើយ public អាចមើលវាបានបន្ទាប់ពី publish។
- Slug ដែលស្ទួន ឬទិន្នន័យមិនត្រឹមត្រូវ ត្រូវបដិសេធ។
- Comment មិនអាចបង្កើតដោយ guest ប្រសិនបើគោលការណ៍ទាមទារ login។
- Query category/search និង pagination ត្រឡប់លទ្ធផលត្រឹមត្រូវ។
- User មិនអាចក្លែង author ID ដោយបញ្ជូន user_id ក្នុង request បាន។

ប្រើ factories និង database refresh strategy សមស្របសម្រាប់ test environment។ Tests គួរត្រួតពិនិត្យ response, database state និង authorization boundary មិនមែនតែ status code មួយប៉ុណ្ណោះ។

## ១០. Seeders និងទិន្នន័យសាកល្បង

បង្កើត seeder សម្រាប់ categories និង demo posts ដើម្បីឱ្យអ្នកពិនិត្យ project ងាយស្រួល។ បង្កើត account សាកល្បងតែក្នុង local/testing environment ប៉ុណ្ណោះ។ កុំដាក់ password ពិត ឬ sample account ដែលមានសិទ្ធិ admin នៅ production seed ដោយស្វ័យប្រវត្តិ។

## ១១. Definition of Done

- Public pages បង្ហាញតែ published posts។
- Author/editor permissions ត្រូវបានអនុវត្តដោយ policy នៅ server។
- Forms មាន validation និងបង្ហាញ error ដែលអាចយល់បាន។
- Search, category filters និង pagination ដំណើរការ។
- Comments មាន authorization និង escaping ត្រឹមត្រូវ។
- Tests គ្របដណ្ដប់លើ flow សំខាន់ និងការពារការចូលប្រើដោយគ្មានសិទ្ធិ។
- README មាន setup, database migration/seed, test command, demo screenshots និង architecture សង្ខេប។
- Production config ប្រើ HTTPS, secure secrets និង error logging ដោយមិនបង្ហាញព័ត៌មានរសើប។

## លំហាត់ពង្រីក

1. បន្ថែម scheduled publishing ដោយ published_at។
2. បន្ថែម tag និងទំព័រ archive តាមខែ។
3. បន្ថែម moderation queue សម្រាប់ comments។
4. បន្ថែម RSS feed និង Open Graph metadata។
5. បន្ថែម image upload ដោយ validate file type/size និងរក្សាទុកក្នុង private/public disk តាមតម្រូវការ។

## សំណួរត្រួតពិនិត្យ

1. ហេតុអ្វី public query ត្រូវ filter status ជា published?
2. តើ Form Request ជួយបំបែក validation និង controller logic ដូចម្តេច?
3. ហេតុអ្វីការលាក់ប៊ូតុង Edit មិនគ្រប់គ្រាន់សម្រាប់ authorization?
4. តើ eager loading ជួយកាត់បន្ថយបញ្ហា N+1 ដូចម្តេច?
5. តើអ្នកនឹងសរសេរ test ណាដើម្បីបញ្ជាក់ថា author ម្នាក់មិនអាចកែ post របស់អ្នកដទៃ?

## ឯកសារយោង

- [Laravel Installation](https://laravel.com/docs/13.x/installation)
- [Laravel Eloquent Relationships](https://laravel.com/docs/13.x/eloquent-relationships)
- [Laravel Validation](https://laravel.com/docs/13.x/validation)
- [Laravel Authorization](https://laravel.com/docs/13.x/authorization)
- [Laravel Testing](https://laravel.com/docs/13.x/testing)
