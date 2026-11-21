# 104 - Schema design and database migrations

## Goals

After this lesson, you should be able to:

- Turn application requirements into related tables and columns.
- Choose keys, data types, nullability, constraints, and indexes deliberately.
- Model one-to-many and many-to-many relationships.
- Keep a database schema understandable and consistent.
- Change a schema with ordered, reviewed, repeatable migrations.
- Plan schema changes so application deployments and existing data remain safe.

## What is a database schema?

A database schema describes the structure and rules of stored data. It includes tables, columns, data types, keys, indexes, constraints, and relationships.

An application schema is a contract shared by the database and the application. PHP code depends on column names and meanings; SQL queries depend on relationships and indexes; reports and integrations may depend on the same stored information. A careless schema change can therefore break more than one page.

Design the structure around the data and the operations the application needs. Avoid designing tables only around how one screen currently looks.

## Start with requirements and entities

Before writing CREATE TABLE, write down:

1. What things must the application store?
2. What identifies each thing?
3. Which values are required, optional, unique, or allowed to repeat?
4. How are the things related?
5. Which operations must be fast?
6. How long must records remain, and what does deleting a record mean?

For a small publishing application, possible entities include:

- Authors
- Articles
- Tags
- The relationship between articles and tags

An entity usually becomes a table. Each row represents one instance of that entity. Each column represents one property with one defined meaning.

Do not put unrelated kinds of data into one overloaded table simply to reduce the table count.

## Tables, rows, and columns

A table should represent one kind of thing. For example, an articles table should contain article properties, not a comma-separated list of tag names.

A column should have a clear meaning and a compatible data type:

- Use integer types for whole-number identifiers and counts.
- Use DECIMAL for exact decimal quantities such as prices; floating-point types approximate values.
- Use a suitable character type for text with known limits.
- Use TEXT when a value can be large and does not need normal short-string indexing.
- Use DATE for a calendar date without a time.
- Use a date-time type for a point in time, with one documented time-zone policy.
- Use a binary type only when the application intentionally stores binary data in the database.

For money, decide the currency and precision model. DECIMAL is suitable for exact decimal arithmetic. Another common design stores integer minor units, such as cents, alongside an explicit currency code. Do not use a floating-point value for accounting amounts.

Choose a maximum length from real requirements, not guesswork. A VARCHAR limit is a database constraint on stored characters, not a substitute for application validation or a product decision about how much text is acceptable.

## Primary keys

A primary key uniquely identifies a row. It must be unique and not null.

A surrogate key such as an auto-incrementing integer is often a practical primary key for an application table. It is stable even if a person's email address, product name, or other business value changes.

A natural key uses a value that has meaning outside the database, such as a government-issued identifier or a globally unique code. Natural keys can be useful when the value is truly stable and guaranteed unique, but changes to those values can affect every reference to them.

Common guidance:

- Give each table one deliberate primary key.
- Keep the primary key stable.
- Avoid using a mutable display value as a key.
- Use an appropriately sized integer for expected growth.
- Do not expose the assumption that IDs are secret or impossible to guess. Authorization must still be checked.

A primary key automatically creates an index in common relational database systems.

## Null, defaults, and required values

NULL means the value is unknown, missing, or not applicable according to the application's model. It is not the same as an empty string, zero, or false.

Use NOT NULL when every valid row must have a value. Use NULL only when the absence of a value has a clear meaning.

A DEFAULT supplies a value when an INSERT omits the column. It does not make a wrong or missing business decision correct. Pick defaults that represent real domain behavior.

For example:

- A published timestamp may be NULL while an article is a draft.
- A quantity may be NOT NULL with a default only if zero is genuinely the intended starting value.
- A user's middle name may be NULL if it is optional.
- A required title should usually be NOT NULL and validated before insertion.

Think through the difference among these values before choosing a design:

| Stored value | Possible meaning |
| --- | --- |
| NULL | No value is known, supplied, or applicable |
| Empty string | A value is present and intentionally has zero characters |
| Zero | A numeric value of zero |
| False | A known negative Boolean value |

Do not use one value to represent several different states. If the application needs more states, model them explicitly.

## Constraints protect data

A constraint asks the database to reject invalid data, even when a write comes from a script other than the main PHP request handler.

Useful constraints include:

- PRIMARY KEY: identifies rows.
- UNIQUE: prevents duplicate values or combinations.
- NOT NULL: requires a value.
- FOREIGN KEY: requires a related parent row to exist.
- CHECK: restricts values to a condition supported by the selected database and version.

Put important invariants in the database as well as validating user input in PHP. Database constraints are not a replacement for friendly validation messages; they are a final integrity boundary.

For MySQL, CHECK constraints are enforced in supported modern releases beginning with MySQL 8.0.16. Confirm the database version and behavior before depending on them in older installations.

Name constraints clearly where the database supports named constraints. Clear names make schema inspection and error diagnosis easier.

## Model relationships

### One-to-many

One author can write many articles, and each article can belong to one author. Store the author's key on the many side:

~~~text
authors (one) -------- (many) articles
                           author_id
~~~

The articles.author_id column is a foreign key to authors.id. This avoids copying an author's name into every article and lets the database check that referenced authors exist.

### Many-to-many

An article can have many tags, and a tag can describe many articles. Use a junction table:

~~~text
articles (one) --- (many) article_tags (many) --- (one) tags
~~~

The junction table contains one row per article-tag pairing. A composite primary key on (article_id, tag_id) prevents the same tag being attached to the same article twice. Add the reverse-order index only if queries that start with a tag need it.

Do not save a collection as a comma-separated string in one column. It makes validation, joins, uniqueness, and updates harder.

### One-to-one

A one-to-one relationship is often represented by a foreign key with a UNIQUE constraint on the referencing column. Before adding a separate table, ask whether the data has a different lifecycle, privacy boundary, or access pattern that makes separation useful.

## Referential actions and deletion

Foreign-key actions describe what happens when a referenced row is updated or deleted. Common options include RESTRICT, CASCADE, and SET NULL, but support and behavior vary by database.

Choose deliberately:

- RESTRICT prevents deletion while dependent records exist.
- CASCADE deletes or updates dependent rows automatically.
- SET NULL clears the reference, so the foreign-key column must allow NULL.

A cascade can be useful for junction rows that have no meaning without their parent. It can be dangerous for records that must be retained for audit, billing, legal, or historical reasons. Prefer an explicit archive or soft-delete policy when the application must retain data. A soft delete is still a domain rule; it does not automatically enforce referential integrity for every query.

Make sure foreign-key column types match their referenced columns. For example, signed and unsigned integer differences can prevent a foreign-key relationship in MySQL. Both tables must use a storage engine that enforces foreign keys.

## Normalization and avoiding duplicate facts

Normalization is a method for reducing duplicated facts and update anomalies.

Suppose an article table repeats the author's email on every article row. When the author changes their email, many rows must be updated, and some may be missed. Store author details once in authors and refer to that row by key.

A practical beginner model:

- Keep one value per column; avoid repeating groups such as tag_1, tag_2, tag_3.
- Store each fact in one authoritative place when possible.
- Use foreign keys to represent relationships.
- Use junction tables for many-to-many relationships.
- Prefer a clear normalized design before considering duplicated or precomputed data.

Normalization is not a goal to pursue blindly. Some systems intentionally store derived values for performance or historical snapshots. Such duplication needs a clear source of truth, update policy, and reason measured against real usage.

## Indexes

An index helps the database locate rows or satisfy ordering without examining every row. Indexes also take disk space and add work to INSERT, UPDATE, and DELETE operations.

Useful index types and purposes:

- Primary-key index: locates a row by its identity.
- Unique index: prevents duplicate values and can speed lookups.
- Single-column index: supports common filters or joins.
- Composite index: supports queries involving multiple columns in a specific order.

Composite index order matters. An index on (status, created_at) is generally useful for filtering by status and then ordering or filtering by created_at. It is not generally interchangeable with an index on (created_at, status). Confirm the behavior with the database's query plan tools and real query patterns.

Consider indexing columns used for:

- Frequent selective WHERE conditions.
- JOIN conditions.
- ORDER BY or GROUP BY patterns that matter.
- Foreign-key relationships, as required by the database engine.

Avoid adding an index to every column. Low-selectivity indexes may not help a query, and extra indexes slow writes and increase storage. Measure using realistic data and query plans.

## Naming and maintainability

Choose a consistent naming convention for tables, columns, keys, and constraints. Common choices include lowercase snake_case names such as article_tags and published_at.

Good names are:

- Specific enough to explain the value.
- Consistent across related tables.
- Free of unnecessary abbreviations.
- Not dependent on a vendor-specific reserved word.

Use id only when the table context makes it clear. In a child table, use names such as author_id rather than an ambiguous author. Keep units in names when they prevent mistakes, such as duration_seconds.

Document important semantics that cannot be inferred from the type alone: time-zone policy, currency, status values, deletion behavior, and whether a value is copied as a historical snapshot.

## Database migrations

A migration is a versioned change to a database schema or, when necessary, its data. A migration describes how to move a database from one known state to another.

A migration system typically:

1. Keeps migration files in version control.
2. Gives each migration a unique ordered version.
3. Records successfully applied versions in a schema history table.
4. Applies each pending migration once, in order.
5. Reports failures and prevents an unsafe blind retry.

A migration history table may store a version, description, and applied timestamp. It is different from the application data tables.

### Why migrations matter

Without migrations, each developer or server can end up with a different schema. A migration makes the intended change reviewable and repeatable across development, testing, staging, and production.

A migration should be small and focused. Prefer one logical schema change per version. Include data transformations only when the version needs them, and make their cost and failure behavior clear.

### Migration rules

- Commit migration files with the application code.
- Do not edit or delete a migration that has already run in a shared environment.
- Add a new migration to correct a previous migration.
- Apply pending migrations in order.
- Keep a backup and a tested restore plan for important production data.
- Review the SQL and estimate its runtime, locks, disk use, and data impact.
- Do not assume that a failed migration left the database unchanged.
- Never mark a migration successful if only some required statements ran.
- Do not make undocumented manual schema edits in production.
- Make the deployment process report which version is installed and which version is expected.

A migration can fail after part of its work has already happened. Some database engines support transactional DDL for particular operations; others auto-commit or have operation-specific behavior. MySQL, for example, implicitly commits around many DDL statements. A migration's schema statements and its history-table insert may therefore not behave as one rollbackable transaction. If a command fails, inspect the database state and migration log before deciding whether to repair, complete, or replace the migration.

### Safe schema changes across application deployments

A database change and an application release are separate operations. During deployment, old and new application code may temporarily run against the same database. Use a rollout that keeps both versions compatible.

A common expand, migrate, contract sequence is:

1. **Expand:** Add a backward-compatible nullable column or new table. Keep old code working.
2. **Deploy compatible code:** Make the new code able to read the old state and, if needed, write both old and new forms.
3. **Backfill:** Populate existing rows in bounded batches. Track progress and make the operation resumable.
4. **Switch reads and writes:** Start using the new field after verifying the data.
5. **Contract:** In a later deployment, remove the old field or make the new field stricter after old code no longer depends on it.

For a large table, one huge UPDATE can hold locks, consume resources, and create replication lag. Plan a bounded backfill, observe the system, and make it safe to resume. For a destructive operation, confirm backups and restore procedures first.

### Rollback and forward fixes

Rolling back application code does not automatically roll back the database schema or the data transformation. A down migration can be unsafe if it discards data or if newer application code has already written values that the old schema cannot store.

Plan a rollback strategy before deployment:

- Can the old application version run with the new schema?
- Would reversing the migration destroy user data?
- Can the data be reconstructed?
- Is a forward fix safer than a destructive rollback?
- Has the restore process been tested?

For many production changes, a carefully prepared forward migration is safer than trying to reverse a partially deployed change.

## PHP application responsibilities

PHP code should validate incoming values, use prepared statements for query values, handle constraint violations, and present useful errors without exposing database internals.

The database should enforce structural rules such as uniqueness and valid references. Application validation can give a helpful message before an insert, but two requests can race; a UNIQUE constraint is still needed to guarantee uniqueness.

Do not treat a database error as proof that a user supplied invalid data. Errors can come from connectivity, permissions, missing migrations, disk capacity, deadlocks, or a programming mistake. Log diagnostic details securely and show a safe response.

Keep migration credentials separate from the application's runtime account. The runtime account usually needs only the data permissions required by the application; schema changes need additional privileges and should run in a controlled deployment step.

## Common mistakes

### Storing lists in one column

Comma-separated values make relationships and constraints difficult. Use a junction table for a many-to-many relationship.

### Repeating a related fact in every row

Duplicating an author's email or category name creates update anomalies. Store the fact once and reference it by key unless a documented snapshot is required.

### Using NULL without defining its meaning

Decide whether absence means unknown, not supplied, or not applicable. Do not conflate NULL with an empty string or zero.

### Cascading deletes without considering retention

CASCADE can remove more data than intended. Use it only when dependent rows should truly cease to exist with their parent.

### Indexing every column

Indexes have write and storage costs. Add them to support measured access patterns and verify query plans.

### Treating migrations as ordinary transactions

DDL rollback behavior differs by database and statement. Verify the engine's behavior and inspect partial changes after failure.

### Rewriting an applied migration

Changing a historical migration makes fresh databases differ from databases where the original version already ran. Add a new corrective migration instead.

### Adding a required column to populated data in one step

Existing rows need a valid value. Use a staged nullable column, backfill, and later constraint change when appropriate.

### Removing a column in the same release that stops using it

Old application processes may still need the column. Deploy the code change first, then remove the column in a later migration.

## Key takeaways

- A schema is a long-lived contract between the database and every application that uses it.
- Model entities as tables and relationships with foreign keys and junction tables.
- Choose primary keys, nullability, data types, defaults, and constraints from clear requirements.
- Use database constraints as a final integrity boundary alongside PHP validation.
- Design indexes around actual query patterns and measure their effect.
- Keep schema changes as ordered, version-controlled migrations.
- Treat a failed migration as potentially partially applied until the actual database state is inspected.
- Roll out risky changes in backward-compatible stages.
- Plan backups, lock impact, data migration, and rollback or forward repair before production changes.
- Use least-privilege database credentials; keep schema-change privileges out of normal request handling.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [103 - Using the MySQLi extension](103-using-the-mysqli-extension.md) | [Q&A](../qnas/104-schema-design-and-database-migrations.md) | [Example](../examples/104-schema-design-and-database-migrations.md) | [105 - Command-line applications and arguments](105-command-line-applications-and-arguments.md) |
