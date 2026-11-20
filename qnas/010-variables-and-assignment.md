# 010 - Variables and Assignment: Questions and Answers

Use this Q&A to review [010 - Variables and Assignment](../lessons/010-variables-and-assignment.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. How does PHP write an ordinary variable named **courseTitle**?

A. **courseTitle**<br>
B. **$courseTitle**<br>
C. **#courseTitle**<br>
D. **@courseTitle**

**Answer: B.** PHP uses **$** before a variable name. The marker is included whenever the code refers to the variable.

### 2. What does this statement do?

~~~php
$lessonNumber = 10;
~~~

A. Compares **$lessonNumber** with **10**<br>
B. Assigns **10** to **$lessonNumber**<br>
C. Prints **10**<br>
D. Declares a new PHP function

**Answer: B.** The single equals sign is the assignment operator. It puts the right-hand value into the variable on the left.

### 3. What is the value of **$nextLesson** after these statements?

~~~php
$currentLesson = 10;
$nextLesson = $currentLesson + 1;
~~~

A. **10**<br>
B. **11**<br>
C. **$currentLesson + 1** as text<br>
D. It has no value

**Answer: B.** PHP evaluates the right-hand expression, **10 + 1**, and assigns the resulting value **11** to **$nextLesson**.

### 4. What does a later assignment do to an ordinary variable?

A. Always creates a second variable with the same name<br>
B. Replaces the variable's current value<br>
C. Converts the variable name into a constant<br>
D. Prints the previous value automatically

**Answer: B.** A variable can be assigned again. Later code reads the most recently assigned value.

### 5. What does this code print?

~~~php
$originalCount = 9;
$copiedCount = $originalCount;
$originalCount = 10;

echo $copiedCount;
~~~

A. **9**<br>
B. **10**<br>
C. **19**<br>
D. Nothing

**Answer: A.** With ordinary assignment of this simple integer value, **$copiedCount** receives the value **9**. Reassigning **$originalCount** does not update **$copiedCount**.

### 6. What is the recommended order when using a variable?

A. Read it first, then assign a value later<br>
B. Assign a deliberate value before reading it<br>
C. Use it without **$** first<br>
D. Assume PHP will choose the value the program needs

**Answer: B.** Initialize a variable before using it. Reading a variable that has not been set causes an undefined-variable diagnostic, and code should not rely on that situation.

### 7. Which statement about **=** is correct?

A. It always checks whether two values are equal<br>
B. It assigns the right-hand value to the left-hand variable<br>
C. It prints both sides<br>
D. It ends a PHP file

**Answer: B.** A single equals sign performs assignment. Equality comparisons use comparison operators, which are covered in a later chapter.

## True or false

### 8. A newline by itself is enough to separate two simple PHP statements.

**Answer: False.** Simple PHP statements need their proper terminator, usually a semicolon. A line break alone does not replace it.

### 9. After assigning **10** to **$completedLessons**, a later assignment of **11** means that later code reads **11** from that variable.

**Answer: True.** Assignment updates the variable's current value. Earlier output or copies of simple values are not retroactively changed.

### 10. If **$name** has not been assigned, PHP guarantees it behaves like an empty string in every version and configuration.

**Answer: False.** An undefined variable is not a safe substitute for an empty string. PHP reports an undefined-variable diagnostic; details vary by PHP version and configuration. Initialize the variable or handle the missing-value case explicitly.

## Short-answer questions

### 11. In **$pageTitle = "Welcome";**, identify the left side, the right side, and the operation.

**Answer:** The left side is **$pageTitle**, the right side is **"Welcome"**, and **=** assigns that string value to the variable.

### 12. Why can **$originalCount** and **$copiedCount** have different values after one was assigned from the other?

**Answer:** Ordinary assignment of a simple value gives the receiving variable its own value. Changing the original variable later does not change that copied integer value. PHP references are a separate feature that can deliberately make names refer to the same variable.

### 13. Why should variable names describe their purpose?

**Answer:** Names such as **$completedLessons** tell a reader what the value means. A vague name such as **$x** makes code harder to understand and maintain.

## Read and predict

### 14. What do the two output lines show?

~~~php
$completedLessons = 9;
echo "Before: ", $completedLessons, PHP_EOL;

$completedLessons = 10;
echo "After: ", $completedLessons, PHP_EOL;
~~~

**Answer:**

~~~text
Before: 9
After: 10
~~~

The second assignment replaces the current value, so the second output uses **10**.

### 15. Is this an assignment or an equality comparison, and what value does **$nextLesson** receive?

~~~php
$currentLesson = 4;
$nextLesson = $currentLesson + 1;
~~~

**Answer:** It is an assignment. PHP calculates **4 + 1** and assigns **5** to **$nextLesson**. The single **=** does not check equality.

## Find and correct the mistake

### 16. What is wrong with this code, and how should it be corrected?

~~~php
echo $courseTitle;
$courseTitle = "PHP Fundamentals";
~~~

**Answer:** The code tries to read **$courseTitle** before assigning it. Assign the value first:

~~~php
$courseTitle = "PHP Fundamentals";
echo $courseTitle;
~~~

### 17. A learner expects **$copy** to become **10** after this code. Is that expectation correct?

~~~php
$original = 9;
$copy = $original;
$original = 10;
~~~

**Answer:** No. **$copy** keeps the simple integer value **9** that it received during assignment. Only **$original** is changed to **10**.

### 18. What does this code do, and how can its name be improved?

~~~php
$x = 24;
echo $x;
~~~

**Answer:** It assigns **24** to **$x** and prints it. If the value represents a count, a descriptive name such as **$completedLessons** would make the code easier to understand.

## Coding practice

### 19. Write a small program that assigns a learner's name and lesson count, then prints both values.

**Answer:**

~~~php
<?php

$learnerName = "Asha";
$completedLessons = 9;

echo "Learner: ", $learnerName, PHP_EOL;
echo "Lessons completed: ", $completedLessons, PHP_EOL;
~~~

The variables are initialized before **echo** reads them.

### 20. Write code that copies a simple integer value, updates the original, and prints both values.

**Answer:**

~~~php
$originalCount = 9;
$copiedCount = $originalCount;

$originalCount = 10;

echo "Original: ", $originalCount, PHP_EOL;
echo "Copy: ", $copiedCount, PHP_EOL;
~~~

Expected output:

~~~text
Original: 10
Copy: 9
~~~

The ordinary assignment copied the simple value into **$copiedCount**. Updating **$originalCount** does not change that copy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [009 - Identifiers, keywords, reserved words, and naming](../lessons/009-identifiers-keywords-reserved-words-and-naming.md) | [Read this lesson](../lessons/010-variables-and-assignment.md) | [Open example](../examples/010-variables-and-assignment.md) | [011 - Variable scope, global variables, and static locals](../lessons/011-variable-scope-global-variables-and-static-locals.md) |
