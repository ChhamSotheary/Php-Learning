# មេរៀនទី ៧៩៖ Laravel Views និង Blade Templates

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បំបែក HTML presentation ចេញពី Controller។
- បញ្ជូន data ទៅ Blade view។
- ប្រើ Blade syntax សម្រាប់បង្ហាញអត្ថបទ និង loop។
- ប្រើ layout/component ដើម្បីកុំចម្លង HTML ដដែលៗ។
- ការពារ XSS ដោយប្រើ escaped output និង CSRF field។

## ១. View និង Blade ជាអ្វី?

View ជាឯកសារសម្រាប់បង្កើត HTML ដែលអ្នកប្រើឃើញ។ Laravel រក្សា views ក្នុង `resources/views/`។ Blade ជា template engine របស់ Laravel ហើយឯកសាររបស់វាប្រើ suffix `.blade.php`។ Blade compile ទៅជា PHP ហើយ cache លទ្ធផលរហូតដល់ template ផ្លាស់ប្តូរ។

កុំសរសេរ HTML page ទាំងមូលជា string ក្នុង Route ឬ Controller។ បញ្ជូន data ទៅ View ដើម្បីបំបែក presentation logic ចេញពី application logic។

## ២. បង្កើត View និងបញ្ជូន Data

បង្កើត `resources/views/courses/show.blade.php`៖

```blade
<h1>{{ $course->title }}</h1>
<p>កម្រិត៖ {{ $course->level }}</p>
```

ពី Controller អាច return view និងបញ្ជូន data ជា array៖

```php
use App\\Models\\Course;

public function show(string $id)
{
    $course = Course::findOrFail($id);

    return view('courses.show', [
        'course' => $course,
    ]);
}
```

ឈ្មោះ view `courses.show` សំដៅទៅ file `resources/views/courses/show.blade.php`។

## ៣. Blade Syntax

បង្ហាញអត្ថបទដោយ `{{ ... }}`៖

```blade
<p>សួស្តី {{ $name }}</p>
```

បង្ហាញ list ដោយ loop៖

```blade
<ul>
    @forelse ($courses as $course)
        <li>{{ $course->title }}</li>
    @empty
        <li>មិនទាន់មានវគ្គសិក្សាទេ។</li>
    @endforelse
</ul>
```

Conditional៖

```blade
@if ($course->level === 'beginner')
    <span>សម្រាប់អ្នកចាប់ផ្តើម</span>
@else
    <span>កម្រិតផ្សេង</span>
@endif
```

Blade directives ដូចជា `@if`, `@foreach`, `@forelse` ជួយឱ្យ template អានងាយ។ ដាក់តែការសម្រេចចិត្តផ្នែកបង្ហាញសាមញ្ញក្នុង view; logic អាជីវកម្មធ្ងន់គួរនៅក្នុង application layer។

## ៤. Layout ដើម្បីប្រើ HTML រួម

បង្កើត layout `resources/views/layouts/app.blade.php`៖

```blade
<!doctype html>
<html lang="km">
<head>
    <meta charset="utf-8">
    <title>@yield('title', 'កម្មវិធី Laravel')</title>
</head>
<body>
    <main>
        @yield('content')
    </main>
</body>
</html>
```

ទំព័រ `resources/views/courses/index.blade.php` អាចប្រើ layout៖

```blade
@extends('layouts.app')

@section('title', 'វគ្គសិក្សា')

@section('content')
    <h1>វគ្គសិក្សា</h1>
    @foreach ($courses as $course)
        <p>{{ $course->title }}</p>
    @endforeach
@endsection
```

Layout ជួយឱ្យ navigation, metadata និង structure រួមមានកន្លែងតែមួយ។

## ៥. Components សម្រាប់ UI ដែលប្រើឡើងវិញ

Blade component សមស្របសម្រាប់ UI ផ្នែកតូចដែលប្រើម្ដងហើយម្ដងទៀត ដូចជា alert ឬ button។ បង្កើត component៖

```bash
php artisan make:component Alert
```

ប្រើ component ក្នុង view៖

```blade
<x-alert>
    ទិន្នន័យត្រូវបានរក្សាទុក។
</x-alert>
```

Anonymous components អាចដាក់ក្នុង `resources/views/components/`។ Component អាចមាន slot សម្រាប់ content និង props សម្រាប់តម្លៃដែលត្រូវកែតាមកន្លែងប្រើ។

## ៦. សុវត្ថិភាព៖ Escaping និង CSRF

Blade `{{ $value }}` escape HTML ដោយស្វ័យប្រវត្តិ ដើម្បីជួយការពារ Cross-Site Scripting (XSS)។ ប្រើវាសម្រាប់ទិន្នន័យដែលអ្នកប្រើបញ្ចូល។

`{!! $value !!}` បង្ហាញ HTML ដោយមិន escape។ កុំប្រើវាលើ user-generated content លុះត្រាតែបាន sanitize ដោយវិធីដែលសុវត្ថិភាព និងមានហេតុផលច្បាស់លាស់។

Form ដែលប្រើ POST/PUT/PATCH/DELETE តាម web routes ត្រូវមាន CSRF token៖

```blade
<form method="POST" action="{{ route('courses.store') }}">
    @csrf
    <label for="title">ចំណងជើង</label>
    <input id="title" name="title" value="{{ old('title') }}">

    @error('title')
        <p>{{ $message }}</p>
    @enderror

    <button type="submit">រក្សាទុក</button>
</form>
```

`@csrf` បង្កើត hidden token field ដែល Laravel ប្រើផ្ទៀងផ្ទាត់ថា form request មកពី session របស់អ្នកប្រើ។ កុំលុប CSRF protection ដើម្បីជៀសវាង error ដោយមិនដោះស្រាយមូលហេតុ។

## ៧. លំហាត់អនុវត្ត

1. បង្កើត `courses/index.blade.php` សម្រាប់បង្ហាញបញ្ជី Course។
2. បញ្ជូន collection ពី Controller ដោយ `view('courses.index', ['courses' => $courses])`។
3. ប្រើ `@forelse` ដើម្បីបង្ហាញសារ បើគ្មាន Course។
4. បង្កើត layout សម្រាប់ title និង content ហើយឱ្យទំព័រ Course ប្រើវា។
5. បង្កើត form សម្រាប់បង្កើត Course ដែលមាន `@csrf`, `old()` និង `@error`។
6. សាកបង្ហាញ string ដែលមាន `<script>` ដោយ `{{ }}` ហើយពន្យល់ថាហេតុអ្វីមិនគួរប្ដូរទៅ `{!! !!}`។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ឈ្មោះ view មិនត្រូវ path**៖ `courses.show` ត្រូវនឹង `resources/views/courses/show.blade.php`។
- **បង្ហាញ user input ដោយ unescaped syntax**៖ អាចបើកផ្លូវ XSS។
- **ភ្លេច `@csrf` ក្នុង form**៖ Laravel អាចបដិសេធ request ដោយ CSRF error។
- **បញ្ជូន model fields មិនចាំបាច់ទៅ view**៖ ជ្រើសទិន្នន័យដែលត្រូវបង្ហាញប៉ុណ្ណោះ។
- **ដាក់ business logic ច្រើនក្នុង Blade**៖ ផ្លាស់ទី logic ទៅ controller/service ហើយទុក view សម្រាប់ presentation។
- **ភ្លេច `@endsection` ឬ `@endforeach`**៖ ពិនិត្យ directive pairs ពេល template parse error។

## សំណួរត្រួតពិនិត្យ

1. តើ Laravel រក្សា Blade views នៅថតណា?
2. `{{ $value }}` ខុសពី `{!! $value !!}` ដូចម្ដេច?
3. តើ `@csrf` ការពារអ្វី?
4. Layout និង component មានគោលបំណងខុសគ្នាដូចម្ដេច?
5. ហេតុអ្វីគួរបំបែក View ចេញពី Controller?

## ឯកសារយោង

- [Laravel 13 Views](https://laravel.com/docs/13.x/views)
- [Laravel 13 Blade Templates](https://laravel.com/docs/13.x/blade)
- [Laravel 13 CSRF Protection](https://laravel.com/docs/13.x/csrf)
