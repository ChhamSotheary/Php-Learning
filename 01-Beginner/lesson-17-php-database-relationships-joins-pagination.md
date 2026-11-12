# មេរៀនទី ១៧៖ Database Relationships, JOIN និង Pagination

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- យល់ពី Primary Key និង Foreign Key។
- បង្កើតទំនាក់ទំនង One-to-Many រវាង Table។
- ប្រើ `INNER JOIN` និង `LEFT JOIN` ដើម្បីអានទិន្នន័យពី Table ច្រើន។
- ជៀសវាងការបង្កើត Query មួយសម្រាប់ Row នីមួយៗ។
- បង្កើត Pagination ដោយប្រើ `LIMIT` និង `OFFSET`។
- ប្រើ PDO Prepared Statements សម្រាប់ Query ដែលមាន Input។
- Validate Page Number និងកំណត់ទំហំទំព័រ។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL និង PDO MySQL driver។ មេរៀននេះបន្តពីការភ្ជាប់ Database និង Prepared Statements ក្នុងមេរៀនទី ១៦។

---

## ១. ហេតុអ្វីត្រូវមាន Relationships?

ពេលទិន្នន័យមានទំនាក់ទំនងគ្នា គួររក្សាទុកក្នុង Table ផ្សេងៗ ដើម្បីកាត់បន្ថយទិន្នន័យស្ទួន និងងាយស្រួលកែប្រែ។

ឧទាហរណ៍៖

- Student ម្នាក់អាចមានការចុះឈ្មោះ Course ច្រើន។
- Course មួយអាចមាន Student ច្រើន។
- Student និង Course មានទំនាក់ទំនង Many-to-Many។

Database អាចភ្ជាប់ Table ទាំងនេះដោយ Key។

## ២. Primary Key និង Foreign Key

**Primary Key** កំណត់អត្តសញ្ញាណ Row មួយៗដោយតម្លៃមិនស្ទួន។ ជាទូទៅប្រើ Column ដូចជា `id`។

**Foreign Key** ជា Column ដែលយោងទៅ Primary Key របស់ Table ផ្សេង ដើម្បីបង្កើតទំនាក់ទំនង និងរក្សា Referential Integrity។

ឧទាហរណ៍ Student ម្នាក់មាន Order ច្រើន៖

```sql
CREATE TABLE students (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE courses (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(150) NOT NULL
);

CREATE TABLE enrollments (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    student_id INT UNSIGNED NOT NULL,
    course_id INT UNSIGNED NOT NULL,
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_enrollments_student
        FOREIGN KEY (student_id) REFERENCES students(id),
    CONSTRAINT fk_enrollments_course
        FOREIGN KEY (course_id) REFERENCES courses(id),
    CONSTRAINT uq_student_course UNIQUE (student_id, course_id)
);
```

Table `enrollments` ជា Junction Table សម្រាប់ភ្ជាប់ Student និង Course។ Unique Constraint ការពារការចុះឈ្មោះ Course ដដែលច្រើនដងដោយ Student ម្នាក់។

Foreign Key ត្រូវការប្រើ Storage Engine ដែលគាំទ្រវា ដូចជា InnoDB។ MySQL ប្រើ InnoDB ជាទូទៅ ប៉ុន្តែអាចបញ្ជាក់បាន៖

```sql
CREATE TABLE example_items (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(100) NOT NULL
) ENGINE=InnoDB;
```

## ៣. បញ្ចូលទិន្នន័យដែលមានទំនាក់ទំនង

បញ្ចូល Student និង Course ជាមុន បន្ទាប់មកបញ្ចូលទំនាក់ទំនងក្នុង Enrollment៖

```sql
INSERT INTO students (name)
VALUES ('Dara'), ('Sokha');

INSERT INTO courses (title)
VALUES ('PHP Basics'), ('MySQL Basics');

INSERT INTO enrollments (student_id, course_id)
VALUES (1, 1), (1, 2), (2, 1);
```

លេខ ID ក្នុង Database ពិតអាចខុសពីឧទាហរណ៍។ កម្មវិធី PHP គួរទាញ ID ពី Database ឬ Query មិនគួរសន្មតថាវាតែងតែចាប់ផ្ដើមពី 1 ទេ។

## ៤. INNER JOIN

`INNER JOIN` ត្រឡប់ Row ដែលមានការផ្គូផ្គងនៅ Table ទាំងពីរ៖

```sql
SELECT
    students.name AS student_name,
    courses.title AS course_title,
    enrollments.enrolled_at
FROM enrollments
INNER JOIN students
    ON students.id = enrollments.student_id
INNER JOIN courses
    ON courses.id = enrollments.course_id
ORDER BY students.name, courses.title;
```

លទ្ធផលនឹងមាន Student និង Course ដែលមាន Enrollment ត្រូវគ្នា។ Student ដែលមិនទាន់ចុះឈ្មោះ Course នឹងមិនបង្ហាញក្នុងលទ្ធផល INNER JOIN នេះទេ។

## ៥. LEFT JOIN

`LEFT JOIN` រក្សា Row ទាំងអស់ពី Table ខាងឆ្វេង ទោះគ្មានការផ្គូផ្គងនៅ Table ខាងស្ដាំក៏ដោយ៖

```sql
SELECT
    students.id,
    students.name,
    courses.title AS course_title
FROM students
LEFT JOIN enrollments
    ON enrollments.student_id = students.id
LEFT JOIN courses
    ON courses.id = enrollments.course_id
ORDER BY students.name, courses.title;
```

បើ Student មិនទាន់ចុះឈ្មោះ Course នោះ `course_title` នឹងជា `NULL`។ ពិនិត្យតម្លៃ Null នៅ PHP មុនបង្ហាញ៖

```php
<?php
$courseTitle = $row['course_title'] ?? null;

if ($courseTitle === null) {
    echo 'មិនទាន់ចុះឈ្មោះ Course';
} else {
    echo htmlspecialchars($courseTitle, ENT_QUOTES, 'UTF-8');
}
```

## ៦. ប្រើ JOIN ជាមួយ PDO

```php
<?php
require __DIR__ . '/database.php';

$sql = '
    SELECT
        students.name AS student_name,
        courses.title AS course_title,
        enrollments.enrolled_at
    FROM enrollments
    INNER JOIN students
        ON students.id = enrollments.student_id
    INNER JOIN courses
        ON courses.id = enrollments.course_id
    WHERE students.id = :student_id
    ORDER BY courses.title
';

$stmt = $pdo->prepare($sql);
$stmt->execute(['student_id' => 1]);

$enrollments = $stmt->fetchAll();

foreach ($enrollments as $enrollment) {
    echo htmlspecialchars(
        $enrollment['student_name'],
        ENT_QUOTES,
        'UTF-8'
    );
    echo ' — ';
    echo htmlspecialchars(
        $enrollment['course_title'],
        ENT_QUOTES,
        'UTF-8'
    );
    echo '<br>';
}
```

ប្រើ Alias ដូចជា `student_name` និង `course_title` ដើម្បីឱ្យ Column Name មិនច្រឡំគ្នា ហើយកូដ PHP ងាយអាន។

## ៧. រាប់ទិន្នន័យជាមួយ JOIN

ប្រើ Aggregate Function ដូចជា `COUNT()` ដើម្បីរាប់ Course របស់ Student ម្នាក់ៗ៖

```sql
SELECT
    students.id,
    students.name,
    COUNT(enrollments.course_id) AS course_count
FROM students
LEFT JOIN enrollments
    ON enrollments.student_id = students.id
GROUP BY students.id, students.name
ORDER BY students.name;
```

ប្រើ `COUNT(enrollments.course_id)` ជំនួស `COUNT(*)` ក្នុងឧទាហរណ៍នេះ ដើម្បីឱ្យ Student ដែលគ្មាន Enrollment មានចំនួន 0។ `COUNT(*)` អាចរាប់ Row ដែលបានបំពេញដោយ LEFT JOIN ផងដែរ។

## ៨. Pagination ជាអ្វី?

Pagination បែងចែកលទ្ធផលច្រើនទៅជាច្រើនទំព័រ។ វាជួយកុំឱ្យទាញទិន្នន័យទាំងអស់មកក្នុង Memory និងធ្វើឱ្យទំព័របង្ហាញលឿនជាងមុន។

MySQL ប្រើ៖

- `LIMIT` កំណត់ចំនួន Row ក្នុងមួយទំព័រ។
- `OFFSET` រំលង Row ចំនួនប៉ុន្មានមុនចាប់ផ្ដើមទាញលទ្ធផល។

ឧទាហរណ៍ ទំព័រទី 2 ដែលមាន 10 Row ក្នុងមួយទំព័រ៖

```sql
SELECT id, name
FROM students
ORDER BY id
LIMIT 10 OFFSET 10;
```

ប្រើ `ORDER BY` ដែលមានលំដាប់ច្បាស់លាស់ ដើម្បីឱ្យ Row មិនរអិលទៅមករវាងទំព័រ។

## ៩. បង្កើត Pagination ដោយ PHP និង PDO

```php
<?php
require __DIR__ . '/database.php';

$perPage = 10;
$pageInput = filter_input(INPUT_GET, 'page', FILTER_VALIDATE_INT);
$page = ($pageInput !== false && $pageInput !== null && $pageInput > 0)
    ? $pageInput
    : 1;

$countStmt = $pdo->query('SELECT COUNT(*) FROM students');
$totalStudents = (int) $countStmt->fetchColumn();

$totalPages = max(1, (int) ceil($totalStudents / $perPage));
$page = min($page, $totalPages);

$offset = ($page - 1) * $perPage;

$stmt = $pdo->prepare(
    'SELECT id, name, email
     FROM students
     ORDER BY id DESC
     LIMIT :limit OFFSET :offset'
);

$stmt->bindValue(':limit', $perPage, PDO::PARAM_INT);
$stmt->bindValue(':offset', $offset, PDO::PARAM_INT);
$stmt->execute();

$students = $stmt->fetchAll();

foreach ($students as $student) {
    echo htmlspecialchars($student['name'], ENT_QUOTES, 'UTF-8');
    echo ' — ';
    echo htmlspecialchars($student['email'], ENT_QUOTES, 'UTF-8');
    echo '<br>';
}

if ($page > 1) {
    echo '<a href="?page=' . ($page - 1) . '">មុន</a> ';
}

echo 'ទំព័រ ' . $page . ' នៃ ' . $totalPages;

if ($page < $totalPages) {
    echo ' <a href="?page=' . ($page + 1) . '">បន្ទាប់</a>';
}
```

ចំណុចសំខាន់៖

- `filter_input()` ត្រួតពិនិត្យ Page Number ថាជាចំនួនគត់។
- `max(1, ...)` ការពារករណីមិនមានទិន្នន័យ ឬ Page សរុបសូន្យ។
- `min()` កំណត់ Page ឱ្យមិនលើសទំព័រចុងក្រោយ។
- `bindValue()` ជាមួយ `PDO::PARAM_INT` បញ្ជាក់ថា LIMIT និង OFFSET ជាចំនួនគត់។
- ប្រើទំហំ Page ថេរ ឬកំណត់អតិបរមា កុំទទួលតម្លៃធំៗពី Query String ដោយគ្មានការកំណត់។

## ១០. Mini Project៖ Course Directory

បង្កើត Database តាម SQL នៅដើមមេរៀន រួចបង្កើត `index.php` ដើម្បីបង្ហាញសិស្ស និង Course របស់ពួកគេ៖

```php
<?php
declare(strict_types=1);

require __DIR__ . '/database.php';

$sql = '
    SELECT
        students.id,
        students.name,
        courses.title AS course_title
    FROM students
    LEFT JOIN enrollments
        ON enrollments.student_id = students.id
    LEFT JOIN courses
        ON courses.id = enrollments.course_id
    ORDER BY students.name, courses.title
';

$rows = $pdo->query($sql)->fetchAll();

$students = [];

foreach ($rows as $row) {
    $studentId = (int) $row['id'];

    if (!isset($students[$studentId])) {
        $students[$studentId] = [
            'name' => $row['name'],
            'courses' => [],
        ];
    }

    if ($row['course_title'] !== null) {
        $students[$studentId]['courses'][] = $row['course_title'];
    }
}
?>
<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>សិស្ស និង Course</title>
</head>
<body>
    <h1>សិស្ស និង Course</h1>

    <?php if ($students === []): ?>
        <p>មិនទាន់មានសិស្សទេ។</p>
    <?php else: ?>
        <?php foreach ($students as $student): ?>
            <section>
                <h2><?= htmlspecialchars($student['name'], ENT_QUOTES, 'UTF-8') ?></h2>

                <?php if ($student['courses'] === []): ?>
                    <p>មិនទាន់ចុះឈ្មោះ Course ទេ។</p>
                <?php else: ?>
                    <ul>
                        <?php foreach ($student['courses'] as $course): ?>
                            <li><?= htmlspecialchars($course, ENT_QUOTES, 'UTF-8') ?></li>
                        <?php endforeach; ?>
                    </ul>
                <?php endif; ?>
            </section>
        <?php endforeach; ?>
    <?php endif; ?>
</body>
</html>
```

ដំណើរការដោយប្រើ PHP Local Server ដូចមេរៀនមុន។ បន្ថែម Student ដែលមាន Course និង Student ដែលមិនទាន់មាន Course ដើម្បីឃើញភាពខុសគ្នារបស់ LEFT JOIN។

## ១១. សុវត្ថិភាព និង Best Practices

1. **កំណត់ Foreign Key និង Constraint នៅ Database។** ការត្រួតពិនិត្យនៅកម្មវិធីតែមួយមិនគ្រប់គ្រាន់ទេ។
2. **ប្រើ Prepared Statements សម្រាប់ Input។** កុំបញ្ចូលតម្លៃផ្ទាល់ទៅក្នុង SQL String។
3. **កុំយកឈ្មោះ Column ឬ Table ពីអ្នកប្រើដោយផ្ទាល់។** ប្រើ Allowlist ដែលកំណត់ក្នុងកូដ។
4. **Validate Page Number និង Page Size។** កំណត់តម្លៃអប្បបរមា និងអតិបរមា។
5. **ប្រើ ORDER BY ក្នុង Pagination។** លទ្ធផលគួរមានលំដាប់ស្ថិរភាព។
6. **Escape ទិន្នន័យមុនបង្ហាញ HTML។** ទិន្នន័យក្នុង Database ក៏អាចមានមាតិកាមិនទុកចិត្តបាន។
7. **ប្រើ Index នៅលើ Foreign Key និង Column ដែលស្វែងរកញឹកញាប់។** វាអាចជួយ Query ដំណើរការលឿន ប៉ុន្តែត្រូវផ្អែកលើរបៀបប្រើប្រាស់ពិត។
8. **កុំផ្ញើ Database Error លម្អិតទៅអ្នកប្រើ។** កត់ត្រាកំហុសក្នុង Log ហើយបង្ហាញសារសមរម្យ។

## ១២. កំហុសដែលជួបញឹកញាប់

- **ភ្លេច JOIN Condition៖** អាចបង្ក Cartesian Product និងបង្ហាញ Row ច្រើនខុសធម្មតា។
- **ប្រើ INNER JOIN ពេលចង់ឃើញ Row ដែលគ្មានទំនាក់ទំនង៖** ត្រូវប្រើ LEFT JOIN។
- **ភ្លេច Alias លើ Column ឈ្មោះដូចគ្នា៖** ប្រើ Alias ដើម្បីឱ្យលទ្ធផលងាយយោង។
- **ប្រើ `COUNT(*)` ជាមួយ LEFT JOIN ដោយមិនយល់ពីលទ្ធផល៖** អាចរាប់ Row ដែលបំពេញដោយ JOIN។
- **ទាញទិន្នន័យទាំងអស់មកបង្ហាញ៖** ប្រើ Pagination នៅពេលបញ្ជីធំ។
- **ប្រើ LIMIT/OFFSET ដោយគ្មាន ORDER BY៖** Row អាចប្ដូរលំដាប់។
- **មិនបម្លែង Page Number ទៅ Integer៖** Input ពី URL គឺមិនទុកចិត្តបាន។
- **ប្រើ String Placeholder សម្រាប់ LIMIT/OFFSET៖** Bind ជា Integer ដោយ `PDO::PARAM_INT`។

## ១៣. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ បង្កើតទំនាក់ទំនង

បង្កើត Table `categories` និង `products`។ Product មួយស្ថិតក្នុង Category មួយ ដូច្នេះ Product ត្រូវមាន Foreign Key `category_id`។

### លំហាត់ទី ២៖ INNER JOIN

សរសេរ Query បង្ហាញឈ្មោះ Product និងឈ្មោះ Category របស់វា។

### លំហាត់ទី ៣៖ LEFT JOIN

កែ Query ដើម្បីឱ្យ Category ដែលមិនទាន់មាន Product ក៏បង្ហាញដែរ។

### លំហាត់ទី ៤៖ រាប់ Product

បង្ហាញ Category នីមួយៗ និងចំនួន Product ដែលមានក្នុង Category នោះ។

### លំហាត់ទី ៥៖ Pagination

បង្កើតទំព័របង្ហាញ Product 5 ក្នុងមួយទំព័រ។ Validate Page Number ហើយបង្ហាញ Link មុន/បន្ទាប់។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង INNER JOIN និង LEFT JOIN ហើយប្រាប់ថាករណីណាគួរប្រើមួយៗ។

## ១៤. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```sql
CREATE TABLE categories (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE
) ENGINE=InnoDB;

CREATE TABLE products (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    category_id INT UNSIGNED NOT NULL,
    name VARCHAR(150) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    CONSTRAINT fk_products_category
        FOREIGN KEY (category_id) REFERENCES categories(id)
) ENGINE=InnoDB;
```

### ចម្លើយលំហាត់ទី ២

```sql
SELECT
    products.name AS product_name,
    categories.name AS category_name
FROM products
INNER JOIN categories
    ON categories.id = products.category_id;
```

### ចម្លើយលំហាត់ទី ៣

```sql
SELECT
    categories.name AS category_name,
    products.name AS product_name
FROM categories
LEFT JOIN products
    ON products.category_id = categories.id;
```

### ចម្លើយលំហាត់ទី ៤

```sql
SELECT
    categories.id,
    categories.name,
    COUNT(products.id) AS product_count
FROM categories
LEFT JOIN products
    ON products.category_id = categories.id
GROUP BY categories.id, categories.name
ORDER BY categories.name;
```

### ចម្លើយលំហាត់ទី ៥

ប្រើគំរូ Pagination នៅផ្នែកទី ៩ ដោយកំណត់ `$perPage = 5` និងប្ដូរ Query ទៅ Table `products`។

### ចម្លើយលំហាត់ទី ៦

INNER JOIN បង្ហាញតែ Row ដែលមានទិន្នន័យត្រូវគ្នានៅ Table ទាំងពីរ។ LEFT JOIN បង្ហាញ Row ទាំងអស់ពី Table ខាងឆ្វេង ហើយ Column ខាងស្ដាំជា NULL នៅពេលគ្មានទំនាក់ទំនង។ ប្រើ INNER JOIN ពេលត្រូវការតែទិន្នន័យដែលមានទំនាក់ទំនង ហើយប្រើ LEFT JOIN ពេលចង់រក្សាទុក Row ទាំងអស់ពី Table មេ។

---

## សេចក្ដីសង្ខេប

- Primary Key កំណត់អត្តសញ្ញាណ Row; Foreign Key បង្កើត និងការពារទំនាក់ទំនង។
- Junction Table តំណាងឱ្យទំនាក់ទំនង Many-to-Many។
- INNER JOIN ត្រឡប់តែ Row ដែលផ្គូផ្គងគ្នា; LEFT JOIN រក្សា Row ទាំងអស់ពី Table ខាងឆ្វេង។
- Aggregate Functions ដូចជា COUNT អាចប្រើជាមួយ JOIN និង GROUP BY។
- Pagination ប្រើ LIMIT និង OFFSET ហើយត្រូវ Validate Input និងកំណត់ ORDER BY។
- ប្រើ Prepared Statements និង Escape លទ្ធផលមុនបង្ហាញ HTML។

## មេរៀនបន្ទាប់

អាចបន្តដោយប្រើ Database Relationships ទាំងនេះដើម្បីសិក្សា Transactions, CRUD Forms និងការរចនារចនាសម្ព័ន្ធកម្មវិធី PHP។
