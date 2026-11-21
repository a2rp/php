# Secure coding and threat modeling: Questions and answers

## Foundations

### 1. What is secure coding?

**Answer:** Designing, implementing, and maintaining software so that data and actions are protected from misuse. **Explanation:** It combines sound design decisions with practical safeguards and verification.

### 2. What is threat modeling?

**Answer:** A structured way to identify threats to a particular system and decide how to address them. **Explanation:** It connects security concerns to the system's assets, data flows, entry points, and trust boundaries.

### 3. What four questions guide a practical threat-modeling discussion?

**Answer:** What are we building? What could go wrong? What will we do about it? Did we verify the safeguards? **Explanation:** These questions lead from understanding the system to reviewing its risk responses.

### 4. Is a threat model a one-time document?

**Answer:** No. **Explanation:** Update it when the system, data flows, dependencies, permissions, or trust boundaries change.

### 5. What is an asset in a threat model?

**Answer:** Something valuable that needs protection, such as credentials, personal data, service availability, or data integrity. **Explanation:** Identifying assets helps the team focus on meaningful consequences.

### 6. What is a trust boundary?

**Answer:** A point where data or control passes between components with different levels of trust or privilege. **Explanation:** A browser-to-server request and an application-to-database connection are examples worth reviewing.

### 7. Name three common application entry points.

**Answer:** Routes, forms, and API endpoints. **Explanation:** Cookies, headers, uploads, background jobs, command-line arguments, and integrations are also possible entry points.

## STRIDE

### 8. What does STRIDE help a team do?

**Answer:** Remember broad categories of threats while reviewing components and data flows. **Explanation:** It is a set of prompts, not proof that every relevant threat has been found.

### 9. Which STRIDE category asks whether an attacker can pretend to be another user?

**Answer:** Spoofing. **Explanation:** Spoofing threatens authentication and identity claims.

### 10. Which STRIDE category concerns unauthorized changes to data or commands?

**Answer:** Tampering. **Explanation:** Tampering threatens integrity.

### 11. Which STRIDE category asks whether an action can happen without an adequate audit record?

**Answer:** Repudiation. **Explanation:** The concern is accountability and whether important actions can be investigated.

### 12. Which STRIDE category concerns exposure of private information?

**Answer:** Information disclosure. **Explanation:** Consider unauthorized reads, accidental public output, and sensitive details in logs.

### 13. Which STRIDE category concerns making a service unavailable?

**Answer:** Denial of service. **Explanation:** Repeated or expensive requests and unbounded resource use can affect availability.

### 14. Which STRIDE category concerns a user gaining access beyond their permissions?

**Answer:** Elevation of privilege. **Explanation:** For example, changing a request so that a regular user can perform an administrator-only action.

### 15. Which is a useful STRIDE exercise for a profile-editing endpoint?

A. Ask only whether the form looks correct
B. Check whether a user can change another user's profile by changing an identifier
C. Assume a hidden edit button blocks unauthorized requests
D. Ignore the database because it is behind the application

**Answer:** B. **Explanation:** A client-controlled object identifier can reveal an authorization failure if the server does not check ownership.

## Risk and safeguards

### 16. What should a threat record include?

**Answer:** The asset, affected component, attack scenario, impact, existing controls, response, owner, and verification method. **Explanation:** Recording these details turns a concern into work that can be prioritized and checked.

### 17. How should a team rank threats?

**Answer:** Use a consistent method that considers likelihood, impact, exposure, and existing controls. **Explanation:** A risk score helps prioritize work but does not prove a low-ranked threat is impossible.

### 18. What does mitigate mean as a risk response?

**Answer:** Reduce the likelihood or impact of a threat with safeguards. **Explanation:** Parameterized SQL is one mitigation for SQL injection risk.

### 19. What does eliminate mean as a risk response?

**Answer:** Remove the feature or design choice that creates the threat. **Explanation:** Removing an unnecessary public upload feature can eliminate its associated upload attack surface.

### 20. Does transferring responsibility to a vendor eliminate all application risk?

**Answer:** No. **Explanation:** A contract or service arrangement can assign some responsibilities, but the application still has its own controls and residual risks to manage.

### 21. Who should accept a significant residual security risk?

**Answer:** An accountable owner with the authority to make that decision. **Explanation:** Record the reason, impact, assumptions, and any remaining safeguards.

### 22. Why should mitigations be written as testable requirements?

**Answer:** So reviewers and tests can check whether the safeguard was implemented. **Explanation:** “Make profiles secure” is vague; “a profile update can change only the signed-in user's record” is verifiable.

## Secure coding

### 23. Is client-side validation sufficient?

**Answer:** No. **Explanation:** A client can be modified or bypassed. Validate values on the server at the boundary where the application receives them.

### 24. What is the difference between validation and output encoding?

**Answer:** Validation checks whether data is acceptable for a field; output encoding makes data safe for a specific rendering context. **Explanation:** Valid text can still need HTML encoding before it is rendered.

### 25. How should values be added to SQL queries?

**Answer:** Use parameterized queries for data values. **Explanation:** Keep query structure separate from values instead of concatenating untrusted text into SQL.

### 26. Can a database parameter stand in for a table name or sort direction?

**Answer:** Usually not. **Explanation:** Map structural choices such as column names or ASC/DESC to a fixed application-owned allow-list.

### 27. Why is escaping not a substitute for parameterized SQL?

**Answer:** Manual escaping is easy to apply incorrectly and depends on context. **Explanation:** Parameterized queries keep values separate from query syntax.

### 28. What is the difference between authentication and authorization?

**Answer:** Authentication establishes who a user is; authorization determines what that user may do. **Explanation:** A signed-in user must still be checked against the requested action and object.

### 29. Why must authorization be checked on the server?

**Answer:** Browser controls such as hidden buttons can be bypassed. **Explanation:** Enforce access on each protected server-side operation, including reads, writes, and exports.

### 30. What is an object-level authorization failure?

**Answer:** A user can access or change an object they do not own or have permission to use. **Explanation:** Check access to the specific record, not just whether the user is signed in.

### 31. What does least privilege mean?

**Answer:** Give a user, process, or service only the permissions it needs. **Explanation:** Limiting permissions reduces the harm if an account or component is compromised.

### 32. What does fail closed mean for a protected operation?

**Answer:** Deny access when the authorization decision cannot be made safely. **Explanation:** A missing role, failed permission lookup, or unexpected state should not silently grant access.

### 33. What information should not be written to application logs?

**Answer:** Passwords, session tokens, secrets, and unnecessary personal data. **Explanation:** Logs need access controls too; avoid turning them into a second store of sensitive information.

### 34. Why should error responses avoid detailed internals?

**Answer:** Detailed traces, file paths, and configuration values can disclose information useful to an attacker. **Explanation:** Give users a limited message and keep protected diagnostic detail in appropriate logs.

### 35. How can a team reduce denial-of-service risk in ordinary application code?

**Answer:** Limit request sizes and expensive work, use timeouts and pagination, and apply suitable rate controls. **Explanation:** Threat models should include repeated or oversized requests, not only data theft.

### 36. What is the main takeaway from this chapter?

**Answer:** Map the system, identify plausible threats, choose specific safeguards, and verify them. **Explanation:** Secure coding is an ongoing design and development practice, not a single checklist or tool.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [122 - Formatting, linting, and code quality tools](../lessons/122-formatting-linting-and-code-quality-tools.md) | [Lesson](../lessons/123-secure-coding-and-threat-modeling.md) | [Example](../examples/123-secure-coding-and-threat-modeling.md) | [124 - Preventing SQL injection](../lessons/124-preventing-sql-injection.md) |