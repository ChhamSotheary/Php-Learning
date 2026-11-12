# មេរៀនទី ៣៧៖ Role-Based Access Control និងការការពារ Resource ក្នុង PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Authentication និង Authorization។
- ប្រើ Role/Permission ដើម្បីកំណត់សកម្មភាពដែលអនុញ្ញាត។
- ពិនិត្យសិទ្ធិនៅ Server រាល់ Request ដោយ Default គឺ Deny។
- ការពារ Insecure Direct Object Reference (IDOR) ដោយពិនិត្យ Owner/Access លើ Record។
- រក្សា Authorization Rules ឱ្យនៅកណ្ដាល និងអាចពិនិត្យបាន។

## ១. Authentication និង Authorization

**Authentication** ឆ្លើយថា “អ្នកនេះជានរណា?” ដូចជា ពិនិត្យ Session ឬ API Token។  
**Authorization** ឆ្លើយថា “អ្នកនេះអាចធ្វើអ្វីលើ Resource នេះ?” ដូចជា អនុញ្ញាតឱ្យ Owner កែ Post របស់ខ្លួន។

ការដែល User បាន Login មិនមានន័យថាគេអាចធ្វើគ្រប់ Action បានទេ។ ត្រូវពិនិត្យសិទ្ធិលើរាល់ Route និង Request ដែលប៉ះពាល់ទិន្នន័យ។

## ២. គោលការណ៍សម្រាប់ Authorization

- **Deny by default**៖ បើគ្មាន Rule អនុញ្ញាតច្បាស់ ត្រូវបដិសេធ។
- **Least privilege**៖ ផ្តល់តែសិទ្ធិដែលត្រូវការសម្រាប់ការងារ។
- ពិនិត្យសិទ្ធិរាល់ Request នៅ Server; កុំទុកចិត្តលើ Button ដែលលាក់នៅ UI។
- កុំទទួល Role ពី Form, Query String ឬ Request Header របស់អ្នកប្រើ។
- ប្រើ Rule ឱ្យស្របនឹង Business Logic និង Resource។
- កត់ត្រា Access Denial ដែលពាក់ព័ន្ធ ដោយកុំ Log Token/Secret។

## ៣. ពិនិត្យ Login និង Role

ឧទាហរណ៍ Helper សម្រាប់ Admin Page៖

```php
<?php
function requireAuthenticatedUser(): array
{
    $userId = $_SESSION['user_id'] ?? null;

    if (!is_int($userId) && !ctype_digit((string) $userId)) {
        http_response_code(401);
        exit('សូម Sign In ជាមុន។');
    }

    return [
        'id' => (int) $userId,
        'role' => $_SESSION['role'] ?? 'user',
    ];
}

function requireRole(array $user, string $requiredRole): void
{
    if (($user['role'] ?? null) !== $requiredRole) {
        http_response_code(403);
        exit('អ្នកមិនមានសិទ្ធិអនុវត្តសកម្មភាពនេះទេ។');
    }
}
```

គំរូនេះសម្រាប់បង្ហាញគំនិតតែប៉ុណ្ណោះ។ ក្នុងកម្មវិធីពិត ត្រូវប្រាកដថា Role នៅ Session បានមកពីទិន្នន័យដែល Server ផ្ទៀងផ្ទាត់ ហើយត្រូវគិតពីការផ្លាស់ប្តូរ Role/ការបិទគណនី។ សម្រាប់សិទ្ធិសំខាន់ អាច Load Role ពី Database ឬប្រើ Version/Revocation Strategy ដើម្បីកុំឱ្យសិទ្ធិចាស់នៅជាប់យូរ។

## ៤. Permission ល្អជាងការពិនិត្យ Role គ្រប់កន្លែង

កម្មវិធីធំអាចកំណត់ Permission ហើយចាត់ Role ទៅ Permission៖

```php
<?php
$rolePermissions = [
    'admin' => ['posts.read', 'posts.create', 'posts.update.any', 'posts.delete.any'],
    'editor' => ['posts.read', 'posts.create', 'posts.update.own'],
    'user' => ['posts.read', 'posts.create'],
];

function can(array $user, string $permission, array $rolePermissions): bool
{
    $role = $user['role'] ?? null;

    return is_string($role)
        && in_array($permission, $rolePermissions[$role] ?? [], true);
}

if (!can($user, 'posts.create', $rolePermissions)) {
    http_response_code(403);
    exit('អ្នកមិនមានសិទ្ធិបង្កើតអត្ថបទទេ។');
}
```

មិនគួរប្រើ Permission ពី Client ឬ Role ដែល User កំណត់ដោយខ្លួនឯងឡើយ។ ប្រព័ន្ធ Role/Permission អាចរក្សានៅ Database ឬ Configuration ដែល Server គ្រប់គ្រង។

## ៥. ការពារ IDOR ដោយពិនិត្យ Ownership

កុំគិតថា User មានសិទ្ធិលើ Record ព្រោះគេផ្ញើ ID មក។ Query ត្រូវពាក់ព័ន្ធ User ID ដែលបាន Authenticate៖

```php
<?php
$stmt = $pdo->prepare(
    'SELECT id, owner_id, title, body
     FROM posts
     WHERE id = :post_id
     LIMIT 1'
);
$stmt->execute(['post_id' => $postId]);
$post = $stmt->fetch(PDO::FETCH_ASSOC);

if ($post === false) {
    http_response_code(404);
    exit('រកមិនឃើញអត្ថបទទេ។');
}

$isOwner = (int) $post['owner_id'] === (int) $user['id'];
$isAdmin = ($user['role'] ?? null) === 'admin';

if (!$isOwner && !$isAdmin) {
    http_response_code(403);
    exit('អ្នកមិនមានសិទ្ធិមើលអត្ថបទនេះទេ។');
}
```

អាចបញ្ចូល Owner Check ទៅក្នុង Query ដើម្បីបន្ថយការភ្លេចពិនិត្យ៖

```php
$stmt = $pdo->prepare(
    'UPDATE posts
     SET title = :title, body = :body
     WHERE id = :post_id AND owner_id = :owner_id'
);
$stmt->execute([
    'title' => $title,
    'body' => $body,
    'post_id' => $postId,
    'owner_id' => $user['id'],
]);

if ($stmt->rowCount() === 0) {
    // Record អាចមិនមាន ឬ User គ្មានសិទ្ធិ; កំណត់ Response តាមគោលការណ៍ App។
}
```

Admin ដែលអាចកែរបស់អ្នកដទៃ ត្រូវមាន Code Path/Permission ច្បាស់លាស់ និងមាន Audit Log សមស្រប។

## ៦. HTTP Status និងការបង្ហាញ Error

- `401 Unauthorized`៖ មិនទាន់ Authenticate ឬ Credential មិនត្រឹមត្រូវ។
- `403 Forbidden`៖ បាន Authenticate ប៉ុន្តែមិនអនុញ្ញាតសកម្មភាព។
- `404 Not Found`៖ Resource មិនមាន។ App ខ្លះជ្រើសប្រើ 404 សម្រាប់លាក់ថា Resource មាន ប៉ុន្តែ User មិនអាចចូលបាន; ត្រូវប្រើឱ្យស្របគ្នា។

កុំបង្ហាញព័ត៌មានខាងក្នុង ឬទិន្នន័យរបស់ Owner ផ្សេងក្នុង Error Response។

## ៧. កំហុសដែលជួបញឹកញាប់

- លាក់ Admin Button ប៉ុន្តែ Endpoint មិនពិនិត្យ Role។
- ពិនិត្យតែថា User បាន Login ដោយមិនពិនិត្យ Ownership។
- ទុកឱ្យ User ផ្ញើ `role=admin` ក្នុង Form ហើយ Update ទៅ Database។
- មាន Rule អនុញ្ញាតដោយ Default ពេលមិនស្គាល់ Role។
- ពិនិត្យសិទ្ធិតែម្តងពេល Login ទោះ Role អាចផ្លាស់ប្តូរពេលក្រោយ។
- ប្រើ ID ដែលពិបាកទាយជំនួស Authorization Check។ UUID ក៏ត្រូវការពិនិត្យសិទ្ធិដែរ។

## ៨. លំហាត់

1. បង្កើត Permission `posts.create`, `posts.update.own`, `posts.update.any`។
2. អនុញ្ញាត Editor កែតែ Post របស់ខ្លួន និង Admin កែ Post ទាំងអស់។
3. សរសេរ Endpoint Update ដែលពិនិត្យ User ID និង Ownership នៅ Server។
4. ពន្យល់ពីភាពខុសគ្នារវាង 401 និង 403។
5. ស្វែងរក Route ក្នុង Project ដែល Login មាន តែ Authorization Check ខ្វះ។

## ៩. ចម្លើយណែនាំខ្លី

- Authentication គឺបញ្ជាក់អត្តសញ្ញាណ; Authorization គឺកំណត់សិទ្ធិលើ Action/Resource។
- Default គឺ Deny ហើយពិនិត្យរាល់ Request។
- Owner ID ត្រូវមកពី Session/Token ដែលបានផ្ទៀងផ្ទាត់ មិនមែនពី Form។
- Role/Permission ត្រូវបានកំណត់ដោយ Server និងមានច្បាប់ច្បាស់។
- កុំចាត់ UUID ឬ ID លាក់ជាការការពារសិទ្ធិ។

## ឯកសារយោង

- [OWASP Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
- [OWASP IDOR Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [មេរៀនទី ២៧៖ Authentication និង Authorization សម្រាប់ API](lesson-27-api-authentication-authorization.md)
