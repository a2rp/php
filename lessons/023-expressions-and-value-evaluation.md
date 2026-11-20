# 023 - Expressions and Value Evaluation

An expression is a piece of PHP code that produces a value. Literals, variables, function calls, assignments, and operator combinations can all be expressions. Understanding the value an expression produces makes it easier to write correct conditions, calculations, and return statements.

## What you will learn

- What makes PHP code an expression
- How literals, variables, constants, and function calls provide values
- How operators combine expressions into new values
- Why an assignment is also an expression
- How expressions appear inside statements
- The difference between an expression's value and its side effects
- How prefix and postfix increment produce different values
- How conditions convert values to boolean
- How short-circuit logic can skip part of an expression
- Why precedence does not guarantee operand evaluation order

## What is an expression?

An expression is code that PHP evaluates to produce a value. A literal such as **7**, a variable such as **$price**, and a comparison such as **$price > 0** are all expressions.

~~~php
<?php
$price = 12.50;
$quantity = 4;

$subtotal = $price * $quantity;

echo $subtotal . PHP_EOL;
~~~

Output:

~~~text
50
~~~

The multiplication expression **$price * $quantity** produces the value **50**. That value is then assigned to **$subtotal**.

## Common sources of expression values

### Literals and variables

A literal writes a value directly in the program. A variable expression provides the variable's current value.

~~~php
<?php
$name = "Mina";
$greeting = "Hello, " . $name;

echo $greeting . PHP_EOL;
~~~

Output:

~~~text
Hello, Mina
~~~

The string literal **"Hello, "** and the variable expression **$name** are joined by the concatenation operator **.**. The complete expression produces a string.

### Constants

A named constant expression gives the value stored under that constant's name.

~~~php
<?php
const TAX_RATE = 0.08;

$tax = 100 * TAX_RATE;

echo $tax . PHP_EOL;
~~~

Output:

~~~text
8
~~~

A constant can participate in larger expressions just like a literal or a variable.

### Function calls

A function call used as an expression has the value returned by the function.

~~~php
<?php
function add(int $left, int $right): int
{
    return $left + $right;
}

$total = add(4, 6);
echo $total . PHP_EOL;
~~~

Output:

~~~text
10
~~~

The call **add(4, 6)** produces the returned integer **10**, which is assigned to **$total**. A function may also perform a side effect, such as printing or changing state; keep its returned value and its side effects conceptually separate.

## Operators produce values

An operator takes one or more operands and produces a result. The operands are expressions too.

~~~php
<?php
$subtotal = 45.00;
$discount = 5.00;

$total = $subtotal - $discount;
$isAffordable = $total <= 50;

echo $total . PHP_EOL;
var_dump($isAffordable);
~~~

Output:

~~~text
40
bool(true)
~~~

The subtraction expression produces a number. The comparison expression produces a boolean value, **true** or **false**.

Operators can be unary, binary, or ternary:

| Form | Operands | Example |
| --- | --- | --- |
| Unary | One | **!$isReady** |
| Binary | Two | **$price * $quantity** |
| Ternary | Three parts | **$score >= 50 ? "pass" : "retry"** |

This chapter focuses on the value an expression produces. How PHP groups expressions with multiple operators is covered in the next chapter.

## Assignments are expressions too

An assignment both changes a variable and produces the value assigned to it.

~~~php
<?php
$first = $second = 7;

echo $first . PHP_EOL;
echo $second . PHP_EOL;
~~~

Output:

~~~text
7
7
~~~

The assignment **$second = 7** produces **7**, and that result is then assigned to **$first**. Although chained assignment is valid, separate assignments are often easier to read when the values have different meanings.

Combined assignments also produce the updated value:

~~~php
<?php
$score = 10;
$score += 5;

echo $score . PHP_EOL;
~~~

Output:

~~~text
15
~~~

The expression **$score += 5** updates **$score** and has the resulting value **15**.

### Avoid accidental assignment in a condition

An assignment inside a condition is still an expression. This can be useful when intentional, but it is also a common source of mistakes.

~~~php
<?php
$limit = 5;

if ($limit = 10) {
    echo "The condition is true." . PHP_EOL;
}
~~~

This assigns **10** to **$limit**. Since **10** is truthy, the body runs. It does not compare **$limit** with **10**.

Write a comparison when you mean to compare:

~~~php
<?php
if ($limit === 10) {
    echo "The limit is ten." . PHP_EOL;
}
~~~

Use strict comparison **===** when both value and type should match.

## Expressions inside statements

A statement performs a step in the program. Many statements contain expressions.

~~~php
<?php
$subtotal = 12 * 3;
~~~

The whole line is an expression statement: it evaluates the assignment expression and ends with a semicolon.

An **if** statement is not itself a simple value expression; it uses a condition expression to decide which statements to execute.

~~~php
<?php
if ($subtotal > 20) {
    echo "Large order" . PHP_EOL;
}
~~~

Here **$subtotal > 20** is the condition expression. **echo** is a language construct used to produce output; it is not a function call that returns the printed string.

A **return** statement can also contain an expression:

~~~php
function square(int $number): int
{
    return $number * $number;
}
~~~

The expression **$number * $number** produces the value returned by the function.
## Prefix and postfix increment values

Both **++$number** and **$number++** increment the variable. Their expression values differ:

- Prefix increment changes the variable first, then produces the new value.
- Postfix increment produces the old value, then changes the variable.

~~~php
<?php
$number = 4;

$oldValue = $number++;
echo $oldValue . PHP_EOL;
echo $number . PHP_EOL;

$newValue = ++$number;
echo $newValue . PHP_EOL;
echo $number . PHP_EOL;
~~~

Output:

~~~text
4
5
6
6
~~~

Use increment by itself when the expression's value is not needed. When you do need the value, make it clear whether the old or new value is intended.

## Truth values of expressions

A condition needs a boolean decision. PHP converts the condition expression to **true** or **false** when necessary.

These values convert to **false**:

- **false**
- The integer **0** and float **0.0**
- The empty string **""** and the string **"0"**
- An empty array **[]**
- **null**

Other values convert to **true**, including the string **"false"**, the string **"0.0"**, non-empty arrays, and objects.

~~~php
<?php
$values = [false, 0, 0.0, "", "0", [], null, "0.0", "false"];

foreach ($values as $value) {
    echo get_debug_type($value) . ": ";
    var_dump((bool) $value);
}
~~~

Output:

~~~text
bool: bool(false)
int: bool(false)
float: bool(false)
string: bool(false)
string: bool(false)
array: bool(false)
null: bool(false)
string: bool(true)
string: bool(true)
~~~

This conversion is often called truthiness. It is convenient for conditions, but it can be too broad for validation. For example, **if ($input)** treats **0**, **"0"**, an empty string, and **null** alike. Compare explicitly when your application needs to distinguish those values.

~~~php
<?php
$input = "0";

if ($input !== "") {
    echo "The input is not an empty string." . PHP_EOL;
}
~~~

This condition checks specifically for the empty string instead of asking whether the value is truthy.

## Short-circuit evaluation

The logical operators **&&** and **||** can stop evaluating once the result is known:

- With **false && expression**, PHP does not need the right-hand expression; the result is already false.
- With **true || expression**, PHP does not need the right-hand expression; the result is already true.

This lets code check a prerequisite before accessing a value that depends on it.

~~~php
<?php
$account = null;

if ($account !== null && $account->isActive()) {
    echo "Account is active." . PHP_EOL;
}
~~~

When **$account** is **null**, the right side is skipped, so PHP does not try to call **isActive()** on **null**.

A short-circuited expression can also skip a function call. Avoid hiding important work inside a logical condition; put important actions on their own clearly visible statement.

## Side effects and expression values

The value produced by an expression is not the same thing as every effect it has on the program. Assignment changes a variable, increment changes a variable, and a function may print output or modify state.

~~~php
<?php
$count = 3;
$before = $count;
$count++;

$total = $before + $count;

echo $before . PHP_EOL;
echo $count . PHP_EOL;
echo $total . PHP_EOL;
~~~

Output:

~~~text
3
4
7
~~~

The first two values are made explicit with separate statements. This is easier to reason about than combining a read and a modification of **$count** in a single larger expression.

## Precedence is not evaluation order

When an expression contains several operators, precedence and associativity determine how PHP groups them. They do not generally specify the order in which the operands themselves are evaluated. The next chapter explains precedence and grouping in detail.

Avoid expressions that both read and modify the same value in a way that depends on evaluation order:

~~~php
<?php
$value = 1;
$result = $value + $value++;
~~~

Do not rely on one particular result for this expression. Its evaluation order is not generally specified. Split the work into separate statements instead:

~~~php
<?php
$value = 1;
$previousValue = $value;
$value++;
$result = $previousValue + $value;
~~~

Now each step and its result are explicit.

## Make expressions easier to read

Parentheses can show the intended grouping even when PHP's rules would already make it clear.

~~~php
<?php
$unitPrice = 12.50;
$quantity = 3;
$subtotal = $unitPrice * $quantity;
$discount = $subtotal >= 30 ? $subtotal * 0.10 : 0;
$total = $subtotal - $discount;

echo $total . PHP_EOL;
~~~

Output:

~~~text
33.75
~~~

The example separates the intermediate values. This makes the calculation easier to review and avoids a long expression whose pieces are difficult to inspect.

## Common mistakes

- Thinking every line of PHP code is an expression
- Forgetting that assignment itself produces a value
- Confusing **=** with a comparison operator such as **===**
- Assuming a function call expression is always free of side effects
- Expecting prefix and postfix increment to produce the same value
- Treating every non-empty string as truthy without remembering the special string **"0"**
- Using truthiness when validation needs a specific comparison
- Assuming the right side of **&&** or **||** always runs
- Relying on an unspecified evaluation order for operands with side effects
- Making a long expression harder to understand instead of naming intermediate values

## Try it yourself

1. Calculate a subtotal from a unit price and quantity, then store the result.
2. Call a function that returns a value and use that value in a calculation.
3. Compare two values and inspect the boolean result.
4. Assign a number with **$number++** and record both the expression result and updated variable.
5. Repeat the previous activity with **++$number** and explain the difference.
6. Cast **"0"**, **"false"**, and an empty array to boolean and compare the results.
7. Write a condition that checks an object is not **null** before calling one of its methods.
8. Rewrite an expression that reads and increments the same variable into separate statements.

## Key takeaways

- An expression is code that produces a value.
- Literals, variables, constants, function calls, assignments, and operators can all be expressions.
- Assignment and increment expressions both produce values and change state.
- Conditions convert values to boolean; remember that **"0"** is false while **"false"** is true.
- Short-circuit logic may skip the right side of **&&** or **||**.
- Keep side effects explicit and avoid depending on an unspecified operand evaluation order.
- Use intermediate variables and parentheses when they make an expression easier to understand.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [022 - Constants, predefined constants, and magic constants](022-constants-predefined-constants-and-magic-constants.md) | [Open Q&A](../qnas/023-expressions-and-value-evaluation.md) | [Open example](../examples/023-expressions-and-value-evaluation.md) | [024 - Operator precedence and associativity](024-operator-precedence-and-associativity.md) |