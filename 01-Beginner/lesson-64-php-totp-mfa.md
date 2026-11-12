# មេរៀនទី ៦៤៖ TOTP Multi-Factor Authentication ក្នុង PHP

## គោលបំណង
- យល់ពី TOTP និងការប្រើជា Second Factor។
- ប្រើ Library ដែលអនុវត្ត RFC 6238។
- រៀបចំ Enrolment, Verify, Rate Limit, Replay Protection និង Recovery Codes។

> TOTP Secret ត្រូវអាចយកមកប្រើវិញដើម្បីផ្ទៀងផ្ទាត់; Encrypt ពេលរក្សាទុក កុំ Hash ដូច Password។ ប្រើ HTTPS។

## ១. TOTP ជាអ្វី?

TOTP គណនាកូដខ្លីពី Shared Secret និងពេលបច្ចុប្បន្ន។ User និង Server ត្រូវមាន Secret ដូចគ្នា និង Clock ស្របគ្នា។ វាជា Possession Factor ប៉ុន្តែអាចត្រូវ Phishing ប្រសិនបើអ្នកប្រើបញ្ចូលកូដនៅទំព័រក្លែងក្លាយ។

កុំសរសេរ TOTP/HOTP Algorithm ឬ Base32 Encoding ផ្ទាល់។ ឧទាហរណ៍ Library OTPHP អនុវត្ត RFC 4226/6238៖

```bash
composer require spomky-labs/otphp symfony/clock
```

ពិនិត្យ Version Constraint និង Documentation របស់ Library Version ដែល Composer បានដំឡើង ព្រោះ API អាចខុសតាម Major Version។

## ២. Enrolment

Flow ដែលណែនាំ៖
1. User Authenticate ថ្មីៗ ឬបញ្ចូល Password ឡើងវិញ។
2. បង្កើត Secret ហើយរក្សាទុកជា Pending Secret ដែល Encrypt។
3. បង្ហាញ QR/Provisioning URI លើ HTTPS ទៅម្ចាស់ Account តែម្តង។
4. ស្នើឱ្យ User បញ្ចូល TOTP Code មួយ ដើម្បី Confirm។
5. ទើបបើក MFA និងបង្កើត Recovery Codes។

ឧទាហរណ៍បង្កើត Secret ដោយ API របស់ OTPHP៖

```php
<?php
use OTPHP\TOTP;
use Symfony\Component\Clock\NativeClock;

$clock = new NativeClock();
$totp = TOTP::generate($clock, 20);
$secret = $totp->getSecret();
$provisioningUri = $totp->getProvisioningUri();

// Encrypt $secret មុនរក្សាទុកជា Pending Secret។
```

កុំ Log Secret/URI ឬទុក QR នៅ Public Cache; Provisioning URI មាន Secret ដូចគ្នា។

## ៣. ផ្ទៀងផ្ទាត់ពេល Login

```php
<?php
use OTPHP\TOTP;
use Symfony\Component\Clock\NativeClock;

$code = $_POST['totp_code'] ?? null;
if (!is_string($code) || !preg_match('/^\d{6}$/', $code)) {
    http_response_code(400);
    exit('កូដមិនត្រឹមត្រូវ។');
}

$secret = decryptStoredTotpSecret($encryptedSecret);
$totp = TOTP::createFromSecret($secret, new NativeClock());

if (!$totp->verify($code)) {
    recordTotpFailure($userId);
    http_response_code(401);
    exit('ការផ្ទៀងផ្ទាត់មិនបានជោគជ័យ។');
}

// អនុវត្ត Atomic Replay Check មុនបញ្ចប់ Login។
markTotpStepAsUsed($userId, $code);
completeSignIn($userId);
```

ឧទាហរណ៍បង្ហាញ Flow ប៉ុណ្ណោះ៖ OTPHP មិនចងចាំ Code ដែលបានប្រើរួចទេ។ App ត្រូវរក្សា Time Step/Code ដែលជោគជ័យ និងបដិសេធការប្រើស្ទួន ដោយមាន Atomic Update ដើម្បីទប់ Request ស្របគ្នា។ កំណត់ Clock Window តូចសម្រាប់ Clock Drift។

## ៤. សុវត្ថិភាព Secret និង Recovery

- Encrypt TOTP Secret នៅ Database ដោយ Key ដែលរក្សាទុកក្រៅ Database/Source Code។
- រក្សា Key Version និងមាន Rotation/Recovery Plan។
- Rate Limit ការសាក TOTP តាម Account និងប្រភព; កុំអនុញ្ញាតឱ្យទាយគ្មានកំណត់។
- បង្កើត Recovery Codes ចៃដន្យ, រក្សាទុកតែ Hash និងប្រើបានម្តង។
- បង្ហាញ Recovery Codes តែម្តង ហើយផ្ញើ Notification ពេលប្រើ។
- ការបិទ MFA/ប្តូរ Secret ត្រូវការ Re-authentication និង Audit Event។
- Account Recovery ត្រូវមានសុវត្ថិភាពស្មើ Login។

## ៥. កំហុសដែលជួបញឹកញាប់

- សរសេរ TOTP Algorithm ផ្ទាល់។
- Hash Secret ហើយមិនអាចគណនា Code វិញ។
- បើក MFA មុន Confirm Code ដំបូង។
- មិន Rate Limit ឬមិនទប់ Code Replay។
- បង្ហាញ QR/URI ឡើងវិញដោយគ្មាន Re-authentication។
- ប្រើ Recovery Flow ខ្សោយជាង Login។

## ៦. លំហាត់

1. រៀបចំ MFA State: Pending → Confirmed → Enabled។
2. Encrypt Secret និងកុំ Log Provisioning URI។
3. បន្ថែម Rate Limit និង Atomic Replay Protection។
4. បង្កើត Hashed, Single-Use Recovery Codes។
5. សាកល្បងដោយ Inject Clock ដើម្បីឱ្យ Test អាចធ្វើឡើងវិញបាន។

## ឯកសារយោង

- [RFC 6238: TOTP](https://www.rfc-editor.org/rfc/rfc6238.html)
- [OWASP Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [OTPHP: Installation and Clock](https://docs.spomky-labs.com/otphp/latest/the-project/installation/)
- [OTPHP: Security Recommendations](https://docs.spomky-labs.com/otphp/latest/the-project/security-recommendations/)
- [មេរៀនទី ៦៣៖ Encryption និង PHP Sodium](lesson-63-php-sodium-encryption.md)
