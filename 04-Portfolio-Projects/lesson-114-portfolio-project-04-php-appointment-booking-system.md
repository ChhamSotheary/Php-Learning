# មេរៀនទី ១១៤៖ Portfolio Project ៤ — PHP Appointment Booking System

## គោលបំណង

បង្កើតកម្មវិធីកក់ពេលណាត់ជួបដោយ PHP, PDO និង MySQL។ អ្នកប្រើអាចមើលពេលទំនេរ និងកក់ពេល។ អ្នកគ្រប់គ្រងអាចបង្កើត availability និងមើលការកក់។ គម្រោងនេះផ្តោតលើ database transactions, validation, authorization, CSRF protection និងការទប់ស្កាត់ការកក់ slot ដដែលពីរដង។

## ១. តួនាទី និងមុខងារ

**អ្នកប្រើប្រាស់**
- ចុះឈ្មោះ ចូល និងចេញពីគណនី។
- មើលថ្ងៃ និងពេលដែលនៅទំនេរ។
- កក់ពេលទំនេរមួយ។
- មើល និងលុបការកក់របស់ខ្លួន តាមលក្ខខណ្ឌកំណត់។

**អ្នកគ្រប់គ្រង**
- បង្កើត ឬបិទពេលទំនេរ។
- មើលការកក់ទាំងអស់ និងកែស្ថានភាព។
- មិនអាចបង្កើត slot ដែលត្រួតគ្នា ឬស្ថិតក្នុងអតីតកាល។

កំណត់ច្បាស់អំពី timezone និងលក្ខខណ្ឌលុបការកក់ ឧទាហរណ៍ អ្នកប្រើអាចលុបមុនពេលណាត់ជួប ២៤ ម៉ោង។

## ២. រៀបចំ Project

~~~text
appointment-booking/
├── public/
│   ├── index.php
│   └── assets/
├── src/
│   ├── Database.php
│   ├── Auth.php
│   ├── Csrf.php
│   ├── BookingRepository.php
│   └── Validation.php
├── templates/
├── migrations/
├── tests/
├── .env.example
├── .gitignore
└── README.md
~~~

Web server document root ត្រូវចង្អុលទៅ public/។ ឯកសារ config, source code, migration និង secret មិនត្រូវអាចទាញយកដោយ browser បានទេ។ រក្សា database credentials នៅ environment variables ហើយមិនត្រូវ commit .env។

## ៣. Database Schema

គំរូតារាង៖

~~~sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(190) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'customer',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE appointment_slots (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    starts_at DATETIME NOT NULL,
    ends_at DATETIME NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'available',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CHECK (ends_at > starts_at),
    INDEX idx_slot_start_status (starts_at, status)
);

CREATE TABLE bookings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    slot_id BIGINT UNSIGNED NOT NULL UNIQUE,
    user_id BIGINT UNSIGNED NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'confirmed',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (slot_id) REFERENCES appointment_slots(id),
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_booking_user (user_id, created_at)
);
~~~

Unique constraint លើ bookings.slot_id ជាការការពារបន្ថែមឱ្យ slot មួយមិនត្រូវបានកក់ដោយ booking ច្រើន។ Constraint នៅ database មានសារៈសំខាន់ ព្រោះការត្រួតពិនិត្យតែក្នុង PHP មិនអាចទប់ race condition បានគ្រប់ករណី។

បើត្រូវរក្សាប្រវត្តិស្ថានភាព ឬអនុញ្ញាតឱ្យលុប/បោះបង់ហើយកក់ឡើងវិញ សូមរចនា status និង uniqueness ឱ្យស្របតាមគោលការណ៍អាជីវកម្ម។ ឧទាហរណ៍ ប្រើ status cancelled និងប្រើ transaction ដើម្បីបញ្ចេញ slot ឡើងវិញដោយសុវត្ថិភាព។

## ៤. Database Connection និង Prepared Statements

ប្រើ PDO exception mode និង native prepared statements ប្រសិនបើ driver គាំទ្រ៖

~~~php
$pdo = new PDO(
    $dsn,
    $dbUser,
    $dbPassword,
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_EMULATE_PREPARES => false,
    ],
);
~~~

Bind តម្លៃក្នុង SQL ដោយ prepared statements។ កុំបញ្ចូល email, user ID ឬ slot ID ដោយ string concatenation។ ទិន្នន័យដែលបង្ហាញក្នុង HTML ត្រូវ escape ដោយ htmlspecialchars ដោយកំណត់ ENT_QUOTES និង UTF-8។

## ៥. លំហូរកក់ និង Transaction

ការត្រួតពិនិត្យថា slot ទំនេរ រួចបញ្ចូល booking ដោយគ្មាន transaction អាចឱ្យអ្នកប្រើពីរនាក់កក់ slot ដូចគ្នាក្នុងពេលប្រហាក់ប្រហែលគ្នា។ ប្រើ transaction និង row lock ក្នុង database ដែលគាំទ្រ៖

~~~php
$pdo->beginTransaction();

try {
    $stmt = $pdo->prepare(
        'SELECT id, starts_at, status
         FROM appointment_slots
         WHERE id = :id
         FOR UPDATE'
    );
    $stmt->execute(['id' => $slotId]);
    $slot = $stmt->fetch(PDO::FETCH_ASSOC);

    if (!$slot || $slot['status'] !== 'available') {
        throw new DomainException('Slot is not available.');
    }

    $insert = $pdo->prepare(
        'INSERT INTO bookings (slot_id, user_id, status)
         VALUES (:slot_id, :user_id, :status)'
    );
    $insert->execute([
        'slot_id' => $slotId,
        'user_id' => $authenticatedUserId,
        'status' => 'confirmed',
    ]);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

ពេលអនុវត្តពិត ត្រូវបកស្រាយ unique constraint violation ជា conflict ដែលអាចយល់បាន ហើយមិនបង្ហាញ SQL ឬ stack trace ទៅអ្នកប្រើ។ Database engine និង isolation behavior អាចខុសគ្នា; សាកល្បងលើ engine ដូច production និងរក្សា unique constraint ជាការការពារចុងក្រោយ។

## ៦. Authentication និង Authorization

ពេលបង្កើត account រក្សា password ដោយ password_hash() ហើយពេល login ផ្ទៀងផ្ទាត់ដោយ password_verify()។ បន្ទាប់ពី login ជោគជ័យ ត្រូវ regenerate session ID។ ពេល logout សម្អាត session និង cookie តាមការកំណត់សុវត្ថិភាព។

- Customer មើល ឬលុបបានតែ booking របស់ខ្លួន។
- Admin អាចគ្រប់គ្រង slots និងការកក់ទាំងអស់។
- កំណត់ user ID ពី session ដែលបាន authenticate; កុំទុកឱ្យ client ជ្រើស user_id ក្នុង form។
- កុំជឿ role ដែលទទួលពី request។ អាន role ពីទិន្នន័យ user ដែលបាន authenticate។
- ពិនិត្យសិទ្ធិនៅ server រាល់ request; ការលាក់ link ក្នុងទំព័រមិនមែនជាការការពារ។

## ៧. Forms, CSRF និង Validation

ការកក់ ការលុប និងការកែស្ថានភាពត្រូវប្រើ POST request និង CSRF token។ កុំធ្វើការផ្លាស់ប្តូរទិន្នន័យដោយ GET។

Validate នៅ server៖

- slot ID ត្រូវជាចំនួនគត់វិជ្ជមាន។
- slot ត្រូវមាននៅក្នុង database និងមាន status available។
- ពេលណាត់ជួបមិនត្រូវកន្លងផុត។
- អ្នកប្រើមិនអាចកក់ slot ដដែលលើសមួយដង។
- ថ្ងៃ និងពេលដែល admin បង្កើត ត្រូវ parse តាម timezone ដែលបានកំណត់ និង end time ត្រូវក្រោយ start time។
- ទិន្នន័យត្រូវបានកាត់ទំហំ និងបដិសេធតម្លៃដែលមិនត្រូវតាមគោលការណ៍។

Validation មិនជំនួស authorization ទេ។ បន្ទាប់ពី validate ID ក៏ត្រូវពិនិត្យថា resource ជារបស់អ្នកប្រើ ឬអ្នកប្រើមានតួនាទីអនុញ្ញាតផងដែរ។

## ៨. ពេលវេលា និង Timezone

រក្សាពេលវេលាដោយគោលការណ៍ timezone តែមួយនៅ database ដែលបានជ្រើសរើស ដូចជា UTC ហើយបម្លែងទៅ timezone របស់អ្នកប្រើនៅពេលបង្ហាញ។ កុំបកស្រាយ datetime-local ជា UTC ដោយស្វ័យប្រវត្តិ; form នោះគ្មាន timezone offset ទេ។ បញ្ជាក់ timezone របស់ admin នៅ config និងធ្វើ conversion ឱ្យច្បាស់។

បង្ហាញ timezone នៅ UI ដើម្បីកុំឱ្យអ្នកប្រើច្រឡំពេលណាត់។ រៀបចំការបម្លែងនៅពេល daylight saving time ប្រសិនបើប្រព័ន្ធប្រើតំបន់ដែលមានការផ្លាស់ប្តូរនេះ។

## ៩. Routes និង Pages

គំរូ routes៖

| Method | Path | អ្នកប្រើ |
|---|---|---|
| GET | /slots | មើលពេលទំនេរ |
| POST | /bookings | Customer កក់ slot |
| GET | /my-bookings | Customer មើលការកក់ខ្លួន |
| POST | /bookings/{id}/cancel | Customer បោះបង់តាមលក្ខខណ្ឌ |
| GET | /admin/slots | Admin គ្រប់គ្រង slots |
| POST | /admin/slots | Admin បង្កើត slot |
| POST | /admin/bookings/{id}/status | Admin កែស្ថានភាព |

ប្រើ front controller ក្នុង public/index.php ដើម្បីបញ្ជូន request ទៅ handler។ បំបែក routing, database access, validation និង HTML templates ឱ្យអាចថែទាំងាយ។ បង្ហាញ response status និង error page ដែលសមស្រប។

## ១០. Tests ដែលគួរមាន

សរសេរ automated tests សម្រាប់ករណី៖

- Guest មិនអាចកក់ slot។
- Customer អាចកក់ slot ទំនេរ។
- Customer ទីពីរមិនអាចកក់ slot ដែលបានកក់រួច។
- Customer មិនអាចមើល ឬលុប booking របស់អ្នកដទៃ។
- Admin អាចបង្កើត slot ត្រឹមត្រូវ តែមិនអាចបង្កើតពេល end មុន start។
- Request គ្មាន CSRF token ត្រូវបានបដិសេធ។
- Slot នៅអតីតកាល ឬមិន available មិនអាចកក់បាន។
- កំហុសក្នុង transaction បង្កឱ្យ rollback ហើយមិនទុក booking ខូច។
- HTML output មិនបកស្រាយ user content ជា script។

សម្រាប់ការប្រកួតប្រជែងនៃការកក់ ត្រូវមាន test ដែលបញ្ជាក់ unique constraint ឬ behavior នៅ transaction។ ដំណើរការលើ test database ដាច់ដោយឡែកពីទិន្នន័យពិត។

## ១១. Logging និង Error Responses

កត់ត្រា exception និង request ID សម្រាប់ជួយ debug។ កុំ log password, session ID, CSRF token ឬព័ត៌មានផ្ទាល់ខ្លួនលើសពីចាំបាច់។ អ្នកប្រើទទួលបានសារខ្លីដែលមិនបង្ហាញ SQL, file path ឬ stack trace។ បំបែក log levels និងបិទការបង្ហាញ error លម្អិតនៅ production។

## ១២. Definition of Done

- User authentication និង role checks ដំណើរការត្រឹមត្រូវ។
- Customer ឃើញតែ bookings ផ្ទាល់ខ្លួន។
- Unique constraint និង transaction ទប់ការកក់ slot ដដែលពីរដង។
- ការផ្លាស់ប្តូរទាំងអស់ប្រើ POST, CSRF protection និង server-side validation។
- Timezone មានការកំណត់ និងបង្ហាញច្បាស់។
- Prepared statements ប្រើសម្រាប់ SQL values ទាំងអស់។
- Automated tests គ្របដណ្ដប់ authorization, validation និងការប្រកួតប្រជែងនៃ booking។
- README មាន installation, database setup, environment variables, test command និងសេចក្ដីណែនាំ deploy។
- Production ប្រើ HTTPS, secret management និង logging ដែលមិនបញ្ចេញទិន្នន័យរសើប។

## លំហាត់ពង្រីក

1. បន្ថែម email confirmation និង reminder ដោយ queue ឬ scheduled task។
2. បន្ថែម waitlist ពេល slot ពេញ។
3. បន្ថែម booking reference ដែលមិនអាចទាយបាន។
4. បន្ថែម audit history សម្រាប់ការផ្លាស់ប្តូរស្ថានភាព។
5. បន្ថែម admin calendar view និង filtering តាមថ្ងៃ/ស្ថានភាព។

## សំណួរត្រួតពិនិត្យ

1. ហេតុអ្វីការត្រួតពិនិត្យថា slot ទំនេរតែមួយដងមិនគ្រប់គ្រាន់?
2. Transaction និង unique constraint ការពារការកក់ស្ទួនតាមរបៀបណា?
3. ហេតុអ្វី user ID និង role ត្រូវយកពី session/server context?
4. តើ CSRF token ការពារបញ្ហាអ្វី?
5. តើគួររក្សាទុក និងបង្ហាញ timezone ដូចម្តេច?

## ឯកសារយោង

- [PHP PDO](https://www.php.net/manual/en/book.pdo.php)
- [PHP Password Hashing](https://www.php.net/manual/en/book.password.php)
- [PHP Sessions](https://www.php.net/manual/en/book.session.php)
- [OWASP Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
