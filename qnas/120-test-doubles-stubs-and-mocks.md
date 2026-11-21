# Test doubles, stubs, and mocks - Q&A

### 1. What is a test double?

**Answer:** A replacement for a collaborator used to control or observe a test scenario. **Explanation:** Stubs, mocks, fakes, spies, and dummies are different roles a test double can play.

### 2. What is the system under test?

**Answer:** The class or behavior whose result the test is checking. **Explanation:** Keep the test focused on that unit rather than re-testing all of its dependencies.

### 3. What does a stub do?

**Answer:** It supplies controlled values or failures to the system under test. **Explanation:** This lets a test exercise a path without calling a real database, clock, or remote service.

### 4. What does a mock do?

**Answer:** It verifies an expected interaction, such as a method call, arguments, or call count. **Explanation:** Use it when communication with a collaborator is part of the behavior being tested.

### 5. How are stubs and mocks different?

**Answer:** A stub controls indirect inputs; a mock observes indirect outputs. **Explanation:** If only the result matters, stub the dependency and assert the result. If a meaningful side effect matters, configure a mock expectation.

### 6. What is a dummy?

**Answer:** A placeholder passed to a method or constructor but not used by the test. **Explanation:** A dummy should not contain behavior the scenario depends on.

### 7. What is a fake?

**Answer:** A simplified working implementation, such as an in-memory repository. **Explanation:** A fake can behave more realistically than a stub, but should not become a second complex implementation.

### 8. What is a spy?

**Answer:** An object that records calls so the test can inspect them later. **Explanation:** A small handwritten spy can help when the test needs to examine a sequence after execution.

### 9. Should a repository integration test mock PDO?

**Answer:** No, if the test is meant to verify database behavior. **Explanation:** Use a real disposable database and driver to check SQL, constraints, and persistence; stub the repository when testing a higher-level service.
### 10. How does a stub help test a rare error path?

**Answer:** Configure a dependency to throw the exception that the system under test must handle. **Explanation:** The test can exercise error handling without making a real service fail.

### 11. Why should a stub return an explicit value for a scenario?

**Answer:** Automatic default values may not represent the case the test intends to model. **Explanation:** Explicit behavior makes the scenario and expected outcome easier to understand.

### 12. When is a callback useful for a stub?

**Answer:** When a small return value depends on the method arguments. **Explanation:** Keep the callback simple; complex business logic belongs in real code with its own tests.

### 13. When is a handwritten fake clearer than a PHPUnit stub?

**Answer:** When several tests need a small, stateful implementation with realistic behavior. **Explanation:** A short in-memory adapter may be easier to read than a large configuration chain.

### 14. Which PHPUnit method creates a stub?

**Answer:** createStub(). **Explanation:** Configure only the return values or exceptions needed for the scenario.

### 15. Which PHPUnit method creates a mock?

**Answer:** createMock(). **Explanation:** Configure expectations when the test must verify the collaborator interaction.

### 16. What does once() require?

**Answer:** Exactly one call to the configured method. **Explanation:** The test fails if the call does not happen or happens more than once.

### 17. What does exactly(2) require?

**Answer:** Exactly two calls. **Explanation:** Use an exact count only when the number of interactions is part of the behavior.

### 18. What does never() verify?

**Answer:** That the configured method is not called. **Explanation:** This is useful for a meaningful forbidden side effect, such as not sending a notification after validation fails.
### 19. What if createMock() is used without configuring an expectation?

**Answer:** Use createStub() when calls do not need verification. **Explanation:** Recent PHPUnit versions report a notice for a mock with no expectations because it is not being used as a mock.

### 20. Should atLeastOnce() be used whenever a call is expected?

**Answer:** Prefer once() or exactly() when the exact count matters. **Explanation:** Broad expectations can hide repeated calls and make the test intent less precise.

### 21. What does with() check on a mock?

**Answer:** It checks that the configured method receives arguments matching the supplied values or constraints. **Explanation:** Use a specific PHPUnit constraint when you need a particular comparison, such as object identity.

### 22. Should every dependency call be verified?

**Answer:** No. **Explanation:** Verify only interactions that represent an important contract or side effect; incidental reads and helper calls make tests brittle.

### 23. Can PHPUnit double a final class?

**Answer:** No, not as an ordinary test double. **Explanation:** Prefer an interface at the boundary or use the real class when it is deterministic and inexpensive.

### 24. Can PHPUnit replace private, final, or static methods in a double?

**Answer:** Not as ordinary doubled methods. **Explanation:** Private and final methods keep their real behavior; static calls are not a useful injectable seam.

### 25. Why are interfaces useful for testability?

**Answer:** They define a stable contract that production adapters and test doubles can both implement. **Explanation:** The unit can receive a controlled dependency through constructor injection.

### 26. Should a value object usually be mocked?

**Answer:** No. **Explanation:** Use the real value object when it is simple and deterministic; a mock can obscure its actual data and behavior.

### 27. Does a mock always need a return-value configuration?

**Answer:** No. Configure a return only when the system under test depends on that result. **Explanation:** For void side effects, an expectation about the call and arguments may be enough.
### 28. When should a test verify that a method is never called?

**Answer:** When avoiding that side effect is important to the scenario. **Explanation:** For example, a rejected payment should not trigger a receipt email.

### 29. Can a mock also provide return values?

**Answer:** Yes. **Explanation:** A mock supports stub behavior too, but createStub() is clearer when no call expectation is needed.

### 30. What might a test double with a long setup script indicate?

**Answer:** The unit may have too many dependencies or responsibilities. **Explanation:** Simplify the double and consider splitting responsibilities at real boundaries.

### 31. How can a test make time-dependent behavior repeatable?

**Answer:** Inject a clock interface and stub it to return a fixed time. **Explanation:** The test no longer depends on the wall clock or waiting.

### 32. Should an HTTP integration test replace the database with a mock?

**Answer:** It depends on the boundary being tested. **Explanation:** A request-pipeline test can stub unrelated external services, but use a real disposable database if persistence is part of the scenario.

### 33. What should replace a global static service call that is hard to control?

**Answer:** An injected interface or adapter around that service. **Explanation:** This creates a clear seam for a stub in unit tests and a real implementation in production.
### 34. What does it mean for a test to be over-mocked?

**Answer:** It replaces many collaborators and verifies incidental implementation details. **Explanation:** Such tests often break during harmless refactors without improving confidence in behavior.

### 35. Why should a PHPUnit double use the API supported by the project's version?

**Answer:** PHPUnit's test-double features and recommendations change across releases. **Explanation:** Check the installed version before using a newer matcher or configuration method.

### 36. What is a simple way to decide between a stub and a mock?

**Answer:** Ask whether the test cares about a value coming in or an interaction going out. **Explanation:** Control input with a stub; verify an important interaction with a mock.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [119 - Integration tests for databases and HTTP](../lessons/119-integration-tests-for-databases-and-http.md) | [Lesson](../lessons/120-test-doubles-stubs-and-mocks.md) | [Example](../examples/120-test-doubles-stubs-and-mocks.md) | [121 - Static analysis with PHPStan and Psalm](../lessons/121-static-analysis-with-phpstan-and-psalm.md) |
