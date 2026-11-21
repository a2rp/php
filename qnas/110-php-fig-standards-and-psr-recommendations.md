# 110 - PHP-FIG standards and PSR recommendations: Q&A

## Multiple-choice questions

### 1. What is PHP-FIG's main purpose?

- **A.** Help PHP projects discuss common standards and interoperability
- **B.** Define the PHP language and release its interpreter
- **C.** Require every PHP application to use the same framework
- **D.** Publish every PHP library itself

**Answer: A.** PHP-FIG brings project representatives together to discuss common conventions and interfaces. **Explanation:** It does not define PHP itself or prescribe one framework for all projects.

### 2. What does PSR stand for?

- **A.** PHP Standard Recommendation
- **B.** Package Source Registry
- **C.** PHP Script Runtime
- **D.** Project Structure Rule

**Answer: A.** PSR means PHP Standard Recommendation. **Explanation:** A PSR documents a shared contract or convention that projects may choose to adopt.

### 3. Does an accepted PSR automatically become mandatory for every PHP project?

- **A.** No; each project decides which standards to adopt
- **B.** Yes; PHP rejects applications that do not follow it
- **C.** Yes, but only for command-line scripts
- **D.** No; accepted PSRs cannot be used by applications

**Answer: A.** Acceptance means the group accepted the specification, not that every project must use it. **Explanation:** Projects choose standards based on interoperability, dependencies, and their own compatibility needs.

### 4. Which PSR defines a shared logger interface?

- **A.** PSR-3
- **B.** PSR-4
- **C.** PSR-11
- **D.** PSR-18

**Answer: A.** PSR-3 defines a logger interface and log-level methods. **Explanation:** A concrete logging package supplies behavior such as writing messages to a file or service.

### 5. Which PSR covers namespace-to-file autoloading?

- **A.** PSR-4
- **B.** PSR-7
- **C.** PSR-14
- **D.** PSR-20

**Answer: A.** PSR-4 maps namespace prefixes to directories and class files. **Explanation:** Composer can generate a loader that follows this mapping.

### 6. Which PSR describes HTTP request and response message interfaces?

- **A.** PSR-7
- **B.** PSR-3
- **C.** PSR-11
- **D.** PSR-16

**Answer: A.** PSR-7 describes HTTP message interfaces such as requests, responses, URIs, and streams. **Explanation:** Other HTTP PSRs address related layers such as handlers, factories, and clients.

### 7. What is PSR-11 about?

- **A.** A container interface for retrieving entries
- **B.** HTTP message immutability
- **C.** Coding style for PHP files
- **D.** Log message severity

**Answer: A.** PSR-11 standardizes a small container lookup contract. **Explanation:** It does not define every registration or automatic dependency-injection feature of a concrete container.

### 8. What is the relationship between PSR-12 and PSR-2?

- **A.** PSR-12 superseded PSR-2; PSR-2 is deprecated
- **B.** PSR-2 superseded PSR-12
- **C.** They are both HTTP client interfaces
- **D.** PSR-12 is a draft version of PSR-4

**Answer: A.** PSR-12 extended the coding-style guidance and superseded PSR-2. **Explanation:** PHP-FIG now also publishes PER Coding Style as newer guidance beyond PSR-12.

### 9. Which coding-style resource extends and replaces PSR-12 in the current status snapshot used by this chapter?

- **A.** PER Coding Style 3.1
- **B.** PSR-0
- **C.** PSR-9
- **D.** PSR-16

**Answer: A.** PER Coding Style 3.1 is the newer PHP-FIG style guide in this chapter's status snapshot. **Explanation:** Projects should confirm the current guide and explicitly document the version they adopt.

### 10. Which PSR concerns server request handlers and middleware?

- **A.** PSR-15
- **B.** PSR-6
- **C.** PSR-12
- **D.** PSR-20

**Answer: A.** PSR-15 defines HTTP server request handler and middleware interfaces. **Explanation:** PSR-7 covers messages, while PSR-17 and PSR-18 cover factories and client behavior.

### 11. Which PSR standardizes an interface for sending HTTP requests as a client?

- **A.** PSR-18
- **B.** PSR-14
- **C.** PSR-1
- **D.** PSR-13

**Answer: A.** PSR-18 defines an HTTP client interface. **Explanation:** A concrete client package performs the network request.

### 12. What does PSR-20 provide?

- **A.** A clock interface for obtaining the current time
- **B.** A standard database schema
- **C.** A package publishing service
- **D.** A logging formatter

**Answer: A.** PSR-20 defines a clock contract. **Explanation:** A service can depend on a clock and tests can provide a controlled implementation when time matters.

## Short-answer questions

### 13. What is the difference between a PSR interface and an implementation package?

**Answer:** A PSR interface defines a contract; an implementation package provides concrete behavior that fulfills the contract. **Explanation:** A project chooses and configures an implementation separately.

### 14. What is one purpose of PSR-1?

**Answer:** It defines basic coding conventions such as class and method naming and baseline PHP source rules. **Explanation:** It is a foundation rather than a complete style guide for every formatting choice.

### 15. What does PSR-6 standardize?

**Answer:** A cache-pool and cache-item interface contract. **Explanation:** Concrete packages implement storage using a particular backend.

### 16. How does PSR-16 differ in purpose from PSR-6?

**Answer:** PSR-16 offers a simpler cache API, while PSR-6 provides a more detailed pool and item contract. **Explanation:** Select the contract that fits the package boundary; a project does not need to implement both automatically.

### 17. What does PSR-14 standardize?

**Answer:** Event dispatcher interfaces for dispatching events to listeners. **Explanation:** The application still defines the meaning of its events and what listeners do.

### 18. What is PSR-17 for?

**Answer:** Factory interfaces for creating PSR-7 HTTP objects. **Explanation:** It standardizes object creation across compatible implementations.

### 19. What does PSR-13 describe?

**Answer:** Interfaces for representing hypermedia links and link providers. **Explanation:** It provides a shared model for links in hypermedia-oriented responses.

### 20. What does the status “draft” mean for a PSR proposal?

**Answer:** The proposal is still under development and may change. **Explanation:** Avoid treating a draft interface as an unchanged, settled public contract.

### 21. Why is a PSR status snapshot dated in a tutorial?

**Answer:** Proposal statuses and available guidance can change over time. **Explanation:** A dated snapshot tells readers when the listed status was checked and when to verify it again.

### 22. What does PSR-15 standardize?

**Answer:** Interfaces for HTTP server request handlers and middleware. **Explanation:** Middleware and handlers can cooperate when they follow the same request/response contracts.

### 23. What does PSR-18 standardize?

**Answer:** An HTTP client interface for sending requests and receiving responses. **Explanation:** The interface does not itself provide a network implementation.

### 24. Why might a service depend on `Psr\Log\LoggerInterface` rather than constructing a concrete logger?

**Answer:** It lets the service use a shared logging contract without choosing the output implementation itself. **Explanation:** The application can configure a compatible logger while the service remains focused on its work.

## Practical and scenario questions

### 25. A developer says every PHP project must adopt all accepted PSRs. What is the correct response?

**Answer:** Accepted PSRs are available recommendations, but each project chooses which ones to adopt. **Explanation:** PHP-FIG promotes interoperability; it does not require every project to use every interface or convention.

### 26. A codebase uses PSR-2 and a developer wants to replace it with PER Coding Style by reformatting every file in a feature commit. What is a better plan?

**Answer:** Agree on the target guide, configure tooling, and perform a separate, deliberate style migration. **Explanation:** Large formatting-only changes obscure functional changes and create noisy review diffs.

### 27. A library's constructor accepts a specific logger class, but consumers want to use different logging backends. What design could improve interoperability?

**Answer:** Accept a shared logger interface such as `Psr\Log\LoggerInterface` where the abstraction fits. **Explanation:** Multiple implementations can satisfy the contract without changing the service's logging calls.

### 28. A project uses PSR-7 and calls `$response->withHeader('X-Mode', 'safe')` but discards the return value. Why may the header be missing?

**Answer:** PSR-7 `with...` methods return a new message instead of mutating the original. **Explanation:** Assign or return the new response object.

### 29. A team sees PSR-5 in a proposal list and adds its annotations to a stable public API without checking status. What should it verify?

**Answer:** Verify the proposal's current status and specification before promising it as a stable contract. **Explanation:** In this chapter's status snapshot, PSR-5 is draft and may change.

### 30. An application has no need to swap its simple logger, but a developer proposes adding several abstraction layers just because PSR-3 exists. What should the team consider?

**Answer:** Use the interface if it creates a useful interoperability or substitution boundary; otherwise, keep the design simple. **Explanation:** Standards are tools, not a requirement to add ceremony to every application.

### 31. A developer assumes PSR-11 automatically wires every constructor dependency. What is the correction?

**Answer:** PSR-11 defines entry lookup, not the complete registration or auto-wiring behavior of a container. **Explanation:** Those features are supplied by a concrete container or application configuration.

### 32. A package implements PSR-18. Does that automatically mean it also provides PSR-17 factories and PSR-7 message classes?

**Answer:** No. Check which interfaces and packages it actually provides. **Explanation:** Related PSRs define distinct contracts and one implementation does not automatically implement every related standard.

### 33. A test needs a deterministic current time, but production code calls the system clock directly in several places. Which accepted interface could help?

**Answer:** PSR-20's clock interface can provide a replaceable source of current time. **Explanation:** A test can supply a controlled implementation when time affects behavior.

### 34. A team is choosing between PSR-6 and PSR-16 for a cache abstraction. What should guide the decision?

**Answer:** Choose the contract that matches the operations and interoperability needs of the package boundary. **Explanation:** PSR-6 provides a more detailed pool/item API, while PSR-16 is simpler; neither must be added without a real need.

### 35. A new project wants a modern PHP-FIG coding-style guide. What should it do?

**Answer:** Check the current PHP-FIG PER Coding Style and PSR status, select the appropriate guide, and configure tooling to enforce it. **Explanation:** PSR-12 remains an accepted PSR, but the current PER guide extends and replaces it in this chapter's status snapshot.

### 36. A draft PSR becomes accepted after this tutorial was published. What should a maintainer do?

**Answer:** Review the newly accepted specification, assess whether it benefits the project, and update the project's documented standards if adopting it. **Explanation:** Adoption remains a project decision even after a proposal is accepted.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [109 - Version constraints, lock files, and packages](../lessons/109-version-constraints-lockfiles-and-packages.md) | [Lesson](../lessons/110-php-fig-standards-and-psr-recommendations.md) | [Example](../examples/110-php-fig-standards-and-psr-recommendations.md) | [111 - Project structure, namespaces, and application layers](../lessons/111-project-structure-namespaces-and-application-layers.md) |
