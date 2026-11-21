# Secure coding and threat modeling

## Security starts with the system, not a checklist

Secure coding is the practice of designing, implementing, and maintaining software so that data and actions are protected from misuse. Threat modeling makes that work specific: it helps a team understand what it is building, what could go wrong, and which safeguards matter for this system.

Threat modeling is not a one-time document or a substitute for implementation review. Revisit it when the application adds a feature, changes a data flow, introduces a dependency, or crosses a new trust boundary.

A practical threat-modeling discussion asks:

1. What are we building and protecting?
2. What could go wrong?
3. What will we do about it?
4. Did we verify that the safeguards work?

## Identify assets, actors, entry points, and boundaries

Start by describing the feature and its valuable assets. Assets may include personal data, credentials, payment records, business operations, service availability, and the integrity of stored information.

Then identify:

- **Actors:** anonymous visitors, signed-in users, administrators, staff, and external services.
- **Entry points:** routes, forms, API endpoints, uploaded files, cookies, headers, background jobs, command-line arguments, and integrations.
- **Data stores:** databases, file storage, caches, logs, and session stores.
- **Trust boundaries:** places where data or control passes between components with different privileges or ownership.

A simple data-flow diagram does not need specialized software. Draw processes, external actors, data stores, and arrows between them. Mark where untrusted input enters and where identity, authorization, validation, encoding, or logging decisions occur.

For a profile-editing feature, a high-level flow might be:

~~~text
Browser
  |
  | profile form and session cookie
  v
PHP route and authorization checks
  |
  | validated values and authenticated user ID
  v
PDO database connection
  |
  v
Users table
~~~

The browser is outside the application's trust boundary. A value received from it remains untrusted even if the normal page form constrains what a user can enter.

## Use STRIDE as a set of threat prompts

STRIDE helps a team remember six categories to consider while reviewing each component and data flow:

| Category | Security property at risk | Example question |
| --- | --- | --- |
| Spoofing | Authentication | Could someone pretend to be another user? |
| Tampering | Integrity | Could someone change data or commands they are not allowed to change? |
| Repudiation | Accountability | Could an important action happen without an adequate audit record? |
| Information disclosure | Confidentiality | Could private data reach an unauthorized person or log? |
| Denial of service | Availability | Could a request consume excessive resources or prevent legitimate use? |
| Elevation of privilege | Authorization | Could a user gain access to another account or an administrative action? |

STRIDE is a prompt, not proof that every threat has been found. Add abuse cases that fit the feature, such as repeating a workflow, skipping a required step, racing two requests, or submitting values that the interface does not normally send.

## Assess and respond to risk

For each plausible threat, record:

- The affected asset and component.
- The entry point or trust boundary involved.
- A realistic attack scenario and its impact.
- Existing controls and remaining gaps.
- The chosen response and the person or team responsible.
- How the mitigation will be verified.

Rank risks using a method the team understands. Consider likelihood and impact, along with exposure and existing controls. The result is a prioritization aid, not a guarantee that a low-ranked threat is impossible.

Common responses include:

- **Mitigate:** reduce the likelihood or impact with a safeguard.
- **Eliminate:** remove the feature or design choice that creates the risk.
- **Transfer:** assign some responsibility by contract or service arrangement, while recording the risk and controls that remain with your application.
- **Accept:** document why the remaining risk is understood and acceptable to the accountable owner.

Turn mitigations into requirements that can be reviewed or tested. “Make profiles secure” is vague. “A profile update changes only the authenticated user's record, even when the request contains another user ID” is specific and testable.

## Secure-coding principles for PHP applications

### Validate data at the server boundary

Client-side validation helps users, but it can be bypassed. Validate on the server that required values exist, have the expected type and shape, and satisfy business limits. Use allow-lists for finite choices such as an action name or sort direction.

Validation is not the same as escaping. The right validation depends on the field's purpose, and legitimate free-form text should not be rejected merely because it contains punctuation.

### Keep data separate from commands

Do not concatenate untrusted values into SQL or operating-system commands. Use parameterized database queries for values. For dynamic structural choices that cannot be parameters, map input to a fixed allow-list owned by the application.

Prefer a library or direct API over starting a shell command. If a system command is unavoidable, avoid constructing a command string from user input and apply narrowly scoped validation.

### Encode output for its destination

Data that is safe to store may still be unsafe to render. Encode values when placing them into HTML or another output context. Use the encoding appropriate to that context, or rely on a well-maintained templating engine with auto-escaping enabled.

### Authorize every protected action

Authentication identifies a user; authorization decides what that user may do. Check access on the server for every protected operation, including reads, updates, exports, and background actions. Do not rely on a hidden button or a user ID supplied by the browser.

Apply authorization to the specific object being accessed. A user allowed to view one record is not automatically allowed to view another record with a different identifier.

### Use safe defaults and least privilege

Grant accounts and services only the permissions they need. Start from deny-by-default for protected actions and explicitly allow authorized behavior. Keep production secrets outside source control and avoid exposing detailed errors to clients.

### Handle failures without leaking sensitive data

Return a useful but limited message to the user. Keep diagnostic details in appropriately protected logs, and do not log passwords, session tokens, secrets, or unnecessary personal data. Logging should support investigation without creating another sensitive-data store.

### Limit resource use

Apply appropriate request-size limits, timeouts, pagination, and rate controls. Threat models should include abuse of expensive operations, repeated authentication attempts, oversized uploads, and unbounded queries.

## Make safeguards verifiable

A threat model becomes useful when its responses lead to concrete work:

- A code change uses a parameterized query.
- An authorization rule is checked at the server-side boundary.
- A test confirms that one user cannot change another user's record.
- A request limit is configured and monitored.
- An output is encoded in the HTML context where it is rendered.
- A log event records an important security action without recording secrets.

Use code review, automated tests, static analysis, configuration review, and security testing as appropriate. No single method replaces the others.

## Keep the model current

Revisit the model when a change affects:

- Sensitive data or who can access it.
- New endpoints, external services, or data stores.
- Authentication, authorization, sessions, or trust boundaries.
- File handling, command execution, or data rendering.
- Deployment, permissions, or operational responsibilities.

Record assumptions and unresolved risks. A threat model should help the team make decisions, not create a false sense of completeness.

## Key takeaways

- Map assets, actors, entry points, data flows, and trust boundaries.
- Use STRIDE and feature-specific abuse cases to find plausible threats.
- Rank risk and turn decisions into specific, verifiable requirements.
- Validate input, parameterize commands, encode output, and authorize each protected action.
- Use least privilege, safe defaults, careful error handling, and resource limits.
- Update the threat model as the application changes.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [122 - Formatting, linting, and code quality tools](122-formatting-linting-and-code-quality-tools.md) | [Q&A](../qnas/123-secure-coding-and-threat-modeling.md) | [Example](../examples/123-secure-coding-and-threat-modeling.md) | [124 - Preventing SQL injection](124-preventing-sql-injection.md) |