# 020 - Nullable, Union, and Intersection Types: Questions and Answers

Use this Q&A to review [020 - Nullable, Union, and Intersection Types](../lessons/020-nullable-union-and-intersection-types.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does **?string** mean in a type declaration?

A. Only a string<br>
B. A string or **null**<br>
C. An optional string that cannot be null<br>
D. Any type

**Answer: B.** The nullable shorthand **?string** means **string|null**.

### 2. Does **?string** by itself make a function argument optional?

A. Yes, every nullable argument can be omitted<br>
B. No, it still needs a default value to be omittable<br>
C. Yes, but only in PHP 8.0<br>
D. No, nullable arguments cannot be passed

**Answer: B.** Nullability says which values are accepted. A default value such as **= null** makes the argument optional.

### 3. Which declaration accepts an integer, a string, or null?

A. **?int|string**<br>
B. **int|string|null**<br>
C. **int&string&null**<br>
D. **int,string,null**

**Answer: B.** A multi-member union that includes null lists all accepted types explicitly.

### 4. What does the **|** operator mean in a type declaration?

A. The value must satisfy every listed type<br>
B. The value may satisfy any one listed type<br>
C. The type is optional<br>
D. The function returns no value

**Answer: B.** A union uses “or”: a value must match at least one member.

### 5. What does **int|string** accept?

A. Only integers<br>
B. Only strings<br>
C. Integers or strings<br>
D. Objects implementing both types

**Answer: C.** Either an integer or a string satisfies the union.

### 6. What does the **&** operator mean in an intersection type?

A. The value can match one of the types<br>
B. An object must satisfy all listed class or interface types<br>
C. The value can be null<br>
D. The parameter is optional

**Answer: B.** An intersection such as **A&B** requires the object to satisfy both **A** and **B**.

### 7. Which types can be used in an intersection?

A. Only scalar types such as **int** and **string**<br>
B. Class and interface types<br>
C. Only **null** and **false**<br>
D. Any two built-in types

**Answer: B.** Intersection members must be class or interface types.

### 8. Which declaration requires an object to implement both interfaces?

A. **Loggable|Cacheable**<br>
B. **Loggable&Cacheable**<br>
C. **?Loggable**<br>
D. **Loggable|null**

**Answer: B.** The intersection requires the value to satisfy both interfaces.

### 9. From which PHP version are union types available?

A. PHP 7.1<br>
B. PHP 7.4<br>
C. PHP 8.0<br>
D. PHP 8.2

**Answer: C.** Union types were added in PHP 8.0.

### 10. From which PHP version are intersection types available?

A. PHP 7.1<br>
B. PHP 8.0<br>
C. PHP 8.1<br>
D. PHP 8.3

**Answer: C.** Intersection types were added in PHP 8.1.

### 11. From which PHP version are DNF types such as **(A&B)|C** available?

A. PHP 8.0<br>
B. PHP 8.1<br>
C. PHP 8.2<br>
D. PHP 8.3

**Answer: C.** DNF type syntax was added in PHP 8.2.

### 12. What does **(A&B)|C** mean?

A. The value must satisfy **A**, **B**, and **C**<br>
B. The value must satisfy **A** or satisfy both **B** and **C**<br>
C. The value must satisfy **A** and either **B** or **C**<br>
D. The value must be null

**Answer: B.** The grouped intersection is one union alternative: **A&B** or **C**.

### 13. Which type is redundant?

A. **int|string**<br>
B. **bool|false**<br>
C. **int|null**<br>
D. **Loggable&Cacheable**

**Answer: B.** **bool** already includes true and false, so adding **false** does not add another possibility.

### 14. Why is **iterable|array** redundant?

A. Arrays cannot be passed to functions<br>
B. Arrays already satisfy the iterable type<br>
C. Iterable only accepts strings<br>
D. The pipe operator means intersection

**Answer: B.** Arrays are iterable, so the **array** member adds no extra values to **iterable**.

### 15. Why is **object|Report** redundant when **Report** is a class?

A. A class instance is already an object<br>
B. Objects cannot be union members<br>
C. **Report** is a scalar type<br>
D. Object is nullable

**Answer: A.** Every instance of **Report** is an object, so **object** already includes that class type.

### 16. Which standalone literal types became available in PHP 8.2?

A. **int**, **string**, and **float**<br>
B. **true**, **false**, and **null**<br>
C. **array**, **object**, and **resource**<br>
D. **callable**, **iterable**, and **mixed**

**Answer: B.** PHP 8.2 added standalone **true**, **false**, and **null** types.

### 17. What syntax is a valid nullable type for a string-or-null value?

A. **string?**<br>
B. **?string**<br>
C. **string&null**<br>
D. **nullable string**

**Answer: B.** The question mark prefix is the nullable shorthand for one base type.

### 18. Which syntax should be used for an integer, string, or null?

A. **?int|string**<br>
B. **int|string|null**<br>
C. **int&string|null**<br>
D. **int?string?**

**Answer: B.** For multiple alternatives plus null, list each union member.

### 19. What happens if a call passes an array to a parameter declared **int|string**?

A. PHP accepts it because all types are allowed<br>
B. PHP throws a **TypeError** if it cannot satisfy either member<br>
C. PHP silently converts it to null<br>
D. PHP changes the function declaration

**Answer: B.** The value must match at least one union member. An array matches neither **int** nor **string**.

### 20. What does a nullable type declaration validate?

A. That the value meets all business rules<br>
B. That the value is the base type or null<br>
C. That the function argument is optional<br>
D. That a string is not empty

**Answer: B.** Nullability only adds **null** as an accepted type. It does not validate the value's meaning.

## True or false

### 21. **?T** is shorthand for **T|null**.

**Answer: True.** Both notations represent the same nullable type for a single base type.

### 22. A nullable parameter without a default can still be required.

**Answer: True.** It accepts null, but the caller must still pass an argument.

### 23. A parameter with a default of null should always omit a nullable type declaration.

**Answer: False.** Write the accepted type explicitly, such as **?string $name = null**. Implicitly nullable parameter declarations are deprecated as of PHP 8.4.

### 24. A union type requires a value to match every union member.

**Answer: False.** It must match at least one member. Requiring all members is the role of an intersection.

### 25. An intersection **A&B** accepts an object that satisfies only **A**.

**Answer: False.** The object must satisfy both **A** and **B**.

### 26. Scalar types such as **int** and **string** can be intersection members.

**Answer: False.** Intersection members must be class or interface types.

### 27. DNF type syntax was added in PHP 8.2.

**Answer: True.** PHP 8.2 supports unions that contain parenthesized intersections.

### 28. The parentheses in **(A&B)|C** group **A&B** as one side of the union.

**Answer: True.** The declaration accepts either an object matching both **A** and **B**, or a value matching **C**.

### 29. A union type automatically makes its argument optional.

**Answer: False.** It broadens accepted types. An argument is omittable only when it has a default value.

### 30. A type declaration can ensure that an integer is positive.

**Answer: False.** **int** accepts negative integers too. Add a separate range check.

### 31. **bool|false** adds a useful alternative to **bool**.

**Answer: False.** The boolean type already includes false, so this is redundant and rejected.

### 32. **?int|string** is the correct syntax for an integer, string, or null.

**Answer: False.** Write **int|string|null**. Question-mark shorthand only prefixes a single type.

## Read the code

### 33. What does this function return for each call?

~~~php
function label(?string $name): string
{
    return $name ?? "Guest";
}

echo label("Asha") . PHP_EOL;
echo label(null) . PHP_EOL;
~~~

**Answer:**

~~~text
Asha
Guest
~~~

**Explanation:** The parameter accepts a string or null. The null-coalescing operator uses the fallback for null.

### 34. Is this call valid?

~~~php
function formatId(int|string $id): string
{
    return (string) $id;
}

formatId(42);
~~~

**Answer: Yes.** The integer **42** matches one member of **int|string**.

### 35. Is this call valid?

~~~php
function formatId(int|string $id): string
{
    return (string) $id;
}

formatId([]);
~~~

**Answer: No.** An array matches neither member of the union, so PHP throws **TypeError**.

### 36. Does this call omit a required argument?

~~~php
function showName(?string $name): void
{
    var_dump($name);
}

showName();
~~~

**Answer: Yes, it is an error.** The parameter accepts **null**, but it has no default value, so the argument is still required.

### 37. What does this DNF declaration accept?

~~~php
function submit((Loggable&Cacheable)|Stringable $item): void
{
}
~~~

**Answer:** It accepts a value that implements both **Loggable** and **Cacheable**, or a value that implements **Stringable**.

**Explanation:** The parentheses make the intersection one alternative in the union.

### 38. Why is this declaration invalid or redundant?

~~~php
function check(bool|false $value): void
{
}
~~~

**Answer:** **bool** already includes false, so **false** is a redundant union member.

### 39. What does **?int** accept?

A value may be an integer or **null**. It does not accept a string just because the string contains digits.

**Explanation:** Nullability adds only **null** to the base type.

### 40. Which PHP versions are needed for these signatures?

~~~php
function choose(int|string $value): int|string {}
function combine(A&B $value): void {}
function publish((A&B)|C $value): void {}
~~~

**Answer:** The first needs PHP 8.0, the second PHP 8.1, and the DNF signature needs PHP 8.2.

**Explanation:** Each syntax was introduced in a different PHP version.

## Write code

### 41. Declare a function parameter that accepts either an integer or a string.

**Answer:**

~~~php
function formatId(int|string $id): string
{
    return "ID: " . $id;
}
~~~

**Explanation:** The pipe separates the permitted alternatives.

### 42. Declare a required nullable string parameter and return a fallback when it is null.

**Answer:**

~~~php
function displayName(?string $name): string
{
    return $name ?? "Guest";
}
~~~

The parameter is still required because it has no default value.

### 43. Make a nullable string parameter optional as well.

**Answer:**

~~~php
function displayName(?string $name = null): string
{
    return $name ?? "Guest";
}
~~~

**Explanation:** The nullable type accepts null; the default makes the parameter optional.

### 44. Write an intersection type requiring an object to implement two interfaces named **Readable** and **Writable**.

**Answer:**

~~~php
function process(Readable&Writable $stream): void
{
    // The argument must satisfy both interfaces.
}
~~~

### 45. Write a PHP 8.2 DNF type accepting either **Readable&Writable** or **Stringable**.

**Answer:**

~~~php
function process((Readable&Writable)|Stringable $value): void
{
}
~~~

**Explanation:** Parentheses group the intersection before the union alternative.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [019 - Type declarations and strict typing](../lessons/019-type-declarations-and-strict-typing.md) | [Read this lesson](../lessons/020-nullable-union-and-intersection-types.md) | [Open example](../examples/020-nullable-union-and-intersection-types.md) | [021 - Mixed, void, never, and literal types](../lessons/021-mixed-void-never-and-literal-types.md) |
