# 027 - Logical Operators and Short-Circuit Evaluation: Questions and Answers

Use this Q&A to review [027 - Logical Operators and Short-Circuit Evaluation](../lessons/027-logical-operators-and-short-circuit-evaluation.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which operator is logical AND?

A. **||**<br>
B. **&&**<br>
C. **!**<br>
D. **xor**

**Answer: B.** Logical AND is written **&&** or **and**.

### 2. When does logical AND return true?

A. When both operands are truthy<br>
B. When either operand is truthy<br>
C. Only when both operands are false<br>
D. Whenever the left operand is truthy

**Answer: A.** Both operands must be truthy for AND to return true.

### 3. When does logical OR return true?

A. Only when both operands are true<br>
B. When at least one operand is truthy<br>
C. Only when both operands are false<br>
D. Only when the right operand is true

**Answer: B.** OR is true if either or both operands are truthy.

### 4. When does **xor** return true?

A. When both operands are truthy<br>
B. When both operands are false<br>
C. When exactly one operand is truthy<br>
D. Whenever either operand is false

**Answer: C.** Exclusive OR is true when the operands have different truth values.

### 5. What does the logical NOT operator **!** do?

A. Reverses a truth value<br>
B. Adds one to a value<br>
C. Checks strict identity<br>
D. Joins two strings

**Answer: A.** NOT turns a truthy value into false and a falsey value into true.

### 6. Which value is false in a PHP logical context?

A. The string **"false"**<br>
B. The string **"0"**<br>
C. The string **"no"**<br>
D. The array **[0]**

**Answer: B.** PHP treats the string **"0"** as false. The other listed values are truthy.

### 7. Which statement about **&&** and **and** is correct?

A. They have different truth rules.<br>
B. They have the same truth rule but different precedence.<br>
C. **and** short-circuits but **&&** does not.<br>
D. **&&** returns one of its operands.

**Answer: B.** Both are logical AND and short-circuit; **&&** binds more tightly than assignment, while **and** binds less tightly.

### 8. Which statement about **||** and **or** is correct?

A. They have the same truth rule but different precedence.<br>
B. **or** always returns its right operand.<br>
C. Only **||** can short-circuit.<br>
D. They mean logical XOR.

**Answer: A.** Both are logical OR and short-circuit, but **||** has higher precedence than assignment and **or** has lower precedence.

### 9. What is stored in **$result** here?

~~~php
<?php
$result = true && false;
~~~

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. The string **"false"**

**Answer: B.** **&&** binds more tightly than assignment, so the complete AND result, false, is assigned.

### 10. What is stored in **$result** here?

~~~php
<?php
$result = true and false;
~~~

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. A syntax error

**Answer: A.** **and** has lower precedence than assignment, so PHP groups this as **($result = true) and false**.

### 11. When does **&&** skip evaluating its right operand?

A. When its left operand is false<br>
B. When its left operand is true<br>
C. Never<br>
D. Only when both operands are variables

**Answer: A.** A false left operand already makes an AND result false, so PHP can skip the right side.

### 12. When does **||** skip evaluating its right operand?

A. When its left operand is false<br>
B. When its left operand is true<br>
C. Never<br>
D. Only when the right side is a function call

**Answer: B.** A true left operand already makes an OR result true, so PHP skips the right side.

### 13. What does PHP logical OR return for **0 || "ready"**?

A. **0**<br>
B. **"ready"**<br>
C. **true**<br>
D. **false**

**Answer: C.** The string is truthy, and a logical OR expression returns a boolean rather than one of its operands.

### 14. Which statement is true about the string **"false"** in a condition?

A. It is false because it spells the word false.<br>
B. It is true because it is a non-empty string other than **"0"**.<br>
C. It is a syntax error.<br>
D. It is converted to null.

**Answer: B.** PHP does not interpret the word's meaning; this non-empty string is truthy.

### 15. Why should an **xor** expression usually be parenthesized when assigned?

A. **xor** has lower precedence than assignment<br>
B. **xor** is not a PHP operator<br>
C. Parentheses convert it into OR<br>
D. **xor** always returns an integer

**Answer: A.** Without parentheses, assignment can happen before the word operator is evaluated.

## Short-answer questions

### 16. What does logical AND require?

**Answer:** Both operands must be truthy.

**Explanation:** If either operand is falsey, the AND result is false.

### 17. What does logical OR require?

**Answer:** At least one operand must be truthy.

**Explanation:** OR is false only when both operands are falsey.

### 18. What is the truth rule for **xor**?

**Answer:** Exactly one operand must be truthy.

**Explanation:** If both have the same truth value, XOR returns false.

### 19. What does the **!** operator return when its operand is truthy?

**Answer:** False.

**Explanation:** Logical NOT reverses the operand's truth value.

### 20. Name four values PHP treats as false in a logical context.

**Answer:** Examples include **false**, **0**, **0.0**, **""**, **"0"**, **[]**, and **null**.

**Explanation:** These are common falsey values. Most other values are truthy.

### 21. Is the string **"false"** false in a condition?

**Answer:** No. It is truthy.

**Explanation:** It is a non-empty string and is not the special falsey string **"0"**.

### 22. What is short-circuit evaluation?

**Answer:** Stopping evaluation of a logical expression once its result is already known.

**Explanation:** For example, a false left operand makes AND false without needing its right operand.

### 23. Which logical operators short-circuit?

**Answer:** **&&**, **and**, **||**, and **or**.

**Explanation:** AND can stop after a false left operand; OR can stop after a true left operand.

### 24. Why can short-circuit AND safely guard a division?

**Answer:** If the divisor check is false, PHP skips the division on the right.

**Explanation:** For example, **$divisor !== 0 && $number / $divisor > 2** does not divide when the first condition is false.

### 25. Do **&&** and **and** have different logical truth rules?

**Answer:** No. Their truth rules are the same.

**Explanation:** Their important difference is operator precedence.

### 26. Why can **$flag = true and false** leave **$flag** as true?

**Answer:** Assignment has higher precedence than **and**.

**Explanation:** PHP groups it as **($flag = true) and false**.

### 27. Why does **$flag = true && false** assign false?

**Answer:** **&&** has higher precedence than assignment.

**Explanation:** PHP evaluates the AND expression first, then assigns its false result.

### 28. What value is returned by a PHP logical OR expression?

**Answer:** A boolean.

**Explanation:** Unlike some languages, PHP's **||** does not return the first truthy operand.

### 29. When should a truthiness check be replaced with a specific comparison?

**Answer:** When zero, an empty string, null, or another falsey value has a distinct meaning in the application.

**Explanation:** A general condition treats several different values as false, so a specific comparison preserves the distinction.

## True or false

### 30. Logical AND returns true when both operands are truthy.

**Answer: True.** Both conditions must hold.

### 31. Logical OR returns false when either operand is truthy.

**Answer: False.** OR is true if at least one operand is truthy.

### 32. **xor** returns true when both operands are true.

**Answer: False.** XOR is true only when exactly one operand is truthy.

### 33. The string **"0"** is false in a PHP logical context.

**Answer: True.** It is one of PHP's special falsey values.

### 34. The string **"false"** is false because of its contents.

**Answer: False.** It is a non-empty string other than **"0"**, so it is truthy.

### 35. **and** and **&&** have identical precedence.

**Answer: False.** **&&** binds more tightly than assignment; **and** binds less tightly.

### 36. PHP evaluates the right side of **false && expression**.

**Answer: False.** The false left side already determines the result, so the right operand is skipped.

### 37. PHP logical operators return booleans.

**Answer: True.** **&&**, **||**, **and**, **or**, **xor**, and **!** produce boolean results.

## Read the code

### 38. What does this code print?

~~~php
<?php
$symbolic = true && false;
$word = true and false;

var_dump($symbolic);
var_dump($word);
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** **&&** binds before assignment. **and** binds after assignment, so the variable receives true.

### 39. What does this code print?

~~~php
<?php
$symbolic = false || true;
$word = false or true;

var_dump($symbolic);
var_dump($word);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** The symbolic OR expression is assigned as a whole. With **or**, the false assignment happens first.

### 40. Which lines are printed?

~~~php
<?php
echo "start" . PHP_EOL;

$result = false && print "right side" . PHP_EOL;

echo "end" . PHP_EOL;
~~~

**Answer:**

~~~text
start
end
~~~

**Explanation:** The left operand of AND is false, so PHP skips the right-side print.

### 41. What does this code print?

~~~php
<?php
var_dump(true xor false);
var_dump(true xor true);
var_dump(false xor false);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
bool(false)
~~~

**Explanation:** XOR is true only when exactly one operand is truthy.

### 42. What does this code print?

~~~php
<?php
var_dump((bool) "0");
var_dump((bool) "false");
var_dump((bool) []);
~~~

**Answer:**

~~~text
bool(false)
bool(true)
bool(false)
~~~

**Explanation:** The string **"0"** and an empty array are falsey. The non-empty string **"false"** is truthy.

## Write code

### 43. Write a condition that divides **$number** by **$divisor** only when the divisor is not zero.

**Answer:**

~~~php
<?php
$isLargeQuotient = $divisor !== 0
    && $number / $divisor > 2;
~~~

**Explanation:** A zero divisor makes the left condition false, so short-circuit AND skips the division.

### 44. Assign the XOR result of **$firstFlag** and **$secondFlag** to **$exactlyOneIsSet**.

**Answer:**

~~~php
<?php
$exactlyOneIsSet = ($firstFlag xor $secondFlag);
~~~

**Explanation:** Parentheses make PHP calculate the word operator expression before assigning it.

### 45. Write a condition that is true only when a user is signed in and has verified their email.

**Answer:**

~~~php
<?php
$mayUseAccount = $isSignedIn && $emailIsVerified;
~~~

**Explanation:** AND requires both boolean conditions to be true.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [026 - Comparison, Identity, and Ordering Operators](../lessons/026-comparison-identity-and-ordering-operators.md) | [Read this lesson](../lessons/027-logical-operators-and-short-circuit-evaluation.md) | [Open example](../examples/027-logical-operators-and-short-circuit-evaluation.md) | [028 - String, Array, Bitwise, and Other Operators](../lessons/028-string-array-bitwise-and-other-operators.md) |