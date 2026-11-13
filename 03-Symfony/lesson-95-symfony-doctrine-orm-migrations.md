# មេរៀនទី ៩៥៖ Symfony Doctrine ORM និង Database Migrations

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ស្គាល់តួនាទី Entity និង Doctrine ORM ក្នុង Symfony។
- កំណត់ការតភ្ជាប់ Database តាម `DATABASE_URL`។
- បង្កើត Entity ដោយ MakerBundle។
- បង្កើត និងអនុវត្ត Migration ដោយសុវត្ថិភាព។

## ១. Doctrine ORM ជាអ្វី?

Doctrine ORM ជួយបម្លែង PHP objects ទៅជាកំណត់ត្រាក្នុង relational database និងបម្លែងត្រឡប់មកវិញ។ Class ដែលតំណាងឱ្យទិន្នន័យក្នុងកម្មវិធីហៅថា **Entity**។ Property របស់ Entity ត្រូវបាន map ទៅ column ហើយ entity instance មួយតំណាងឱ្យ row មួយ។

ឧទាហរណ៍៖ `Product` Entity អាចតំណាងឱ្យតារាង `product` ដែលមាន column `name` និង `price`។

## ២. ដំឡើង Doctrine និងរៀបចំ Database

ក្នុង Symfony project ដែលប្រើ Symfony Flex៖

```bash
composer require symfony/orm-pack
composer require --dev symfony/maker-bundle
```

កំណត់ការតភ្ជាប់ក្នុង `.env.local` សម្រាប់ម៉ាស៊ីនរបស់អ្នក។ ឧទាហរណ៍ MySQL៖

```dotenv
DATABASE_URL="mysql://app_user:password@127.0.0.1:3306/app_db?serverVersion=8.0.37"
```

កែ username, password, database និង `serverVersion` ឱ្យត្រូវនឹង database server របស់អ្នក។ `.env.local` មិនគួរត្រូវ commit ទៅ Git ព្រោះវាអាចផ្ទុក credentials។ បើ credentials មានតួអក្សរពិសេសក្នុង URL ត្រូវ URL-encode តាមការណែនាំរបស់ Doctrine/Symfony។

បង្កើត database ប្រសិនបើមិនទាន់មាន៖

```bash
php bin/console doctrine:database:create
```

## ៣. បង្កើត Entity

ប្រើ MakerBundle ដើម្បីបង្កើត Entity និង properties៖

```bash
php bin/console make:entity Product
```

បញ្ចូលព័ត៌មាន៖

- Property: `name`, type: `string`, length: `160`, nullable: `no`
- Property: `priceInCents`, type: `integer`, nullable: `no`
- ចុច Enter ទទេដើម្បីបញ្ចប់

ឧទាហរណ៍នៃ Entity ដែលបានបង្កើត៖

```php
<?php

namespace App\Entity;

use App\Repository\ProductRepository;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ProductRepository::class)]
class Product
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 160)]
    private ?string $name = null;

    #[ORM\Column]
    private ?int $priceInCents = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getName(): ?string
    {
        return $this->name;
    }

    public function setName(string $name): static
    {
        $this->name = $name;

        return $this;
    }

    public function getPriceInCents(): ?int
    {
        return $this->priceInCents;
    }

    public function setPriceInCents(int $priceInCents): static
    {
        $this->priceInCents = $priceInCents;

        return $this;
    }
}
```

`#[ORM\Entity]` ប្រាប់ Doctrine ថា class នេះអាចរក្សាទុកក្នុង database។ `#[ORM\Column]` កំណត់ column។ `id` ជា primary key ដែល Doctrine បង្កើនតម្លៃដោយស្វ័យប្រវត្តិ។ ការរក្សាតម្លៃជាចំនួនតូចបំផុត ដូចជា សេន ជួយជៀសវាងបញ្ហាបង្គត់លេខទសភាគសម្រាប់ប្រាក់។

## ៤. បង្កើត និងពិនិត្យ Migration

Entity គ្រាន់តែពណ៌នាអំពី schema ដែលកម្មវិធីចង់បាន។ វាមិនបង្កើត ឬកែតារាងក្នុង database ដោយខ្លួនឯងទេ។ បង្កើត migration៖

```bash
php bin/console make:migration
```

បើកមើល migration ដែលបានបង្កើតនៅក្នុង `migrations/`។ ពិនិត្យ SQL នៅក្នុង `up()` និង `down()` ថាតើវាធ្វើការផ្លាស់ប្ដូរត្រឹមត្រូវ ហើយមិនលុបទិន្នន័យដែលត្រូវរក្សាទុក។

អាចពិនិត្យភាពខុសគ្នារវាង Entity mapping និង database ដោយ៖

```bash
php bin/console doctrine:schema:validate
```

បន្ទាប់ពីពិនិត្យរួច អនុវត្ត migration៖

```bash
php bin/console doctrine:migrations:migrate
```

ពេលប្រព័ន្ធសួរបញ្ជាក់ សូមពិនិត្យឈ្មោះ database និង SQL មុនឆ្លើយយល់ព្រម។ Migration គឺជាឯកសារកំណត់ត្រាផ្លាស់ប្ដូរ schema ដែលអាចដាក់ជាមួយ source code ដើម្បីឱ្យ environment ផ្សេងៗធ្វើបច្ចុប្បន្នភាពតាមលំដាប់ដូចគ្នា។

## ៥. កែប្រែ Schema នៅពេលក្រោយ

បើចង់បន្ថែម property ថ្មី ដូចជា `description` អាចរត់៖

```bash
php bin/console make:entity Product
```

បញ្ចូល property ថ្មី រួចបង្កើត migration ថ្មី៖

```bash
php bin/console make:migration
```

ពិនិត្យ migration មុននឹងអនុវត្ត៖

```bash
php bin/console doctrine:migrations:migrate
```

កុំកែ migration ចាស់ដែលបានដំណើរការរួចលើ environment រួម ឬ production។ បង្កើត migration ថ្មីដើម្បីបង្ហាញការផ្លាស់ប្ដូរបន្ថែម។

## កំហុសដែលជួបញឹកញាប់

- **Connection refused**៖ ពិនិត្យថា database server កំពុងដំណើរការ ហើយ host/port ត្រឹមត្រូវ។
- **Unknown database**៖ បង្កើត database ឬកែឈ្មោះនៅក្នុង `DATABASE_URL`។
- **Server version ខុស**៖ កំណត់ `serverVersion` ឱ្យស្របនឹង DBMS ដែលបានដំឡើង។
- **Entity និង schema មិនត្រូវគ្នា**៖ រត់ `make:migration`, ពិនិត្យឯកសារ និងអនុវត្តវា។
- **បញ្ចូល password ទៅ Git**៖ រក្សាទុក credentials ក្នុង `.env.local` ឬ secret manager របស់ deployment ហើយកុំ commit ឯកសារសម្ងាត់។

## លំហាត់

1. បន្ថែម property `description` ប្រភេទ `text` ទៅ `Product` ហើយបង្កើត migration។
2. ពន្យល់ពីភាពខុសគ្នារវាង Entity និង Migration។
3. ប្ដូរ `priceInCents` ទៅជា `price` ប្រភេទ integer (សេន) និងពិនិត្យ migration ដែល MakerBundle បង្កើត។
4. សាកល្បង `doctrine:schema:validate` មុន និងក្រោយអនុវត្ត migration។

## សំណួរត្រួតពិនិត្យ

1. តើ Entity តំណាងឱ្យអ្វីក្នុង database?
2. តើ `DATABASE_URL` ប្រើសម្រាប់អ្វី?
3. ហេតុអ្វីត្រូវពិនិត្យ migration មុនរត់លើ database?
4. តើហេតុអ្វីមិនគួរកែ migration ចាស់ដែលបានប្រើរួច?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4: Databases and the Doctrine ORM](https://symfony.com/doc/7.4/doctrine.html)
- [Doctrine Migrations Bundle](https://symfony.com/bundles/DoctrineMigrationsBundle/current/index.html)
