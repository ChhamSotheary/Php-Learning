# មេរៀនទី ៣២៖ HTTPS និង Security Headers សម្រាប់ PHP

## គោលបំណងមេរៀន

បន្ទាប់ពីរៀនចប់ អ្នកនឹងអាច៖

- ពន្យល់ពីមូលហេតុដែល HTTPS ត្រូវការសម្រាប់ Login និងទិន្នន័យទាំងអស់។
- កំណត់ Response Headers សម្រាប់ការការពារបន្ថែម។
- បង្កើត Content Security Policy (CSP) ដោយចាប់ផ្តើមពី Policy ដែលមិនបំបែក Site។
- ប្រើ HSTS ដោយយល់ពីលក្ខខណ្ឌ និងហានិភ័យនៃការកំណត់វា។
- ដឹងថា Security Headers ជាស្រទាប់បន្ថែម មិនជំនួសការកែ Vulnerability ក្នុង Code ទេ។

> តម្រូវការ៖ PHP 8+ និង HTTPS ត្រូវបានកំណត់ត្រឹមត្រូវនៅ Web Server/Load Balancer។ ឧទាហរណ៍ Headers អាចត្រូវកែតាម App និង Infrastructure របស់អ្នក។

## ១. HTTPS ជាអ្វី ហេតុអ្វីត្រូវប្រើ?

HTTPS ប្រើ TLS ដើម្បីការពារទំនាក់ទំនងរវាង Browser និង Server ពីការលួចមើល ឬកែប្រែតាមផ្លូវ។ ប្រើ HTTPS គ្រប់ Route រួមទាំង Login, API និង Static Content; កុំផ្ញើ Password ឬ Session Cookie លើ HTTP។

ត្រូវដំឡើង Certificate ត្រឹមត្រូវ និងធ្វើ Redirect HTTP ទៅ HTTPS នៅ Web Server ឬ Reverse Proxy។ បើ TLS ត្រូវបញ្ចប់នៅ Proxy ត្រូវកំណត់ Trust Proxy ឱ្យច្បាស់ មិនទុកចិត្ត `X-Forwarded-Proto` ពី Client ណាមួយដោយផ្ទាល់ទេ។

## ២. បន្ថែម Headers ក្នុង PHP

ត្រូវផ្ញើ Headers មុន Output ដូចជា HTML ឬ Whitespace៖

```php
<?php
// បញ្ជូន JSON API
header('Content-Type: application/json; charset=utf-8');
header('X-Content-Type-Options: nosniff');
header('Referrer-Policy: strict-origin-when-cross-origin');
header('X-Frame-Options: DENY'); // សម្រាប់ Browser ចាស់; CSP frame-ancestors គឺជាជម្រើសទំនើប

echo json_encode(['status' => 'ok'], JSON_UNESCAPED_UNICODE);
```

Headers ទាំងនេះមានគោលបំណងខុសៗគ្នា៖

- `Content-Type` បញ្ជាក់ប្រភេទ Response; កុំឱ្យ Browser ស្មានប្រភេទផ្សេង។
- `X-Content-Type-Options: nosniff` ជួយបិទ MIME sniffing។
- `Referrer-Policy` កំណត់ព័ត៌មាន Referrer ដែល Browser ផ្ញើទៅកន្លែងផ្សេង។
- `X-Frame-Options` កំណត់ការបើក Page ក្នុង Frame សម្រាប់ការការពារ Clickjacking នៅ Browser ចាស់។ គេហទំព័រដែលត្រូវបើកក្នុង Frame ត្រូវកំណត់ Policy តាមតម្រូវការ។

## ៣. Content Security Policy (CSP)

CSP ប្រាប់ Browser ថា Resource ដូចជា Script និង Image អាច Load ពីប្រភពណាខ្លះ។ វាជួយកាត់បន្ថយផលប៉ះពាល់ XSS ប៉ុន្តែមិនជំនួស Output Escaping ទេ។

Policy តឹងខាងក្រោមគឺជាគំរូសម្រាប់ API/ទំព័រដែលមិន Load Resource ពីក្រៅ៖

```php
header("Content-Security-Policy: default-src 'none'; frame-ancestors 'none'; base-uri 'none'");
```

សម្រាប់ HTML Site ដែលប្រើ CSS/Image ត្រូវកំណត់ Source តាម Resource ពិត។ ឧទាហរណ៍ដំបូងសម្រាប់សាកល្បង Report-Only៖

```php
header("Content-Security-Policy-Report-Only: default-src 'self'; object-src 'none'; base-uri 'self'; frame-ancestors 'none'");
```

Report-Only មិនទប់ Resource ទេ; វាអនុញ្ញាតឱ្យសង្កេត Policy មុន Enforce។ មុនដាក់ CSP ពិត ត្រូវពិនិត្យ Inline Script, Inline Style, CDN, Image និង Form Endpoint។ កុំប្រើ `'unsafe-inline'` ដោយមិនចាំបាច់; Nonce ឬ Hash ត្រូវបង្កើតតាម Request និងអនុវត្តឱ្យត្រឹមត្រូវ។

## ៤. HSTS សម្រាប់ HTTPS

Strict-Transport-Security (HSTS) ប្រាប់ Browser ដែលបានទទួល Header តាម HTTPS រួច ឱ្យប្រើ HTTPS សម្រាប់ Domain នោះរយៈពេលកំណត់៖

```php
// ផ្ញើតែពេល Request នេះពិតជាមកតាម HTTPS ដែលបានបញ្ជាក់ពី Trusted Proxy ឬ Web Server។
header('Strict-Transport-Security: max-age=31536000');
```

ចាប់ផ្តើមដោយ `max-age` ខ្លីក្នុងការសាកល្បង ហើយបង្កើនពេលប្រាកដថា HTTPS ដំណើរការគ្រប់ Subdomain/Route ដែលពាក់ព័ន្ធ។ `includeSubDomains` មានឥទ្ធិពលទៅលើ Subdomain ទាំងអស់; កុំដាក់វា ឬចុះ Preload ដោយមិនបានរៀបចំ HTTPS លើគ្រប់ Hostname។ Browser មិនអនុវត្ត HSTS ដែលទទួលតាម HTTP ទេ។

ជាទូទៅ HSTS និង HTTP→HTTPS Redirect ត្រូវកំណត់នៅ Web Server ឬ Load Balancer ដើម្បីគ្របដណ្តប់គ្រប់ Response។

## ៥. រៀបចំ Headers នៅកន្លែងតែមួយ

បង្កើត Bootstrap ឬ Middleware មួយសម្រាប់ Header ដែលសមនឹង Response របស់ App៖

```php
<?php
function sendCommonSecurityHeaders(): void
{
    header('X-Content-Type-Options: nosniff');
    header('Referrer-Policy: strict-origin-when-cross-origin');
    header('X-Frame-Options: DENY');

    // CSP និង HSTS ត្រូវរៀបចំតាម Resource និង HTTPS/Proxy Configuration ពិត។
}
```

ហៅវាមុន Output នៅក្នុង Front Controller ឬ Middleware។ ជៀសវាងផ្ញើ Header ស្ទួន ឬ Policy ខុសគ្នារវាង Route ដោយគ្មានចេតនា។ អាចកំណត់ Headers នៅ Web Server ជំនួស PHP ដើម្បីធានាថាវាគ្របដណ្តប់ Error Response ផងដែរ។

## ៦. សាកល្បងនិងត្រួតពិនិត្យ

ប្រើ Developer Tools → Network ដើម្បីមើល Response Headers នៅ Route សំខាន់ៗ៖ Home, Login, API, Error និង Redirect។ ពិនិត្យថា៖

- Response មាន HTTPS និង Certificate ត្រឹមត្រូវ។
- Cookie សម័យ Login មាន `Secure`, `HttpOnly`, និង `SameSite` សមស្រប។
- CSP មិនបំបែកមុខងារធម្មតា ហើយមិនអនុញ្ញាតប្រភពធំទូលាយដោយមិនចាំបាច់។
- HSTS មិនត្រូវបានផ្ញើលើ HTTP ឬពី Forwarded Header ដែល Client អាចក្លែងបាន។
- Headers មាននៅលើ Error Responses ផងដែរ ប្រសិនបើកំណត់នៅ Proxy/Web Server។

## ៧. កំហុសដែលជួបញឹកញាប់

- គិតថា Header តែមួយអាចធ្វើឱ្យ App មានសុវត្ថិភាព ទោះមាន SQL Injection ឬ XSS ក៏ដោយ។
- ដាក់ HSTS `includeSubDomains` មុនពិនិត្យថា Subdomain ទាំងអស់មាន HTTPS។
- ដាក់ទៅ Production CSP ដែលមាន `default-src 'none'` ដោយមិនពិនិត្យ Resource របស់ Site។
- ទុកចិត្ត `X-Forwarded-Proto` ដែលភ្ជាប់មកពី Internet ដោយគ្មាន Trusted Proxy។
- ផ្ញើ Headers ក្រោយមាន Output រួច ហើយ PHP មិនអាចកែ Response Header បាន។

## ៨. លំហាត់

1. បន្ថែម `nosniff` និង Referrer Policy ទៅ JSON Endpoint។
2. កំណត់ CSP Report-Only សម្រាប់ Page ដែលប្រើតែ Resource ក្នុង Domain ដូចគ្នា។
3. សរសេរបញ្ជី Subdomain និង Resource ដែលត្រូវប្រើ មុនដាក់ HSTS/CSP ទៅ Production។
4. ពិនិត្យ Headers នៅ Login, API, Redirect និង Error Response។
5. ពន្យល់ថាហេតុអ្វី HSTS មិនត្រូវទុកចិត្តលើ HTTP ឬ Proxy ដែលមិនបានកំណត់ Trust។

## ៩. ចម្លើយណែនាំខ្លី

- HTTPS/TLS គួរគ្របដណ្តប់គ្រប់ទំព័រ និង API។
- ចាប់ផ្តើម CSP ដោយ Report-Only ហើយកែតាម Resource ដែលបានប្រើ មុន Enforce។
- ផ្ញើ HSTS តែបន្ទាប់ពី HTTPS ត្រឹមត្រូវ និងស្ថិតក្រោមការគ្រប់គ្រងរបស់អ្នក។
- Security Headers ជាស្រទាប់ការពារបន្ថែម; នៅតែត្រូវ Validate Input, Escape Output, ប្រើ Prepared Statements និង CSRF Protection។

## ឯកសារយោង

- [OWASP HTTP Headers Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)
- [MDN: Strict-Transport-Security](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
- [MDN: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [PHP: header()](https://www.php.net/manual/en/function.header.php)
