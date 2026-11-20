# 009 - Identifiers, Keywords, Reserved Words, and Naming: Questions and Answers

Use this Q&A to review [009 - Identifiers, Keywords, Reserved Words, and Naming](../lessons/009-identifiers-keywords-reserved-words-and-naming.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What is an identifier in a program?

A. A name used to refer to a program element<br>
B. A required line break<br>
C. A PHP installation folder<br>
D. A browser request

**Answer: A.** Identifiers include names used for things such as variables, functions, classes, methods, and constants.

### 2. Which is the variable marker in **$firstName**?

A. The **$** character<br>
B. The uppercase **N**<br>
C. The whole word **firstName**<br>
D. The semicolon after the assignment

**Answer: A.** PHP writes a variable with a dollar sign followed by its name. In this example, **firstName** is the name and **$** is the marker.

### 3. Which is a valid ordinary PHP variable name?

A. **$user2**<br>
B. **$2user**<br>
C. **$user-name**<br>
D. **$user name**

**Answer: A.** A variable name begins with a letter or underscore and can contain digits after its first character. Spaces and hyphens are not part of one ordinary variable name.

### 4. Which variable names refer to two different variables because of capitalization?

A. **$name** and **$Name**<br>
B. **$name** and **$name**<br>
C. **$user_count** and **$user_count**<br>
D. **$title** and **$title**

**Answer: A.** PHP variable names are case-sensitive. A change in letter case changes the variable name.

### 5. Which character can validly begin an ordinary PHP variable name after the **$** marker?

A. An underscore<br>
B. A digit<br>
C. A hyphen<br>
D. A space

**Answer: A.** A letter or underscore can begin the name. Digits can appear after the first character.

### 6. Which statement describes **$first-name**?

A. The hyphen is not allowed as part of one ordinary variable name<br>
B. It is a variable name with two words<br>
C. It is the same variable as **$firstName**<br>
D. It is a PHP keyword

**Answer: A.** A hyphen is not permitted inside an ordinary variable name. In an expression, the hyphen is an operator between names rather than part of one variable name.

### 7. Which is a PHP keyword from the lesson's examples?

A. **if**<br>
B. **courseTitle**<br>
C. **visitorName**<br>
D. **php-practice**

**Answer: A.** **if** has a special role in PHP syntax. The other examples are ordinary descriptive names, not PHP keywords.

### 8. Why should you avoid reusing a PHP-provided name for your own function or class?

A. It may be reserved or conflict with a built-in identifier<br>
B. PHP only permits one-word names<br>
C. Variables cannot contain letters<br>
D. It makes the source file open as HTML

**Answer: A.** PHP has keywords and predefined identifiers. Choose names that clearly belong to your application and do not conflict with language or built-in names.

### 9. Which naming style does this course use for example variables?

A. camelCase, such as **$firstName**<br>
B. all spaces, such as **$first name**<br>
C. hyphen-case, such as **$first-name**<br>
D. leading-number style, such as **$2first**

**Answer: A.** This course uses camelCase for consistency. It is a style choice, not a PHP syntax requirement.

### 10. Which statement about PHP's variable-name rules is accurate?

A. A variable name starts with a letter or underscore, followed by letters, digits, or underscores<br>
B. A variable name must start with a digit<br>
C. A variable name can contain any spaces and punctuation<br>
D. Variable names are case-insensitive

**Answer: A.** This is the basic rule for ordinary variable names taught in this chapter. The **$** marker appears before the name.

### 11. What is the safest beginner choice when a name might be reserved in PHP?

A. Choose a different descriptive name for your application symbol<br>
B. Use it anyway and hope every PHP version accepts it<br>
C. Add spaces to the reserved word<br>
D. Write the name in a browser URL

**Answer: A.** Avoid collisions with keywords and predefined identifiers. Descriptive application-specific names are easier to understand and less likely to conflict.

### 12. What does PHP reserve symbols beginning with **__** for?

A. Special internal or magic functionality<br>
B. Page titles only<br>
C. Local terminal commands<br>
D. Comments in HTML

**Answer: A.** PHP reserves the double-underscore prefix for magic functionality. Do not use it for your own symbols unless implementing documented behavior.

## True or false

### 1. In **$firstName**, the dollar sign is part of how PHP marks the variable.

**Answer: True.** PHP places **$** before a variable name when referring to the variable in code.

### 2. **$user2** is invalid because a variable name can never contain a number.

**Answer: False.** Digits are allowed after the first character. A variable name cannot begin with a digit.

### 3. PHP treats **$count** and **$Count** as the same variable.

**Answer: False.** Variable names are case-sensitive, so those are different names.

### 4. **$first_name** and **$firstName** can both be valid variable names.

**Answer: True.** Both follow the basic syntax rules. A project should choose a consistent style.

### 5. camelCase is required by PHP's parser for variable names.

**Answer: False.** camelCase is a naming convention. PHP accepts other valid spellings, such as **$first_name**.

### 6. The list of PHP reserved and predefined identifiers can be treated as exhaustive and never change.

**Answer: False.** The official lists are not exhaustive and PHP can evolve. Avoid names that conflict with current language or built-in symbols.

### 7. A hyphen can be included in an ordinary variable name if the name has two words.

**Answer: False.** Use camelCase or underscores instead. A hyphen is not part of one ordinary PHP variable name.

### 8. PHP reserves symbols beginning with **__** for special or magic functionality.

**Answer: True.** Use a normal descriptive prefix for application names unless implementing documented magic behavior.

### 9. **$this** can be assigned like any ordinary variable.

**Answer: False.** PHP manages **$this** in object methods, and it cannot be assigned as an ordinary variable.

### 10. A good variable name should help readers understand what the value represents.

**Answer: True.** Descriptive names reduce guesswork and make code easier to maintain.

## Short-answer questions

### 1. Define an identifier.

**Answer:** An identifier is a name used to refer to a program element, such as a variable, function, class, method, or constant.

### 2. What are the basic rules for an ordinary PHP variable name?

**Answer:** Write **$** before it. The name starts with a letter or underscore, then may contain letters, digits, and underscores. Do not put spaces or hyphens in the name.

### 3. Why are **$name** and **$Name** not interchangeable?

**Answer:** PHP variable names are case-sensitive. Their capitalization differs, so PHP treats them as separate variables.

### 4. What is a keyword?

**Answer:** It is a word with a special role in PHP syntax, such as **if**, **function**, **class**, or **return**.

### 5. What is the difference between a naming convention and a syntax rule?

**Answer:** A syntax rule determines whether PHP can parse a name. A naming convention is a style choice people follow for consistency and readability. camelCase is this course's chosen variable style, not a PHP requirement.

### 6. Why should you avoid names that conflict with PHP's predefined identifiers?

**Answer:** They can be reserved or conflict with built-in language functionality. A distinctive application-specific name avoids ambiguity and compatibility problems.

### 7. What should you remember about **$this**?

**Answer:** It is a special variable that PHP provides in object methods; it cannot be assigned like an ordinary variable. Object methods are covered later in the course.

### 8. Why should custom symbols not begin with **__**?

**Answer:** PHP reserves that prefix for special or magic functionality. Using it for application symbols risks a collision with language behavior.

### 9. Which style does this course use for variables, and is it mandatory?

**Answer:** The course uses camelCase, such as **$courseTitle**. It is not mandatory; a project should use a consistent style and follow its established conventions.

### 10. Why are names such as **$x** or **$temp** often poor choices for important values?

**Answer:** They do not explain what the values represent, so readers must search through the code or guess. Prefer names such as **$userCount** or **$pageTitle** when those match the value.

## Troubleshooting practice

### 1. PHP reports a syntax error near **$2user**. What is wrong?

**Answer:** The variable name starts with a digit after the **$** marker.

**Correction:** Rename it to a name that starts with a letter, such as **$user2**.

### 2. A value appears missing because one part of the code uses **$userName** and another uses **$username**. Why?

**Answer:** Variable names are case-sensitive, so the two spellings refer to different variables.

**Correction:** Choose one spelling and use it consistently everywhere.

### 3. Code uses **$first-name** as if it were one variable. How should it be renamed?

**Answer:** Use a valid style such as **$firstName** or **$first_name**. A hyphen is not allowed in one ordinary variable name.

### 4. A function or class declaration conflicts with a PHP keyword. What should you do?

**Answer:** Choose another descriptive name that does not reuse a keyword or predefined identifier.

### 5. A new class method is named **__render** even though it is not implementing PHP magic behavior. What is a better choice?

**Answer:** Rename it to a normal descriptive method name, such as **renderPage**. PHP reserves names beginning with **__** for magic functionality.

### 6. Two valid styles, **$firstName** and **$first_name**, appear in the same project. What should the team do?

**Answer:** Pick the project's existing convention and use it consistently for new names. The syntax permits both, but mixing styles makes code less predictable.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [008 - Statements, instruction separation, and comments](../lessons/008-statements-instruction-separation-and-comments.md) | [Read this lesson](../lessons/009-identifiers-keywords-reserved-words-and-naming.md) | [Open example](../examples/009-identifiers-keywords-reserved-words-and-naming.md) | [010 - Variables and assignment](../lessons/010-variables-and-assignment.md) |
