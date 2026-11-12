# មេរៀនទី ៩៤៖ Symfony Forms និង Data Validation

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បង្កើត Form Type ដាច់ដោយឡែកពី Controller។
- ភ្ជាប់ Form ទៅ PHP object ដោយប្រើ \`data_class\`។
- ទទួល និងដំណើរការ HTTP request តាម Form component។
- កំណត់ច្បាប់ Validation ដោយ PHP attributes។
- បង្ហាញសារ error និងបញ្ចប់ការបញ្ជូន Form។

## ១. Form និង Validation ធ្វើការយ៉ាងដូចម្តេច?

Symfony Form ភ្ជាប់ទិន្នន័យ PHP និង HTML form ទៅវិញទៅមក៖ ពេលបង្ហាញ form វាបម្លែងទិន្នន័យពី object ទៅជា field ក្នុង HTML។ ពេលអ្នកប្រើបញ្ជូន form វាអានតម្លៃពី request បម្លែងទៅតាមប្រភេទត្រឹមត្រូវ ហើយផ្គូផ្គងទិន្នន័យទៅ object។

Validation គឺជាការត្រួតពិនិត្យថាទិន្នន័យគោរពតាមច្បាប់ដែលយើងកំណត់ឬអត់។ ឧទាហរណ៍ ឈ្មោះកិច្ចការមិនអាចទទេ ហើយត្រូវមានប្រវែងមិនលើសកំណត់។ Form អាចដំណើរការ validation ទាំងនេះដោយស្វ័យប្រវត្តិ នៅពេលហៅ \`isValid()\` បន្ទាប់ពីបញ្ជូន request។

## ២. ដំឡើង Components

ក្នុង Symfony project ដែលប្រើ Symfony Flex សូមដំឡើង Form, Validator និង Twig bridge៖

\`\`\`bash
composer require symfony/form symfony/validator
\`\`\`

ដើម្បីបង្ហាញ Form ក្នុង Twig ត្រូវមាន Twig bridge ផងដែរ។ Project ដែលបានរៀបចំជា \`--webapp\` ជាទូទៅមាន package ទាំងនេះរួចហើយ។ បើខ្វះ Twig integration៖

\`\`\`bash
composer require symfony/twig-bridge
\`\`\`

## ៣. បង្កើត PHP Class និង Validation Rules

បង្កើត \`src/Entity/Task.php\`៖

\`\`\`php
<?php

namespace App\\Entity;

use Symfony\\Component\\Validator\\Constraints as Assert;

class Task
{
    #[Assert\\NotBlank(message: 'សូមបញ្ចូលឈ្មោះកិច្ចការ។')]
    #[Assert\\Length(
        max: 120,
        maxMessage: 'ឈ្មោះកិច្ចការមិនអាចលើសពី {{ limit }} តួអក្សរទេ។'
    )]
    private string $title = '';

    #[Assert\\Length(
        max: 1000,
        maxMessage: 'សេចក្ដីពិពណ៌នាមិនអាចលើសពី {{ limit }} តួអក្សរទេ។'
    )]
    private string $description = '';

    public function getTitle(): string
    {
        return $this->title;
    }

    public function setTitle(string $title): void
    {
        $this->title = $title;
    }

    public function getDescription(): string
    {
        return $this->description;
    }

    public function setDescription(string $description): void
    {
        $this->description = $description;
    }
}
\`\`\`

\`NotBlank\` បដិសេធតម្លៃទទេ ឬមានតែចន្លោះ។ \`Length\` កំណត់ប្រវែងអតិបរមា។ ច្បាប់ទាំងនេះនៅលើ object ដើម្បីឱ្យកម្មវិធីអាចប្រើឡើងវិញ ទោះទិន្នន័យមកពី form, API ឬប្រភពផ្សេងក៏ដោយ។

## ៤. បង្កើត Form Type

បង្កើត \`src/Form/TaskType.php\`៖

\`\`\`php
<?php

namespace App\\Form;

use App\\Entity\\Task;
use Symfony\\Component\\Form\\AbstractType;
use Symfony\\Component\\Form\\Extension\\Core\\Type\\TextareaType;
use Symfony\\Component\\Form\\Extension\\Core\\Type\\TextType;
use Symfony\\Component\\Form\\FormBuilderInterface;
use Symfony\\Component\\OptionsResolver\\OptionsResolver;

class TaskType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title', TextType::class, [
                'label' => 'ឈ្មោះកិច្ចការ',
            ])
            ->add('description', TextareaType::class, [
                'label' => 'សេចក្ដីពិពណ៌នា',
                'required' => false,
            ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Task::class,
        ]);
    }
}
\`\`\`

ឈ្មោះ field \`title\` និង \`description\` ត្រូវនឹង properties របស់ \`Task\`។ Symfony ប្រើ PropertyAccess ដើម្បីផ្គូផ្គង field ទៅ getter និង setter។ \`data_class\` ប្រាប់ Form ថាទិន្នន័យគោលដៅជាប្រភេទ object អ្វី។

## ៥. បង្កើត Controller ដើម្បីដំណើរការ Form

បង្កើត ឬកែ \`src/Controller/TaskController.php\`៖

\`\`\`php
<?php

namespace App\\Controller;

use App\\Entity\\Task;
use App\\Form\\TaskType;
use Symfony\\Bundle\\FrameworkBundle\\Controller\\AbstractController;
use Symfony\\Component\\HttpFoundation\\Request;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;

class TaskController extends AbstractController
{
    #[Route('/tasks/new', name: 'task_new', methods: ['GET', 'POST'])]
    public function new(Request $request): Response
    {
        $task = new Task();
        $form = $this->createForm(TaskType::class, $task);

        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            // នៅទីនេះអាចរក្សាទុក $task ទៅ database
            $this->addFlash('success', 'បានបង្កើតកិច្ចការដោយជោគជ័យ។');

            return $this->redirectToRoute('task_new');
        }

        return $this->render('task/new.html.twig', [
            'form' => $form,
        ]);
    }
}
\`\`\`

លំដាប់សំខាន់ៗគឺ៖

1. បង្កើត object និង Form។
2. ហៅ \`handleRequest($request)\` ដើម្បីអានទិន្នន័យពី request។
3. ពិនិត្យថា Form បានបញ្ជូនហើយ និងទិន្នន័យត្រឹមត្រូវ។
4. បើត្រឹមត្រូវ ទើបធ្វើសកម្មភាព ដូចជារក្សាទុក object។
5. បើមិនទាន់បញ្ជូន ឬមាន error ត្រឡប់ទៅបង្ហាញ Form វិញ។

លំនាំ redirect បន្ទាប់ពីបញ្ជូនដោយជោគជ័យជួយជៀសវាង browser សួរបញ្ជូន form ម្ដងទៀតពេល refresh។

## ៦. បង្ហាញ Form និង Error ក្នុង Twig

បង្កើត \`templates/task/new.html.twig\`៖

\`\`\`twig
{% extends 'base.html.twig' %}

{% block title %}បង្កើតកិច្ចការ{% endblock %}

{% block body %}
    <h1>បង្កើតកិច្ចការ</h1>

    {{ form_start(form) }}
        {{ form_row(form.title) }}
        {{ form_row(form.description) }}
        <button type="submit">រក្សាទុក</button>
    {{ form_end(form) }}
{% endblock %}
\`\`\`

\`form_start()\` បង្កើតស្លាក HTML \`<form>\` និងបញ្ចូល hidden fields ចាំបាច់ រួមទាំង CSRF token នៅពេល CSRF protection បានបើក។ \`form_row()\` បង្ហាញ label, input និង error របស់ field។ \`form_end()\` បញ្ចប់ form ហើយបង្ហាញ field ដែលនៅសល់ ដូចជា hidden CSRF field។

## ៧. សាកល្បង

ដំណើរការ Symfony local server៖

\`\`\`bash
symfony server:start
\`\`\`

បើមិនមាន Symfony CLI អាចប្រើ៖

\`\`\`bash
php -S 127.0.0.1:8000 -t public
\`\`\`

បើក \`http://127.0.0.1:8000/tasks/new\` ហើយសាកល្បង៖

- ចុចរក្សាទុកដោយមិនបញ្ចូលឈ្មោះ៖ Form គួរបង្ហាញសារ “សូមបញ្ចូលឈ្មោះកិច្ចការ”។
- បញ្ចូលឈ្មោះត្រឹមត្រូវ៖ Controller ចូលក្នុងលក្ខខណ្ឌ \`isValid()\` ហើយ redirect ត្រឡប់មក route ដដែល។
- បញ្ចូលឈ្មោះវែងជាង 120 តួអក្សរ៖ Form គួរបង្ហាញសារ error ពី \`Length\`។

បើប្រើ PHP built-in server តែឯង សូមចំណាំថា Symfony CLI ជាជម្រើសងាយស្រួលជាងសម្រាប់ routing និងបម្រើ project។ កំណត់ Document Root និង front controller ឱ្យត្រឹមត្រូវតាមការរៀបចំ project។

## ៨. កំហុសដែលជួបញឹកញាប់

- **ភ្លេច \`handleRequest()\`**៖ Form មិនទទួលទិន្នន័យ POST ទេ។
- **ភ្លេចពិនិត្យទាំង \`isSubmitted()\` និង \`isValid()\`**៖ អាចធ្វើសកម្មភាពទោះមិនទាន់មានការបញ្ជូន ឬទិន្នន័យមិនត្រឹមត្រូវ។
- **Form field មិនត្រូវនឹង object property**៖ ពិនិត្យឈ្មោះ field, getter និង setter។
- **ខ្វះ Validator component**៖ Symfony មិនអាចដំណើរការ constraints បានទេ។ ដំឡើង \`symfony/validator\`។
- **ប្រើ HTML \`required\` ជំនួស server validation**៖ ការត្រួតពិនិត្យក្នុង browser អាចត្រូវបានរំលង។ ត្រូវផ្ទៀងផ្ទាត់នៅ server ជានិច្ច។

## លំហាត់

1. បន្ថែម property \`priority\` ទៅ \`Task\` ហើយបង្កើត Choice field ដែលអនុញ្ញាតតែ \`low\`, \`normal\`, និង \`high\`។
2. កំណត់ឱ្យ \`description\` មានអប្បបរមា 10 តួអក្សរ នៅពេលអ្នកប្រើបំពេញវា។
3. ប្ដូរ Controller ឱ្យ redirect ទៅ route \`task_index\` ក្រោយបង្កើតកិច្ចការជោគជ័យ។

## សំណួរត្រួតពិនិត្យ

1. តើ \`handleRequest()\` មានតួនាទីអ្វី?
2. តើភាពខុសគ្នារវាង Form field និង Validation constraint ជាអ្វី?
3. ហេតុអ្វីត្រូវពិនិត្យ \`isSubmitted()\` និង \`isValid()\` មុនរក្សាទុក object?
4. តើ \`data_class\` ជួយអ្វីខ្លះ?

## ឯកសារយោងផ្លូវការ

- [Symfony Forms 7.4](https://symfony.com/doc/7.4/forms.html)
- [Symfony Validation 7.4](https://symfony.com/doc/7.4/validation.html)
- [Symfony CSRF Protection](https://symfony.com/doc/7.4/security/csrf.html)
