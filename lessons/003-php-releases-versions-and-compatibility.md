# 003 - PHP Releases, Versions, and Compatibility

PHP changes over time. New releases can add features and improve the runtime, while a project may need to keep working on a particular version. Understanding release numbers and support periods helps you choose a runtime and plan upgrades.

This chapter explains PHP version numbers, point releases, support stages, and practical compatibility checks. Version support changes over time, so the support table below is a dated snapshot.

## What you will learn

- How to read a PHP version number
- The difference between a release branch and a point release
- How active support, security support, and end of life differ
- What can make a project incompatible with a PHP version
- How to plan and check an upgrade

## Reading a PHP version number

A version such as **8.6.0** has three parts:

| Part | Example | Meaning |
| --- | --- | --- |
| Major | 8 | The major version family |
| Minor | 6 | A release line within the major family |
| Point | 0 | A particular release within that line |

The **8.6** release branch contains point releases such as 8.6.0, 8.6.1, and later maintenance releases. The point number starts at zero for the initial release and may increase as the branch receives updates.

PHP typically adds language features in major or minor releases and fixes bugs in point releases. This is a convention, not an absolute guarantee. Read release notes before upgrading, especially when changing the major or minor line.

## Release branches and point releases

A **release branch** is a line such as PHP 8.5 or PHP 8.6. It may receive multiple point releases during its support lifetime.

A **point release** is a particular build within a branch, such as 8.6.1. Point releases can include bug fixes and security fixes. When a branch receives an important fix, update to a fixed point release instead of assuming every release with the same first two numbers is identical.

The version installed on a computer, server, or hosting account is the version that actually runs the application. The version used by a terminal may differ from the version configured for a website, so check the environment that will run the code.

## PHP support stages

PHP's release branches have defined support periods:

1. **Active support:** for two years after the initial stable release, reported bugs and security issues are fixed through regular point releases.
2. **Security support:** for two more years, only critical security issues are addressed, and releases are made as needed.
3. **End of life:** after the support period ends, the branch no longer receives PHP fixes.

A branch that receives security fixes only is still supported, but it is no longer in active support. An end-of-life branch may continue to run, but it no longer receives official security fixes. Running an EOL branch increases the risk that newly found vulnerabilities remain unpatched.

## Support snapshot checked on November 20, 2026

The PHP 8.6.0 stable release was announced on November 19, 2026. The following table summarizes the PHP branches shown as supported on November 20, 2026:

| Branch | Status on November 20, 2026 | Active support ends | Security support ends |
| --- | --- | --- | --- |
| 8.2 | Security fixes only | December 31, 2024 | December 31, 2026 |
| 8.3 | Security fixes only | December 31, 2025 | December 31, 2027 |
| 8.4 | Active support | December 31, 2026 | December 31, 2028 |
| 8.5 | Active support | December 31, 2027 | December 31, 2029 |
| 8.6 | Active support | December 31, 2028 | December 31, 2030 |

Treat this as a dated snapshot, not a permanent list. Before choosing a version for a new or existing project, check PHP's current support schedule and the hosting provider's available versions. The current point release also changes over time.

## What compatibility means

An application is compatible with a PHP version when its code, configuration, extensions, and dependencies work correctly on that runtime.

Compatibility can be affected by several things:

- **Language features:** code may use syntax or functions introduced after an older runtime was released.
- **Behavior changes:** an operation can behave differently between versions.
- **Removed features:** code that depended on an old feature may stop working after it is removed.
- **Deprecated features:** a feature still works for now but is marked for future removal. Deprecation warnings give maintainers time to replace it.
- **Extensions:** an application may require extensions that are absent or different on a server.
- **Configuration:** runtime settings can affect behavior.
- **Dependencies:** libraries and frameworks can declare which PHP versions they support.

Compatibility is not just a question of whether PHP can parse one file. The complete application must be checked in its real environment.

## How to choose a version

For a new project, choose a maintained PHP branch that is available in the deployment environment and supported by the libraries the project needs. Prefer a branch with active support when practical, then keep it updated to the latest appropriate point release.

For an existing project, do not choose solely by asking which version number is newest. Check:

1. Which PHP versions the application currently supports.
2. Which PHP versions its dependencies require or allow.
3. Which branches the host or production platform provides.
4. What changed between the current and target versions.
5. Whether the application works after the upgrade.

A project may need to support more than one PHP version. In that case, use only language features available in the oldest version you claim to support, and check the project on each supported runtime.

## A careful upgrade process

Use a repeatable process instead of changing a production runtime without preparation:

1. Record the current PHP runtime and the target branch and point release.
2. Read the migration notes for every version line crossed.
3. Check application and dependency version requirements.
4. Update the local development or staging runtime to match the target.
5. Review deprecation warnings and fix them where practical.
6. Run the project's automated checks and exercise important user flows.
7. Deploy the target runtime in a controlled way and monitor errors.
8. Keep the runtime and dependencies updated after the upgrade.

If an application jumps across several release branches, review the changes for each branch rather than only the final one.

## Checking the runtime

The command below prints the PHP version available to the terminal:

~~~sh
php -v
~~~

A PHP script can also inspect its runtime:

~~~php
<?php

echo PHP_VERSION, PHP_EOL;
echo PHP_VERSION_ID, PHP_EOL;
~~~

**PHP_VERSION** contains the version as text, such as 8.6.0. **PHP_VERSION_ID** contains an integer representation that is convenient for version checks. Later lessons cover running PHP commands and writing conditional logic in more depth.

A command-line PHP installation is not necessarily the same runtime used by a web server. Check both the CLI and web deployment environments when the project uses both.

## Version checks do not prove compatibility

A check such as “PHP is at least 8.2” tells you that a runtime meets a minimum version threshold. It does not prove that every library, extension, setting, or application behavior is compatible. Version numbers are one part of the check; tests and deployment checks are still needed.

Also, a runtime version check cannot protect older PHP from syntax it cannot parse. If a file contains syntax introduced in a newer version, PHP may fail while parsing the file before the program can run a version condition inside it. Declare version requirements and use compatible code before running the application.

## Try it yourself

1. In the version 8.6.0, identify the major, minor, and point numbers.
2. Explain how active support differs from security-only support.
3. What happens when a PHP branch reaches end of life?
4. Name three things besides PHP syntax that can affect application compatibility.
5. Why should deprecation warnings be reviewed before upgrading?
6. Why might checking the terminal's PHP version be insufficient for a website?
7. Outline a safe way to prepare an application for a newer PHP branch.

## Key points

- A PHP version has a major, minor, and point component.
- A release branch can receive multiple point releases.
- PHP branches move from active support to security-only support and then end of life.
- The PHP support table changes, so version status should be checked when making upgrade decisions.
- Compatibility includes code, behavior, extensions, configuration, dependencies, and the deployment environment.
- Read migration notes, check requirements, and verify the application before deploying a new runtime.
- A version check alone does not prove that an application works on that version.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [002 - How PHP fits into a web application](002-how-php-fits-into-a-web-application.md) | [Open Q&A](../qnas/003-php-releases-versions-and-compatibility.md) | [Open example](../examples/003-php-releases-versions-and-compatibility.md) | [004 - Install PHP and inspect the runtime](004-install-php-and-inspect-the-runtime.md) |
