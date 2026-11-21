# 113 - Dependency injection and service design: Q&A

## Multiple-choice questions

### 1. What is dependency injection?

- **A.** Supplying a class with collaborators from outside that class
- **B.** Requiring every class to use a global container
- **C.** Automatically creating a database for each object
- **D.** Replacing all concrete classes with interfaces

**Answer: A.** Dependency injection provides the objects or values a class needs from outside. **Explanation:** The class can focus on its behavior instead of constructing every collaborator itself.

### 2. Which is usually the clearest way to provide a required collaborator?

- **A.** Constructor injection
- **B.** A hidden global variable
- **C.** A string lookup from a container in every method
- **D.** A nullable property that is never checked

**Answer: A.** A constructor makes required dependencies visible when the object is created. **Explanation:** If an object cannot perform its responsibility without a collaborator, it should normally require that collaborator.

### 3. What is a dependency injection container mainly for?

- **A.** Automating object construction and connection according to configuration
- **B.** Replacing all PHP constructors
- **C.** Storing every database row
- **D.** Enforcing the same architecture on all projects

**Answer: A.** A container can manage construction recipes and resolve an object graph. **Explanation:** Manual constructor wiring is also dependency injection and is often enough for a small project.

### 4. What does PSR-11 standardize?

- **A.** A common interface for retrieving and checking container entries
- **B.** One universal autowiring algorithm
- **C.** A mandatory YAML configuration format
- **D.** One lifecycle policy for all services

**Answer: A.** PSR-11 defines common `get()` and `has()` container operations. **Explanation:** It does not define every container's registration, autowiring, configuration, or lifetime behavior.

### 5. What is a service locator in this context?

- **A.** An object used by application code to look up arbitrary dependencies while it runs
- **B.** A repository that stores an application's services in a database
- **C.** A PHP namespace for interfaces
- **D.** A function that maps HTTP paths to controllers

**Answer: A.** A service locator lets a class retrieve dependencies by identifier instead of declaring them directly. **Explanation:** This hides the class's real requirements and is generally discouraged inside application code.

### 6. When is an interface useful?

- **A.** When it defines a meaningful boundary or supports a real alternative
- **B.** Whenever any class is created
- **C.** Only when a container is installed
- **D.** When it hides all dependencies from the constructor

**Answer: A.** A contract is useful when it clarifies ownership, protects a seam, or enables meaningful substitution. **Explanation:** An interface with no useful boundary can add needless indirection.

### 7. Is using `new` always a design problem?

- **A.** No; direct construction is natural for value and domain objects, while replaceable collaborators are often wired at the edge
- **B.** Yes; all object creation must happen in a container
- **C.** Yes; PHP does not support `new` in application code
- **D.** No; every infrastructure service should be constructed inside domain rules

**Answer: A.** `new` is appropriate for many simple objects. **Explanation:** The design concern is hiding the construction of important collaborators inside classes that should receive them.

### 8. What is the composition root?

- **A.** The application boundary where concrete objects are selected and connected
- **B.** The database table containing class names
- **C.** A base class from which every service must inherit
- **D.** A PHP autoloading configuration key

**Answer: A.** The composition root wires implementations together, often in a bootstrap or entry point. **Explanation:** Application services can depend on their collaborators without choosing concrete implementations themselves.

### 9. Which is an example of a side effect?

- **A.** Sending an email or writing to a database
- **B.** Comparing two integers in memory
- **C.** Declaring a namespace
- **D.** Constructing a small immutable value object

**Answer: A.** Side effects change or interact with something outside the local computation. **Explanation:** Keeping such behavior behind explicit collaborators makes it easier to see and configure.

### 10. Why can setter injection be risky for a required dependency?

- **A.** The object can be created before it has the dependency it needs
- **B.** PHP does not allow setter methods
- **C.** Setters always create a database transaction
- **D.** Setter injection automatically disables interfaces

**Answer: A.** Setter injection can leave an object only partly configured. **Explanation:** Constructor injection is usually clearer when the collaborator is required for valid operation.

### 11. What is a fake in testing?

- **A.** A simplified working implementation, such as an in-memory repository
- **B.** A class that always throws an exception
- **C.** Any production service with a different class name
- **D.** A generated Composer file

**Answer: A.** A fake behaves like a lightweight substitute for the dependency. **Explanation:** It can make a focused test independent from an external database or provider.

### 12. How should configuration values usually reach a service?

- **A.** Read and validate them at the application boundary, then pass relevant values or a settings object
- **B.** Have every service read environment variables independently
- **C.** Store them as public global variables
- **D.** Hide them in a container lookup inside every method

**Answer: A.** Centralized configuration makes defaults and deployment differences easier to review. **Explanation:** A service should receive the configuration it needs rather than secretly reading process-wide state.

## Short-answer questions

### 13. What is a dependency?

**Answer:** An object or value that a class needs to perform its responsibility. **Explanation:** A repository or message sender can be a dependency of an application use case.

### 14. What does constructor injection make visible?

**Answer:** The collaborators required to create a usable instance of the class. **Explanation:** Callers and maintainers can see the class's requirements at its construction point.

### 15. How is a container different from dependency injection?

**Answer:** Dependency injection is the practice of supplying dependencies from outside; a container is optional tooling that can automate object construction. **Explanation:** Manually calling constructors at a bootstrap is still dependency injection.

### 16. When might direct dependence on a concrete class be simpler than an interface?

**Answer:** When the dependency is stable, local, and has no meaningful alternate implementation or boundary need. **Explanation:** Interfaces should serve the design, not exist only to increase the number of types.

### 17. Where should concrete implementations usually be selected?

**Answer:** At the composition root or in the dependency injection container's configuration. **Explanation:** Application and domain code can then depend on the behavior they need rather than choosing infrastructure.

### 18. What does PSR-11 not specify?

**Answer:** It does not prescribe a universal registration API, autowiring algorithm, configuration format, or service lifetime policy. **Explanation:** It standardizes basic entry retrieval and existence checks.

### 19. How can an injected recording sender help test a use case?

**Answer:** It can record the recipient and message so a test can inspect what the use case requested without contacting a mail provider. **Explanation:** It replaces the external effect with a small controlled collaborator.

### 20. Why is looking dependencies up inside a service generally discouraged?

**Answer:** The class's actual requirements are hidden from its constructor and errors may appear only when a lookup path runs. **Explanation:** Explicit dependencies make the service easier to understand and configure.

### 21. Give one case where direct `new` construction is appropriate.

**Answer:** Creating a value object or domain object from local input. **Explanation:** These objects often have simple construction and do not represent a replaceable infrastructure collaborator.

### 22. Why should a shared service avoid storing request-specific mutable state?

**Answer:** A later request or worker operation could observe state left behind by an earlier one. **Explanation:** Pass operation-specific data to methods and understand the lifetime of container-managed objects.

### 23. Why should services avoid reading environment variables directly in many places?

**Answer:** It spreads configuration parsing and defaults across the codebase. **Explanation:** Loading and validating configuration at the application boundary makes behavior easier to review and change.

### 24. What might a constructor with many unrelated dependencies indicate?

**Answer:** The class may own too many responsibilities. **Explanation:** Inspect what the class does and split cohesive behavior if warranted instead of hiding dependencies in a container or generic helper.

## Practical and scenario questions

### 25. `SendOrderReceipt` constructs a new SMTP client inside its `send()` method. What change would improve its design?

**Answer:** Inject a receipt-sending collaborator through the constructor and create its concrete SMTP implementation at the composition root. **Explanation:** This makes the external effect visible and lets the use case remain independent of provider setup.

### 26. Sending a receipt is required, but the sender property defaults to `null` and the service silently skips delivery when it is absent. What is the problem?

**Answer:** The service can appear to complete successfully while failing a required behavior. **Explanation:** Require the sender in the constructor or model a genuinely optional action explicitly.

### 27. A use case must be able to send receipts through SMTP in production and record them in a focused test. Is a `ReceiptSender` interface reasonable?

**Answer:** Yes, if it describes the behavior the use case needs and both implementations satisfy that contract. **Explanation:** This is a meaningful external boundary with a useful test alternative.

### 28. A developer creates an interface for every class, including a local immutable value object with one obvious implementation. What should guide the team?

**Answer:** Keep interfaces where they express a useful boundary, substitution, or ownership; remove ceremonial ones that add no benefit. **Explanation:** More abstractions do not automatically make code easier to maintain.

### 29. `SendOrderReceipt` receives `ContainerInterface` and calls `get('orders')` and `get('sender')` internally. What is a better approach?

**Answer:** Inject the specific repository and sender dependencies directly. **Explanation:** This makes the constructor accurately describe what the service needs and avoids service-locator behavior.

### 30. A small CLI tool has only a few services and one entry point. Must it install a dependency injection container?

**Answer:** No. It can wire the objects with explicit constructor calls in its bootstrap. **Explanation:** Add a container when object assembly becomes repetitive enough to justify the extra tool.

### 31. A use case test currently sends a real email. How can the test avoid that side effect?

**Answer:** Supply a fake or recording sender that captures the message instead of contacting the mail provider. **Explanation:** The test can assert the use case's behavior safely and deterministically.

### 32. A service constructor takes a mail sender, SQL connection, cache, PDF renderer, user manager, and unrelated analytics client. What should be reviewed?

**Answer:** Whether the class combines several responsibilities and can be split into cohesive services. **Explanation:** Do not just pass the entire container or bundle unrelated dependencies into a generic helper to hide the design problem.

### 33. A method creates `new OrderNumber($input)` as a small domain value object. Is this automatically a DI violation?

**Answer:** No. **Explanation:** Directly constructing a value object from local data is normal; DI is most useful for collaborators whose behavior or lifecycle is chosen externally.

### 34. Every repository and mail class reads environment variables in its constructor. What would simplify configuration?

**Answer:** Read and validate environment settings in a bootstrap/configuration layer, then pass the needed values or cohesive settings objects to infrastructure constructors. **Explanation:** This avoids duplicated parsing and makes service requirements visible.

### 35. A long-running worker reuses a service that stores the current customer's email in a mutable property. What is the risk?

**Answer:** A later job may see stale data from an earlier job, potentially causing incorrect behavior or a data leak. **Explanation:** Keep request/job-specific values local to an operation and use lifetimes appropriate to the worker.

### 36. A team assumes that implementing PSR-11 automatically gives it autowiring and a YAML service configuration. Is that correct?

**Answer:** No. **Explanation:** PSR-11 standardizes basic container access methods; specific containers define registration, autowiring, configuration, and lifecycle behavior.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [112 - Routing, controllers, and middleware](../lessons/112-routing-controllers-and-middleware.md) | [Lesson](../lessons/113-dependency-injection-and-service-design.md) | [Example](../examples/113-dependency-injection-and-service-design.md) | [114 - Application configuration and logging](../lessons/114-application-configuration-and-logging.md) |
