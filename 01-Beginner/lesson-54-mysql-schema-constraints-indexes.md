# មេរៀនទី ៥៤៖ រចនា Database Schema និង Constraints ក្នុង MySQL

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- រៀបចំទិន្នន័យជាតារាង និងកំណត់ Primary/Foreign Keys។
- ប្រើ `NOT NULL`, `UNIQUE`, `CHECK` និង Foreign Key Constraints។
- ជ្រើសរើស Referential Action ដូចជា RESTRICT, CASCADE និង SET NULL។
- បង្កើត Index តាម Query ដែលកម្មវិធីប្រើពិត។
- យល់ពីមូលហេតុដែល Constraint គួរអនុវត្តនៅ Database ផងដែរ។

> ឧទាហរណ៍សម្រាប់ MySQL 8.4 និង InnoDB។ ត្រូវពិនិត្យ Database Version និង Migration Plan របស់ Project មុនយកទៅប្រើ។

## ១. Schema ជាកិច្ចសន្យារបស់ទិន្នន័យ

Database Schema កំណត់ថា៖

- តើមាន Table និង Column អ្វីខ្លះ។
- តើ Column អាចទទេ ឬមានតម្លៃស្ទួនឬអត់។
- តើ Record មួយភ្ជាប់ទៅ Record ផ្សេងយ៉ាងដូចម្តេច។
- តើតម្លៃណាខ្លះអាចទទួលយកបាន។

Validation នៅ PHP ជួយផ្តល់ Error Message ល្អទៅអ្នកប្រើ ប៉ុន្តែ Database Constraint ជួយទប់ Data មិនត្រឹមត្រូវពី Code Path ផ្សេង, Worker, Script ឬ Request ដែលមានប្រកួតប្រជែង។

## ២. Primary Key, Foreign Key និង Unique

ឧទាហរណ៍ User និង Post៖

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(254) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uq_users_email UNIQUE (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE posts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    owner_id BIGINT UNSIGNED NOT NULL,
    slug VARCHAR(160) NOT NULL,
    title VARCHAR(200) NOT NULL,
    body TEXT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uq_posts_slug UNIQUE (slug),
    CONSTRAINT fk_posts_owner
        FOREIGN KEY (owner_id) REFERENCES users(id)
        ON DELETE RESTRICT,
    CONSTRAINT chk_posts_status
        CHECK (status IN ('draft', 'published'))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

- **Primary Key** កំណត់អត្តសញ្ញាណ Row មួយ។
- **Foreign Key** បង្ខំឱ្យ Reference មាន Parent Record ត្រឹមត្រូវ។
- **Unique** មិនអនុញ្ញាតតម្លៃស្ទួនក្នុង Column/Column Group។
- **NOT NULL** បង្ខំឱ្យមានតម្លៃ។
- **CHECK** កំណត់ Rule លើតម្លៃក្នុង Row។

Validation នៅ Application នៅតែចាំបាច់ ដើម្បីឆ្លើយតប Error ឱ្យល្អ; Constraint ជាខ្សែការពារបន្ថែម។ ការប្រើ `ON DELETE RESTRICT` មានន័យថា User ដែលនៅមាន Post មិនអាចលុបបានរហូតដល់ការពាក់ព័ន្ធត្រូវបានដោះស្រាយ។

## ៣. Referential Actions

ជ្រើសរើស Delete/Update Action តាមអត្ថន័យទិន្នន័យ៖

- **RESTRICT/NO ACTION**៖ បដិសេធការលុប Parent បើនៅមាន Child Record។
- **CASCADE**៖ លុប/កែ Child តាម Parent; ប្រើតែពេល Child មិនមានអត្ថន័យឯករាជ្យ។
- **SET NULL**៖ ប្តូរ Foreign Key ទៅ NULL; Column ត្រូវអនុញ្ញាត NULL។

កុំដាក់ `ON DELETE CASCADE` ដោយសារតែវាធ្វើឱ្យ Delete ងាយ។ វាអាចលុបទិន្នន័យច្រើនជាងរំពឹង។ វាយតម្លៃ Retention, Audit និងអាជីវកម្មមុនជ្រើស។

## ៤. Normalize ទិន្នន័យ និងកាត់បន្ថយស្ទួន

Normalization ជួយបំបែក Entity និងកាត់បន្ថយការចម្លងទិន្នន័យដោយមិនចាំបាច់។ ឧទាហរណ៍ កុំរក្សា User Email ដដែលក្នុងគ្រប់ Post ប្រសិនបើអាចយោងទៅ `users.id` បាន។

ទោះយ៉ាងណា ការរចនាត្រូវគិតពី Query/Business Requirement ពិត។ Denormalization អាចប្រើសម្រាប់ Performance ប៉ុន្តែត្រូវមានច្បាប់ Update និង Consistency ច្បាស់។ កុំ Duplicate Data ដោយចៃដន្យហើយគ្មានផែនការធ្វើឱ្យតម្លៃស្របគ្នា។

## ៥. Index តាម Query Pattern

Index ជួយស្វែងរក/តម្រៀបទិន្នន័យ ប៉ុន្តែបង្កើន Storage និងចំណាយពេលពេល Insert/Update។ បង្កើត Index តាម Query ដែលប្រើ៖

```sql
CREATE INDEX idx_posts_owner_status_created
    ON posts (owner_id, status, created_at, id);
```

Index Composite មានលំដាប់ Column; វាមានប្រយោជន៍ខុសគ្នាតាម Prefix និង Query Predicate។ ប្រើ `EXPLAIN` ដើម្បីពិនិត្យ Query Plan និងវាស់វែងលើទិន្នន័យតំណាង។ កុំបង្កើត Index លើគ្រប់ Column ដោយស្វ័យប្រវត្តិ។

## ៦. Migration និង Constraint លើទិន្នន័យចាស់

មុនបន្ថែម `NOT NULL`, `UNIQUE` ឬ Foreign Key ទៅ Table ដែលមានទិន្នន័យ៖

1. ពិនិត្យទិន្នន័យចាស់ដែលខ្វះ/ស្ទួន/Reference ខូច។
2. សម្អាត ឬបំពេញ Data ដោយ Migration ដែលអាចត្រួតពិនិត្យបាន។
3. សាកល្បង Migration លើ Staging និង Dataset តំណាង។
4. វាយតម្លៃ Lock/Runtime និង Compatibility ជាមួយ App Version កំពុងដំណើរការ។
5. បន្ថែម Constraint បន្ទាប់ពីទិន្នន័យបានត្រៀមរួច។
6. មាន Recovery/Backup Plan សម្រាប់ការផ្លាស់ប្តូរដែលមានហានិភ័យ។

កុំសន្មតថា Migration អាច Rollback ទិន្នន័យដែលបានលុប ឬបម្លែងបានគ្រប់ករណី។

## ៧. កំហុសដែលជួបញឹកញាប់

- មិនដាក់ Unique Constraint ហើយពឹងតែ PHP Check ដែលអាច Race Condition។
- ប្រើ Foreign Key Action CASCADE ដោយមិនយល់ពីទិន្នន័យដែលនឹងលុប។
- អនុញ្ញាត NULL/Empty String ដោយមិនបញ្ជាក់អត្ថន័យ។
- បង្កើត Index ច្រើនពេក ហើយធ្វើឱ្យ Write យឺត។
- បន្ថែម Constraint ទៅទិន្នន័យចាស់ដែលមាន Invalid Rows។
- គិតថា Validation នៅ Form គ្រប់គ្រាន់សម្រាប់ Database Integrity។

## ៨. លំហាត់

1. រចនា Tables `users`, `posts` និង `comments` ជាមួយ PK/FK។
2. កំណត់ Unique Email និង Slug។
3. ជ្រើសរើស Delete Policy សម្រាប់ Post និង Comment ហើយពន្យល់ហេតុផល។
4. បង្កើត Index សម្រាប់ Query រក Post តាម Owner/Status និងតម្រៀបតាម Date។
5. រៀបចំ Migration Plan ដើម្បីបន្ថែម Unique Constraint លើ Table ដែលមាន Data ចាស់។

## ៩. ចម្លើយណែនាំខ្លី

- ដាក់ Primary Key និង Foreign Key ដើម្បីកំណត់អត្តសញ្ញាណ និងទំនាក់ទំនង។
- ប្រើ Database Unique Constraint សម្រាប់ច្បាប់មិនឱ្យទិន្នន័យស្ទួន។
- ជ្រើស Referential Action តាមអត្ថន័យ/Retention របស់ Data មិនមែនតាមភាពងាយស្រួល។
- បង្កើត Index តាម Query ដែលវាស់បាន និងពិនិត្យដោយ `EXPLAIN`។
- សម្អាតទិន្នន័យចាស់ និងសាកល្បងមុនបន្ថែម Constraint ទៅ Production Schema។

## ឯកសារយោង

- [MySQL 8.4: Foreign Key Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-foreign-keys.html)
- [MySQL 8.4: CHECK Constraints](https://dev.mysql.com/doc/refman/8.4/en/create-table-check-constraints.html)
- [MySQL 8.4: CREATE INDEX](https://dev.mysql.com/doc/refman/8.4/en/create-index.html)
- [មេរៀនទី ១៧៖ Database Relationships, JOIN និង Pagination](lesson-17-php-database-relationships-joins-pagination.md)
- [មេរៀនទី ១៨៖ PDO Transactions និង Data Integrity](lesson-18-php-pdo-transactions.md)
