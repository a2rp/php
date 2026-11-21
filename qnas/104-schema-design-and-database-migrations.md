# 104 - Schema design and database migrations: Q&A

## Multiple-choice questions

### 1. What does a database schema describe?

- **A.** The structure and rules for stored data
- **B.** Only the rows currently stored
- **C.** The visual layout of a PHP page
- **D.** Only the database server's password

**Answer: A.** A schema describes objects such as tables, columns, data types, keys, constraints, and indexes. **Explanation:** The rows are data stored inside that structure; the schema defines how that data is organized and constrained.

### 2. Which table design best represents an article that can have several tags, while each tag can belong to many articles?

- **A.** Store tag names in a comma-separated article column
- **B.** Add tag_1, tag_2, and tag_3 columns to articles
- **C.** Use an article_tags junction table
- **D.** Duplicate the full article row for each tag

**Answer: C.** A junction table stores one row per article-tag pair. **Explanation:** It models the many-to-many relationship as two one-to-many relationships and lets the database enforce valid references and unique pairs.

### 3. What is the main purpose of a primary key?

- **A.** To make a row uniquely identifiable
- **B.** To encrypt every column
- **C.** To order every query automatically
- **D.** To replace application authorization

**Answer: A.** A primary key uniquely identifies a row and cannot be NULL. **Explanation:** It supports reliable references and updates, but it is not a secret and does not replace authorization checks.

### 4. When should a column generally be declared NOT NULL?

- **A.** When every valid row must have a value for it
- **B.** Whenever its value is a string
- **C.** Only when it has an index
- **D.** When the application never displays it

**Answer: A.** NOT NULL expresses that the value is required for every valid row. **Explanation:** Use NULL only when absence has a meaningful domain interpretation; do not confuse NULL with an empty string or zero.

### 5. Which database feature prevents two rows from using the same email address?

- **A.** A UNIQUE constraint
- **B.** A comment
- **C.** A PHP type declaration
- **D.** An ORDER BY clause

**Answer: A.** A UNIQUE constraint makes the database reject duplicate values for the constrained key. **Explanation:** PHP can check first to provide a friendly message, but concurrent requests can race, so the database must enforce the invariant.

### 6. Which statement about foreign keys is correct?

- **A.** They help ensure that a child row references an existing parent row
- **B.** They automatically encrypt the parent row
- **C.** They are a replacement for primary keys
- **D.** They always delete the parent when a child is deleted

**Answer: A.** A foreign key enforces a relationship between referencing and referenced rows. **Explanation:** The delete or update action is chosen separately; it is not always CASCADE.

### 7. What is a common use for a composite index on (status, created_at)?

- **A.** Queries filtering by status and then using created_at
- **B.** Replacing all table constraints
- **C.** Guaranteeing that created_at is unique
- **D.** Encrypting status values

**Answer: A.** Column order affects which query patterns can benefit from a composite index. **Explanation:** An index beginning with status is not generally interchangeable with one beginning with created_at. Verify query plans.

### 8. Why should indexes not be added to every column?

- **A.** Indexes use storage and add work to writes
- **B.** A database can have only one index
- **C.** Indexes always make every query slower
- **D.** An index makes a column nullable

**Answer: A.** Indexes can speed selected reads but consume space and must be maintained as rows change. **Explanation:** Add indexes for real query patterns and confirm their value with realistic data and query plans.

### 9. What is a database migration?

- **A.** A versioned change that moves a database schema or data from one state to another
- **B.** A PHP page that lists every row
- **C.** A database backup file only
- **D.** A method for escaping SQL values

**Answer: A.** A migration records a deliberate schema or data change in a repeatable, ordered form. **Explanation:** Versioned migrations help development, test, staging, and production environments converge on the same intended database structure.

### 10. What should a developer generally do after a migration has already run in a shared environment but needs correction?

- **A.** Edit the old file so it looks right
- **B.** Add a new migration that corrects the schema
- **C.** Delete the migration history table
- **D.** Ignore the difference

**Answer: B.** Add a new migration to correct a migration that has already been applied. **Explanation:** Editing an applied migration can make new environments produce a different schema than existing ones.

### 11. Why might a schema history row be absent even though some DDL ran?

- **A.** The DDL may have committed separately and a later statement failed
- **B.** A primary key prevents all history inserts
- **C.** MySQL stores migrations only in PHP files
- **D.** DDL always rolls back with the history insert

**Answer: A.** DDL transaction behavior varies, and MySQL implicitly commits around many DDL statements. **Explanation:** Inspect the actual schema and history before retrying; do not assume the entire migration rolled back.

### 12. What is the expand, migrate, contract approach?

- **A.** Add a compatible schema, move code and data gradually, then remove obsolete structure later
- **B.** Drop all tables and recreate them for every release
- **C.** Rename all columns at once while old code is running
- **D.** Skip backups because migrations are versioned

**Answer: A.** Staged changes let old and new application versions coexist during a deployment. **Explanation:** Additions and backfills happen before later cleanup, reducing the chance of breaking an active process.

## Short-answer questions

### 13. What is the difference between a primary key and a unique constraint?

**Answer:** A primary key is the table's chosen row identifier; a unique constraint prevents duplicate values in another key. **Explanation:** A table normally has one primary key but may have multiple unique constraints. A primary key is not nullable, while NULL handling for unique constraints is database-specific.

### 14. What is a foreign key?

**Answer:** A foreign key is a constraint on one or more columns that requires referenced values to exist in a related table. **Explanation:** It prevents orphaned references when the database enforces it. Parent and child column definitions must be compatible, and the database may require or create supporting indexes.

### 15. What does NULL mean, and how is it different from an empty string?

**Answer:** NULL represents an absent, unknown, or not-applicable value according to the data model; an empty string is a present string containing no characters. **Explanation:** They have different comparison and query behavior. Define which state the application means instead of using them interchangeably.

### 16. How should a many-to-many relationship usually be modeled?

**Answer:** Use a junction table with foreign keys to each related table. **Explanation:** Each junction row represents one pair. A composite unique key or primary key on both IDs can prevent duplicate pairs.

### 17. Why should a mutable value such as a person's email usually not be the only primary key?

**Answer:** The value can change, and changing a key may require updating every dependent reference. **Explanation:** A stable surrogate key is often easier to preserve. A unique constraint can still enforce that the email is not duplicated.

### 18. What is the purpose of a CHECK constraint?

**Answer:** A CHECK constraint rejects rows whose values do not satisfy a specified condition. **Explanation:** It can enforce rules such as an allowed range or set of values. Check support and enforcement vary by database and version, so verify before relying on it.

### 19. What should guide index selection?

**Answer:** The application's common filters, joins, ordering, grouping, and observed query plans should guide index selection. **Explanation:** Indexes have storage and write costs. Confirm with representative data instead of creating an index for every column.

### 20. What does a migration history table record?

**Answer:** It records which versioned migrations have been applied, often with a version, description, and timestamp. **Explanation:** It helps the deployment process identify pending changes. It does not prove that every statement in a partially failed migration completed, so inspect the schema after failures.

### 21. Why should an applied migration file not be silently edited?

**Answer:** Existing databases have already applied its old contents, while new databases would apply the edited contents. **Explanation:** That creates inconsistent schema histories. Preserve the historical migration and add a new corrective version.

### 22. Why is a large backfill often split into batches?

**Answer:** Batches can limit long locks, resource spikes, transaction size, and replication lag. **Explanation:** A resumable backfill can track progress and continue after interruption. The correct batch size depends on the database and workload.

### 23. What does a backward-compatible schema change mean?

**Answer:** It means the changed schema can continue to support the application versions that may still be running during deployment. **Explanation:** For example, adding a nullable column is often safer than immediately renaming or dropping a column still used by old code.

### 24. What is the difference between a rollback and a forward fix?

**Answer:** A rollback attempts to reverse a change; a forward fix applies another migration that moves the database into a corrected state. **Explanation:** Reversal may destroy newer data or be impossible after writes occur. Production teams often prefer a planned forward fix when it preserves data more safely.

## Practical scenarios

### 25. An articles table stores tag names as a comma-separated string. What should the design change be?

**Answer:** Create a tags table and an article_tags junction table with foreign keys to articles and tags. **Explanation:** Store one relationship per row, enforce unique article-tag pairs, and query the relationships with joins. A delimited string makes referential integrity and reliable filtering difficult.

### 26. Two requests check that a username is available at the same time, and both try to register it. What guarantees uniqueness?

**Answer:** A UNIQUE constraint on the username column guarantees the database will not store both values. **Explanation:** Application validation can give an early message, but concurrent checks can race. Handle the duplicate-key error as an expected business outcome.

### 27. A production migration adds a required column to a large populated table and fails midway. What should the operator do first?

**Answer:** Stop and inspect the database schema, data, migration history, and application versions before retrying or marking the migration applied. **Explanation:** DDL may already have committed some work. Use a reviewed repair or forward migration based on the observed state, and preserve a backup or recovery point.

### 28. A developer wants to delete an author and set all article author_id values to NULL. What must be true for this behavior?

**Answer:** The foreign key must use an appropriate SET NULL action, and articles.author_id must allow NULL. **Explanation:** The domain must also permit authorless articles. If articles must retain a historical author, use a different deletion or archival policy.

### 29. A new required status column is needed for millions of existing rows while old PHP instances are still serving requests. Suggest a safe rollout.

**Answer:** Add the column in a backward-compatible nullable or defaulted form, deploy compatible code, backfill existing rows in batches, verify the data, and enforce NOT NULL in a later migration. **Explanation:** Staging lets old and new versions coexist and avoids making a large data rewrite part of one disruptive schema statement.

### 30. A query frequently filters by status, sorts by created_at, and limits the result count. What should the developer do before adding an index?

**Answer:** Inspect the actual SQL, representative data, and query plan, then evaluate a composite index such as (status, created_at) against that workload. **Explanation:** Index column order matters, and the database may choose a different plan. Measure the read benefit and write/storage cost.

### 31. A team finds that production has a manual column missing from the migration files. What should happen?

**Answer:** Document and inspect the drift, then add a versioned migration that brings all environments to a reviewed common schema. **Explanation:** Do not hide the difference by editing old migrations or blindly dropping the column. Check its data and application use first.

### 32. A table's foreign key fails to create even though both columns are integers. What should be checked?

**Answer:** Compare the complete column definitions, including size and signedness, storage engines, referenced key/index, and database-specific restrictions. **Explanation:** In MySQL, corresponding fixed-precision integer types must match in size and sign, and the foreign-key relationship requires suitable indexes.

### 33. A developer proposes ON DELETE CASCADE from customers to invoices. What question should be answered before accepting it?

**Answer:** Determine whether invoices are allowed to disappear when a customer is deleted. **Explanation:** Financial or audit records often need retention. If history must remain, restrict deletion or use an archive or anonymization policy instead of cascading away the invoices.

### 34. A migration command reports failure, but the migration history table has no row for that version. Can the operator safely run it again immediately?

**Answer:** No. First inspect which statements took effect and whether any schema or data changes are partial. **Explanation:** The absent history row does not guarantee that DDL rolled back. Repair or complete the migration deliberately before updating the ledger.

### 35. A developer suggests storing a product's price in FLOAT. What is a better design choice?

**Answer:** Use DECIMAL with an intentional precision and scale, or store integer minor units with a currency code. **Explanation:** Floating-point numbers approximate many decimal fractions and can introduce rounding surprises in exact monetary calculations.

### 36. An online ALTER TABLE is expected to be nonblocking, but it waits for a long time. What should be investigated?

**Answer:** Check the specific operation's online DDL support and look for long transactions or metadata locks on the table. **Explanation:** Online DDL still has operation-specific limits and may need metadata locks. Plan and observe production changes instead of assuming every ALTER is instant.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [103 - Using the MySQLi extension](../lessons/103-using-the-mysqli-extension.md) | [Lesson](../lessons/104-schema-design-and-database-migrations.md) | [Example](../examples/104-schema-design-and-database-migrations.md) | [105 - Command-line applications and arguments](../lessons/105-command-line-applications-and-arguments.md) |
