# មេរៀនទី ១៨៖ PDO Transactions និងភាពត្រឹមត្រូវនៃទិន្នន័យ

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ថា Database Transaction ដោះស្រាយបញ្ហាអ្វី។
- ប្រើ `beginTransaction()`, `commit()` និង `rollBack()` ជាមួយ PDO។
- ធ្វើឱ្យការផ្លាស់ប្ដូរច្រើនជំហានជោគជ័យទាំងអស់ ឬត្រឡប់ក្រោយទាំងអស់។
- ប្រើ Transaction សម្រាប់ការងារដូចជា ផ្ទេរប្រាក់ ឬបង្កើត Order។
- យល់ពី InnoDB, Row Lock និងការប្រុងប្រយ័ត្នពេលមាន Request ស្របគ្នា។
- កត់ត្រាកំហុសដោយមិនបង្ហាញព័ត៌មានសម្ងាត់។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ, MySQL/InnoDB និង PDO MySQL។ មេរៀននេះបន្តពី PDO និង Prepared Statements ក្នុងមេរៀនទី ១៦។

---

## ១. Transaction ជាអ្វី?

Transaction គឺជាក្រុមប្រតិបត្តិការលើ Database ដែលត្រូវចាត់ទុកជាការងារតែមួយ។ ឧទាហរណ៍ ការផ្ទេរប្រាក់ត្រូវ៖

1. ដកប្រាក់ពីគណនីដើម។
2. បញ្ចូលប្រាក់ទៅគណនីគោលដៅ។
3. កត់ត្រាប្រតិបត្តិការផ្ទេរ។

បើជំហានណាមួយបរាជ័យ មិនគួរទុកឱ្យប្រាក់ត្រូវបានដកតែម្ខាងទេ។ Transaction អនុញ្ញាតឱ្យ Commit ការផ្លាស់ប្ដូរទាំងអស់ ឬ Rollback ដើម្បីត្រឡប់ការផ្លាស់ប្ដូរទាំងនោះវិញ។

គោលការណ៍ ACID ត្រូវបានប្រើដើម្បីពិពណ៌នាលក្ខណៈរបស់ Transaction៖

- **Atomicity**៖ ការផ្លាស់ប្ដូរទាំងអស់កើតឡើងជាមួយគ្នា ឬមិនកើតឡើងទាំងអស់។
- **Consistency**៖ ទិន្នន័យនៅតែគោរពលក្ខខណ្ឌ និង Constraint របស់ Database។
- **Isolation**៖ Transaction ដែលកំពុងដំណើរការមិនគួរបង្កលទ្ធផលច្របូកច្របល់ជាមួយ Transaction ផ្សេង។
- **Durability**៖ បន្ទាប់ពី Commit ទិន្នន័យត្រូវបានរក្សាទុកតាមលក្ខណៈរបស់ Database។

ការធានាពិតប្រាកដអាស្រ័យលើ Database, Storage Engine និងការកំណត់របស់ Server។

## ២. ចាប់ផ្ដើម និងបញ្ចប់ Transaction

PDO ផ្ដល់ Method សំខាន់ ៣៖

- `beginTransaction()` ចាប់ផ្ដើម Transaction។
- `commit()` រក្សាទុកការផ្លាស់ប្ដូរទាំងអស់។
- `rollBack()` បោះបង់ការផ្លាស់ប្ដូរចាប់ពីពេល Transaction ចាប់ផ្ដើម។

```php
<?php
require __DIR__ . '/database.php';

try {
    $pdo->beginTransaction();

    // ដាក់ INSERT, UPDATE ឬ DELETE ដែលជាផ្នែកនៃការងារតែមួយនៅទីនេះ។

    $pdo->commit();
    echo 'ការងាររួចរាល់។';
} catch (Throwable $e) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    error_log($e->getMessage());
    http_response_code(500);
    echo 'មិនអាចបញ្ចប់ការងារបានទេ។';
}
```

បើ Exception កើតមុន Commit នោះ Catch នឹង Rollback ដើម្បីកុំឱ្យសល់ការផ្លាស់ប្ដូរតែផ្នែកមួយ។ ប្រើ `inTransaction()` មុន Rollback ដើម្បីពិនិត្យថា Transaction នៅបើកឬអត់។

## ៣. តម្រូវការ MySQL សម្រាប់ Transaction

MySQL មាន Storage Engine ច្រើន។ តារាងដែលប្រើ Transaction ត្រូវប្រើ Engine ដែលគាំទ្រ Transaction ដូចជា **InnoDB**។

```sql
CREATE TABLE accounts (
    id INT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    owner_name VARCHAR(100) NOT NULL,
    balance DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
    CONSTRAINT chk_accounts_balance_nonnegative CHECK (balance >= 0)
) ENGINE=InnoDB;

CREATE TABLE transfers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    from_account_id INT UNSIGNED NOT NULL,
    to_account_id INT UNSIGNED NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_transfers_from
        FOREIGN KEY (from_account_id) REFERENCES accounts(id),
    CONSTRAINT fk_transfers_to
        FOREIGN KEY (to_account_id) REFERENCES accounts(id),
    CONSTRAINT chk_transfer_amount_positive CHECK (amount > 0)
) ENGINE=InnoDB;
```

CHECK Constraint ត្រូវបានអនុវត្តដោយ MySQL ជំនាន់ថ្មីៗ។ កម្មវិធីនៅតែត្រូវ Validate Input ផងដែរ; Constraint នៅ Database ជាខ្សែការពារបន្ថែម។

**ប្រយ័ត្ន៖** DDL ដូចជា `CREATE TABLE` និង `DROP TABLE` អាចបង្ក Implicit Commit នៅ MySQL។ កុំរំពឹងថា Rollback នឹងលុបចោល DDL និងការផ្លាស់ប្ដូរមុនវា។ បង្កើត Schema ក្រៅ Transaction ដែលដំណើរការក្នុងកម្មវិធី។

## ៤. ឧទាហរណ៍៖ ផ្ទេរប្រាក់

កូដខាងក្រោមផ្ទេរប្រាក់ពីគណនីមួយទៅគណនីមួយទៀត។ ដើម្បីជៀសវាងបញ្ហាភាពជាក់លាក់នៃ Floating Point កូដទទួលចំនួនជា Integer សេន ឧទាហរណ៍ 1250 ស្មើ 12.50។

```php
<?php
declare(strict_types=1);

require __DIR__ . '/database.php';

function transferMoney(
    PDO $pdo,
    int $fromAccountId,
    int $toAccountId,
    int $amountCents
): void {
    if ($fromAccountId < 1 || $toAccountId < 1) {
        throw new InvalidArgumentException('លេខគណនីមិនត្រឹមត្រូវ។');
    }

    if ($fromAccountId === $toAccountId) {
        throw new InvalidArgumentException('គណនីដើម និងគោលដៅត្រូវខុសគ្នា។');
    }

    if ($amountCents < 1) {
        throw new InvalidArgumentException('ចំនួនប្រាក់ត្រូវធំជាងសូន្យ។');
    }

    $amount = number_format($amountCents / 100, 2, '.', '');

    try {
        $pdo->beginTransaction();

        $debit = $pdo->prepare(
            'UPDATE accounts
             SET balance = balance - :amount
             WHERE id = :account_id
               AND balance >= :minimum_balance'
        );

        $debit->execute([
            'amount' => $amount,
            'account_id' => $fromAccountId,
            'minimum_balance' => $amount,
        ]);

        if ($debit->rowCount() !== 1) {
            throw new RuntimeException(
                'រកមិនឃើញគណនីដើម ឬសមតុល្យមិនគ្រប់គ្រាន់។'
            );
        }

        $credit = $pdo->prepare(
            'UPDATE accounts
             SET balance = balance + :amount
             WHERE id = :account_id'
        );

        $credit->execute([
            'amount' => $amount,
            'account_id' => $toAccountId,
        ]);

        if ($credit->rowCount() !== 1) {
            throw new RuntimeException('រកមិនឃើញគណនីគោលដៅ។');
        }

        $record = $pdo->prepare(
            'INSERT INTO transfers
                (from_account_id, to_account_id, amount)
             VALUES
                (:from_id, :to_id, :amount)'
        );

        $record->execute([
            'from_id' => $fromAccountId,
            'to_id' => $toAccountId,
            'amount' => $amount,
        ]);

        $pdo->commit();
    } catch (Throwable $e) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        throw $e;
    }
}

try {
    transferMoney($pdo, 1, 2, 1250);
    echo 'ផ្ទេរប្រាក់ 12.50 រួចរាល់។';
} catch (InvalidArgumentException $e) {
    http_response_code(422);
    echo htmlspecialchars($e->getMessage(), ENT_QUOTES, 'UTF-8');
} catch (Throwable $e) {
    error_log('Transfer failed: ' . $e->getMessage());
    http_response_code(400);
    echo 'មិនអាចផ្ទេរប្រាក់បានទេ។ សូមពិនិត្យព័ត៌មានហើយសាកល្បងម្ដងទៀត។';
}
```

កូដនេះប្រើ UPDATE ដែលមានលក្ខខណ្ឌ `balance >= :minimum_balance`។ Database ធ្វើការពិនិត្យសមតុល្យ និងដកប្រាក់ក្នុងប្រតិបត្តិការតែមួយ ដើម្បីកាត់បន្ថយ Race Condition។ ប្រសិនបើការបញ្ចូលប្រាក់ទៅគណនីគោលដៅ ឬការកត់ត្រា Transfer បរាជ័យ Transaction នឹង Rollback ការដកប្រាក់ផងដែរ។

សម្រាប់ប្រព័ន្ធហិរញ្ញវត្ថុពិត ត្រូវមានការរចនាបន្ថែម ដូចជា Idempotency, Audit Log, ការការពារការផ្ញើ Request ស្ទួន និងការត្រួតពិនិត្យប្រតិបត្តិការជាក់លាក់។

## ៥. ឧទាហរណ៍៖ បង្កើត Order និង Order Items

Order មួយអាចត្រូវការ៖

1. បង្កើត Row ក្នុង `orders`។
2. បញ្ចូល Order Items ច្រើនក្នុង `order_items`។
3. បន្ថយចំនួនស្តុកក្នុង `products`។

ប្រសិនបើជំហានទី 3 បរាជ័យ មិនគួរទុក Order ទទេ ឬ Item ខ្លះៗឱ្យនៅសល់ទេ។ ដាក់ប្រតិបត្តិការទាំងនេះក្នុង Transaction ហើយ Commit តែពេលទាំងអស់ជោគជ័យ។

គំរូលំដាប់កូដ៖

```php
<?php
try {
    $pdo->beginTransaction();

    // 1. INSERT INTO orders (...)
    // 2. ទទួល Order ID ដោយ $pdo->lastInsertId()
    // 3. INSERT INTO order_items (...) សម្រាប់ Item នីមួយៗ
    // 4. UPDATE products
    //    SET stock = stock - :quantity
    //    WHERE id = :product_id AND stock >= :quantity
    // 5. ពិនិត្យ rowCount() ថាស្តុកមានគ្រប់គ្រាន់

    $pdo->commit();
} catch (Throwable $e) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    error_log('Create order failed: ' . $e->getMessage());
    throw $e;
}
```

Prepared Statements នៅតែត្រូវប្រើសម្រាប់ Input ទាំងអស់។ សម្រាប់ស្តុក ការធ្វើ UPDATE ជាមួយលក្ខខណ្ឌ `stock >= :quantity` ជួយឱ្យការបន្ថយស្តុក និងការពិនិត្យចំនួនស្តុកកើតឡើងក្នុង SQL Statement ដដែល។

## ៦. Isolation និងការកែប្រែស្របគ្នា

ពេល Request ច្រើនកែប្រែ Row ដូចគ្នាក្នុងពេលតែមួយ អាចមាន Race Condition ប្រសិនបើកម្មវិធីអានតម្លៃចាស់ ហើយសម្រេចចិត្តពីវា មុនពេលសរសេរតម្លៃថ្មី។

វិធីដោះស្រាយរួមមាន៖

- ធ្វើ UPDATE ជាមួយលក្ខខណ្ឌនៅក្នុង SQL ដូចឧទាហរណ៍សមតុល្យ និងស្តុក។
- ប្រើ Row Lock ដូចជា `SELECT ... FOR UPDATE` ក្នុង Transaction នៅពេលត្រូវអានតម្លៃ ហើយបន្តកែប្រែវា។
- កំណត់ Constraint នៅ Database ដើម្បីបដិសេធទិន្នន័យមិនត្រឹមត្រូវ។
- រក្សា Transaction ឱ្យខ្លី ដើម្បីកុំកាន់ Lock យូរ។

ឧទាហរណ៍ Row Lock៖

```sql
SELECT id, balance
FROM accounts
WHERE id = :account_id
FOR UPDATE;
```

Row Lock គួរប្រើនៅក្នុង Transaction ហើយអាកប្បកិរិយាជាក់លាក់អាស្រ័យលើ Database និង Isolation Level។ កុំចាប់ផ្ដើម Transaction រួចរង់ចាំអ្នកប្រើចុចប៊ូតុង ឬធ្វើការងារយូរដែលមិនចាំបាច់។

## ៧. គ្រប់គ្រងកំហុសឱ្យត្រឹមត្រូវ

- ចាប់ផ្ដើម Transaction ដោយ `$pdo->beginTransaction()` ដើម្បីឱ្យ PDO ដឹងពី Transaction។
- Commit នៅពេលការងារទាំងអស់ជោគជ័យ។
- Rollback នៅពេលកើត Exception មុន Commit។
- ប្រើ `$pdo->inTransaction()` មុនពេល Rollback។
- កត់ត្រាព័ត៌មានលម្អិតនៅ Server Log។
- បង្ហាញសារទូទៅទៅអ្នកប្រើ; កុំបង្ហាញ SQL, File Path ឬ Stack Trace។

បើ Script បញ្ចប់ដោយមិន Commit Transaction ដែលចាប់ផ្ដើមតាម PDO នោះ PDO អាច Rollback Transaction ដែលនៅសល់នៅពេល Connection បិទ។ ទោះយ៉ាងណា កូដគួរដោះស្រាយដោយច្បាស់ មិនគួរពឹងតែការសម្អាតដោយស្វ័យប្រវត្តិទេ។

## ៨. Mini Project៖ ផ្ទេរប្រាក់ក្នុង Demo

1. បង្កើត Tables `accounts` និង `transfers` តាម SQL នៅផ្នែកទី ៣។
2. បញ្ចូលគណនីសាកល្បង៖

```sql
INSERT INTO accounts (owner_name, balance)
VALUES ('Dara', 100.00), ('Sokha', 25.00);
```

3. ដាក់ Function `transferMoney()` នៅក្នុងឯកសារ PHP ដែលភ្ជាប់ `database.php`។
4. សាកល្បងផ្ទេរ 12.50 ពីគណនី 1 ទៅគណនី 2។
5. ពិនិត្យសមតុល្យថ្មី និង Row នៅក្នុង `transfers`។
6. សាកល្បងផ្ទេរចំនួនធំជាងសមតុល្យ។
7. បញ្ជាក់ ថាករណីបរាជ័យមិនបានផ្លាស់ប្ដូរសមតុល្យណាមួយ ហើយមិនបានបង្កើត Transfer Row ទេ។

> ប្រើតែ Database សាកល្បង។ កុំដំណើរការឧទាហរណ៍នេះលើទិន្នន័យពិត។

## ៩. Best Practices

1. **ប្រើ Transaction ពេលប្រតិបត្តិការច្រើនពឹងផ្អែកគ្នា។** កុំប្រើវាជុំវិញ Query មួយដែលមិនចាំបាច់។
2. **ធ្វើ Transaction ឱ្យខ្លី។** កុំរង់ចាំ Network Request ឬ User Input ខណៈ Transaction នៅបើក។
3. **ប្រើ InnoDB ឬ Engine ដែលគាំទ្រ Transaction។**
4. **កុំដាក់ DDL ក្នុង Transaction ដែលរំពឹងថា Rollback បាន។** MySQL អាច Commit ដោយស្វ័យប្រវត្តិពេល DDL។
5. **ដោះស្រាយ Concurrent Updates។** ប្រើ Conditional UPDATE, Constraint ឬ Lock តាមលក្ខណៈការងារ។
6. **ប្រើ DECIMAL សម្រាប់ប្រាក់នៅ Database។** ជៀសវាងការគណនារូបិយប័ណ្ណដោយ Floating Point; ក្នុង PHP អាចទទួលជាចំនួនសេន ឬប្រើ Library ដែលគាំទ្រ Decimal។
7. **កុំលាក់កំហុសដោយគ្មាន Log។** បង្ហាញ Error ដែលអាចធ្វើអ្វីបានទៅអ្នកប្រើ ប៉ុន្តែកត់ត្រាព័ត៌មានបច្ចេកទេសនៅ Server។

## ១០. កំហុសដែលជួបញឹកញាប់

- **ភ្លេច Commit៖** ការផ្លាស់ប្ដូរមិនត្រូវបានបញ្ចប់តាមការរំពឹង។
- **មិន Rollback ពេល Exception៖** អាចទុក Connection ក្នុងស្ថានភាព Transaction មិនទាន់ចប់។
- **Rollback បន្ទាប់ពី Commit រួច៖** មិនអាចត្រឡប់ការផ្លាស់ប្ដូរដែល Commit រួចបានទេ។
- **ប្រើ Table Engine ដែលមិនគាំទ្រ Transaction៖** Rollback អាចមិនមានប្រសិទ្ធភាព។
- **ដាក់ CREATE TABLE ឬ ALTER TABLE ក្នុង Transaction ហើយរំពឹងថា Rollback បាន៖** DDL អាចបង្ក Implicit Commit។
- **ប្រើ Float សម្រាប់ចំនួនប្រាក់៖** អាចមានកំហុសភាពជាក់លាក់។
- **ធ្វើការងារយូរពេកក្នុង Transaction៖** អាចកាន់ Lock និងប៉ះពាល់ដល់ Request ផ្សេង។
- **គិតថា Transaction ជំនួស Validation ឬ Prepared Statements៖** ត្រូវប្រើរួមគ្នា។

## ១១. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Rollback ដោយចេតនា

ចាប់ផ្ដើម Transaction បញ្ចូល Row មួយទៅ Table សាកល្បង រួចហៅ `rollBack()`។ ពិនិត្យថា Row មិននៅសល់ទេ។

### លំហាត់ទី ២៖ បង្កើត Order ជា Transaction

បង្កើត Order និង Order Items ច្រើនក្នុង Transaction តែមួយ។ បោះ Exception ប្រសិនបើ Item ណាមួយមិនមានស្តុក ហើយ Rollback ទាំងអស់។

### លំហាត់ទី ៣៖ ពិនិត្យ Transaction State

ប្រើ `inTransaction()` មុន Rollback ដើម្បីការពារ Exception ថ្មីពេលគ្មាន Transaction សកម្ម។

### លំហាត់ទី ៤៖ Concurrent Stock

សរសេរ UPDATE បន្ថយ Stock តែបើ Stock ធំជាង ឬស្មើ Quantity ដែលត្រូវលក់។ ប្រើ `rowCount()` ដើម្បីដឹងថាបានបន្ថយឬអត់។

### លំហាត់ទី ៥៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង `commit()` និង `rollBack()` ហើយបង្ហាញឧទាហរណ៍មួយដែលគួរប្រើ Transaction។

## ១២. ចម្លើយគំរូ

### ចម្លើយលំហាត់ទី ១

```php
<?php
$pdo->beginTransaction();

$stmt = $pdo->prepare(
    'INSERT INTO audit_demo (message) VALUES (:message)'
);
$stmt->execute(['message' => 'សាកល្បង Rollback']);

$pdo->rollBack();
```

បន្ទាប់មក Query Table ដើម្បីពិនិត្យថា Row មិនត្រូវបានរក្សាទុក។

### ចម្លើយលំហាត់ទី ២

```php
<?php
try {
    $pdo->beginTransaction();

    $order = $pdo->prepare(
        'INSERT INTO orders (customer_id, status)
         VALUES (:customer_id, :status)'
    );
    $order->execute([
        'customer_id' => $customerId,
        'status' => 'pending',
    ]);

    $orderId = (int) $pdo->lastInsertId();

    foreach ($items as $item) {
        $stock = $pdo->prepare(
            'UPDATE products
             SET stock = stock - :quantity
             WHERE id = :product_id AND stock >= :required_quantity'
        );

        $stock->execute([
            'quantity' => $item['quantity'],
            'product_id' => $item['product_id'],
            'required_quantity' => $item['quantity'],
        ]);

        if ($stock->rowCount() !== 1) {
            throw new RuntimeException('ស្តុកមិនគ្រប់គ្រាន់។');
        }

        $line = $pdo->prepare(
            'INSERT INTO order_items (order_id, product_id, quantity)
             VALUES (:order_id, :product_id, :quantity)'
        );

        $line->execute([
            'order_id' => $orderId,
            'product_id' => $item['product_id'],
            'quantity' => $item['quantity'],
        ]);
    }

    $pdo->commit();
} catch (Throwable $e) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    error_log('Order transaction failed: ' . $e->getMessage());
    throw $e;
}
```

### ចម្លើយលំហាត់ទី ៣

```php
<?php
if ($pdo->inTransaction()) {
    $pdo->rollBack();
}
```

### ចម្លើយលំហាត់ទី ៤

```sql
UPDATE products
SET stock = stock - :quantity
WHERE id = :product_id
  AND stock >= :required_quantity;
```

បើ `rowCount()` ជា 1 នោះបានបន្ថយស្តុក។ បើជា 0 នោះ Product មិនមាន ឬស្តុកមិនគ្រប់គ្រាន់។

### ចម្លើយលំហាត់ទី ៥

`commit()` រក្សាទុកការផ្លាស់ប្ដូរទាំងអស់ក្នុង Transaction។ `rollBack()` បោះបង់ការផ្លាស់ប្ដូរដែលមិនទាន់ Commit។ ឧទាហរណ៍ផ្ទេរប្រាក់ត្រូវប្រើ Transaction ព្រោះការដក និងការបញ្ចូលប្រាក់ត្រូវជោគជ័យជាមួយគ្នា។

---

## សេចក្ដីសង្ខេប

- Transaction ធ្វើឱ្យការផ្លាស់ប្ដូរច្រើនជំហានអាច Commit ឬ Rollback ជាក្រុម។
- ប្រើ `beginTransaction()` ដើម្បីចាប់ផ្ដើម, `commit()` ដើម្បីបញ្ចប់ដោយជោគជ័យ, និង `rollBack()` ពេលបរាជ័យ។
- MySQL Tables ត្រូវប្រើ InnoDB ដើម្បីគាំទ្រ Transaction។
- កុំដាក់ DDL ក្នុង Transaction ដែលរំពឹងថានឹង Rollback។
- ប្រើ Conditional UPDATE, Constraints និង Lock ដើម្បីគិតគូរពីការកែប្រែស្របគ្នា។
- Transaction មិនជំនួស Prepared Statements, Validation ឬ Logging ទេ។

## ឯកសារយោងផ្លូវការ

- [PDO Transactions និង Auto-commit](https://www.php.net/manual/en/pdo.transactions.php)
- [PDO::beginTransaction()](https://www.php.net/manual/en/pdo.begintransaction.php)
- [PDO::commit()](https://www.php.net/manual/en/pdo.commit.php)
- [PDO::rollBack()](https://www.php.net/manual/en/pdo.rollback.php)

## មេរៀនបន្ទាប់

អាចបន្តដោយសិក្សាការរៀបចំ CRUD Forms និងការបែងចែកកូដ PHP ទៅជា Files និង Layers ដែលងាយថែទាំ។
