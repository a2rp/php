# 051 - Dates, Times, and Time Zones: Q&A

Review immutable date-time objects, time zones, format tokens, strict parsing, intervals, schedules, and daylight-saving behavior. Try each question before reading its answer. See the [chapter lesson](../lessons/051-dates-times-and-time-zones.md).

## Multiple-choice questions

### 1. What happens when **modify()** is called on a **DateTimeImmutable** object?

A. It changes the original object.
B. It returns a new object with the modification.
C. It converts the object to a timestamp.
D. It deletes the time zone.

**Answer:** B. It returns a new object with the modification.

**Explanation:** The original remains unchanged. Keep the returned object if you need the modified value.

### 2. Which value is a regional IANA time-zone identifier?

A. **+05:30**
B. **EST**
C. **Asia/Kolkata**
D. **GMT+1**

**Answer:** C. **Asia/Kolkata**

**Explanation:** IANA identifiers represent regions with time-zone rules. A fixed offset does not describe regional transitions, and abbreviations can be ambiguous.

### 3. What does **setTimezone()** do to a **DateTimeImmutable**?

A. Returns a new object displaying the same instant in another zone
B. Changes the instant to a different moment
C. Changes the object's calendar date only
D. Converts the object into a string

**Answer:** A. Returns a new object displaying the same instant in another zone.

**Explanation:** The clock fields and offset may differ, but the point in time stays the same.

### 4. In a PHP date format string, which token represents minutes?

A. **m**
B. **M**
C. **i**
D. **H**

**Answer:** C. **i**

**Explanation:** **m** is the month. Format tokens are case-sensitive; **H** is a 24-hour hour.

### 5. What can **DateTimeImmutable::createFromFormat()** return if parsing fails?

A. Always an empty string
B. **false**
C. Always a DateTime object
D. A Unix timestamp

**Answer:** B. **false**

**Explanation:** It can also return an object for fields that were normalized from out-of-range values, so inspect parse warnings and validate the result.

### 6. What is the purpose of **!** at the beginning of a create-from-format pattern?

A. It makes the format case-insensitive.
B. It resets fields that were not supplied instead of inheriting the current date and time.
C. It forces the date into UTC.
D. It rejects leap years.

**Answer:** B. It resets fields that were not supplied instead of inheriting the current date and time.

**Explanation:** For **!Y-m-d**, omitted time fields become zero-like values, so the parsed date is at midnight.

### 7. What does **DateTimeImmutable::getLastErrors()** return on PHP 8.2 or later when there are no warnings or errors?

A. **false**
B. Always **null**
C. Always an empty string
D. A new DateTime object

**Answer:** A. **false**

**Explanation:** Older PHP versions return an array with zero error and warning counts instead.

### 8. What does **new DateInterval('P1W')** represent?

A. One calendar week
B. One month
C. One Unix timestamp
D. One time zone

**Answer:** A. One calendar week.

**Explanation:** In the ISO 8601 duration syntax, **P** begins the period and **W** denotes weeks.

### 9. By default, what are **DatePeriod**'s start and end rules?

A. Start excluded, end included
B. Start included, end excluded
C. Both excluded
D. Both included

**Answer:** B. Start included, end excluded.

**Explanation:** Check these boundaries when generating a schedule; options can change them.

### 10. What is a Unix timestamp commonly used to measure?

A. A named time zone
B. Seconds from the Unix epoch
C. The month name in a locale
D. A calendar interval in months

**Answer:** B. Seconds from the Unix epoch.

**Explanation:** Timestamps are useful for elapsed-time arithmetic. They do not themselves retain a regional time-zone identifier.

### 11. How long is every local calendar day?

A. Always exactly 24 elapsed hours
B. It can differ around a daylight-saving transition
C. Always exactly 23 hours
D. Always 24 hours plus the local offset

**Answer:** B. It can differ around a daylight-saving transition.

**Explanation:** Local clocks may skip or repeat an hour. A calendar day and a fixed elapsed duration are different concepts.

### 12. What does **diff()** return for two date-time objects?

A. A **DateInterval** describing the calendar difference
B. A formatted string only
C. A **DateTimeZone** object
D. A count of months only

**Answer:** A. A **DateInterval** describing the calendar difference.

**Explanation:** For exact elapsed seconds, compare timestamps or compare values in UTC, especially across daylight-saving changes.

### 13. Which format token outputs a time-zone identifier such as **Asia/Kolkata**?

A. **e**
B. **i**
C. **m**
D. **s**

**Answer:** A. **e**

**Explanation:** **e** outputs the time-zone identifier. **P** outputs a numeric offset with a colon.

### 14. Why prefer an IANA time-zone identifier over an abbreviation such as **EST**?

A. An identifier can represent regional time-zone rules; an abbreviation can be ambiguous.
B. Identifiers always mean UTC.
C. Abbreviations cannot be printed.
D. Identifiers remove the need for date-time objects.

**Answer:** A. An identifier can represent regional time-zone rules; an abbreviation can be ambiguous.

**Explanation:** Regional zones include their time-zone database rules. A label or fixed offset alone does not capture all regional history or daylight-saving transitions.

### 15. How many fixed days does **P1M** always represent?

A. Exactly 28 days
B. Exactly 30 days
C. Exactly 31 days
D. It is a calendar month, not a fixed number of days

**Answer:** D. It is a calendar month, not a fixed number of days.

**Explanation:** Months have different lengths and calendar arithmetic can overflow near month-end.

## Short-answer questions

### 16. What is the main difference between **DateTimeImmutable** and **DateTime**?

**Answer:** **DateTimeImmutable** modification methods return a new object; **DateTime** methods modify the mutable object.

**Explanation:** Immutable values make it explicit when an updated value is created.

### 17. Why should code specify a time zone when interpreting a local date-time string?

**Answer:** The same wall-clock text can represent different instants in different zones.

**Explanation:** Passing a **DateTimeZone** avoids silently depending on the server's default time zone.

### 18. Does **setTimezone()** change the instant or just its display zone?

**Answer:** It changes the display zone and returns a new object representing the same instant.

**Explanation:** The local date, clock time, and UTC offset can change when the same instant is displayed elsewhere.

### 19. What does **date_default_timezone_set()** affect?

**Answer:** It sets the default zone for date-time operations that otherwise rely on PHP's default.

**Explanation:** Explicit zones on objects remain explicit. Application configuration should set a known default if code depends on one.

### 20. Why use **!Y-m-d** to parse a date-only value?

**Answer:** The **!** resets date-time fields not supplied by the input, so missing time fields become midnight rather than the current clock time.

**Explanation:** Supply a **DateTimeZone** too when the date should be interpreted in a particular zone.

### 21. Why check both **getLastErrors()** and the formatted output after parsing?

**Answer:** The parser may normalize out-of-range values and still return a date object.

**Explanation:** Warnings reveal overflow. A round-trip comparison ensures a fixed-format input returns exactly the original date text.

### 22. What changed about **getLastErrors()** in PHP 8.2?

**Answer:** It returns **false** when there are no warnings or errors. Before PHP 8.2, it returned an array with zero counts.

**Explanation:** Code that supports both versions should handle either **false** or an error-information array.

### 23. What do **P** and **T** mean in a **DateInterval** string?

**Answer:** **P** begins the period specification; **T** separates date units from time units.

**Explanation:** **P1DT2H** means one day and two hours; **PT2H** means two hours.

### 24. How can PHP generate a sequence of recurring dates?

**Answer:** Create a **DatePeriod** from a start date, a **DateInterval**, and an end date or recurrence count, then iterate over it.

**Explanation:** Remember that the start is included and the end is excluded by default.

### 25. When is a timestamp useful?

**Answer:** When calculating a fixed elapsed duration, such as the number of seconds between two instants.

**Explanation:** Subtracting timestamps avoids confusing calendar units with elapsed time. Store other information, such as a display zone, separately when needed.

### 26. How does **diff()** differ from subtracting timestamps?

**Answer:** **diff()** describes a calendar interval between date-time values. Timestamp subtraction measures elapsed seconds.

**Explanation:** Across a time-zone transition, the calendar clock difference may not equal a simple number of 24-hour days.

### 27. What is daylight saving time's effect on local times?

**Answer:** A local clock can skip forward or repeat an hour when its region changes offset.

**Explanation:** Some local wall-clock times can be ambiguous or nonexistent. Use a region's time-zone rules and define application behavior for those cases.

### 28. Why is a fixed offset not always enough for a regional schedule?

**Answer:** A fixed offset does not contain a region's historical or daylight-saving rules.

**Explanation:** Use a regional IANA identifier when the schedule follows a place such as a city or country.

### 29. How can a program reject an invalid date such as February 31?

**Answer:** Parse the expected format, inspect parse warnings and errors, and compare the formatted result with the original input.

**Explanation:** Date-time parsing can normalize an overflow into a later date. A successful object creation alone is not strict validation.

## True or false

### 30. **DateTimeImmutable::add()** changes the original object.

**Answer:** False.

**Explanation:** It returns a new immutable date-time object. Save the returned value to use the updated date.

### 31. If no time zone is included in the input and none is passed, PHP may use its configured default time zone.

**Answer:** True.

**Explanation:** Explicit zones make interpretation predictable and portable.

### 32. Calling **setTimezone()** changes the underlying instant.

**Answer:** False.

**Explanation:** It changes how the same instant is represented locally.

### 33. In PHP date formatting, **m** is the month and **i** is the minute.

**Answer:** True.

**Explanation:** Format characters are case-sensitive and easy to confuse.

### 34. A local calendar day always equals exactly 86,400 elapsed seconds.

**Answer:** False.

**Explanation:** Daylight-saving transitions can make a local day shorter or longer in elapsed seconds.

### 35. Since PHP 8.2, **getLastErrors()** returns false when there are no parse warnings or errors.

**Answer:** True.

**Explanation:** Earlier versions returned an array with zero counts.

### 36. By default, a **DatePeriod** includes its end date.

**Answer:** False.

**Explanation:** Its start is included and its end is excluded by default.

### 37. **P1M** always means exactly thirty days.

**Answer:** False.

**Explanation:** It means one calendar month, whose length varies.

## Predict the output

### 38. What does this print?

~~~php
<?php

$date = new DateTimeImmutable(
    '2026-04-10 09:05:00',
    new DateTimeZone('Asia/Kolkata')
);

echo $date->format('Y-m-d H:i P');
~~~

**Answer:**

~~~text
2026-04-10 09:05 +05:30
~~~

**Explanation:** The **P** token formats the offset with a colon.

### 39. What does this print?

~~~php
<?php

$start = new DateTimeImmutable(
    '2026-04-10 09:00:00',
    new DateTimeZone('UTC')
);

$nextDay = $start->add(new DateInterval('P1D'));

echo $start->format('Y-m-d H:i') . ' / ';
echo $nextDay->format('Y-m-d H:i');
~~~

**Answer:**

~~~text
2026-04-10 09:00 / 2026-04-11 09:00
~~~

**Explanation:** **add()** returns a new value one calendar day later. The original remains unchanged.

### 40. What does this print?

~~~php
<?php

$utc = new DateTimeImmutable(
    '2026-04-10 03:30:00',
    new DateTimeZone('UTC')
);

$india = $utc->setTimezone(new DateTimeZone('Asia/Kolkata'));

echo $utc->format('H:i P') . ' / ';
echo $india->format('H:i P');
~~~

**Answer:**

~~~text
03:30 +00:00 / 09:00 +05:30
~~~

**Explanation:** Both values represent the same instant; the local clock and offset differ.

### 41. What does this print?

~~~php
<?php

$date = new DateTimeImmutable(
    '2024-02-29',
    new DateTimeZone('UTC')
);

echo $date->format('Y-m-d');
~~~

**Answer:**

~~~text
2024-02-29
~~~

**Explanation:** 2024 is a leap year, so February 29 is a valid date.

### 42. What does this print?

~~~php
<?php

$first = new DateTimeImmutable('2026-04-10', new DateTimeZone('UTC'));
$second = new DateTimeImmutable('2026-04-13', new DateTimeZone('UTC'));

echo $first->diff($second)->format('%R%a days');
~~~

**Answer:**

~~~text
+3 days
~~~

**Explanation:** The second date is three days after the first, so **%R** shows a positive sign and **%a** the total full days.

## Coding questions

### 43. Create one instant in UTC, display it in India time, and print both time zones.

**Answer:**

~~~php
<?php

$instant = new DateTimeImmutable(
    '2026-04-10 03:30:00',
    new DateTimeZone('UTC')
);

$indiaTime = $instant->setTimezone(
    new DateTimeZone('Asia/Kolkata')
);

echo $instant->format('Y-m-d H:i P e') . "\n";
echo $indiaTime->format('Y-m-d H:i P e') . "\n";
~~~

**Expected output:**

~~~text
2026-04-10 03:30 +00:00 UTC
2026-04-10 09:00 +05:30 Asia/Kolkata
~~~

**Explanation:** The time-zone conversion keeps the instant and changes its local representation.

### 44. Parse a date-only string strictly and reject overflow values.

**Answer:**

~~~php
<?php

function isValidDate(string $input): bool
{
    $timezone = new DateTimeZone('UTC');
    $date = DateTimeImmutable::createFromFormat('!Y-m-d', $input, $timezone);
    $errors = DateTimeImmutable::getLastErrors();

    $hasProblems = $errors !== false
        && ($errors['warning_count'] > 0 || $errors['error_count'] > 0);

    return $date !== false
        && !$hasProblems
        && $date->format('Y-m-d') === $input;
}

var_dump(isValidDate('2024-02-29'));
var_dump(isValidDate('2023-02-29'));
~~~

**Expected output:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Leap day is valid in 2024. The parser may normalize the 2023 overflow; warnings and the round-trip check reject it.

### 45. Print weekly dates from a start date up to, but not including, an end date.

**Answer:**

~~~php
<?php

$timezone = new DateTimeZone('UTC');
$start = new DateTimeImmutable('2026-04-01', $timezone);
$end = new DateTimeImmutable('2026-04-23', $timezone);
$week = new DateInterval('P7D');

$period = new DatePeriod($start, $week, $end);

foreach ($period as $date) {
    echo $date->format('Y-m-d') . "\n";
}
~~~

**Expected output:**

~~~text
2026-04-01
2026-04-08
2026-04-15
2026-04-22
~~~

**Explanation:** The period includes its start date and yields weekly intervals until the end boundary, which is excluded by default.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [050 - Regular Expressions with PCRE](../lessons/050-regular-expressions-with-pcre.md) | [Read lesson](../lessons/051-dates-times-and-time-zones.md) | [Open example](../examples/051-dates-times-and-time-zones.md) | [052 - Text Encoding, Localization, and Internationalization](../lessons/052-text-encoding-localization-and-internationalization.md) |
