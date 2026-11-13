# មេរៀនទី ១០២៖ Symfony Events, Listeners និង Subscribers

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ពន្យល់ពី Event Dispatcher, Event និង Listener។
- បង្កើត Custom Event ដើម្បីជូនដំណឹងថា Order ត្រូវបានដាក់។
- ប្រើ Event Subscriber ដើម្បីចុះឈ្មោះ listener ជាច្រើន។
- ពិនិត្យ listeners ដែល Symfony បានចុះឈ្មោះ។

## ១. Event System ជាអ្វី?

Event ជាការជូនដំណឹងថាមានអ្វីមួយកើតឡើងក្នុងកម្មវិធី។ ឧទាហរណ៍ បន្ទាប់ពី Order ត្រូវបានបង្កើត អ្នកអាច dispatch event មួយ។ Listener ផ្សេងអាចឆ្លើយតបដោយផ្ញើ email ឬកត់ត្រា log។

Event ជួយឱ្យផ្នែកដែលបង្កើត Order មិនចាំបាច់ដឹងលម្អិតពីគ្រប់សកម្មភាពបន្ថែម។ ប៉ុន្តែកុំប្រើ events ច្រើនហួសហេតុសម្រាប់លំហូរដែលត្រូវការការឆ្លើយតបភ្លាមៗ និងអាចមើលឃើញបានច្បាស់ក្នុង code path។

## ២. បង្កើត Custom Event

បង្កើត \`src/Event/OrderPlacedEvent.php\`៖

\`\`\`php
<?php

namespace App\\Event;

use App\\Entity\\Order;
use Symfony\\Contracts\\EventDispatcher\\Event;

final class OrderPlacedEvent extends Event
{
    public function __construct(
        private Order $order,
    ) {
    }

    public function getOrder(): Order
    {
        return $this->order;
    }
}
\`\`\`

Event ផ្ទុកព័ត៌មានដែល Listener ត្រូវការ។ កុំដាក់ព័ត៌មានលម្អិតដែលមិនចាំបាច់ ឬទិន្នន័យសម្ងាត់ចូលក្នុង event។

## ៣. Dispatch Event

Inject dispatcher ទៅ service ដែលដាក់ Order៖

\`\`\`php
<?php

namespace App\\Service;

use App\\Entity\\Order;
use App\\Event\\OrderPlacedEvent;
use Symfony\\Contracts\\EventDispatcher\\EventDispatcherInterface;

final class OrderPlacer
{
    public function __construct(
        private EventDispatcherInterface $eventDispatcher,
    ) {
    }

    public function orderPlaced(Order $order): void
    {
        // រក្សាទុក Order តាម logic របស់កម្មវិធីជាមុន...

        $this->eventDispatcher->dispatch(new OrderPlacedEvent($order));
    }
}
\`\`\`

\`dispatch()\` បញ្ជូន event ទៅ dispatcher ដែលហៅ subscribers/listeners សម្រាប់ event class នោះ។ ឈ្មោះ method \`orderPlaced()\` នៅទីនេះគ្រាន់តែឧទាហរណ៍; ក្នុងកម្មវិធីពិតដាក់ persistence, transaction និង business rules ឱ្យត្រឹមត្រូវ។

## ៤. បង្កើត Event Subscriber

បង្កើត \`src/EventSubscriber/OrderPlacedSubscriber.php\`៖

\`\`\`php
<?php

namespace App\\EventSubscriber;

use App\\Event\\OrderPlacedEvent;
use Psr\\Log\\LoggerInterface;
use Symfony\\Component\\EventDispatcher\\EventSubscriberInterface;

final class OrderPlacedSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private LoggerInterface $logger,
    ) {
    }

    public static function getSubscribedEvents(): array
    {
        return [
            OrderPlacedEvent::class => 'onOrderPlaced',
        ];
    }

    public function onOrderPlaced(OrderPlacedEvent $event): void
    {
        $order = $event->getOrder();

        $this->logger->info('មាន Order ថ្មីត្រូវបានដាក់។', [
            'order_id' => $order->getId(),
        ]);
    }
}
\`\`\`

Subscriber ប្រកាស event ដែលខ្លួនស្ដាប់ក្នុង \`getSubscribedEvents()\`។ Project Symfony ដែលមាន service autoconfiguration តាមលំនាំដើម ជាទូទៅរកឃើញ \`EventSubscriberInterface\` ដោយស្វ័យប្រវត្តិ។ បើ service config ត្រូវបានប្ដូរ សូមពិនិត្យការចុះឈ្មោះ subscriber របស់ project។

## ៥. Listener និង Subscriber ខុសគ្នាដូចម្ដេច?

- **Listener** ជាធម្មតាត្រូវបានចុះឈ្មោះជាមួយឈ្មោះ event និង method/timing ក្នុង service configuration ឬ PHP attribute \`#[AsEventListener]\`។
- **Subscriber** បញ្ជាក់ក្នុង class ខ្លួនឯងថាស្ដាប់ event ណាខ្លះ តាម \`getSubscribedEvents()\`។

ប្រើ Listener សម្រាប់ការឆ្លើយតបតែមួយ ឬ wiring តូច។ ប្រើ Subscriber ពេល class ត្រូវស្ដាប់ event ច្រើន ឬចង់ឱ្យការចុះឈ្មោះនៅជិត logic។ Priority ខ្ពស់ដំណើរការមុន priority ទាប។

## ៦. ស្វែងរក និង Debug Events

បង្ហាញ event listeners ដែលបានចុះឈ្មោះ៖

\`\`\`bash
php bin/console debug:event-dispatcher
\`\`\`

សម្រាប់ event ជាក់លាក់ អាចបន្ថែមឈ្មោះ event តាមជំនួយរបស់ command៖

\`\`\`bash
php bin/console debug:event-dispatcher App\\Event\\OrderPlacedEvent
\`\`\`

ពិនិត្យ output ព្រោះ service discovery និង version អាចធ្វើឱ្យ command syntax មានភាពខុសបន្តិច។ Event subscriber មិនដំណើរការ អាចមកពី event class ដែល dispatch មិនត្រូវនឹងឈ្មោះក្នុង \`getSubscribedEvents()\` ឬ class មិនបានចុះឈ្មោះជា service។

## កំហុសដែលជួបញឹកញាប់

- **Dispatch event មុនទិន្នន័យបានរក្សាទុក**៖ listener អាចទទួល object ដែលមិនទាន់មាន ID ឬ persistence។
- **Subscriber មិនត្រូវបានចុះឈ្មោះ**៖ ពិនិត្យ service autoconfiguration ឬ tag។
- **Event name/type mismatch**៖ ប្រើ class ដូចគ្នានៅកន្លែង dispatch និង subscribe។
- **ដាក់ការងារយូរក្នុង synchronous listener**៖ វាធ្វើឱ្យ request យឺត; ពិចារណា queue សម្រាប់ការងារមិនចាំបាច់រង់ចាំ។
- **កែ Response នៅ event ខុស**៖ kernel event នីមួយៗមានគោលបំណង និងប្រភេទ event ផ្សេងគ្នា; អាន docs មុន register listener។

## លំហាត់

1. បង្កើត \`OrderPlacedEvent\` ដែលផ្ទុក Order object។
2. បង្កើត Subscriber ដែលកត់ត្រា Order ID ក្នុង log។
3. បន្ថែម listener មួយទៀតសម្រាប់ផ្ញើ confirmation email តាម Mailer service។
4. ប្រើ \`debug:event-dispatcher\` ដើម្បីបញ្ជាក់ថា subscriber ត្រូវបានចុះឈ្មោះ។
5. ពន្យល់ថាការផ្ញើ email ដែលយូរគួរធ្វើក្នុង request ដោយផ្ទាល់ ឬតាម queue ហេតុអ្វី។

## សំណួរត្រួតពិនិត្យ

1. តើ Event Dispatcher ធ្វើអ្វី?
2. តើ Custom Event គួរផ្ទុកព័ត៌មានអ្វី?
3. តើ Listener ខុសពី Subscriber ដូចម្ដេច?
4. តើ Priority ប៉ះពាល់ដល់លំដាប់ដំណើរការយ៉ាងដូចម្ដេច?
5. តើពេលណាគួរផ្ទេរការងាររបស់ listener ទៅ background queue?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Events and Event Listeners](https://symfony.com/doc/7.4/event_dispatcher.html)
- [Symfony 7.4 Messenger](https://symfony.com/doc/7.4/messenger.html)
