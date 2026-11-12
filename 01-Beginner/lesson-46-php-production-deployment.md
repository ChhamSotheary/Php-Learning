# មេរៀនទី ៤៦៖ Deploy PHP Application ទៅ Production

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- រៀបចំ PHP App សម្រាប់ Production ដោយ Composer។
- កំណត់ Document Root ឱ្យចង្អុលទៅថត `public/`។
- បំបែក Secret និង Configuration ចេញពី Source Code។
- រៀបចំ Migration, Health Check និង Rollback។
- គិតពី HTTPS, Logs, Permission និងការត្រួតពិនិត្យក្រោយ Deploy។

> ដំណាក់កាល Deploy ខុសគ្នាតាម Hosting/Cloud Provider។ ត្រូវអនុវត្តតាម Documentation របស់ Platform និងប្រើ Environment/Secret Store របស់វា។

## ១. Development និង Production ខុសគ្នា

Production ជាបរិស្ថានពិតដែល User ប្រើ។ មុន Deploy ត្រូវប្រាកដថា៖

- Debug Output និង Error Display មិនបង្ហាញទៅ User។
- Secret មិននៅក្នុង Git ឬ Public Directory។
- HTTPS/TLS និង Domain ត្រូវបានកំណត់ត្រឹមត្រូវ។
- Database User មានសិទ្ធិត្រឹមតែអ្វីដែល App ត្រូវការ។
- File/Cache/Log Directory មាន Permission សមស្រប។
- មានវិធីស្តារកំណែចាស់ និង Backup ទិន្នន័យតាមគោលការណ៍អាជីវកម្ម។

## ២. Document Root គួរជា `public/`

រចនាសម្ព័ន្ធសាមញ្ញ៖

```text
project/
  app/
  config/
  vendor/
  storage/
  public/
    index.php
  .env
  composer.json
  composer.lock
```

Web Server Document Root ត្រូវចង្អុលទៅ `project/public` ប៉ុណ្ណោះ។ កុំបម្រើ `.env`, `composer.json`, Source Code, Log ឬ Private Upload តាម URL ដោយផ្ទាល់។

`public/index.php` ជា Front Controller ដែលទទួល Request ហើយបញ្ជូនទៅ Router/App។ ថត `storage/` ឬ Upload Directory គួរនៅក្រៅ Public Web Root បើមិនចាំបាច់ចូលដោយផ្ទាល់។

## ៣. Composer សម្រាប់ Production

Commit `composer.lock` ដើម្បីកំណត់ Dependency Version ដូចគ្នារវាង Developer/CI/Production។ នៅ Deployment អាចដំឡើងតែ Production Dependency និងបង្កើនល្បឿន Autoloader៖

```bash
composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader
```

ប្រើ `install` ដើម្បីគោរព Lock File; កុំរត់ `composer update` នៅលើ Production ដោយគ្មានការពិនិត្យ និងការកំណត់ Release។ CI គួររត់ Test និង Quality Checks មុនបង្កើត Artifact សម្រាប់ Deploy។

## ៤. Configuration និង Permission

ផ្តល់ Environment Variables/Secrets នៅ Deployment System៖

- Database Host/User/Password
- Application Secret/Signing Key
- SMTP/API Credentials
- Application Environment និង Base URL

កុំដាក់ Production Secret ក្នុង Docker Image, Commit ឬ Deployment Artifact ដែលអាចចែករំលែក។ កំណត់ File Permission តាម Least Privilege; កុំប្រើ `777`។ PHP Process គួរសរសេរបានតែថតដែលត្រូវការ ដូចជា Cache ឬ Private Upload។

## ៥. Database Migration ដោយប្រុងប្រយ័ត្ន

Migration អាចប៉ះពាល់ទិន្នន័យពិត៖

1. ពិនិត្យ Migration លើ Test/Staging ដោយ Schema ស្រដៀង Production។
2. Backup/Recovery Strategy ត្រូវបានសាកល្បងតាមគោលការណ៍របស់ប្រព័ន្ធ។
3. ចៀសវាង Migration ដែល Lock Table យូរ ឬបំបែកកំណែ App ចាស់ដែលកំពុងដំណើរការ។
4. Deploy Schema ដែលអាចឆបគ្នាជាមួយ Release ចាស់ថ្មីក្នុងអំឡុងផ្លាស់ប្តូរ (Expand/Contract)។
5. រត់ Migration ដោយ Account មានសិទ្ធិគ្រប់គ្រាន់ត្រឹមពេលចាំបាច់ ហើយមិនប្រើ Runtime DB User ជា Admin ទេ។

កុំរត់ Migration ដោយស្វ័យប្រវត្តិនៅរាល់ HTTP Request។

## ៦. Release, Health Check និង Rollback

អនុវត្ត Deployment ដែលអាចត្រឡប់វិញបាន៖

- បង្កើត Release Directory/Artifact ដាច់ពីគ្នាសម្រាប់ Version នីមួយៗ។
- ត្រៀម Configuration និង Dependency មុនប្តូរទៅ Release ថ្មី។
- ប្តូរ Traffic/Current Release ដោយវិធី Atomic ដែល Platform គាំទ្រ។
- រត់ Health Check ដែលបញ្ជាក់ថា App និង Dependency សំខាន់ៗអាចឆ្លើយតប។
- ពិនិត្យ Error Rate, Latency, Logs និង Queue ក្រោយ Deploy។
- បើមានបញ្ហា Rollback App ទៅ Release មុន តែត្រូវពិនិត្យភាពឆបគ្នានៃ Database Migration ជាមុន។

Health Check មិនគួរបង្ហាញ Secret, Configuration ទាំងមូល ឬ Stack Trace ទៅសាធារណៈ។ អាចមាន Liveness Check សម្រាប់ដឹងថា Process នៅរស់ និង Readiness Check សម្រាប់ដឹងថាសេវាត្រៀមទទួល Traffic ឬអត់។

## ៧. Cache និង PHP Runtime

- កំណត់ PHP Version/Extensions ដែលត្រូវការឱ្យត្រូវនឹង `composer.json`។
- ប្រើ PHP-FPM ឬ Runtime របស់ Platform តាមការណែនាំរបស់ Hosting។
- កំណត់ OPcache សម្រាប់ Production តាម PHP/Platform Configuration។
- ក្រោយ Deploy Code ថ្មី ត្រូវធានាថា OPcache/Worker ចាស់ត្រូវបាន Reload តាមវិធីរបស់ Platform។
- បើមាន Queue Worker ឬ Scheduler ត្រូវ Deploy/Restart និងពិនិត្យវាដាច់ដោយឡែកពី Web Process។

## ៨. Checklist មុន Deploy

- [ ] Tests និង Checks ក្នុង CI ជោគជ័យ។
- [ ] `composer.lock` មានក្នុង Release។
- [ ] Document Root ចង្អុលទៅ `public/`។
- [ ] Production Secrets មកពី Secret Store/Deployment Environment។
- [ ] `display_errors` បិទ និង Log មានការការពារ។
- [ ] HTTPS, Session Cookie និង Security Headers ត្រឹមត្រូវ។
- [ ] Migration និង Rollback Plan ត្រូវបានពិនិត្យ។
- [ ] Upload/Cache/Log Permission មិនទូលំទូលាយ។
- [ ] Health Check និង Monitoring អាចរកឃើញបញ្ហា។

## ៩. កំហុសដែលជួបញឹកញាប់

- Upload Source Code ទាំងមូលទៅ Public Web Root។
- មិន Commit `composer.lock` ហើយទទួល Dependency Version ខុសគ្នា។
- រត់ `composer update` ឬ Migration ដោយគ្មាន Release/Recovery Plan។
- ដាក់ Secret ក្នុង Repository ឬ Public `.env`។
- ប្រើ Permission `777` ដើម្បីដោះស្រាយ Write Error។
- Deploy ដោយមិនមាន Health Check ឬគ្មានវិធី Rollback។
- ភ្លេច Restart Queue Worker/Reload PHP Runtime ពេល Release ថ្មី។

## ១០. លំហាត់

1. រៀបចំ Project Tree ឱ្យមាន `public/`, `src/`, `storage/` និង `vendor/`។
2. សរសេរ CI/Deploy Steps ដែលរត់ Tests មុន Composer Production Install។
3. បង្កើត Health Check ដែលមិនបង្ហាញ Secret។
4. រៀបចំ Migration Plan ដែលគាំទ្រកំណែ App ចាស់ និងថ្មីក្នុងពេល Deploy។
5. សរសេរ Rollback Checklist និងកំណត់ថាតើ Database Migration អាចត្រឡប់ក្រោយបានឬអត់។

## ឯកសារយោង

- [Composer CLI: install and autoloader options](https://getcomposer.org/doc/03-cli.md)
- [PHP-FPM Installation and Configuration](https://www.php.net/manual/en/install.fpm.php)
- [មេរៀនទី ៣៣៖ Configuration និង Secrets ក្នុង PHP](lesson-33-php-configuration-secrets.md)
- [មេរៀនទី ៤០៖ Error Handling, Logging និង Debugging](lesson-40-php-error-logging-debugging.md)
