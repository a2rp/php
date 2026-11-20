# 022 - Constants, Predefined Constants, and Magic Constants

A constant gives a name to a value that should not be reassigned during a script's execution. PHP also provides predefined constants for runtime information and magic constants whose values come from the source-code location or context.

## What you will learn

- How to declare constants with **const** and **define()**
- How constants differ from variables and class constants
- How namespaces affect named constants
- How to check for and read a constant by name
- Which useful constants PHP provides
- How magic constants report file, function, class, trait, namespace, and property context
- Why extension-provided constants may not always exist
- How to avoid hard-coded paths and fragile assumptions

## What is a constant?

A constant associates a name with a value that cannot be changed or undefined after it is created. Unlike a variable, a constant has no dollar sign.

~~~php
<?php
const APP_NAME = "Study Planner";

echo APP_NAME . PHP_EOL;
~~~

Output:

~~~text
Study Planner
~~~

By convention, constant names are often uppercase with underscores. PHP constant names are case-sensitive, so **APP_NAME** and **App_Name** are different names. The uppercase style is a convention that makes constants easier to recognize.

Constants and variables are separate names:

~~~php
<?php
const STATUS = "ready";
$status = "changed";

echo STATUS . PHP_EOL;
echo $status . PHP_EOL;
~~~

Output:

~~~text
ready
changed
~~~

Changing **$status** does not change **STATUS**.

## Defining constants with const

Use **const** for a constant whose name and value are known in source code.

~~~php
<?php
const APP_NAME = "Study Planner";
const DEFAULT_PAGE_SIZE = 20;
const SUPPORTED_LANGUAGES = ["en", "hi"];

echo APP_NAME . PHP_EOL;
echo DEFAULT_PAGE_SIZE . PHP_EOL;
echo SUPPORTED_LANGUAGES[1] . PHP_EOL;
~~~

Output:

~~~text
Study Planner
20
hi
~~~

A **const** declaration is processed as part of compiling the file. It must be declared in a permitted declaration scope, such as the top level or a namespace. It cannot be placed inside a function or a runtime control block such as **if**, **for**, or **try**.

~~~php
<?php
if (PHP_SAPI === "cli") {
    define("APP_MODE", "command-line");
}
~~~

Use **define()** for a constant that must be created conditionally at runtime. The function takes the constant's name as a string and returns **true** when it succeeds.

### Keep constant values intentional

Constants are useful for stable configuration values, fixed labels, and values shared by multiple parts of an application. They are not a substitute for configuration that must change per request, user, or environment.

A constant is not a security mechanism. Do not put secrets in source-controlled constants; use an environment or secret-management mechanism appropriate to the application.

## Defining constants with define()

Use **define()** when the name or decision to create a constant must be made at runtime.

~~~php
<?php
define("BUILD_CHANNEL", "stable");

echo BUILD_CHANNEL . PHP_EOL;
~~~

Output:

~~~text
stable
~~~

Unlike a **const** declaration, **define()** can be called inside a function or conditional block. The name is passed as a string, so it can also be assembled dynamically.

~~~php
<?php
$settingName = "FEATURE_REPORTS";
$enabled = true;

define($settingName, $enabled);

if (constant($settingName)) {
    echo "Reports are enabled." . PHP_EOL;
}
~~~

Output:

~~~text
Reports are enabled.
~~~

Do not use **define()** merely because it is available. Prefer **const** for fixed declarations that are clear in the source. Use **define()** when runtime creation is actually needed.

### Constants cannot be reassigned

Once a named constant exists, PHP does not allow code to assign a new value to it or unset it. Choose clear names and values when defining constants. Do not define the same name repeatedly from different files.

## Constant scope and namespaces

A named constant is not limited by function variable scope. It can be read inside a function without a **global** declaration.

~~~php
<?php
const APP_NAME = "Study Planner";

function printAppName(): void
{
    echo APP_NAME . PHP_EOL;
}

printAppName();
~~~

Namespaces can qualify constants just as they qualify functions and classes.

~~~php
<?php
namespace Planner;

const APP_NAME = "Study Planner";

echo APP_NAME . PHP_EOL;
echo __NAMESPACE__ . PHP_EOL;
~~~

Inside the **Planner** namespace, the constant is named **Planner\APP_NAME**. A namespace **const** declaration is available within that namespace and its fully qualified name can be used from another namespace.

When **define()** is called from a namespace, its string name is not automatically prefixed with that namespace. Include the namespace in the string when you want to define a namespaced constant:

~~~php
<?php
namespace Planner;

define(__NAMESPACE__ . "\BUILD_CHANNEL", "local");

echo constant(__NAMESPACE__ . "\BUILD_CHANNEL") . PHP_EOL;
~~~

This defines **Planner\BUILD_CHANNEL**. A call such as **define("BUILD_CHANNEL", "local")** would define the global constant **BUILD_CHANNEL**, even when called from a namespaced file.

## Checking and reading constants by name

Use **defined()** to find out whether a constant name is already defined. This is particularly useful for optional extension constants or constants that depend on configuration.

~~~php
<?php
if (defined("PHP_VERSION")) {
    echo "PHP_VERSION is available." . PHP_EOL;
}
~~~

Use **constant()** when the name is stored in a variable or assembled dynamically.

~~~php
<?php
$constantName = "PHP_VERSION";

if (defined($constantName)) {
    echo constant($constantName) . PHP_EOL;
}
~~~

Refer to a known constant directly when its name is fixed. Dynamic lookup is useful when the name itself comes from data, but do not use untrusted input as a constant name without a reason and validation.

## Useful predefined constants

PHP defines core constants that describe the current runtime. Common examples include:

| Constant | What it describes |
| --- | --- |
| **PHP_VERSION** | The PHP version as a string |
| **PHP_VERSION_ID** | The PHP version as an integer suitable for comparisons |
| **PHP_MAJOR_VERSION** | The PHP major version number |
| **PHP_MINOR_VERSION** | The PHP minor version number |
| **PHP_OS_FAMILY** | The operating-system family |
| **PHP_SAPI** | The current server API, such as **cli** |
| **PHP_INT_SIZE** | The number of bytes in an integer |
| **PHP_INT_MAX** | The largest supported integer |
| **PHP_EOL** | The platform's newline sequence |
| **DIRECTORY_SEPARATOR** | The platform's directory separator |

For example, **PHP_VERSION_ID** makes a version check easy to read:

~~~php
<?php
if (PHP_VERSION_ID >= 80200) {
    echo "PHP 8.2 or later is running." . PHP_EOL;
}
~~~

The integer format is **major × 10000 + minor × 100 + patch**. For example, **80200** represents PHP 8.2.0. A version check should express the minimum feature version your code actually needs.

PHP also has constants supplied by extensions. Those constants are available only when the corresponding extension is present and enabled. If code can run without an extension, check an extension-specific constant with **defined()** before reading it.

Do not assume that every constant found on one machine is available on every server. The loaded extensions and the PHP build can differ.

## Magic constants

Magic constants look like constants but are special language tokens. PHP resolves their values from where they appear in the source code. They are not ordinary constants created by **const** or **define()**.

| Magic constant | Value |
| --- | --- |
| **__LINE__** | The current source-code line number |
| **__FILE__** | The full path and filename of the current file |
| **__DIR__** | The directory containing the current file |
| **__FUNCTION__** | The current function name |
| **__CLASS__** | The current class name |
| **__TRAIT__** | The current trait name |
| **__METHOD__** | The current class and method name, or function name outside a method |
| **__NAMESPACE__** | The current namespace name |
| **__PROPERTY__** | The property name in a property hook; PHP 8.4 and later |

Magic constants are written with double underscores. Their values depend on the location or context where they appear.

### File and directory context

**__FILE__** gives the full path and filename of the current file. **__DIR__** gives its containing directory and normally has no trailing slash.

~~~php
<?php
echo basename(__FILE__) . PHP_EOL;
echo basename(__DIR__) . PHP_EOL;
~~~

When the script is saved as **paths.php** in a folder named **config**, this prints:

~~~text
paths.php
config
~~~

Using **__DIR__** to build a path relative to the current PHP file is usually more reliable than assuming the process was started from a particular working directory.

~~~php
<?php
$configPath = __DIR__ . DIRECTORY_SEPARATOR . "settings.php";
~~~

The file is not automatically included by constructing this string; the code must still load it with an appropriate **include** or **require** statement.

### Function, class, and trait context

**__FUNCTION__**, **__CLASS__**, **__TRAIT__**, and **__METHOD__** identify the code context. A class name inside a trait can refer to the class that uses the trait, while **__TRAIT__** still identifies the trait itself.

~~~php
<?php
function showFunctionName(): void
{
    echo __FUNCTION__ . PHP_EOL;
}

class Report
{
    public function showMethodName(): void
    {
        echo __CLASS__ . PHP_EOL;
        echo __METHOD__ . PHP_EOL;
        echo __FUNCTION__ . PHP_EOL;
    }
}

showFunctionName();
(new Report())->showMethodName();
~~~

Output:

~~~text
showFunctionName
Report
Report::showMethodName
showMethodName
~~~

Outside its corresponding context, a context-specific magic constant such as **__CLASS__** is generally an empty string.

### Namespace context and ::class

**__NAMESPACE__** contains the current namespace name. In the global namespace it is an empty string.

The **::class** syntax is related but distinct: it resolves a class name to a string without creating an object.

~~~php
<?php
namespace Planner;

class Entry
{
}

echo Entry::class . PHP_EOL;
echo __NAMESPACE__ . PHP_EOL;
~~~

Output:

~~~text
Planner\Entry
Planner
~~~

In production output, the backslash in the fully qualified class name is one character. **Entry::class** is useful when passing a class name to an API or referring to a class without hard-coding its full namespace.

## Magic constants are not ordinary constants

Because magic constants are resolved from their source location, **defined("__FILE__")** does not check whether the magic token exists. **defined()** and **constant()** are for named constants.

Defining an ordinary constant with a similar name does not change the value of a magic constant. Avoid trying to replace or override magic constants.

## Common mistakes

- Adding a dollar sign to a constant name
- Assuming uppercase spelling is what makes a constant immutable
- Using a **const** declaration inside a function or runtime conditional
- Forgetting that **define()** takes the name as a string
- Assuming **define()** automatically adds the current namespace
- Treating a magic constant as an ordinary constant for **defined()**
- Assuming extension constants exist even when an extension is disabled
- Using the current working directory when a path should be based on the PHP file's location
- Treating **::class** as object construction
- Putting credentials or other secrets into source-controlled constants

## Try it yourself

1. Declare a top-level **const** for an application name and print it from a function.
2. Use **define()** to create a constant only when a feature is enabled.
3. Store a constant name in a variable and read its value with **constant()** after checking **defined()**.
4. Print **PHP_VERSION**, **PHP_VERSION_ID**, and **PHP_SAPI**.
5. Print the current script's filename using **basename(__FILE__)**.
6. Build a path relative to the current file using **__DIR__** and **DIRECTORY_SEPARATOR**.
7. Add a namespace and compare **__NAMESPACE__** with **ClassName::class**.
8. Use a trait and inspect **__CLASS__**, **__TRAIT__**, and **__METHOD__** inside one of its methods.

## Key takeaways

- Use **const** for fixed source-level declarations and **define()** when runtime creation is needed.
- Constants have no dollar sign and cannot be reassigned or undefined.
- Namespaced constants have qualified names; **define()** does not automatically apply the current namespace.
- Use **defined()** and **constant()** for optional or dynamically named constants.
- Core predefined constants describe PHP and its runtime; extension constants depend on enabled extensions.
- Magic constants are resolved from source location and context, not created like ordinary constants.
- **__DIR__** is useful for paths relative to the current file.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [021 - Mixed, void, never, and literal types](021-mixed-void-never-and-literal-types.md) | [Open Q&A](../qnas/022-constants-predefined-constants-and-magic-constants.md) | [Open example](../examples/022-constants-predefined-constants-and-magic-constants.md) | [023 - Expressions and value evaluation](023-expressions-and-value-evaluation.md) |