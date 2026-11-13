# មេរៀនទី ១១៥៖ Portfolio Project ៥ — Secure PHP Storefront និង Order Management

## គោលបំណង

បង្កើតហាងអនឡាញតូចមួយដោយ PHP, PDO និង MySQL។ អ្នកទិញអាចរកមើលផលិតផល បន្ថែមទៅ cart និងដាក់ order។ Admin អាចគ្រប់គ្រងផលិតផល និងមើល order។ គម្រោងនេះប្រមូលផ្តុំមូលដ្ឋាន PHP, database, sessions, security, transactions, testing និង deployment។

កុំដំណើរការការទូទាត់កាតពិតដោយខ្លួនឯង។ សម្រាប់ការទូទាត់ពិត ត្រូវប្រើ payment provider ដែលមានសុវត្ថិភាព ហើយទទួលលទ្ធផលតាម webhook ដែលបានផ្ទៀងផ្ទាត់។

## ១. មុខងារ និងការកំណត់ Scope

**អ្នកទិញ**
- មើល catalog, ស្វែងរក និងច្រោះ category។
- បន្ថែម/កែចំនួនផលិតផលក្នុង cart។
- checkout ដោយបញ្ចូលព័ត៌មានទំនាក់ទំនង និងអាសយដ្ឋានចាំបាច់។
- មើលស្ថានភាព order ដោយប្រើ account ឬ order reference ដែលមិនអាចទាយបាន។

**Admin**
- បង្កើត និងកែផលិតផល តម្លៃ ស្ថានភាព និង stock។
- មើល order និងផ្លាស់ប្តូរស្ថានភាពតាម state transition ដែលអនុញ្ញាត។
- មើល audit history សម្រាប់សកម្មភាពសំខាន់។

មេរៀននេះប្រើការទូទាត់សាកល្បង ឬ cash-on-delivery ជា scope ដំបូង។ កុំរក្សាទុក card number, CVV ឬ payment credentials ក្នុង database របស់កម្មវិធី។

## ២. រចនាសម្ព័ន្ធ Project

~~~text
storefront/
├── public/
│   ├── index.php
│   └── assets/
├── src/
│   ├── Database.php
│   ├── Router.php
│   ├── Auth.php
│   ├── Csrf.php
│   ├── Cart.php
│   ├── OrderService.php
│   └── repositories/
├── templates/
├── migrations/
├── tests/
├── .env.example
├── .gitignore
└── README.md
~~~

កំណត់ document root ទៅ public/។ ប្រើ Composer autoload សម្រាប់ source classes ហើយកុំបង្ហាញ src/, migrations/, .env ឬ log files តាម web server។

## ៣. Database Tables

Schema គំរូ៖

~~~sql
CREATE TABLE products (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sku VARCHAR(80) NOT NULL UNIQUE,
    name VARCHAR(180) NOT NULL,
    description TEXT NULL,
    price_cents INT UNSIGNED NOT NULL,
    stock_quantity INT UNSIGNED NOT NULL DEFAULT 0,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    public_id CHAR(32) NOT NULL UNIQUE,
    customer_email VARCHAR(190) NOT NULL,
    status VARCHAR(30) NOT NULL,
    currency CHAR(3) NOT NULL,
    subtotal_cents BIGINT UNSIGNED NOT NULL,
    total_cents BIGINT UNSIGNED NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_orders_status_created (status, created_at)
);

CREATE TABLE order_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NULL,
    sku_snapshot VARCHAR(80) NOT NULL,
    name_snapshot VARCHAR(180) NOT NULL,
    unit_price_cents INT UNSIGNED NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    line_total_cents BIGINT UNSIGNED NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id) ON DELETE SET NULL,
    INDEX idx_order_items_order (order_id)
);
~~~

រក្សា snapshot នៃឈ្មោះ SKU និងតម្លៃក្នុង order_items ដើម្បីឱ្យ invoice ចាស់នៅត្រឹមត្រូវ ទោះបីផលិតផលបច្ចុប្បន្នត្រូវបានកែ ឬលុបក៏ដោយ។ ប្រើ integer cents សម្រាប់ប្រាក់ ហើយរក្សា currency code ជាក់លាក់។

## ៤. Cart និង Trust Boundaries

Cart ក្នុង session អាចរក្សា product IDs និង quantities ប៉ុណ្ណោះ។ នៅពេលបង្ហាញ cart និង checkout ត្រូវទាញឈ្មោះ តម្លៃ និងស្ថានភាពផលិតផលពី database ម្ដងទៀត។ កុំជឿ price, total ឬ stock ដែល browser ផ្ញើមក។

ក្រោយ login ត្រូវ regenerate session ID។ កំណត់ session cookie ឱ្យប្រើ HttpOnly, Secure នៅ HTTPS និង SameSite តាមតម្រូវការកម្មវិធី។ កំណត់អាយុកាល session និងបិទ session នៅពេល logout។

## ៥. Checkout ជាមួយ Transaction

លំហូរ checkout គួរធ្វើក្នុង transaction៖

1. ផ្ទៀងផ្ទាត់ input និង CSRF token។
2. ចាប់ផ្តើម database transaction។
3. ទាញ product rows ដែលត្រូវការ និង lock stock rows តាមលំដាប់ ID ដើម្បីកាត់បន្ថយ deadlock។
4. ពិនិត្យថាផលិតផលនៅ active និង stock គ្រប់គ្រាន់។
5. គណនាតម្លៃពី database ដោយ integer cents។
6. បង្កើត order និង order item snapshots។
7. បន្ថយ stock ដោយលក្ខខណ្ឌ stock នៅសល់គ្រប់គ្រាន់។
8. commit; បើមានកំហុស ត្រូវ rollback។

គំរូ conditional stock update៖

~~~sql
UPDATE products
SET stock_quantity = stock_quantity - :quantity
WHERE id = :id
  AND is_active = TRUE
  AND stock_quantity >= :quantity
~~~

បន្ទាប់ពី execute ត្រូវពិនិត្យ affected rows។ បើមិនស្មើ 1 មានន័យថា stock មិនគ្រប់ ឬផលិតផលមិនអាចលក់បាន។ កុំទទួល total ពី client។ ត្រូវចងចាំថាការហៅ payment provider មិនគួរធ្វើក្នុង transaction database ដែលបើកយូរ; ប្រើ order state ដែលបានគ្រប់គ្រង និង webhook/idempotency សម្រាប់លំហូរទូទាត់។

## ៦. Order State និង Idempotency

កំណត់ state transitions ច្បាស់ ឧទាហរណ៍៖

~~~text
pending -> confirmed -> shipped -> completed
pending -> cancelled
confirmed -> cancelled (បើអនុញ្ញាត និង stock ត្រូវបានស្ដារវិញ)
~~~

មិនអនុញ្ញាតឱ្យ request កំណត់ status តាមតម្លៃណាមួយដោយសេរីទេ។ សរសេរ service ដែលពិនិត្យ transition និងសិទ្ធិអ្នកប្រើ។

សម្រាប់ការដាក់ order ពីរដងដោយ double-click ឬ retry network សូមទទួល idempotency key ចៃដន្យមួយក្នុង checkout ហើយដាក់ unique constraint លើ key ក្នុង order request/table។ ការស្នើម្តងទៀតដោយ key ដដែលគួរត្រឡប់ order ដដែល មិនបង្កើត order ថ្មី។

## ៧. Security និង Validation

- ប្រើ prepared statements សម្រាប់ SQL values។
- ប្រើ CSRF token លើ cart update, checkout និង admin action។
- Validate product ID, quantity, email និង fields ទាំងអស់នៅ server។
- កំណត់ quantity អប្បបរមា/អតិបរមា និងទំហំ request។
- Escape user-provided text ពេលបង្ហាញ HTML។
- ពិនិត្យ admin permission នៅ server រាល់ endpoint។
- កុំបង្ហាញ order ដោយលេខ ID ដែលអាចទាយបានសម្រាប់ guest; ប្រើ public reference ចៃដន្យ និងកុំដាក់ព័ត៌មានរសើបក្នុង URL។
- កុំ log password, session ID, CSRF token ឬ payment secret។
- កំណត់ security headers និង HTTPS តាម deployment។
- ប្រើ authorization ដើម្បីឱ្យ customer មើល order ផ្ទាល់ខ្លួនតែប៉ុណ្ណោះ។

## ៨. Admin Inventory និង Audit Log

ពេលកែ stock ឬតម្លៃ ត្រូវកត់ត្រាអ្នកធ្វើ សកម្មភាព ពេលវេលា និងតម្លៃមុន/ក្រោយដែលចាំបាច់។ Audit log មិនគួររក្សាទុក secret ឬ card details ទេ។ ផ្ដល់ admin UI សម្រាប់ការកែដោយ CSRF និងការផ្ទៀងផ្ទាត់ role។ កុំអនុញ្ញាត mass assignment ទៅ fields ដូចជា role, price ឬ status ដោយគ្មាន allowlist។

## ៩. Tests

បង្កើត tests សម្រាប់៖

- Cart មិនអាចកែតម្លៃពី browser បាន។
- Checkout ជោគជ័យបង្កើត order snapshots និងបន្ថយ stock។
- Stock មិនគ្រប់ធ្វើឱ្យ checkout បរាជ័យ និងគ្មាន order/stock corruption។
- Double submission ជាមួយ idempotency key មួយបង្កើត order តែមួយ។
- Customer មិនអាចមើល order របស់អ្នកដទៃ។
- Guest មិនអាចចូល admin action។
- CSRF ខុស ឬ input មិនត្រឹមត្រូវត្រូវបានបដិសេធ។
- Order state transition មិនត្រឹមត្រូវត្រូវបានបដិសេធ។
- Product ដែលបានបិទមិនអាចទិញបាន។
- HTML output មិនអនុញ្ញាត XSS។

ប្រើ test database និង seed data សម្រាប់ tests។ បំបែក unit tests សម្រាប់តម្លៃ/state logic និង integration tests សម្រាប់ PDO/database transaction។

## ១០. Deployment និងប្រតិបត្តិការ

README គួរមានតម្រូវការ PHP extensions, Composer install, environment setup, database migration, test command និងការបង្កើត admin ដំបូងដោយសុវត្ថិភាព។ Production ត្រូវបិទ display_errors, បើក logging, ប្រើ HTTPS, កំណត់ permissions លើ storage និង backup database។

ដាក់ migration ក្នុង release process និងមានផែនការស្ដារឡើងវិញ។ កុំដាក់ sample admin password ក្នុង README ឬ repository។ បើបន្ថែម payment provider ត្រូវប្រើ hosted checkout/tokenization របស់ provider និង verify webhook signature តាមឯកសារផ្លូវការ។

## ១១. Definition of Done

- Catalog, cart, checkout និង order management ដំណើរការតាម requirements។
- តម្លៃត្រូវគណនាពី server-side database data។
- Transaction និង constraints ការពារការបាត់បង់ stock និង order ខូច។
- Order snapshots រក្សាប្រវត្តិតម្លៃបាន។
- Idempotency ការពារការបញ្ជូន checkout ស្ទួន។
- Authentication, authorization, CSRF និង validation បានអនុវត្តលើសកម្មភាពផ្លាស់ប្តូរ។
- Automated tests គ្របដណ្ដប់ការទិញ សិទ្ធិ និងករណី stock មិនគ្រប់។
- README ពន្យល់ការដំឡើង និងរបៀបប្រើ project ដោយគ្មាន secrets។
- Production deployment មាន HTTPS, logging, backup និងការគ្រប់គ្រង configuration។

## លំហាត់បន្ថែម

1. ភ្ជាប់ sandbox payment provider ដោយប្រើ hosted checkout។
2. បន្ថែម webhook signature verification និង idempotent event handling។
3. បន្ថែម email receipt តាម queue។
4. បន្ថែម product categories និង image upload ដែលមាន validation។
5. បន្ថែម report លក់ដែលច្រោះតាមថ្ងៃ និង currency។

## សំណួរត្រួតពិនិត្យ

1. ហេតុអ្វី cart មិនគួរជឿ price ដែលរក្សាទុកក្នុង session?
2. តើ order item snapshot ការពារប្រវត្តិទិញយ៉ាងដូចម្តេច?
3. តើ transaction និង conditional stock update ជួយបង្ការការលក់លើស stockយ៉ាងដូចម្តេច?
4. Idempotency key ដោះស្រាយបញ្ហាអ្វី?
5. ហេតុអ្វីមិនគួររក្សាទុក card number និង CVV ក្នុង database ផ្ទាល់ខ្លួន?

## ឯកសារយោង

- [PHP PDO](https://www.php.net/manual/en/book.pdo.php)
- [PHP Sessions](https://www.php.net/manual/en/book.session.php)
- [PHP Password Hashing](https://www.php.net/manual/en/book.password.php)
- [OWASP Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [OWASP Cross-Site Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
