# មេរៀនទី ១១៦៖ Portfolio Project ៦ — PHP URL Shortener

## គោលបំណង

បង្កើតសេវាកម្ម URL shortener ដោយ PHP, PDO និង MySQL។ អ្នកប្រើអាចបញ្ចូល URL ដើម ហើយទទួលបាន short link។ ពេលមានអ្នកបើក short link កម្មវិធី redirect ទៅ URL ដើម។ អ្នកប្រើដែលបាន login អាចគ្រប់គ្រង link របស់ខ្លួន និងមើលស្ថិតិសង្ខេប។

គម្រោងនេះហាត់ routing, URL validation, random identifiers, database constraints, redirect security, rate limiting និង privacy-conscious analytics។

## ១. មុខងារ និងច្បាប់សុវត្ថិភាព

- អ្នកប្រើអាចបង្កើត short link ពី URL ដែលបាន validate។
- ប្រព័ន្ធបង្កើត code ខ្លីដែលមិនអាចទាយបានងាយ។
- អ្នកប្រើអាចកំណត់ expiration និងបិទ link របស់ខ្លួន។
- Link ដែលផុតកំណត់ ឬបិទរួច មិនត្រូវ redirect។
- អ្នកប្រើអាចមើល click count និងស្ថិតិសរុបរបស់ link ផ្ទាល់ខ្លួន។
- Admin អាចបិទ link ដែលរំលោភគោលការណ៍ប្រើប្រាស់។
- កំណត់ rate limit និងការការពារ spam/abuse។

សេវាកម្មនេះជួយ redirect ទៅ URL ខាងក្រៅ ដូច្នេះត្រូវមានគោលការណ៍ប្រឆាំង phishing និង malware។ កុំចាត់ទុក URL ណាមួយថាសុវត្ថិភាពដោយសារតែវាមានទម្រង់ត្រឹមត្រូវតែប៉ុណ្ណោះ។

## ២. Project Structure

~~~text
url-shortener/
├── public/
│   └── index.php
├── src/
│   ├── Router.php
│   ├── Database.php
│   ├── UrlValidator.php
│   ├── ShortCodeGenerator.php
│   ├── RateLimiter.php
│   └── LinkRepository.php
├── templates/
├── migrations/
├── tests/
├── .env.example
├── .gitignore
└── README.md
~~~

Document root ត្រូវចង្អុលទៅ public/។ Source, migrations, logs និង secrets ត្រូវនៅក្រៅ public directory។ ប្រើ Composer autoload និងបំបែក controller/service/repository ឱ្យមានទំនួលខុសត្រូវច្បាស់។

## ៣. Database Schema

គំរូ tables៖

~~~sql
CREATE TABLE short_links (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    owner_id BIGINT UNSIGNED NULL,
    code VARCHAR(16) NOT NULL UNIQUE,
    original_url VARCHAR(2048) NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    expires_at DATETIME NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (owner_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX idx_link_owner_created (owner_id, created_at),
    INDEX idx_link_status_expiry (status, expires_at)
);

CREATE TABLE link_clicks (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    short_link_id BIGINT UNSIGNED NOT NULL,
    clicked_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (short_link_id) REFERENCES short_links(id) ON DELETE CASCADE,
    INDEX idx_click_link_time (short_link_id, clicked_at)
);
~~~

រក្សា click analytics តាមតម្រូវការអប្បបរមា។ កុំរក្សា IP ពេញ ឬ user-agent លម្អិតដោយគ្មានគោលបំណងច្បាស់ និងការកំណត់ retention។ អាចប្រើ aggregate counters ជំនួស click event rows ប្រសិនបើមិនត្រូវការវិភាគតាមពេល។

## ៤. បង្កើត Short Code

ប្រើ random bytes ជំនួសលេខ ID ដែលអាចទាយបាន៖

~~~php
$code = rtrim(strtr(base64_encode(random_bytes(6)), '+/', '-_'), '=');
~~~

បើ short code ប៉ះ unique constraint ត្រូវបង្កើត code ថ្មី ហើយសាកម្ដងទៀតក្នុងចំនួនកំណត់។ កុំយក code ដែលផ្ញើដោយ user មកប្រើដោយគ្មាន validation និងការត្រួតពិនិត្យ uniqueness។ បដិសេធពាក្យដែលប៉ះទង្គិចនឹង route សំខាន់ៗ ដូចជា login, admin និង api ប្រសិនបើប្រព័ន្ធមាន route ទាំងនេះ។

## ៥. Validate URL ដោយប្រុងប្រយ័ត្ន

PHP filter_var(..., FILTER_VALIDATE_URL) អាចជួយពិនិត្យ syntax ប៉ុន្តែមិនធានាថា URL មានសុវត្ថិភាព ឬ domain មិនមែនជា phishing ទេ។ កំណត់ policy ឱ្យច្បាស់៖

- អនុញ្ញាតតែ scheme http និង https។
- បដិសេធ username/password ដែលបង្កប់ក្នុង URL ប្រសិនបើមិនចាំបាច់។
- កំណត់ប្រវែង URL អតិបរមា។
- អាចបដិសេធ localhost, private IP ranges និង internal hostnames ប្រសិនបើ server មាន crawler/preview ដែល fetch URL។
- បើប្រព័ន្ធមិន fetch URL ដើមទេ វាមិនមាន server-side request តាម URL នោះទេ ប៉ុន្តែអ្នកប្រើនៅតែអាចប្រើ redirect ដើម្បីបោកប្រាស់អ្នកទទួល link។
- បង្ហាញ domain គោលដៅឱ្យអ្នកប្រើឃើញមុនពេលចេញ short link។

បើបន្ថែម preview, screenshot ឬ URL metadata fetcher ត្រូវការពារ SSRF យ៉ាងតឹងរឹង និងគោរព redirect chain, DNS rebinding និង IP validation។

## ៦. Redirect Request Flow

លំហូរ redirect៖

1. ទទួល code ពី route ដូចជា /abc123។
2. Validate code format និង query link ដោយ prepared statement។
3. ពិនិត្យ status និង expires_at។
4. បង្កើន click counter ឬបន្ថែម aggregate event តាម privacy policy។
5. ឆ្លើយ HTTP 302 ឬ 301 ទៅ destination ដែលបានរក្សាទុក។
6. បើ link គ្មាន/ផុតកំណត់/បិទ បង្ហាញ 404 ឬ 410 តាមការរចនា។

សម្រាប់ link ដែលអាចប្ដូរគោលដៅបាន ប្រើ 302/307 ដើម្បីកុំឱ្យ browser និង proxy cache redirect ចាស់យូរ។ 301/308 សមស្របតែសម្រាប់ redirect អចិន្ត្រៃយ៍ដែលមិនមានគោលដៅផ្លាស់ប្តូរ។ កុំទទួល destination URL ពី query string ពេល redirect; អានវាពី database តាម code។

## ៧. User Accounts និង Authorization

បើ link ជា private-to-owner feature៖

- រក្សា owner_id ពី authenticated user context មិនមែនពី form។
- User មើល កែ បិទ ឬលុបតែ link របស់ខ្លួន។
- Admin actions ត្រូវពិនិត្យ role នៅ server។
- ប្រើ password_hash/password_verify សម្រាប់ account credentials។
- ការកែ ឬលុប link ប្រើ POST/PATCH/DELETE និង CSRF token។
- កុំបង្ហាញ secret link management token ក្នុង logs ឬ referrer។

សម្រាប់ការមើល analytics, query ត្រូវកំណត់ owner_id និង link id ជាមួយគ្នា ដើម្បីទប់ IDOR។ ការលាក់ប៊ូតុងមិនជំនួស authorization នៅ server ទេ។

## ៨. Rate Limiting និង Abuse Prevention

កំណត់ rate limits ដាច់ពីគ្នាសម្រាប់ link creation, login និង redirect lookup។ អាចប្រើ database counter ឬ cache ដែលមាន atomic operations និង expiration។ កុំ rely តែលើ IP ព្រោះអ្នកប្រើច្រើនអាចចែករំលែក IP និង IP អាចប្ដូរបាន។

បន្ថែម៖

- Domain reputation/blocklist តាមប្រភពទិន្នន័យដែលបានថែទាំ។
- សមត្ថភាព report abuse និង admin disable។
- Retention policy សម្រាប់ click data។
- Alert លើការបង្កើត link ជាបរិមាណខ្ពស់។
- កុំស្កេន URL ដោយស្វ័យប្រវត្តិពី server ដោយគ្មាន SSRF protections។

## ៩. Tests

សរសេរ tests សម្រាប់៖

- URL មិនត្រឹមត្រូវ និង scheme ដែលមិនអនុញ្ញាត ត្រូវបានបដិសេធ។
- Code collision បង្កើត code ថ្មី និងមិន overwrite link ចាស់។
- Active link redirect ទៅ destination ដែលបានរក្សាទុក។
- Expired/disabled/nonexistent code មិន redirect។
- User មិនអាចកែ ឬមើល analytics របស់ link របស់អ្នកដទៃ។
- CSRF ខុសលើការកែ link ត្រូវបដិសេធ។
- Rate limit រារាំងការបង្កើត spam។
- Code ដែលមាន route reserved ត្រូវបានបដិសេធ។
- Analytics មិនរក្សាទុកព័ត៌មានលម្អិតលើស privacy policy។

ប្រើ test database និង clock ដែលអាចកំណត់បានសម្រាប់សាកល្បង expiry។ Test redirect response ដោយពិនិត្យ status និង Location header។

## ១០. Deployment

- ប្រើ HTTPS និង security headers។
- ដាក់ secrets ក្នុង environment/configuration management។
- បិទ display_errors នៅ production និងកត់ error ដោយមិន log token ឬ URL ដែលមាន sensitive query parameters។
- បង្កើត database backup និង retention schedule។
- កំណត់ monitoring សម្រាប់ rate limit, error rates និង abuse reports។
- សរសេរ acceptable use policy និង privacy notice សម្រាប់ analytics។
- រៀបចំការដោះស្រាយ domain reputation និង takedown request។

## ១១. Definition of Done

- Short codes ចៃដន្យ និងមាន unique constraint។
- Redirect យក destination ពី database និងពិនិត្យ status/expiry មុនឆ្លើយតប។
- URL policy អនុញ្ញាតតែ scheme ដែលបានកំណត់។
- Owner/admin authorization ត្រូវបានពិនិត្យនៅ server។
- Link mutations មាន CSRF protection និង validation។
- Rate limits និង abuse controls មានការកំណត់។
- Analytics មាន retention និង privacy policy ច្បាស់។
- Tests គ្របដណ្ដប់ redirect, expiry, authorization និង URL validation។
- README មាន setup, config, test, deploy និង privacy/abuse notes។

## លំហាត់បន្ថែម

1. បន្ថែម custom alias ដែល validate និងមិនប៉ះ route reserved។
2. បន្ថែម expiration notification មុន link ផុតកំណត់។
3. បង្កើត dashboard ដែលបង្ហាញ click counts តាមថ្ងៃ ដោយ aggregate data។
4. បន្ថែម API endpoint មាន token authentication និង rate limiting។
5. បង្កើត admin workflow សម្រាប់ report, review និង takedown។

## សំណួរត្រួតពិនិត្យ

1. ហេតុអ្វី URL syntax validation មិនអាចធានាថា destination មានសុវត្ថិភាព?
2. តើ random code និង unique constraint ដោះស្រាយបញ្ហាអ្វី?
3. តើពេលណា 302 សមជាង 301?
4. តើការកំណត់ owner_id ក្នុង query ជួយការពារ IDOR ដូចម្តេច?
5. តើ click analytics អាចប្រមូលទិន្នន័យតិចបំផុតដោយរបៀបណា?

## ឯកសារយោង

- [PHP PDO](https://www.php.net/manual/en/book.pdo.php)
- [PHP Random Bytes](https://www.php.net/manual/en/function.random-bytes.php)
- [PHP Filter Functions](https://www.php.net/manual/en/book.filter.php)
- [OWASP Server-Side Request Forgery Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [OWASP Session Management](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
