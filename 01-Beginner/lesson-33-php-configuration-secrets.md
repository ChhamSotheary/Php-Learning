# មេរៀនទី ៣៣៖ Configuration និង Secrets ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Configuration ធម្មតា និង Secret ដូចជា Database Password/API Key។
- អានតម្លៃ Environment Variables ក្នុង PHP ដោយពិនិត្យករណីខ្វះតម្លៃ។
- រក្សា Secret ក្រៅ Git Repository និង Source Code។
- ផ្ទៀងផ្ទាត់ Configuration នៅពេលកម្មវិធីចាប់ផ្តើម។
- ជៀសវាងការបង្ហាញ Secret ក្នុង Log, Error Response និង Git History។

> Environment Variables ជាវិធីមួយក្នុងការផ្ញើ Configuration ទៅកម្មវិធី ប៉ុន្តែមិនមែនជា Vault ដោយស្វ័យប្រវត្តិទេ។ ត្រូវកំណត់សិទ្ធិ និងការបញ្ចូល Secret តាមបរិស្ថាន Deployment ផងដែរ។

## ១. Configuration និង Secret ខុសគ្នាយ៉ាងដូចម្តេច?

Configuration ជាតម្លៃប្ដូរតាម Environment ដូចជា Base URL, Debug Mode និង Database Host។ Secret គឺតម្លៃដែលអាចផ្តល់សិទ្ធិចូលប្រើ ឬបញ្ជាក់អត្តសញ្ញាណ ដូចជា Database Password, API Key, Signing Key និង Encryption Key។

ច្បាប់សំខាន់ៗ៖

- កុំសរសេរ Secret ជាប់ក្នុង PHP Source Code។
- កុំ Commit `.env` ដែលមាន Secret ទៅ Git។
- ប្រើ Secret Manager របស់ Platform ឬ Deployment System នៅពេលអាចធ្វើបាន។
- ផ្តល់សិទ្ធិឱ្យតែកម្មវិធី/អ្នកដែលត្រូវការប៉ុណ្ណោះ។
- បំបែក Secret រវាង Development, Staging និង Production។
- មានផែនការប្តូរ/Revocation បើ Secret លេចធ្លាយ។

## ២. អាន Environment Variable ដោយមានការត្រួតពិនិត្យ

`getenv()` អាចត្រឡប់ `false` បើ Variable មិនមាន។ កុំប្រើតម្លៃទទេជំនួសដោយស្ងៀមស្ងាត់សម្រាប់ Secret៖

```php
<?php
function requiredEnv(string $name): string
{
    $value = getenv($name);

    if ($value === false || trim($value) === '') {
        throw new RuntimeException(
            'Required environment variable is missing: ' . $name
        );
    }

    return $value;
}

$dbHost = requiredEnv('DB_HOST');
$dbName = requiredEnv('DB_NAME');
$dbUser = requiredEnv('DB_USER');
$dbPassword = requiredEnv('DB_PASSWORD');
```

Error ខាងលើមានតែឈ្មោះ Configuration ដែលខ្វះ មិនបង្ហាញតម្លៃ Secret ទេ។ ក្នុង Web App ត្រូវចាប់ Configuration Error នៅ Startup/Deployment ហើយកុំបញ្ជូន Stack Trace ទៅ Browser។

## ៣. បង្កើត PDO ពី Configuration

```php
<?php
$dsn = sprintf(
    'mysql:host=%s;dbname=%s;charset=utf8mb4',
    requiredEnv('DB_HOST'),
    requiredEnv('DB_NAME')
);

$pdo = new PDO(
    $dsn,
    requiredEnv('DB_USER'),
    requiredEnv('DB_PASSWORD'),
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ]
);
```

កុំបោះពុម្ព `$dsn`, Username ឬ Password ទៅ Log ពេល Connection បរាជ័យ។ Log សារទូទៅ និង Request ID; បង្ហាញ Error ដែលមានព័ត៌មានលម្អិតតែក្នុង Log ដែលបានការពារ។

## ៤. Environment Variable និង `.env`

Environment Variables អាចកំណត់ដោយ Web Server, Process Manager, Container Orchestrator ឬ Hosting Platform។ របៀបកំណត់ពិតអាស្រ័យលើបរិស្ថាននោះ។

ឯកសារ `.env` មានប្រយោជន៍សម្រាប់ Development ជាមួយ Library ដែលអានវា ប៉ុន្តែ PHP មិនអាន `.env` ដោយខ្លួនឯងទេ។ បើប្រើ Composer Package ដូចជា phpdotenv៖

- `.env` ត្រូវដាក់ក្នុង `.gitignore`។
- បង្កើត `.env.example` មានតែឈ្មោះ Variable និង Placeholder មិនមាន Secret ពិត។
- កុំបង្ហោះ `.env` ទៅ Web Root ឬបម្រើវាជាឯកសារសាធារណៈ។
- សម្រាប់ Production ចូលចិត្ត Secret Manager/Deployment Secret Store ដែលមាន Access Control និង Audit តាមសមត្ថភាពរបស់ Platform។
- កុំចាត់ទុក Container Image `ENV` ឬ `ARG` ដែល Hardcode ជាកន្លែងរក្សា Secret។

ឧទាហរណ៍ `.env.example`៖

```dotenv
APP_ENV=local
APP_DEBUG=false
DB_HOST=127.0.0.1
DB_NAME=learning_php
DB_USER=replace_me
DB_PASSWORD=replace_me
```

## ៥. Parse Boolean និងតម្លៃមានទម្រង់

Environment Variables ជាអត្ថបទ។ កុំសន្មតថា `"false"` បម្លែងទៅជា Boolean false ដោយស្វ័យប្រវត្តិ៖

```php
<?php
$debugRaw = getenv('APP_DEBUG');

$debug = filter_var(
    $debugRaw === false ? 'false' : $debugRaw,
    FILTER_VALIDATE_BOOLEAN
);
```

សម្រាប់ Port ឬចំនួនកំណត់ ត្រូវ Validate ជាចំនួន និង Range ត្រឹមត្រូវ មុនប្រើ។ បើទម្រង់មិនត្រឹមត្រូវ គួរបរាជ័យនៅ Startup ជំនួសឱ្យប្រើ Default ដែលបង្កឱ្យបញ្ហាលាក់កំបាំង។

## ៦. Logs និងការឆ្លើយតប Error

កុំ Log៖

- Password, API Key, Access Token, Session ID ឬ Secret Key។
- Request Header ទាំងអស់ដោយមិន Filter។
- Connection String ដែលមាន Credential។
- ទិន្នន័យផ្ទាល់ខ្លួនដែលមិនចាំបាច់។

បើ Secret លេចធ្លាយ៖

1. បិទ ឬ Revoke Secret ចាស់។
2. បង្កើត Secret ថ្មីតាមប្រព័ន្ធគ្រប់គ្រង Secret។
3. Update Deployment និង Restart/Reload កម្មវិធីតាមតម្រូវការ។
4. ពិនិត្យ Log និងប្រវត្តិប្រើប្រាស់ ដើម្បីវាយតម្លៃផលប៉ះពាល់។
5. លុប Secret ចេញពី Repository History តាមដំណើរការរបស់ស្ថាប័ន ប៉ុន្តែសូមចងចាំថា ការលុប History មិនធ្វើឱ្យ Secret ចាស់មានសុវត្ថិភាពឡើងវិញទេ។

## ៧. កំហុសដែលជួបញឹកញាប់

- Commit `.env` ឬបញ្ចូល API Key ក្នុង Source។
- មានតែ `.gitignore` ប៉ុន្តែ Secret ត្រូវបាន Commit រួចហើយ។
- ដាក់ Production Secret ដូចគ្នាទៅ Development និង Staging។
- Log PDO Exception ទាំងមូលដោយមិនពិនិត្យថាមាន Credential ឬ SQL Parameter ឬអត់។
- ប្រើ Default Password/Signing Key បើ Environment Variable ខ្វះ។
- គិតថា Environment Variable មិនអាចមើលឃើញដោយ Process ផ្សេង ឬ Tools របស់ Platform បានទេ។

## ៨. លំហាត់

1. សរសេរ Function `requiredEnv()` ដែលបរាជ័យបើ Secret ខ្វះ។
2. រៀបចំ `.env.example` ដែលគ្មាន Password ពិត។
3. បន្ថែម `.env` ទៅ `.gitignore` មុនបង្កើតឯកសារមូលដ្ឋាន។
4. កែ Connection Error Log ឱ្យមិនបញ្ចេញ Credential។
5. សរសេរជំហានដែលក្រុមអ្នកនឹងធ្វើ បើ API Key ត្រូវ Commit ទៅ Git ដោយចៃដន្យ។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ `getenv()` ហើយពិនិត្យ `false`/តម្លៃទទេ។
- Secret ត្រូវបានបញ្ចូលនៅ Deployment ដោយ Secret Store ឬ Mechanism ដែលគ្រប់គ្រង Access បាន។
- `.env.example` មាន Placeholder ប៉ុណ្ណោះ; `.env` ក្នុង Local មិនត្រូវ Commit។
- បើ Secret លេចធ្លាយ ត្រូវ Revoke/Rotate ជាបន្ទាន់ ហើយស៊ើបអង្កេតការប្រើប្រាស់។ កុំគ្រាន់តែលុបបន្ទាត់ពី Commit ចុងក្រោយ។
- កុំបង្ហាញ ឬ Log តម្លៃ Secret។

## ឯកសារយោង

- [PHP: getenv()](https://www.php.net/manual/en/function.getenv.php)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
