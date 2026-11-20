# 024 - Operator Precedence and Associativity

An expression can contain several operators. **Precedence** determines how PHP groups operators with different precedence, and **associativity** determines grouping when operators share a precedence level. Parentheses make the intended grouping explicit.

## What you will learn

- What operator precedence and associativity control
- How parentheses change grouping
- How common PHP operators are ordered from higher to lower precedence
- Why **&&** and **and** can produce different assignments
- Why relational comparisons and ternary expressions should not be chained
- How PHP 8 changed concatenation and ternary grouping
- How the pipe operator fits into PHP 8.5 and later
- Why precedence does not determine the order in which operands are evaluated

## Precedence controls grouping

Higher-precedence operators group more tightly than lower-precedence operators.

~~~php
<?php
$result = 1 + 5 * 3;

echo $result . PHP_EOL;
~~~

Output:

~~~text
16
~~~

Multiplication has higher precedence than addition, so PHP groups the expression like this:

~~~text
1 + (5 * 3)
~~~

Parentheses can explicitly select a different grouping:

~~~php
<?php
$result = (1 + 5) * 3;

echo $result . PHP_EOL;
~~~

Output:

~~~text
18
~~~

Use parentheses whenever they make the intended calculation easier to see, even if PHP's precedence rules already produce that grouping.

## Associativity controls grouping at the same level

When neighboring operators have the same precedence, associativity determines how they group.

### Left-associative operators

Subtraction is left-associative:

~~~php
<?php
$result = 10 - 3 - 2;

echo $result . PHP_EOL;
~~~

PHP groups this as **(10 - 3) - 2**, so the result is **5**.

### Right-associative operators

Exponentiation and assignment are right-associative:

~~~php
<?php
$power = 2 ** 3 ** 2;

$first = $second = 7;

echo $power . PHP_EOL;
echo $first . ", " . $second . PHP_EOL;
~~~

PHP groups these as **2 ** (3 ** 2)** and **$first = ($second = 7)**. The output is:

~~~text
512
7, 7
~~~

Associativity is about grouping. It does not generally promise the order in which operand expressions are evaluated.

## Common precedence groups

The following overview lists PHP's operator groups from higher to lower precedence. Operators in the same row have the same precedence. The table is a quick map; use parentheses to make important grouping explicit.

| Higher to lower | Operators or syntax | Associativity or note |
| --- | --- | --- |
| 1 | **new**, **clone** | Special forms |
| 2 | ****** | Right |
| 3 | Unary **+**, unary **-**, **++**, **--**, **~**, casts, **@** | Prefix or postfix form |
| 4 | **instanceof** | Left |
| 5 | **!** | Unary |
| 6 | *****, **/**, **%** | Left |
| 7 | Binary **+**, binary **-** | Left |
| 8 | **<<**, **>>** | Left |
| 9 | **.** | Left; lower than **+**, **-**, **<<**, and **>>** since PHP 8.0 |
| 10 | **|>** | Left; PHP 8.5 and later |
| 11 | **<**, **<=**, **>**, **>=** | Non-associative |
| 12 | **==**, **!=**, **===**, **!==**, **<=>** | Non-associative |
| 13 | **&** | Left |
| 14 | **^** | Left |
| 15 | **|** | Left |
| 16 | **&&** | Left |
| 17 | **||** | Left |
| 18 | **??** | Right |
| 19 | **? :** | Non-associative since PHP 8.0 |
| 20 | **=**, **+=**, **-=**, ***=**, **/=**, **.=**, **??=**, and other assignments | Right |
| 21 | **yield from**, **=>**, **yield** | Generator syntax |
| 22 | **print** | Special form |
| 23 | **and** | Left |
| 24 | **xor** | Left |
| 25 | **or** | Left |
| 26 | **include**, **include_once**, **require**, **require_once** | Special forms |
| 27 | **fn (...) =>** | Arrow-function syntax |
| 28 | **throw** | Special form |

The table summarizes grouping rules, not a recommendation to write long expressions without parentheses. Some entries are language constructs or syntax forms rather than everyday operators.

## The word operators and symbolic operators

The logical operators **&&** and **||** have higher precedence than assignment. The word operators **and** and **or** have lower precedence than assignment.

~~~php
<?php
$symbolic = true && false;
$word = true and false;

var_dump($symbolic);
var_dump($word);
~~~

Output:

~~~text
bool(false)
bool(true)
~~~

The first assignment is grouped as **$symbolic = (true && false)**. The second is grouped as **($word = true) and false**, so **$word** receives **true**.

Use **&&** and **||** for ordinary conditions. Parenthesize a word operator expression if you have a deliberate reason to use it with assignment.

## Non-associative comparisons

Relational and equality operators are non-associative. Do not write a mathematical-looking chain such as **$low < $value < $high**. It does not mean that **$value** is between both bounds.

Write the two comparisons explicitly and combine their boolean results:

~~~php
$isInRange = $low <= $value && $value <= $high;
~~~

This means **$low <= $value** and **$value <= $high**. Each comparison produces a boolean, and **&&** combines them.

## Ternary expressions need explicit grouping

Since PHP 8.0, the ternary operator is non-associative. A nested ternary must use parentheses to show which branch contains the next ternary.

~~~php
<?php
$score = 45;

$result = $score >= 50
    ? "pass"
    : ($score >= 40 ? "review" : "retry");

echo $result . PHP_EOL;
~~~

Output:

~~~text
review
~~~

Avoid stacking ternaries when a clear **if/elseif/else** statement would be easier to read.

## Concatenation and arithmetic

The concatenation operator **.** changed precedence in PHP 8.0. It now has lower precedence than arithmetic **+** and **-**, and lower precedence than bitwise shifts.

Use parentheses when concatenation and arithmetic appear together:

~~~php
<?php
$total = 10 + 5;
$message = "Total: " . $total;

echo $message . PHP_EOL;
~~~

Output:

~~~text
Total: 15
~~~

This two-step form makes it clear that arithmetic produces **15** before the string is built. Do not rely on the historical precedence between **.** and arithmetic operators; code written before PHP 8 may group differently.

## Null coalescing and assignment

The null coalescing operator **??** has lower precedence than **||** and **&&**, and higher precedence than assignment. It is right-associative.

Parentheses can make a default expression clear:

~~~php
<?php
$displayName = $name ?? "Guest";
~~~

The assignment receives the value of **$name** when it is set and not **null**; otherwise it receives **"Guest"**.

When combining **??** with comparisons, logical operators, or assignments, add parentheses to show which result should be used.

## The pipe operator in PHP 8.5 and later

PHP 8.5 added the pipe operator **|>**. It passes the value on its left to the callable on its right. In the precedence table, it binds more tightly than comparisons and less tightly than concatenation and shifts.

~~~php
<?php
$title = "  PHP Notes  ";

$slug = $title
    |> trim(...)
    |> strtolower(...);

echo $slug . PHP_EOL;
~~~

Output:

~~~text
php notes
~~~

This syntax requires PHP 8.5 or later. On earlier PHP versions, call the functions directly:

~~~php
<?php
$slug = strtolower(trim($title));
~~~

The pipe operator is optional syntax; the precedence concepts in the rest of the chapter also apply to earlier PHP versions.

## Precedence is not evaluation order

Precedence and associativity say how an expression is grouped. They do not generally specify which operand PHP evaluates first.

Avoid expressions where the result depends on the order of side effects:

~~~php
$value = 1;
$result = $value + $value++;
~~~

Do not rely on one particular result for this expression. Split the work into separate statements:

~~~php
<?php
$value = 1;
$previous = $value;
$value++;
$result = $previous + $value;
~~~

Now each read and modification has a clear place in the sequence.

## A practical rule

When an expression is important, make it easy to review:

- Use parentheses to show grouping.
- Store meaningful intermediate results in named variables.
- Keep assignments and increments separate from comparisons when possible.
- Prefer **&&** and **||** for ordinary logical conditions.
- Avoid chained comparisons and unparenthesized nested ternaries.
- Do not rely on an unspecified operand evaluation order.

## Common mistakes

- Assuming precedence determines the order of operand evaluation
- Forgetting that ***** groups before **+**
- Assuming all logical operator spellings have the same precedence
- Writing **$value = true and false** when the intention is to assign the whole logical result
- Chaining relational comparisons as if PHP interpreted them like mathematical notation
- Nesting ternary expressions without parentheses
- Relying on the pre-PHP-8 precedence of **.** with arithmetic
- Using the PHP 8.5 **|>** operator on an earlier PHP version
- Removing parentheses just to make an expression shorter

## Try it yourself

1. Add parentheses to make **2 + 3 * 4** evaluate as **20**.
2. Group **10 - 3 - 2** in the opposite order and compare the results.
3. Predict the values assigned by **$a = true && false** and **$b = true and false**.
4. Rewrite a chained range comparison using two comparisons and **&&**.
5. Nest a ternary with explicit parentheses, then rewrite it as **if/elseif/else**.
6. Build a message from a numeric calculation using a temporary variable or parentheses.
7. On PHP 8.5 or later, pipe a string through **trim()** and **strtolower()**.
8. Find an expression that modifies and reads the same variable, and split it into separate statements.

## Key takeaways

- Precedence determines grouping between different operator levels.
- Associativity determines grouping for operators at the same level.
- Parentheses are the clearest way to communicate intentional grouping.
- **&&** and **||** bind more tightly than assignment; **and** and **or** bind less tightly.
- Relational comparisons and ternary operators should not be chained without explicit grouping.
- PHP 8 changed concatenation precedence and made ternary non-associative.
- PHP 8.5 added the **|>** pipe operator.
- Grouping rules do not promise the general order of operand evaluation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [023 - Expressions and value evaluation](023-expressions-and-value-evaluation.md) | [Open Q&A](../qnas/024-operator-precedence-and-associativity.md) | [Open example](../examples/024-operator-precedence-and-associativity.md) | [025 - Arithmetic, assignment, and increment operators](025-arithmetic-assignment-and-increment-operators.md) |