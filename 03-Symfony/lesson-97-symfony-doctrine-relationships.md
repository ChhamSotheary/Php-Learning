# មេរៀនទី ៩៧៖ Symfony Doctrine Relationships និង Associations

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ជ្រើសរើសប្រភេទទំនាក់ទំនង Entity ដែលសមស្រប។
- បង្កើតទំនាក់ទំនង ManyToOne និង OneToMany។
- ស្គាល់ foreign key និង owning side។
- រក្សាទុក និងអាន Entity ដែលមានទំនាក់ទំនងគ្នា។

## ១. ប្រភេទទំនាក់ទំនង

Doctrine គាំទ្រ៖

- **ManyToOne / OneToMany**៖ Product ច្រើនអាចជាកម្មសិទ្ធិរបស់ Category មួយ។ តារាង Product មាន foreign key `category_id`។
- **ManyToMany**៖ Student ច្រើនចូលរៀន Course ច្រើន។ ត្រូវការតារាងភ្ជាប់។
- **OneToOne**៖ User ម្នាក់មាន Profile មួយ ហើយ Profile នោះជារបស់ User ម្នាក់។

សំណួរដែលគួរសួរមុនកំណត់ mapping៖ ក្នុងទិសនីមួយៗ តើអាចភ្ជាប់ object ច្រើនបានទេ?

## ២. បង្កើត Category និងទំនាក់ទំនង Product

បើមាន Entity `Product` ពីមេរៀនមុន រួចហើយ សូមបង្កើត Category៖

```bash
php bin/console make:entity Category
```

បន្ថែម property `name` ប្រភេទ `string`។ បន្ទាប់មកបន្ថែមទំនាក់ទំនងទៅ Product៖

```bash
php bin/console make:entity Product
```

ជ្រើស property name `category`, type `relation`, class `Category`, relation type `ManyToOne`។ បើ Product មិនអាចគ្មាន Category បាន សូមកំណត់ nullable = no។ អនុញ្ញាតឱ្យ MakerBundle បង្កើត `products` collection នៅ Category ផង ប្រសិនបើចង់អាន products ពី category។

ឧទាហរណ៍ mapping សំខាន់ៗ៖

```php
// src/Entity/Product.php

use Doctrine\ORM\Mapping as ORM;

#[ORM\ManyToOne(targetEntity: Category::class, inversedBy: 'products')]
#[ORM\JoinColumn(nullable: false)]
private ?Category $category = null;
```

```php
// src/Entity/Category.php

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;

#[ORM\OneToMany(mappedBy: 'category', targetEntity: Product::class)]
private Collection $products;

public function __construct()
{
    $this->products = new ArrayCollection();
}

/** @return Collection<int, Product> */
public function getProducts(): Collection
{
    return $this->products;
}
```

`ManyToOne` ជា owning side ក្នុងឧទាហរណ៍នេះ ព្រោះ foreign key ស្ថិតនៅក្នុងតារាង Product។ `OneToMany` ជា inverse side ដែលចង្អុលទៅ property `category` របស់ Product តាម `mappedBy`។ ក្នុង database នឹងមាន column `category_id` លើតារាង `product`។

## ៣. បង្កើត Migration

ទំនាក់ទំនងថ្មីផ្លាស់ប្ដូរ database schema ដូច្នេះបង្កើត និងពិនិត្យ migration៖

```bash
php bin/console make:migration
```

ពិនិត្យ migration ឱ្យប្រាកដថាវាបន្ថែម foreign key ត្រឹមត្រូវ។ បើមានទិន្នន័យ Product ចាស់ ហើយ column ថ្មីជា NOT NULL, ត្រូវរៀបចំទិន្នន័យចាស់ ឬជ្រើស default/nullable តាមស្ថានភាពជាក់ស្ដែង មុនអនុវត្ត។

```bash
php bin/console doctrine:migrations:migrate
```

## ៤. កំណត់ និងរក្សាទុកទំនាក់ទំនង

ផ្គូផ្គង object ទៅ object ដោយប្រើ Entity instance; កុំផ្គូផ្គងតែ ID ដោយដៃ៖

```php
$category = new Category();
$category->setName('ឧបករណ៍កុំព្យូទ័រ');

$product = new Product();
$product->setName('ក្តារចុច');
$product->setPriceInCents(2599);
$product->setCategory($category);

$entityManager->persist($category);
$entityManager->persist($product);
$entityManager->flush();
```

Doctrine បង្កើត foreign key តាម Category object ដែលបានដាក់លើ Product។ វិធីនេះរក្សាការគ្រប់គ្រងទំនាក់ទំនងនៅក្នុង object model ហើយ Doctrine សរសេរ SQL ឱ្យ។

បើ Category មានរួច អាចរកវាតាម repository ហើយផ្ដល់វាទៅ Product៖

```php
$category = $categoryRepository->find($categoryId);

if ($category === null) {
    throw $this->createNotFoundException('រកមិនឃើញ Category ទេ។');
}

$product->setCategory($category);
$entityManager->persist($product);
$entityManager->flush();
```

## ៥. អានទិន្នន័យតាមទំនាក់ទំនង

បន្ទាប់ពីរក Product បាន៖

```php
$product = $productRepository->find($id);

if ($product === null) {
    throw $this->createNotFoundException();
}

$categoryName = $product->getCategory()->getName();
```

ទំនាក់ទំនងមួយចំនួនត្រូវបាន lazy-load៖ Doctrine អាចរង់ចាំរហូតដល់កូដស្នើសុំ Category ទើប query ទៅ database។ វាស្រួលសម្រាប់កូដ ប៉ុន្តែបើ loop លើ Products ច្រើនហើយរាល់ដងស្នើ Category អាចបង្កើត queries ច្រើន។ ពេលក្រោយអាចប្រើ JOIN ក្នុង Repository ដើម្បីកាត់បន្ថយ queries ដែលមិនចាំបាច់។

## កំហុសដែលជួបញឹកញាប់

- **ភ្លេចកំណត់ owning side**៖ ការកំណត់ទំនាក់ទំនងត្រូវធ្វើលើ Product `ManyToOne` ក្នុងឧទាហរណ៍នេះ។
- **ប្រើ `mappedBy` ខុសឈ្មោះ**៖ ត្រូវតែស្មើនឹង property name លើ entity ម្ខាងទៀត។
- **មិន initialize Collection**៖ កំណត់ `ArrayCollection` ក្នុង constructor។
- **អនុវត្ត NOT NULL foreign key លើតារាងមានទិន្នន័យចាស់**៖ រៀបចំ migration និងទិន្នន័យចាស់ឱ្យសមស្រប។
- **សន្មតថា OneToMany បង្កើត column នៅតារាង Category**៖ foreign key ស្ថិតនៅម្ខាង ManyToOne។

## លំហាត់

1. បង្កើត Category មួយ ហើយភ្ជាប់ Products ចំនួនពីរទៅវា។
2. សរសេរ controller ឬ command ដែលបង្ហាញឈ្មោះ Product និង Category។
3. ពន្យល់ថាហេតុអ្វី `ManyToOne` ជា owning side ក្នុងឧទាហរណ៍នេះ។
4. ពិនិត្យ SQL migration និងរក column ដែលជ foreign key។

## សំណួរត្រួតពិនិត្យ

1. តើ ManyToOne និង OneToMany ជាទំនាក់ទំនងពីរដាច់ដោយឡែក ឬជាទិដ្ឋភាពពីរនៃទំនាក់ទំនងតែមួយ?
2. តើ foreign key `category_id` ស្ថិតនៅតារាងណា?
3. តើ `mappedBy` សំដៅទៅ property ណា?
4. តើ lazy loading អាចបង្កើតបញ្ហាអ្វីពេល loop លើ object ច្រើន?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4: How to Work with Doctrine Associations / Relations](https://symfony.com/doc/7.4/doctrine/associations.html)
- [Symfony 7.4: Databases and the Doctrine ORM](https://symfony.com/doc/7.4/doctrine.html)
