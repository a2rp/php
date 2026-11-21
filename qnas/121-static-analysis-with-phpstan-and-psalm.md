# Static analysis with PHPStan and Psalm: Questions and answers

## Fundamentals

### 1. What is static analysis?

**Answer:** Static analysis examines source code without executing the application. **Explanation:** It uses syntax, type declarations, PHPDoc, and control-flow information to report code that may be incorrect.

### 2. How is static analysis different from a test?

**Answer:** A test runs selected behavior and checks its result; static analysis inspects code paths without running them. **Explanation:** Tests can verify a specific runtime outcome, while analysis can find type mistakes across code that a test may not execute. Use both.

### 3. Name two PHP static analysis tools covered in this chapter.

**Answer:** PHPStan and Psalm. **Explanation:** They share many goals, but have different configuration formats, issue names, and strictness settings.

### 4. Does a successful analyzer run prove the application is bug-free?

**Answer:** No. **Explanation:** Static analysis can only reason from the code and metadata it understands. It does not prove that requirements are correct or that every runtime interaction works.

### 5. Which is an example of a finding a static analyzer may report?

A. A method call on a value that may be null
B. Whether a user likes the page design
C. Whether a remote API will always be available
D. Whether a product rule is fair

**Answer:** A. **Explanation:** Nullable values and invalid method calls can often be detected from types and control flow. The other choices need runtime evidence or human judgment.

## Types and PHPDoc

### 6. What information do native PHP types provide?

**Answer:** They declare parameter, property, and return contracts that PHP can enforce at runtime. **Explanation:** A native type such as int or User describes the broad kind of value allowed.

### 7. What extra detail can PHPDoc express?

**Answer:** Details such as array shapes, lists, generics, and refined string or integer types. **Explanation:** These descriptions give analyzers and developers more information than a broad native array type alone.

### 8. What does array{email: non-empty-string, marketing: bool} describe?

**Answer:** An array with an email key containing a non-empty string and a marketing key containing a boolean. **Explanation:** An array shape describes known keys and the type of each associated value.

### 9. Does writing a PHPDoc array shape validate request data at runtime?

**Answer:** No. **Explanation:** PHPDoc informs tools; it does not inspect actual input. Validate external values in executable code before relying on a precise type.

### 10. When is mixed a reasonable input type?

**Answer:** When a function genuinely receives a value whose type is not yet known, such as decoded or external data. **Explanation:** The function should validate that value and narrow it before using it. Do not use mixed to avoid describing a known contract.

### 11. What does type narrowing mean?

**Answer:** Using checks and branches to reduce a broad type to a more precise type. **Explanation:** After checking that a value is a string and not empty, the code can safely handle it as a non-empty string.

### 12. Which is a better way to handle an unknown array from a request?

A. Trust a PHPDoc comment and use every key
B. Convert the value to a string and continue
C. Validate that it is an array and check required keys and value types
D. Suppress all analyzer findings in the file

**Answer:** C. **Explanation:** Runtime validation establishes facts about untrusted input; annotations then describe the validated result.

## Running and configuring tools

### 13. How can PHPStan and Psalm be added to a Composer project?

**Answer:** Add them as development dependencies with Composer, selecting versions compatible with the project. **Explanation:** Development dependencies keep analysis tools available to the team without making them application runtime dependencies.

### 14. Does a project have to use both PHPStan and Psalm?

**Answer:** No. **Explanation:** Either tool can be used independently. Using both is an optional project choice and requires maintaining both configurations.

### 15. What does a PHPStan rule level control?

**Answer:** The strictness of the checks PHPStan applies. **Explanation:** PHPStan's levels are cumulative; raising the level adds stricter checks. Current PHPStan documentation describes levels 0 through 10.

### 16. Can PHPStan level 6 be treated as Psalm error level 6?

**Answer:** No. **Explanation:** The tools use different rule systems and the numbers do not represent equivalent settings.

### 17. What belongs in an analyzer's project paths?

**Answer:** Source code and, when useful, the project's own tests or scripts. **Explanation:** Third-party dependencies under vendor are normally read as dependency metadata rather than analyzed as code the project owns.

### 18. Why should analyzer configuration be committed?

**Answer:** So developers and CI apply the same paths and strictness settings. **Explanation:** Uncommitted local settings can make analysis results differ across machines.

### 19. What should a team do when upgrading an analyzer?

**Answer:** Review release changes, update the locked dependency deliberately, and address any new findings. **Explanation:** Analyzer versions can change supported PHP versions, diagnostics, or recommended configuration.

## Findings and baselines

### 20. What should you do first when an analyzer reports an error?

**Answer:** Read the full message and inspect the relevant code and inferred type. **Explanation:** The report may reveal a real defect, a missing type description, or an analyzer integration that needs configuration.

### 21. What is an analyzer baseline?

**Answer:** A record of existing findings that lets a project adopt analysis without treating every old issue as new. **Explanation:** A baseline can keep known debt from blocking adoption while CI catches additional findings.

### 22. Is a baseline a permanent fix for the findings it contains?

**Answer:** No. **Explanation:** It is a migration aid. Review it, avoid adding entries casually, and remove entries as issues are fixed.

### 23. How can a team use a baseline responsibly?

**Answer:** Commit it, prevent it from growing without review, and reduce it as existing issues are corrected. **Explanation:** This lets new code meet a standard while old code is improved gradually.

### 24. What is the main risk of adding broad ignore rules?

**Answer:** They can hide real and unrelated defects. **Explanation:** Prefer fixing the cause. If suppression is justified, keep it narrow and explain why it is safe.

### 25. Where should static analysis run?

**Answer:** Locally and in continuous integration. **Explanation:** Local runs help developers fix issues early; CI applies the same checks to every proposed change.

### 26. How can CI prevent new analyzer issues from merging?

**Answer:** Run the analyzers as required checks and fail the job when unignored issues remain. **Explanation:** A non-zero analyzer result can block the change until it is corrected or intentionally handled.

## Practical understanding

### 27. A value might be null before a method call. What should you do?

**Answer:** Check or otherwise establish that the value is non-null before calling the method. **Explanation:** The guard handles the runtime case and lets the analyzer understand that the value is safe in that branch.

### 28. An analyzer says a return value is the wrong type. What are two likely fixes?

**Answer:** Correct the implementation or correct the declared type if the declaration does not match the intended behavior. **Explanation:** The implementation and its native/PHPDoc contracts should describe the same behavior.

### 29. A framework uses magic properties that the analyzer cannot see. What can help?

**Answer:** A supported framework extension, bootstrap, stub, or precise PHPDoc may provide the missing information. **Explanation:** Configure the actual behavior instead of suppressing every related finding.

### 30. Can static analysis replace input validation?

**Answer:** No. **Explanation:** Types and PHPDoc do not transform or validate data received from a request, file, database, or network.

### 31. Can static analysis replace integration tests?

**Answer:** No. **Explanation:** Integration tests exercise real boundaries such as databases and HTTP. Static analysis does not prove those systems communicate correctly at runtime.

### 32. What is a useful way to begin analysis on a legacy codebase?

**Answer:** Analyze code the team owns, fix clear issues, and use a reviewed baseline only for remaining known findings. **Explanation:** Then stop new findings and reduce the baseline over time.

### 33. Why should analyzer commands and versions match between local development and CI?

**Answer:** Consistency makes results reproducible. **Explanation:** A committed lock file and committed configuration help ensure the same tool versions and rules run in both places.

### 34. What is the key difference between a type annotation and a runtime check?

**Answer:** An annotation describes a value to tools; a runtime check examines the actual value while the program runs. **Explanation:** Safe code uses runtime validation for untrusted input and accurate annotations to communicate the result.

### 35. What is the main takeaway from this chapter?

**Answer:** Use precise types, validate unknown input, act on analyzer findings, and run analysis alongside tests. **Explanation:** PHPStan and Psalm improve confidence when their reports are reviewed and their configuration is maintained rather than treated as a guarantee.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [120 - Test doubles, stubs, and mocks](../lessons/120-test-doubles-stubs-and-mocks.md) | [Lesson](../lessons/121-static-analysis-with-phpstan-and-psalm.md) | [Example](../examples/121-static-analysis-with-phpstan-and-psalm.md) | [122 - Formatting, linting, and code quality tools](../lessons/122-formatting-linting-and-code-quality-tools.md) |