# 023 - Expressions and Value Evaluation: Questions and Answers

Use this Q&A to review [023 - Expressions and Value Evaluation](../lessons/023-expressions-and-value-evaluation.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What is an expression?

A. Any line that contains a semicolon<br>
B. Code that PHP evaluates to produce a value<br>
C. A comment in a PHP file<br>
D. Only a number written in source code

**Answer: B.** Expressions produce values, such as a literal, variable value, function result, or operator result.

### 2. Which of these is an expression?

A. **$price * $quantity**<br>
B. A comment<br>
C. A blank line<br>
D. A closing PHP tag

**Answer: A.** The multiplication expression combines two operand values and produces a result.

### 3. What value does an assignment expression produce?

A. Always **null**<br>
B. The name of the variable<br>
C. The value assigned to the variable<br>
D. No value under any circumstances

**Answer: C.** For example, **$total = 8** assigns and produces the value **8**.

### 4. What value does a function call expression have?

A. The value returned by the function<br>
B. The function's source code<br>
C. The number of arguments<br>
D. Always **true**

**Answer: A.** A call such as **add(2, 3)** evaluates to the value returned by **add()**.

### 5. What does a comparison expression such as **$total > 10** produce?

A. A string<br>
B. An array<br>
C. A boolean value<br>
D. The original value of **$total**

**Answer: C.** A comparison evaluates to either **true** or **false**.

### 6. What is the difference between an expression and a statement?

A. Expressions produce values; statements perform program steps and may contain expressions<br>
B. Expressions end files; statements start files<br>
C. Statements always produce integers<br>
D. They are exactly the same concept

**Answer: A.** For example, **$total > 10** is an expression inside an **if** statement.

### 7. What value does **$number++** produce?

A. The value after incrementing<br>
B. The value before incrementing<br>
C. Always **1**<br>
D. **null**

**Answer: B.** Postfix increment produces the old value, then increments the variable.

### 8. What value does **++$number** produce?

A. The value before incrementing<br>
B. The value after incrementing<br>
C. Always **0**<br>
D. **null**

**Answer: B.** Prefix increment updates the variable first and produces the new value.

### 9. Which string converts to **false** in a boolean context?

A. **"false"**<br>
B. **"0"**<br>
C. **"no"**<br>
D. **"0.0"**

**Answer: B.** The string **"0"** is a special false-like string. Other non-empty strings shown here are truthy.

### 10. Which statement about **"false"** is correct in a boolean context?

A. It always converts to **false** because of its spelling<br>
B. It is a non-empty string and converts to **true**<br>
C. It causes a type error<br>
D. It becomes the boolean **false** automatically

**Answer: B.** PHP checks the actual string value; the text **"false"** is not the special string **"0"**.

### 11. What does **false && expression** do?

A. It always evaluates the right side first<br>
B. It skips the right side because the result is already false<br>
C. It changes **false** into **null**<br>
D. It executes the right side twice

**Answer: B.** The **&&** operator short-circuits once the result is known.

### 12. What does **true || expression** do?

A. It skips the right side because the result is already true<br>
B. It always evaluates both sides<br>
C. It produces **false**<br>
D. It throws an exception

**Answer: A.** The **||** operator short-circuits when its left side is already true.

### 13. What do operator precedence and associativity determine?

A. How an expression is grouped<br>
B. The type of every variable in the program<br>
C. Whether a function exists<br>
D. The order in which all operands must execute

**Answer: A.** Precedence and associativity determine grouping; they do not generally specify operand evaluation order.

### 14. What is the safest response when an expression both reads and changes the same variable?

A. Depend on the output from one PHP version<br>
B. Split the operations into separate statements<br>
C. Add more increments to the expression<br>
D. Replace the variable with a constant

**Answer: B.** Separate statements make the intended evaluation sequence explicit.

### 15. Why can **if ($value)** be too broad for input validation?

A. It only accepts arrays<br>
B. Several distinct values, including **0**, **"0"**, **""**, and **null**, convert to false<br>
C. It never converts values to boolean<br>
D. It compares only object identity

**Answer: B.** Use an explicit comparison when the application must distinguish these values.

## Short-answer questions

### 16. Name four common sources of expression values.

**Answer:** Literals, variables, constants, and function calls.

**Explanation:** Operators and assignments are also expressions.

### 17. What does the expression **$left + $right** produce?

**Answer:** The result of adding the current values of the two operands.

**Explanation:** The expression can be used in an assignment, a return statement, or another larger expression.

### 18. What is the value of **$x = 5** as an expression?

**Answer:** **5**.

**Explanation:** The assignment stores **5** in **$x** and itself evaluates to the assigned value.

### 19. Does a function call always have side effects?

**Answer:** No.

**Explanation:** A function call has a returned value if it returns one; it may also perform side effects, but those are separate properties.

### 20. Give one example of a side effect.

**Answer:** Assigning a new value to a variable, incrementing a variable, printing output, or changing object state.

**Explanation:** Side effects change observable program state beyond merely calculating a result.

### 21. Why is **$a = $b = 7** valid?

**Answer:** Assignment is an expression, so the inner assignment produces **7** for the outer assignment.

**Explanation:** Chained assignment is valid, though separate assignments may be clearer.

### 22. What is the difference between **=** and **===**?

**Answer:** **=** assigns a value; **===** compares both value and type.

**Explanation:** Confusing assignment with comparison can change state inside a condition.

### 23. Why does **$limit = 10** make an **if** condition true?

**Answer:** The expression assigns and evaluates to **10**, which is truthy.

**Explanation:** It does not ask whether the existing value equals **10**.

### 24. What is an expression statement?

**Answer:** An expression followed by a semicolon, used as a statement.

**Explanation:** For example, **$total = 12 * 3;** evaluates the assignment expression and completes a program step.

### 25. What does **$value++** return as an expression?

**Answer:** The value of **$value** before incrementing.

**Explanation:** The variable itself has the incremented value after the expression completes.

### 26. What does **++$value** return as an expression?

**Answer:** The incremented value.

**Explanation:** Prefix increment updates first, then yields the new value.

### 27. List four values that convert to **false**.

**Answer:** **false**, **0**, **""**, and **null**.

**Explanation:** The float **0.0**, string **"0"**, and empty array **[]** also convert to **false**.

### 28. Why can **&&** safely guard a method call on a nullable object?

**Answer:** If the left side is false, short-circuiting skips the right side.

**Explanation:** A condition like **$account !== null && $account->isActive()** will not call the method when **$account** is **null**.

### 29. Does precedence specify the order in which operands are evaluated?

**Answer:** No.

**Explanation:** Precedence and associativity determine grouping. PHP does not generally promise operand evaluation order for every expression.

## True or false

### 30. Every PHP expression produces a value.

**Answer: True.** Producing a value is the defining property of an expression.

### 31. The line **$total = 5 + 3;** contains an expression and is an expression statement.

**Answer: True.** The addition is part of the assignment expression, and the statement ends with a semicolon.

### 32. An assignment expression always produces **null**.

**Answer: False.** It produces the value being assigned.

### 33. A comparison such as **$age >= 18** produces a boolean.

**Answer: True.** It evaluates to **true** or **false**.

### 34. **$number++** and **++$number** produce the same expression value.

**Answer: False.** Postfix yields the old value; prefix yields the new value.

### 35. The string **"false"** converts to false because it spells the word false.

**Answer: False.** It is a non-empty string, so it converts to **true**.

### 36. The string **"0"** converts to false in a boolean context.

**Answer: True.** It is the special non-empty string that PHP treats as false-like.

### 37. If the left side of **false && expression** is false, PHP still must evaluate the right side.

**Answer: False.** **&&** short-circuits once the result is known.

### 38. Operator precedence always determines which operand's side effects happen first.

**Answer: False.** Precedence determines grouping, not the general evaluation order of operands.

## Read the code

### 39. What does this code print?

~~~php
<?php
$price = 12;
$quantity = 3;
$total = $price * $quantity;

echo $total . PHP_EOL;
~~~

**Answer:**

~~~text
36
~~~

**Explanation:** The multiplication expression produces **36**, which is assigned to **$total**.

### 40. What does this code print?

~~~php
<?php
$number = 4;
$old = $number++;
$new = ++$number;

echo $old . PHP_EOL;
echo $new . PHP_EOL;
echo $number . PHP_EOL;
~~~

**Answer:**

~~~text
4
6
6
~~~

**Explanation:** Postfix stores the old value **4** and increments to **5**. Prefix then increments to **6** and stores **6**.

### 41. What does each condition evaluate to?

~~~php
<?php
var_dump((bool) "0");
var_dump((bool) "false");
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** **"0"** is the special false-like string. **"false"** is an ordinary non-empty string.

### 42. Does this call **checkAccount()**?

~~~php
<?php
function checkAccount(): bool
{
    echo "checked" . PHP_EOL;
    return true;
}

$accountExists = false;
$result = $accountExists && checkAccount();
~~~

**Answer:** No.

**Explanation:** The left side of **&&** is false, so PHP already knows the result and skips the function call.

## Write code

### 43. Calculate a total from a unit price and quantity, then print it.

**Answer:**

~~~php
<?php
$unitPrice = 8;
$quantity = 3;
$total = $unitPrice * $quantity;

echo $total . PHP_EOL;
~~~

**Explanation:** Multiplication produces the value assigned to **$total**.

### 44. Write a condition that checks an object is not null before calling its **isActive()** method.

**Answer:**

~~~php
<?php
if ($account !== null && $account->isActive()) {
    echo "Active account" . PHP_EOL;
}
~~~

**Explanation:** The left side prevents the method call when **$account** is **null**.

### 45. Rewrite a postfix increment so each step is explicit and the old value is saved.

**Answer:**

~~~php
<?php
$number = 4;
$oldValue = $number;
$number++;

echo $oldValue . PHP_EOL;
echo $number . PHP_EOL;
~~~

**Explanation:** Separate statements clearly show the old value and the updated variable without relying on a larger expression.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [022 - Constants, predefined constants, and magic constants](../lessons/022-constants-predefined-constants-and-magic-constants.md) | [Read this lesson](../lessons/023-expressions-and-value-evaluation.md) | [Open example](../examples/023-expressions-and-value-evaluation.md) | [024 - Operator precedence and associativity](../lessons/024-operator-precedence-and-associativity.md) |