# មេរៀនទី ៩៦៖ Symfony Doctrine CRUD និង Repositories

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- ប្រើ Repository ដើម្បីស្វែងរក Entities។
- បង្កើត និងកែប្រែ Entity ដោយប្រើ EntityManager។
- លុប Entity និងយល់ពីពេលដែល Doctrine ផ្ញើ SQL។
- រៀបចំ Controller ឱ្យបង្ហាញបញ្ជី និងព័ត៌មានលម្អិត។

> មេរៀននេះបន្តពីមេរៀនទី ៩៥។ ត្រូវមាន Doctrine តភ្ជាប់ database និង Migration សម្រាប់ Entity ដែលចង់ប្រើរួចហើយ។

## ១. Repository និង EntityManager

Doctrine ផ្ដល់ Repository សម្រាប់ស្វែងរក Entities ប្រភេទមួយ។ ជាទូទៅ Repository មាន methods ដូចជា \`find()\`, \`findOneBy()\`, \`findBy()\`, និង \`findAll()\`។

EntityManager គ្រប់គ្រង Entities ដែលកំពុងដំណើរការ និងរក្សាការផ្លាស់ប្ដូរទៅ database។ \`persist()\` ប្រាប់ Doctrine ឱ្យតាមដាន Entity ថ្មី ប៉ុន្តែមិនទាន់បញ្ជូន SQL ទេ។ \`flush()\` ទើបបញ្ជូនការបញ្ចូល ឬកែប្រែទៅ database។

## ២. បង្កើតទិន្នន័យថ្មី

ឧទាហរណ៍ក្នុង Controller៖

\`\`\`php
<?php

namespace App\\Controller;

use App\\Entity\\Product;
use Doctrine\\ORM\\EntityManagerInterface;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class ProductController
{
    #[Route('/products/example', name: 'product_example_create', methods: ['POST'])]
    public function createExample(EntityManagerInterface $entityManager): Response
    {
        $product = new Product();
        $product->setName('ក្តារចុច');
        $product->setPriceInCents(2599);

        $entityManager->persist($product);
        $entityManager->flush();

        return new Response('បានបង្កើត Product លេខ '.$product->getId());
    }
}
\`\`\`

កុំបង្កើត endpoint ពិតដែលអនុញ្ញាតឱ្យនរណាក៏អាចបញ្ចូលទិន្នន័យបានដោយគ្មាន validation និង authorization។ ឧទាហរណ៍នេះបង្ហាញតែការប្រើ Doctrine; មេរៀន Form និង Security បង្ហាញការទទួល input និងការការពារបន្ថែម។

## ៣. ស្វែងរក និងបង្ហាញទិន្នន័យ

អាច inject repository ទៅ controller ដោយស្វ័យប្រវត្តិ៖

\`\`\`php
<?php

namespace App\\Controller;

use App\\Entity\\Product;
use App\\Repository\\ProductRepository;
use Symfony\\Bundle\\FrameworkBundle\\Controller\\AbstractController;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class ProductController extends AbstractController
{
    #[Route('/products', name: 'product_index', methods: ['GET'])]
    public function index(ProductRepository $products): Response
    {
        return $this->render('product/index.html.twig', [
            'products' => $products->findBy([], ['id' => 'DESC']),
        ]);
    }

    #[Route('/products/{id}', name: 'product_show', methods: ['GET'])]
    public function show(ProductRepository $products, int $id): Response
    {
        $product = $products->find($id);

        if ($product === null) {
            throw $this->createNotFoundException('រកមិនឃើញ Product នេះទេ។');
        }

        return $this->render('product/show.html.twig', [
            'product' => $product,
        ]);
    }
}
\`\`\`

\`find($id)\` ត្រឡប់ Entity មួយ ឬ \`null\`។ ត្រូវដោះស្រាយករណី \`null\` ដើម្បីឆ្លើយតប 404 ជំនួសឱ្យការប្រើ object ដែលមិនមាន។ ក្នុង Twig អាចបង្ហាញ properties តាម៖

\`\`\`twig
<h1>{{ product.name }}</h1>
<p>តម្លៃ៖ {{ (product.priceInCents / 100)|number_format(2) }}</p>
\`\`\`

Twig escape តម្លៃដែលបង្ហាញជាលំនាំដើម។ ការបម្លែងតម្លៃប្រាក់សម្រាប់បង្ហាញមិនគួរប្រើជាទិន្នន័យគណនាផ្លូវការ ព្រោះការបង្ហាញជាទសភាគគឺសម្រាប់ UI។

## ៤. កែប្រែ Entity

ពេល Doctrine បានទាញ Entity ចេញពី database វាតាមដានការផ្លាស់ប្ដូររបស់ object នោះ។ កែ property រួចហៅ \`flush()\`៖

\`\`\`php
$product = $products->find($id);

if ($product === null) {
    throw $this->createNotFoundException();
}

$product->setName('ក្តារចុចឥតខ្សែ');
$entityManager->flush();
\`\`\`

ជាធម្មតាមិនចាំបាច់ហៅ \`persist()\` ម្ដងទៀតលើ Entity ដែល Doctrine កំពុងគ្រប់គ្រងទេ។ \`flush()\` ប្រៀបធៀបស្ថានភាព ហើយធ្វើ UPDATE នៅពេលចាំបាច់។

## ៥. លុប Entity

\`\`\`php
$product = $products->find($id);

if ($product === null) {
    throw $this->createNotFoundException();
}

$entityManager->remove($product);
$entityManager->flush();
\`\`\`

\`remove()\` កំណត់ Entity សម្រាប់លុប; \`flush()\` ទើបអនុវត្ត DELETE។ ក្នុងកម្មវិធីពិត ការលុបគួរប្រើ HTTP method \`POST\` ឬ \`DELETE\` និងការការពារ CSRF/Authorization តាមតម្រូវការ។ កុំលុបដោយសារ GET request ព្រោះ GET គួរតែសម្រាប់អានទិន្នន័យ។

## ៦. Query តាមលក្ខខណ្ឌ

ឧទាហរណ៍រក Products ដែលមានឈ្មោះជាក់លាក់៖

\`\`\`php
$product = $products->findOneBy(['name' => 'ក្តារចុច']);

$recentProducts = $products->findBy(
    ['name' => 'ក្តារចុច'],
    ['id' => 'DESC'],
    10
);
\`\`\`

បើ query មានលក្ខខណ្ឌស្មុគស្មាញ ឬត្រូវប្រើញឹកញាប់ សូមបង្កើត custom method ក្នុង \`ProductRepository\` ជំនួសការដាក់ query logic ទាំងអស់ក្នុង Controller។ កុំបញ្ចូល user input ទៅក្នុង SQL string ដោយ concatenation។ ប្រើ QueryBuilder parameters ឬ Doctrine repository APIs ដែល bind តម្លៃដោយសុវត្ថិភាព។

## ៧. សាកល្បងដោយ Doctrine Console

នៅពេលបង្កើតទិន្នន័យសាកល្បងក្នុង local environment អាចពិនិត្យតារាងដោយ៖

\`\`\`bash
php bin/console dbal:run-sql "SELECT id, name, price_in_cents FROM product"
\`\`\`

ពិនិត្យ routes៖

\`\`\`bash
php bin/console debug:router
\`\`\`

សូមកុំដំណើរការ INSERT, UPDATE ឬ DELETE សាកល្បងលើ database production។ ប្រើ development database និងទិន្នន័យសាកល្បងដែលអាចលុបចោលបាន។

## កំហុសដែលជួបញឹកញាប់

- **ហៅ \`persist()\` ហើយមិនឃើញ row ថ្មី**៖ ត្រូវហៅ \`flush()\` ផងដែរ។
- **កែ Entity ប៉ុន្តែមិនបាន UPDATE**៖ ពិនិត្យថា Entity នោះត្រូវបានទាញមកពី EntityManager ហើយហៅ \`flush()\`។
- **ប្រើ null ជា Entity**៖ ពិនិត្យលទ្ធផល Repository ហើយឆ្លើយ 404 ឬលទ្ធផលសមស្រប។
- **ប្រើ GET ដើម្បីលុប**៖ ប្រើ POST/DELETE ហើយការពារ CSRF និងពិនិត្យសិទ្ធិ។
- **សរសេរ SQL ដោយភ្ជាប់ string**៖ bind parameters ឬប្រើ Repository/QueryBuilder។

## លំហាត់

1. បង្កើត route \`GET /products\` ដែលបង្ហាញ Products ទាំងអស់ក្នុង Twig។
2. បង្កើត route បង្ហាញ Product មួយតាម ID ហើយដោះស្រាយករណីរកមិនឃើញ។
3. បង្កើត POST route ដែលកែឈ្មោះ Product មួយ ហើយពន្យល់ថាហេតុអ្វីមិនគួរប្រើ GET។
4. សាកល្បង \`findBy()\` ជាមួយលំដាប់ និងចំនួនកំណត់។

## សំណួរត្រួតពិនិត្យ

1. តើ Repository និង EntityManager មានតួនាទីខុសគ្នាដូចម្ដេច?
2. តើ \`persist()\` និង \`flush()\` ខុសគ្នាដូចម្ដេច?
3. តើ Repository \`find()\` អាចត្រឡប់តម្លៃអ្វីខ្លះ?
4. តើហេតុអ្វីការលុបមិនគួរប្រើ GET?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4: Databases and the Doctrine ORM](https://symfony.com/doc/7.4/doctrine.html)
- [Symfony 7.4: Routing](https://symfony.com/doc/7.4/routing.html)
