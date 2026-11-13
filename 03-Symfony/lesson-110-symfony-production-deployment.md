# មេរៀនទី ១១០៖ Symfony Production Deployment និង Release Checklist

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- រៀបចំ Symfony app សម្រាប់ production environment។
- ដំឡើង Composer dependencies ដោយគ្មាន development packages។
- កំណត់ production variables និង secrets ដោយសុវត្ថិភាព។
- រៀបចំ cache, migrations, assets និង workers ជាលំដាប់។

## ១. Development មិនស្មើ Production

Production app គួរប្រើ `APP_ENV=prod`, `APP_DEBUG=0`, credentials ដាច់ដោយឡែក និង database ដែលមាន backup។ កុំ deploy local `.env.local`, debug toolbar, test database ឬ test accounts ទៅ production។

មុន deploy សូមរៀបចំ staging environment ដែលស្រដៀង production ដើម្បីសាកល្បង configuration និង migrations។

## ២. ដំណើរការរៀបចំ Release

លំដាប់សាមញ្ញសម្រាប់ release៖

1. ជ្រើស source code version/tag ដែលត្រូវ deploy។
2. ដំឡើង production dependencies ដោយ Composer។
3. កំណត់ production environment variables និង secrets។
4. បង្កើត/បង្ហោះ assets ប្រសិនបើ project ប្រើ asset pipeline។
5. រត់ database migrations តាម rollout plan។
6. clear និង warm Symfony cache។
7. restart workers និងពិនិត្យ health/logs។

ឧទាហរណ៍ Composer install៖

```bash
composer install --no-dev --optimize-autoloader
```

`--no-dev` មិនដំឡើង packages សម្រាប់ development/testing; `--optimize-autoloader` បង្កើត optimized autoloader សម្រាប់ production។

## ៣. Environment Variables និង APP_SECRET

កំណត់ environment variables នៅ hosting platform ឬប្រើ production-specific secret store។ APP_SECRET ត្រូវជាតម្លៃ random តែមួយសម្រាប់ application production ហើយរក្សាឱ្យដដែលរវាង deployments; ការប្ដូរវាអាចធ្វើឱ្យ signed URIs និង remember-me cookies ចាស់មិនត្រឹមត្រូវ។

បង្កើតតម្លៃ random ឧទាហរណ៍៖

```bash
php -r 'echo bin2hex(random_bytes(16));'
```

កុំ commit APP_SECRET, database passwords, API tokens ឬ production decryption key ទៅ repository។ កំណត់តម្លៃទាំងនេះតាម secret manager/environment របស់ host។

## ៤. Symfony Cache និង Assets

Clear/warm cache ក្នុង production mode៖

```bash
APP_ENV=prod APP_DEBUG=0 php bin/console cache:clear
```

PowerShell៖

```powershell
$env:APP_ENV = 'prod'
$env:APP_DEBUG = '0'
php bin/console cache:clear
```

បើ project ប្រើ AssetMapper សូមរត់ការដំឡើង/compile assets តាម pipeline របស់វា។ បើប្រើ Webpack Encore ឬ build tool ផ្សេង សូមអនុវត្ត build command ដែល project កំណត់។ កុំសន្មតថាគ្រប់ Symfony app មាន asset build ដូចគ្នា។

## ៥. Database Migrations និង Rollback

ពិនិត្យ migration file និង backup policy មុនប្ដូរ schema production។ រត់ migration តាម deployment process ដែលបានសាកល្បង៖

```bash
APP_ENV=prod php bin/console doctrine:migrations:migrate --no-interaction
```

Migration អាចប៉ះពាល់ទិន្នន័យ ឬមាន downtime អាស្រ័យលើ database និង schema change។ សម្រាប់ change ធំ ត្រូវរៀបចំ backward-compatible rollout និង rollback/recovery plan មុន deploy។ កុំរត់ command ដែលបង្កើត/លុប production database ដោយមិនពិនិត្យ target។

## ៦. Workers និង Scheduled Jobs

បើប្រើ Messenger, queue worker អាចបន្តប្រើ code/config ចាស់បន្ទាប់ពី deploy។ Restart workers តាម process manager របស់អ្នក។ ពិនិត្យ failed messages, cron jobs, scheduled commands និង external cache (Redis/APCu) ប្រសិនបើ project ប្រើ។

## ៧. Health Check និង Logs

ក្រោយ deploy សូមពិនិត្យ៖

- Homepage និង critical routes ឆ្លើយតបបាន។
- Database និង external services អាចភ្ជាប់បាន។
- Logs មិនមាន exception កើតឡើងជាបន្តបន្ទាប់។
- Worker ដំណើរការ និង queue មិនកកស្ទះ។
- Secrets មិនបង្ហាញក្នុង logs ឬ error pages។
- Debug mode បិទ និង public web root ចង្អុលទៅថត `public/`។

ប្រើ platform monitoring/health checks និង logging ដែលសមស្របនឹង host របស់អ្នក។

## ៨. CI/CD និង Release Safety

CI pipeline អាចរត់ static analysis, tests, Composer validation និង build assets មុនអនុញ្ញាត release។ សូមអានមេរៀន CI/CD មុននេះសម្រាប់ការរត់ checks ដោយស្វ័យប្រវត្តិ។

កំណត់ release artifact ឱ្យច្បាស់ និងរក្សា release មុនសម្រាប់ rollback។ កុំធ្វើ migration, asset build ឬប្ដូរ secrets ដោយមិនដឹងថា process កំពុងដំណើរការលើ environment ណា។

## កំហុសដែលជួបញឹកញាប់

- **APP_DEBUG នៅតែ 1**៖ អាចបង្ហាញ stack trace និងព័ត៌មានលម្អិតដល់អ្នកប្រើ។
- **Composer dependencies ខុស**៖ ប្រើ composer.lock ដែលបាន commit និង install ដោយ Composer។
- **Production secret បាត់**៖ ពិនិត្យថា host ផ្ដល់ env variables/keys មុន app boot។
- **Migration មិនបានរត់**៖ ពិនិត្យ migration status និង logs បន្ទាប់ពី release។
- **Worker នៅប្រើ code ចាស់**៖ restart workers តាម deployment plan។
- **ចង្អុល web server ទៅ project root**៖ web server document root ត្រូវជា public/។
- **បញ្ចេញ release ដោយគ្មាន rollback plan**៖ សាកល្បង staging, backup និង migration strategy ជាមុន។

## លំហាត់

1. បង្កើត release checklist សម្រាប់ Symfony app របស់អ្នក។
2. កំណត់ APP_ENV និង APP_DEBUG ក្នុង local shell ហើយពិនិត្យ config។
3. រៀបចំ staging database ដាច់ពី development/production។
4. សរសេរ rollback plan សម្រាប់ migration ដែលបន្ថែម column ថ្មី។
5. រាយបញ្ជី worker, cron jobs និង assets ដែល release ត្រូវគ្រប់គ្រង។

## សំណួរត្រួតពិនិត្យ

1. តើហេតុអ្វី production ប្រើ APP_DEBUG=0?
2. តើ Composer flags --no-dev និង --optimize-autoloader ធ្វើអ្វី?
3. តើហេតុអ្វីត្រូវរក្សា APP_SECRET ឱ្យដដែលរវាង releases?
4. តើ migrations production ត្រូវត្រៀមអ្វីខ្លះ?
5. តើអ្វីខ្លះត្រូវពិនិត្យក្រោយ deploy?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Deployment](https://symfony.com/doc/7.4/deployment.html)
- [Symfony 7.4 Configuration](https://symfony.com/doc/7.4/configuration.html)
- [Symfony 7.4 Secrets](https://symfony.com/doc/7.4/configuration/secrets.html)
