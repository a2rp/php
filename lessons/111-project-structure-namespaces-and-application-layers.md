# 111 - Project structure, namespaces, and application layers

## Goals

After this lesson, you should be able to:

- Choose a project layout that fits an application's size and responsibilities.
- Identify which files should be reachable from the web and which should remain private.
- Organize PHP classes with namespaces and Composer autoloading.
- Explain the roles of presentation, application, domain, and infrastructure code.
- Trace a request through those responsibilities without tying business rules to a framework or database.
- Keep dependencies moving toward stable application rules and introduce abstractions only where they help.
- Recognize common organization problems and improve a codebase gradually.

## Why project structure matters

A project structure is a way to help people find code and understand its responsibilities. PHP does not require every application to use one directory layout. A short command-line script, a small website, a framework application, and a reusable package can all need different arrangements.

A good structure makes related code easy to locate, keeps unrelated concerns apart, and makes important boundaries visible. It should grow in response to actual complexity. Copying a large framework's folder tree into a small application can add ceremony without making the code clearer.

The examples in this chapter describe one practical layout for a small application. Treat the names as conventions that a team can adapt, not as language rules.

## A practical project layout

~~~text
shop/
├── public/
│   └── index.php                 web server entry point
├── src/
│   ├── Application/
│   ├── Domain/
│   ├── Infrastructure/
│   └── Presentation/
├── templates/
├── config/
├── database/
├── tests/
├── var/                          generated local data and logs
├── vendor/                       Composer-managed dependencies
├── .env                          local secrets; normally ignored by Git
├── .env.example                  safe names and example values
├── .gitignore
├── composer.json
└── composer.lock
~~~

This is a starting point, not a checklist. A small application may not need `Domain/`, `Infrastructure/`, or `Presentation/` on day one. A command-line package may not have a `public/` directory or templates at all. Add a directory when it gives code a clear home.

### Keep the web document root narrow

For a web application, configure the web server's document root to `public/`. That directory should contain only files that are meant to be directly served, such as the front controller and public assets. Keep `src/`, `config/`, `tests/`, `.env`, and `vendor/` outside the document root.

The front controller is the public entry point. It loads Composer's generated autoloader and starts the application. The web server should send application requests to that entry point according to its configuration. Routing rules are covered in the next chapter.

~~~php
<?php

declare(strict_types=1);

require dirname(__DIR__) . '/vendor/autoload.php';

// Application startup and request handling are configured here.
~~~

Using `dirname(__DIR__)` makes the path relative to this file instead of depending on the process's current working directory. That distinction matters because a process can be started from a different directory than expected.

### Know what belongs in each directory

| Location | Typical responsibility |
| --- | --- |
| `public/` | Public entry point and files the web server may serve directly |
| `src/` | Application-owned PHP source code |
| `Application/` | Use cases that coordinate work for a user or system action |
| `Domain/` | Business concepts and rules that should not depend on web or storage details |
| `Infrastructure/` | Concrete database, filesystem, mail, logging, and external-service adapters |
| `Presentation/` | HTTP-facing or command-line input/output translation when the project needs it |
| `templates/` | HTML templates or other presentation files |
| `config/` | Non-secret application and service configuration |
| `database/` | Migrations, seed data, and database-related project files |
| `tests/` | Automated checks, often grouped to mirror important code boundaries |
| `var/` | Generated cache, logs, and runtime files; usually not committed |
| `vendor/` | Dependencies installed by Composer; normally not committed |

Directory names vary by framework and team. Consistency and clear ownership matter more than choosing these exact names.

## Namespaces organize PHP symbols

A namespace gives classes and other supported PHP symbols a qualified name. It prevents unrelated code from needing the same global class name. A PHP file declares its namespace near the beginning, after an optional `declare` statement and before other code.

~~~php
<?php

declare(strict_types=1);

namespace App\Domain\Order;

final class Order
{
    public function __construct(
        private readonly string $number,
    ) {
    }

    public function number(): string
    {
        return $this->number;
    }
}
~~~

The class's fully qualified name is `App\Domain\Order\Order`. Within another namespace, import a class with `use` when that makes code easier to read:

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

use App\Domain\Order\Order;

final class OrderSummary
{
    public function numberFor(Order $order): string
    {
        return $order->number();
    }
}
~~~

A namespace is a name organization mechanism. It does not automatically create a directory, load a file, or provide security. Autoloading is the separate mechanism that finds the PHP file for a class.

## Map namespaces to files with Composer

Composer can generate a PSR-4 autoloader. A namespace prefix maps to a base directory; the remaining namespace segments map to subdirectories, and the class name maps to a PHP filename. Match directory and class-name casing consistently, especially when code may run on a case-sensitive filesystem.

For example, this mapping:

~~~json
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
~~~

allows a class named `App\Domain\Order\Order` to be stored at `src/Domain/Order/Order.php`. Once the project's Composer autoloader is available, application code can refer to that class without manually requiring its file each time.

After adding or changing an autoload mapping in `composer.json`, regenerate Composer's autoload files from the project root. When the mapping is already configured, Composer's generated loader can discover new PSR-4 classes without adding one `require` statement for each class.

A mapping is a convention shared between namespace names and file paths. If the namespace, directory casing, filename, or mapping disagrees, autoloading may fail. Check each part rather than adding increasingly broad fallback paths.

## Separate responsibilities into layers

Layers are useful when they represent different reasons for code to change. They are not a requirement to create one class per folder, and they do not require a specific design pattern.

### Presentation layer

Presentation code translates input and output at the application's boundary. In a web application, that may mean reading an HTTP request, validating its shape, calling an application action, and turning the result into an HTTP response or rendered page. A command-line presentation layer might parse command-line arguments and print a result instead.

Presentation code should not be the only place where an important business rule exists. A validation rule that also applies to imports, scheduled jobs, or an API should live in a reusable application or domain responsibility rather than only inside a page handler.

### Application layer

The application layer coordinates a use case: a meaningful action the system performs, such as registering a customer or placing an order. It obtains the required input, calls the relevant domain behavior, requests persistence through an appropriate boundary, and returns a result for presentation code.

An application service should express the action clearly. Avoid letting it become a large miscellaneous class that contains every use case in the system.

### Domain layer

The domain layer represents the business concepts and rules that matter to the application. Depending on the problem, this may include entities, value objects, domain services, and business-specific exceptions. A simple application may not need all of these forms.

Keep domain code independent of HTTP request objects, HTML templates, SQL connection details, and framework controllers. Those details can change for reasons separate from the business rule.

### Infrastructure layer

Infrastructure code connects the application to technical details: a PDO database, a filesystem, a mail provider, a logger, or an external API. An adapter translates between the application's needs and the chosen tool's API.

For example, an application can depend on an `OrderRepository` contract while an infrastructure class implements it with PDO. The contract is useful if it keeps the use case independent of persistence details; it is unnecessary if it only renames a concrete call without creating a meaningful boundary.

## Follow a request through the application

A typical request might move through responsibilities like this:

~~~text
HTTP request
    -> public entry point and bootstrap
    -> request/response boundary
    -> application use case
    -> domain rule
    -> repository contract
    -> database adapter
    -> application result
    -> HTTP response or rendered page
~~~

Each arrow is a call or returned value, not necessarily a new class. The next chapter covers routing, controllers, and middleware in more detail. This chapter focuses on assigning responsibilities so those web details do not swallow the rest of the application.

## Keep dependencies pointed in a useful direction

Stable application and domain rules should not need to know how an HTTP framework, database driver, or mail provider works. Concrete infrastructure can depend on application contracts and implement them. At the composition boundary, such as the bootstrap, the application chooses concrete implementations and passes them to the use cases that need them.

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

interface OrderRepository
{
    public function save(OrderData $order): void;
}
~~~

~~~php
<?php

declare(strict_types=1);

namespace App\Infrastructure\Persistence;

use App\Application\Order\OrderData;
use App\Application\Order\OrderRepository;
use PDO;

final class PdoOrderRepository implements OrderRepository
{
    public function __construct(private readonly PDO $pdo)
    {
    }

    public function save(OrderData $order): void
    {
        // Translate the application's data into a database operation.
    }
}
~~~

These snippets show a possible seam, not a rule that every model must have an interface. For a tiny application with one stable persistence choice, a direct implementation may be easier to understand. Introduce a contract when it protects a useful boundary, supports a real alternative, or improves focused testing.

The bootstrap or composition root is a natural place to construct concrete services and connect them. Avoid creating a database connection, mail client, or framework container deep inside a domain rule; that hides dependencies and makes the rule harder to reuse.

## Configuration, secrets, and generated files

Keep environment-specific values such as database credentials and API tokens out of source code and version control. A checked-in example configuration can show the required variable names with harmless placeholder values. A local environment file containing real secrets should be excluded from Git.

Keep generated dependencies and runtime output out of source control unless the project's deployment model explicitly requires them. Composer projects normally commit `composer.json` and `composer.lock`, while excluding `vendor/`. Runtime directories such as `var/cache/` and `var/log/` should be writable where needed and ignored when their contents are generated.

Configuration should make the selected services and their settings understandable. It should not become a second application full of hidden business decisions.

## Organize by layer or by feature?

A layer-first layout groups code by technical role:

~~~text
src/
├── Application/
├── Domain/
└── Infrastructure/
~~~

A feature-first layout groups a feature's related code together:

~~~text
src/
└── Order/
    ├── Application/
    ├── Domain/
    └── Infrastructure/
~~~

Either can work. Layer-first can be easy to understand in a small codebase with a few clear responsibilities. Feature-first can make a large application easier to navigate when each feature has substantial code. Some teams use a modest hybrid. Choose based on where developers look for related changes, and avoid deeply nested directories that make files harder to find.

## Common organization mistakes

### Putting the whole application in one entry-point file

A single file may be fine for a short exercise. As the application grows, move reusable responsibilities into named classes or functions and keep the entry point focused on startup.

### Making every class a `Manager`, `Helper`, or `Utils`

Generic names hide what a class owns. Prefer a name that describes the behavior or role, such as `RegisterCustomer`, `PdoOrderRepository`, or `PriceCalculator`.

### Treating folders as architecture

A folder named `Domain` does not make code independent. The actual dependencies and responsibilities determine whether a boundary exists.

### Creating an interface for every class

An interface with one implementation and no meaningful boundary can add indirection. Use contracts where they support substitution, a stable dependency direction, or a real integration boundary.

### Putting secrets or private source under the document root

A server misconfiguration could expose files that were meant to stay private. Point the web document root at `public/` and keep sensitive and internal files outside it.

### Renaming a whole project to imitate a template

Move code in small steps. Preserve behavior, update autoload mappings and tests with each move, and keep each change reviewable.

## A practical way to improve an existing project

1. Identify the real public entry points and configure the web document root narrowly.
2. Find the code that contains business rules and the code that talks to external systems.
3. Name those responsibilities before moving files.
4. Add or correct namespaces and Composer autoload mappings consistently.
5. Move one cohesive group at a time and update imports and tests.
6. Keep the bootstrap responsible for selecting and connecting concrete services.
7. Remove abstractions or folders that do not make the system easier to understand.

## Key takeaways

- PHP does not require a universal project directory structure; choose a layout that makes responsibilities easy to find.
- In a web application, use a narrow `public/` document root and keep private source, configuration, dependencies, and secrets outside it.
- Namespaces organize symbol names; Composer's PSR-4 autoloader maps namespace prefixes to file paths.
- Match namespace, directory, and filename casing consistently.
- Presentation translates boundary input and output, application code coordinates use cases, domain code owns business rules, and infrastructure adapts external tools.
- Layers represent responsibilities; they do not require a class or interface for every operation.
- Keep concrete service construction near the application bootstrap and pass dependencies to the code that needs them.
- Organize by layer, by feature, or with a simple hybrid based on project size and cohesion.
- Introduce structure gradually and keep each boundary useful and understandable.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [110 - PHP-FIG standards and PSR recommendations](110-php-fig-standards-and-psr-recommendations.md) | [Q&A](../qnas/111-project-structure-namespaces-and-application-layers.md) | [Example](../examples/111-project-structure-namespaces-and-application-layers.md) | [112 - Routing, controllers, and middleware](112-routing-controllers-and-middleware.md) |
