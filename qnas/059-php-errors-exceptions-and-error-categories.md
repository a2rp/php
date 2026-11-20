# 059 - PHP Errors, Exceptions, and Error Categories: Q&A

Review diagnostic levels, the distinction between diagnostics and Throwable objects, bitmask configuration, and PHP version changes. Try each question before reading its answer. See the [chapter lesson](../lessons/059-php-errors-exceptions-and-error-categories.md).

## Multiple-choice questions

### 1. What is E_WARNING?

A. An Exception subclass
B. A diagnostic severity for a runtime warning
C. A syntax error class
D. A synonym for TypeError

**Answer:** B. A diagnostic severity for a runtime warning.

**Explanation:** E_WARNING is a diagnostic constant, not a Throwable class.

### 2. What is the main difference between E_WARNING and TypeError?

A. They are the same category
B. E_WARNING is a diagnostic level; TypeError is a Throwable object
C. TypeError is a diagnostic mask
D. E_WARNING always stops the program

**Answer:** B. E_WARNING is a diagnostic level; TypeError is a Throwable object.

**Explanation:** A diagnostic may be displayed or logged; a TypeError is thrown and can be caught.

### 3. Which level represents a syntax parsing failure?

A. E_NOTICE
B. E_PARSE
C. E_DEPRECATED
D. E_USER_WARNING

**Answer:** B. E_PARSE.

**Explanation:** A parse error prevents the affected source file from executing.

### 4. What does E_DEPRECATED generally signal?

A. A feature or behavior may be removed in a future PHP version
B. A fatal memory error always occurred
C. A user requested a warning
D. The file is encoded in the wrong character set

**Answer:** A. A feature or behavior may be removed in a future PHP version.

**Explanation:** Deprecation diagnostics give maintainers a chance to update code before upgrading.

### 5. Which level is normally used to create an application-generated warning with trigger_error()?

A. E_WARNING
B. E_USER_WARNING
C. E_PARSE
D. E_CORE_ERROR

**Answer:** B. E_USER_WARNING.

**Explanation:** User-generated trigger_error() levels use the E_USER_* constants.

### 6. What is E_ALL?

A. A Throwable class
B. A bitmask containing diagnostic levels
C. A setting that forces all errors to display in a web page
D. A function that writes logs

**Answer:** B. A bitmask containing diagnostic levels.

**Explanation:** It selects levels for reporting; it does not by itself control display or logging.

### 7. How are two diagnostic levels combined in PHP code?

A. With logical OR only
B. With bitwise OR
C. By adding their names as strings
D. With a comma

**Answer:** B. With bitwise OR.

**Explanation:** For example, E_WARNING | E_USER_WARNING combines the two bit flags.

### 8. Which configuration is a suitable production web default?

A. display_errors On and log_errors Off
B. display_errors Off and log_errors On
C. error_reporting 0 and log_errors Off
D. Show traces to every visitor

**Answer:** B. display_errors Off and log_errors On.

**Explanation:** Keep internal details out of public responses and record useful diagnostics in protected logs.

### 9. What does error_reporting() configure?

A. Which diagnostic levels are reported
B. Whether a Throwable is an Exception or Error
C. Whether a file can be parsed
D. The location of all PHP source files

**Answer:** A. Which diagnostic levels are reported.

**Explanation:** Display and logging behavior are controlled separately by configuration and handlers.

### 10. What does display_errors control?

A. Whether diagnostics are printed in script output
B. Which classes can be thrown
C. Whether error constants exist
D. Whether logs are encrypted

**Answer:** A. Whether diagnostics are printed in script output.

**Explanation:** Production web applications should generally avoid displaying internal diagnostics to users.

### 11. What does log_errors control?

A. Whether diagnostics are written to the configured error log
B. Whether every warning becomes an exception
C. Whether PHP loads a file
D. Whether a catch block runs

**Answer:** A. Whether diagnostics are written to the configured error log.

**Explanation:** The error log destination is configured separately and should be protected.

### 12. As of PHP 8.4, what is the status of E_STRICT?

A. Required for every application
B. Unused and deprecated
C. A new Throwable subclass
D. The only level that E_ALL excludes

**Answer:** B. Unused and deprecated.

**Explanation:** Old tutorials may show it, but modern PHP code should not rely on E_STRICT.

### 13. As of PHP 8.4, what is deprecated when using trigger_error()?

A. E_USER_WARNING
B. E_USER_NOTICE
C. E_USER_ERROR
D. E_USER_DEPRECATED

**Answer:** C. E_USER_ERROR.

**Explanation:** New code should throw an exception or call exit() when deliberate termination is intended.

### 14. Which statement best describes E_RECOVERABLE_ERROR in current PHP?

A. It is the main way modern type failures are thrown
B. It is effectively unused
C. It is equivalent to E_USER_NOTICE
D. It is required for every exception

**Answer:** B. It is effectively unused.

**Explanation:** Current PHP generally represents engine-level programming failures with Throwable types such as TypeError.

### 15. What does trigger_error() do with an unsupported error-level argument in PHP 8 or later?

A. Silently ignores it
B. Throws ValueError
C. Converts it to E_ALL
D. Returns null

**Answer:** B. It throws ValueError.

**Explanation:** Use one of the supported E_USER_* levels and check the PHP version targeted by the project.

## Short-answer questions

### 16. What are PHP's two broad error-reporting mechanisms?

**Answer:** Severity-based diagnostic errors and Throwable objects.

**Explanation:** Diagnostics are reported by level; Throwable objects are thrown and handled with try/catch.

### 17. Why is E_WARNING not the same as an Exception?

**Answer:** E_WARNING is a diagnostic severity constant, while an Exception is an object implementing Throwable.

**Explanation:** A function may emit a warning and return false without throwing an Exception.

### 18. Why can't code in a file catch a parse error that prevents that file from executing?

**Answer:** PHP must parse the file before executing its statements, so the file's try/catch code never runs.

**Explanation:** Configure startup and parse diagnostics outside the affected script, such as in PHP or server configuration.

### 19. Why should code use E_ALL instead of a copied numeric mask?

**Answer:** PHP can add or change diagnostic levels, so the numeric value of the mask may differ across releases.

**Explanation:** Named constants and bitwise operators are clearer and more compatible.

### 20. What should a production site do with internal diagnostics?

**Answer:** Avoid displaying them to visitors and send them to a protected error log.

**Explanation:** Error output may reveal paths, SQL, configuration, or secrets.

### 21. Does error_reporting(E_ALL) turn warnings into exceptions?

**Answer:** No.

**Explanation:** It selects diagnostic levels to report. A deliberate custom error handler is needed to convert selected diagnostics into ErrorException.

### 22. What should new code use instead of trigger_error(..., E_USER_ERROR) on PHP 8.4 or later?

**Answer:** Throw a meaningful exception, or call exit() when intentional script termination is appropriate.

**Explanation:** Using E_USER_ERROR with trigger_error() is deprecated as of PHP 8.4.

### 23. What does E_USER_WARNING represent?

**Answer:** An application-generated warning diagnostic, commonly raised with trigger_error().

**Explanation:** It remains a diagnostic rather than becoming a custom Exception object automatically.

### 24. What reporting level is recommended for development?

**Answer:** E_ALL.

**Explanation:** It helps reveal warnings, notices, deprecations, and other issues while they can still be fixed.

### 25. What is the difference between display_errors and log_errors?

**Answer:** display_errors controls output to the script response; log_errors controls writing diagnostics to the configured log.

**Explanation:** These settings have separate purposes and should be configured deliberately.

### 26. Can a non-fatal diagnostic be followed by a false return value?

**Answer:** Yes.

**Explanation:** The diagnostic and the function result are distinct signals; inspect the documented return value.

### 27. Why should code avoid hard-coding the integer value of E_WARNING or E_ALL?

**Answer:** Numeric values are implementation constants and may change or differ as PHP evolves.

**Explanation:** Use the symbolic constant names instead.

### 28. Why should PHP error categories be checked during a version upgrade?

**Answer:** Diagnostics may be added, removed, or changed in severity, and some conditions can become Throwable errors.

**Explanation:** Running supported code with E_ALL helps catch compatibility issues before release.

## True or false

### 29. E_WARNING is a class that can be caught with catch (E_WARNING $exception).

**Answer:** False.

**Explanation:** E_WARNING is an integer diagnostic level, not a class.

### 30. E_PARSE represents a parsing problem that prevents the affected file from executing.

**Answer:** True.

**Explanation:** The parser must successfully parse the file before its statements can run.

### 31. E_ALL controls whether diagnostics are shown in the browser.

**Answer:** False.

**Explanation:** E_ALL selects reported levels. display_errors controls whether diagnostics are printed into output.

### 32. The numeric value of E_ALL should be copied into long-lived application code.

**Answer:** False.

**Explanation:** Use E_ALL by name because error-level bitmasks can evolve.

### 33. E_STRICT is unused and deprecated as of PHP 8.4.

**Answer:** True.

**Explanation:** It is a legacy diagnostic level and should not be relied on in modern PHP.

### 34. Using E_USER_ERROR with trigger_error() is deprecated as of PHP 8.4.

**Answer:** True.

**Explanation:** Use an exception or exit() instead, according to the intended control flow.

### 35. Every E_WARNING automatically stops the PHP script.

**Answer:** False.

**Explanation:** E_WARNING is non-fatal in normal handling, though the attempted operation can still fail.

### 36. A TypeError is a Throwable object rather than a diagnostic-level constant.

**Answer:** True.

**Explanation:** TypeError is an Error subclass that can be handled by try/catch.

## Predict the result or behavior

### 37. What does this print?

~~~php
<?php

$levels = E_WARNING | E_USER_WARNING;
echo ($levels & E_WARNING) !== 0 ? 'included' : 'missing';
~~~

**Answer:**

~~~text
included
~~~

**Explanation:** Bitwise OR combines the levels, and bitwise AND checks whether E_WARNING is present.

### 38. Which diagnostic category is raised?

~~~php
trigger_error('Profile value is missing.', E_USER_NOTICE);
~~~

**Answer:** E_USER_NOTICE.

**Explanation:** This is an application-generated diagnostic. It does not automatically create or throw an Exception.

### 39. A function emits E_WARNING and returns false. What does var_dump($result) show if $result contains that return value?

**Answer:**

~~~text
bool(false)
~~~

**Explanation:** The warning may be displayed or logged separately. The function's return value must still be checked.

### 40. What does the condition print?

~~~php
<?php

$mask = E_ALL & ~E_DEPRECATED;
var_dump(($mask & E_DEPRECATED) === 0);
~~~

**Answer:**

~~~text
bool(true)
~~~

**Explanation:** The bitwise NOT mask removes E_DEPRECATED from the selected levels.

### 41. In PHP 8.4, what compatibility issue does this line introduce?

~~~php
trigger_error('Fatal application problem.', E_USER_ERROR);
~~~

**Answer:** Passing E_USER_ERROR to trigger_error() is deprecated as of PHP 8.4.

**Explanation:** New code should throw an exception or call exit() when stopping execution is intentional.

### 42. Can this catch block handle E_WARNING?

~~~php
try {
    performOperation();
} catch (ErrorException $exception) {
    echo 'caught';
}
~~~

**Answer:** Not by itself.

**Explanation:** A diagnostic is not automatically an ErrorException. A custom error handler must deliberately convert selected diagnostic levels into that Throwable type.

## Coding questions

### 43. Show a PHP configuration for development that reports all levels and displays diagnostics.

**Answer:**

~~~ini
error_reporting = E_ALL
display_errors = On
log_errors = On
~~~

**Explanation:** This makes issues visible while developing and can also retain logs. Do not use display_errors = On for a public production site.

### 44. Combine E_WARNING and E_USER_WARNING and check whether E_WARNING is selected.

**Answer:**

~~~php
$mask = E_WARNING | E_USER_WARNING;

if (($mask & E_WARNING) !== 0) {
    echo 'E_WARNING is selected.';
}
~~~

**Explanation:** Error levels are bit flags. Use named constants and bitwise operations instead of numeric masks.

### 45. Replace a deprecated E_USER_ERROR trigger with a Throwable that a caller can catch.

**Answer:**

~~~php
if (!$configurationIsValid) {
    throw new RuntimeException(
        'The application configuration is invalid.'
    );
}
~~~

**Explanation:** A caller can catch the specific exception type at a layer with a recovery policy. For intentional process termination, exit() is the other documented alternative.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [058 - try, catch, finally, and throwing exceptions](../lessons/058-try-catch-finally-and-throwing-exceptions.md) | [Read lesson](../lessons/059-php-errors-exceptions-and-error-categories.md) | [Open example](../examples/059-php-errors-exceptions-and-error-categories.md) | [060 - Error reporting, handlers, and assertions](../lessons/060-error-reporting-handlers-and-assertions.md) |
