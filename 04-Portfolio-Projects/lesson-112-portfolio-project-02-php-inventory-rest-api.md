# មេរៀនទី ១១២៖ Portfolio Project ២ — PHP Inventory REST API

## គោលបំណងគម្រោង

បង្កើត REST API ដោយ PHP, PDO និង MySQL សម្រាប់គ្រប់គ្រង inventory។ Client អាចចូលប្រើ API ដោយ bearer token ហើយធ្វើ CRUD លើ products របស់ workspace ខ្លួន។

គម្រោងនេះបង្ហាញជំនាញ៖ HTTP/JSON, routing, authentication, authorization, validation, SQL transactions, pagination, error handling និង OpenAPI documentation។

## ១. API Requirements

API មាន endpoints៖

| Method | Path | តួនាទី |
|---|---|---|
| POST | /api/v1/register | បង្កើត account |
| POST | /api/v1/login | ចេញ token |
| POST | /api/v1/logout | បិទ token បច្ចុប្បន្ន |
| GET | /api/v1/products | រាយបញ្ជី products ជាមួយ pagination |
| POST | /api/v1/products | បង្កើត product |
| GET | /api/v1/products/{id} | មើល product |
| PATCH | /api/v1/products/{id} | កែ fields ដែលអនុញ្ញាត |
| DELETE | /api/v1/products/{id} | លុប product |

ប្រើ HTTP status codes ឱ្យសមស្រប៖ 200/201 សម្រាប់ជោគជ័យ, 204 សម្រាប់លុប, 400 សម្រាប់ malformed request, 401 សម្រាប់មិន authenticated, 403 សម្រាប់គ្មានសិទ្ធិ, 404 សម្រាប់មិនមាន resource និង 422 សម្រាប់ validation failure។

## ២. Database Schema

គំរូ tables៖

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(190) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE api_tokens (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    token_hash CHAR(64) NOT NULL UNIQUE,
    expires_at DATETIME NOT NULL,
    revoked_at DATETIME NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE products (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    sku VARCHAR(80) NOT NULL,
    name VARCHAR(160) NOT NULL,
    quantity INT UNSIGNED NOT NULL DEFAULT 0,
    unit_price_cents INT UNSIGNED NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    UNIQUE KEY uniq_user_sku (user_id, sku),
    INDEX idx_products_user_name (user_id, name),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

Unique key លើ `(user_id, sku)` ធ្វើឱ្យ SKU unique ក្នុង workspace របស់ user ម្នាក់ៗ។ តម្លៃប្រាក់រក្សាជាចំនួន cents ដើម្បីកុំឱ្យ floating-point rounding ប៉ះពាល់ការគណនា។

## ៣. រៀបចំ API Structure

```text
inventory-api/
├── public/
│   └── index.php
├── src/
│   ├── Database.php
│   ├── Router.php
│   ├── Request.php
│   ├── Response.php
│   ├── AuthService.php
│   ├── ProductRepository.php
│   └── Validation.php
├── migrations/
├── docs/
│   └── openapi.yaml
├── .env.example
├── .gitignore
└── README.md
```

`public/index.php` ជា front controller។ វាអាន request method/path, route ទៅ handler, បង្កើត response JSON និងបញ្ជូន response ទៅ client។ កុំឱ្យ web server បម្រើ `src/`, migrations ឬ config files ជាសាធារណៈ។

## ៤. Token Authentication

ពេល login ជោគជ័យ បង្កើត token random ដែលមិនអាចទាយបាន ហើយផ្ញើតម្លៃពេញតែម្តងទៅ client៖

```php
$plainToken = bin2hex(random_bytes(32));
$tokenHash = hash('sha256', $plainToken);
```

រក្សាទុកតែ `tokenHash` ជាមួយ user ID និង expiry នៅ database។ រាល់ request បន្ទាប់ client ផ្ញើ៖

```http
Authorization: Bearer <token>
Accept: application/json
```

Server hash bearer token ដែលទទួលបាន រួចរកមើល hash ដែល active និងមិនទាន់ផុតកំណត់។ ពេល logout កំណត់ revoked_at។ កុំ log bearer token និងកុំបង្ហាញ token ពេញក្រោយចេញវាម្តងដំបូង។

## ៥. Authorization និង Tenant Isolation

រាល់ products query ត្រូវកំណត់ owner ពី authenticated user៖

```sql
SELECT id, sku, name, quantity, unit_price_cents
FROM products
WHERE id = :id AND user_id = :user_id
```

កុំទទួល `user_id` ពី JSON body ជាអ្នកកំណត់ owner។ Server ត្រូវយក user ID ពី token authentication context។ បើ resource មិនជារបស់ user បច្ចុប្បន្ន អាចឆ្លើយ 404 ដើម្បីកុំបង្ហាញថា resource នោះមាននៅក្រោមអ្នកដទៃ។

## ៦. JSON និង Validation

ពិនិត្យ Content-Type និង parse JSON ដោយគ្រប់គ្រង error៖

```php
$payload = json_decode(
    file_get_contents('php://input'),
    true,
    512,
    JSON_THROW_ON_ERROR,
);
```

Validate fields តាម allowlist៖

- SKU មានទម្រង់ត្រឹមត្រូវ និងមិនទទេ។
- Name មិនទទេ និងមិនលើស 160 តួអក្សរ។
- Quantity ជាចំនួនគត់មិនអវិជ្ជមាន។
- Unit price ជាចំនួនគត់ cents មិនអវិជ្ជមាន។
- PATCH ទទួលតែ fields ដែល API អនុញ្ញាត។

ប្រសិនបើ JSON មិនត្រឹមត្រូវ ត្រឡប់ 400។ បើទ្រង់ទ្រាយត្រឹមត្រូវតែ fields មិនគោរព rules ត្រឡប់ 422 ជាមួយ error structure ដែលមិនបញ្ចេញ stack trace។

## ៧. Pagination និង Filtering

ប្រើ query parameters ដូចជា `?page=1&per_page=20&search=mouse`។ កំណត់ upper bound លើ `per_page` (ឧ. អតិបរមា 100) ហើយបម្លែង page/limit ទៅ integer មុនប្រើ។ SQL values ត្រូវ bind ដោយ prepared statements; column names និង sort direction ត្រូវជ្រើសពី allowlist ព្រោះ identifiers មិនអាច bind ដូច data values បាន។

Response ឧទាហរណ៍៖

```json
{
  "data": [
    {
      "id": 12,
      "sku": "MOUSE-01",
      "name": "Wireless mouse",
      "quantity": 8,
      "unit_price_cents": 1250
    }
  ],
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 1
  }
}
```

## ៨. Error Response និង Security

ប្រើ response format ស្ថិរភាព ដូចជា៖

```json
{
  "error": {
    "code": "validation_failed",
    "message": "សូមពិនិត្យទិន្នន័យដែលបានបញ្ជូន។",
    "fields": {
      "name": "ឈ្មោះមិនអាចទទេបាន។"
    }
  }
}
```

Response production មិនត្រូវបង្ហាញ SQL query, stack trace, filesystem path ឬ secret។ កត់ exception នៅ server log ដោយកុំបញ្ចូល password/token/body ដែលមានព័ត៌មានរសើប។

## ៩. OpenAPI និង Manual Testing

សរសេរ `docs/openapi.yaml` ដើម្បីកំណត់ paths, request schema, response schema, status codes និង bearer authentication។ ឯកសារនេះអាចប្រើបង្កើត docs UI ឬ client SDK។

សាកល្បង API ដោយ curl ឬ REST client៖

```bash
curl -i http://localhost:8000/api/v1/products
```

សាកល្បងទាំង៖

- Request គ្មាន token → 401។
- Token ផុតកំណត់/revoked → 401។
- User A ទៅ product របស់ User B → គ្មានការបែកធ្លាយទិន្នន័យ។
- JSON ខូច និង fields មិនត្រឹមត្រូវ។
- Duplicate SKU សម្រាប់ user ដដែល និងសម្រាប់ user ផ្សេង។
- Pagination limit ដែលលើសអតិបរមា។

## ១០. Definition of Done

- API endpoints ទាំងអស់មាន authorization តាម user។
- Token ត្រូវបានរក្សាទុកជា hash និងមាន expiry/revocation។
- SQL query ប្រើ prepared statements និង tenant filter។
- Validation និង error responses មានទម្រង់ស្ថិរភាព។
- OpenAPI document គ្រប់ endpoints។
- README បង្ហាញ setup, environment variables, migrations និង request examples។
- Test coverage មាន authentication failure, authorization boundary, validation និង CRUD។

## លំហាត់ពង្រីក

1. បន្ថែម rate limiting ទៅ login endpoint។
2. បន្ថែម audit log សម្រាប់ product create/update/delete។
3. បន្ថែម transaction ពេលកែ quantity ពី stock movement។
4. បង្កើត integration tests ដោយ test database ដាច់ដោយឡែក។
5. រៀបចំ deployment ទៅ staging ដោយ secrets និង HTTPS។

## សំណួរត្រួតពិនិត្យ

1. តើហេតុអ្វីត្រូវរក្សា token ជា hash?
2. តើ tenant isolation ត្រូវអនុវត្តនៅ query ណាខ្លះ?
3. តើ 401 ខុសពី 403 ដូចម្ដេច?
4. តើហេតុអ្វី sort column ត្រូវ allowlist?
5. តើ OpenAPI ជួយអ្នកអភិវឌ្ឍ client យ៉ាងដូចម្ដេច?

## ឯកសារយោង

- [PHP PDO Manual](https://www.php.net/manual/en/book.pdo.php)
- [PHP JSON Functions](https://www.php.net/manual/en/book.json.php)
- [PHP Random Bytes](https://www.php.net/manual/en/function.random-bytes.php)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
