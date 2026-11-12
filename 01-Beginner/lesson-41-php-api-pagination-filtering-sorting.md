# មេរៀនទី ៤១៖ Pagination, Filtering និង Sorting សម្រាប់ PHP API

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Offset Pagination និង Cursor Pagination។
- Validate `page` និង `per_page` ដោយកំណត់អតិបរមា។
- បង្កើត Filtering ដោយ Prepared Statements។
- ប្រើ Allowlist សម្រាប់ Sort Column និង Direction។
- ត្រឡប់ JSON Response ដែលមាន Metadata និងទម្រង់ស្ថិតស្ថេរ។

> តម្រូវការ៖ PHP 8+, PDO និងចំណេះដឹង REST API មេរៀនទី ២៦។ ឧទាហរណ៍ SQL ប្រើ MySQL។

## ១. ហេតុអ្វី API ត្រូវការ Pagination?

បើ API ត្រឡប់ Record ទាំងអស់ក្នុង Request តែមួយ វាអាចប្រើ Memory/Network ច្រើន និងធ្វើឱ្យ Response យឺត។ Pagination បែងចែកទិន្នន័យជាផ្នែកតូចៗ ហើយកំណត់ចំនួនអតិបរមាដែល Client អាចស្នើបាន។

ត្រូវមានលំដាប់ដែលមិនប្រែប្រួលសម្រាប់ Pagination។ បើ Sort មិនមាន Tie-Breaker ដូចជា ID នោះ Record អាចលោត ឬស្ទួនរវាង Page។

## ២. Validate Page និង Page Size

ទទួល Query Parameter ជា String និង Validate ជាចំនួនគត់៖

```php
<?php
function queryInt(string $name, int $default, int $min, int $max): int
{
    $raw = $_GET[$name] ?? (string) $default;

    if (!is_string($raw)) {
        http_response_code(400);
        exit('Query Parameter មិនត្រឹមត្រូវ។');
    }

    $value = filter_var(
        $raw,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => $min, 'max_range' => $max]]
    );

    if ($value === false) {
        http_response_code(400);
        exit('Query Parameter មិនត្រឹមត្រូវ។');
    }

    return $value;
}

$page = queryInt('page', 1, 1, 1_000_000);
$perPage = queryInt('per_page', 20, 1, 100);
$offset = ($page - 1) * $perPage;
```

ក្នុងកម្មវិធីពិត ត្រូវពិនិត្យ Overflow/Offset សមស្របនឹង Database និងទំហំ Dataset។ កំណត់ `per_page` អតិបរមា ដើម្បីកុំឱ្យ Client ស្នើទិន្នន័យធំពេក។

## ៣. Filtering ដោយ Prepared Statement

បង្កើត SQL Condition តាម Filter ដែលបានអនុញ្ញាត ហើយ Bind តម្លៃ៖

```php
<?php
$conditions = [];
$params = [];

$status = $_GET['status'] ?? null;
if ($status !== null) {
    $allowedStatuses = ['draft', 'published'];

    if (
        !is_string($status)
        || !in_array($status, $allowedStatuses, true)
    ) {
        http_response_code(400);
        exit('Filter មិនត្រឹមត្រូវ។');
    }

    $conditions[] = 'status = :status';
    $params['status'] = $status;
}

$search = $_GET['q'] ?? null;
if ($search !== null) {
    if (!is_string($search) || mb_strlen($search, 'UTF-8') > 100) {
        http_response_code(400);
        exit('ពាក្យស្វែងរកមិនត្រឹមត្រូវ។');
    }

    $conditions[] = 'title LIKE :search';
    $params['search'] = '%' . $search . '%';
}

$where = $conditions === [] ? '' : 'WHERE ' . implode(' AND ', $conditions);
```

Prepared Statements ការពារតម្លៃដែលបាន Bind; វាមិនចាំបាច់ធ្វើឱ្យ Wildcard `%` និង `_` ក្នុង LIKE ក្លាយជាអក្សរធម្មតាទេ។ បើត្រូវការស្វែងរកអក្សរតាមពិត ត្រូវ Escape Wildcard តាម Syntax Database។

## ៤. Sort Column និង Direction ដោយ Allowlist

SQL Identifier និង Keyword មិនអាច Bind ជា Placeholder បានទេ។ កុំភ្ជាប់ Sort Parameter ពី User ដោយផ្ទាល់៖

```php
<?php
$sortMap = [
    'created' => 'created_at',
    'title' => 'title',
];

$sortKey = $_GET['sort'] ?? 'created';
if (!is_string($sortKey) || !isset($sortMap[$sortKey])) {
    http_response_code(400);
    exit('Sort មិនត្រឹមត្រូវ។');
}

$directionInput = $_GET['direction'] ?? 'desc';
$directions = ['asc' => 'ASC', 'desc' => 'DESC'];

if (!is_string($directionInput) || !isset($directions[$directionInput])) {
    http_response_code(400);
    exit('Direction មិនត្រឹមត្រូវ។');
}

$orderBy = $sortMap[$sortKey] . ' ' . $directions[$directionInput] . ', id DESC';
```

នៅទីនេះ Column និង Direction ចេញពី Allowlist ដែល Server កំណត់។ `id DESC` ជា Tie-Breaker ដើម្បីឱ្យលទ្ធផលមានលំដាប់ច្បាស់។

## ៥. Query និង JSON Response

```php
<?php
$sql = 'SELECT id, title, status, created_at FROM posts '
    . $where
    . ' ORDER BY ' . $orderBy
    . ' LIMIT :limit OFFSET :offset';

$stmt = $pdo->prepare($sql);

foreach ($params as $name => $value) {
    $stmt->bindValue(':' . $name, $value, PDO::PARAM_STR);
}
$stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$stmt->execute();

$items = $stmt->fetchAll(PDO::FETCH_ASSOC);

header('Content-Type: application/json; charset=utf-8');
echo json_encode([
    'data' => $items,
    'meta' => [
        'page' => $page,
        'per_page' => $perPage,
        'count' => count($items),
    ],
], JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR);
```

ចំណាំ៖ Driver/Database អាចមានច្បាប់ខុសគ្នាអំពី Bind LIMIT/OFFSET; សាកល្បងលើ Driver ពិតដែល Project ប្រើ។ កុំដាក់ Value ដែលមិនបាន Validate ទៅ SQL ដោយផ្ទាល់។

បើត្រូវការចំនួន Page សរុប អាច Query `COUNT(*)` ដោយប្រើ Filter ដូចគ្នា ប៉ុន្តែវាមាន Cost នៅ Dataset ធំ។ API ខ្លះប្រើ `has_more` ជំនួស Total Count។

## ៦. Offset និង Cursor Pagination

**Offset**៖ `LIMIT 20 OFFSET 100` ងាយប្រើ និងអនុញ្ញាតឱ្យទៅ Page ឆ្ងាយ ប៉ុន្តែ Offset ធំអាចយឺត ហើយការបញ្ចូល Record ថ្មីអាចធ្វើឱ្យ Page ផ្លាស់ប្តូរ។

**Cursor**៖ ស្នើ Record បន្ទាប់ពី Key ចុងក្រោយ ដូចជា `created_at, id`។ វាសមស្របសម្រាប់ Feed ធំ និងទិន្នន័យផ្លាស់ប្តូរញឹកញាប់ ប៉ុន្តែត្រូវមាន Sort Key មានស្ថិរភាព ហើយ Cursor ត្រូវ Validate/Encode មិនគួរទុកចិត្តពី Client ទេ។

ឧទាហរណ៍ទិសដៅ Query Cursor សម្រាប់លំដាប់ចាស់ទៅថ្មី៖

```sql
SELECT id, title, created_at
FROM posts
WHERE (created_at, id) > (:cursor_created_at, :cursor_id)
ORDER BY created_at ASC, id ASC
LIMIT :limit
```

Cursor គួរត្រូវបានចុះហត្ថលេខា ឬរក្សាទុកជាទម្រង់ដែលមិនអាចកែទៅស្នើលទ្ធផលខុសគោលបំណងបាន។ ទោះ Cursor ត្រូវបានចុះហត្ថលេខាក៏ដោយ វាមិនជំនួស Authorization ទេ។

## ៧. កំហុសដែលជួបញឹកញាប់

- អនុញ្ញាត `per_page` ធំពេក។
- ភ្ជាប់ Sort Column ពី Query String ទៅ SQL។
- ប្រើ Sort មិនមាន Tie-Breaker ហើយឃើញ Record ស្ទួន/បាត់។
- Bind `LIMIT` ជា String ហើយ Driver បកស្រាយខុស។
- ផ្ញើទិន្នន័យទាំងអស់នៅក្នុង API Response មួយ។
- មិន Validate Filter ឬមិនកំណត់ប្រវែង Search Query។
- ភ្លេចពិនិត្យ Authorization/Owner Constraint នៅ List Query។

## ៨. លំហាត់

1. បង្កើត Endpoint List Products ដែលមាន `page` និង `per_page` ចន្លោះ 1–100។
2. បន្ថែម Filter Status តាម Allowlist។
3. អនុញ្ញាត Sort តែ `name` និង `created_at` ហើយមាន ID Tie-Breaker។
4. បន្ថែម Metadata `page`, `per_page`, `has_more`។
5. ពិភាក្សាថា Dataset របស់អ្នកគួរប្រើ Offset ឬ Cursor Pagination។

## ៩. ចម្លើយណែនាំខ្លី

- Validate Page/Limit ជាចំនួនគត់ និងកំណត់ Upper Bound។
- Bind Filter Values ដោយ Prepared Statement។
- Map Sort Key របស់ Client ទៅ SQL Column តាម Allowlist; Bind Identifier មិនបានទេ។
- បន្ថែម Tie-Breaker ដូចជា ID ដើម្បីឱ្យលំដាប់មាន Deterministic។
- Cursor Pagination ល្អសម្រាប់បន្តលើ Dataset ធំ; Offset ងាយប្រើសម្រាប់ទិន្នន័យតូច/ទំព័រលោត។

## ឯកសារយោង

- [PHP: PDO::prepare()](https://www.php.net/manual/en/pdo.prepare.php)
- [មេរៀនទី ១៧៖ Database Relationships និង Pagination](lesson-17-php-database-relationships-joins-pagination.md)
- [មេរៀនទី ២៦៖ បង្កើត JSON REST API ដោយ PHP](lesson-26-php-json-rest-api.md)
