# 022 - Constants, Predefined Constants, and Magic Constants: Complete Example

This PHP CLI project demonstrates source-level constants, a runtime-defined namespaced constant, a runtime condition, PHP's core predefined constants, and magic constants that identify source and code context.

## Question

How can a PHP application keep stable values in named constants, create a constant at runtime when needed, inspect PHP runtime information, and build file paths based on the current script?

## Goal

Create this project:

~~~text
php-constants-lab/
|-- README.md
|-- constants-demo.php
~~~

The **constants-demo.php** script demonstrates:

- Declaring fixed constants with **const**
- Creating namespaced constants with **define()**
- Looking up a dynamic constant name with **defined()** and **constant()**
- Reading core constants such as **PHP_VERSION_ID** and **PHP_SAPI**
- Using **__FILE__**, **__DIR__**, **__NAMESPACE__**, and **__FUNCTION__**
- Inspecting class and trait context through magic constants

## Complete code: constants-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

namespace Demo;

const APP_NAME = "Northstar Notes";
const DEFAULT_TAGS = ["php", "learning"];

define(__NAMESPACE__ . '\BUILD_CHANNEL', "local");

if (PHP_SAPI === "cli") {
    define(__NAMESPACE__ . '\RUN_MODE', "command-line");
}

function showOverview(): void
{
    echo "App: " . APP_NAME . PHP_EOL;
    echo "Tags: " . implode(", ", DEFAULT_TAGS) . PHP_EOL;

    $buildChannelName = __NAMESPACE__ . '\BUILD_CHANNEL';
    if (defined($buildChannelName)) {
        echo "Build channel: " . constant($buildChannelName) . PHP_EOL;
    }

    $runModeName = __NAMESPACE__ . '\RUN_MODE';
    if (defined($runModeName)) {
        echo "Run mode: " . constant($runModeName) . PHP_EOL;
    }

    echo "PHP_VERSION: "
        . (defined("PHP_VERSION") ? "available" : "missing")
        . PHP_EOL;
    echo "SAPI: " . PHP_SAPI . PHP_EOL;
    echo "PHP 8 or later: "
        . (PHP_VERSION_ID >= 80000 ? "yes" : "no")
        . PHP_EOL;
    echo "Namespace: " . __NAMESPACE__ . PHP_EOL;
    echo "Function: " . __FUNCTION__ . PHP_EOL;
    echo "File: " . basename(__FILE__) . PHP_EOL;
    echo "Folder: " . basename(__DIR__) . PHP_EOL;
}

trait ShowsContext
{
    public function showTraitContext(): void
    {
        echo "Class from trait: " . __CLASS__ . PHP_EOL;
        echo "Trait: " . __TRAIT__ . PHP_EOL;
        echo "Method: " . __METHOD__ . PHP_EOL;
        echo "Function: " . __FUNCTION__ . PHP_EOL;
    }
}

final class Notebook
{
    use ShowsContext;

    public static function showClassContext(): void
    {
        echo "Class: " . __CLASS__ . PHP_EOL;
        echo "Method: " . __METHOD__ . PHP_EOL;
    }
}

showOverview();
Notebook::showClassContext();
(new Notebook())->showTraitContext();
~~~

## Complete code: README.md

~~~~markdown
# PHP Constants Lab

A small command-line project for learning named constants, predefined constants, and magic constants.

## Requirements

- PHP 7.4 or later
- A terminal

## Run

~~~sh
php constants-demo.php
~~~

## Expected output

~~~text
App: Northstar Notes
Tags: php, learning
Build channel: local
Run mode: command-line
PHP_VERSION: available
SAPI: cli
PHP 8 or later: yes
Namespace: Demo
Function: DemoshowOverview
File: constants-demo.php
Folder: php-constants-lab
Class: DemoNotebook
Method: DemoNotebook::showClassContext
Class from trait: DemoNotebook
Trait: DemoShowsContext
Method: DemoShowsContext::showTraitContext
Function: showTraitContext
~~~

The **File** and **Folder** lines use the names from this project's layout. If you rename the file or containing folder, those two output lines change accordingly.
~~~~

## Expected output

~~~text
App: Northstar Notes
Tags: php, learning
Build channel: local
Run mode: command-line
PHP_VERSION: available
SAPI: cli
PHP 8 or later: yes
Namespace: Demo
Function: DemoshowOverview
File: constants-demo.php
Folder: php-constants-lab
Class: DemoNotebook
Method: DemoNotebook::showClassContext
Class from trait: DemoNotebook
Trait: DemoShowsContext
Method: DemoShowsContext::showTraitContext
Function: showTraitContext
~~~

The **File** and **Folder** lines use the names from this project's layout. If you rename the file or containing folder, those two output lines change accordingly.

## Understand the code

### Fixed constants

**APP_NAME** and **DEFAULT_TAGS** are declared with **const**. They are known in source code and do not need runtime conditional creation. Their names are in the **Demo** namespace, so this file can refer to them by their unqualified names.

### Runtime-defined constants

The **BUILD_CHANNEL** constant is created by **define()** at runtime. The **RUN_MODE** constant is created only when PHP is running through the CLI SAPI. Both names include the **Demo** namespace explicitly.

The script builds each full name as a string. **defined()** checks whether the name exists, then **constant()** reads its value. This avoids looking up a missing conditional constant.

### Predefined constants

**PHP_VERSION**, **PHP_SAPI**, and **PHP_VERSION_ID** are provided by PHP. The script uses the version identifier in a numeric comparison and prints the SAPI used to run the script.

Core predefined constants are available as part of PHP. Constants supplied by optional extensions may not be available everywhere, so check those names with **defined()** before using them.

### Magic constants

**__NAMESPACE__** and **__FUNCTION__** identify the namespace and function context for **showOverview()**. **__FILE__** and **__DIR__** describe the script's path. The example prints only their final path components so its expected output is stable across machines.

The class method uses **__CLASS__** and **__METHOD__**. The method inside **ShowsContext** shows that **__TRAIT__** names the trait while **__CLASS__** identifies **Notebook**, the class using the trait.

## Make a change

1. Change **APP_NAME** and print the updated value.
2. Add another item to **DEFAULT_TAGS** and print the array.
3. Change the runtime **BUILD_CHANNEL** value.
4. Run the script through a web server and observe how **PHP_SAPI** and the conditional **RUN_MODE** change.
5. Print **PHP_VERSION** to see the exact version string on your machine.
6. Rename the script and observe how **basename(__FILE__)** changes.
7. Move the script to another folder and observe how **__DIR__** changes.
8. Add another class that uses **ShowsContext** and compare the output from the trait method.

## Common mistakes

- Adding a dollar sign when reading a constant
- Declaring a **const** inside a runtime conditional
- Assuming **define()** automatically adds the current namespace
- Calling **constant()** before checking an optional name with **defined()**
- Assuming extension-provided constants are installed on every machine
- Using the terminal working directory when a path should be based on the PHP file
- Assuming **__CLASS__** inside a trait always names the trait's consuming class in the same way as **__TRAIT__**
- Treating **ClassName::class** as if it creates an object
- Putting credentials in source-controlled constants

## Key takeaways

- Use **const** for fixed source declarations and **define()** for runtime creation.
- Namespaced constants need qualified names when referenced from outside their namespace.
- PHP core constants expose runtime information; optional extension constants need availability checks.
- Magic constants are resolved from source location and the surrounding code context.
- Build file-relative paths from **__DIR__**, not from assumptions about the current working directory.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [021 - Mixed, void, never, and literal types](../lessons/021-mixed-void-never-and-literal-types.md) | [Read this lesson](../lessons/022-constants-predefined-constants-and-magic-constants.md) | [Open Q&A](../qnas/022-constants-predefined-constants-and-magic-constants.md) | [023 - Expressions and value evaluation](../lessons/023-expressions-and-value-evaluation.md) |