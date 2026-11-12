# មេរៀនទី ៦២៖ Internationalization និង Localization ក្នុង PHP

## គោលបំណងមេរៀន

ក្រោយរៀនចប់ អ្នកនឹងអាច៖

- បែងចែក Internationalization (i18n) និង Localization (l10n)។
- ប្រើ PHP Intl Extension ដើម្បី Format លេខ, រូបិយប័ណ្ណ និងកាលបរិច្ឆេទ។
- កំណត់ Locale និង Time Zone ឱ្យច្បាស់។
- បង្កើតសារតាមភាសា និងចំនួន (Plural)។
- ជៀសវាងប្រើតម្លៃ Format សម្រាប់គណនាផ្ទៃក្នុង ឬច្រឡំ Locale ជាមួយ Encoding។

> តម្រូវការ៖ PHP 8+ និង Intl Extension ដែលប្រើ ICU Library។ លទ្ធផល Format អាចខុសតិចតួចតាម ICU/Locale Data Version។

## ១. i18n និង l10n ជាអ្វី?

- **Internationalization (i18n)**៖ រៀបចំកម្មវិធីឱ្យគាំទ្រភាសា/តំបន់ជាច្រើន។
- **Localization (l10n)**៖ កំណត់អត្ថបទ, លេខ, កាលបរិច្ឆេទ និងក្បួនតាម Locale ជាក់លាក់។

Locale មិនមែនត្រឹមតែបកប្រែអត្ថបទទេ។ វាប៉ះពាល់លំដាប់ថ្ងៃ/ខែ, សញ្ញាទសភាគ, ការដាក់ខ្ទង់ពាន់, រូបិយប័ណ្ណ និងទម្រង់ពាក្យតាមចំនួន។

## ២. ពិនិត្យ Intl Extension និង Allowlist Locale

```php
<?php
if (!extension_loaded('intl')) {
    throw new RuntimeException('The PHP intl extension is required.');
}

$supportedLocales = ['km-KH', 'en-US', 'th-TH'];
$requestedLocale = $_GET['lang'] ?? 'km-KH';

if (
    !is_string($requestedLocale)
    || !in_array($requestedLocale, $supportedLocales, true)
) {
    $requestedLocale = 'km-KH';
}
```

Locale គួរត្រូវបានកំណត់ពី User Preference ដែលបាន Validate ឬគណនាតាម Policy របស់ App។ កុំទទួល Locale String ណាមួយពី Client ដោយគ្មានកំណត់។ អាចរក្សា User Locale ក្នុង Profile ឬប្រើ Accept-Language ជា Hint ប៉ុណ្ណោះ។

## ៣. Format លេខ និងរូបិយប័ណ្ណ

ប្រើ `NumberFormatter` សម្រាប់បង្ហាញលេខតាម Locale៖

```php
<?php
$formatter = new NumberFormatter('km-KH', NumberFormatter::DECIMAL);
$formatted = $formatter->format(1234567.89);

if ($formatted === false) {
    throw new RuntimeException('Unable to format number.');
}

echo $formatted;
```

Format រូបិយប័ណ្ណសម្រាប់ Display៖

```php
<?php
$moneyFormatter = new NumberFormatter('km-KH', NumberFormatter::CURRENCY);
$formattedMoney = $moneyFormatter->formatCurrency(19.95, 'USD');

if ($formattedMoney === false) {
    throw new RuntimeException('Unable to format currency.');
}

echo $formattedMoney;
```

កុំរក្សា String ដែល Format រួចដូចជា `$19.95` ជាតម្លៃសម្រាប់គណនា។ រក្សាចំនួនក្នុង Unit ដែលមាន Precision ច្បាស់ (ឧ. minor units/សេន) និងកូដ Currency ដាច់ដោយឡែក; Format តែពេលបង្ហាញ។ ការគណនាប្រាក់ត្រូវគិតក្បួន Decimal/Minor Unit របស់ Currency នោះ។

## ៤. Format កាលបរិច្ឆេទ និង Time Zone

```php
<?php
$date = new DateTimeImmutable('2026-11-12 09:30:00', new DateTimeZone('Asia/Phnom_Penh'));

$dateFormatter = new IntlDateFormatter(
    'km-KH',
    IntlDateFormatter::LONG,
    IntlDateFormatter::SHORT,
    'Asia/Phnom_Penh'
);

$result = $dateFormatter->format($date);

if ($result === false) {
    throw new RuntimeException('Unable to format date.');
}

echo $result;
```

រក្សាពេលវេលាផ្ទៃក្នុងតាមក្បួនប្រព័ន្ធដែលបានកំណត់ (ជាទូទៅ UTC សម្រាប់ Timestamp) ហើយបម្លែងទៅ Time Zone របស់ User ពេលបង្ហាញ។ កុំសន្មតថា Time Zone របស់ Server ជា Time Zone របស់អ្នកប្រើ។

កាលបរិច្ឆេទដែលមកពី User ត្រូវ Parse ដោយ Format/Time Zone ដែលច្បាស់ និង Validate Error; កុំ Parse String មិនច្បាស់តាម Default Locale។

## ៥. សារតាមចំនួនដោយ MessageFormatter

ភាសាផ្សេងៗមានក្បួន Singular/Plural ខុសគ្នា។ `MessageFormatter` អាចប្រើ ICU Message Pattern៖

```php
<?php
$pattern = '{count, plural, one{# message} other{# messages}}';
$messageFormatter = new MessageFormatter('en-US', $pattern);
$message = $messageFormatter->format(['count' => 3]);

if ($message === false) {
    throw new RuntimeException('Unable to format message.');
}

echo $message;
```

ក្បួន Plural មិនដូចគ្នាគ្រប់ភាសាទេ។ ប្រើ Translation Message Pattern សម្រាប់ Locale នីមួយៗ ហើយឱ្យ Translator/Native Speaker ពិនិត្យសារនោះ។ កុំភ្ជាប់លេខទៅ String ដោយផ្ទាល់ក្នុងភាសាដែលក្បួនពាក្យប្រែប្រួល។

## ៦. Localization និង Unicode

- រក្សាទុក Text ជា UTF-8; Database/HTTP Response ត្រូវកំណត់ Encoding សមស្រប។
- កុំប្រើ `strtolower()`/`strlen()` សម្រាប់ក្បួន Unicode ទាំងអស់; ត្រូវជ្រើស mbstring/Intl Function តាមការងារ។
- កុំច្រឡំ Normalization ជាមួយ Validation; Unicode អាចមាន Sequence ច្រើនតំណាងអក្សរដូចគ្នា។
- កំណត់ Locale Data និង Date/Number Format នៅ Presentation Layer។
- ប្រើ Translation Catalog/Framework Translator សម្រាប់ App ធំ ជំនួសការបង្កើត If/Else ភាសាច្រើននៅក្នុង Template។

## ៧. កំហុសដែលជួបញឹកញាប់

- ប្រើ Format Number/Date តែមួយសម្រាប់អ្នកប្រើគ្រប់ Locale។
- គណនាប្រាក់ដោយ String ដែលបាន Format ឬ Float ដោយមិនគិត Precision។
- ប្រើ Time Zone Default របស់ Server ដោយមិនកំណត់។
- ជឿ Locale ពី Query String ដោយគ្មាន Allowlist។
- សរសេរ Plural Logic ដោយបន្ថែម “s” ដោយសន្មតថាគ្រប់ភាសាដូចគ្នា។
- គិតថា Intl Format ធានាថាគ្រប់ Client បង្ហាញដូចគ្នាទាំងស្រុង; ICU/Browser/Locale Data អាចខុសគ្នា។

## ៨. លំហាត់

1. បន្ថែម Locale Allowlist សម្រាប់ Khmer និង English។
2. Format ចំនួន 1234567.89 ជា Decimal តាម Locale ទាំងពីរ។
3. Format DateTime មួយក្នុង UTC និង Asia/Phnom_Penh។
4. សរសេរ Message Pattern ដែលមានចំនួន Item ខុសគ្នា។
5. ពន្យល់ពីមូលហេតុដែល Database គួររក្សា Numeric Value មិនមែន String ដែលបាន Format។

## ៩. ចម្លើយណែនាំខ្លី

- ប្រើ Intl `NumberFormatter`, `IntlDateFormatter` និង `MessageFormatter` សម្រាប់ Presentation តាម Locale។
- Validate/Allowlist Locale និងកំណត់ Time Zone ច្បាស់។
- រក្សាតម្លៃលេខ/ប្រាក់ក្នុងទម្រង់ដែលគណនាបានត្រឹមត្រូវ ហើយ Format ពេលបង្ហាញ។
- Plural Rules ខុសតាមភាសា; ប្រើ Translation Pattern មិនមែនបន្ថែមអក្សរដោយសន្មត។
- ពិនិត្យថា PHP Intl Extension បានបើកនៅ Environment ដែលត្រូវដំណើរការ។

## ឯកសារយោង

- [PHP Manual: Internationalization (Intl)](https://www.php.net/manual/en/book.intl.php)
- [PHP Manual: NumberFormatter](https://www.php.net/manual/en/class.numberformatter.php)
- [PHP Manual: IntlDateFormatter](https://www.php.net/manual/en/class.intldateformatter.php)
- [PHP Manual: MessageFormatter](https://www.php.net/manual/en/class.messageformatter.php)
- [មេរៀនទី ១៤៖ PHP Date and Time](lesson-14-php-date-and-time.md)
