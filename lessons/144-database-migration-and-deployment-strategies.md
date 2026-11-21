# Database migration and deployment strategies

A database schema and the PHP code that uses it must change together safely. A migration is a versioned, reviewable change to a database schema or data. A deployment strategy decides when that change runs, which application versions can run alongside it, and how the team will respond if a release fails.

A migration that works on a small development database can still block requests, consume disk, or lose data in production. Plan migrations as part of the release, test them against realistic data, and make the transition safe for every application instance that may be running.

## Learning goals

After this chapter, you should be able to:

- Explain schema migrations, data migrations, and seed data.
- Keep migration files versioned and record which ones have run.
- Plan the order of database and PHP code changes.
- Use additive changes and expand-contract releases for rolling deployments.
- Run long data backfills in bounded, resumable batches.
- Understand why backups, transactions, and rollback plans are not interchangeable.
- Deploy migrations once through a controlled release process.

## What is a database migration?

A database migration is a tracked change that moves a database from one known structure or data state to another.

Common examples include:

- Creating a table or index.
- Adding, changing, or retiring a column.
- Adding a constraint after existing rows have been checked.
- Copying or transforming existing rows into a new format.
- Removing an obsolete structure after the application no longer depends on it.

A migration is different from seed data:

- A **schema migration** changes the structure or rules of the database.
- A **data migration** transforms data already stored in the database.
- **Seed data** creates known initial or demonstration records, usually for development or tests.

Keep production reference data changes deliberate and versioned too. Do not run development fixtures against real customer data.

## Version and track migrations

Store migration files in source control alongside the application. Give each migration a unique, ordered identifier and a name describing its purpose, for example:

~~~text
20261121_120000_add_customer_display_name
20261121_130000_backfill_customer_display_name
~~~

A migration tool typically records applied migration identifiers in a history table. This lets the release process determine which migrations are pending and prevents routine deployments from repeating completed work.

Good migration files are:

- Small enough to review and test.
- Focused on one logical change.
- Ordered and named consistently.
- Kept unchanged after they have run in a shared environment.
- Paired with a clear operational note for expensive or risky steps.

If an applied migration was wrong, create a new corrective migration. Editing an already-applied file makes the recorded history disagree with the file and can produce different databases in different environments.

Run migrations through one controlled deployment job. Do not make every PHP web worker, request, or autoscaled instance attempt to migrate at startup. A migration tool's lock can help prevent two operators or jobs from running simultaneously, but the release process should still designate one migration runner.

## Design changes to tolerate old and new code

During a rolling release, old and new PHP processes may run at the same time. Queue workers, scheduled jobs, and long-lived processes can continue running an older version after the web server has been updated. A schema change must account for all of them.

Prefer an additive change first:

1. **Expand:** Add the new table, column, or index without removing the old structure. Prefer a nullable or otherwise backward-compatible addition when appropriate.
2. **Deploy compatible code:** Deploy code that understands the old and new shape. During a rolling update, account for instances that have not yet received the new version.
3. **Drain old workers:** Restart or drain web workers, queue workers, and scheduled processes so they no longer run code that cannot handle the expanded schema.
4. **Backfill:** Copy or transform existing data in manageable batches. Make the process resumable and safe to repeat.
5. **Verify:** Check counts, nulls, constraints, and application behavior before switching reads or writes.
6. **Switch:** Move application reads to the new representation. Keep compatibility writes or fallback reads while rollback to the previous application version remains possible.
7. **Contract later:** Remove the old column or table only in a later release, after no active code, queued job, report, or rollback package needs it.

This is often called an **expand-contract** strategy. The separation gives each release a compatible intermediate state. It does not make a destructive change automatically safe; measure the migration, check locks, and define the rollback point.

## Choose a deployment strategy

### Maintenance-window deployment

For a small application that can tolerate a planned interruption, stop or pause writes, apply and verify the migration, deploy the matching code, then resume traffic. This is straightforward, but the interruption and recovery time must be acceptable.

### Rolling deployment

Instances update gradually while the service remains available. Every intermediate application version must work with the database schema throughout the rollout. Add the new schema first, deploy compatibility code, and postpone destructive changes until old instances are gone.

### Blue-green deployment

A new application environment is prepared alongside the current one and traffic is switched after verification. Both environments may use the same database during the transition, so both versions must be compatible with the schema. Switching web traffic does not automatically switch background workers or undo a migration.

### Canary deployment

A new application version receives a small portion of traffic first. Monitor errors, latency, and business outcomes before expanding the rollout. A canary still shares the database and therefore needs a compatible schema from the start.

A feature flag can control when the new behavior becomes active, but it does not replace a compatible schema or a plan for old processes.

## Deploy code and schema in a deliberate order

A typical expand-contract release can follow this sequence:

1. Review the migration and identify affected tables, indexes, constraints, and data.
2. Test it on the database engine and version used in production, with representative data volume.
3. Confirm backup or point-in-time recovery coverage and rehearse the required restore procedure.
4. Deploy the additive schema change through the single migration runner.
5. Verify the resulting schema and migration history.
6. Deploy PHP code compatible with both the old and expanded schema.
7. Drain or restart old web processes and background workers.
8. Backfill existing rows in bounded batches.
9. Verify data completeness and application behavior.
10. Enable the new behavior gradually when a feature flag or canary is available.
11. Monitor database load, locks, error rates, and business outcomes.
12. Remove the old schema in a later release after the rollback window and compatibility need have ended.

The exact order varies with the application and database. Write down the deployment sequence before running a migration that affects production data.

## Make data backfills safe to operate

A large one-shot update can hold locks, create a large transaction, generate substantial logs, or compete with user requests. For larger tables, process a bounded set at a time.

A production backfill should:

- Use a stable key or another deterministic way to select work.
- Process a limited batch, then commit and continue.
- Be idempotent where possible: running it again should not corrupt already-correct rows.
- Record progress without putting sensitive data in deployment logs.
- Stop safely on failure and allow an operator to resume.
- Be throttled or paused if normal application traffic is affected.
- Account for concurrent application writes so newly created or changed rows are not missed.

For a small table, a single update may be appropriate. For a large or busy table, test the plan and execution time using realistic data. There is no universal batch size; select one from measurements and monitor the production run.

## Understand transactions and DDL limits

A transaction can make a group of supported data changes atomic: commit the work if it succeeds or roll it back if it fails. PHP's PDO API provides transaction methods, but the database driver and storage engine must support the behavior you need.

Do not assume every schema change can be rolled back with a transaction. Some database systems issue implicit commits for DDL statements. Index creation, column changes, and constraint validation can also require locks or scan a large table. Locking and online-DDL behavior depend on the database engine, version, operation, and table definition.

Treat a schema migration as a database-specific operation:

- Read the documentation for the actual engine and version.
- Test lock duration, disk use, and runtime on a realistic copy.
- Use an online or concurrent operation only when that specific operation and engine support it.
- Schedule a maintenance window when the expected blocking or resource cost is not acceptable.
- Separate DDL from transactional data updates if the engine's DDL behavior requires it.

A transaction is useful, but it is not a backup and it is not a universal migration rollback plan.

## Prepare for failure and recovery

Before the release, decide what to do if the migration fails, the application fails after the schema change, or the data is only partly transformed.

A recovery plan should identify:

- The last known good application release.
- Which migrations completed and the actual schema state.
- Whether old and new application versions remain compatible.
- Whether the safest response is a code rollback, a forward fix, a compensating data change, or a database restore.
- Who can approve and run a restore, and how long it takes.
- Whether restoring would discard writes made after the backup.

Do not run a down migration automatically just because a deployment failed. A down migration may drop data or may no longer match the database state. In many cases, a forward fix is safer. If a restore is needed, understand its effect on all writes made since the recovery point.

Backups matter only if they can be restored. Rehearse restoration in a separate environment and verify the restored application and data before relying on it.

## Use PHP transactions for suitable data changes

For a group of related DML operations that the selected database supports transactionally, PDO can commit them together:

~~~php
<?php
$pdo->beginTransaction();

try {
    $insertCustomer->execute($customerParameters);
    $insertAuditEvent->execute($auditParameters);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

This is a data-change example, not a universal wrapper for DDL migrations. Configure PDO to throw exceptions or check every return value, and do not send raw exception details to visitors. Follow the database engine's transaction behavior for the statements in use.

## Test migrations before production

Test each migration against more than a brand-new empty database:

- **Fresh install:** Can the latest schema be created from scratch?
- **Upgrade:** Does the migration work from the prior supported schema?
- **Representative data:** Does it preserve values, handle nulls, and finish in an acceptable time?
- **Application compatibility:** Can old and new application versions run during the transition?
- **Failure behavior:** If a batch or deployment stops partway through, can it resume or be corrected safely?
- **Restore:** Can the team restore a backup or point-in-time copy and validate it?
- **Post-migration behavior:** Do key pages, writes, jobs, and reports behave as expected?

Use an isolated test database with appropriate synthetic or sanitized data. Do not test migrations by experimenting on production customer records.

## Common mistakes

- Running migrations from every web request or every application instance at startup.
- Removing a column in the same release that first stops using it.
- Editing a migration after it has run in a shared environment.
- Assuming DDL always rolls back with PDO transactions.
- Running a large backfill as one unbounded update without testing its effect.
- Applying a migration without checking database locks and version-specific behavior.
- Treating a code rollback as safe even though the new schema is incompatible with the previous code.
- Assuming a backup is usable without a restore rehearsal.
- Starting a rollout without checking queue workers, schedulers, and other long-lived processes.

## Deployment checklist

Before a production migration:

- [ ] Review the SQL or migration code and identify its data and availability risks.
- [ ] Test on the production database engine and version with representative data.
- [ ] Confirm the backup or point-in-time recovery window and test restoration.
- [ ] Verify the old and new application releases can coexist with the intermediate schema.
- [ ] Choose one migration runner and prevent accidental concurrent runs.
- [ ] Decide whether a maintenance window, rolling release, blue-green release, or canary is appropriate.
- [ ] Plan data backfill batches, monitoring, pause conditions, and resume behavior.
- [ ] Define rollback, forward-fix, and restore decision points.
- [ ] Verify migrations and application behavior after deployment.
- [ ] Schedule destructive contract changes as a separate later release.

## Key takeaways

- A migration is a versioned database change that must be deployed with compatible application code.
- Run migrations once through a controlled release process, not from web requests or every instance.
- Add new structures before depending on them; remove old structures only after old code and rollback needs are gone.
- Backfill large datasets in bounded, monitored, resumable batches.
- DDL transaction, lock, and online-operation behavior varies by database engine and operation.
- A backup, a code rollback, and a schema rollback solve different problems.
- Test upgrades, compatibility, failure recovery, and restores before production.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [143 - Production configuration, permissions, and logging](143-production-configuration-permissions-and-logging.md) | [Q&A](../qnas/144-database-migration-and-deployment-strategies.md) | [Example](../examples/144-database-migration-and-deployment-strategies.md) | [145 - PHP version compatibility and migration guides](145-php-version-compatibility-and-migration-guides.md) |