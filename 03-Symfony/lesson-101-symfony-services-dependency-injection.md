# មេរៀនទី ១០១៖ Symfony Services និង Dependency Injection Container

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បែងចែក business logic ទៅក្នុង Service class។
- ប្រើ constructor injection ដើម្បីទទួល dependencies។
- យល់ពី Service Container និង Autowiring។
- ពិនិត្យ services ដែល Symfony ស្គាល់។

## ១. Service និង Container

**Service** គឺជា PHP object ដែលទទួលខុសត្រូវលើការងារជាក់លាក់ ដូចជា គណនាតម្លៃបញ្ជាទិញ ឬរៀបចំ email។ **Dependency Injection (DI)** គឺការផ្ដល់ object ដែល class ត្រូវការពីខាងក្រៅ ជំនួសឱ្យ class បង្កើតវាដោយខ្លួនឯង។

**Service Container** របស់ Symfony រក្សា recipe សម្រាប់បង្កើត និងភ្ជាប់ services។ វាប្រើ constructor type hints ដើម្បីដឹងថា service មួយត្រូវការ dependencies អ្វីខ្លះ។

## ២. បង្កើត Service

បង្កើត `src/Service/PriceCalculator.php`៖

```php
<?php

namespace App\Service;

class PriceCalculator
{
    /**
     * តម្លៃជា cents ដើម្បីជៀសវាងការគណនាលេខទសភាគ។
     *
     * @param list<int> $pricesInCents
     */
    public function total(array $pricesInCents): int
    {
        return array_sum($pricesInCents);
    }
}
```

Class នេះធ្វើការគណនាតែមួយប្រភេទ។ Controller មិនចាំបាច់ស្គាល់ថាតើការគណនាត្រូវបានធ្វើដូចម្ដេចទេ។

## ៣. ចាក់ Service ទៅ Controller

Symfony អាចបញ្ជូន service តាម type hint របស់ method argument៖

```php
<?php

namespace App\Controller;

use App\Service\PriceCalculator;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class OrderController
{
    #[Route('/orders/total', name: 'order_total', methods: ['GET'])]
    public function total(PriceCalculator $calculator): Response
    {
        $totalInCents = $calculator->total([1200, 350, 450]);

        return new Response('សរុប: '.number_format($totalInCents / 100, 2));
    }
}
```

Controller មិនបង្កើត `PriceCalculator` ដោយប្រើ `new` ទេ។ Container ផ្ដល់វាឱ្យ ព្រោះ class ស្ថិតក្នុង namespace ដែល project កំណត់ឱ្យ auto-discover។

## ៤. ចាក់ Dependency តាម Constructor

បើ Service មួយត្រូវការ Service ផ្សេង សូមកំណត់តាម constructor៖

```php
<?php

namespace App\Service;

use Psr\Log\LoggerInterface;

class OrderProcessor
{
    public function __construct(
        private PriceCalculator $calculator,
        private LoggerInterface $logger,
    ) {
    }

    /** @param list<int> $pricesInCents */
    public function calculateTotal(array $pricesInCents): int
    {
        $total = $this->calculator->total($pricesInCents);
        $this->logger->info('បានគណនាតម្លៃសរុប។');

        return $total;
    }
}
```

ឥឡូវ Container អាចបង្កើត `OrderProcessor`, `PriceCalculator` និង logger តាម dependency graph។ កុំសរសេរ business logic ទាំងមូលក្នុង Controller ព្រោះវាធ្វើឱ្យពិបាកប្រើឡើងវិញ និងសាកល្បង។

## ៥. Autowiring និង Autoconfiguration

Project Symfony ថ្មីជាទូទៅមាន `config/services.yaml` ដែលប្រហាក់ប្រហែល៖

```yaml
services:
    _defaults:
        autowire: true
        autoconfigure: true

    App\:
        resource: '../src/'
```

- `autowire: true` អនុញ្ញាតឱ្យ Container ជ្រើស service តាម type hint។
- `autoconfigure: true` អនុញ្ញាតឱ្យ Symfony ដាក់ tags ទូទៅដោយស្វ័យប្រវត្តិ ដូចជា command ឬ event subscriber។
- `resource` ធ្វើឱ្យ classes ក្នុង `src/` អាចប្រើជាសេវាកម្ម។

ប្រសិនបើ constructor ត្រូវការ scalar ដូចជា string API URL, type hint មិនគ្រប់គ្រាន់ឱ្យ Container ដឹងតម្លៃទេ។ ប្រើ configuration parameter ឬ bind value តាម project config ហើយកុំ hard-code secrets ក្នុង PHP។

## ៦. ពិនិត្យ Container

មើល interfaces/services ដែលអាច autowire បាន៖

```bash
php bin/console debug:autowiring
```

ស្វែងរក service ដែលបានចុះឈ្មោះ៖

```bash
php bin/console debug:container
```

មើល config ជាក់លាក់៖

```bash
php bin/console debug:container App\Service\PriceCalculator
```

## កំហុសដែលជួបញឹកញាប់

- **Cannot autowire service**៖ ពិនិត្យ type hint, namespace និង service configuration។
- **Interface មួយមាន implementation ច្រើន**៖ Container មិនដឹងថាត្រូវជ្រើសមួយណា; កំណត់ alias។
- **Inject scalar តាម type hint ដោយរំពឹងថា container ដឹងតម្លៃ**៖ កំណត់ parameter/bind ឱ្យច្បាស់។
- **Class បង្កើត dependencies ដោយ `new`**៖ វាភ្ជាប់ class តឹងពេក; inject dependencies តាម constructor។
- **ចាក់ Container ទាំងមូលទៅ Service**៖ ចាក់ dependency ជាក់លាក់ដែលត្រូវការប៉ុណ្ណោះ។

## លំហាត់

1. បន្ថែម method `average()` ទៅ `PriceCalculator` ដោយការពារករណី array ទទេ។
2. បង្កើត `OrderProcessor` ដែល inject `PriceCalculator` និង `LoggerInterface`។
3. ប្រើ CLI ដើម្បីពិនិត្យថា service ទាំងពីរត្រូវបានរកឃើញ។
4. បង្កើត interface `TaxCalculatorInterface` និង implementation មួយ ហើយកំណត់ alias ក្នុង Container។
5. សរសេរពន្យល់ថា DI ធ្វើឱ្យការសាកល្បង class ងាយជាងដូចម្ដេច។

## សំណួរត្រួតពិនិត្យ

1. តើ Service Container មានតួនាទីអ្វី?
2. តើ Autowiring ប្រើព័ត៌មានអ្វីដើម្បីបញ្ជូន service?
3. តើហេតុអ្វីគួរចាក់ dependency តាម constructor?
4. តើពេលណាត្រូវកំណត់ alias?
5. តើហេតុអ្វីមិនគួរ inject Container ទាំងមូល?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Service Container](https://symfony.com/doc/7.4/service_container.html)
- [Symfony 7.4 Dependency Injection](https://symfony.com/doc/7.4/service_container/autowiring.html)
