# មេរៀនទី ១០០៖ Symfony Security Voters សម្រាប់ Object Permissions

## គោលបំណង

ក្រោយរៀនមេរៀននេះ អ្នកអាច៖

- យល់ពីរបៀប Symfony សម្រេចសិទ្ធិតាម Voter។
- បង្កើត PostVoter សម្រាប់សិទ្ធិ view និង edit។
- ហៅ Voter ពី Controller និង Twig។
- ដាក់ច្បាប់សិទ្ធិលើ object នៅកន្លែងតែមួយ។

## ១. ពេលណាគួរប្រើ Voter?

ប្រើ role សម្រាប់សិទ្ធិទូទៅ ដូចជា admin។ ប្រើ Voter នៅពេលច្បាប់ពាក់ព័ន្ធនឹង object ជាក់លាក់ ឧទាហរណ៍៖

- អ្នកនិពន្ធអាចកែ Post របស់ខ្លួន។
- អ្នកប្រើបាន login អាចមើល Post សាធារណៈ។
- Admin អាចមើល ឬកែ Post ទាំងអស់។

Voter ប្រមូលច្បាប់ authorization នៅកន្លែងតែមួយ ហើយអាចហៅពី controller, service ឬ template តាម authorization checker។

## ២. បង្កើត PostVoter

សន្មតថា `Post` មាន method `getAuthor()` និង `isPrivate()`; `User` ជា class សម្រាប់អ្នកប្រើដែលបាន login។ បង្កើត `src/Security/PostVoter.php`៖

```php
<?php

namespace App\Security;

use App\Entity\Post;
use App\Entity\User;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Vote;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;

class PostVoter extends Voter
{
    public const VIEW = 'POST_VIEW';
    public const EDIT = 'POST_EDIT';

    protected function supports(string $attribute, mixed $subject): bool
    {
        return in_array($attribute, [self::VIEW, self::EDIT], true)
            && $subject instanceof Post;
    }

    protected function voteOnAttribute(
        string $attribute,
        mixed $subject,
        TokenInterface $token,
        ?Vote $vote = null,
    ): bool {
        $user = $token->getUser();

        if (!$user instanceof User) {
            $vote?->addReason('អ្នកប្រើមិនទាន់បានចូលទេ។');

            return false;
        }

        /** @var Post $post */
        $post = $subject;

        if (in_array('ROLE_ADMIN', $user->getRoles(), true)) {
            return true;
        }

        return match ($attribute) {
            self::VIEW => !$post->isPrivate() || $post->getAuthor() === $user,
            self::EDIT => $post->getAuthor() === $user,
            default => false,
        };
    }
}
```

`supports()` ប្រាប់ Symfony ថា Voter នេះទទួលខុសត្រូវចំពោះ attributes និង subject ប្រភេទណា។ `voteOnAttribute()` ទទួល current user ពី security token ហើយត្រឡប់ `true` ដើម្បីអនុញ្ញាត ឬ `false` ដើម្បីបដិសេធ។ បើ project ប្រើ User class ឈ្មោះផ្សេង ត្រូវកែ type hint ឱ្យត្រូវ។

ក្នុង Symfony app ស្តង់ដារដែល configure service autowiring/autoconfiguration រួច Voter ក្នុង `src/` ត្រូវបានចុះឈ្មោះជាសេវាកម្មដោយស្វ័យប្រវត្តិ។ បើមិនដូច្នោះទេ ត្រូវចុះឈ្មោះ service និង tag `security.voter` តាមឯកសាររបស់ project។

## ៣. ហៅ Voter ពី Controller

ប្រើ `denyAccessUnlessGranted()`៖

```php
use App\Entity\Post;
use App\Security\PostVoter;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class PostController extends AbstractController
{
    #[Route('/posts/{id}/edit', name: 'post_edit', methods: ['GET', 'POST'])]
    public function edit(Post $post): Response
    {
        $this->denyAccessUnlessGranted(PostVoter::EDIT, $post);

        // បន្តបង្ហាញ ឬដំណើរការ form ក្រោយពិនិត្យសិទ្ធិ

        return new Response('អាចកែ Post នេះបាន។');
    }
}
```

Symfony ស្វែងរក Voter ដែលគាំទ្រ `POST_EDIT` និង `Post` object នេះ។ បើគ្មាន Voter ណាអនុញ្ញាត អ្នកប្រើនឹងទទួល Access Denied។

អាចប្រើ `#[IsGranted]` លើ method ផងដែរ៖

```php
use Symfony\Component\Security\Http\Attribute\IsGranted;

#[IsGranted(PostVoter::EDIT, subject: 'post')]
public function edit(Post $post): Response
{
    // ...
}
```

## ៤. ហៅ Voter ពី Twig

```twig
{% if is_granted('POST_EDIT', post) %}
    <a href="{{ path('post_edit', { id: post.id }) }}">កែ Post</a>
{% endif %}
```

ការពិនិត្យក្នុង Twig ជួយលាក់ link ដែលអ្នកប្រើមិនអាចប្រើបាន ប៉ុន្តែវាមិនជំនួសការពិនិត្យក្នុង Controller ទេ។ អ្នកប្រើអាចស្នើ URL ដោយផ្ទាល់បាន។

## ៥. របៀបសម្រេចសិទ្ធិ

នៅពេលកូដហៅ `isGranted()`, `denyAccessUnlessGranted()` ឬ access control Symfony ប្រមូលការបោះឆ្នោតពី Voters។ Voter ដែល `supports()` មិនត្រូវនឹង attribute/subject អាច abstain; Voter ដែលគាំទ្រ ហើយច្បាប់អនុញ្ញាត នឹងអនុម័ត។ Access decision strategy របស់ Symfony ប្រមូលលទ្ធផលទាំងនេះដើម្បីសម្រេចចុងក្រោយ។

បើមាន Voters ច្រើន ឬច្បាប់ស្មុគស្មាញ សូមរក្សា logic ឱ្យច្បាស់ ហើយពិនិត្យកុំឱ្យ Voter ពីរមានការសម្រេចផ្ទុយគ្នាដោយមិនដឹងខ្លួន។

## កំហុសដែលជួបញឹកញាប់

- **ភ្លេចផ្ទៀងផ្ទាត់ subject type**៖ ត្រូវពិនិត្យ `$subject instanceof Post` មុនប្រើវា។
- **អនុញ្ញាតអ្នកមិនទាន់ login ដោយចៃដន្យ**៖ ពិនិត្យថា `$token->getUser()` ជា User ពិត។
- **ប្រើ object ខុសក្នុង Controller**៖ Voter ទទួល object ត្រូវនឹង subject ដែលបានផ្ញើ។
- **លាក់ button ហើយមិនការពារ route**៖ ត្រូវពិនិត្យសិទ្ធិ server-side នៅគ្រប់ action ដែលការពារ។
- **សរសេរ role logic ផ្ទួនៗក្នុង Controller ច្រើន**៖ ផ្លាស់ទៅ Voter ដើម្បីកុំឱ្យច្បាប់ខុសគ្នាតាម route។
- **ប្រើ reason ដែលបង្ហាញព័ត៌មានផ្ទៃក្នុងដល់អ្នកប្រើ**៖ vote reason មានប្រយោជន៍សម្រាប់ debugging; កុំបង្ហាញវាដោយផ្ទាល់បើមានព័ត៌មានសម្ងាត់។

## លំហាត់

1. បន្ថែម `POST_DELETE` ហើយអនុញ្ញាតឱ្យ author ឬ admin លុប Post។
2. បង្កើត Controller action ដើម្បីមើល Post ដោយហៅ `POST_VIEW`។
3. លាក់ប៊ូតុង Edit ក្នុង Twig សម្រាប់អ្នកដែលគ្មានសិទ្ធិ។
4. សាកល្បង user ផ្សេងកែ Post ដែលមិនមែនជារបស់ខ្លួន ហើយធានាថាបាន 403។

## សំណួរត្រួតពិនិត្យ

1. តើ `supports()` ធ្វើអ្វី?
2. តើ Voter ទទួល current user ពីណា?
3. តើហេតុអ្វីត្រូវការទាំង Twig check និង Controller check?
4. តើអ្វីកំណត់ថា Voter មួយអនុវត្តចំពោះ subject ណា?

## ឯកសារយោងផ្លូវការ

- [Symfony 7.4: How to Use Voters to Check User Permissions](https://symfony.com/doc/7.4/security/voters.html)
- [Symfony 7.4: Security](https://symfony.com/doc/7.4/security.html)
