# 052 - Text Encoding, Localization, and Internationalization: Q&A

Review UTF-8, encoding validation and conversion, Unicode character units, localization, ICU formatting, normalization, collation, and JSON. Try each question before reading its answer. See the [chapter lesson](../lessons/052-text-encoding-localization-and-internationalization.md).

## Multiple-choice questions

### 1. What does a character encoding define?

A. How text is mapped to bytes
B. Which words are translated
C. How arrays are sorted
D. Which PHP classes are loaded

**Answer:** A. How text is mapped to bytes.

**Explanation:** A string's bytes must be interpreted using the correct encoding to recover text.

### 2. What is a practical encoding choice for web text?

A. UTF-8
B. A different encoding for every PHP string
C. Guessing an encoding at runtime
D. ASCII only

**Answer:** A. UTF-8.

**Explanation:** UTF-8 supports Unicode and is widely used for web content. A consistent choice across input, storage, and output avoids conversion errors.

### 3. What does **strlen()** count?

A. Bytes
B. Grapheme clusters
C. Words
D. Unicode normalization forms

**Answer:** A. Bytes.

**Explanation:** A multibyte UTF-8 character can occupy more than one byte.

### 4. What does **mb_strlen($text, 'UTF-8')** count?

A. Bytes only
B. Encoded characters in the specified encoding
C. Words separated by spaces
D. Filesystem characters

**Answer:** B. Encoded characters in the specified encoding.

**Explanation:** It counts multibyte encoded characters as characters, not as individual bytes.

### 5. Which function counts user-perceived grapheme clusters?

A. **strlen()**
B. **mb_detect_encoding()**
C. **grapheme_strlen()**
D. **count()**

**Answer:** C. **grapheme_strlen()**

**Explanation:** It counts grapheme units, which can group multiple code points into one user-perceived character.

### 6. What does **mb_check_encoding($text, 'UTF-8')** do?

A. Checks whether the bytes are valid UTF-8
B. Converts the text to UTF-8
C. Detects the user's language
D. Normalizes the text to NFC

**Answer:** A. Checks whether the bytes are valid UTF-8.

**Explanation:** It validates the byte sequence; it does not convert it or infer what encoding the source intended.

### 7. What should be known before calling **mb_convert_encoding()** on input?

A. The actual source encoding
B. The user's currency
C. The output HTML element
D. The array's sort order

**Answer:** A. The actual source encoding.

**Explanation:** Conversion needs the correct source and target encodings. Guessing the source can corrupt already valid text.

### 8. Why can automatic encoding detection be unreliable?

A. A byte sequence can be valid under more than one encoding.
B. UTF-8 contains no bytes.
C. PHP strings have no length.
D. It always converts everything to ASCII.

**Answer:** A. A byte sequence can be valid under more than one encoding.

**Explanation:** Use reliable metadata or a documented source encoding rather than guessing whenever possible.

### 9. What is a locale?

A. A set of language and regional conventions used for operations such as formatting
B. A character encoding
C. A PHP variable type
D. A translated text file only

**Answer:** A. A set of language and regional conventions used for operations such as formatting.

**Explanation:** A locale does not itself convert bytes or translate every message.

### 10. Which PHP extension provides ICU-based localized formatting and collation classes?

A. **intl**
B. **mbstring** only
C. **json**
D. **pcre**

**Answer:** A. **intl**

**Explanation:** The **intl** extension exposes ICU features such as **NumberFormatter**, **IntlDateFormatter**, **MessageFormatter**, **Collator**, and **Normalizer**.

### 11. Does **NumberFormatter::formatCurrency()** convert a value from one currency into another?

A. Yes, it reads live exchange rates.
B. No, it formats the number and currency for a locale.
C. It only changes the decimal separator.
D. It returns a numeric exchange-rate value.

**Answer:** B. No, it formats the number and currency for a locale.

**Explanation:** Formatting a number with a currency code does not perform exchange-rate conversion.

### 12. What does **IntlDateFormatter** provide?

A. Locale-aware date and time formatting
B. UTF-8 validation
C. Database date storage
D. Currency exchange

**Answer:** A. Locale-aware date and time formatting.

**Explanation:** Use it when month names, ordering, or other date conventions should follow a locale.

### 13. What does **MessageFormatter** help with?

A. Building locale-aware messages with arguments and plural rules
B. Converting all PHP files to UTF-8
C. Sorting filesystem paths
D. Hashing passwords

**Answer:** A. Building locale-aware messages with arguments and plural rules.

**Explanation:** Message patterns let locale-specific resources control word order and plural forms.

### 14. What does **Normalizer::normalize()** do?

A. Produces text in a selected Unicode normalization form
B. Translates the string into a locale
C. Converts a currency amount
D. Removes all accents

**Answer:** A. Produces text in a selected Unicode normalization form.

**Explanation:** Normalization can make canonically equivalent text use a consistent representation. It does not translate text or generally remove accents.

### 15. Which function sorts strings according to a locale's collation rules?

A. **Collator::sort()**
B. **sort()** with no locale context
C. **array_values()**
D. **mb_convert_encoding()**

**Answer:** A. **Collator::sort()**

**Explanation:** A **Collator** uses locale-specific comparison rules that can account for language conventions.

## Short-answer questions

### 16. What is the difference between an encoding and a locale?

**Answer:** An encoding maps bytes to text. A locale describes language and regional conventions such as formatting and collation.

**Explanation:** A locale does not convert an incorrectly encoded string.

### 17. Why use UTF-8 consistently across application boundaries?

**Answer:** It prevents different components from interpreting the same bytes under conflicting encodings.

**Explanation:** Source files, HTTP responses, database connections, JSON, and application strings should agree on the encoding or explicitly convert at a boundary.

### 18. What does **mb_check_encoding()** validate?

**Answer:** Whether a byte sequence is valid for a specified encoding.

**Explanation:** It does not determine the intended encoding of unknown bytes and does not convert them.

### 19. Why is **mb_detect_encoding()** not a substitute for encoding metadata?

**Answer:** Multiple encodings can accept the same byte sequence, so a detected result is only a guess.

**Explanation:** Prefer a declared content encoding or documented source format.

### 20. What arguments should a conversion routine know?

**Answer:** The source encoding and the target encoding.

**Explanation:** Pass both explicitly when possible, for example converting known ISO-8859-1 input to UTF-8.

### 21. Why can two visually identical strings compare as different?

**Answer:** Unicode can represent a visible character with different code-point sequences, such as a precomposed character or a base character plus a combining mark.

**Explanation:** Their bytes can differ. If consistent comparison is required, normalize both to the same Unicode form.

### 22. What does NFC normalization do in broad terms?

**Answer:** It transforms canonically equivalent sequences into a composed normalization form where applicable.

**Explanation:** NFC can provide a consistent representation; it does not translate words or remove all diacritics.

### 23. When should **mb_strlen()** be used instead of **strlen()**?

**Answer:** Use **mb_strlen()** when the requirement is to count encoded characters in a known multibyte encoding such as UTF-8.

**Explanation:** **strlen()** counts bytes. For user-perceived grapheme clusters, use a grapheme-aware function instead.

### 24. When is **grapheme_strlen()** preferable to **mb_strlen()**?

**Answer:** When the product requirement is to count user-perceived grapheme units, such as a base letter with a combining mark as one displayed character.

**Explanation:** **mb_strlen()** counts encoded characters, which do not always align with grapheme clusters.

### 25. Why might **htmlspecialchars()** still be needed for valid UTF-8 text?

**Answer:** Encoding correctness does not escape markup characters for an HTML output context.

**Explanation:** UTF-8 handling and output-context escaping solve different problems.

### 26. How should a number be prepared for a localized display?

**Answer:** Keep it numeric internally and format it with **NumberFormatter** for the target locale at the output boundary.

**Explanation:** Avoid changing commas and periods manually, since conventions vary by locale.

### 27. Does formatting with a currency code apply exchange rates?

**Answer:** No.

**Explanation:** A currency formatter selects symbols and formatting conventions. Currency conversion requires separate exchange-rate data and business rules.

### 28. Why can a translated sentence be difficult to build by concatenating fragments?

**Answer:** Languages can vary in word order, grammar, and plural categories.

**Explanation:** Store complete locale-specific messages and pass values to a formatter such as **MessageFormatter**.

### 29. What encoding does **json_encode()** require for strings?

**Answer:** UTF-8.

**Explanation:** Invalid UTF-8 causes an encoding error unless an explicit handling flag such as substitution is chosen. Decide whether to reject or replace the data.

## True or false

### 30. PHP automatically stores an encoding label with every string.

**Answer:** False.

**Explanation:** PHP strings are byte sequences; the program must know how to interpret their bytes.

### 31. **mb_check_encoding()** converts invalid bytes to valid UTF-8.

**Answer:** False.

**Explanation:** It checks validity. Conversion and invalid-input policy are separate decisions.

### 32. **strlen()** counts the number of UTF-8 bytes.

**Answer:** True.

**Explanation:** It reports bytes, not encoded characters or grapheme clusters.

### 33. A locale and a character encoding are the same thing.

**Answer:** False.

**Explanation:** A locale describes regional conventions; an encoding maps text to bytes.

### 34. **NumberFormatter** performs currency exchange when a currency code is supplied.

**Answer:** False.

**Explanation:** It formats an amount and code; it does not know exchange rates.

### 35. Unicode normalization always removes accents.

**Answer:** False.

**Explanation:** Canonical normalization changes representation form; it does not generally remove accents.

### 36. A valid UTF-8 declaration in an HTTP header converts malformed response bytes automatically.

**Answer:** False.

**Explanation:** The header tells the receiver how to interpret bytes; it does not rewrite them.

### 37. **JSON_THROW_ON_ERROR** can make JSON encoding failures explicit through an exception.

**Answer:** True.

**Explanation:** It avoids silently relying on the global JSON error state.

## Predict the output

### 38. What does this print when the source is UTF-8 and mbstring is enabled?

~~~php
<?php

$text = 'café';

echo strlen($text) . ' / ';
echo mb_strlen($text, 'UTF-8');
~~~

**Answer:**

~~~text
5 / 4
~~~

**Explanation:** The UTF-8 **é** uses two bytes, so the string has five bytes but four encoded characters.

### 39. What does this print when intl is enabled?

~~~php
<?php

$composed = "\u{00E9}";
$decomposed = "e\u{0301}";

var_dump($composed === $decomposed);

$normalized = Normalizer::normalize($decomposed, Normalizer::FORM_C);
var_dump($normalized === $composed);
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** The original sequences differ. NFC normalization makes the decomposed string use the composed representation.

### 40. What does this print when intl is enabled with current German decimal conventions?

~~~php
<?php

$formatter = new NumberFormatter('de_DE', NumberFormatter::DECIMAL);
echo $formatter->format(12345.67);
~~~

**Answer:**

~~~text
12.345,67
~~~

**Explanation:** The locale uses a dot for grouping and a comma for the decimal separator. Locale data can affect presentation details.

### 41. What does this print when intl is enabled?

~~~php
<?php

$formatter = new MessageFormatter(
    'en_US',
    '{count, plural, one {# file} other {# files}}'
);

echo $formatter->format(['count' => 2]);
~~~

**Answer:**

~~~text
2 files
~~~

**Explanation:** The plural rule selects the **other** form for a count of two.

### 42. What does this print?

~~~php
<?php

$data = ['city' => 'Montréal'];

echo json_encode($data, JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE);
~~~

**Answer:**

~~~text
{"city":"Montréal"}
~~~

**Explanation:** The input is UTF-8, and **JSON_UNESCAPED_UNICODE** keeps Unicode characters literal in the JSON string instead of escaping them.

## Coding questions

### 43. Validate UTF-8 input and reject it when the bytes are malformed.

**Answer:**

~~~php
<?php

function requireUtf8(string $input): string
{
    if (!mb_check_encoding($input, 'UTF-8')) {
        throw new InvalidArgumentException('Input must be valid UTF-8.');
    }

    return $input;
}

echo requireUtf8('café');
~~~

**Expected output:**

~~~text
café
~~~

**Explanation:** The function checks validity and rejects invalid bytes. It does not guess or convert an unknown source encoding.

### 44. Format a number for US and German readers while keeping the input numeric.

**Answer:**

~~~php
<?php

$amount = 12345.67;

$us = new NumberFormatter('en_US', NumberFormatter::DECIMAL);
$german = new NumberFormatter('de_DE', NumberFormatter::DECIMAL);

echo $us->format($amount) . "\n";
echo $german->format($amount) . "\n";
~~~

**Expected output:**

~~~text
12,345.67
12.345,67
~~~

**Explanation:** **NumberFormatter** applies locale-specific separators. The underlying value remains a number.

### 45. Format a message with a locale-aware plural pattern.

**Answer:**

~~~php
<?php

$pattern = '{count, plural, one {# message} other {# messages}}';
$formatter = new MessageFormatter('en_US', $pattern);

foreach ([1, 4] as $count) {
    echo $formatter->format(['count' => $count]) . "\n";
}
~~~

**Expected output:**

~~~text
1 message
4 messages
~~~

**Explanation:** The formatter chooses the message form from the count and locale rules. Real applications should keep translated patterns in locale resources so translators can adjust word order and plural categories.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [051 - Dates, Times, and Time Zones](../lessons/051-dates-times-and-time-zones.md) | [Read lesson](../lessons/052-text-encoding-localization-and-internationalization.md) | [Open example](../examples/052-text-encoding-localization-and-internationalization.md) | [053 - Files, Directories, and Filesystem Operations](../lessons/053-files-directories-and-filesystem-operations.md) |
