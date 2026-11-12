
# មេរៀនទី ២៥៖ Dependency Injection និង Design Patterns

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ថា Dependency ជាអ្វី និងហេតុអ្វីត្រូវ Inject វា។
- ប្រើ Constructor Injection ដើម្បីផ្ដល់ Dependencies ឱ្យ Class។
- ពឹងផ្អែកលើ Interface ជំនួស Concrete Class។
- ប្រើ Strategy Pattern ដើម្បីប្ដូរឥរិយាបថតាម Implementation។
- យល់ពីតួនាទីរបស់ Factory និង Composition Root។
- ជៀសវាង Service Locator ក្នុង Domain/Application Classes។
- សរសេរកូដដែលងាយ Test និងប្ដូរ Implementation។

> តម្រូវការ៖ PHP 8.0 ឡើងទៅ និងចំណេះដឹងអំពី OOP, Interfaces និង PHPUnit ពីមេរៀនមុនៗ។

---

## ១. Dependency ជាអ្វី?

Dependency គឺជា Object ឬ Service ដែល Class មួយត្រូវការដើម្បីបំពេញការងារ។

ឧទាហរណ៍ OrderService អាចត្រូវការ៖

- OrderRepository ដើម្បីរក្សាទុក Order។
- PaymentGateway ដើម្បីទទួលការទូទាត់។
- Logger ដើម្បីកត់ត្រាកំហុស។

បើ OrderService បង្កើត Concrete Class ទាំងនេះដោយខ្លួនឯង វាភ្ជាប់ខ្លាំងទៅនឹង Implementation ជាក់លាក់។ ការសាកល្បង និងប្ដូរទៅសេវាផ្សេងនឹងពិបាក។

## ២. Dependency Injection ជាអ្វី?

Dependency Injection (DI) គឺការផ្ដល់ Object ដែល Class ត្រូវការពីខាងក្រៅ ជំនួសឱ្យឱ្យ Class បង្កើត ឬស្វែងរកវាដោយខ្លួនឯង។

កូដដែលភ្ជាប់ខ្លាំង៖

```php
<?php
class OrderService
{
    public function placeOrder(array $order): void
    {
        $repository = new MySqlOrderRepository();
        $repository->save($order);
    }
}
```

OrderService មិនអាចងាយប្ដូរ Repository ឬប្រើ Test Double បានទេ ព្រោះវាបង្កើត MySqlOrderRepository ដោយផ្ទាល់។

កូដដែលទទួល Dependency ពីខាងក្រៅ៖

```php
<?php
class OrderService
{
    public function __construct(
        private OrderRepository $repository
    ) {
    }

    public function placeOrder(array $order): void
    {
        $this->repository->save($order);
    }
}
```

ឥឡូវនេះអ្នកអាចផ្ដល់ Repository ពិត ឬ Fake Repository ក្នុង Test បាន។

## ៣. Constructor Injection និង Interface

បង្កើត Interface សម្រាប់កិច្ចសន្យា៖

```php
<?php
interface OrderRepository
{
    public function save(array $order): void;
}
```

បង្កើត Implementation សម្រាប់ MySQL៖

```php
<?php
final class PdoOrderRepository implements OrderRepository
{
    public function __construct(
        private PDO $pdo
    ) {
    }

    public function save(array $order): void
    {
        $stmt = $this->pdo->prepare(
            'INSERT INTO orders (customer_id, total_cents)
             VALUES (:customer_id, :total_cents)'
        );

        $stmt->execute([
            'customer_id' => $order['customer_id'],
            'total_cents' => $order['total_cents'],
        ]);
    }
}
```

OrderService ពឹងផ្អែកលើ OrderRepository Interface មិនមែន PdoOrderRepository៖

```php
<?php
final class OrderService
{
    public function __construct(
        private OrderRepository $repository
    ) {
    }

    public function placeOrder(array $order): void
    {
        if (($order['total_cents'] ?? 0) < 1) {
            throw new InvalidArgumentException('ចំនួនសរុបមិនត្រឹមត្រូវ។');
        }

        $this->repository->save($order);
    }
}
```

ចំណាំ៖ ឧទាហរណ៍នេះផ្ដោតលើ DI។ កម្មវិធីពិតគួរប្រើ Domain Object ឬ DTO មាន Type ជាក់លាក់ ជំនួស Array ដែលមាន Shape មិនច្បាស់។

## ៤. Composition Root

Composition Root ជាចំណុចមួយនៅពេលចាប់ផ្ដើមកម្មវិធី ដែលបង្កើត និងភ្ជាប់ Object ទាំងអស់ជាមួយគ្នា។

```php
<?php
require dirname(__DIR__) . '/vendor/autoload.php';
require dirname(__DIR__) . '/config/database.php';

$repository = new PdoOrderRepository($pdo);
$orderService = new OrderService($repository);

// Controller ឬ Route អាចប្រើ $orderService បន្ត។
```

Object ដែលបង្កើតនៅចំណុចនេះត្រូវបាន Inject ទៅ Class ដែលត្រូវការវា។ កុំឱ្យ Domain Service ចាប់ផ្ដើម Database Connection ឬស្វែងរក Object ពី Global State ដោយផ្ទាល់។

## ៥. Strategy Pattern

Strategy Pattern ដាក់ Algorithm ឬឥរិយាបថដែលអាចប្ដូរបាននៅក្នុង Implementation ផ្សេងៗ ដែលអនុវត្ត Interface ដូចគ្នា។

ឧទាហរណ៍ ប្រព័ន្ធអាចបង់ប្រាក់ដោយ Card ឬ Bank Transfer៖

```php
<?php
interface PaymentMethod
{
    public function pay(int $amountInCents): string;
}

final class CardPayment implements PaymentMethod
{
    public function pay(int $amountInCents): string
    {
        return 'បង់ដោយ Card ចំនួន ' . $amountInCents . ' សេន';
    }
}

final class BankTransferPayment implements PaymentMethod
{
    public function pay(int $amountInCents): string
    {
        return 'បង់តាម Bank Transfer ចំនួន ' . $amountInCents . ' សេន';
    }
}
```

CheckoutService មិនចាំបាច់ដឹងថា Payment Method មួយណាត្រូវបានប្រើ៖

```php
<?php
final class CheckoutService
{
    public function __construct(
        private PaymentMethod $paymentMethod
    ) {
    }

    public function checkout(int $amountInCents): string
    {
        if ($amountInCents < 1) {
            throw new InvalidArgumentException('ចំនួនបង់ត្រូវធំជាងសូន្យ។');
        }

        return $this->paymentMethod->pay($amountInCents);
    }
}

$checkout = new CheckoutService(new CardPayment());
echo $checkout->checkout(2500);
```

ប្ដូរ Strategy ដោយ Inject Implementation ផ្សេង៖

```php
$checkout = new CheckoutService(new BankTransferPayment());
```

សេវាខាងក្រៅពិតៗត្រូវគ្រប់គ្រង Credentials, Network Error, Idempotency និងការផ្ទៀងផ្ទាត់ប្រតិបត្តិការ។ ឧទាហរណ៍ខាងលើមិនភ្ជាប់ទៅ Payment Provider ពិតទេ។

## ៦. Factory Pattern

Factory ទទួលខុសត្រូវបង្កើត Object មួយ ឬច្រើន តាមលក្ខខណ្ឌដែលបានកំណត់។ វាជួយដាក់ Logic សម្រេចថាតើ Concrete Class មួយណាត្រូវបង្កើតនៅកន្លែងតែមួយ។

```php
<?php
final class PaymentMethodFactory
{
    public function __construct(
        private CardGateway $cardGateway,
        private BankGateway $bankGateway
    ) {
    }

    public function create(string $method): PaymentMethod
    {
        return match ($method) {
            'card' => new CardPayment($this->cardGateway),
            'bank' => new BankTransferPayment($this->bankGateway),
            default => throw new InvalidArgumentException(
                'Payment Method មិនត្រូវបានគាំទ្រ។'
            ),
        };
    }
}
```

កុំទទួលឈ្មោះ Class ពីអ្នកប្រើ ហើយបង្កើត Object តាមឈ្មោះនោះដោយផ្ទាល់។ ប្រើ Allowlist ដូចជា match ខាងលើ ដើម្បីកំណត់តម្លៃដែលអាចជ្រើសបាន។

Factory សមស្របពេលការបង្កើត Object មានលក្ខខណ្ឌ ឬត្រូវភ្ជាប់ Dependencies ច្រើន។ កុំបង្កើត Factory សម្រាប់ Class សាមញ្ញដែល new មួយបន្ទាត់គ្រប់គ្រាន់។

## ៧. Test Dependency ដោយ Fake Implementation

DI ធ្វើឱ្យអាចសាកល្បង Service ដោយមិនភ្ជាប់ Database៖

```php
<?php
final class InMemoryOrderRepository implements OrderRepository
{
    public array $savedOrders = [];

    public function save(array $order): void
    {
        $this->savedOrders[] = $order;
    }
}
```

Test អាចផ្ដល់ Fake Repository ទៅ OrderService៖

```php
<?php
use PHPUnit\Framework\TestCase;

final class OrderServiceTest extends TestCase
{
    public function testSavesValidOrder(): void
    {
        $repository = new InMemoryOrderRepository();
        $service = new OrderService($repository);

        $order = [
            'customer_id' => 10,
            'total_cents' => 2500,
        ];

        $service->placeOrder($order);

        self::assertSame([$order], $repository->savedOrders);
    }
}
```

Test នេះពិនិត្យ Logic របស់ OrderService ដោយមិនត្រូវការ MySQL Server។ Integration Test ផ្សេងអាចពិនិត្យ PdoOrderRepository ជាមួយ Test Database។

## ៨. DI Container និង Service Locator

DI Container អាចបង្កើត និងភ្ជាប់ Object នៅក្នុងកម្មវិធីធំៗ។ វាអាចជួយសម្រួលការភ្ជាប់ Dependency ប៉ុន្តែ Class មិនគួរទទួល Container ហើយយក Object ដោយខ្លួនឯងទេ។

រចនាបថដែលគួរជៀសវាង (Service Locator)៖

```php
<?php
final class OrderService
{
    public function __construct(
        private Psr\Container\ContainerInterface $container
    ) {
    }

    public function placeOrder(array $order): void
    {
        $repository = $this->container->get('order_repository');
        $repository->save($order);
    }
}
```

OrderService លាក់ Dependencies ដែលវាត្រូវការនៅក្នុង Method។ អ្នកអាន Constructor មិនដឹងថាវាត្រូវការ Repository អ្វីទេ ហើយ Test ក៏ពិបាករៀបចំ។

ប្រើ Constructor Injection ដោយបញ្ជាក់ Dependency ឱ្យច្បាស់៖

```php
<?php
final class OrderService
{
    public function __construct(
        private OrderRepository $repository
    ) {
    }
}
```

Container អាចប្រើនៅ Composition Root ដើម្បីផ្ដល់ OrderRepository ទៅ OrderService។ PSR-11 កំណត់ Interface សម្រាប់អាន Entries ពី Container ប៉ុន្តែណែនាំកុំឱ្យ Pass Container ទៅ Object ដើម្បីឱ្យវាទាញ Dependencies ដោយខ្លួនឯង។

## ៩. Mini Project៖ ប្ដូរ Notification Channel

កំណត់ Interface និង Implementations៖

```php
<?php
interface MessageChannel
{
    public function send(string $recipient, string $message): bool;
}

final class EmailChannel implements MessageChannel
{
    public function send(string $recipient, string $message): bool
    {
        // ជំនួសដោយ Email Provider ក្នុងកម្មវិធីពិត។
        return $recipient !== '' && $message !== '';
    }
}

final class SmsChannel implements MessageChannel
{
    public function send(string $recipient, string $message): bool
    {
        // ជំនួសដោយ SMS Provider ក្នុងកម្មវិធីពិត។
        return $recipient !== '' && $message !== '';
    }
}
```

Service ទទួល Channel តាម Constructor៖

```php
<?php
final class NotificationService
{
    public function __construct(
        private MessageChannel $channel
    ) {
    }

    public function notify(string $recipient, string $message): bool
    {
        if (trim($recipient) === '' || trim($message) === '') {
            throw new InvalidArgumentException(
                'អ្នកទទួល និងសារមិនអាចទទេបានទេ។'
            );
        }

        return $this->channel->send($recipient, $message);
    }
}
```

បញ្ចូល Dependencies នៅ Entry Point៖

```php
<?php
$emailNotifications = new NotificationService(new EmailChannel());
$smsNotifications = new NotificationService(new SmsChannel());

$emailNotifications->notify(
    'student@example.com',
    'ចុះឈ្មោះរួចរាល់'
);

$smsNotifications->notify(
    '+85512345678',
    'មានការជូនដំណឹងថ្មី'
);
```

## ១០. Best Practices

1. ចាក់ Dependencies តាម Constructor ដើម្បីឱ្យតម្រូវការរបស់ Class មើលឃើញច្បាស់។
2. ពឹងផ្អែកលើ Interface ពេលត្រូវប្ដូរ Implementation ឬ Test វាដាច់ដោយឡែក។
3. ដាក់ Object Wiring នៅ Composition Root ឬ Framework Configuration។
4. ប្រើ Strategy ពេលមាន Algorithm/Behavior ច្រើនដែលបំពេញកិច្ចសន្យាដូចគ្នា។
5. ប្រើ Factory ពេលការបង្កើត Object មាន Logic ឬជម្រើសច្រើន។
6. កុំប្រើ Pattern ដោយសារតែចង់មាន Class ច្រើន; Pattern គួរដោះស្រាយបញ្ហាជាក់ស្តែង។
7. កុំ Pass Container ទៅ Domain Service ដើម្បីឱ្យវាស្វែងរក Dependency ខ្លួនឯង។
8. Test Service ដោយ Fake ឬ Stub ដើម្បីមិនពឹងលើ Network និង Database ពិត។
9. កុំទទួល Concrete Class Name ពី Request ដើម្បីបង្កើត Object។
10. រក្សាការទទួលខុសត្រូវរបស់ Class ឱ្យច្បាស់ និងមានតិចតួចតាមសមស្រប។

## ១១. កំហុសដែលជួបញឹកញាប់

- Class បង្កើត Dependency ខាងក្នុងដោយ new ទាំងអស់។
- Constructor ទទួល Container តែមួយ ហើយ Dependency លាក់ក្នុង Method។
- ប្រើ Interface សម្រាប់ Class មួយដោយគ្មានហេតុផលប្ដូរ ឬ Test។
- ប្រើ Inheritance ដើម្បីចែករំលែក Code ទោះទំនាក់ទំនងមិនមែន subtype។
- Factory ធំមួយដែលស្គាល់ និងបង្កើតគ្រប់ Class ក្នុងកម្មវិធី។
- Strategy ដែលមាន Implementation តែមួយ និងគ្មានតម្រូវការប្ដូរ។
- Fake Test Implementation ដែលមាន Logic ស្មុគស្មាញ ឬខុសពី Real Implementation។
- បង្កើត Object ពី Input ដែលមិនបាន Validate។
- គិតថា DI Container ជំនួសការរចនា Dependencies ឱ្យច្បាស់។

## ១២. លំហាត់អនុវត្ត

### លំហាត់ទី ១៖ Repository Injection

បង្កើត Interface ProductRepository និង Class ProductService ដែលទទួល Repository តាម Constructor។

### លំហាត់ទី ២៖ Fake Repository

បង្កើត InMemoryProductRepository ដែលរក្សាទុក Product ក្នុង Array ហើយប្រើវាក្នុង Test របស់ ProductService។

### លំហាត់ទី ៣៖ Strategy

បង្កើត DiscountStrategy Interface និង Implementations សម្រាប់គ្មានការបញ្ចុះតម្លៃ និងបញ្ចុះ 10%។ Inject Strategy ទៅ InvoiceCalculator។

### លំហាត់ទី ៤៖ Factory

បង្កើត Factory ដែលទទួលប្រភេទ Discount ជា String តែជ្រើស Implementation ពី Allowlist។ បដិសេធតម្លៃមិនស្គាល់។

### លំហាត់ទី ៥៖ Service Locator

ពន្យល់ថាហេតុអ្វី OrderService ដែលទទួល Container ហើយហៅ get() ក្នុង Method ពិបាកសាកល្បងជាង Constructor Injection។

### លំហាត់ទី ៦៖ សំណួរយល់ដឹង

ពន្យល់ភាពខុសគ្នារវាង Dependency Injection, Dependency Injection Container, Factory និង Strategy Pattern។

## ១៣. ចម្លើយសម្រាប់ពិនិត្យខ្លួនឯង

- Dependency Injection គឺការផ្ដល់ Object ដែល Class ត្រូវការពីខាងក្រៅ។
- DI Container ជា Tool ដែលអាចបង្កើត និងភ្ជាប់ Dependencies។
- Factory បង្កើត Object តាមលក្ខខណ្ឌ ឬ Configuration។
- Strategy ផ្ដល់ Algorithms/Behaviors ច្រើនដែលអាចប្ដូរគ្នាតាម Interface។
- Constructor Injection ធ្វើឱ្យ Dependencies ដែល Class ត្រូវការមើលឃើញច្បាស់។
- Service Locator លាក់ការទាញ Dependency នៅក្នុង Class ហើយធ្វើឱ្យតម្រូវការមិនច្បាស់។

---

## សេចក្ដីសង្ខេប

- Dependency ជា Object ឬ Service ដែល Class ត្រូវការ។
- Dependency Injection ផ្ដល់វាពីខាងក្រៅ ជាទូទៅតាម Constructor។
- Interface កាត់បន្ថយការភ្ជាប់ទៅ Concrete Class ហើយជួយ Test។
- Strategy ប្ដូរ Behavior; Factory សម្រេចការបង្កើត Object។
- Composition Root ជាកន្លែងភ្ជាប់ Objects សម្រាប់ចាប់ផ្ដើមកម្មវិធី។
- DI Container គួរប្រើនៅកម្រិត Composition Root; ជៀសវាង Service Locator ក្នុង Domain Classes។

## ឯកសារយោង

- [PSR-11 Container Interface](https://www.php-fig.org/psr/psr-11/)
- [PSR-11 Meta Document និង Service Locator](https://www.php-fig.org/psr/psr-11/meta/)

## មេរៀនបន្ទាប់

អាចបន្តដោយរៀន API Design និងការបង្កើត JSON REST API ដោយប្រើ PHP, HTTP Status Codes និង Request Validation។
