# មេរៀនទី ៦៣៖ Encryption និង PHP Sodium

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Hashing និង Encryption។
- ប្រើ Authenticated Encryption ដោយ PHP Sodium។
- រក្សា Key ក្រៅ Source Code និងគ្រប់គ្រង Key Rotation។
- ប្រើ Nonce មិនស្ទួន និង Associated Data តាមតម្រូវការ។
- Validate Ciphertext និងដោះស្រាយការឌិគ្រីបដែលបរាជ័យដោយសុវត្ថិភាព។

> ឧទាហរណ៍ប្រើ XChaCha20-Poly1305 AEAD របស់ Sodium។ ត្រូវមាន Sodium Extension ដែលមានក្នុង PHP builds ទំនើបជាច្រើន។ អ៊ិនគ្រីបទិន្នន័យតែពេលមានតម្រូវការពិត; ជ្រើសរើសថាតើ Data គួរតែ Encrypt, Hash ឬមិនរក្សាទុក។

## ១. Hashing និង Encryption

- **Hashing** ជាការបម្លែងមួយទិស; ប្រើសម្រាប់ Password/Token Verification ដែលមិនចាំបាច់យកតម្លៃដើមមកវិញ។
- **Encryption** អាចឌិគ្រីបត្រឡប់បាន បើមាន Key; ប្រើសម្រាប់ទិន្នន័យដែលត្រូវរក្សា និងត្រូវអានវិញ។
- **Authenticated Encryption (AEAD)** ផ្តល់ Confidentiality និង Integrity; បើ Ciphertext ត្រូវបានកែប្រែ Decrypt នឹងបរាជ័យ។

កុំ Encrypt Password ដើម្បី Login—ប្រើ `password_hash()` និង `password_verify()`។ កុំបង្កើត Algorithm ផ្ទាល់ខ្លួន ឬប្រើ Encoding ដូចជា Base64 ជំនួស Encryption។

## ២. Key និង Nonce

សម្រាប់ XChaCha20-Poly1305៖

- Key មានទំហំកំណត់ដោយ `SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_KEYBYTES`។
- Nonce មានទំហំកំណត់ដោយ `SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES`។
- បង្កើត Nonce ចៃដន្យដោយ `random_bytes()` សម្រាប់ Ciphertext នីមួយៗ។
- រក្សា Key ក្នុង Secret Manager/Environment ដែលមាន Access Control; កុំ Commit Key ទៅ Git។
- Key តែមួយត្រូវប្រើជាមួយ Algorithm ដែលត្រូវគ្នា និងមានផែនការ Rotate/Version។

បង្កើត Key ម្តងនៅ Setup តាម Secret Management Process របស់អ្នក៖

```php
<?php
$key = sodium_crypto_aead_xchacha20poly1305_ietf_keygen();
echo base64_encode($key);
```

ដាក់លទ្ធផលទៅ Secret Store ដែលបានការពារ។ កុំរត់ Keygen រាល់ Request ព្រោះទិន្នន័យចាស់នឹងមិនអាច Decrypt ដោយ Key ថ្មីបាន។

## ៣. អ៊ិនគ្រីប និងបង្កើត Envelope

ឧទាហរណ៍នេះប្រើ Associated Data ដើម្បីភ្ជាប់ Ciphertext ទៅ Context ដូចជា Record ID។ Associated Data មិនត្រូវបាន Encrypt ប៉ុន្តែត្រូវបាន Authenticate៖

```php
<?php
function encryptionKey(): string
{
    $encoded = getenv('APP_ENCRYPTION_KEY');

    if (!is_string($encoded) || $encoded === '') {
        throw new RuntimeException('Encryption key is not configured.');
    }

    $key = base64_decode($encoded, true);

    if (
        $key === false
        || strlen($key) !== SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_KEYBYTES
    ) {
        throw new RuntimeException('Encryption key has an invalid format.');
    }

    return $key;
}

function encryptValue(string $plaintext, string $recordContext): string
{
    $key = encryptionKey();
    $nonce = random_bytes(
        SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES
    );

    $ciphertext = sodium_crypto_aead_xchacha20poly1305_ietf_encrypt(
        $plaintext,
        $recordContext,
        $nonce,
        $key
    );

    // រក្សា Version, Nonce និង Ciphertext ជា Base64 Envelope។
    $envelope = "\x01" . $nonce . $ciphertext;

    return base64_encode($envelope);
}
```

`recordContext` ត្រូវបង្កើតនៅ Server តាមទម្រង់ស្ថិរភាព ដូចជា `customer:123:tax_id:v1`; កុំយក Context ពី User ដោយផ្ទាល់។ Base64 នៅទីនេះគ្រាន់តែជាទម្រង់រក្សា Binary Data ក្នុង Text Column ប៉ុណ្ណោះ។

## ៤. ឌិគ្រីប និងពិនិត្យ Integrity

```php
<?php
function decryptValue(string $encodedEnvelope, string $recordContext): string
{
    $envelope = base64_decode($encodedEnvelope, true);

    if ($envelope === false) {
        throw new RuntimeException('Encrypted value is invalid.');
    }

    $version = ord($envelope[0] ?? "\x00");
    $nonceBytes = SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_NPUBBYTES;
    $tagBytes = SODIUM_CRYPTO_AEAD_XCHACHA20POLY1305_IETF_ABYTES;

    if (
        $version !== 1
        || strlen($envelope) < 1 + $nonceBytes + $tagBytes
    ) {
        throw new RuntimeException('Encrypted value is invalid.');
    }

    $nonce = substr($envelope, 1, $nonceBytes);
    $ciphertext = substr($envelope, 1 + $nonceBytes);
    $key = encryptionKey();

    $plaintext = sodium_crypto_aead_xchacha20poly1305_ietf_decrypt(
        $ciphertext,
        $recordContext,
        $nonce,
        $key
    );

    if ($plaintext === false) {
        throw new RuntimeException('Unable to decrypt encrypted value.');
    }

    return $plaintext;
}
```

បើ Ciphertext, Key ឬ Associated Data មិនត្រូវគ្នា Decrypt ត្រឡប់ `false`។ កុំបង្ហាញមូលហេតុលម្អិតទៅ Client; Log តែ Error Code/Context ដែលមិនមានទិន្នន័យរសើប។

## ៥. រក្សាទុក និង Key Rotation

ក្នុង Database រក្សា Envelope និង Key Version/Identifier ដើម្បីដឹងថាត្រូវប្រើ Key ណាសម្រាប់ Decrypt។ កុំរក្សា Key នៅជិត Ciphertext ដោយគ្មានការការពារបន្ថែម។

សម្រាប់ Rotation៖

1. បង្កើត Key ថ្មី និងកំណត់ Version/Identifier។
2. Encrypt តម្លៃថ្មីដោយ Key ថ្មី។
3. Decrypt ទិន្នន័យចាស់ដោយ Key Version ចាស់ ហើយ Re-encrypt តាមផែនការ។
4. ផ្ទៀងផ្ទាត់ Migration/Backup និងលទ្ធភាព Rollback។
5. បិទ Key ចាស់ក្រោយទិន្នន័យទាំងអស់បានផ្លាស់ និងគ្មាន Service ត្រូវការវាទៀត។

ការបាត់ Key អាចធ្វើឱ្យទិន្នន័យមិនអាចស្តារបាន។ ការលេចធ្លាយ Key ត្រូវធ្វើ Revocation/Rotation និងវាយតម្លៃទិន្នន័យដែលបាន Encrypt ជាមួយវា។

## ៦. អ្វីដែលមិនត្រូវអ៊ិនគ្រីបដោយស្វ័យប្រវត្តិ

- កុំ Encrypt Password សម្រាប់ផ្ទៀងផ្ទាត់ Login; ប្រើ Password Hash។
- កុំបង្កើត Key ពី Password ដោយផ្ទាល់; ត្រូវមាន Key Derivation ដែលសមស្រប ប្រសិនបើ Use Case ទាមទារ។
- កុំ Encrypt Field បើ App ត្រូវ Search/Sort វាដោយផ្ទាល់ ដោយមិនយល់ពី Trade-off។
- កុំ Log Plaintext, Key, Nonce+Ciphertext Envelope ប្រសិនបើមិនចាំបាច់។
- កុំសន្មតថា Encryption ជំនួស Access Control, HTTPS, Backup Protection ឬ Key Management។

## ៧. កំហុសដែលជួបញឹកញាប់

- ប្រើ Key ថេរដាក់ក្នុង Source Code។
- ប្រើ Nonce ដដែលជាមួយ Key ដដែល។
- មិនផ្ទៀងផ្ទាត់ Authentication Tag/Decrypt Result។
- បាត់ Key ឬមិនមាន Key Version/Rotation Plan។
- ច្រឡំ Base64 ជា Encryption។
- Encrypt Password ជំនួស Password Hashing។
- បង្ហាញ Decryption Error ឬ Plaintext ក្នុង Log/Response។

## ៨. លំហាត់

1. បង្កើត Sodium Key ម្តង និងរក្សាទុកជា Base64 ក្នុង Secret Store សម្រាប់ Local Test។
2. Encrypt/Decrypt String ដោយ XChaCha20-Poly1305 និង Associated Data។
3. កែ Ciphertext ឬ Associated Data ហើយបញ្ជាក់ថា Decryption បរាជ័យ។
4. បន្ថែម Version Byte/Key ID ទៅ Envelope។
5. រចនា Key Rotation Plan ដែលអាច Decrypt ទិន្នន័យចាស់បានរហូតដល់ Re-encrypt ចប់។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ Sodium AEAD ដែលមាន API ស្តង់ដារ; កុំបង្កើត Crypto Algorithm ផ្ទាល់។
- បង្កើត Nonce ថ្មីដោយ `random_bytes()` សម្រាប់ Ciphertext នីមួយៗ។
- Key ត្រូវមកពី Secret Management និងមិន Commit ទៅ Repository។
- កុំច្រឡំ Hashing, Encryption និង Encoding។
- រក្សា Algorithm/Key Version និងមានផែនការ Backup/Rotation។

## ឯកសារយោង

- [PHP: XChaCha20-Poly1305 Encrypt](https://www.php.net/manual/en/function.sodium-crypto-aead-xchacha20poly1305-ietf-encrypt.php)
- [PHP: XChaCha20-Poly1305 Decrypt](https://www.php.net/manual/en/function.sodium-crypto-aead-xchacha20poly1305-ietf-decrypt.php)
- [PHP: Sodium Extension](https://www.php.net/manual/en/book.sodium.php)
- [មេរៀនទី ៣៣៖ Configuration និង Secrets ក្នុង PHP](lesson-33-php-configuration-secrets.md)
