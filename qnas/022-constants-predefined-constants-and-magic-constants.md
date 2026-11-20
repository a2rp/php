# 022 - Constants, Predefined Constants, and Magic Constants: Questions and Answers

Use this Q&A to review [022 - Constants, Predefined Constants, and Magic Constants](../lessons/022-constants-predefined-constants-and-magic-constants.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does a PHP named constant represent?

A. A value that can be reassigned with a dollar sign<br>
B. A name for a value that cannot be changed after definition<br>
C. A variable limited to one function<br>
D. A value that is recalculated every time it is read

**Answer: B.** A named constant keeps its value after it has been defined and cannot be reassigned or undefined.

### 2. Which syntax reads a named constant?

A. **$APP_NAME**<br>
B. **APP_NAME**<br>
C. **@APP_NAME**<br>
D. **constant APP_NAME**

**Answer: B.** A constant name has no dollar sign. The dollar sign marks a variable such as **$appName**.

### 3. Which declaration is suitable for a fixed source-level constant?

A. **const APP_NAME = "Planner";**<br>
B. **var APP_NAME = "Planner";**<br>
C. **let APP_NAME = "Planner";**<br>
D. **static APP_NAME = "Planner";**

**Answer: A.** PHP uses the **const** keyword to declare a fixed constant in source code.

### 4. When is **define()** especially useful?

A. When a constant must be created conditionally at runtime<br>
B. When an ordinary variable needs a new value<br>
C. When declaring a function parameter<br>
D. When creating a loop

**Answer: A.** **define()** is a function, so code can call it at runtime, including inside a conditional or function.

### 5. Which is the correct call to define a constant named **BUILD_LABEL**?

A. **define(BUILD_LABEL, "local");**<br>
B. **define("$BUILD_LABEL", "local");**<br>
C. **define("BUILD_LABEL", "local");**<br>
D. **const("BUILD_LABEL", "local");**

**Answer: C.** **define()** receives the constant name as a string and then its value.

### 6. Are PHP named constants case-sensitive?

A. No, uppercase and lowercase are always identical<br>
B. Yes, **APP_NAME** and **App_Name** are different names<br>
C. Only names created by **const** are case-sensitive<br>
D. Only names created by **define()** are case-sensitive

**Answer: B.** Constant names are case-sensitive. Uppercase spelling is a convention, not a requirement for immutability.

### 7. Where can a normal **const** declaration be placed?

A. Inside an **if** block at runtime<br>
B. Inside a function body<br>
C. At an allowed declaration scope such as the file or namespace level<br>
D. Only inside a class

**Answer: C.** A namespace-level **const** is a declaration, not a runtime statement. It cannot be placed inside a function or runtime conditional.

### 8. What is the main difference between **const** and **define()**?

A. **const** creates variables and **define()** creates functions<br>
B. **const** is a source-level declaration; **define()** creates a constant at runtime<br>
C. **define()** constants can be reassigned<br>
D. **const** names always include a dollar sign

**Answer: B.** Choose between them based on when and how the constant needs to be declared.

### 9. What does **defined("PHP_VERSION")** return when the constant exists?

A. The constant's value<br>
B. Its name as a string<br>
C. A boolean indicating whether it exists<br>
D. The PHP version as an integer

**Answer: C.** **defined()** checks existence. Use **constant()** to retrieve a value from a name stored in a string.

### 10. Which core predefined constant contains a numeric version identifier?

A. **PHP_VERSION**<br>
B. **PHP_VERSION_ID**<br>
C. **PHP_SAPI**<br>
D. **PHP_EOL**

**Answer: B.** **PHP_VERSION_ID** is an integer designed for version comparisons.

### 11. What does **__FILE__** provide?

A. The current function name<br>
B. The full path and filename of the current PHP file<br>
C. The current working directory of the terminal<br>
D. The PHP version

**Answer: B.** **__FILE__** refers to the file in which the magic constant appears.

### 12. What does **__DIR__** provide?

A. The directory containing the current PHP file<br>
B. The directory selected in the web browser<br>
C. The current namespace<br>
D. The current function name

**Answer: A.** **__DIR__** is based on the file's location, which is often useful for building file paths.

### 13. Which magic constant contains the current namespace?

A. **__CLASS__**<br>
B. **__TRAIT__**<br>
C. **__NAMESPACE__**<br>
D. **__FUNCTION__**

**Answer: C.** **__NAMESPACE__** is replaced with the namespace in which it appears.

### 14. What does **ClassName::class** evaluate to?

A. A new object of that class<br>
B. The class name as a string<br>
C. The class's source file contents<br>
D. The name of the current function

**Answer: B.** The **::class** syntax resolves a class name to a string without instantiating it.

### 15. What is true of extension-provided constants?

A. They are guaranteed on every PHP installation<br>
B. They are available only when their extension is available and enabled<br>
C. They are always variables<br>
D. They can be used only inside classes

**Answer: B.** Optional extensions may not be installed or enabled, so check an optional extension constant with **defined()** before reading it.

## Short-answer questions

### 16. Why are constant names conventionally written in uppercase?

**Answer:** Uppercase spelling makes constants easy to distinguish from variables and other identifiers.

**Explanation:** It is a style convention. PHP does not require uppercase names.

### 17. Can a variable and a constant have the same name text?

**Answer:** Yes. They are distinct names, for example **STATUS** and **$STATUS**.

**Explanation:** Changing the variable does not modify the constant.

### 18. Can a named constant be reassigned or undefined after it is created?

**Answer:** No.

**Explanation:** The value remains fixed for the lifetime of the script execution.

### 19. Why can **const** not be declared inside an **if** statement?

**Answer:** A namespace-level **const** is processed as a declaration rather than created by a runtime statement.

**Explanation:** Use **define()** when the decision to create a constant must happen conditionally at runtime.

### 20. Can **define()** be called inside a function?

**Answer:** Yes.

**Explanation:** It is a runtime function call, so it can execute in a function or conditional when needed.

### 21. What does **constant($name)** do?

**Answer:** It reads the value of the constant whose name is stored in **$name**.

**Explanation:** Check the name with **defined($name)** first if the constant may not exist.

### 22. Why should code check an optional extension constant with **defined()**?

**Answer:** The extension that provides it may not be installed or enabled.

**Explanation:** The constant can therefore be undefined on another machine even if it exists locally.

### 23. Does **define()** automatically add the current namespace to its string name?

**Answer:** No.

**Explanation:** Include the namespace in the name string when defining a namespaced constant with **define()**.

### 24. What does **PHP_VERSION** contain?

**Answer:** The running PHP version as a string.

**Explanation:** **PHP_VERSION_ID** is the integer form that is convenient for feature-version comparisons.

### 25. What kind of value is **PHP_SAPI**?

**Answer:** A string describing the current Server API, for example **cli** when run from the command line.

**Explanation:** The value depends on how PHP is running.

### 26. What is **PHP_EOL**?

**Answer:** A predefined constant containing the platform's newline sequence.

**Explanation:** It is useful when printing line-oriented output across operating systems.

### 27. How does **__LINE__** differ from a normal named constant?

**Answer:** Its value is derived from the source-code line where it appears.

**Explanation:** It is a magic constant resolved from source location, not a value created with **const** or **define()**.

### 28. What does **__METHOD__** identify inside a class method?

**Answer:** The class and method name.

**Explanation:** For example, a method named **summary()** in **Report** has a method context like **Report::summary**.

### 29. What is special about **__CLASS__** inside a trait method?

**Answer:** It identifies the class that uses the trait.

**Explanation:** **__TRAIT__** still identifies the trait itself.

### 30. When is **__PROPERTY__** available?

**Answer:** In a property hook, starting with PHP 8.4.

**Explanation:** It contains the property name for the hook in which it appears.

## True or false

### 31. A named constant is read with a dollar sign.

**Answer: False.** The dollar sign is used for variables, not named constants.

### 32. Uppercase spelling is required for PHP to treat a name as a constant.

**Answer: False.** Uppercase is a naming convention. The declaration mechanism establishes a constant.

### 33. A variable can be changed without changing a constant with similar spelling.

**Answer: True.** They occupy separate naming forms.

### 34. A namespace-level **const** can be declared inside a runtime **if** block.

**Answer: False.** Use **define()** for runtime conditional creation.

### 35. A constant defined by **define()** is automatically placed in the current namespace.

**Answer: False.** The namespace must be part of the string passed to **define()** if that is the intended name.

### 36. **__FILE__** and **__DIR__** are based on the current PHP source file.

**Answer: True.** They describe the file and its directory, not the terminal's current working directory.

### 37. **defined("__FILE__")** is the normal way to check whether the magic constant exists.

**Answer: False.** Magic constants are language tokens, not constants created with **define()**. **defined()** checks ordinary named constants.

### 38. Every extension-provided constant exists on every PHP installation.

**Answer: False.** Availability depends on the extension being present and enabled.

## Read the code

### 39. What does this code print?

~~~php
<?php
const APP_NAME = "Planner";

function printName(): void
{
    echo APP_NAME . PHP_EOL;
}

printName();
~~~

**Answer:**

~~~text
Planner
~~~

**Explanation:** A named constant can be read inside the function without declaring it with **global**.

### 40. What does this code print?

~~~php
<?php
define("BUILD_CHANNEL", "local");

$name = "BUILD_CHANNEL";

echo defined($name) ? constant($name) : "missing";
~~~

**Answer:**

~~~text
local
~~~

**Explanation:** **defined()** checks the dynamically stored name, and **constant()** reads its value.

### 41. What does this code print when the file is stored as **paths.php** inside a directory named **config**?

~~~php
<?php
echo basename(__FILE__) . PHP_EOL;
echo basename(__DIR__) . PHP_EOL;
~~~

**Answer:**

~~~text
paths.php
config
~~~

**Explanation:** The magic constants refer to this file's path and containing directory. The example uses **basename()** so the output does not depend on the machine's full path.

### 42. What does **Entry::class** evaluate to inside the **Planner** namespace?

**Answer:** The string **Planner\Entry**.

**Explanation:** It resolves a class name without creating an **Entry** object. The actual string contains one namespace separator.

## Write code

### 43. Declare a constant named **APP_NAME** with the value **"Planner"**, then print it.

**Answer:**

~~~php
<?php
const APP_NAME = "Planner";

echo APP_NAME . PHP_EOL;
~~~

**Explanation:** The constant is declared at the file's top level and read without a dollar sign.

### 44. Check whether a dynamically named constant exists and print its value if it does.

**Answer:**

~~~php
<?php
$name = "PHP_VERSION";

if (defined($name)) {
    echo constant($name) . PHP_EOL;
}
~~~

**Explanation:** **defined()** guards the lookup and **constant()** reads the value by name.

### 45. Build a path to **settings.php** next to the current PHP file.

**Answer:**

~~~php
<?php
$settingsPath = __DIR__ . DIRECTORY_SEPARATOR . "settings.php";
~~~

**Explanation:** The path is based on the source file's directory rather than the process's working directory.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [021 - Mixed, void, never, and literal types](../lessons/021-mixed-void-never-and-literal-types.md) | [Read this lesson](../lessons/022-constants-predefined-constants-and-magic-constants.md) | [Open example](../examples/022-constants-predefined-constants-and-magic-constants.md) | [023 - Expressions and value evaluation](../lessons/023-expressions-and-value-evaluation.md) |