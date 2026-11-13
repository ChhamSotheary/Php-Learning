# មេរៀនទី ៩៨៖ Symfony Security និង User Authentication

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- បង្កើត User class សម្រាប់ Symfony Security។
- យល់ពី User Provider និង Firewall។
- រក្សាទុក password ជា hash ដោយ password hasher។
- រៀបចំ form login និង logout ជាមូលដ្ឋាន។

## ១. Authentication និង Authorization

**Authentication** ឆ្លើយថា “អ្នកប្រើនេះជានរណា?” ឧទាហរណ៍ ពិនិត្យ email និង password ហើយបង្កើត session ដែលមាន user ចូលប្រើ។

**Authorization** ឆ្លើយថា “អ្នកប្រើនេះអាចធ្វើអ្វីបាន?” ឧទាហរណ៍ អនុញ្ញាតតែអ្នកគ្រប់គ្រងចូលទៅកាន់ \`/admin\`។ មេរៀននេះផ្ដោតលើ Authentication; ការកំណត់សិទ្ធិលម្អិតនឹងរៀនបន្ត។

## ២. ដំឡើង និងបង្កើត User

ដំឡើង SecurityBundle ប្រសិនបើមិនទាន់មាន៖

\`\`\`bash
composer require symfony/security-bundle
composer require --dev symfony/maker-bundle
\`\`\`

បង្កើត User ដោយ Maker៖

\`\`\`bash
php bin/console make:user
\`\`\`

ជ្រើស \`User\`, រក្សាទុកក្នុង Doctrine ប្រសិនបើកម្មវិធីប្រើ database, កំណត់ \`email\` ជា identifier ហើយជ្រើសថាត្រូវប្រើ password hashing។ Maker បង្កើត User entity/repository និងធ្វើបច្ចុប្បន្នភាព Security configuration។

បន្ទាប់មកបង្កើត និងពិនិត្យ migration ដូចមេរៀន Doctrine៖

\`\`\`bash
php bin/console make:migration
php bin/console doctrine:migrations:migrate
\`\`\`

User ត្រូវ implement \`UserInterface\`; បើកម្មវិធីប្រើ password វាក៏ implement \`PasswordAuthenticatedUserInterface\` ផងដែរ។ User identifier ជាទូទៅជា email ឬ username ដែលមិនស្ទួនក្នុង database។

## ៣. កំណត់ Password Hasher

ក្នុង \`config/packages/security.yaml\` ត្រូវមានការកំណត់ hasher សម្រាប់ user ដែលផ្ទៀងផ្ទាត់ដោយ password៖

\`\`\`yaml
security:
    password_hashers:
        Symfony\\Component\\Security\\Core\\User\\PasswordAuthenticatedUserInterface: 'auto'
\`\`\`

មិនត្រូវរក្សាទុក password ដើម (plain text) ឬ encrypt ដើម្បីអាច decrypt ត្រឡប់វិញទេ។ ពេលចុះឈ្មោះ ត្រូវប្រើ \`UserPasswordHasherInterface\` ដើម្បី hash password មុនរក្សាទុក។ ពេល login Symfony ប្រៀបធៀប password ដែលបញ្ចូលនឹង hash ដែលបានរក្សាទុក។

ឧទាហរណ៍ service សម្រាប់បង្កើត user៖

\`\`\`php
<?php

namespace App\\Service;

use App\\Entity\\User;
use Symfony\\Component\\PasswordHasher\\Hasher\\UserPasswordHasherInterface;

class UserRegistration
{
    public function __construct(
        private UserPasswordHasherInterface $passwordHasher,
    ) {
    }

    public function hashPassword(User $user, string $plainPassword): void
    {
        $user->setPassword(
            $this->passwordHasher->hashPassword($user, $plainPassword)
        );
    }
}
\`\`\`

ឧទាហរណ៍នេះបង្ហាញតែការបង្កើត hash។ កុំប្រើ plain password ជា database column ឬបង្ហាញវាក្នុង logs។

## ៤. User Provider និង Firewall

- **User Provider** ដឹងពីរបៀបស្វែងរក User ក្នុង database តាម identifier។
- **Firewall** ជ្រើសរើសយន្តការផ្ទៀងផ្ទាត់សម្រាប់ request ដែលចូលមក។
- **Password Hasher** បង្កើត និងផ្ទៀងផ្ទាត់ password hash។

MakerBundle ជាទូទៅកំណត់ entity provider ដែលស្វែងរកតាម email។ ក្នុង \`security.yaml\` project របស់អ្នក សូមពិនិត្យថា provider ឈ្មោះដែល firewall ប្រើគឺត្រូវគ្នា។ Configuration អាចខុសគ្នាតាម project និង version; រក្សាផ្នែក environment firewall ដែល Symfony បានបង្កើត ហើយបន្ថែម login configuration តាម project នោះ។

## ៥. បង្កើត Login Form

អាចប្រើ MakerBundle បង្កើត authenticator៖

\`\`\`bash
php bin/console make:security:form-login
\`\`\`

Maker សួរឈ្មោះ Controller និង route ហើយបង្កើត Controller, login template និង configuration ដែលត្រូវការ។ ឧទាហរណ៍ login template ត្រូវ render form error និង CSRF token តាមការបង្កើតរបស់ Maker។ ពិនិត្យ route name និង template location ដែលវាបង្កើត ហើយសាកល្បងក្នុង browser។

នៅពេលមាន login request លំហូរទូទៅគឺ៖

1. Request ចូលក្នុង firewall ដែលត្រូវគ្នា។
2. Authenticator អាន identifier និង password ពី request។
3. User Provider ស្វែងរក User តាម identifier។
4. Password Hasher ពិនិត្យ password ទល់នឹង hash។
5. បើជោគជ័យ Symfony រក្សា authentication state ក្នុង session និងបញ្ជូនទៅទំព័រកំណត់។
6. បើបរាជ័យ បង្ហាញ login page ជាមួយសារកំហុសសមស្រប ដោយមិនបង្ហាញថា email ឬ password ណាខុស។

## ៦. Logout

ក្នុង firewall អាចកំណត់ logout path ដែល login generator បានបង្កើត ឬប្រើ route ផ្ទាល់ខ្លួន៖

\`\`\`yaml
security:
    firewalls:
        main:
            # provider និង form_login ត្រូវតាម config របស់ project
            logout:
                path: app_logout
                # target: app_home
\`\`\`

Route name ក្នុង \`path\` ត្រូវមានពិត។ កុំធ្វើ logout តាម GET ដែលអាចត្រូវបានបើកដោយ link ឬ third-party content; ប្រើ flow ដែល framework រៀបចំ និងការពារ CSRF តាម config របស់អ្នក។

## ៧. សាកល្បង និង Debug

ពិនិត្យ route និង configuration៖

\`\`\`bash
php bin/console debug:router
php bin/console debug:config security
\`\`\`

សាកល្បង registration/login ជាមួយ test account ក្នុង development database៖

- password មិនត្រូវទទេ និងត្រូវបាន hash មុនរក្សាទុក។
- login ដោយព័ត៌មានត្រឹមត្រូវគួរបង្កើត session។
- login ដោយព័ត៌មានខុសគួរបរាជ័យដោយមិនបង្ហាញព័ត៌មានសម្ងាត់។
- logout គួរបញ្ចប់ authenticated session។
- កុំប្រើ password ពិតរបស់អ្នកក្នុងទិន្នន័យសាកល្បង ឬដាក់ក្នុង Git។

## កំហុសដែលជួបញឹកញាប់

- **User មិនអាច login**៖ ពិនិត្យថា identifier ដែល User Provider ប្រើត្រូវនឹង property របស់ User។
- **Password មិនអាចផ្ទៀងផ្ទាត់**៖ ធានាថាបាន hash ដោយ hasher របស់ Symfony មុនរក្សាទុក។
- **ភ្លេច migration**៖ User table មិនមានក្នុង database រហូតដល់ migration បានអនុវត្ត។
- **Firewall គ្មាន provider ឬ authenticator ត្រឹមត្រូវ**៖ ពិនិត្យ security config របស់ project និងលទ្ធផល \`debug:config security\`។
- **បង្ហាញព័ត៌មាន login លម្អិតពេក**៖ ប្រើសារបរាជ័យទូទៅ ដើម្បីកុំឱ្យអ្នកវាយប្រហារស្វែងរក email ដែលមានគណនី។

## លំហាត់

1. ប្រើ MakerBundle បង្កើត User និង form-login ក្នុង Symfony project ថ្មី។
2. ពិនិត្យថា User provider ស្វែងរកតាម email។
3. បង្កើត User test មួយ ដោយ hash password តាម \`UserPasswordHasherInterface\`។
4. សាកល្បង login ជោគជ័យ, password ខុស និង logout។
5. គូរលំហូរពី HTTP request រហូតដល់ authenticated session។

## សំណួរត្រួតពិនិត្យ

1. តើ Authentication ខុសពី Authorization ដូចម្ដេច?
2. តើ User Provider មានតួនាទីអ្វី?
3. តើហេតុអ្វីត្រូវ hash password ជំនួសការរក្សាទុក plain text?
4. តើ Firewall ធ្វើអ្វីចំពោះ request?
5. តើ session ជួយអ្វីក្រោយ login ជោគជ័យ?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4 Security](https://symfony.com/doc/7.4/security.html)
- [Symfony 7.4 Password Hashing](https://symfony.com/doc/7.4/security/passwords.html)
- [Symfony 7.4 Login Form](https://symfony.com/doc/7.4/security/form_login.html)
