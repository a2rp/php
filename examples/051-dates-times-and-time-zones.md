# 051 - Dates, Times, and Time Zones: Complete Example

This example creates date-time objects with explicit time zones, formats and converts them, validates a fixed-format date, adds intervals, compares calendar time with elapsed time across a daylight-saving change, and generates a weekly schedule.

## Goal

Build a small schedule report that makes date-time assumptions visible instead of relying on the server's default time zone.

## Requirements

- PHP 8.0 or later.
- The PHP time-zone database must include the IANA zones used below, including **UTC**, **Asia/Kolkata**, and **America/New_York**.
- The date-validation code handles the different no-error return values from **getLastErrors()** before and after PHP 8.2.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

function isValidDate(string $input, DateTimeZone $timezone): bool
{
    // The leading ! resets omitted time fields to midnight.
    $date = DateTimeImmutable::createFromFormat('!Y-m-d', $input, $timezone);
    $errors = DateTimeImmutable::getLastErrors();

    $hasProblems = $errors !== false
        && ($errors['warning_count'] > 0 || $errors['error_count'] > 0);

    // A round-trip check rejects dates the parser normalized by overflow.
    return $date !== false
        && !$hasProblems
        && $date->format('Y-m-d') === $input;
}

// Interpret one known instant in UTC, then display it in India.
$utc = new DateTimeZone('UTC');
$india = new DateTimeZone('Asia/Kolkata');

$instant = new DateTimeImmutable('2026-04-10 03:30:00', $utc);
$indiaDisplay = $instant->setTimezone($india);

echo "Same instant in two zones:\n";
echo $instant->format('Y-m-d H:i:s P e') . "\n";
echo $indiaDisplay->format('Y-m-d H:i:s P e') . "\n";

// Immutable arithmetic returns new objects; the original still exists.
$start = new DateTimeImmutable('2026-04-10 09:00:00', $utc);
$nextDay = $start->add(new DateInterval('P1D'));
$twoHoursEarlier = $start->sub(new DateInterval('PT2H'));

echo "Immutable interval operations:\n";
echo 'Original: ' . $start->format('Y-m-d H:i:s') . "\n";
echo 'One day later: ' . $nextDay->format('Y-m-d H:i:s') . "\n";
echo 'Two hours earlier: ' . $twoHoursEarlier->format('Y-m-d H:i:s') . "\n";

// Format tokens are case-sensitive: m is month, i is minute.
$meeting = new DateTimeImmutable(
    '2026-04-10 09:05:00',
    $india
);

echo "Formatted meeting:\n";
echo $meeting->format('Y-m-d H:i:s P e') . "\n";
echo $meeting->format(DateTimeInterface::ATOM) . "\n";

// Parse a fixed date format and reject calendar overflows.
$validInput = '2024-02-29';
$invalidInput = '2023-02-29';

echo "Date validation:\n";
echo $validInput . ': ' . (isValidDate($validInput, $utc) ? 'valid' : 'invalid') . "\n";
echo $invalidInput . ': ' . (isValidDate($invalidInput, $utc) ? 'valid' : 'invalid') . "\n";

// A calendar day and 24 elapsed hours differ across this DST change.
$newYork = new DateTimeZone('America/New_York');
$beforeSpringForward = new DateTimeImmutable('2021-03-13 12:00:00', $newYork);

$oneCalendarDayLater = $beforeSpringForward->add(new DateInterval('P1D'));
$24ElapsedHoursLater = $beforeSpringForward->add(new DateInterval('PT24H'));

echo "Daylight-saving comparison:\n";
echo 'Start: ' . $beforeSpringForward->format('Y-m-d H:i:s P') . "\n";
echo 'One calendar day later: ' . $oneCalendarDayLater->format('Y-m-d H:i:s P') . "\n";
echo 'After 24 elapsed hours: ' . $24ElapsedHoursLater->format('Y-m-d H:i:s P') . "\n";
echo 'Elapsed seconds in that calendar day: '
    . ($oneCalendarDayLater->getTimestamp() - $beforeSpringForward->getTimestamp())
    . "\n";

// A DatePeriod includes its start by default and excludes its end by default.
$scheduleStart = new DateTimeImmutable('2026-04-01', $utc);
$scheduleEnd = new DateTimeImmutable('2026-04-23', $utc);
$week = new DateInterval('P7D');
$weeklySchedule = new DatePeriod($scheduleStart, $week, $scheduleEnd);

echo "Weekly schedule:\n";

foreach ($weeklySchedule as $date) {
    echo $date->format('Y-m-d') . "\n";
}
~~~

## Expected output

~~~text
Same instant in two zones:
2026-04-10 03:30:00 +00:00 UTC
2026-04-10 09:00:00 +05:30 Asia/Kolkata
Immutable interval operations:
Original: 2026-04-10 09:00:00
One day later: 2026-04-11 09:00:00
Two hours earlier: 2026-04-10 07:00:00
Formatted meeting:
2026-04-10 09:05:00 +05:30 Asia/Kolkata
2026-04-10T09:05:00+05:30
Date validation:
2024-02-29: valid
2023-02-29: invalid
Daylight-saving comparison:
Start: 2021-03-13 12:00:00 -05:00
One calendar day later: 2021-03-14 12:00:00 -04:00
After 24 elapsed hours: 2021-03-14 13:00:00 -04:00
Elapsed seconds in that calendar day: 82800
Weekly schedule:
2026-04-01
2026-04-08
2026-04-15
2026-04-22
~~~

## How it works

### Convert for display

The **$instant** object starts at a known UTC time. **setTimezone()** returns a new object for the same instant in **Asia/Kolkata**. The local clock advances by five hours and thirty minutes, but the moment represented is unchanged.

The example passes explicit zones instead of relying on the machine's default. This is especially important when parsing a local time or creating an event for a particular region.

### Immutable arithmetic and formatting

**add()** and **sub()** return new objects, leaving **$start** untouched. The **P1D** interval is one calendar day; **PT2H** is two hours.

The format strings use **m** for month and **i** for minutes. **P** prints a numeric offset and **e** prints the time-zone identifier. **DateTimeInterface::ATOM** supplies a standard ISO-style date-time representation with its offset.

### Validate a date-only string

The helper parses the exact **Y-m-d** format and passes the intended zone. The leading **!** resets omitted time fields to midnight. It checks both parser warnings and errors and then formats the value back to compare it with the original input.

The parser can normalize an overflowing date such as **2023-02-29** into a later date. Checking only whether an object was returned would incorrectly accept that input.

### Calendar day versus elapsed time

On March 14, 2021, **America/New_York** moved its clock forward for daylight saving. Adding **P1D** preserves the local noon on the next calendar day, which is only **82,800** elapsed seconds later. Adding **PT24H** advances by a full 86,400 seconds, reaching 13:00 local time.

Choose calendar arithmetic for rules such as “same local time tomorrow.” Use timestamps or UTC values when the requirement is an exact elapsed duration.

### Recurring schedule

**DatePeriod** iterates from the start date in one-week steps. Its start is included by default and its end is excluded by default, so April 22 is included while April 23 is not.

## Try it yourself

1. Display the same UTC instant in **Europe/London** and compare its offset at different dates.
2. Change the date parser input to **2024-02-31** and observe why warnings and round-trip validation matter.
3. Change the interval to **P1M** and test dates near the end of a month.
4. Set the schedule end to April 15 and confirm the default end boundary.
5. Compare the elapsed seconds for **P1D** and **PT24H** across the daylight-saving transition.
6. Change the display format to include the weekday name and time-zone offset.

## Common mistakes

- Relying on the server's default zone to interpret input.
- Thinking **setTimezone()** changes the instant.
- Forgetting to retain the new value returned by **DateTimeImmutable** methods.
- Assuming **createFromFormat()** always rejects overflowing dates.
- Confusing **m** (month) with **i** (minute).
- Treating one local calendar day as always 86,400 seconds.
- Forgetting that **DatePeriod** excludes its end by default.

## Key takeaways

- Specify the time zone when interpreting or creating date-time values.
- Convert zones for display without changing the represented instant.
- Parse fixed formats carefully and reject warnings or normalized overflow.
- Use **DateInterval** and **DatePeriod** for calendar operations and recurring schedules.
- Use timestamps or UTC arithmetic for exact elapsed durations across time-zone changes.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [050 - Regular Expressions with PCRE](../lessons/050-regular-expressions-with-pcre.md) | [Read lesson](../lessons/051-dates-times-and-time-zones.md) | [Open Q&A](../qnas/051-dates-times-and-time-zones.md) | [052 - Text Encoding, Localization, and Internationalization](../lessons/052-text-encoding-localization-and-internationalization.md) |
