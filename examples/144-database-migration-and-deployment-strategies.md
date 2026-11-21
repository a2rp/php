# Database migration and deployment strategies — Example

## Scenario

A PHP application currently stores a customer's name in a **customers.name** column. The application needs a separate **display_name** column. The site must stay available during deployment, and old PHP workers may remain active while a release rolls out.

The safe approach is to add the new column first, deploy code that supports both columns, drain old-only processes, backfill existing data in batches, verify the result, and retire the old column only in a later release.

The SQL and PHP below are illustrative. Check syntax, data types, locks, and transaction behavior for the database engine and version used by the application.

## Starting assumptions

- The table has a stable primary key named **id**.
- The existing **name** value is available for the initial display name.
- The migration runner records applied migrations in its migration history.
- The release process runs migrations once, not once per web request or PHP instance.
- The PHP deployment has a protected database connection supplied through its environment.

## Step 1: Review and test the change

Before production:

1. Check how the application reads and writes customer names.
2. Check whether reports, queue jobs, scheduled tasks, and other services query the old column.
3. Test the migration on the production database engine and version.
4. Use representative data volume to measure runtime, lock behavior, and disk use.
5. Confirm a restorable backup or point-in-time recovery window.
6. Prepare the application release and a recovery decision before the migration begins.

Do not assume that a small local table predicts how a production schema change will behave.

## Step 2: Add the new column

Create a versioned migration and run it through the deployment migration job:

~~~sql
ALTER TABLE customers
    ADD COLUMN display_name VARCHAR(120) NULL;
~~~

This is an additive change: the old application can continue using **name**, while new code can begin supporting **display_name**. The exact DDL impact depends on the database and table. Verify lock and runtime behavior before applying it to production.

The migration tool should record the applied migration. If the job stops after the schema changes but before the history is updated, inspect the actual table and history before retrying; do not blindly rerun the DDL.

## Step 3: Deploy compatibility code

After the column exists, deploy PHP code that prefers the new value when present and falls back to the old one:

~~~php
<?php
function customerDisplayName(array $customer): string
{
    $newName = trim((string) ($customer['display_name'] ?? ''));

    if ($newName !== '') {
        return $newName;
    }

    return (string) ($customer['name'] ?? '');
}
~~~

When a customer changes the display name, keep both columns synchronized during the compatibility period:

~~~php
<?php
$statement = $pdo->prepare(
    'UPDATE customers
     SET name = :legacy_name,
         display_name = :display_name
     WHERE id = :id'
);

$statement->execute([
    'legacy_name' => $displayName,
    'display_name' => $displayName,
    'id' => $customerId,
]);
~~~

Use prepared statements for values. The compatibility release must be able to run with the expanded schema, and it must continue maintaining the old column while rollback to old code remains possible.

Wait until old-only web workers, queue consumers, and scheduled processes have been drained or restarted. If an old process can still update only **name**, it can race with the backfill and leave **display_name** stale.

## Step 4: Backfill existing rows in batches

Run the backfill as a controlled command or deployment job after old-only writers have been drained. The example selects at most 250 rows per batch. Each batch uses a transaction for its data updates, then commits before the next batch.

Supply **DATABASE_DSN**, **DATABASE_USER**, and **DATABASE_PASSWORD** through the protected deployment environment. Do not hard-code credentials into the script or print them in job output.

~~~php
<?php
declare(strict_types=1);

$dsn = getenv('DATABASE_DSN');
$username = getenv('DATABASE_USER');
$password = getenv('DATABASE_PASSWORD');

if ($dsn === false || $dsn === '' || $username === false || $username === '' || $password === false || $password === '') {
    fwrite(STDERR, "Database connection settings are missing.\n");
    exit(1);
}

$pdo = new PDO(
    $dsn,
    $username,
    $password,
    [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
    ]
);



$updateCustomer = $pdo->prepare(
    'UPDATE customers
     SET display_name = :display_name
     WHERE id = :id
       AND display_name IS NULL'
);

$totalProcessed = 0;

while (true) {
    $rows = $pdo->query(
        'SELECT id, name
         FROM customers
         WHERE display_name IS NULL
           AND name IS NOT NULL
         ORDER BY id
         LIMIT 250'
    )->fetchAll();

    if ($rows === []) {
        break;
    }

    $pdo->beginTransaction();

    try {
        foreach ($rows as $row) {
            $updateCustomer->execute([
                'display_name' => $row['name'],
                'id' => $row['id'],
            ]);
        }

        $pdo->commit();
    } catch (Throwable $exception) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        throw $exception;
    }

    $batchCount = count($rows);
    $totalProcessed += $batchCount;

    fwrite(
        STDOUT,
        sprintf("Processed %d rows; total processed: %d\n", $batchCount, $totalProcessed)
    );
}
~~~

This script updates rows whose new value is still null, so completed rows are skipped on a subsequent run. It avoids printing customer names or credentials. Run only one copy at a time, monitor its impact, and stop or throttle it if it harms normal traffic.

The example assumes the source name is the correct initial display name. If the real transformation is more complex, define and test that rule before running the backfill. If null source names are legitimate, they remain null here and should be handled according to the application's requirements.

## Step 5: Verify before switching reads

Check how many rows with an existing name still need a display name:

~~~sql
SELECT COUNT(*) AS remaining_rows
FROM customers
WHERE display_name IS NULL
  AND name IS NOT NULL;
~~~

Confirm that the result is expected, then test the application paths that read and update customer names. Also inspect the migration history, error logs, database load, and any relevant background jobs.

Do not switch all reads just because the backfill command exited successfully. Verify the data and behavior first.

## Step 6: Switch behavior, then contract later

After verification:

1. Enable the code path that reads **display_name**.
2. Keep a fallback to **name** while it is useful for recovery.
3. Continue writing both columns until the rollback window has passed.
4. Monitor errors and customer-facing behavior.
5. In a later release, confirm no application, report, job, or rollback package needs **name**.
6. Only then create a separate reviewed migration to retire the old column.

Do not combine dropping **name** with the first release that starts using **display_name**. Keeping it temporarily makes a code rollback more practical and gives the team time to verify the new path.

## Recovery plan for this example

- **Migration fails before the column is added:** Inspect the database and migration history, correct the issue, then retry through the controlled runner.
- **Column exists but the migration history is missing the entry:** Inspect the actual schema and tool behavior. Reconcile the history carefully; do not blindly rerun the DDL.
- **Backfill stops partway through:** Check transaction outcome and remaining-row count. Since processed rows are no longer null, rerunning the batch process can continue with the rows still needing a value.
- **The new application release fails before the old column is retired:** If the old application remains compatible with the added column, roll back the code while leaving the additive schema in place. Confirm old code can still read and write correctly.
- **A destructive contract migration has already run:** A code rollback may no longer be sufficient. Follow the planned forward-fix or restore procedure and account for writes made after the recovery point.

A database restore can discard newer writes. Confirm the recovery point and expected data loss before choosing it.

## Expected release timeline

~~~text
Review and test
    -> add nullable display_name column
    -> deploy compatibility code that reads and writes both columns
    -> drain old-only workers
    -> backfill existing rows in bounded batches
    -> verify remaining rows and application behavior
    -> switch reads to display_name
    -> wait through the rollback window
    -> remove name in a separate later release
~~~

## Expected result

- The old release continues to work after the additive schema change.
- New code can operate during a rolling deployment.
- Existing rows are copied in small transactions instead of one unbounded update.
- Operators can verify progress, pause the work, and resume after a failure.
- The old column remains available until application compatibility and rollback needs have ended.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [143 - Production configuration, permissions, and logging](../lessons/143-production-configuration-permissions-and-logging.md) | [Lesson](../lessons/144-database-migration-and-deployment-strategies.md) | [Q&A](../qnas/144-database-migration-and-deployment-strategies.md) | [145 - PHP version compatibility and migration guides](../lessons/145-php-version-compatibility-and-migration-guides.md) |