# 052 - Text Encoding, Localization, and Internationalization

Text can look the same on screen while being stored as different bytes. It can also need different date, number, sorting, and message rules for different audiences. This chapter explains how PHP works with encodings and introduces the tools for locale-aware output.

## What you will learn

- Keep text in a consistent encoding, commonly UTF-8.
- Validate and convert strings when the source encoding is known.
- Choose byte, character, or grapheme operations for the task.
- Normalize Unicode text when consistent comparison or storage requires it.
- Format numbers and dates for a locale with the **intl** extension.
- Sort text and build translated plural-aware messages.
- Preserve UTF-8 across HTTP, HTML, JSON, and database boundaries.

## Encoding, localization, and internationalization

These terms describe related but different concerns:

- A **character encoding** maps text to bytes. UTF-8 is a common encoding for web content and supports Unicode.
- **Internationalization** (often shortened to **i18n**) prepares an application to support different languages, regions, calendars, number conventions, and text rules.
- **Localization** (often shortened to **l10n**) supplies the translations and regional choices for one audience, such as French for France or English for India.
- A **locale** identifies preferences and rules for formatting or comparing text. It is not a character encoding and does not translate messages by itself.

PHP strings are byte sequences. PHP does not automatically attach an encoding to each string or make every string function Unicode-aware. A Unicode-aware workflow depends on knowing the encoding at each boundary and using functions that understand it.

## Use UTF-8 consistently

A practical application boundary plan is:

1. Save PHP source and text files as UTF-8.
2. Know the encoding of incoming text instead of guessing.
3. Validate or convert data at input boundaries.
4. Keep application text in UTF-8.
5. Configure each output or storage boundary to use the intended encoding.
6. Encode or escape output for its destination context.

For an HTML response, declare UTF-8 in the HTTP response header and document:

~~~php
<?php

header('Content-Type: text/html; charset=UTF-8');
~~~

The response header tells the browser how to decode the bytes. A matching HTML metadata declaration can also describe the document:

~~~html
<meta charset="UTF-8">
~~~

A declaration does not convert incorrectly encoded bytes. The bytes themselves must be valid for the declared encoding.

When rendering untrusted text into HTML, use context-appropriate escaping, for example **htmlspecialchars($text, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8')** for an HTML text or quoted attribute context. Encoding and escaping solve different problems: valid UTF-8 can still contain markup characters that must be escaped.

## Validate and convert known encodings

The **mbstring** extension provides encoding-aware functions. **mb_check_encoding()** checks whether bytes are valid for a specified encoding:

~~~php
<?php

$input = "café";

if (mb_check_encoding($input, 'UTF-8')) {
    echo 'Valid UTF-8';
}
~~~

This validates the byte sequence; it does not discover what encoding a source intended. When a file or upstream system is known to use a legacy encoding, convert from that known source to UTF-8:

~~~php
<?php

$legacyBytes = "caf\xE9"; // The final byte represents é in ISO-8859-1.
$utf8Text = mb_convert_encoding($legacyBytes, 'UTF-8', 'ISO-8859-1');

echo $utf8Text;
~~~

**mb_detect_encoding()** can estimate a likely encoding from candidates, but valid byte sequences may be valid in several encodings. Prefer reliable metadata, a documented file encoding, or a protocol declaration over guessing.

Do not apply conversion blindly. Converting UTF-8 as if it were another encoding can corrupt already valid text. Record the actual source encoding and convert only when necessary.

## Count and manipulate Unicode text

Some PHP string functions operate on bytes:

- **strlen()** returns the number of bytes.
- **substr()** uses byte offsets.
- **strpos()** reports a byte offset.

The **mbstring** functions such as **mb_strlen()**, **mb_substr()**, and **mb_strpos()** work with characters in a specified encoding, commonly UTF-8. The **intl** extension provides grapheme functions such as **grapheme_strlen()** for user-perceived grapheme clusters.

~~~php
<?php

$text = 'café';

echo strlen($text) . "\n";
echo mb_strlen($text, 'UTF-8') . "\n";
~~~

For this UTF-8 text, **strlen()** counts five bytes while **mb_strlen()** counts four encoded characters. A grapheme cluster may contain multiple code points—for example, a base letter with a combining accent—so a grapheme count can differ from a character count.

Choose the unit that matches the requirement:

| Need | Typical tool |
| --- | --- |
| Bytes for a protocol or binary limit | **strlen()** |
| Encoded characters in known UTF-8 text | **mb_strlen()**, **mb_substr()** |
| User-perceived grapheme clusters | **grapheme_strlen()**, **grapheme_substr()** |

The mbstring and intl extensions may need to be enabled in the PHP installation.

## Normalize Unicode when comparisons require it

Some visible characters can be represented by different Unicode code-point sequences. For example, an accented letter may be represented as one precomposed code point or as a base letter followed by a combining mark. These strings can look identical but compare as different byte sequences.

The **Normalizer** class from **intl** can convert text to a chosen Unicode normalization form. NFC is commonly used to compose canonical sequences:

~~~php
<?php

$precomposed = "\u{00E9}";
$decomposed = "e\u{0301}";

var_dump($precomposed === $decomposed);

$normalized = Normalizer::normalize($decomposed, Normalizer::FORM_C);
var_dump($normalized === $precomposed);
~~~

Normalization is a data policy, not a general cleanup step. Apply a consistent form when comparison, indexing, or storage requires it. Do not assume normalization removes accents or makes different words equivalent.

## Use locales for numbers and currencies

Number conventions vary: decimal and grouping separators, digit shapes, and currency placement can differ by locale. Keep numeric data numeric, then format it for the audience at the output boundary.

The **NumberFormatter** class uses the **intl** extension and ICU locale data:

~~~php
<?php

$usFormatter = new NumberFormatter('en_US', NumberFormatter::DECIMAL);
$germanFormatter = new NumberFormatter('de_DE', NumberFormatter::DECIMAL);

echo $usFormatter->format(12345.67) . "\n";
echo $germanFormatter->format(12345.67) . "\n";
~~~

A currency formatter formats a number and currency label; it does not convert between currencies or apply exchange rates:

~~~php
<?php

$formatter = new NumberFormatter('en_US', NumberFormatter::CURRENCY);
echo $formatter->formatCurrency(12345.67, 'USD');
~~~

Do not parse formatted output by replacing commas and periods manually. Use the matching locale-aware parser when localized numeric input must be accepted, then validate the parsed value.

## Format dates for the audience

The core **DateTimeImmutable::format()** method uses explicit format tokens but is not locale translation. For localized month names, weekday names, or ordering, use **IntlDateFormatter** with an explicit locale and time zone.

~~~php
<?php

$date = new DateTimeImmutable(
    '2026-04-10 15:30:00',
    new DateTimeZone('UTC')
);

$formatter = new IntlDateFormatter(
    'fr_FR',
    IntlDateFormatter::LONG,
    IntlDateFormatter::SHORT,
    'Europe/Paris'
);

echo $formatter->format($date);
~~~

The formatter's time zone controls how a date-time is displayed. Supply the intended zone explicitly rather than relying on the server default. Localized wording and punctuation can change with ICU locale data, so avoid tests that depend on an unpinned runtime's exact punctuation unless that output is the product requirement.

## Build messages that can be translated

Do not build user-facing sentences by inserting values into English word order and appending a translated noun. Languages can place values and nouns in different orders and can have different plural categories.

**MessageFormatter** uses ICU message patterns. A plural pattern lets the locale choose the appropriate message form:

~~~php
<?php

$pattern = '{count, plural, one {# file} other {# files}}';
$formatter = new MessageFormatter('en_US', $pattern);

echo $formatter->format(['count' => 1]) . "\n";
echo $formatter->format(['count' => 3]) . "\n";
~~~

Store each locale's translated message pattern in translation resources rather than embedding all languages in application logic. Translators need the complete message and context, not fragments that code assembles.

## Sort text by locale rules

Byte-wise or basic string sorting may not match the expectations of a language. The **Collator** class applies locale-sensitive collation rules:

~~~php
<?php

$names = ['Émile', 'Ana', 'Zoë'];
$collator = new Collator('fr_FR');

$collator->sort($names);
print_r($names);
~~~

The locale affects sorting conventions such as the treatment of accents and letters. Use the locale expected by the audience; one universal alphabetic order does not fit every language.

## Keep JSON and databases encoding-safe

**json_encode()** expects string data to be UTF-8. Use **JSON_THROW_ON_ERROR** when encoding failures should become exceptions:

~~~php
<?php

$data = ['city' => 'Montréal'];
$json = json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);

echo $json;
~~~

If data may contain invalid bytes, first decide whether to reject it or intentionally replace invalid sequences. **JSON_INVALID_UTF8_SUBSTITUTE** can substitute invalid input, but replacement can lose information; it should not hide an unknown source-encoding problem.

For a database, configure the connection and schema to use a compatible Unicode encoding. Keep the connection charset aligned with the application encoding, and let the database driver handle the protocol conversion. Do not manually convert values twice.

## About setlocale()

PHP's **setlocale()** changes C library locale settings for selected categories. It depends on locales installed on the operating system and can affect process-wide behavior. Its results can vary between development and production systems.

For new code, prefer locale-aware **intl** classes with an explicit locale for user-facing number, date, message, and collation formatting. Use **setlocale()** when a specific system-level locale operation requires it, and configure it deliberately.

## Common mistakes

- Treating PHP strings as if they always carry encoding metadata.
- Assuming **strlen()** counts UTF-8 characters.
- Guessing a source encoding and converting valid text blindly.
- Assuming UTF-8 declarations convert malformed or legacy bytes.
- Comparing visually identical Unicode text without considering normalization.
- Formatting numbers by manually changing punctuation.
- Treating currency formatting as exchange-rate conversion.
- Concatenating translated sentence fragments or handling plural forms with a simple singular/plural branch.
- Relying on the server's locale or time zone for user-facing output.
- Assuming one locale's collation order works for every language.

## Try it yourself

1. Compare **strlen()**, **mb_strlen()**, and **grapheme_strlen()** for plain text, an accented character, and a combining sequence.
2. Validate a UTF-8 string with **mb_check_encoding()** and reject malformed bytes.
3. Convert a known ISO-8859-1 byte string to UTF-8 with **mb_convert_encoding()**.
4. Normalize composed and decomposed forms to NFC, then compare the results.
5. Format the same numeric value for **en_US** and **de_DE** with **NumberFormatter**.
6. Format a **DateTimeImmutable** using **IntlDateFormatter** with two locales and explicit time zones.
7. Add a plural pattern to a **MessageFormatter** and test counts of zero, one, and several.
8. Sort accented names with **Collator** for the intended audience locale.
9. Encode UTF-8 JSON with **JSON_THROW_ON_ERROR** and decide how invalid input should be handled.

## Key takeaways

- Agree on an encoding at every boundary; UTF-8 is a practical application-wide choice.
- Validate byte sequences and convert only from a known source encoding.
- Choose byte, character, or grapheme functions based on what is being measured.
- Normalize Unicode when consistent comparison or storage requires a canonical form.
- Use explicit locales with **intl** for localized numbers, dates, messages, and sorting.
- Locale-aware currency formatting does not perform currency conversion.
- JSON strings must be UTF-8, and invalid input needs an intentional error policy.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [051 - Dates, Times, and Time Zones](051-dates-times-and-time-zones.md) | [Open Q&A](../qnas/052-text-encoding-localization-and-internationalization.md) | [Open example](../examples/052-text-encoding-localization-and-internationalization.md) | [053 - Files, Directories, and Filesystem Operations](053-files-directories-and-filesystem-operations.md) |
