# មេរៀនទី ៤២៖ API Versioning និង OpenAPI Documentation

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីមូលហេតុដែល API ត្រូវការ Versioning។
- ជ្រើសរើសវិធីដាក់ Version ក្នុង URL ឬ Header។
- សរសេរ OpenAPI Specification សម្រាប់ Endpoint មួយ។
- កត់ត្រា Request, Response, Error និង Authentication។
- រៀបចំការផ្លាស់ប្តូរ API ដោយមិនបំបែក Client ចាស់ភ្លាមៗ។

## ១. API Versioning ជាអ្វី?

API Versioning បង្ហាញពីកិច្ចសន្យា (Contract) របស់ API ដែល Client អាចពឹងផ្អែកបាន។ បើកែ Response ឬ Behavior ដោយបំបែក Client ចាស់ អាចត្រូវបង្កើត Version ថ្មី ឬផែនការផ្លាស់ប្តូរដែលច្បាស់។

ការផ្លាស់ប្តូរដែលជាទូទៅអាចជា Breaking Change៖

- លុប ឬប្តូរឈ្មោះ Field។
- ប្តូរ Type របស់ Field ឬទម្រង់កាលបរិច្ឆេទ។
- ប្តូរអត្ថន័យ Status Code ឬ Business Behavior។
- ធ្វើឱ្យ Field ដែលពីមុន Optional ទៅជា Required។

ការបន្ថែម Field Optional ថ្មី ជាញឹកញាប់អាចរក្សា Backward Compatibility ប៉ុន្តែ Client ខ្លះអាចបរាជ័យបើវាមិនអើពើ Field មិនស្គាល់។ ត្រូវពិចារណាឥរិយាបថ Client ពិត។

## ២. ដាក់ Version ក្នុង URL

វិធីសាមញ្ញគឺដាក់ Major Version ក្នុង Path៖

```text
GET /api/v1/posts
GET /api/v2/posts
```

អត្ថប្រយោជន៍គឺមើលឃើញច្បាស់ និងងាយ Route/Debug។ Header Versioning ក៏អាចប្រើបាន ប៉ុន្តែត្រូវមានការគ្រប់គ្រង Cache និង Documentation ច្បាស់។ ជ្រើសរើសវិធីមួយ ហើយប្រើឱ្យស្របគ្នា។

Version មិនចាំបាច់បង្កើតសម្រាប់រាល់ការកែតូចទេ។ កំណត់ច្បាប់ថាតើអ្វីជា Breaking Change, រយៈពេល Support និងរបៀបប្រកាស Deprecation។

## ៣. OpenAPI ជាអ្វី?

OpenAPI Specification (OAS) ជាទម្រង់ស្តង់ដារសម្រាប់ពិពណ៌នា HTTP API ដោយមិនចងជាប់នឹង Programming Language។ វាអាចពិពណ៌នាអំពី Paths, Methods, Parameters, Request Body, Response Schema, Security និង Error Responses។

ឯកសារនេះអាចជួយអ្នកប្រើ API និង Tooling បង្កើត Client/Test ដោយមិនចាំបាច់ទាយ Contract ពី Source Code។ វាមិនមែនជាការជំនួស Security Check ឬការអនុវត្ត API ទេ។

## ៤. OpenAPI YAML ឧទាហរណ៍

បង្កើត `openapi.yaml` សម្រាប់ List និង Create Post៖

```yaml
openapi: 3.1.1
info:
  title: Learning PHP API
  version: 1.0.0
  description: API សម្រាប់គ្រប់គ្រងអត្ថបទសិក្សា។
servers:
  - url: https://api.example.com
paths:
  /api/v1/posts:
    get:
      summary: បញ្ជីអត្ថបទ
      parameters:
        - name: page
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: per_page
          in: query
          required: false
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        "200":
          description: បញ្ជីអត្ថបទ
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Post"
                  meta:
                    type: object
        "400":
          description: Query Parameter មិនត្រឹមត្រូវ
    post:
      summary: បង្កើតអត្ថបទ
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [title, body]
              properties:
                title:
                  type: string
                  maxLength: 200
                body:
                  type: string
      responses:
        "201":
          description: អត្ថបទត្រូវបានបង្កើត
        "401":
          description: មិនបាន Authenticate
        "403":
          description: គ្មានសិទ្ធិបង្កើត
        "422":
          description: Input មិនត្រឹមត្រូវ
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
  schemas:
    Post:
      type: object
      required: [id, title, body]
      properties:
        id:
          type: integer
        title:
          type: string
        body:
          type: string
```

YAML Indentation មានសារៈសំខាន់។ ត្រូវ Validate Spec ដោយ OpenAPI Tool ក្នុង Build/CI ដើម្បីរក Syntax/Reference ខូច។

## ៥. រៀបចំ API Version ក្នុង PHP

អាច Route តាម Prefix នៅ Front Controller៖

```php
<?php
$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);

if (!is_string($path)) {
    http_response_code(400);
    exit;
}

if (str_starts_with($path, '/api/v1/')) {
    require __DIR__ . '/../routes/api-v1.php';
} elseif (str_starts_with($path, '/api/v2/')) {
    require __DIR__ . '/../routes/api-v2.php';
} else {
    http_response_code(404);
}
```

ក្នុង Framework ពិត ប្រើ Router របស់ Framework ជំនួសការចែក Route ដោយ String ផ្ទាល់។ កុំឱ្យ URL Version ធ្វើឱ្យ Authorization/Validation ខុសគ្នាដោយចៃដន្យ; Version នីមួយៗត្រូវមាន Security Middleware និង Contract ដែលបានពិនិត្យ។

## ៦. អ្វីដែល Documentation គួរបញ្ជាក់

សម្រាប់រាល់ Endpoint កត់ត្រា៖

- HTTP Method និង Path។
- Authentication/Permission ដែលត្រូវការ។
- Query/Path Parameters, Type, Default និង Limit។
- Request Body Schema និង Field ដែល Required។
- Response Body ឧទាហរណ៍ និង Status Code។
- Error Format និងមូលហេតុដែលអាចកើត។
- Pagination, Rate Limit និង Idempotency ប្រសិនបើពាក់ព័ន្ធ។
- Version Support/Deprecation Policy និងកាលបរិច្ឆេទបិទ API ចាស់។

កុំដាក់ API Key ឬ Token ពិតក្នុង Documentation/Example។

## ៧. Deprecation និង Migration

ពេលបង្កើត Version ថ្មី៖

1. កត់ត្រា Breaking Changes និងផែនការផ្លាស់ប្តូរ។
2. ផ្តល់ការណែនាំ Client ឱ្យច្បាស់ និងឧទាហរណ៍ Request/Response។
3. ប្រសិនបើចាំបាច់ ផ្ញើ Deprecation/Sunset Notice តាម Policy របស់សេវា។
4. តាមដានការប្រើ Version ចាស់ ដោយមិន Log Credential។
5. បិទ Version ចាស់ក្រោយដំណឹង និងរយៈពេលគាំទ្រសមស្រប។

កុំបិទ Version ចាស់ដោយគ្មានការជូនដំណឹងដែលសមនឹងផលប៉ះពាល់អ្នកប្រើ។

## ៨. កំហុសដែលជួបញឹកញាប់

- API ដំណើរការមិនដូច Documentation។
- មិនកត់ Error Status ឬ Authentication Requirement។
- បង្កើត Version ថ្មីសម្រាប់ការកែគ្រប់យ៉ាង ហើយបង្កើនការថែទាំ។
- កែ Response របស់ Version ចាស់ដោយ Breaking Change។
- ដាក់ Secret ពិតនៅក្នុង OpenAPI Example។
- គិតថា OpenAPI Validation ធានាថា Runtime API មានសុវត្ថិភាព។

## ៩. លំហាត់

1. សរសេរ OpenAPI Path សម្រាប់ `GET /api/v1/products/{id}`។
2. កត់ត្រា 200, 401, 403 និង 404 Response។
3. បន្ថែម Bearer Authentication Scheme និងពិពណ៌នា Permission។
4. រៀបចំច្បាប់ថាតើ Field បន្ថែមថ្មីណាអាចរក្សា Compatibility។
5. សរសេរផែនការប្រកាស Deprecation សម្រាប់ Version ចាស់។

## ១០. ចម្លើយណែនាំខ្លី

- Versioning គ្រប់គ្រង API Contract និងផ្តល់ផ្លូវសម្រាប់ Breaking Change។
- OpenAPI ពិពណ៌នា Path, Schema, Security និង Responses ក្នុងទម្រង់ Machine-readable។
- កត់ត្រា Authentication, Permission និង Error Status របស់ Endpoint ទាំងអស់។
- Validate YAML/OpenAPI Spec ដោយ Tooling ក្នុង Development/CI។
- កុំដាក់ Secret ពិតក្នុង Spec ហើយកុំយល់ថា Spec ជំនួសការត្រួតពិនិត្យ Runtime។

## ឯកសារយោង

- [OpenAPI Specification v3.1.1](https://spec.openapis.org/oas/v3.1.1.html)
- [មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP](lesson-26-php-json-rest-api.md)
- [មេរៀនទី ៤១៖ Pagination, Filtering និង Sorting សម្រាប់ API](lesson-41-php-api-pagination-filtering-sorting.md)
