# មេរៀនទី ១០៩៖ Symfony Configuration, Environments និង Secrets

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ស្វែងរក configuration files ក្នុង Symfony project។
- បែងចែក Symfony environment និង environment variables។
- ប្រើ dotenv files សម្រាប់ local/test configuration។
- រក្សា credentials ក្នុង secret store ឬ Symfony Secrets vault។

## ១. ទីតាំង Configuration

Symfony រក្សា config ក្នុងថត config/៖

- config/packages/ មាន configuration សម្រាប់ packages។
- config/routes.yaml និង config/routes/ មាន routing។
- config/services.yaml កំណត់ service discovery និង DI។
- config/packages/dev/, test/, prod/ អាចមាន overrides តាម environment។

Symfony Flex ជាទូទៅបង្កើត config files ពេលដំឡើង package។ ពិនិត្យ options ដោយប្រើ debug:config ឬ config:dump-reference។

## ២. Symfony Environments

Environment ទូទៅ៖

- dev សម្រាប់អភិវឌ្ឍ និង debug។
- test សម្រាប់ automated tests។
- prod សម្រាប់ deployment។

APP_ENV ជ្រើស environment របស់កម្មវិធី។ ក្នុង production ជាទូទៅកំណត់ APP_ENV=prod និង APP_DEBUG=0 តាម server/container environment។

```bash
APP_ENV=prod php bin/console cache:clear
```

នៅ PowerShell៖

```powershell
$env:APP_ENV = 'prod'
php bin/console cache:clear
```

## ៣. Environment Variables និង Dotenv

ប្រើ .env សម្រាប់ default values ដែលអាច share ជាមួយ source code ហើយ .env.local សម្រាប់ local overrides ដែលមិនគួរ commit។ ឧទាហរណ៍ config យោង environment variable៖

```yaml
framework:
    mailer:
        dsn: '%env(MAILER_DSN)%'
```

Symfony អាន dotenv files តាមលំដាប់ និង precedence ដែលបានកំណត់។ System environment variables ដែលកំណត់ដោយ server អាច override values ក្នុង dotenv files។ ពិនិត្យ docs តាម version និង deployment setup របស់អ្នក។

Symfony CLI អាចបង្កើត .env.local.php សម្រាប់ boot លឿន។ ក្រោយកែ dotenv ក្នុង production build សូមបង្កើត env dump ថ្មីតាម deployment process; ឧទាហរណ៍ command ដែល Symfony ផ្ដល់គឺ composer dump-env prod។

## ៤. កុំដាក់ Secrets ក្នុង Git

កុំរក្សា database password, API token, SMTP password ឬ private key ក្នុង source code ឬ dotenv file ដែល commit។ ប្រើ៖

- Local development៖ .env.local ក្នុង gitignore។
- CI/production៖ secret manager ឬ environment variables ដែល platform គ្រប់គ្រង។
- Symfony Secrets vault៖ encrypted values អាចដាក់ក្នុង repository តាម project policy ប៉ុន្តែ production decryption key ត្រូវរក្សាក្រៅ Git។

បើ credential បាន commit រួច សូម rotate វានៅ provider; ការលុប file ក្នុង commit ថ្មីមិនលុបពី Git history ទេ។

## ៥. ប្រើ Symfony Secrets Vault

Symfony Secrets ត្រូវការ Sodium extension។ បង្កើត keys សម្រាប់ environment dev៖

```bash
php bin/console secrets:generate-keys
```

បន្ថែម secret តាម prompt ដែលលាក់ input៖

```bash
php bin/console secrets:set DATABASE_PASSWORD
```

យោងវាក្នុង config ដូច environment variable៖

```yaml
doctrine:
    dbal:
        password: '%env(DATABASE_PASSWORD)%'
```

Public encryption key និង encrypted vault values អាច commit តាម policy របស់ project។ កុំ commit production decryption key។ រក្សាវាក្នុង secret manager/CI environment ហើយបញ្ជូនទៅ server ពេល deploy។

ជ្រើស production vault ក្នុង PowerShell៖

```powershell
$env:APP_RUNTIME_ENV = 'prod'
php bin/console secrets:generate-keys
Remove-Item Env:APP_RUNTIME_ENV
```

បន្ទាប់មកបន្ថែម production secret តាម command និង environment ដែល Symfony docs កំណត់។ secrets:list បង្ហាញឈ្មោះ secret; --reveal បង្ហាញតម្លៃពិត ដូច្នេះប្រើដោយប្រុងប្រយ័ត្ន។

## ៦. Test និង Production Values

Test environment គួរប្រើ test database និង dummy API credentials។ កុំឱ្យ automated tests ទៅកាន់ production services។ Secrets ក្នុង dev/prod vault មិនបំពេញ values ក្នុង test environment ដោយស្វ័យប្រវត្តិទេ; កំណត់ dummy values ក្នុង .env.test ឬ test vault ដាច់ដោយឡែក។

មុនរត់ command ដែលអាច clear cache, ផ្លាស់ប្ដូរ database ឬបង្ហាញ secret សូមពិនិត្យ environment៖

```bash
php bin/console about
php bin/console debug:container --env-vars
php bin/console secrets:list
```

## កំហុសដែលជួបញឹកញាប់

- Commit .env.local ឬ production decryption key។
- កែ dotenv ប៉ុន្តែ production នៅប្រើ env dump ចាស់។
- ច្រឡំ APP_ENV ដែលជ្រើស behavior ជាមួយ secret value ដូចជា password។
- ប្រើ production credentials ក្នុង test suite។
- មិនពិនិត្យថា system environment variable override dotenv value។
- បង្ហាញ secret តាម secrets:list --reveal ដោយគ្មានមូលហេតុ។

## លំហាត់

1. ប្ដូរ APP_ENV រវាង dev និង test ហើយពិនិត្យ config។
2. ដាក់ MAILER_DSN ក្នុង .env.local ហើយធានាថា file មិន commit។
3. កំណត់ test database URL ក្នុង .env.test.local។
4. បង្កើត dummy secret និងយោងតាម %env(...)% ក្នុង config។
5. សរសេរថា production decryption key ត្រូវរក្សាទុកនៅឯណា។

## សំណួរត្រួតពិនិត្យ

1. តើ dev, test និង prod ប្ដូរអ្វីក្នុង Symfony?
2. តើ .env.local ខុសពី .env ដូចម្ដេច?
3. ហេតុអ្វី production decryption key មិនគួរនៅ Git?
4. តើ system environment variables ប៉ះពាល់ dotenv values យ៉ាងដូចម្ដេច?
5. តើហេតុអ្វី test ត្រូវប្រើ credentials និង database ដាច់ដោយឡែក?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Configuring Symfony](https://symfony.com/doc/7.4/configuration.html)
- [Symfony 7.4 Secrets](https://symfony.com/doc/7.4/configuration/secrets.html)
