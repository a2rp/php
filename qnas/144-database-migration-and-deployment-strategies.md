# Database migration and deployment strategies — Q&A

Use these questions to review schema changes, PHP releases, and database recovery planning. Answers and explanations appear immediately after each question.

## Multiple-choice questions

### 1. What is a database migration?

- A. A one-time manual edit that is not recorded.
- B. A versioned change that moves a database structure or data from one known state to another.
- C. A PHP page that runs whenever a visitor opens the home page.
- D. A backup file.
**Answer: B.** A migration is a tracked, reviewable database change. Recording which migrations ran helps keep environments consistent.

### 2. Which statement best describes seed data?

- A. It is always production customer data.
- B. It is initial or known data commonly used to prepare development and test environments.
- C. It is a schema rollback.
- D. It is a database password.
**Answer: B.** Schema migrations change structure, data migrations transform existing records, and seed data creates known initial records. Development fixtures should not be run casually against production records.

### 3. Where should migrations normally run?

- A. From every public web request.
- B. Once through a controlled deployment job.
- C. From every autoscaled PHP worker at startup.
- D. From a visitor's browser.
**Answer: B.** A single controlled runner avoids many application instances racing to change the same database. Migration tools can also use a history table and locking.

### 4. What should you do when an already-applied migration was incorrect?

- A. Rewrite the migration file and assume all databases will update.
- B. Delete its history record without inspecting the schema.
- C. Add a new corrective migration that moves the database from its actual state to the intended state.
- D. Ask every web worker to rerun it.
**Answer: C.** Applied migration files are part of the shared history. Editing them later makes the source file disagree with databases where the original already ran.

### 5. In an expand-contract deployment, when should an old column usually be removed?

- A. In the same release that introduces the replacement.
- B. After the application no longer needs it, old processes are gone, and the rollback window is considered.
- C. Before the new application is deployed.
- D. Before a backup is made.
**Answer: B.** Keeping both structures during the transition gives old and new code time to coexist safely. Removing the old structure later is the contract step.

## Short-answer questions

### 6. Why can a rolling deployment have both old and new PHP code using the database at once?

**Answer:** Instances update gradually, while some web workers, queue consumers, scheduled jobs, or long-running processes can continue using the previous release. The intermediate schema must remain compatible with every active process.

### 7. List the main steps in an expand-contract release.

**Answer:**

1. Expand the schema with an additive change.
2. Deploy code compatible with both old and new structures.
3. Drain or restart old processes.
4. Backfill existing data in controlled batches.
5. Verify the data and application behavior.
6. Switch reads or writes to the new representation.
7. Remove the old structure in a later release after compatibility and rollback needs end.

**Explanation:** Separating the steps avoids requiring every application instance and database object to change at exactly the same instant.

### 8. Why is one large backfill update risky on a busy table?

**Answer:** It can hold locks, create a large transaction, generate substantial database logs, consume resources, and delay ordinary requests. Bounded batches make progress observable and let operators pause or resume the work.

### 9. What makes a backfill resumable and safe to repeat?

**Answer:** It selects work deterministically, changes only records that still need the transformation, commits bounded batches, records safe progress, and can continue after interruption without damaging already-correct rows.

### 10. Does a PDO transaction guarantee that every DDL migration can be rolled back?

**Answer:** No. DDL transaction behavior varies by database. Some engines implicitly commit certain schema changes, and some operations have engine-specific locking behavior. Check the actual database's documentation and test the exact operation.

### 11. What is the difference between a code rollback and a database restore?

**Answer:** A code rollback deploys an earlier application version. A database restore replaces or rewinds database state to a recovery point and may discard writes made after that point. A schema rollback is yet another operation and may be impossible or destructive after data has changed.

### 12. Why is a backup not enough by itself?

**Answer:** A backup is useful only if it can be restored within the required time and the restored application and data can be validated. Rehearse restoration before an incident.

### 13. Why should each applied migration file remain unchanged?

**Answer:** Applied files form part of the environment's recorded history. Changing one after it has run can make a fresh installation behave differently from an existing production database. Correct mistakes with a new migration.

### 14. What should happen if a migration job fails halfway through?

**Answer:** Stop and inspect the migration history and actual schema or data state before retrying. Determine whether the operation is transactional, partially applied, resumable, or needs a corrective migration. Do not blindly delete history or rerun destructive SQL.

## Deployment scenarios

### 15. A new feature needs a display_name column while old PHP instances are still running. What is a safer rollout?

**Answer:** First add the column in a backward-compatible way, such as allowing nulls when appropriate. Deploy code that can work with both representations, drain old workers, backfill existing rows in batches, verify completeness, and only then switch reads. Remove the old column in a later release.

### 16. A blue-green deployment switches web traffic to a new PHP release. Does that automatically make a schema migration safe?

**Answer:** No. Both application environments may use the same database during the transition. The schema must support both versions, and queue workers or scheduled jobs must be considered separately. Switching traffic does not reverse a migration.

### 17. A team wants to run a migration from every PHP process when the application starts. What is the problem?

**Answer:** Multiple processes can attempt the same change concurrently, block one another, or leave inconsistent state. Run migrations once in a controlled deployment step; use the migration tool's locking support where available.

### 18. A table alteration is described as “online.” Can you assume it never blocks requests?

**Answer:** No. Locking, table scans, and metadata-lock behavior depend on the database engine, version, operation, and table definition. Test the exact operation on representative data and plan for its resource and availability impact.

### 19. A release fails after the schema has expanded but before the old column is removed. What may be the safest first response?

**Answer:** If the previous application version remains compatible with the expanded schema, it may be possible to roll back the code while leaving the additive schema in place. Confirm the actual database state and compatibility first. Do not automatically run a destructive down migration.

### 20. What should a production migration review include?

**Answer:** Review the SQL and data impact, database version and locking behavior, realistic runtime and volume, backup and restore plan, migration runner and concurrency controls, old/new application compatibility, backfill and monitoring plan, and recovery choices.

### 21. Why should the deployment plan account for queue workers and scheduled jobs?

**Answer:** These processes can keep old code alive after web traffic has moved to a new release. They may read or write the database using assumptions that no longer match the schema. Drain, restart, or otherwise coordinate them in the release plan.

### 22. Which checks are useful before and after a migration?

**Answer:** Beforehand, test fresh installation, upgrade from the supported previous schema, realistic data volume, application-version compatibility, failure recovery, and restore procedures. Afterward, verify schema history, data completeness, key application paths, background jobs, errors, locks, and database load.

## Practical exercise

### 23. Put these actions in a safer order: remove the old column, add a new nullable column, deploy compatibility code, backfill data, switch reads to the new column.

**Answer:**

1. Add the new nullable column.
2. Deploy compatibility code that can handle both columns and writes the needed values.
3. Drain old code that cannot write the new column.
4. Backfill existing data and verify it.
5. Switch reads to the new column while preserving rollback compatibility.
6. Remove the old column in a later release after its use and rollback need have ended.

**Explanation:** The intermediate releases maintain compatibility while code, schema, and existing data transition at different times.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [143 - Production configuration, permissions, and logging](../lessons/143-production-configuration-permissions-and-logging.md) | [Lesson](../lessons/144-database-migration-and-deployment-strategies.md) | [Example](../examples/144-database-migration-and-deployment-strategies.md) | [145 - PHP version compatibility and migration guides](../lessons/145-php-version-compatibility-and-migration-guides.md) |