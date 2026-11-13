# មេរៀនទី ៩៩៖ Symfony Authorization, Roles និង Access Control

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- កំណត់ roles សម្រាប់ User។
- ការពារ URL ដោយ \`access_control\`។
- ការពារ Controller ដោយ \`#[IsGranted]\` ឬ \`denyAccessUnlessGranted()\`។
- បង្ហាញ ឬលាក់ UI តាម role ដោយយល់ថា UI check មិនជំនួស server authorization ទេ។

## ១. Roles ជាអ្វី?

Role ជាស្លាកសិទ្ធិដែលភ្ជាប់ទៅ User ឧទាហរណ៍ \`ROLE_USER\` និង \`ROLE_ADMIN\`។ Symfony ធ្វើ Role Hierarchy ដោយស្វ័យប្រវត្តិ៖ \`ROLE_ADMIN\` ដែលមានក្នុង User អាចទទួលសិទ្ធិ \`ROLE_USER\` តាម configuration។

Role មិនគួរជំនួសការត្រួតពិនិត្យសិទ្ធិលើទិន្នន័យនីមួយៗទេ។ អ្នកប្រើដែលចូលបាន មិនចាំបាច់មានសិទ្ធិកែសម្រួលគ្រប់ Product ឬ Post ទាំងអស់ឡើយ។

## ២. ការពារ URL តាម Access Control

ក្នុង \`config/packages/security.yaml\` អាចកំណត់៖

\`\`\`yaml
security:
    access_control:
        - { path: ^/admin/login$, roles: PUBLIC_ACCESS }
        - { path: ^/admin, roles: ROLE_ADMIN }
        - { path: ^/account, roles: ROLE_USER }
\`\`\`

Symfony ពិនិត្យច្បាប់ពីលើចុះក្រោម ហើយប្រើតែច្បាប់ដំបូងដែលត្រូវនឹង request។ ដូច្នេះដាក់ exception ជាក់លាក់ (ឧទាហរណ៍ login ដែលបើកជាសាធារណៈ) មុនច្បាប់ទូលំទូលាយ \`^/admin\`។ \`^\` មានន័យថា URL ត្រូវចាប់ផ្ដើមដោយ path នោះ។

\`PUBLIC_ACCESS\` អនុញ្ញាតឱ្យអ្នកមិនទាន់ login ចូលបាន។ \`ROLE_USER\` ត្រូវការអ្នកប្រើដែលបាន authenticated ហើយមាន role នោះ។

## ៣. ការពារ Controller

ប្រើ attribute លើ class ឬ method៖

\`\`\`php
<?php

namespace App\\Controller;

use Symfony\\Bundle\\FrameworkBundle\\Controller\\AbstractController;
use Symfony\\Component\\HttpFoundation\\Response;
use Symfony\\Component\\Routing\\Attribute\\Route;
use Symfony\\Component\\Security\\Http\\Attribute\\IsGranted;

#[IsGranted('ROLE_ADMIN')]
class AdminController extends AbstractController
{
    #[Route('/admin', name: 'admin_dashboard', methods: ['GET'])]
    public function index(): Response
    {
        return new Response('សូមស្វាគមន៍ Admin');
    }
}
\`\`\`

ឬអាចពិនិត្យក្នុង method៖

\`\`\`php
$this->denyAccessUnlessGranted('ROLE_ADMIN');
\`\`\`

បើអ្នកប្រើមិនទាន់ login Symfony អាចចាប់ផ្ដើម authentication flow។ បើបាន login ប៉ុន្តែគ្មានសិទ្ធិ នឹងទទួលបាន 403។ Attribute/controller check មានប្រយោជន៍សម្រាប់ action ជាក់លាក់; \`access_control\` ល្អសម្រាប់ផ្នែក URL ធំៗ។

## ៤. Role Hierarchy

កំណត់ role hierarchy ក្នុង security configuration៖

\`\`\`yaml
security:
    role_hierarchy:
        ROLE_ADMIN: ROLE_USER
        ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]
\`\`\`

User ដែលមាន \`ROLE_ADMIN\` នឹងទទួលសិទ្ធិពី \`ROLE_USER\`។ ប្រើ hierarchy ដើម្បីចែកចាយសិទ្ធិជាស្រទាប់។ កុំដាក់ role ទៅ User តាមទិន្នន័យដែលមិនទុកចិត្ត ដូចជា form registration សាធារណៈ។

## ៥. ពិនិត្យសិទ្ធិលើ Object

បើចង់ឱ្យអ្នកប្រើកែបានតែ Post ដែលខ្លួនជាម្ចាស់ ច្បាប់ \`ROLE_USER\` មិនគ្រប់គ្រាន់ទេ។ ត្រូវពិនិត្យទំនាក់ទំនងរវាង User បច្ចុប្បន្ន និង object៖

\`\`\`php
use App\\Entity\\Post;
use Symfony\\Component\\Security\\Http\\Attribute\\IsGranted;

#[Route('/posts/{id}/edit', name: 'post_edit', methods: ['GET', 'POST'])]
#[IsGranted('edit', subject: 'post')]
public function edit(Post $post): Response
{
    // បន្តកែ Post បន្ទាប់ពី voter អនុញ្ញាត
}
\`\`\`

នៅពេលប្រើ custom attribute ឈ្មោះ \`edit\` ត្រូវមាន Voter ដែលកំណត់ថា user អាច edit post នេះឬអត់។ Voter ជាប្រធានបទមេរៀនបន្ទាប់។ សម្រាប់ឧទាហរណ៍ខ្លី អាចប្រើ៖

\`\`\`php
$this->denyAccessUnlessGranted('edit', $post);
\`\`\`

កុំសន្មតថា parameter ID ក្នុង URL មានន័យថា user មានសិទ្ធិលើ object នោះ។ ត្រូវធ្វើ authorization server-side រាល់ពេលអាន ឬកែ resource ដែលការពារ។

## ៦. ពិនិត្យ Role ក្នុង Twig

អាចលាក់ link ដោយ៖

\`\`\`twig
{% if is_granted('ROLE_ADMIN') %}
    <a href="{{ path('admin_dashboard') }}">ផ្ទាំង Admin</a>
{% endif %}
\`\`\`

នេះគ្រាន់តែជាការគ្រប់គ្រងអ្វីដែលបង្ហាញក្នុង UI។ អ្នកប្រើអាចវាយ URL ដោយផ្ទាល់ ឬផ្ញើ request តាមវិធីផ្សេងបាន ដូច្នេះ route/controller ត្រូវការការការពារ server-side ដដែល។

## ៧. សាកល្បង Configuration

មើល security configuration ដែល Symfony បាន load៖

\`\`\`bash
php bin/console debug:config security
php bin/console debug:router
\`\`\`

សាកល្បងយ៉ាងហោចណាស់៖

- អ្នកមិនទាន់ login ទៅ \`/admin\` ត្រូវបានបញ្ជូនទៅ login flow។
- \`ROLE_USER\` ចូល \`/account\` បាន ប៉ុន្តែមិនអាចចូល \`/admin\`។
- \`ROLE_ADMIN\` ចូល \`/admin\` បាន។
- URL login ដែលមាន \`PUBLIC_ACCESS\` នៅតែអាចបើកបាន។

## កំហុសដែលជួបញឹកញាប់

- **ដាក់ច្បាប់ទូលំទូលាយមុនច្បាប់ជាក់លាក់**៖ access control ប្រើតែ rule ដំបូងដែល match។
- **គិតថាលាក់ button គ្រប់គ្រាន់**៖ ការអនុញ្ញាតត្រូវពិនិត្យនៅ server។
- **ផ្ដល់ ROLE_ADMIN ពី registration form**៖ user អាចកែ request ដើម្បីក្លាយជា admin។
- **មិនពិនិត្យ owner នៃ object**៖ ការចូលជាសមាជិកមិនស្មើនឹងសិទ្ធិលើទិន្នន័យគ្រប់គ្នា។
- **ច្រឡំ 401/redirect និង 403**៖ អ្នកមិនទាន់ login ត្រូវ authenticate; អ្នកបាន login តែខ្វះសិទ្ធិទទួល 403 ជាធម្មតា។

## លំហាត់

1. ការពារ \`/admin\` ដោយ \`ROLE_ADMIN\` និងរក្សា \`/admin/login\` ជា \`PUBLIC_ACCESS\`។
2. បន្ថែម \`ROLE_EDITOR\` ក្នុង role hierarchy ឱ្យមានសិទ្ធិ \`ROLE_USER\`។
3. ការពារ Controller action មួយដោយ \`#[IsGranted]\`។
4. លាក់ admin link ក្នុង Twig ហើយពន្យល់ថាហេតុអ្វីត្រូវការការពារ server-side ដដែល។
5. រៀបចំការត្រួតពិនិត្យសិទ្ធិ edit Post តាម owner សម្រាប់មេរៀន Voter បន្ទាប់។

## សំណួរត្រួតពិនិត្យ

1. តើ Authentication និង Authorization ខុសគ្នាដូចម្ដេច?
2. តើ Symfony ប្រើ access_control rule ណា នៅពេលមានច្រើន rule match?
3. តើ \`ROLE_ADMIN\` និង \`ROLE_USER\` គ្រប់គ្រាន់សម្រាប់ពិនិត្យម្ចាស់ Post ដែរឬទេ?
4. តើ \`is_granted()\` ក្នុង Twig អាចជំនួសការការពារ Controller បានទេ?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Security](https://symfony.com/doc/7.4/security.html)
- [Symfony 7.4 Voters](https://symfony.com/doc/7.4/security/voters.html)
