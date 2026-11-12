# មេរៀនទី ៨៧៖ ដាក់ Laravel Application ទៅ Production

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- រៀបចំ server សម្រាប់ Laravel 13។
- កំណត់ environment variables និងសុវត្ថិភាព debug mode។
- ដំឡើង dependencies និងបង្កើត assets សម្រាប់ production។
- ដាក់ web server ឱ្យចង្អុលទៅ `public/`។
- រត់ migrations និង optimization commands នៅពេល deploy។
- រៀបចំ workers និងពិនិត្យសុខភាព application ក្រោយ deploy។

## ១. តម្រូវការ Server

Laravel 13 ត្រូវការ PHP 8.3 ឬថ្មីជាងនេះ និង PHP extensions ដែល framework ត្រូវការ ដូចជា PDO, OpenSSL, Mbstring, Fileinfo, Ctype, Tokenizer និង XML។ ពិនិត្យ PHP version និង extensions នៅលើ server ពិត មិនមែនតែ local machine ទេ៖

```bash
php -v
php -m
```

Server ត្រូវមាន Composer និង database/network access ដែល application ត្រូវការ។ ប្រសិនបើមាន frontend assets ត្រូវបង្កើត production build តាម Node package manager ដែល project ប្រើ។

## ២. Environment និង Secrets

កំណត់ environment តាម deployment platform ឬ secret manager៖

- `APP_ENV=production`
- `APP_DEBUG=false`
- `APP_URL=https://example.com`
- Database credentials និង mail/storage secrets តាម provider
- Queue/cache drivers តាម infrastructure

កុំ commit `.env` ដែលមាន secrets ពិតទៅ Git។ ត្រូវប្រាកដថា `APP_KEY` ត្រូវបានបង្កើត និងរក្សាសុវត្ថិភាព។ ការប្តូរ key ក្នុង production អាចធ្វើឱ្យ encrypted data ឬ sessions ចាស់មិនអាចប្រើបាន។

`APP_DEBUG` ត្រូវជា `false` នៅ production ដើម្បីកុំឱ្យ error page បង្ហាញ stack trace ឬ configuration details ដល់អ្នកប្រើ។

## ៣. រៀបចំ Code និង Dependencies

ក្នុង release directory ថ្មី ឬ deployment environment រត់៖

```bash
composer install --no-dev --optimize-autoloader
```

បើមាន frontend build៖

```bash
npm ci
npm run build
```

`npm ci` សមស្របសម្រាប់ reproducible install ពេលមាន lockfile។ បើ project ប្រើ Bun ឬ tool ផ្សេង សូមប្រើ command និង lockfile ដែល project កំណត់។

មុនបញ្ចូល release ទៅ production សូមពិនិត្យ configuration, migrations, queued jobs, storage permissions និងការបម្រុងទុក database តាម deployment process របស់អ្នក។

## ៤. Web Server Document Root

Web server ត្រូវបម្រើ Laravel ពី `public/` ហើយបញ្ជូន request ទៅ `public/index.php`។ កុំចង្អុល document root ទៅ project root និងកុំចម្លង `index.php` ទៅ project root ព្រោះវាអាចបង្ហាញ `.env`, Composer files ឬ source code ឱ្យសាធារណៈចូលបាន។

ផ្តល់ write permissions តែ directories ដែល Laravel ត្រូវសរសេរ ដូចជា `storage/` និង `bootstrap/cache/`។ កុំផ្តល់ permission ឱ្យ web user សរសេរគ្រប់ project files ដោយមិនចាំបាច់។

## ៥. Database Migrations

បន្ទាប់ពី backup និងពិនិត្យ target database រួច រត់ migrations ជា production command៖

```bash
php artisan migrate --force
```

`--force` បញ្ជាក់ថាអនុញ្ញាតដំណើរការ command ក្នុង production environment។ កុំប្រើវាលើ environment ខុស។ សម្រាប់ schema changes គួររៀបចំ migration ឱ្យមាន backward compatibility ប្រសិនបើ release មាន server ច្រើន ឬមាន worker កំពុងដំណើរការ។

កុំប្រើ `migrate:fresh` លើ production database ព្រោះវាលុប tables ទាំងអស់។ បើ deploy ទៅ server ច្រើន សូមប្រើ locking strategy សមស្រប ដើម្បីជៀសវាង servers ច្រើនរត់ migrations ស្របគ្នា។

## ៦. Cache Configuration និង Optimization

Laravel ផ្តល់ command `optimize` សម្រាប់ cache configuration, events, routes និង views ដែលពាក់ព័ន្ធ៖

```bash
php artisan optimize
```

អាចប្រើ command ជាក់លាក់ក្នុង deployment pipeline បើចង់កំណត់ជំហានដាច់ពីគ្នា៖

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

បន្ទាប់ពី configuration cache ត្រូវបានបង្កើត កុំហៅ `env()` នៅក្នុង application code ក្រៅពី configuration files។ ប្រើ `config()` ក្នុង app code។

`optimize:clear` លុប optimize-generated cache files ហើយអាច clear default cache store ផងដែរ; ពិនិត្យផលប៉ះពាល់លើ shared cache មុនប្រើ។

## ៧. Worker និង Long-Running Services

Queue workers, Laravel Reverb និង Octane ជា long-running processes។ ក្រោយ deploy code ថ្មី ត្រូវ reload ឬ restart workers តាម process manager/platform ដើម្បីឱ្យវាផ្ទុក code ថ្មី។ បើ worker មិនត្រូវបានគ្រប់គ្រង អ្នកអាចរកឃើញថា jobs នៅតែប្រើ code version ចាស់។

ពិនិត្យ queue backlog និង failed jobs ក្រោយ release។ ត្រូវមាន process monitor ដែលអាចចាប់ផ្តើម worker ឡើងវិញពេលវាឈប់។

## ៨. Post-deployment Checks

ក្រោយ deploy សូមពិនិត្យ៖

- Homepage និងសំខាន់ៗ routes ឆ្លើយតបបាន។
- Database connection និង migrations ស្ថិតក្នុងស្ថានភាពត្រឹមត្រូវ។
- Storage និង public file links ដំណើរការ។
- Mail/queue jobs អាចដំណើរការតាម configuration។
- Logs មិនបង្ហាញ error ថ្មី និងមិនមាន secrets។
- HTTPS, health check និង monitoring មានសកម្មភាព។

រក្សា rollback plan និង database backup តាមតម្រូវការរបស់ application។ Rollback code មិនតែងតែ rollback schema/data ដោយសុវត្ថិភាពទេ។

## លំហាត់

1. សរសេរ checklist សម្រាប់ Laravel 13 release ទៅ production។
2. ពន្យល់ហេតុអ្វី document root ត្រូវជា `public/`។
3. សរសេរជំហានសម្រាប់ secret configuration ដោយមិនដាក់ secret ក្នុង Git។
4. ពន្យល់ភាពខុសគ្នារវាង `migrate --force` និង `migrate:fresh`។
5. សរសេរហេតុអ្វី workers ត្រូវ restart ក្រោយ deploy។
6. រៀបចំ post-deployment checks សម្រាប់ web, database, queue និង logs។

## កំហុសដែលជួបញឹកញាប់

- **បើក `APP_DEBUG` នៅ production**៖ បង្ហាញព័ត៌មានរសើប។
- **បម្រើ project root**៖ អាចបង្ហាញ files ដែលមិនគួរចូលតាម web។
- **ប្តូរ `APP_KEY` ដោយមិនរៀបចំ**៖ អាចធ្វើឱ្យ encrypted data មិនអាចអាន។
- **រត់ destructive migrations លើ production**៖ អាចបាត់បង់ទិន្នន័យ។
- **ភ្លេច restart queue workers**៖ jobs បន្តប្រើ code ចាស់។
- **ហៅ `env()` ពី application code ក្រោយ config cache**៖ តម្លៃអាចជា null។
- **មិនពិនិត្យ file permissions**៖ app មិនអាចសរសេរ logs/cache ឬផ្ទុយទៅវិញសរសេរ files មិនគួរ។
- **មិនមាន backup ឬ rollback plan**៖ ការកែ release អាចប៉ះពាល់ដល់ទិន្នន័យ និងសេវាកម្ម។

## សំណួរត្រួតពិនិត្យ

1. Laravel 13 ត្រូវការ PHP version អប្បបរមាប៉ុន្មាន?
2. ហេតុអ្វី document root គួរចង្អុលទៅ `public/`?
3. តើ `APP_DEBUG` គួរកំណត់ដូចម្ដេចនៅ production?
4. តើ `config:cache` មានន័យអ្វីចំពោះការហៅ `env()` ក្នុង app code?
5. ហេតុអ្វីមិនត្រូវប្រើ `migrate:fresh` លើ production?
6. តើត្រូវធ្វើអ្វីជាមួយ long-running workers ក្រោយ deploy?

## ឯកសារយោង

- [Laravel 13 Deployment](https://laravel.com/docs/13.x/deployment)
- [Laravel 13 Configuration](https://laravel.com/docs/13.x/configuration)
- [Laravel 13 Database Migrations](https://laravel.com/docs/13.x/migrations)
