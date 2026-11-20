# 014 - Null, Booleans, Integers, and Floating-Point Numbers: Questions and Answers

Use this Q&A to review [014 - Null, Booleans, Integers, and Floating-Point Numbers](../lessons/014-null-booleans-integers-and-floating-point-numbers.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. How many values does PHP's **null** type have?

A. None<br>
B. One: **null**<br>
C. Two: **null** and **undefined**<br>
D. Every empty value

**Answer: B.** The **null** type has one value, **null**. An undefined variable is a different program state, even though careless code can make the distinction hard to see.

### 2. Which value is a boolean literal?

A. **"false"**<br>
B. **0**<br>
C. **false**<br>
D. **"0"**

**Answer: C.** The unquoted word **false** is a boolean literal. Quoted **"false"** is a string; **0** is an integer and **"0"** is a string.

### 3. What does **isset($name)** return when **$name** has been assigned **null**?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. It always throws an exception

**Answer: B.** **isset()** returns false for an undefined variable and for a variable whose value is **null**. Use **is_null()** when you need to check specifically for null.

### 4. What type is produced by this literal?

~~~php
0b1010
~~~

A. **string**<br>
B. **float**<br>
C. **int**<br>
D. **bool**

**Answer: C.** The **0b** prefix marks a binary integer. The value of binary **1010** is decimal **10**.

### 5. Which prefix begins a hexadecimal integer literal?

A. **0b**<br>
B. **0o**<br>
C. **0x**<br>
D. **0d**

**Answer: C.** A hexadecimal literal starts with **0x** or **0X**. Binary uses **0b**; octal can use **0o** in modern PHP.

### 6. Since which PHP version can octal integer literals use the **0o** prefix?

A. PHP 5.0<br>
B. PHP 7.0<br>
C. PHP 8.1<br>
D. PHP 8.4

**Answer: C.** The **0o** and **0O** octal prefixes were added in PHP 8.1. Older PHP versions use a leading zero for octal notation.

### 7. What is the purpose of underscores in a numeric literal such as **1_000_000**?

A. They change the number into a string<br>
B. They separate digits for readability without changing the value<br>
C. They mark a decimal point<br>
D. They make the number unlimited in size

**Answer: B.** In PHP 7.4 and newer, underscores can visually group digits in numeric literals. They do not change the value or increase its range.

### 8. Which constants report the runtime's integer limits?

A. **PHP_INT_MIN** and **PHP_INT_MAX**<br>
B. **PHP_MIN** and **PHP_MAX**<br>
C. **INT_START** and **INT_END**<br>
D. **PHP_FLOAT_MIN** and **PHP_FLOAT_MAX**

**Answer: A.** **PHP_INT_MIN** and **PHP_INT_MAX** expose the supported integer range. **PHP_INT_SIZE** reports the number of bytes used for an integer.

### 9. What may PHP do when an integer operation exceeds the platform's integer range?

A. Always wrap around to zero<br>
B. Convert the result to a float<br>
C. Convert the result to a boolean<br>
D. Automatically create a larger integer type with unlimited precision

**Answer: B.** An out-of-range integer literal or arithmetic result is represented as a float. Very large values may then lose integer precision.

### 10. What type does the **/** operator return in PHP?

A. Always **int**<br>
B. Always **float**<br>
C. **bool** when the result is whole<br>
D. **null** when there is a remainder

**Answer: B.** The division operator returns a float, including when the quotient has no fractional part. Use **intdiv()** when an integer quotient is intended.

### 11. What does **intdiv(17, 5)** return?

A. **int(3)**<br>
B. **float(3.4)**<br>
C. **int(4)**<br>
D. **null**

**Answer: A.** **intdiv()** returns the integer quotient and discards the fractional part by rounding toward zero. It does not round to the nearest integer.

### 12. Why can direct equality checks on calculated floats be unreliable?

A. Floats are always strings<br>
B. Some decimal fractions cannot be represented exactly in binary floating-point<br>
C. PHP compares every float as null<br>
D. Floats cannot be used in expressions

**Answer: B.** Many common decimal fractions have a tiny representation error in binary. The result can be extremely close to the expected value but not exactly equal.

## True or false

### 13. **null**, **false**, **0**, and **0.0** all have the same PHP type.

**Answer: False.** They are respectively a null value, a boolean, an integer, and a float. Their meanings also differ.

### 14. The quoted string **"true"** is the same type and value as the boolean literal **true**.

**Answer: False.** **true** is a boolean; **"true"** is a four-character string.

### 15. **is_null($value)** tests whether a value is specifically null.

**Answer: True.** It returns a boolean indicating whether the supplied value is **null**.

### 16. An integer has an unlimited range in PHP.

**Answer: False.** The integer range is finite and platform-dependent. Check **PHP_INT_MIN**, **PHP_INT_MAX**, or **PHP_INT_SIZE** on the current runtime.

### 17. The expression **17 / 5** returns an integer because both operands are integers.

**Answer: False.** The **/** operator returns a float. Here the result is **3.4**.

### 18. In **intdiv(-7, 2)**, the fractional part is rounded toward zero.

**Answer: True.** The result is **-3**, not **-4**. **intdiv()** truncates the quotient toward zero.

### 19. Every decimal fraction written in source code has an exact floating-point representation.

**Answer: False.** Many base-10 fractions, including **0.1**, cannot be represented exactly in the binary format commonly used for floats.

### 20. **var_dump()** can show the type and value of a variable.

**Answer: True.** It is useful while learning and debugging. Avoid leaving diagnostic output in user-facing pages.

## Short-answer questions

### 21. When would you use **null** instead of **false**?

**Answer:** Use **null** to represent the absence of a value, such as an optional value that has not been supplied. Use **false** when the data is a boolean answer that means “no.”

**Explanation:** A missing fee and a fee explicitly marked as unavailable may have different application meanings from a yes-or-no setting. Choose the representation that matches the domain.

### 22. Why can **isset($value)** and **is_null($value)** give different results for the same assigned variable?

**Answer:** For a variable assigned **null**, **isset($value)** returns false, while **is_null($value)** returns true.

**Explanation:** **isset()** checks whether the variable exists and has a non-null value. **is_null()** specifically checks the value. An undefined variable also makes **isset()** false, so it does not by itself distinguish undefined from assigned-null.

### 23. How can you check whether the current PHP runtime uses 32-bit or 64-bit integers?

**Answer:** Inspect **PHP_INT_SIZE**. A common 32-bit build reports 4 bytes and a common 64-bit build reports 8 bytes.

**Explanation:** Use the runtime constant instead of assuming the development machine and production server have identical integer sizes.

### 24. What does **0x2A** represent, and what is its decimal value?

**Answer:** It is a hexadecimal integer literal with decimal value **42**.

**Explanation:** The hexadecimal digits **2A** mean **2 × 16 + 10**, which equals **42**.

### 25. Why is storing a money amount as a float potentially risky?

**Answer:** Floating-point arithmetic can have small representation and rounding errors, so exact decimal totals may not behave as expected.

**Explanation:** A simple application can often store integer minor units, such as cents, and format them for display. More demanding financial calculations may require decimal arithmetic designed for that purpose.

### 26. What is the difference between **/** and **intdiv()**?

**Answer:** **/** returns a floating-point quotient. **intdiv()** returns an integer quotient with the fractional part discarded toward zero.

**Explanation:** For example, **17 / 5** is **3.4**, while **intdiv(17, 5)** is **3**.

## Read the code

### 27. What are the type and output values?

~~~php
$fee = null;
var_dump($fee);
var_dump(is_null($fee));
var_dump(isset($fee));
~~~

**Answer:**

~~~text
NULL
bool(true)
bool(false)
~~~

**Explanation:** The variable exists and contains **null**. **is_null()** tests that value; **isset()** returns false for null.

### 28. What do the four values represent?

~~~php
var_dump(false);
var_dump("false");
var_dump(0);
var_dump(0.0);
~~~

**Answer:**

~~~text
bool(false)
string(5) "false"
int(0)
float(0)
~~~

**Explanation:** A boolean, string, integer, and float remain distinct even when they might be used informally to mean “no” or “empty.”

### 29. What is printed?

~~~php
var_dump(0b1010);
var_dump(0o12);
var_dump(0x0A);
~~~

**Answer:**

~~~text
int(10)
int(10)
int(10)
~~~

**Explanation:** Each literal uses a different base, but all three encode the same integer. The **0o** form requires PHP 8.1 or newer.

### 30. What are the values and types produced by these expressions?

~~~php
var_dump(17 / 5);
var_dump(intdiv(17, 5));
~~~

**Answer:**

~~~text
float(3.4)
int(3)
~~~

**Explanation:** **/** performs floating-point division. **intdiv()** returns the quotient with the fractional part removed toward zero.

### 31. What does the comparison print?

~~~php
$result = 0.1 + 0.2;
var_dump($result === 0.3);
~~~

**Answer:**

~~~text
bool(false)
~~~

**Explanation:** The stored binary floating-point result is not exactly the same as the literal **0.3**. For approximate calculations, compare using an appropriate tolerance instead of exact equality.

## Write code

### 32. Write code that inspects a null value, a boolean, an integer, and a float.

**Answer:**

~~~php
$nickname = null;
$isActive = true;
$attemptCount = 2;
$averageScore = 87.5;

var_dump($nickname);
var_dump($isActive);
var_dump($attemptCount);
var_dump($averageScore);
~~~

**Explanation:** The output should identify each value as **NULL**, **bool**, **int**, or **float**.

### 33. Write code that prints the integer size and supported minimum and maximum.

**Answer:**

~~~php
echo PHP_INT_SIZE . PHP_EOL;
echo PHP_INT_MIN . PHP_EOL;
echo PHP_INT_MAX . PHP_EOL;
~~~

**Explanation:** These values describe the PHP runtime executing the script. Their values can differ across platforms.

### 34. Write code that compares the integer quotient and ordinary division of 23 by 4.

**Answer:**

~~~php
var_dump(23 / 4);
var_dump(intdiv(23, 4));
~~~

**Output:**

~~~text
float(5.75)
int(5)
~~~

**Explanation:** Ordinary division preserves the fractional result as a float. **intdiv()** gives the whole-number quotient rounded toward zero.

### 35. Write a type check that confirms a value is a float.

**Answer:**

~~~php
$measurement = 2.75;

if (is_float($measurement)) {
    echo "The measurement is a float.";
}
~~~

**Output:**

~~~text
The measurement is a float.
~~~

**Explanation:** **is_float()** checks the PHP type. It does not check whether the measurement is within a valid application range.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [013 - The PHP Type System and Value Model](../lessons/013-the-php-type-system-and-value-model.md) | [Read this lesson](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) | [Open example](../examples/014-null-booleans-integers-and-floating-point-numbers.md) | [015 - Strings and numeric strings](../lessons/015-strings-and-numeric-strings.md) |
