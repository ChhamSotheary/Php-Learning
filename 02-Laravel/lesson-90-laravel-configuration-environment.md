# មេរៀនទី ៩០៖ Laravel Configuration និង Environment

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក `.env` និង configuration files។
- ប្រើ `env()` នៅកន្លែងត្រឹមត្រូវ និងអានតម្លៃដោយ `config()`។
- រក្សា secrets ចេញពី source control។
- ស្គាល់តួនាទីរបស់ Laravel Service Providers។
- យល់ពីឥទ្ធិពលរបស់ `config:cache` ក្នុង production។

## ១. Environment និង `.env`

Laravel ប្រើ environment values ដើម្បីកំណត់តម្លៃដែលខុសគ្នាតាម local, testing និង production ដូចជា database host, mail service ឬ cache driver។

Project ថ្មីមាន `.env.example` ដែលបង្ហាញឈ្មោះតម្លៃទូទៅ។ `.env` ផ្ទាល់ខ្លួនអាចមាន secrets ហើយត្រូវរក្សាចេញពី Git។ ចែករំលែកតែ `.env.example` ដែលមិនមាន credentials ពិត។

ឧទាហរណ៍ក្នុង environment៖

```dotenv
APP_NAME="Learning App"
APP_ENV=local
APP_DEBUG=true
PARTNER_API_URL=https://api.example.test
PARTNER_API_TOKEN=replace-with-local-secret
```

កុំបញ្ចូល token ពិតក្នុង example file ឬ commit ទៅ repository។ Production secrets គួរដាក់តាម secret manager ឬ environment configuration របស់ hosting platform។

## ២. បង្កើត Configuration File ផ្ទាល់ខ្លួន

បង្កើត `config/services.php` entry សម្រាប់ partner API៖

```php
return [
    // configuration entries ផ្សេងៗ...

    'partner' => [
        'url' => env('PARTNER_API_URL'),
        'token' => env('PARTNER_API_TOKEN'),
    ],
];
```

ប្រើ `env()` ក្នុង configuration files ដើម្បីអាន environment variables។ នៅក្នុង Controller ឬ Service ផ្សេងៗ ប្រើ `config()`៖

```php
$baseUrl = config('services.partner.url');
$token = config('services.partner.token');
```

កុំហៅ `env()` ដោយផ្ទាល់ក្នុង application code។ ពេល configuration ត្រូវបាន cache `.env` មិនត្រូវបាន load នៅក្នុង request/Artisan ដូចធម្មតាទេ ហើយ `env()` ក្រៅ config files អាចត្រឡប់ null។

## ៣. Configuration Commands

មើល configuration values៖

```bash
php artisan config:show services
```

Cache configuration សម្រាប់ production deployment៖

```bash
php artisan config:cache
```

សម្អាត configuration cache នៅពេល local troubleshooting៖

```bash
php artisan config:clear
```

ជាទូទៅ `config:cache` ជាជំហាន deployment មិនមែន command ដែលត្រូវរត់ជាប្រចាំក្នុង local development ទេ ព្រោះតម្លៃ configuration តែងតែផ្លាស់ប្តូរពេលកំពុងសរសេរកូដ។

## ៤. Service Providers

Service Provider ជាកន្លែង bootstrap services របស់កម្មវិធី។ Provider ជាទូទៅមាន៖

- `register()` — bind services ទៅ service container។
- `boot()` — រៀបចំការងារដែលត្រូវធ្វើក្រោយ providers ផ្សេងបាន register ដូចជា view composers ឬ callbacks។

បង្កើត provider៖

```bash
php artisan make:provider PartnerServiceProvider
```

ឧទាហរណ៍ binding ក្នុង `register()`៖

```php
public function register(): void
{
    $this->app->bind(
        PartnerClient::class,
        fn () => new PartnerClient(
            config('services.partner.url'),
            config('services.partner.token'),
        ),
    );
}
```

ក្នុង `register()` គួរធ្វើតែការចុះឈ្មោះ bindings។ កុំចុះ Route ឬ Event listeners នៅទីនោះ; ប្រើ `boot()` ឬកន្លែងដែល framework ណែនាំ។ Provider របស់ app ត្រូវបានចុះឈ្មោះក្នុង `bootstrap/providers.php`; Artisan generator ជាទូទៅធ្វើការចុះឈ្មោះនេះដោយស្វ័យប្រវត្តិ។

## ៥. Secrets និង Error Hygiene

- កុំ log API token, database password ឬ session secrets។
- កុំបង្ហាញ configuration ទាំងមូលនៅ public endpoint។
- កុំដាក់ secrets ក្នុង screenshots, test fixtures ឬ documentation។
- ប្រើ `config()` នៅ application layer ហើយប្រមូល secrets ពី secure environment។
- ប្រសិនបើ secret លេចធ្លាយ សូម rotate/revoke វាតាម service ដែលចេញ secret នោះ។

## ៦. លំហាត់អនុវត្ត

1. បន្ថែម `PARTNER_API_URL` និង `PARTNER_API_TOKEN` ទៅ local `.env` ដោយមិន commit file នេះ។
2. បន្ថែម configuration entry ក្នុង `config/services.php`។
3. អាន URL និង token ដោយ `config('services.partner...')` ក្នុង Service class។
4. រត់ `php artisan config:show services` និងសង្កេតតម្លៃ local។
5. រត់ `config:cache` ក្នុង local sandbox បន្ទាប់មកពន្យល់ថាហេតុអ្វី `env()` ក្រៅ config files មិនគួរប្រើ។
6. បង្កើត Service Provider ដើម្បី bind PartnerClient ទៅ container។
7. ស្វែងរកថា `.env` ត្រូវបានរារាំងពី Git ដោយរបៀបណា ដោយកុំបង្ហាញ secret ក្នុង output។

## ៧. កំហុសដែលជួបញឹកញាប់

- **ហៅ `env()` ក្នុង Controller/Service**៖ ប្រើ `config()` ជំនួស។
- **Commit `.env`**៖ វាអាចលាតត្រដាង database/API credentials។
- **រត់ `config:cache` ក្នុង local ហើយមិន clear**៖ ការកែ `.env` អាចមិនបង្ហាញតាមការរំពឹង។
- **ដាក់ side effects ក្នុង `register()`**៖ ការចុះ Route/Event គួរធ្វើក្នុង lifecycle stage សមស្រប។
- **បង្ហាញ secret តាម diagnostic endpoint**៖ កុំ return configuration credentials ទៅ client។
- **មិនបែងចែក config តាម environment**៖ local/test/production ត្រូវប្រើ credentials និង services ដាច់ដោយឡែក។

## សំណួរត្រួតពិនិត្យ

1. `.env.example` និង `.env` ខុសគ្នាដូចម្ដេច?
2. តើ `env()` គួរហៅនៅទីណា?
3. `config:cache` មានផលប៉ះពាល់អ្វីចំពោះ `env()`?
4. `register()` និង `boot()` ក្នុង Service Provider ខុសគ្នាដូចម្ដេច?
5. តើត្រូវធ្វើអ្វីបន្ទាប់ពី API token លេចធ្លាយ?

## ឯកសារយោង

- [Laravel 13 Configuration](https://laravel.com/docs/13.x/configuration)
- [Laravel 13 Service Providers](https://laravel.com/docs/13.x/providers)
- [Laravel 13 Deployment](https://laravel.com/docs/13.x/deployment)
