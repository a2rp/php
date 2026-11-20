# 025 - Arithmetic, Assignment, and Increment Operators: Questions and Answers

Use this Q&A to review [025 - Arithmetic, Assignment, and Increment Operators](../lessons/025-arithmetic-assignment-and-increment-operators.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which operator performs multiplication?

A. **+**<br>
B. *****<br>
C. **%**<br>
D. **/**

**Answer: B.** The asterisk operator multiplies numeric operands.

### 2. What does **17 - 9** evaluate to?

A. **8**<br>
B. **26**<br>
C. **-8**<br>
D. **1**

**Answer: A.** Subtraction returns the difference between its left and right operands.

### 3. What does **2 ** 5** evaluate to?

A. **10**<br>
B. **25**<br>
C. **32**<br>
D. **7**

**Answer: C.** Exponentiation raises the left operand to the power of the right operand: **2 × 2 × 2 × 2 × 2 = 32**.

### 4. What is the result of **17 % 5**?

A. **2**<br>
B. **3**<br>
C. **3.4**<br>
D. **0**

**Answer: A.** Five goes into seventeen three times, with a remainder of two.

### 5. Which expression returns the integer quotient of 17 divided by 5?

A. **17 / 5**<br>
B. **intdiv(17, 5)**<br>
C. **17 % 5**<br>
D. **17 ** 5**

**Answer: B.** The intdiv() function returns the integer quotient and truncates the fractional part toward zero.

### 6. What does **7 / 2** produce?

A. **3** as an integer in every case<br>
B. **3.5** as a float<br>
C. **1** as a remainder<br>
D. A syntax error

**Answer: B.** Division returns the regular quotient. A fractional quotient is a float.

### 7. When can division with integer operands return an integer?

A. Only when the result is negative<br>
B. When the division is exact<br>
C. Never; division always returns a float<br>
D. Only when using intdiv()

**Answer: B.** PHP can return an integer when both operands are integers and divide evenly.

### 8. What is the result of **intdiv(-7, 2)**?

A. **-4**<br>
B. **-3**<br>
C. **-3.5**<br>
D. **1**

**Answer: B.** Integer division truncates toward zero; it does not round down to the next more-negative integer.

### 9. Which statement about the modulo operator is correct?

A. It returns a fractional quotient.<br>
B. PHP converts its operands to integers before calculating the remainder.<br>
C. It rounds both operands to the nearest integer.<br>
D. It is another spelling of division.

**Answer: B.** The operands of **%** are converted to integers before the remainder is calculated.

### 10. What is the result of **-8 % 3**?

A. **2**<br>
B. **-2**<br>
C. **-1**<br>
D. **0**

**Answer: B.** The remainder has the same sign as the dividend. PHP truncates the quotient toward zero, so **-8 % 3** is **-2**.

### 11. Which operator updates a numeric variable by multiplying it by a value?

A. ***=**<br>
B. **+=**<br>
C. **%=**<br>
D. **-=**

**Answer: A.** **$value *= $factor** multiplies the current value by the factor and stores the result back in the variable.

### 12. What does **$total += 4** do?

A. Compares total with 4<br>
B. Adds 4 to total and stores the sum in total<br>
C. Makes total equal to 4 only when it is null<br>
D. Divides total by 4

**Answer: B.** The compound assignment is a shorter way to express a numeric addition and assignment.

### 13. What value does postfix increment return?

A. The value after incrementing<br>
B. The old value before incrementing<br>
C. Always 1<br>
D. A boolean

**Answer: B.** The postfix form returns the original value, then increments the variable.

### 14. What value does prefix increment return?

A. The old value before incrementing<br>
B. The updated value after incrementing<br>
C. Always 0<br>
D. A string

**Answer: B.** Prefix increment changes the variable first, then returns the new value.

### 15. Which is a good basic representation for an exact price of $12.50 in arithmetic?

A. The string **"$12.50"**<br>
B. The float **12.50** in every financial system<br>
C. The integer **1250** cents<br>
D. The integer **12** dollars

**Answer: C.** Storing minor currency units as an integer avoids common binary floating-point representation issues for simple amounts. A real financial system must also define its rounding rules.

## Short-answer questions

### 16. What does the addition operator do?

**Answer:** It returns the sum of its numeric operands.

**Explanation:** For example, **8 + 3** evaluates to **11**.

### 17. What does the modulo operator return?

**Answer:** The integer remainder after division.

**Explanation:** For example, **17 % 5** returns **2**, since five fits into seventeen three times with two left over.

### 18. What is the difference between **/** and **intdiv()**?

**Answer:** **/** returns a regular quotient; **intdiv()** returns an integer quotient truncated toward zero.

**Explanation:** **7 / 2** gives **3.5**, while **intdiv(7, 2)** gives **3**.

### 19. Does **intdiv()** round to the nearest integer?

**Answer:** No.

**Explanation:** It truncates the fractional part toward zero. For example, **intdiv(-7, 2)** returns **-3**.

### 20. Why can **8 / 2** and **7 / 2** have different result types?

**Answer:** The first division is exact, while the second has a fractional result.

**Explanation:** PHP can return an integer for exact division of integer operands and a float when a fractional quotient is needed.

### 21. What does the exponentiation operator calculate?

**Answer:** It raises the value on its left to the power on its right.

**Explanation:** For example, **2 ** 4** evaluates to **16**.

### 22. What does **$value -= 3** mean for a numeric variable?

**Answer:** Subtract **3** from the current value and store the result back in **$value**.

**Explanation:** It is the subtraction compound assignment.

### 23. What value does an assignment expression produce?

**Answer:** The value assigned to the variable.

**Explanation:** In **$score = 10**, the assignment expression's value is **10** as well as storing **10** in **$score**.

### 24. Is **=** the operator for checking whether two values are equal?

**Answer:** No. It assigns a value.

**Explanation:** Comparison uses operators such as **==** or **===**. Their behavior is covered in the next chapter.

### 25. How do **$count++** and **++$count** differ when used in a larger expression?

**Answer:** Postfix returns the old value before changing the variable; prefix changes the variable first and returns the new value.

**Explanation:** Both forms increase the variable by one. The difference is the value of the increment expression.

### 26. What does postfix decrement do?

**Answer:** It returns the old value and then subtracts one from the variable.

**Explanation:** The operator is written after the variable as **$count--**.

### 27. What does prefix decrement do?

**Answer:** It subtracts one first and returns the updated value.

**Explanation:** The operator is written before the variable as **--$count**.

### 28. Why should code check a divisor before using it?

**Answer:** A divisor must not be zero.

**Explanation:** Division, modulo, and integer division with a zero divisor raise a division-by-zero error in current PHP versions.

### 29. Why should an application avoid assuming that float arithmetic is exact?

**Answer:** Many decimal fractions cannot be represented exactly in binary floating-point.

**Explanation:** A calculation can contain a tiny rounding difference, even when the decimal arithmetic looks exact.

## True or false

### 30. The multiplication operator is written as an asterisk.

**Answer: True.** For example, **6 * 4** evaluates to **24**.

### 31. The modulo operator returns the quotient as a float.

**Answer: False.** It returns an integer remainder after converting its operands to integers.

### 32. The result of **intdiv(7, 2)** is **4** because it rounds up.

**Answer: False.** The result is **3** because intdiv() truncates toward zero instead of rounding.

### 33. The remainder from **-8 % 3** has the same sign as the dividend.

**Answer: True.** The dividend is the left operand, **-8**, so the remainder is negative: **-2**.

### 34. The compound assignment **$total += 5** updates **$total**.

**Answer: True.** It adds five to the current value and stores the result back in that variable.

### 35. Postfix increment returns the updated value first and then changes the variable.

**Answer: False.** Postfix returns the old value first, then increments the variable.

### 36. Prefix decrement changes the variable before returning its expression value.

**Answer: True.** **--$value** decreases first and returns the updated value.

### 37. A zero divisor is safe when using intdiv() because it returns zero.

**Answer: False.** A zero divisor raises a DivisionByZeroError in current PHP versions.

### 38. Storing a simple price as an integer number of cents avoids using a float for that stored amount.

**Answer: True.** Integer minor units are a useful basic representation, although percentage calculations still need a deliberate rounding policy.

## Read the code

### 39. What does this code print?

~~~php
<?php
var_dump(8 / 2);
var_dump(7 / 2);
~~~

**Answer:**

~~~text
int(4)
float(3.5)
~~~

**Explanation:** The first division is exact with integer operands; the second requires a fractional result.

### 40. What does this code print?

~~~php
<?php
echo intdiv(17, 5) . PHP_EOL;
echo 17 % 5 . PHP_EOL;
~~~

**Answer:**

~~~text
3
2
~~~

**Explanation:** Integer division returns the quotient; modulo returns the remainder.

### 41. What does this code print?

~~~php
<?php
echo intdiv(-7, 2) . PHP_EOL;
echo -7 % 2 . PHP_EOL;
~~~

**Answer:**

~~~text
-3
-1
~~~

**Explanation:** Integer division truncates toward zero. The remainder is negative because the dividend is negative.

### 42. What does this code print?

~~~php
<?php
$total = 20;
$total += 5;
$total *= 2;
$total -= 10;

echo $total . PHP_EOL;
~~~

**Answer:**

~~~text
40
~~~

**Explanation:** The value changes in order: **20 → 25 → 50 → 40**.

### 43. What does this code print?

~~~php
<?php
$count = 4;
$oldValue = $count++;
$newValue = ++$count;

echo $oldValue . PHP_EOL;
echo $newValue . PHP_EOL;
echo $count . PHP_EOL;
~~~

**Answer:**

~~~text
4
6
6
~~~

**Explanation:** Postfix stores the old value **4** and then increments to **5**. Prefix increments to **6** before storing its result.

## Write code

### 44. Write PHP code to calculate how many complete boxes of 6 items can be filled from 20 items, and how many items remain.

**Answer:**

~~~php
<?php
$completeBoxes = intdiv(20, 6);
$remainingItems = 20 % 6;

echo $completeBoxes . PHP_EOL;
echo $remainingItems . PHP_EOL;
~~~

Output:

~~~text
3
2
~~~

**Explanation:** Three boxes use eighteen items, leaving two. Integer division gets the complete boxes and modulo gets the remainder.

### 45. A quantity is stored in **$quantity** and a unit price in integer cents as **$unitPriceCents**. Write code to calculate the subtotal.

**Answer:**

~~~php
<?php
$subtotalCents = $unitPriceCents * $quantity;
~~~

**Explanation:** Multiplying the integer price in cents by the item count keeps this simple subtotal as an integer number of cents.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [024 - Operator Precedence and Associativity](../lessons/024-operator-precedence-and-associativity.md) | [Read this lesson](../lessons/025-arithmetic-assignment-and-increment-operators.md) | [Open example](../examples/025-arithmetic-assignment-and-increment-operators.md) | [026 - Comparison, Identity, and Ordering Operators](../lessons/026-comparison-identity-and-ordering-operators.md) |