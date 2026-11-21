# 110 - PHP-FIG standards and PSR recommendations

## Goals

After this lesson, you should be able to:

- Explain what PHP-FIG and a PSR are for.
- Distinguish an accepted recommendation from a rule that every PHP project must follow.
- Recognize the accepted PSRs most often encountered in application and package work.
- Use an interface standard without coupling application code to one implementation.
- Check whether a standard is accepted, draft, deprecated, or abandoned.
- Apply a style guide consistently without mistaking formatting rules for application architecture.
- Adopt standards gradually while preserving compatibility with a project's existing code.

## What is PHP-FIG?

PHP-FIG means the PHP Framework Interoperability Group. It is a group of PHP project representatives who discuss common interfaces and practices that can help separate libraries and frameworks work together.

PSR means PHP Standard Recommendation. A PSR is a documented proposal for a shared contract or convention. Examples include a logging interface, an autoloading rule, and HTTP message interfaces.

PHP-FIG does not define the PHP language, decide how every application must be structured, or require every project to adopt every accepted PSR. A project chooses the standards that fit its goals, dependencies, and compatibility requirements.

## PSR status matters

A PSR has a status during its development. An accepted PSR is the group's accepted specification. A draft is still open to change. Deprecated and abandoned proposals should not be treated as current recommendations for new code without checking their replacement or context.

The status snapshot below follows the PHP-FIG index as checked on 2026-11-21. Statuses can change, so check the current index when adopting a specification in a real project. This tutorial intentionally keeps external reference URLs out of its files.

## Accepted PSRs at a glance

| PSR | Subject | What it helps standardize |
| --- | --- | --- |
| PSR-1 | Basic Coding Standard | Basic source file and naming conventions for interoperable PHP code |
| PSR-3 | Logger Interface | A shared logging interface and log-level methods |
| PSR-4 | Autoloading Standard | Mapping namespaces to file paths for autoloading |
| PSR-6 | Caching Interface | A cache-pool and cache-item contract |
| PSR-7 | HTTP Message Interface | Request, response, URI, stream, and uploaded-file interfaces |
| PSR-11 | Container Interface | A small interface for retrieving entries from a container |
| PSR-12 | Extended Coding Style Guide | A coding-style guide that extends PSR-1 and supersedes PSR-2 |
| PSR-13 | Hypermedia Links | Interfaces for representing links and link providers |
| PSR-14 | Event Dispatcher | Interfaces for dispatching events to listeners |
| PSR-15 | HTTP Handlers | Interfaces for server request handlers and middleware |
| PSR-16 | Simple Cache | A simpler cache contract for common get/set operations |
| PSR-17 | HTTP Factories | Factory interfaces for creating PSR-7 HTTP objects |
| PSR-18 | HTTP Client | An interface for sending HTTP requests and receiving responses |
| PSR-20 | Clock | An interface for obtaining the current time |

The list is a catalog, not a requirement to use every interface. A small script may not need a container, event dispatcher, HTTP client abstraction, or cache interface.

## Accepted, draft, deprecated, and abandoned

As of the status snapshot above:

- Accepted: PSR-1, PSR-3, PSR-4, PSR-6, PSR-7, PSR-11, PSR-12, PSR-13, PSR-14, PSR-15, PSR-16, PSR-17, PSR-18, and PSR-20.
- Draft: PSR-5 (PHPDoc Standard), PSR-19 (PHPDoc tags), PSR-21 (Internationalization), and PSR-22 (Application Tracing).
- Deprecated: PSR-0 (older autoloading standard) and PSR-2 (older coding-style guide).
- Abandoned: PSR-8 (Huggable Interface), PSR-9 (Security Advisories), and PSR-10 (Security Reporting Process).

A proposal number is not evidence that a specification was accepted. In particular, PSR-5, PSR-19, PSR-21, and PSR-22 are drafts in this snapshot. Treat a draft as work in progress.

## PSR-1 and code style

PSR-1 sets basic coding conventions intended to make shared PHP code easier to recognize. Its core topics include:

- PHP source uses standard PHP tags and UTF-8 without a byte-order mark.
- A file should primarily declare symbols or perform side effects, rather than mix both without a reason.
- Class names use StudlyCaps, method names use camelCase, and class constants use uppercase words separated by underscores.
- Namespaces and classes follow an autoloading standard.

PSR-1 is a baseline. It does not define every formatting choice a project will need.

### PSR-12 and the newer PER Coding Style

PSR-12 extends PSR-1 with detailed style rules for files, imports, declarations, classes, methods, control structures, and other PHP syntax. It superseded PSR-2, which is deprecated.

PHP-FIG also publishes PHP Evolving Recommendations (PERs). Its current Coding Style guide is PER Coding Style 3.1, which extends and replaces PSR-12 and includes guidance for newer PHP syntax. For a new project, choose one style guide, document it, and configure a formatter or linter for it. For an existing codebase, avoid a large unrelated formatting rewrite just to change the style label.

A style rule affects how code is formatted. It does not decide whether an application should use MVC, a particular framework, dependency injection, or a specific folder layout.

## PSR-4: autoloading

PSR-4 maps a namespace prefix to a directory. For example, a Composer mapping for `App\` to `src/` allows `App\Service\Mailer` to map to `src/Service/Mailer.php`.

PSR-4 helps packages and applications agree on how namespaced classes are found. In a Composer project, the mapping is configured in `composer.json`, and the generated loader is included from the application's entry point.

The previous chapter covered PSR-4 in detail. Keep directory and namespace capitalization consistent, particularly when code will run on a case-sensitive filesystem.

## Interface PSRs and implementation packages

Several PSRs define interfaces rather than complete working components. An application installs a package that implements the interface. Different implementations can then provide the same contract.

| Standard | Interface contract | Possible implementation role |
| --- | --- | --- |
| PSR-3 | Logger methods and levels | Write structured messages to files, a service, or another destination |
| PSR-6 / PSR-16 | Cache operations | Store and retrieve values using a memory, file, or network cache |
| PSR-7 | HTTP message objects | Represent requests and responses in a framework or middleware pipeline |
| PSR-11 | Container entry lookup | Retrieve configured services from a dependency container |
| PSR-14 | Event dispatch | Deliver event objects to registered listeners |
| PSR-15 | Request handlers and middleware | Process a server request and produce a response |
| PSR-17 | HTTP object factories | Create request, response, stream, and uploaded-file objects |
| PSR-18 | HTTP client | Send a request and return an HTTP response |
| PSR-20 | Clock | Obtain the current time through a replaceable service |

The PSR normally defines the shared contract, not the package choice, configuration, or behavior of every concrete implementation. Read the implementation's own documentation and compatibility requirements.

### Depend on the interface where appropriate

Suppose application code only needs to write a log message. It can accept `Psr\Log\LoggerInterface` instead of constructing a specific logger inside the business service:

~~~php
<?php

declare(strict_types=1);

namespace App\Order;

use Psr\Log\LoggerInterface;

final class OrderProcessor
{
    public function __construct(
        private LoggerInterface $logger
    ) {
    }

    public function process(string $orderId): void
    {
        $this->logger->info('Order processed', [
            'order_id' => $orderId,
        ]);
    }
}
~~~

A bootstrap file can provide a concrete implementation. The service can then focus on its work while the application configures where log records go.

This is useful when an interface is a real boundary in the design. Do not add an abstraction solely because a PSR exists; a direct dependency may be simpler when there is only one implementation and no need to substitute it.

## HTTP-related PSRs

HTTP standards are easier to understand as separate layers:

- PSR-7 describes request and response message interfaces, including associated URI and stream interfaces.
- PSR-15 describes server-side handlers and middleware that process a server request and return a response.
- PSR-17 describes factories for creating PSR-7 objects.
- PSR-18 describes a client interface for sending requests and receiving responses.
- PSR-13 describes hypermedia link interfaces.

These standards can interoperate, but one does not automatically provide all the others. Check which interfaces a framework or package implements and what dependencies it requires.

A PSR-7 message is designed as an immutable message: methods such as `withHeader()` return a new message value rather than modifying the original object. Application code should keep using the returned object.

## Cache and container standards

PSR-6 and PSR-16 both describe caching contracts, with different levels of detail and usage style. Choose the contract that matches the package boundary instead of implementing both automatically.

PSR-11 is a small container interface for retrieving entries. It does not define an application's service registration language, automatic wiring strategy, or complete dependency injection architecture. Those features belong to a concrete container implementation or application design.

## Events and clocks

PSR-14 standardizes event dispatch interfaces so code that emits or handles events can work across compatible dispatchers. The event's meaning and application-specific listener behavior remain the responsibility of the application.

PSR-20 provides a clock interface. A service that needs the current time can depend on a clock abstraction, which makes tests able to supply a controlled clock. Use it when time is a meaningful dependency, not as mandatory ceremony around every call to the system clock.

## How to adopt a PSR in a project

1. Identify an actual interoperability need or project convention.
2. Confirm the PSR's current status and read its scope.
3. Check the PHP version and package requirements of its interfaces or implementation.
4. Decide whether the application or a library needs the interface as a public boundary.
5. Choose a maintained implementation package if the PSR defines only an interface.
6. Configure the project to enforce the desired rules, such as a style checker or dependency policy.
7. Introduce the change incrementally and preserve existing behavior.
8. Document the project's chosen convention so contributors can follow it.

A package can implement a PSR without an application adopting every other PSR. Check the package's declared compatibility and behavior rather than assuming that two standards imply a complete integration.

## Common mistakes

### Treating an accepted PSR as mandatory for all PHP code

Acceptance means PHP-FIG accepted the specification, not that every project must adopt it. A project should choose standards that help its users and dependencies interoperate.

### Treating a draft as a settled contract

Draft specifications may change. Avoid promising a draft interface as a stable public API without a deliberate compatibility plan.

### Confusing PSR-2 and PSR-12

PSR-2 is deprecated. PSR-12 superseded it, and the newer PER Coding Style guide extends and replaces PSR-12. Follow the style standard the project actually documents.

### Assuming PSR interfaces come with implementations

An interface package defines a contract. Select and configure an implementation package separately.

### Adding every PSR to every project

Each abstraction has a cost. Use the relevant interface when it creates a useful integration boundary; keep small applications simple.

### Assuming PSR-11 defines all dependency injection behavior

PSR-11 describes container lookup, not all ways a container can register or construct services.

### Modifying an HTTP message but ignoring the return value

PSR-7 `with...` methods return a new message. Assign or return that new value instead of assuming the original changed.

### Reformatting an entire repository while fixing an unrelated feature

Style-only changes create noisy diffs. Separate formatting migrations from behavior changes and agree on the target guide first.

## Key takeaways

- PHP-FIG facilitates discussion about interoperability; it does not mandate every accepted PSR for all projects.
- PSR stands for PHP Standard Recommendation, and PSR status should be checked before adoption.
- Accepted interface PSRs define contracts; an implementation package supplies concrete behavior.
- PSR-1 is a basic coding standard; PSR-4 covers autoloading.
- PSR-12 superseded deprecated PSR-2; PER Coding Style 3.1 is the newer PHP-FIG style guide in this status snapshot.
- PSR-3 logging, PSR-7 HTTP messages, PSR-11 containers, PSR-14 events, PSR-15 handlers, PSR-18 clients, and PSR-20 clocks are useful only where those boundaries matter.
- Use the interface when it improves interoperability or substitution; do not add abstraction without a need.
- Document the standards your project adopts and enforce them consistently.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [109 - Version constraints, lock files, and packages](109-version-constraints-lockfiles-and-packages.md) | [Q&A](../qnas/110-php-fig-standards-and-psr-recommendations.md) | [Example](../examples/110-php-fig-standards-and-psr-recommendations.md) | [111 - Project structure, namespaces, and application layers](111-project-structure-namespaces-and-application-layers.md) |
