# 051 - Dates, Times, and Time Zones

Date and time values look simple until time zones, daylight-saving changes, parsing, and elapsed-time calculations are involved. PHP's **DateTimeImmutable**, **DateTimeZone**, and **DateInterval** classes provide tools for handling those cases explicitly.

## What you will learn

- Create and format date-time values with **DateTimeImmutable**.
- Choose and convert IANA time zones.
- Parse a known input format and detect invalid or overflowing dates.
- Add calendar intervals and compare date-time objects.
- Distinguish calendar differences from elapsed seconds.
- Understand why daylight-saving transitions affect local clock arithmetic.

## Use date-time objects

Prefer **DateTimeImmutable** for application code that passes dates between functions. It represents a date and time, and methods such as **modify()**, **add()**, and **setTimezone()** return a new object instead of changing the existing one.

~~~php
<?php

$start = new DateTimeImmutable('2026-04-10 09:00:00');
$nextWeek = $start->modify('+1 week');

echo $start->format('Y-m-d H:i:s') . "\n";
echo $nextWeek->format('Y-m-d H:i:s') . "\n";
~~~

The original **$start** remains April 10. The returned **$nextWeek** represents April 17.

The older mutable **DateTime** class changes the same object when modification methods are called. If using it, be mindful when a reference to that object is shared. The immutable form helps make such changes explicit.

## Choose a time zone

A date-time has a time zone or offset that determines how its wall-clock value maps to an instant. Use a **DateTimeZone** object to specify one:

~~~php
<?php

$utc = new DateTimeZone('UTC');
$india = new DateTimeZone('Asia/Kolkata');
$newYork = new DateTimeZone('America/New_York');

$meeting = new DateTimeImmutable('2026-04-10 09:00:00', $india);

echo $meeting->format('Y-m-d H:i:s P e');
~~~

Prefer IANA time-zone identifiers such as **Asia/Kolkata** and **America/New_York** when you need regional rules. A fixed offset such as **+05:30** identifies an offset but does not describe a region's historical or daylight-saving rules. Abbreviations can be ambiguous.

You can set a process-wide default with **date_default_timezone_set()**, but explicit time zones on date-time values make code easier to reason about. A default is still useful for application configuration; do not let an unknown machine default decide how user input is interpreted.

### Convert a date-time for display

**setTimezone()** returns a new **DateTimeImmutable** with the same instant shown in another time zone. The clock fields can change; the point in time does not.

~~~php
<?php

$utc = new DateTimeZone('UTC');
$india = new DateTimeZone('Asia/Kolkata');

$instant = new DateTimeImmutable('2026-04-10 03:30:00', $utc);
$indiaDisplay = $instant->setTimezone($india);

echo $instant->format('Y-m-d H:i:s P e') . "\n";
echo $indiaDisplay->format('Y-m-d H:i:s P e') . "\n";
~~~

This is useful when storing or exchanging a known instant in UTC and displaying it in a viewer's time zone.

## Format date-time values

Call **format()** with case-sensitive format characters:

| Character | Meaning | Example |
| --- | --- | --- |
| **Y** | Four-or-more-digit year | **2026** |
| **m** | Month with leading zero | **04** |
| **d** | Day with leading zero | **10** |
| **H** | 24-hour hour | **09** |
| **i** | Minute | **05** |
| **s** | Second | **00** |
| **P** | UTC offset with colon | **+05:30** |
| **e** | Time-zone identifier | **Asia/Kolkata** |
| **c** | ISO 8601 date and time | **2026-04-10T09:05:00+05:30** |
| **U** | Unix timestamp in seconds | seconds since the Unix epoch |

The letters are case-sensitive: **m** is the month, while **i** is minutes; **H** is a 24-hour clock, while **h** is a 12-hour clock.

~~~php
<?php

$local = new DateTimeImmutable(
    '2026-04-10 09:05:00',
    new DateTimeZone('Asia/Kolkata')
);

echo $local->format('Y-m-d H:i:s P') . "\n";
echo $local->format(DateTimeInterface::ATOM) . "\n";
~~~

Formatting is not locale translation. Names from tokens such as **F** and **l** are in English. Use an internationalization formatter when localized month or weekday names are required.

## Parse a known format

For a known input format, use **DateTimeImmutable::createFromFormat()** rather than relying on a permissive free-form parse:

~~~php
<?php

$utc = new DateTimeZone('UTC');
$input = '2026-04-10';

$date = DateTimeImmutable::createFromFormat('!Y-m-d', $input, $utc);

if ($date === false) {
    echo 'Could not parse the date';
} else {
    echo $date->format('Y-m-d H:i:s P');
}
~~~

The format characters describe the input: **Y** is a year, **m** a month, and **d** a day. The leading **!** resets fields that were not provided instead of inheriting the current date and time. Because a date-only input omits a time, it becomes midnight in the supplied time zone.

A parse can normalize out-of-range fields. For example, a day greater than the number of days in its month may overflow into the next month. Check **getLastErrors()** and, for strict fixed-format input, compare the formatted result with the original input:

~~~php
<?php

$input = '2026-02-31';
$utc = new DateTimeZone('UTC');

$date = DateTimeImmutable::createFromFormat('!Y-m-d', $input, $utc);
$errors = DateTimeImmutable::getLastErrors();

$hasParseProblems = $errors !== false
    && ($errors['warning_count'] > 0 || $errors['error_count'] > 0);

$isValidDate = $date !== false
    && !$hasParseProblems
    && $date->format('Y-m-d') === $input;

var_dump($isValidDate);
~~~

This rejects both reported overflows and values that do not round-trip exactly. Since PHP 8.2, **getLastErrors()** returns **false** when there are no warnings or errors; older versions return an array with zero counts in that case.

For user input, validate both the expected format and the calendar value. A returned date object alone does not prove the original date was valid.

## Intervals and calendar arithmetic

A **DateInterval** represents an instruction such as “one week” or “two hours.” Its ISO 8601 duration format uses **P** for the period, date units before **T**, and time units after **T**:

| Interval | Meaning |
| --- | --- |
| **P1D** | One calendar day |
| **P1W** | One week |
| **P1M** | One calendar month |
| **PT2H** | Two hours |
| **P1DT2H** | One day and two hours |

Use **add()** or **sub()** to apply an interval:

~~~php
<?php

$start = new DateTimeImmutable(
    '2026-04-10 09:00:00',
    new DateTimeZone('UTC')
);

$oneDayLater = $start->add(new DateInterval('P1D'));
$twoHoursEarlier = $start->sub(new DateInterval('PT2H'));

echo $oneDayLater->format('Y-m-d H:i:s') . "\n";
echo $twoHoursEarlier->format('Y-m-d H:i:s') . "\n";
~~~

A calendar month is not a fixed number of seconds or days. Adding a month near the end of a month can overflow into the following month. Decide what the product rule should be for cases such as “one month after January 31,” and implement that rule deliberately.

For repeated dates, **DatePeriod** can iterate from a start through a sequence of intervals:

~~~php
<?php

$start = new DateTimeImmutable('2026-04-01', new DateTimeZone('UTC'));
$end = new DateTimeImmutable('2026-04-16', new DateTimeZone('UTC'));
$weekly = new DateInterval('P7D');

$period = new DatePeriod($start, $weekly, $end);

foreach ($period as $date) {
    echo $date->format('Y-m-d') . "\n";
}
~~~

The start is included by default, while the end is excluded by default. Check those boundary rules when building a schedule.

## Compare dates and calculate differences

Date-time objects can be compared directly:

~~~php
<?php

$first = new DateTimeImmutable('2026-04-10 09:00:00', new DateTimeZone('UTC'));
$second = new DateTimeImmutable('2026-04-10 11:00:00', new DateTimeZone('UTC'));

var_dump($first < $second);
~~~

**diff()** returns a **DateInterval** describing the calendar difference between two date-time values:

~~~php
<?php

$first = new DateTimeImmutable('2026-04-10', new DateTimeZone('UTC'));
$second = new DateTimeImmutable('2026-04-13', new DateTimeZone('UTC'));

$interval = $first->diff($second);

echo $interval->format('%R%a days');
~~~

Use **%a** for the total number of full days in the interval. By default, the interval can be directional; **%R** includes its sign. The optional absolute argument can make the interval nonnegative.

### Calendar difference versus elapsed time

A local calendar day is not always exactly 24 elapsed hours. When a region changes its offset for daylight saving, a local clock may skip forward or repeat an hour. **diff()** is aware of these changes and describes how to move between local calendar values; its hour count may not represent a fixed number of elapsed seconds.

For elapsed time, compare Unix timestamps or first convert both values to UTC and compare there:

~~~php
<?php

$start = new DateTimeImmutable('2026-04-10 09:00:00', new DateTimeZone('UTC'));
$end = new DateTimeImmutable('2026-04-10 11:30:00', new DateTimeZone('UTC'));

$elapsedSeconds = $end->getTimestamp() - $start->getTimestamp();

echo $elapsedSeconds;
~~~

Unix timestamps count seconds from the Unix epoch. They are useful for elapsed-time arithmetic, while named time zones and calendar intervals are useful for human schedules.

## Set the default time zone

Set a known default at application startup if code relies on PHP's default time zone:

~~~php
<?php

date_default_timezone_set('UTC');

echo date('Y-m-d H:i:s');
~~~

Prefer passing an explicit **DateTimeZone** when parsing user input or creating values whose zone matters. A default setting affects operations that otherwise use the environment's default; it does not convert a date-time object that already has an explicit zone.

## Common mistakes

- Creating date-times without deciding which time zone their input uses.
- Assuming an abbreviation or a fixed offset contains a region's daylight-saving rules.
- Assuming **DateTimeImmutable** methods modify the original object.
- Assuming **createFromFormat()** rejects every overflowing value automatically.
- Forgetting that **m** means month and **i** means minute in format strings.
- Treating one calendar month or day as a fixed number of seconds.
- Using **diff()** when exact elapsed seconds are required across a daylight-saving transition.
- Forgetting **DatePeriod** includes its start and excludes its end by default.
- Relying on the machine's default time zone without configuring it.

## Try it yourself

1. Create one instant in UTC and display it in **Asia/Kolkata** and **America/New_York**.
2. Add **P1D** and **PT24H** to a local date near a daylight-saving transition and compare the results.
3. Parse **2024-02-29** with **createFromFormat()**, then try **2023-02-29** and inspect parse warnings.
4. Format the same value with **Y-m-d H:i:s P e** and **DateTimeInterface::ATOM**.
5. Use **DatePeriod** to print four weekly dates; decide whether the end date should be included.
6. Calculate an elapsed duration with timestamps and a calendar difference with **diff()**.
7. Explain how an explicit time zone makes parsing more predictable.

## Key takeaways

- Use **DateTimeImmutable** to make changes return a new date-time value.
- Use IANA time-zone identifiers for regional time rules and convert zones for display.
- Parse fixed input with **createFromFormat()**, check warnings, and validate the round trip.
- Use **DateInterval** and **DatePeriod** for calendar-based operations and schedules.
- Calendar days and elapsed durations can differ around daylight-saving transitions.
- Compare timestamps or UTC values when measuring exact elapsed seconds.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [050 - Regular Expressions with PCRE](050-regular-expressions-with-pcre.md) | [Open Q&A](../qnas/051-dates-times-and-time-zones.md) | [Open example](../examples/051-dates-times-and-time-zones.md) | [052 - Text Encoding, Localization, and Internationalization](052-text-encoding-localization-and-internationalization.md) |
