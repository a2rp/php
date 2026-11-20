# 003 - PHP Releases, Versions, and Compatibility: Questions and Answers

Review [003 - PHP Releases, Versions, and Compatibility](../lessons/003-php-releases-versions-and-compatibility.md) with these questions. Answers and explanations follow each question.

## Multiple-choice questions

### 1. In PHP version 8.6.0, what does the final zero represent?

A. The major version<br>
B. The minor version<br>
C. The point release number<br>
D. The support status

**Answer: C.** In 8.6.0, 8 is the major component, 6 is the minor component, and 0 is the point component.

### 2. What is a PHP release branch?

A. A line such as PHP 8.6 that can receive multiple point releases<br>
B. A database table<br>
C. A single PHP source file<br>
D. A hosting account

**Answer: A.** A branch such as 8.6 is a release line. Versions such as 8.6.0 and later point releases belong to that line.

### 3. What is the typical purpose of a point release?

A. To replace all PHP versions<br>
B. To deliver maintenance updates such as bug or security fixes<br>
C. To guarantee every application is compatible<br>
D. To add a database to PHP

**Answer: B.** Point releases typically focus on fixes. Read the release information because conventions are not absolute and a point update can still matter to an application.

### 4. During active support, what kind of issues does PHP address?

A. Reported bugs and security issues through regular point releases<br>
B. Only new language features<br>
C. Only problems reported by one hosting provider<br>
D. No issues until end of life

**Answer: A.** Active support is the first support period for a branch and includes bug and security fixes.

### 5. What does security-only support mean?

A. The branch receives every new language feature<br>
B. Only critical security issues are addressed, on an as-needed basis<br>
C. The branch can no longer run<br>
D. The branch receives no fixes of any kind

**Answer: B.** Security-only support follows active support. The branch remains supported for critical security issues, but regular bug fixes are no longer part of the active period.

### 6. What happens when a branch reaches end of life?

A. It receives more frequent bug fixes<br>
B. It is guaranteed to work with all new dependencies<br>
C. It no longer receives official PHP fixes<br>
D. It automatically upgrades itself

**Answer: C.** An end-of-life branch no longer receives PHP fixes, including security fixes. Continuing to use it can leave known vulnerabilities unpatched.

### 7. Which item can affect application compatibility besides PHP syntax?

A. Required extensions<br>
B. Runtime configuration<br>
C. Library requirements<br>
D. All of the above

**Answer: D.** Compatibility depends on the runtime, extensions, configuration, dependencies, and application behavior, not just whether one file parses.

### 8. What is a deprecation warning?

A. A warning that a feature is discouraged and may be removed in a future version<br>
B. Proof that PHP is not installed<br>
C. A successful database connection<br>
D. A message that changes the version number

**Answer: A.** A deprecation warning gives maintainers notice that they should move away from a feature before it may be removed.

### 9. Why might php -v in a terminal not tell you which version a website uses?

A. The terminal cannot display PHP versions<br>
B. The web server may use a different PHP runtime from the command line<br>
C. PHP versions only exist in browsers<br>
D. Point releases are invisible to servers

**Answer: B.** Command-line PHP and the web server's configured PHP runtime can differ. Check the environment that actually runs the application.

### 10. Why does a version check inside a script not always protect older PHP from newer syntax?

A. The old runtime may fail while parsing the file before the check runs<br>
B. PHP never checks version numbers<br>
C. Version checks change the installed runtime<br>
D. New syntax is always ignored

**Answer: A.** PHP must parse the file before executing its statements. If the parser does not understand syntax in that file, an in-script condition cannot prevent the parse failure.

### 11. What does PHP_VERSION_ID provide?

A. An integer representation of the current PHP version<br>
B. The name of the web server<br>
C. The database password<br>
D. The list of installed extensions

**Answer: A.** PHP_VERSION_ID is useful for numeric version comparisons. PHP_VERSION provides the version as text.

### 12. Which is the best upgrade approach?

A. Upgrade production immediately without reading release notes<br>
B. Check requirements, review migration notes, test in a matching environment, then deploy carefully<br>
C. Ignore dependency requirements if the site starts<br>
D. Keep using an end-of-life version forever

**Answer: B.** A controlled upgrade checks compatibility before deployment and monitors the application after the runtime changes.

## True or false

### 1. PHP typically adds features in major or minor releases and fixes bugs in point releases.

**Answer: True, as a general convention.** The PHP manual describes this as typical behavior, but it is not an absolute promise. Check release notes for the exact version.

### 2. If two PHP versions have the same major number, all code is guaranteed to behave identically.

**Answer: False.** Behavior and available features can change between minor versions, and point releases can also matter. Compatibility must be checked rather than assumed.

### 3. On November 20, 2026, PHP 8.2 was in active support.

**Answer: False.** In the support snapshot for that date, PHP 8.2 was in security-only support and its security support was scheduled to end on December 31, 2026.

### 4. On November 20, 2026, PHP 8.6.0 had been released.

**Answer: True.** PHP 8.6.0 was announced on November 19, 2026. The dated snapshot in the lesson records that release.

### 5. Passing a minimum PHP version check proves that every dependency works.

**Answer: False.** A minimum version check verifies only a version threshold. Dependencies, extensions, configuration, and behavior still need to be checked.

### 6. An application may run in both CLI and web environments that use different PHP versions.

**Answer: True.** Separate PHP installations or server configuration can make the command-line runtime differ from the runtime serving the website.

## Short-answer questions

### 1. Break down PHP 8.6.0 into its version components.

**Answer:** Major version 8, minor version 6, point release 0.

### 2. What is the difference between active support and security-only support?

**Answer:** Active support includes reported bug and security fixes through regular point releases. Security-only support addresses critical security issues on an as-needed basis.

### 3. Why is an end-of-life PHP version a concern?

**Answer:** It no longer receives official PHP fixes. If a new security problem is found, the old branch may remain vulnerable.

### 4. Name four areas to review when checking compatibility.

**Answer:** Application code, required PHP extensions, runtime configuration, and library or framework requirements.

**Explanation:** Other relevant areas include behavior changes and the production hosting environment.

### 5. Why should a developer read migration notes before upgrading?

**Answer:** Migration notes describe new behavior, deprecated features, and backward-incompatible changes that can affect existing code.

### 6. Why should deprecation warnings be addressed before a future upgrade?

**Answer:** A deprecated feature may be removed later. Replacing it early reduces migration work and helps prevent the application from breaking on a newer branch.

### 7. What is the difference between PHP_VERSION and PHP_VERSION_ID?

**Answer:** PHP_VERSION is a text version such as 8.6.0. PHP_VERSION_ID is an integer representation useful for numeric checks.

### 8. Why is a compatibility check more than comparing version numbers?

**Answer:** The app also depends on language behavior, extensions, configuration, dependencies, and the actual environment where it runs.

## Version and support practice

### 1. What are the components in version 8.5.3?

**Answer:** Major 8, minor 5, point 3.

### 2. According to the support snapshot dated November 20, 2026, which status applied to PHP 8.4?

**Answer:** PHP 8.4 was in active support, scheduled to end on December 31, 2026. Security support was scheduled through December 31, 2028.

### 3. A project runs on PHP 8.2 and must keep receiving PHP security fixes. What should its maintainers plan?

**Answer:** They should plan a move to a branch that remains supported after PHP 8.2 reaches end of life, while checking that the app, dependencies, extensions, and host support the target.

**Explanation:** In the dated snapshot, PHP 8.2 security support ended on December 31, 2026. The exact current support state should be checked at upgrade time because the schedule changes.

### 4. A command-line version check says PHP 8.6, but the hosted site behaves like an older version. Give a likely reason.

**Answer:** The command line and web server are using different PHP runtimes or configurations. Check the version configured for the website as well as the terminal version.

### 5. A script contains a feature introduced in PHP 8.6 and begins with a condition that checks whether PHP is at least 8.6. Will that condition necessarily protect PHP 8.5?

**Answer: No.** PHP 8.5 may fail to parse the newer syntax before it can execute the condition. The project must declare and use compatible requirements before running the file.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [002 - How PHP fits into a web application](../lessons/002-how-php-fits-into-a-web-application.md) | [Read this lesson](../lessons/003-php-releases-versions-and-compatibility.md) | [Open example](../examples/003-php-releases-versions-and-compatibility.md) | [004 - Install PHP and inspect the runtime](../lessons/004-install-php-and-inspect-the-runtime.md) |
