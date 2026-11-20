# 026 - Comparison, Identity, and Ordering Operators: Questions and Answers

Use this Q&A to review [026 - Comparison, Identity, and Ordering Operators](../lessons/026-comparison-identity-and-ordering-operators.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which operator checks that two values are equal and have the same type?

A. **==**<br>
B. **===**<br>
C. **!=**<br>
D. **<=>**

**Answer: B.** The identity operator requires both equal values and matching types.

### 2. What is the result of **20 == "20"**?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. A syntax error

**Answer: A.** A loose comparison can treat a numeric string and a number as equal.

### 3. What is the result of **20 === "20"**?

A. **true**<br>
B. **false**<br>
C. **20**<br>
D. A syntax error

**Answer: B.** The integer and string have different types, so they are not identical.

### 4. Which pair are both loose not-equal operators?

A. **!==** and **<>**<br>
B. **!=** and **<>**<br>
C. **!=** and **===**<br>
D. **<=>** and **!==**

**Answer: B.** Both **!=** and **<>** compare inequality after type juggling.

### 5. When is **$left !== $right** true?

A. Only when both values are the same type<br>
B. When their values differ or their types differ<br>
C. Only when both are strings<br>
D. Only when the left value is smaller

**Answer: B.** Strict not-identity is true if either the value or the type differs.

### 6. How does PHP compare the numeric strings **"10"** and **"2"** with **<**?

A. As numbers, so the result is **false**<br>
B. As numbers, so the result is **true**<br>
C. As strings, so the result is **true**<br>
D. They cannot be compared

**Answer: A.** Two numeric strings are compared numerically; ten is not less than two.

### 7. What is the result of **"apple" < "banana"**?

A. **true**<br>
B. **false**<br>
C. **"apple"**<br>
D. It depends on whether the strings are numeric

**Answer: A.** Ordinary strings are ordered lexicographically, and “apple” comes before “banana” in that comparison.

### 8. In PHP 8 or later, what is the result of **0 == "apple"**?

A. **true**<br>
B. **false**<br>
C. **0**<br>
D. A syntax error

**Answer: B.** PHP 8 compares the string representations in this case, and **"0"** is not equal to **"apple"**.

### 9. What did **0 == "apple"** return before PHP 8.0?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. It was always a syntax error

**Answer: A.** Before PHP 8, the non-numeric string was converted to zero for this loose comparison.

### 10. What is the result of **0 == false**?

A. **true**<br>
B. **false**<br>
C. **0**<br>
D. It is invalid

**Answer: A.** A loose comparison involving a boolean converts both operands to booleans.

### 11. What does a negative result from **$left <=> $right** tell you?

A. The left value is less than the right value<br>
B. The left value is greater than the right value<br>
C. The values have identical types<br>
D. The values are both false

**Answer: A.** The spaceship operator returns a value below zero when the left operand is less than the right operand.

### 12. What does **5 <=> 5** return?

A. **true**<br>
B. **false**<br>
C. **0**<br>
D. **1**

**Answer: C.** The spaceship operator returns zero when the operands compare equal.

### 13. Can a PHP program write a mathematical-looking chained comparison such as **$low < $value < $high**?

A. Yes, it checks both bounds automatically<br>
B. Yes, but only for integers<br>
C. No, relational comparison operators are non-associative<br>
D. Only if the variables are constants

**Answer: C.** PHP does not allow adjacent relational comparisons at the same non-associative precedence level.

### 14. What is a better approach when approximate floating-point values must be compared?

A. Compare their strings with **===**<br>
B. Compare the absolute difference with a chosen tolerance<br>
C. Cast both to booleans<br>
D. Use **<>** instead

**Answer: B.** A tolerance accounts for small representation and rounding differences in floating-point calculations.

### 15. What kind of value does **<=>** return?

A. Always a boolean<br>
B. An ordering value below, equal to, or above zero<br>
C. A string containing the operator name<br>
D. A copy of the left operand

**Answer: B.** It returns an integer that indicates less-than, equal, or greater-than ordering.

## Short-answer questions

### 16. What does a typical comparison operator return?

**Answer:** A boolean value, either **true** or **false**.

**Explanation:** The spaceship operator **<=>** is the exception; it returns an integer ordering result.

### 17. What is the difference between **==** and **===**?

**Answer:** **==** compares values after type juggling; **===** requires equal values and the same type.

**Explanation:** For example, **42 == "42"** is true, while **42 === "42"** is false.

### 18. What is the difference between **!=** and **!==**?

**Answer:** **!=** is loose inequality; **!==** is strict not-identity.

**Explanation:** Strict not-identity is true if either the values differ or their types differ.

### 19. Are **!=** and **<>** different in meaning?

**Answer:** No. Both are loose not-equal operators.

**Explanation:** They are two spellings of the same comparison operation.

### 20. When does PHP compare two strings numerically?

**Answer:** When both strings are numeric strings.

**Explanation:** For example, **"10" < "2"** is evaluated as the numeric comparison **10 < 2**, which is false.

### 21. What changed in PHP 8 for a loose comparison between a number and a non-numeric string?

**Answer:** PHP 8 compares their string representations instead of converting the non-numeric string to zero.

**Explanation:** As a result, **0 == "apple"** is false in PHP 8 and later.

### 22. Why can **0 == false** be true while **0 === false** is false?

**Answer:** Loose comparison converts the values for comparison; strict identity also checks their types.

**Explanation:** Zero can compare equal to false after boolean conversion, but an integer is not a boolean.

### 23. What does **$left <=> $right** return when the operands compare equal?

**Answer:** Zero.

**Explanation:** A negative value indicates left is less, and a positive value indicates left is greater.

### 24. Why should ordinary string ordering not be treated as a human-language sort?

**Answer:** PHP's ordinary string comparison does not automatically apply language, locale, or case-insensitive sorting rules.

**Explanation:** Use a sorting method appropriate for the text and language your application displays.

### 25. What does non-associative mean for relational comparison operators?

**Answer:** Two relational operators of that same precedence cannot be placed next to each other without grouping.

**Explanation:** A range check must be written as separate comparisons rather than chained mathematical notation.

### 26. How can a range check be expressed without chaining comparisons?

**Answer:** Compare the value to the lower bound and compare it separately to the upper bound.

**Explanation:** The resulting booleans can then be combined with a logical operator, covered in the next chapter.

### 27. Why should two calculated floats usually not be checked for exact equality?

**Answer:** Floating-point representation can introduce tiny rounding differences.

**Explanation:** Compare their absolute difference against a tolerance suited to the calculation.

### 28. What kind of values should be used with ordering operators?

**Answer:** Values whose ordering is meaningful and defined for the application.

**Explanation:** Do not assume unrelated or incomparable types have a useful application-specific order.

### 29. What type does an HTML form submit for a value that looks like an integer?

**Answer:** A string.

**Explanation:** Validate the submitted text and convert it deliberately when a numeric value is required.

## True or false

### 30. **===** checks both the value and type of its operands.

**Answer: True.** It is the strict identity operator.

### 31. **<>** is a strict not-identical operator.

**Answer: False.** **<>** is another spelling of loose not-equal; **!==** is strict not-identical.

### 32. **"10" < "2"** is true because the first string starts with the character 1.

**Answer: False.** Both are numeric strings, so PHP compares their numeric values; ten is not less than two.

### 33. In PHP 8 or later, **0 == "apple"** is false.

**Answer: True.** A non-numeric string is not converted to zero for this comparison in PHP 8 and later.

### 34. **0 == false** and **0 === false** have the same result.

**Answer: False.** Loose equality is true after boolean conversion; strict identity is false because the types differ.

### 35. The spaceship operator returns a boolean value.

**Answer: False.** It returns an integer ordering result below, equal to, or above zero.

### 36. PHP accepts **$low < $value < $high** as a chained mathematical range comparison.

**Answer: False.** Relational comparison operators are non-associative; write the comparisons separately.

### 37. A tolerance can help compare approximate floating-point results.

**Answer: True.** It accepts small differences caused by how floats represent decimal values.

## Read the code

### 38. What does this code print?

~~~php
<?php
$number = 5;
$text = "5";

var_dump($number == $text);
var_dump($number === $text);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Loose equality accepts the numeric string as equal to the number. Strict identity also checks the type.

### 39. What does this code print on PHP 8 or later?

~~~php
<?php
var_dump(0 == "apple");
~~~

**Answer:**

~~~text
bool(false)
~~~

**Explanation:** PHP 8 does not convert the non-numeric string to zero in this comparison.

### 40. What does this code print?

~~~php
<?php
var_dump("10" < "2");
var_dump("apple" < "banana");
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** The first pair consists of numeric strings and is compared numerically. The second pair is compared in lexicographic string order.

### 41. What does this code print?

~~~php
<?php
var_dump(0 == false);
var_dump(0 === false);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Loose comparison converts both values to booleans; strict identity also requires the same type.

### 42. What does this code print?

~~~php
<?php
var_dump(3 <=> 8);
var_dump(8 <=> 8);
var_dump(12 <=> 8);
~~~

**Answer:**

~~~text
int(-1)
int(0)
int(1)
~~~

**Explanation:** The left operand is respectively less than, equal to, and greater than the right operand.

## Write code

### 43. Write a strict check that confirms **$submittedId** has the same value and type as **$storedId**.

**Answer:**

~~~php
<?php
$matches = $submittedId === $storedId;
~~~

**Explanation:** The identity operator checks both value and type.

### 44. Write a range check that makes **$isInRange** true when **$value** is at least **$minimum** and no greater than **$maximum**.

**Answer:**

~~~php
<?php
$isInRange = $value >= $minimum && $value <= $maximum;
~~~

**Explanation:** The expression contains two separate comparisons and combines their boolean results. The next chapter explains **&&**.

### 45. Write a sorting callback that orders products by ascending numeric price using the spaceship operator.

**Answer:**

~~~php
<?php
usort(
    $products,
    function (array $first, array $second): int {
        return $first['price'] <=> $second['price'];
    }
);
~~~

**Explanation:** The callback returns a negative value when the first price is lower, zero for equal prices, and a positive value when it is higher. That gives the sorting function the order it needs.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [025 - Arithmetic, Assignment, and Increment Operators](../lessons/025-arithmetic-assignment-and-increment-operators.md) | [Read this lesson](../lessons/026-comparison-identity-and-ordering-operators.md) | [Open example](../examples/026-comparison-identity-and-ordering-operators.md) | [027 - Logical Operators and Short-Circuit Evaluation](../lessons/027-logical-operators-and-short-circuit-evaluation.md) |