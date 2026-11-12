# មេរៀនទី ៤៩៖ សុវត្ថិភាព Dependencies និង Composer Audit

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Direct Dependency និង Transitive Dependency។
- យល់ពីតួនាទីរបស់ `composer.lock` ក្នុងការដំឡើង Version ដដែល។
- ប្រើ `composer audit` ដើម្បីពិនិត្យ Vulnerability Advisory។
- ធ្វើ Dependency Update ដោយមានការពិនិត្យ និង Test។
- បន្ថែម Dependency Audit ទៅ CI ដោយមិនលាក់បញ្ហា។

## ១. Dependency ជាផ្នែកមួយនៃសុវត្ថិភាពកម្មវិធី

Dependency គឺជា Library ឬ Package ខាងក្រៅដែល Project ប្រើ។ **Direct Dependency** ត្រូវបានកំណត់ដោយ Project ផ្ទាល់; **Transitive Dependency** គឺ Package ដែល Dependency ផ្សេងទៀតត្រូវការ។

បញ្ហាសុវត្ថិភាពក្នុង Dependency អាចប៉ះពាល់កម្មវិធី ទោះកូដរបស់យើងមិនបានសរសេរបញ្ហានោះក៏ដោយ។ ត្រូវដឹងថា Project ដំឡើងអ្វី និងមានវិធីធ្វើបច្ចុប្បន្នភាព។

## ២. តួនាទីរបស់ composer.json និង composer.lock

- `composer.json` កំណត់ Dependency Requirement និង Metadata របស់ Project។
- `composer.lock` កត់ Version ជាក់លាក់ដែលបាន Resolve សម្រាប់ Package ទាំងនោះ។
- `composer install` ដំឡើងតាម Lock File ប្រសិនបើមាន។
- `composer update` Resolve Version ថ្មីតាម Constraint ហើយកែ Lock File។

Commit `composer.lock` សម្រាប់ Application ដើម្បីឱ្យ Local, CI និង Production ប្រើ Dependency Version ដូចគ្នា។ កុំលុប Lock File ដើម្បីដោះស្រាយ Version Conflict ដោយមិនយល់ពីផលប៉ះពាល់។

## ៣. ពិនិត្យ Dependency Advisory

ប្រើ Composer Audit៖

```bash
composer audit
```

Audit អាចរាយការណ៍ពី Security Advisories និងព័ត៌មាន Package ផ្សេងៗតាម Composer Version/Configuration។ ពិនិត្យឈ្មោះ Package, កំណែដែលរងផលប៉ះពាល់ និងកំណែដែលបានជួសជុល។

កុំសន្មតថា Audit រកឃើញ Vulnerability គ្រប់ប្រភេទ៖ វាមិនជំនួស Code Review, Test, Static Analysis ឬការត្រួតពិនិត្យ Package Provenance ទេ។ រក្សា Composer ឱ្យបានទាន់សម័យ ដើម្បីទទួលបានមុខងារ Audit និង Advisory Data បច្ចុប្បន្ន។

## ៤. Update ដោយមានការគ្រប់គ្រង

Flow Update ដែលណែនាំ៖

1. រកមើល Package ចាស់ ឬ Advisory ដោយ `composer outdated` និង `composer audit`។
2. អាន Release Notes/Changelog របស់ Package។
3. Update Package ដែលចាំបាច់ ដោយបញ្ជាក់ Package ឬ Scope ដែលចង់កែ។
4. ពិនិត្យ Diff របស់ `composer.lock` និង `composer.json`។
5. រត់ Unit/Integration Test, Static Analysis និង CI។
6. Deploy តាម Release Process និងតាមដានបញ្ហាក្រោយ Deploy។

ឧទាហរណ៍ Update Package ជាក់លាក់៖

```bash
composer update vendor/package
```

បើមាន Advisory សំខាន់ ត្រូវកែឱ្យបានឆាប់តាម Risk និង Availability របស់ Patch; បើ Update អាចបំបែក API ត្រូវរៀបចំការធ្វើតេស្ត និង Migration Plan។ កុំបង្ខំ Version ឬបិទ Audit ដោយគ្មានការវាយតម្លៃហានិភ័យ។

## ៥. ដាក់ Audit ក្នុង CI

បន្ថែម Step ទៅ GitHub Actions ក្រោយ Composer Install ឬមុន Test៖

```yaml
- name: Audit Composer dependencies
  run: composer audit
```

Composer Audit មាន Exit Code ដែលអាចធ្វើឱ្យ CI បរាជ័យពេលរកឃើញបញ្ហា។ កុំកំណត់ Workflow ឱ្យមើលរំលង Exit Code ឬប្រើ Ignore Rule ទូលំទូលាយ។ បើមាន Exception ដែលបានពិនិត្យ ត្រូវមាន Owner, មូលហេតុ, Deadline និងវិធីតាមដានការជួសជុល។

## ៦. ពិនិត្យប្រភព និងការដំណើរការ Package

Dependency អាចមាន Composer Plugin ឬ Scripts ដែលរត់ពេល Install។ ដូច្នេះ៖

- ដំឡើង Package ដែលមានប្រភព និង Maintainer អាចទុកចិត្តបាន។
- ពិនិត្យ Package ថាត្រូវបានថែទាំ និងមិនត្រូវបាន Abandon ដោយគ្មានជម្រើស។
- ពិនិត្យ Composer Plugins និង Scripts មុនអនុញ្ញាតឱ្យដំណើរការ។
- កុំដំឡើង Package ពី URL/Repository មិនទុកចិត្តនៅ CI/Production។
- កំណត់ Dependency ឱ្យតូចតាមតម្រូវការ និងលុប Package ដែលមិនប្រើ។

## ៧. កំហុសដែលជួបញឹកញាប់

- មិន Commit `composer.lock` ក្នុង Application។
- រត់ `composer update` នៅ Production ដោយមិនពិនិត្យ Lock Diff/Test។
- មិនរត់ `composer audit` ឬមើលរំលង Advisory។
- គិតថា Dependency Audit ធានាថា App គ្មាន Vulnerability។
- ដាក់ Ignore សម្រាប់ Package ដែលរងបញ្ហាដោយគ្មាន Deadline/Owner។
- មិនពិនិត្យ Composer Plugin/Script ដែលអាចរត់កូដពេល Install។

## ៨. លំហាត់

1. រត់ `composer audit` ក្នុង Project និងពិនិត្យលទ្ធផល។
2. ស្វែងរក Direct/Transitive Dependencies តាម `composer show --tree`។
3. Update Package មួយក្នុង Branch ដាច់ដោយឡែក ហើយពិនិត្យ Lock Diff។
4. រត់ Tests និង Static Analysis ក្រោយ Update។
5. បន្ថែម `composer audit` ទៅ GitHub Actions និងពិភាក្សាថា CI គួរបរាជ័យនៅករណីណា។

## ៩. ចម្លើយណែនាំខ្លី

- `composer.lock` កំណត់ Version ជាក់លាក់សម្រាប់ Install ដែលអាចធ្វើឡើងវិញបាន។
- `composer audit` ជួយរក Advisory ដែលស្គាល់ ប៉ុន្តែមិនរកឃើញកំហុសគ្រប់ប្រភេទ។
- Update ដោយពិនិត្យ Release Notes, Lock Diff, Tests និង CI។
- ដាក់ Audit ក្នុង CI; កុំលាក់បញ្ហាដោយ Ignore ដោយគ្មានហេតុផល និងផែនការជួសជុល។
- ពិនិត្យ Package ប្រភព, ការថែទាំ, Plugins និង Scripts មុនប្រើ។

## ឯកសារយោង

- [Composer CLI: audit, install, update](https://getcomposer.org/doc/03-cli.md)
- [Composer Basic Usage: composer.lock](https://getcomposer.org/doc/01-basic-usage.md)
- [មេរៀនទី ២៣៖ Composer Dependencies និង Version Constraints](lesson-23-composer-dependencies-version-constraints.md)
- [មេរៀនទី ៤៧៖ CI/CD ដោយ GitHub Actions](lesson-47-php-github-actions-ci.md)
