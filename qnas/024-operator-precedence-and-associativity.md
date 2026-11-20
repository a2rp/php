# 024 - Operator Precedence and Associativity: Questions and Answers

Use this Q&A to review [024 - Operator Precedence and Associativity](../lessons/024-operator-precedence-and-associativity.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does operator precedence determine?

A. The order in which files are included<br>
B. How operators of different precedence are grouped<br>
C. The type of every variable<br>
D. Whether a function is called

**Answer: B.** Higher-precedence operators group more tightly than lower-precedence operators.

### 2. What is the value of **1 + 5 * 3**?

A. **18**<br>
B. **16**<br>
C. **21**<br>
D. **9**

**Answer: B.** Multiplication groups before addition, so the expression is **1 + (5 * 3)**.

### 3. What do parentheses do in an expression?

A. Force the grouping written inside them<br>
B. Change every operator into an assignment<br>
C. Always make an expression evaluate left to right<br>
D. Disable type checking

**Answer: A.** Parentheses explicitly group the enclosed expression.

### 4. How does **10 - 3 - 2** group?

A. **10 - (3 - 2)**<br>
B. **(10 - 3) - 2**<br>
C. **10 - (3 + 2)**<br>
D. It is invalid

**Answer: B.** Subtraction is left-associative, so equal-precedence subtractions group from the left.

### 5. How does **2 ** 3 ** 2** group?

A. **(2 ** 3) ** 2**<br>
B. **2 ** (3 ** 2)**<br>
C. **(2 * 3) ** 2**<br>
D. It is invalid

**Answer: B.** Exponentiation is right-associative, so the right exponent is grouped first.

### 6. How does **$a = $b = 7** group?

A. **($a = $b) = 7**<br>
B. **$a = ($b = 7)**<br>
C. **($a = 7) = $b**<br>
D. It always assigns different values

**Answer: B.** Assignment is right-associative. Both variables receive **7**.

### 7. Which operator has higher precedence than assignment?

A. **and**<br>
B. **or**<br>
C. **&&**<br>
D. All have lower precedence

**Answer: C.** The symbolic logical operator **&&** binds more tightly than assignment. The word operators **and** and **or** bind more loosely.

### 8. What value is stored in **$result** by **$result = true && false**?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. The string **"false"**

**Answer: B.** **&&** groups before assignment, so the boolean result **false** is assigned.

### 9. What value is stored in **$result** by **$result = true and false**?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. It is a syntax error

**Answer: A.** **and** has lower precedence than assignment. PHP groups this as **($result = true) and false**.

### 10. Since PHP 8.0, how does concatenation **.** group relative to arithmetic **+** and **-**?

A. It has higher precedence than both<br>
B. It has lower precedence than both<br>
C. It has exactly the same precedence as both<br>
D. It has no precedence

**Answer: B.** PHP 8 lowered the precedence of **.** below addition and subtraction.

### 11. Since PHP 8.0, how does the ternary operator associate?

A. Left-associative<br>
B. Right-associative<br>
C. Non-associative<br>
D. It always groups as an assignment

**Answer: C.** Since PHP 8.0, nested ternaries require explicit parentheses to disambiguate their grouping.

### 12. What does the operator **|>** do in PHP 8.5 and later?

A. Performs a bitwise OR<br>
B. Passes the value on its left to the callable on its right<br>
C. Defines a union type<br>
D. Assigns a value only when it is null

**Answer: B.** The pipe operator passes its left-hand value into a callable and evaluates to the callable's result.

### 13. Does operator precedence determine the order in which operand side effects happen?

A. Yes, always<br>
B. No, not in general<br>
C. Only for arithmetic<br>
D. Only for strings

**Answer: B.** Precedence determines grouping. PHP does not generally specify operand evaluation order for every expression.

### 14. Which comparison is the clearest way to check that **$value** is between **$low** and **$high**?

A. **$low < $value < $high**<br>
B. **$low < $value && $value < $high**<br>
C. **$low && $value && $high**<br>
D. **$low < $high < $value**

**Answer: B.** Write each comparison explicitly and combine their boolean results with **&&**.

### 15. What should you do when concatenation and arithmetic appear in the same expression?

A. Rely on the PHP version to choose the expected grouping<br>
B. Add parentheses or use an intermediate variable<br>
C. Replace every operator with **and**<br>
D. Remove spaces

**Answer: B.** Explicit grouping avoids ambiguity and makes code easier to review, especially across PHP versions.

## Short-answer questions

### 16. What is operator precedence?

**Answer:** The relative grouping strength of operators.

**Explanation:** An operator with higher precedence groups more tightly than one with lower precedence.

### 17. What is associativity?

**Answer:** A rule for grouping operators that share the same precedence level.

**Explanation:** For example, subtraction groups left to right, while assignment groups right to left.

### 18. How can you force **1 + 5** to group before multiplication by **3**?

**Answer:** Write **(1 + 5) * 3**.

**Explanation:** Parentheses select the intended grouping.

### 19. What value does **10 - 3 - 2** produce?

**Answer:** **5**.

**Explanation:** It groups as **(10 - 3) - 2** because subtraction is left-associative.

### 20. What value does **2 ** 3 ** 2** produce?

**Answer:** **512**.

**Explanation:** It groups as **2 ** (3 ** 2)**, which is **2 ** 9**.

### 21. Why can **true && false** and **true and false** behave differently when assigned?

**Answer:** The two operator spellings have different precedence relative to assignment.

**Explanation:** **&&** groups before assignment; **and** groups after assignment.

### 22. What value is assigned by **$flag = true and false**?

**Answer:** **true** is assigned to **$flag**.

**Explanation:** The expression groups as **($flag = true) and false**.

### 23. What changed about the concatenation operator in PHP 8.0?

**Answer:** The **.** operator now has lower precedence than **+**, **-**, and bitwise shifts.

**Explanation:** Expressions that relied on the older grouping may behave differently, so add parentheses.

### 24. Why should nested ternary expressions use parentheses?

**Answer:** The ternary operator is non-associative since PHP 8.0.

**Explanation:** Parentheses explicitly identify which branch contains the nested ternary.

### 25. What does non-associative mean for relational comparisons?

**Answer:** You should not chain operators at that level without explicitly combining comparisons.

**Explanation:** Write two comparisons, such as **$low <= $value && $value <= $high**.

### 26. Does **$low < $value < $high** express a mathematical range check in PHP?

**Answer:** No.

**Explanation:** Write both comparisons separately and combine their boolean results.

### 27. What does the PHP 8.5 pipe operator **|>** pass to its right-hand callable?

**Answer:** The value produced by the expression on its left.

**Explanation:** The complete pipe expression evaluates to the callable's returned value.

### 28. What should you do with an expression that reads and increments the same variable?

**Answer:** Split the read and increment into separate statements.

**Explanation:** This avoids relying on an unspecified operand evaluation order.

### 29. What is the difference between grouping and evaluation order?

**Answer:** Grouping determines which operations form subexpressions; evaluation order determines when operands are actually evaluated.

**Explanation:** Precedence and associativity describe grouping, but do not generally establish evaluation order for all operands.

## True or false

### 30. Multiplication has higher precedence than addition.

**Answer: True.** That is why **1 + 5 * 3** evaluates to **16**.

### 31. Parentheses can change the grouping of an expression.

**Answer: True.** **(1 + 5) * 3** groups differently from **1 + 5 * 3**.

### 32. Subtraction is right-associative.

**Answer: False.** Subtraction is left-associative.

### 33. Assignment is right-associative.

**Answer: True.** **$a = $b = 7** groups as **$a = ($b = 7)**.

### 34. **and** and **&&** have the same precedence.

**Answer: False.** **&&** has higher precedence than assignment; **and** has lower precedence.

### 35. The concatenation operator has lower precedence than addition in PHP 8 and later.

**Answer: True.** This change took effect in PHP 8.0.

### 36. PHP 8 allows unparenthesized nested ternaries to associate left to right.

**Answer: False.** Since PHP 8.0, the ternary operator is non-associative.

### 37. The PHP 8.5 pipe operator is the same as bitwise OR.

**Answer: False.** The pipe operator is **|>** and passes a value to a callable; bitwise OR is **|**.

### 38. Precedence rules generally promise which operand will be evaluated first.

**Answer: False.** They define grouping, not general operand evaluation order.

## Read the code

### 39. What does this code print?

~~~php
<?php
echo 1 + 5 * 3 . PHP_EOL;
echo (1 + 5) * 3 . PHP_EOL;
~~~

**Answer:**

~~~text
16
18
~~~

**Explanation:** Multiplication groups before addition in the first expression. The parentheses change the grouping in the second.

### 40. What values are printed?

~~~php
<?php
$a = true && false;
$b = true and false;

var_dump($a);
var_dump($b);
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** **&&** has higher precedence than assignment; **and** has lower precedence.

### 41. What is the value of **$result**?

~~~php
$result = 10 - 3 - 2;
~~~

**Answer:** **5**.

**Explanation:** Subtraction is left-associative, so PHP groups this as **(10 - 3) - 2**.

### 42. Is this nested ternary valid on PHP 8 or later?

~~~php
<?php
$result = $score >= 50 ? "pass" : $score >= 40 ? "review" : "retry";
~~~

**Answer:** No.

**Explanation:** Since PHP 8.0, nested ternary expressions must use parentheses to show their intended grouping.

## Write code

### 43. Write an expression that returns **20** by adding **2** and **3** before multiplying by **4**.

**Answer:**

~~~php
<?php
$result = (2 + 3) * 4;
~~~

**Explanation:** The parentheses force addition to group before multiplication.

### 44. Assign the result of both boolean comparisons **$low <= $value** and **$value <= $high** to **$insideRange**.

**Answer:**

~~~php
<?php
$insideRange = $low <= $value && $value <= $high;
~~~

**Explanation:** Each comparison returns a boolean, and **&&** requires both to be true.

### 45. Write a nested ternary that returns **"pass"** for a score of at least 50, **"review"** for at least 40, and **"retry"** otherwise.

**Answer:**

~~~php
<?php
$result = $score >= 50
    ? "pass"
    : ($score >= 40 ? "review" : "retry");
~~~

**Explanation:** Parentheses explicitly group the second ternary inside the first expression's false branch.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [023 - Expressions and value evaluation](../lessons/023-expressions-and-value-evaluation.md) | [Read this lesson](../lessons/024-operator-precedence-and-associativity.md) | [Open example](../examples/024-operator-precedence-and-associativity.md) | [025 - Arithmetic, assignment, and increment operators](../lessons/025-arithmetic-assignment-and-increment-operators.md) |