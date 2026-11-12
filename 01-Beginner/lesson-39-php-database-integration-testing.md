# មេរៀនទី ៣៩៖ Integration Testing ជាមួយ PHP និង Database

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Integration Test ពី Unit Test។
- ភ្ជាប់ PHPUnit ទៅ Database សម្រាប់ Test ដោយឡែក។
- រៀបចំ Schema និង Test Data ដែលអាចធ្វើឡើងវិញបាន។
- សម្អាតទិន្នន័យបន្ទាប់ពី Test ដោយ Transaction ឬ Fixture។
- ការពារកុំឱ្យ Test ប៉ះ Production Database។

> Integration Test ប៉ះ Database ពិត ឬ Test Database ហើយអាចមាន Side Effect។ ប្រើ Database ដាច់ដោយឡែកសម្រាប់ Test ជានិច្ច។

## ១. Integration Test ជាអ្វី?

Unit Test ពិនិត្យ Logic តូចមួយដោយមិនចាំបាច់ភ្ជាប់ Database។ Integration Test ពិនិត្យថា Component រួមគ្នាដំណើរការត្រឹមត្រូវ ដូចជា Repository + PDO + Database Schema។ វាជួយចាប់កំហុស SQL, Column, Constraint និង Mapping ដែល Unit Test មិនឃើញ។

Integration Test អាចយឺត និងត្រូវការសេវា Database ដំណើរការ។ ដូច្នេះ ដាក់វាដាច់ពី Unit Test ប្រសិនបើ Project មាន Test ច្រើន។

## ២. រៀបចំ Test Database ដោយសុវត្ថិភាព

បង្កើត Database ឧទាហរណ៍ `learning_php_test` ដែល Credential មិនអាចចូល Production បាន។ អាចកំណត់ Environment Variables ដាច់ដោយឡែក៖

```text
TEST_DB_HOST=127.0.0.1
TEST_DB_NAME=learning_php_test
TEST_DB_USER=learning_php_test_user
TEST_DB_PASSWORD=local_test_only
```

កុំប្រើ Production Secret ឬទិន្នន័យផ្ទាល់ខ្លួនពិតក្នុង Test។ អាចប្រើ Fixture Data សិប្បនិម្មិត។

បង្កើត Connection សម្រាប់ Test៖

```php
<?php
function createTestPdo(): PDO
{
    $host = getenv('TEST_DB_HOST');
    $database = getenv('TEST_DB_NAME');
    $username = getenv('TEST_DB_USER');
    $password = getenv('TEST_DB_PASSWORD');

    foreach ([$host, $database, $username, $password] as $value) {
        if ($value === false || $value === '') {
            throw new RuntimeException('Test database configuration is incomplete.');
        }
    }

    if ($database !== 'learning_php_test') {
        throw new RuntimeException('Refusing to run tests against an unexpected database.');
    }

    $pdo = new PDO(
        "mysql:host={$host};dbname={$database};charset=utf8mb4",
        $username,
        $password,
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
        ]
    );

    return $pdo;
}
```

ឈ្មោះ Database Guard ខាងលើជាការពិនិត្យបន្ថែម មិនមែនការជំនួស Credential ដាច់ដោយឡែក ឬ Permission តូចបំផុតទេ។ Test User គួរមានសិទ្ធិត្រឹមតែ Database សម្រាប់ Test។

## ៣. សាកល្បង Repository

សន្មតថា `UserRepository` មាន Method `findByEmail()`។ Test គួររៀបចំ Schema និង Record ដែលវាត្រូវការ ដោយមិនពឹងទិន្នន័យនៅសល់ពី Test ផ្សេង៖

```php
<?php
use PHPUnit\Framework\TestCase;

final class UserRepositoryTest extends TestCase
{
    private PDO $pdo;
    private UserRepository $users;

    protected function setUp(): void
    {
        $this->pdo = createTestPdo();
        $this->pdo->beginTransaction();

        $this->users = new UserRepository($this->pdo);

        $this->pdo->exec(
            'CREATE TEMPORARY TABLE test_users (
                id INTEGER PRIMARY KEY AUTO_INCREMENT,
                email VARCHAR(255) NOT NULL UNIQUE
            )'
        );
    }

    protected function tearDown(): void
    {
        if ($this->pdo->inTransaction()) {
            $this->pdo->rollBack();
        }
    }

    public function testFindsUserByEmail(): void
    {
        $insert = $this->pdo->prepare(
            'INSERT INTO test_users (email) VALUES (:email)'
        );
        $insert->execute(['email' => 'learner@example.test']);

        $user = $this->users->findByEmail('learner@example.test');

        self::assertSame('learner@example.test', $user['email']);
    }
}
```

ឧទាហរណ៍នេះបង្ហាញគំនិត ប៉ុន្តែ Repository ត្រូវប្រើ Table ដូចគ្នានឹង Test Schema។ MySQL Temporary Table មាន Scope តាម Connection ហើយមិនគ្រប់គ្រង Foreign Key/Schema ទាំងអស់ដូច Production បានទេ។ ក្នុង Project ពិត ប្រើ Migration សម្រាប់ Test Schema និង Fixture ដែលត្រូវនឹង Repository របស់អ្នក។

## ៤. ប្រើ Transaction សម្រាប់ Cleanup

ការចាប់ Transaction នៅ `setUp()` ហើយ Rollback នៅ `tearDown()` ជួយសម្អាតការផ្លាស់ប្តូរ ប្រសិនបើ Test មិនបាន Commit។ ត្រូវប្រាកដថា៖

- Code ដែលកំពុងតេស្តប្រើ Connection ដូចគ្នា ឬការផ្លាស់ប្តូររបស់វាត្រូវបាន Cleanup ផ្សេង។
- Table Engine គាំទ្រ Transaction (ឧ. InnoDB)។
- Code មិនធ្វើ DDL/Operation ដែលបណ្តាលឱ្យ Implicit Commit ក្នុង Database របស់អ្នក។
- Test មិនប្រើ External Service ដែល Transaction អាច Rollback មិនបាន។

បើ Test ច្រើនដំណើរការប៉ារ៉ាឡែល កុំឱ្យវាចែករំលែក Record ឬ Table ដែលមានឈ្មោះដូចគ្នាដោយគ្មាន Isolation។

## ៥. Migrations និង Fixtures

វិធីដែលអាចថែទាំបាន៖

1. ដំណើរការ Migration ដើម្បីបង្កើត Schema Test។
2. បញ្ចូល Fixture Data តិចតួច និងកំណត់ត្រាដែល Test ត្រូវការ។
3. ឱ្យ Test បង្កើត Record ផ្ទាល់ខ្លួន ដើម្បីកុំឱ្យពឹង Order។
4. Cleanup ដោយ Transaction, Truncate ឬ Recreate Database ដោយអាស្រ័យលើ Test Setup។
5. ប្រើ Dummy Data; កុំ Copy Production Database ដោយមិនបានលុបទិន្នន័យផ្ទាល់ខ្លួន។

## ៦. Test Configuration និងដំណើរការ

អាចដាក់ Test ទៅ Suite ផ្សេងក្នុង `phpunit.xml`៖

```xml
<testsuites>
    <testsuite name="Unit">
        <directory>tests/Unit</directory>
    </testsuite>
    <testsuite name="Integration">
        <directory>tests/Integration</directory>
    </testsuite>
</testsuites>
```

ដំណើរការ Suite ជាក់លាក់តាម PHPUnit Version ដែល Project បានដំឡើង៖

```bash
vendor/bin/phpunit --testsuite Integration
```

កំណត់ Test Database Environment Variables មុនដំណើរការ ហើយឱ្យ CI បង្កើត Database ស្អាតសម្រាប់ Job នីមួយៗ។

## ៧. កំហុសដែលជួបញឹកញាប់

- Test ភ្ជាប់ទៅ Database Production ដោយសារប្រើ Environment Variable ខុស។
- Test ពឹងលើ Record ដែល Test មុនបានបង្កើត។
- មិន Cleanup ទិន្នន័យក្រោយ Test។
- គិតថា Rollback Transaction សម្អាត Side Effect ពី Email ឬ HTTP Call បាន។
- ប្រើ SQLite Test តែ Production ប្រើ MySQL ហើយសន្មតថា SQL/Type/Lock Behavior ដូចគ្នាទាំងអស់។
- ដំណើរការ Parallel Test លើ Table/Record ចែករំលែកដោយគ្មាន Isolation។

## ៨. លំហាត់

1. បង្កើត Database សម្រាប់ Test ដាច់ដោយឡែក និង User ដែលមានសិទ្ធិមានកម្រិត។
2. សរសេរ Integration Test សម្រាប់ Repository ដែលស្វែងរក Record តាម ID។
3. បញ្ចូល Test Record នៅក្នុង Test ហើយ Cleanup នៅ `tearDown()`។
4. បន្ថែម Migration/Fixture ដើម្បីរៀបចំ Schema ស្អាត។
5. ពិនិត្យថា CI Test មិនអាចប្រើ Production Database Configuration បាន។

## ៩. ចម្លើយណែនាំខ្លី

- Unit Test ពិនិត្យ Logic តូចៗ; Integration Test ពិនិត្យការសហការរវាង Component ដូចជា PDO និង Database។
- ប្រើ Test Database/credential ដាច់ដោយឡែក និងទិន្នន័យសិប្បនិម្មិត។
- បង្កើត Data ក្នុង Test ផ្ទាល់ ហើយ Cleanup ដោយ Rollback/Fixture Strategy។
- Transaction Rollback មិនអាចលុប Side Effect នៅក្រៅ Database បានទេ។
- ប្រើ Database Engine ដូច Production សម្រាប់ Integration Test សំខាន់ៗ។

## ឯកសារយោង

- [PHP: PDO Transactions](https://www.php.net/manual/en/pdo.transactions.php)
- [PHPUnit Manual](https://docs.phpunit.de/)
- [មេរៀនទី ៣៨៖ Unit Testing ដោយ PHPUnit](lesson-38-phpunit-unit-testing.md)
