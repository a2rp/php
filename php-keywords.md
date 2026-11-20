# PHP Keywords and Reserved Words

This guide covers PHP language keywords, language constructs, type names that are reserved, soft-reserved identifiers, and magic constants. These groups are related, but they are not interchangeable. The main keyword list follows the PHP 8.6 language reference. Check the PHP version required by a project before using syntax introduced in a newer release.

## Contents

- [How to read this guide](#how-to-read-this-guide)
- [Complete keyword list](#complete-keyword-list)
- [Declarations and object-oriented code](#declarations-and-object-oriented-code)
- [Control flow](#control-flow)
- [Functions, inclusion, and language constructs](#functions-inclusion-and-language-constructs)
- [Operators written as words](#operators-written-as-words)
- [Reserved type names and literals](#reserved-type-names-and-literals)
- [Soft-reserved and newly discouraged names](#soft-reserved-and-newly-discouraged-names)
- [Magic constants](#magic-constants)
- [Names that are not keywords](#names-that-are-not-keywords)
- [Naming and version notes](#naming-and-version-notes)
- [Continue learning](#continue-learning)

## How to read this guide

A keyword has special meaning in PHP syntax. A language construct may look like a function but is handled by the language grammar. A reserved type name may be disallowed as a class name even though it is not in the main keyword inventory. A soft-reserved name is accepted in some positions today, but using it as an identifier can conflict with current or future syntax.

Do not use any of these names for your own variables, functions, classes, constants, namespaces, or members unless you know the exact rule for that PHP version and context. Prefer descriptive names that are not reserved.

PHP keywords are generally case-insensitive, but the conventional spelling is lowercase. Variable names are case-sensitive. Use consistent casing for all user-defined names.

## Complete keyword list

The following is the complete language-keyword inventory in alphabetical order, including language constructs such as `echo`, `isset`, and `require`.

~~~text
__halt_compiler  abstract  and  array  as
break  callable  case  catch  class
clone  const  continue  declare  default
die  do  echo  else  elseif
empty  enddeclare  endfor  endforeach  endif
endswitch  endwhile  eval  exit  extends
final  finally  fn  for  foreach
function  global  goto  if  implements
include  include_once  instanceof  insteadof  interface
isset  list  match  namespace  new
or  print  private  protected  public
readonly  require  require_once  return  static
switch  throw  trait  try  unset
use  var  while  xor  yield
yield from
~~~

The spellings with parentheses are still language constructs, not ordinary function calls. For example, `isset($value)`, `empty($value)`, `exit()`, and `die()` look function-like, but their syntax and behavior are defined by PHP.

## Declarations and object-oriented code

| Keyword | Purpose |
| --- | --- |
| `abstract` | Marks a class or method as incomplete and intended for extension. |
| `class` | Declares a class. |
| `const` | Declares a constant at file or class scope. |
| `enum` | Soft-reserved word used to declare an enumeration. See the soft-reserved section. |
| `extends` | Declares a class or interface inheritance relationship. |
| `final` | Prevents a class from being extended or a method from being overridden. |
| `function` | Declares a named function or method. |
| `implements` | States that a class or enum fulfills an interface contract. |
| `interface` | Declares a contract that classes can implement. |
| `namespace` | Declares or refers to a namespace. |
| `new` | Creates an object or anonymous class instance. |
| `private` | Restricts member access to its declaring class. |
| `protected` | Restricts member access to its class and compatible descendants. |
| `public` | Makes a member accessible from any scope. |
| `readonly` | Declares readonly properties or a readonly class, subject to version and context rules. |
| `static` | Declares static members and supports late static binding. |
| `trait` | Declares reusable methods and properties for classes. |
| `use` | Imports names, uses a trait, or imports a closure variable. |
| `as` | Provides an alias in imports or an alternate name when iterating. |
| `insteadof` | Resolves a method name conflict between traits. |
| `var` | Legacy visibility syntax for public properties. Prefer explicit visibility. |
| `clone` | Clones an object, optionally invoking its `__clone()` method. |

A small declaration example:

~~~php
<?php
interface Logger
{
    public function write(string $message): void;
}

final class FileLogger implements Logger
{
    public function __construct(
        private readonly string $path,
    ) {}

    public function write(string $message): void
    {
        // Write the message using a suitable filesystem API.
    }
}
~~~

The keyword `use` has several meanings. At file scope it imports a class, function, or constant name. Inside a class it adds a trait. Inside a closure it captures variables from the outer scope.

## Control flow

| Keyword | Purpose |
| --- | --- |
| `if`, `elseif`, `else` | Selects a branch based on a condition. |
| `switch`, `case`, `default` | Selects a matching branch. Use `break` when fall-through is not intended. |
| `match` | Produces a value from strict comparisons against arms. |
| `for` | Repeats with initialization, condition, and update expressions. |
| `foreach` | Iterates over arrays and iterable values. |
| `while` | Repeats while a condition is true. |
| `do` | Starts a loop that checks its condition after the body runs. |
| `break` | Exits a loop or switch. |
| `continue` | Skips to the next loop iteration. |
| `goto` | Jumps to a label in the same file. Avoid it in ordinary application flow. |
| `return` | Returns a value from a function or ends the current function. |
| `throw` | Raises a `Throwable`. |
| `try`, `catch`, `finally` | Runs code with exception handling and cleanup. |
| `declare` | Sets a directive for a file or code block. |
| `enddeclare`, `endfor`, `endforeach`, `endif`, `endswitch`, `endwhile` | Close alternative colon-based control structures. Braces are usually easier to read. |

Example:

~~~php
$status = match ($code) {
    200 => 'ok',
    404 => 'not found',
    default => 'other',
};
~~~

## Functions, inclusion, and language constructs

| Keyword or construct | Purpose |
| --- | --- |
| `fn` | Starts an arrow function. |
| `global` | Imports a global variable into a function scope. Prefer passing dependencies as parameters. |
| `yield` | Produces a value from a generator. |
| `yield from` | Delegates generator output to another iterable or generator. |
| `include`, `include_once` | Loads a file and reports a warning if it cannot be loaded. |
| `require`, `require_once` | Loads a required file and stops execution on failure. |
| `isset` | Checks whether a variable or property is set and is not null. |
| `empty` | Checks whether a value is absent or evaluates as empty. |
| `unset` | Removes a variable or array entry. |
| `list` | Assigns values from an array to variables using destructuring syntax. |
| `array` | Legacy array-construction syntax, such as `array(1, 2)`. Short array syntax `[1, 2]` is preferred. |
| `callable` | A type declaration for values that can be called. It is also reserved as a type name. |
| `echo`, `print` | Write output. `echo` accepts multiple arguments; `print` returns the integer 1. |
| `exit`, `die` | End the current script, optionally with a status or message. |
| `eval` | Evaluates a string as PHP code. Avoid it with dynamic or untrusted input. |
| `__halt_compiler` | Stops parsing the current file and can expose following bytes as data. Rarely needed in application code. |

`isset`, `empty`, `unset`, `include`, `require`, `echo`, and `print` are not ordinary user-defined functions. They have special language behavior.

## Operators written as words

| Operator | Meaning |
| --- | --- |
| `and` | Logical AND with lower precedence than `&&`. |
| `or` | Logical OR with lower precedence than `||`. |
| `xor` | Logical exclusive OR. |

The different precedence of `and` and `or` can change assignment results. Prefer `&&` and `||` for conditions unless the lower precedence is intentional.

~~~php
$result = true and false;
// $result is true because assignment binds more tightly than "and".

$result = true && false;
// $result is false.
~~~

## Reserved type names and literals

Some words have special meaning in type declarations or class-name positions. They are not all part of the main keyword list.

| Name | Meaning |
| --- | --- |
| `int`, `float`, `bool`, `string` | Scalar types. |
| `array`, `callable` | Built-in type names. Since PHP 8.5, they are reserved against use as class, interface, trait, or enum names. |
| `iterable`, `object`, `mixed` | Built-in types for iterable values, objects, or values of any type. |
| `void`, `never` | Return types describing no returned value or a function that never completes normally. |
| `true`, `false`, `null` | Literal values. In supported type positions, they can also express narrow types. |
| `self`, `parent` | Special class-relative names used in type declarations and member references. |
| `static` | A keyword that can be used as a late-static return type in supported positions. |
| `resource`, `numeric` | Soft-reserved names. Avoid using them for class-like declarations. |

A type declaration may combine types:

~~~php
function findUser(int|string $id): User|null
{
    // Look up a user and return null when no record exists.
}
~~~

A nullable type can be written as `?User` or `User|null`. Union types use `|`, intersection types use `&`, and supported PHP versions can combine them in disjunctive normal form. A type name being reserved does not mean it is a construct keyword in every syntactic position.

## Soft-reserved and newly discouraged names

Soft-reserved words can still be accepted as class, interface, trait, or enum names in some contexts, but should be avoided because PHP uses or may use them in syntax.

| Name | Guidance |
| --- | --- |
| `enum` | Used in enum declarations. Do not use it as a class-like name. |
| `resource` | A PHP value category and soft-reserved identifier. |
| `numeric` | Soft-reserved. Avoid it as a class-like name. |

PHP 8.6 deprecates additional identifier uses as part of reserving names for future syntax:

- `let` and `is` as class, function, or constant names.
- `_` as a constant name.
- `namespace` as a class-constant name.
- `readonly` as a function name.

Use ordinary descriptive identifiers instead. For code that must support multiple PHP versions, check all names against the oldest and newest target versions.

## Magic constants

Magic constants look like constants, but their values depend on where PHP encounters them.

| Magic constant | Value |
| --- | --- |
| `__LINE__` | Current source line number. |
| `__FILE__` | Full path of the current file. |
| `__DIR__` | Directory of the current file. |
| `__FUNCTION__` | Current function name. |
| `__CLASS__` | Current class name, including its namespace. |
| `__TRAIT__` | Current trait name, including its namespace. |
| `__METHOD__` | Current method name, including its class or trait. |
| `__NAMESPACE__` | Current namespace name. |
| `__PROPERTY__` | Current property name in supported property contexts. |
| `__COMPILER_HALT_OFFSET__` | Byte offset after `__halt_compiler()`. |

Magic constants are not user-defined constants and cannot be reassigned.

## Names that are not keywords

These names are important in PHP, but they belong to other categories:

| Name or form | What it is |
| --- | --- |
| `$this` | Special variable referring to the current object in a non-static method. |
| `$_GET`, `$_POST`, `$_SERVER`, `$_FILES`, `$_SESSION` | Predefined superglobal variables. |
| `PHP_VERSION`, `PHP_EOL`, `PHP_INT_MAX` | Predefined constants. |
| `__construct()`, `__get()`, `__set()`, `__toString()` | Magic methods, not language keywords. |
| `DateTimeImmutable`, `PDO`, `Throwable`, `stdClass` | Predefined classes or interfaces, not keywords. |
| `#[Attribute]`, `#[Override]` | Attribute syntax and attribute classes. The attribute names are not keywords. |
| `->`, `::`, `??`, `?->`, `=>`, `...` | Operators or punctuation, not keywords. |

A PHP keyword may be allowed as a method or property name even when it is forbidden as a function or class name. Do not assume that an allowed member name is safe in every syntactic position.

## Naming and version notes

- Use lowercase spelling for keywords, even though PHP generally parses keywords without regard to case.
- Variables are case-sensitive. Keep variable and class naming consistent with the project style.
- Do not use reserved words as identifiers. A name accepted by one PHP version may be reserved or deprecated in another.
- `fn` arrow functions were introduced in PHP 7.4.
- `match` expressions were introduced in PHP 8.0.
- Enums and readonly properties were introduced in PHP 8.1. The word `enum` remains categorized as soft-reserved in the keyword appendix.
- `true`, `false`, and `null` can be standalone types starting with PHP 8.2.
- The `array` and `callable` names became reserved against class-like declarations in PHP 8.5.
- PHP 8.6 adds the identifier deprecations listed above. Run compatibility checks and review migration notes when upgrading.
- Use the minimum PHP version in project configuration to prevent unsupported syntax from reaching older runtimes.

## Continue learning

- [PHP summary](php-summary.md)
- [Identifiers, keywords, reserved words, and naming](lessons/009-identifiers-keywords-reserved-words-and-naming.md)
- [PHP quick reference and glossary](lessons/153-php-quick-reference-and-glossary.md)
- [All lessons, Q&A, and examples](README.md#lessons)
