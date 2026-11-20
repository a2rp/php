# 052 - Text Encoding, Localization, and Internationalization: Complete Example

This example checks UTF-8 input, compares byte, code-point, and grapheme counts, converts bytes from a known legacy encoding, normalizes Unicode, formats values for explicit locales, creates plural-aware messages, sorts naturally, and encodes JSON safely.

## Goal

Handle user-facing text with its encoding and locale stated explicitly. The script uses **mbstring** for encoding-aware text work and **intl** for Unicode normalization, locale formatting, messages, and collation.

## Requirements

- PHP 8.1 or later.
- The **mbstring** extension.
- The **intl** extension with ICU locale data for **en_US**, **de_DE**, and **fr_FR**.
- Save the file as UTF-8.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

if (!extension_loaded('mbstring') || !extension_loaded('intl')) {
    fwrite(STDERR, "Enable the mbstring and intl extensions.\n");
    exit(1);
}

// These checks make the script's extension requirements explicit.
if (!mb_check_encoding('café', 'UTF-8')) {
    throw new RuntimeException('The source file or input is not valid UTF-8.');
}

$text = 'café';
$decomposed = "e\u{0301}";

echo "UTF-8 and text length:\n";
echo 'Valid UTF-8: ' . (mb_check_encoding($text, 'UTF-8') ? 'yes' : 'no') . "\n";
echo 'Bytes in café: ' . strlen($text) . "\n";
echo 'Code points in café: ' . mb_strlen($text, 'UTF-8') . "\n";
echo 'Code points in decomposed é: ' . mb_strlen($decomposed, 'UTF-8') . "\n";
echo 'Grapheme clusters in decomposed é: ' . grapheme_strlen($decomposed) . "\n";

// This input is deliberately malformed UTF-8: C3 must be followed by a continuation byte.
$malformed = "\xC3\x28";
echo 'Malformed input is valid UTF-8: '
    . (mb_check_encoding($malformed, 'UTF-8') ? 'yes' : 'no')
    . "\n";

// Convert only because the source encoding is known to be ISO-8859-1.
$legacyBytes = "caf\xE9";
$converted = mb_convert_encoding($legacyBytes, 'UTF-8', 'ISO-8859-1');

echo "Known legacy encoding conversion:\n";
echo 'Converted text: ' . $converted . "\n";

// Canonically equivalent text can have different code-point sequences.
// NFC turns the decomposed sequence into its composed form.
$normalized = Normalizer::normalize($decomposed, Normalizer::FORM_C);

if ($normalized === false) {
    throw new RuntimeException('Unicode normalization failed.');
}

$composed = "\u{00E9}";

echo "Unicode normalization:\n";
echo 'NFC forms match: ' . ($normalized === $composed ? 'yes' : 'no') . "\n";

// The same numeric value uses different separators in different locales.
$amount = 12345.67;
$usNumber = new NumberFormatter('en_US', NumberFormatter::DECIMAL);
$germanNumber = new NumberFormatter('de_DE', NumberFormatter::DECIMAL);

echo "Locale-aware number formatting:\n";
echo 'en_US: ' . $usNumber->format($amount) . "\n";
echo 'de_DE: ' . $germanNumber->format($amount) . "\n";

// The pattern fixes the order; the locale supplies the month name.
// An explicit time zone avoids relying on the machine's default time zone.
$date = new DateTimeImmutable('2026-04-10 12:00:00', new DateTimeZone('UTC'));
$frenchDate = new IntlDateFormatter(
    'fr_FR',
    IntlDateFormatter::NONE,
    IntlDateFormatter::NONE,
    'UTC',
    IntlDateFormatter::GREGORIAN,
    'd MMMM y'
);

echo "Locale-aware date formatting:\n";
echo 'fr_FR: ' . $frenchDate->format($date->getTimestamp()) . "\n";

// ICU plural rules select the right message form for each count.
$messagePattern = '{count, plural, one {# file} other {# files}}';
$fileMessage = new MessageFormatter('en_US', $messagePattern);

echo "Plural-aware messages:\n";
echo $fileMessage->format(['count' => 1]) . "\n";
echo $fileMessage->format(['count' => 3]) . "\n";

// Numeric collation sorts the numeric parts by value instead of as plain text.
$items = ['item10', 'item2', 'item1'];
$collator = new Collator('en_US');
$collator->setAttribute(Collator::NUMERIC_COLLATION, Collator::ON);

if (!$collator->sort($items)) {
    throw new RuntimeException('Locale-aware sorting failed.');
}

echo "Locale-aware numeric sorting:\n";
echo implode(', ', $items) . "\n";

// JSON text is UTF-8. Throw on unexpected encoding errors, and opt into
// substitution only for this example's deliberately malformed input.
$validJson = json_encode(
    ['city' => 'Montréal'],
    JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR
);

$substitutedJson = json_encode(
    ['text' => $malformed],
    JSON_UNESCAPED_UNICODE
        | JSON_INVALID_UTF8_SUBSTITUTE
        | JSON_THROW_ON_ERROR
);

echo "UTF-8 JSON:\n";
echo $validJson . "\n";
echo 'Malformed input with substitution: ' . $substitutedJson . "\n";
~~~

## Expected output

The French month name depends on the installed ICU locale data. With standard French data, the output is:

~~~text
UTF-8 and text length:
Valid UTF-8: yes
Bytes in café: 5
Code points in café: 4
Code points in decomposed é: 2
Grapheme clusters in decomposed é: 1
Malformed input is valid UTF-8: no
Known legacy encoding conversion:
Converted text: café
Unicode normalization:
NFC forms match: yes
Locale-aware number formatting:
en_US: 12,345.67
de_DE: 12.345,67
Locale-aware date formatting:
fr_FR: 10 avril 2026
Plural-aware messages:
1 file
3 files
Locale-aware numeric sorting:
item1, item2, item10
UTF-8 JSON:
{"city":"Montréal"}
Malformed input with substitution: {"text":"�("}
~~~

## How it works

- **strlen()** counts bytes. In UTF-8, **café** uses five bytes even though it has four Unicode code points.
- **mb_strlen()** counts code points. A decomposed accented character has two code points.
- **grapheme_strlen()** counts user-perceived text clusters. The decomposed accent is one grapheme cluster.
- **mb_check_encoding()** validates bytes against the encoding you expect. It does not guess an unknown source encoding.
- **mb_convert_encoding()** is safe here because the incoming bytes are known to use ISO-8859-1. A different source encoding needs a different explicit label.
- **Normalizer::normalize()** converts canonically equivalent sequences to NFC before comparison.
- **NumberFormatter** uses the requested locale's number conventions. It formats a number; it does not convert currencies or exchange rates.
- **IntlDateFormatter** uses a fixed pattern and an explicit time zone while localizing the month name.
- **MessageFormatter** applies ICU plural rules instead of joining translated sentence fragments.
- **Collator** with numeric collation sorts the numeric portions of item names by numeric value.
- **json_encode()** receives valid UTF-8 for the normal case. **JSON_THROW_ON_ERROR** makes unexpected encoding failures visible; **JSON_INVALID_UTF8_SUBSTITUTE** is an explicit recovery policy for the malformed demonstration input.

## Try it yourself

1. Replace **café** with an emoji or a family emoji sequence. Compare the byte, code-point, and grapheme counts.
2. Change the legacy input to known Windows-1252 bytes and pass **Windows-1252** as the source encoding.
3. Normalize both composed and decomposed versions of a word, then compare their normalized values.
4. Format the amount using **NumberFormatter::CURRENCY** for two locales. Remember that formatted currency symbols do not perform exchange-rate conversion.
5. Change **fr_FR** to another locale and observe its month name.
6. Test message counts of zero, one, and several with a locale-appropriate plural pattern.
7. Turn off numeric collation and compare how the item names sort.
8. Remove **JSON_INVALID_UTF8_SUBSTITUTE** and observe the **JsonException** raised for malformed input.

## Common mistakes

- Treating **strlen()** as a count of visible characters.
- Assuming a declaration that text is UTF-8 converts bytes that use a different encoding.
- Guessing an encoding instead of recording it at the input boundary.
- Comparing composed and decomposed Unicode strings without normalizing when canonical equivalence matters.
- Manually inserting locale-specific commas, periods, or spaces.
- Assuming a formatted currency amount has been converted to another currency.
- Relying on the server's default locale or time zone.
- Silently discarding invalid UTF-8 without choosing a documented policy.

## Key takeaways

- Keep text in a known encoding, commonly UTF-8, and validate it at boundaries.
- Choose byte, code-point, or grapheme operations according to the question being answered.
- Convert only when the source encoding is known.
- Normalize Unicode when canonical comparison or storage consistency requires it.
- Use explicit locales and **intl** for user-facing numbers, dates, messages, and sorting.
- Handle invalid UTF-8 deliberately when producing JSON.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [051 - Dates, Times, and Time Zones](../lessons/051-dates-times-and-time-zones.md) | [Open Q&A](../qnas/052-text-encoding-localization-and-internationalization.md) | [Open lesson](../lessons/052-text-encoding-localization-and-internationalization.md) | [053 - Files, Directories, and Filesystem Operations](../lessons/053-files-directories-and-filesystem-operations.md) |
