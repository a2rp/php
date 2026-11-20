# 018 - Type Juggling, Casting, and Conversion: Questions and Answers

Use this Q&A to review [018 - Type Juggling, Casting, and Conversion](../lessons/018-type-juggling-casting-and-conversion.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What determines the current type of a PHP variable that has no type declaration?

A. The variable's name<br>
B. The value currently assigned to it<br>
C. The file extension<br>
D. The number of times it has been used

**Answer: B.** PHP variables can hold values of different types. Assigning a new value changes the type the variable currently holds.

### 2. What is type juggling?

A. PHP's context-dependent automatic interpretation of a value as another type<br>
B. A way to rename variables<br>
C. A function that sorts arrays<br>
D. A requirement that every variable have a declaration

**Answer: A.** PHP may interpret values as another type in contexts such as arithmetic, concatenation, conditions, and comparisons.

### 3. What does this code print for **$result**?

~~~php
$quantity = "4";
$result = $quantity * 3;

var_dump($result);
~~~

A. **string(1) "12"**<br>
B. **int(12)**<br>
C. **bool(true)**<br>
D. A **TypeError**

**Answer: B.** The numeric string **"4"** is interpreted as a number in the multiplication. The result is the integer **12**.

### 4. After arithmetic uses a numeric string, what happens to the original variable?

A. It always becomes an integer<br>
B. It remains a string unless the program assigns a different value to it<br>
C. It becomes a boolean<br>
D. It is deleted

**Answer: B.** Interpreting a value numerically for an operation does not necessarily change the type stored in the variable.

### 5. What does arithmetic with a non-numeric string do in PHP 8 and later?

A. Always treats the string as zero without reporting a problem<br>
B. Throws a **TypeError** when the value cannot be interpreted as a number<br>
C. Converts it to an array<br>
D. Stops the PHP process before the operation is evaluated

**Answer: B.** PHP 8 throws a **TypeError** for arithmetic with a non-numeric string. Validate input before using it in arithmetic.

### 6. Which operator joins strings in PHP?

A. **+**<br>
B. **.**<br>
C. **=>**<br>
D. **->**

**Answer: B.** The dot operator concatenates strings. Scalar values used with it can be converted to strings.

### 7. What string does **(string) true** produce?

A. **"true"**<br>
B. **"1"**<br>
C. **"0"**<br>
D. An empty string

**Answer: B.** Boolean true converts to the string **"1"**. Boolean false converts to an empty string.

### 8. Which statement about boolean conversion is correct?

A. The string **"false"** converts to false<br>
B. The string **"0"** converts to false<br>
C. Every non-empty array converts to false<br>
D. The integer **1** converts to false

**Answer: B.** Among strings, the empty string and **"0"** are false-like. The non-empty string **"false"** is true-like.

### 9. What does **(bool) "false"** produce?

A. **bool(false)**<br>
B. **bool(true)**<br>
C. **string("false")**<br>
D. A **TypeError**

**Answer: B.** The string is non-empty and is not **"0"**, so it converts to true.

### 10. What does **(bool) "0"** produce?

A. **bool(true)**<br>
B. **bool(false)**<br>
C. **int(0)**<br>
D. **null**

**Answer: B.** The string **"0"** is a special false-like value in PHP's boolean conversion.

### 11. What does **(bool) []** produce?

A. **bool(true)**<br>
B. **bool(false)**<br>
C. **array()**<br>
D. **null**

**Answer: B.** An empty array is false-like. A non-empty array is true-like.

### 12. What does the **(int)** cast do to a floating-point value's fractional part?

A. Rounds to the nearest integer<br>
B. Discards the fractional part toward zero<br>
C. Always rounds upward<br>
D. Converts the value to a string

**Answer: B.** Casting **12.9** to an integer gives **12**, and casting **-12.9** gives **-12**. It does not round to the nearest integer.

### 13. Which cast requests a floating-point value?

A. **(decimal)**<br>
B. **(float)**<br>
C. **(real number)**<br>
D. **(number)**

**Answer: B.** **(float)** and its alias **(double)** request a floating-point conversion.

### 14. What does **(string) false** produce?

A. The text **"false"**<br>
B. The text **"0"**<br>
C. An empty string<br>
D. **null**

**Answer: C.** Boolean false converts to an empty string. Choose a display label explicitly if the text **"false"** is needed.

### 15. What does a cast such as **(int) $input** change?

A. It always changes the type stored in **$input**<br>
B. It converts the expression's value; the original variable remains unchanged unless reassigned<br>
C. It validates the input for the application<br>
D. It changes all variables with the same name

**Answer: B.** A cast produces a converted value for an expression. Assigning it back is what changes the variable's stored value.

### 16. Which function changes the type of a variable in place?

A. **settype()**<br>
B. **is_numeric()**<br>
C. **gettype()**<br>
D. **var_dump()**

**Answer: A.** **settype()** changes the variable passed to it. A cast expression alone does not modify the original variable.

### 17. What does **is_numeric("12.5")** return?

A. **true**<br>
B. **false**<br>
C. **12.5**<br>
D. A string

**Answer: A.** A numeric string such as **"12.5"** is numeric. **is_numeric()** does not establish every application rule, such as an allowed range.

### 18. Which comparison checks that both type and value match?

A. **==**<br>
B. **===**<br>
C. **=**<br>
D. **!=**

**Answer: B.** Strict equality with **===** requires the same type and value. Loose equality with **==** can convert values before comparing.

### 19. Why is casting untrusted input not enough to make it valid?

A. Casts only work on variables named **$input**<br>
B. Conversion does not check the application's meaning, allowed range, or other rules<br>
C. PHP does not allow casts<br>
D. Every cast produces a string

**Answer: B.** A value can be convertible but still invalid for the application. Validate according to the input's purpose before using it.

### 20. Which statement about floats is correct?

A. Every decimal fraction has an exact binary float representation<br>
B. Floats have limited precision and can involve rounding effects<br>
C. Floats are always integers internally<br>
D. A float cannot be used in arithmetic

**Answer: B.** Floating-point values have limited precision. Exact decimal currency calculations need a deliberate representation.

## True or false

### 21. A numeric string used in multiplication must become an integer in the original variable.

**Answer: False.** PHP can interpret it as a number for that operation while the variable continues to hold a string.

### 22. **(int) 12.9** rounds to **13**.

**Answer: False.** It discards the fraction toward zero, so the result is **12**.

### 23. The string **"0"** is false-like in a boolean context.

**Answer: True.** **"0"** is one of PHP's special false-like values.

### 24. The string **"0.0"** is also false-like because it represents numeric zero.

**Answer: False.** The false-like string is exactly **"0"**. The non-empty string **"0.0"** converts to true.

### 25. **(bool) "false"** is false because the text spells the word “false.”

**Answer: False.** It is a non-empty string, so it converts to true. Parse known textual values explicitly.

### 26. **settype($value, "integer")** changes **$value** in place.

**Answer: True.** **settype()** changes the variable's value and type.

### 27. **is_numeric()** proves that a submitted value is within the application's allowed range.

**Answer: False.** It only checks whether the value is numeric. The application must check range and any other domain rules separately.

### 28. In PHP 8 and later, arithmetic with a non-numeric string can throw a **TypeError**.

**Answer: True.** Check and validate the value before using it in arithmetic.

## Read the code

### 29. What does this code print?

~~~php
$value = "12";
$result = $value + 3;

var_dump($value);
var_dump($result);
~~~

**Answer:**

~~~text
string(2) "12"
int(15)
~~~

**Explanation:** PHP interprets **$value** numerically for addition. The operation does not reassign a value to **$value**, so it stays a string.

### 30. What does this code print?

~~~php
$value = "4.0";
$result = $value * 3;

var_dump($result);
~~~

**Answer:**

~~~text
float(12)
~~~

**Explanation:** The numeric string contains a decimal point, so it is interpreted as a floating-point value for multiplication.

### 31. What does the following code print?

~~~php
$message = "Items: " . 3;

var_dump($message);
~~~

**Answer:**

~~~text
string(8) "Items: 3"
~~~

**Explanation:** Concatenation converts the integer to its string representation and joins it to the text.

### 32. What do these values print?

~~~php
var_dump((bool) "");
var_dump((bool) "0");
var_dump((bool) "false");
var_dump((bool) "0.0");
~~~

**Answer:**

~~~text
bool(false)
bool(false)
bool(true)
bool(true)
~~~

**Explanation:** The empty string and **"0"** are false-like. The non-empty strings **"false"** and **"0.0"** are true-like.

### 33. What does this code print?

~~~php
var_dump(0 == "0");
var_dump(0 === "0");
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Loose comparison can convert the string before comparing. Strict comparison also checks that the types match, and here one value is an integer while the other is a string.

### 34. What does this code print?

~~~php
$amount = "12.9";
$whole = (int) $amount;

var_dump($amount);
var_dump($whole);
~~~

**Answer:**

~~~text
string(4) "12.9"
int(12)
~~~

**Explanation:** The cast creates an integer result without changing **$amount**. The fractional part is discarded toward zero.

### 35. What does this code print?

~~~php
$value = "25";
settype($value, "integer");

var_dump($value);
~~~

**Answer:**

~~~text
int(25)
~~~

**Explanation:** Unlike a cast expression assigned to another variable, **settype()** changes **$value** itself.

### 36. Is the input numeric, and what does the conditional print?

~~~php
$input = "24";

if (is_numeric($input)) {
    echo "numeric";
} else {
    echo "not numeric";
}
~~~

**Answer:**

~~~text
numeric
~~~

**Explanation:** **"24"** is a numeric string. The check does not establish whether 24 is allowed for a particular application.

### 37. What kind of error occurs in PHP 8 and later?

~~~php
$quantity = "four";
$total = $quantity * 3;
~~~

**Answer:** A **TypeError** is thrown because **"four"** cannot be interpreted as a number for arithmetic.

**Explanation:** Validate the input before performing the operation. Do not depend on invalid text being converted to zero.

## Write code

### 38. Multiply a numeric string by **3**, then show that the original variable remains a string.

**Answer:**

~~~php
$quantity = "4";
$total = $quantity * 3;

var_dump($quantity);
var_dump($total);
~~~

**Output:**

~~~text
string(1) "4"
int(12)
~~~

### 39. Convert **"12.9"** to an integer without changing the original variable.

**Answer:**

~~~php
$input = "12.9";
$whole = (int) $input;

var_dump($input);
var_dump($whole);
~~~

**Output:**

~~~text
string(4) "12.9"
int(12)
~~~

**Explanation:** The cast result is stored in **$whole**. The original **$input** remains a string.

### 40. Check whether input is numeric before converting it to a float.

**Answer:**

~~~php
$input = "12.5";

if (is_numeric($input)) {
    $amount = (float) $input;
    echo "Amount: " . $amount . PHP_EOL;
} else {
    echo "Enter a numeric amount." . PHP_EOL;
}
~~~

**Output:**

~~~text
Amount: 12.5
~~~

**Explanation:** A real application should also validate any required precision, range, or business rule.

### 41. Display the words **Enabled** or **Disabled** based on a boolean without relying on boolean-to-string conversion.

**Answer:**

~~~php
$isEnabled = false;
$label = $isEnabled ? "Enabled" : "Disabled";

echo $label . PHP_EOL;
~~~

**Output:**

~~~text
Disabled
~~~

**Explanation:** Casting false directly to a string gives an empty string. An explicit label communicates the intended meaning.

### 42. Show how **==** and **===** compare an integer and a numeric string.

**Answer:**

~~~php
$number = 0;
$text = "0";

var_dump($number == $text);
var_dump($number === $text);
~~~

**Output:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Loose equality can convert operands. Strict equality also requires matching types.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [017 - Objects, resources, callables, and iterables](../lessons/017-objects-resources-callables-and-iterables.md) | [Read this lesson](../lessons/018-type-juggling-casting-and-conversion.md) | [Open example](../examples/018-type-juggling-casting-and-conversion.md) | [019 - Type declarations and strict typing](../lessons/019-type-declarations-and-strict-typing.md) |
