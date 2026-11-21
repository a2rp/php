# 103 - Using the MySQLi extension: Example

## Goal

Connect to MySQL with MySQLi, use a prepared statement to read rows, and create a published note together with its audit event in one InnoDB transaction.

## Question

How can a PHP script use MySQLi prepared statements and a transaction without placing credentials in source code or relying on mysqlnd for result fetching?

## Database setup

Create or choose a MySQL database, then run this schema once using a database administrator or migration tool. Configure the PHP application account with only the permissions it needs; the script itself needs SELECT and INSERT access to these tables.

~~~sql
CREATE TABLE lesson_notes (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_lesson_notes_status_id (status, id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE lesson_note_events (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    note_id INT UNSIGNED NOT NULL,
    event_type VARCHAR(40) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    INDEX idx_lesson_note_events_note_id (note_id),
    CONSTRAINT fk_lesson_note_events_note
        FOREIGN KEY (note_id) REFERENCES lesson_notes (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO lesson_notes (title, status)
VALUES ('Example draft note', 'draft');
~~~

The example tables use InnoDB so their related writes can be committed or rolled back together. Run the insert once to add the sample draft row.

## Complete example

Create a file named mysqli-demo.php:

~~~text
project/
└── mysqli-demo.php
~~~

~~~php
<?php

declare(strict_types=1);

function requiredEnvironmentValue(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException(
            'Required database configuration is missing.'
        );
    }

    return $value;
}

function runDemo(): void
{
    mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

    $mysqli = null;

    try {
        $host = requiredEnvironmentValue('DB_HOST');
        $username = requiredEnvironmentValue('DB_USER');
        $password = requiredEnvironmentValue('DB_PASSWORD');
        $database = requiredEnvironmentValue('DB_NAME');

        $portInput = getenv('DB_PORT');
        $port = 3306;

        if ($portInput !== false && $portInput !== '') {
            $validatedPort = filter_var(
                $portInput,
                FILTER_VALIDATE_INT
            );

            if (
                $validatedPort === false
                || $validatedPort < 1
                || $validatedPort > 65535
            ) {
                throw new RuntimeException(
                    'Database port configuration is invalid.'
                );
            }

            $port = $validatedPort;
        }

        $mysqli = new mysqli(
            $host,
            $username,
            $password,
            $database,
            $port
        );

        $mysqli->set_charset('utf8mb4');

        // Read existing draft notes without using get_result().
        $draftStatus = 'draft';
        $draftQuery = $mysqli->prepare(
            'SELECT id, title
             FROM lesson_notes
             WHERE status = ?
             ORDER BY id'
        );
        $draftQuery->bind_param('s', $draftStatus);
        $draftQuery->execute();
        $draftQuery->bind_result($draftId, $draftTitle);

        echo 'Draft notes:' . PHP_EOL;

        while ($draftQuery->fetch()) {
            echo '- ' . $draftTitle . PHP_EOL;
        }

        $draftQuery->close();

        $transactionStarted = false;

        try {
            $mysqli->begin_transaction();
            $transactionStarted = true;

            $title = 'MySQLi transaction note';
            $status = 'published';

            $insertNote = $mysqli->prepare(
                'INSERT INTO lesson_notes (title, status)
                 VALUES (?, ?)'
            );
            $insertNote->bind_param('ss', $title, $status);
            $insertNote->execute();

            $noteId = (int) $mysqli->insert_id;
            $insertNote->close();

            $eventType = 'created';
            $insertEvent = $mysqli->prepare(
                'INSERT INTO lesson_note_events (note_id, event_type)
                 VALUES (?, ?)'
            );
            $insertEvent->bind_param('is', $noteId, $eventType);
            $insertEvent->execute();
            $insertEvent->close();

            $mysqli->commit();
            $transactionStarted = false;
        } catch (Throwable $exception) {
            if ($transactionStarted) {
                try {
                    $mysqli->rollback();
                } catch (Throwable $rollbackException) {
                    error_log(
                        'MySQLi rollback failed: '
                        . $rollbackException->getMessage()
                    );
                }
            }

            throw $exception;
        }

        echo 'Saved note and audit event in one transaction.' . PHP_EOL;

        // Read the new row using bind_result(), which does not require mysqlnd.
        $savedNote = $mysqli->prepare(
            'SELECT title, status
             FROM lesson_notes
             WHERE id = ?'
        );
        $savedNote->bind_param('i', $noteId);
        $savedNote->execute();
        $savedNote->bind_result($savedTitle, $savedStatus);

        if (!$savedNote->fetch()) {
            throw new RuntimeException('The saved note was not found.');
        }

        echo 'Saved note status: '
            . $savedStatus
            . PHP_EOL;

        $savedNote->close();

        $eventCountQuery = $mysqli->prepare(
            'SELECT COUNT(*)
             FROM lesson_note_events
             WHERE note_id = ?'
        );
        $eventCountQuery->bind_param('i', $noteId);
        $eventCountQuery->execute();
        $eventCountQuery->bind_result($eventCount);
        $eventCountQuery->fetch();

        echo 'Audit events for saved note: '
            . $eventCount
            . PHP_EOL;

        $eventCountQuery->close();
    } catch (Throwable $exception) {
        error_log('MySQLi example failed: ' . $exception->getMessage());
        throw $exception;
    } finally {
        if ($mysqli instanceof mysqli) {
            $mysqli->close();
        }
    }
}

try {
    runDemo();
} catch (Throwable $exception) {
    fwrite(
        STDERR,
        'The database example could not be completed.' . PHP_EOL
    );
    exit(1);
}
~~~

## Configure and run

Set the connection values in the environment used by the PHP process:

~~~text
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=your_database
DB_USER=your_application_user
DB_PASSWORD=your_application_password
~~~

Then run the script from its directory:

~~~sh
php mysqli-demo.php
~~~

The PHP runtime must have the MySQLi extension enabled, and the configured account must be able to read and insert into the two tables.

Expected output when the setup SQL has inserted one draft row:

~~~text
Draft notes:
- Example draft note
Saved note and audit event in one transaction.
Saved note status: published
Audit events for saved note: 1
~~~

Running the PHP script again adds another published note and audit event. The draft list remains the same because the new notes use published status. Do not run the setup INSERT again unless you want another draft row.

## How it works

1. **Runtime configuration:** Connection details are read from environment variables. The PHP file contains no real credentials.
2. **Error mode:** Strict MySQLi reporting makes database operation failures throw exceptions.
3. **Character set:** set_charset('utf8mb4') configures the client connection for text.
4. **Prepared read:** The draft query uses a question-mark marker and bind_param() to supply the status value.
5. **Portable result binding:** bind_result() fetches selected columns into PHP variables and does not require get_result() or mysqlnd.
6. **Transaction:** The script inserts a published note and a related audit event in one InnoDB transaction.
7. **Generated ID:** insert_id supplies the new note ID for the audit row.
8. **Commit or rollback:** Both inserts commit together. A failure rolls back active transactional work; a rollback failure is logged without replacing the original failure.
9. **Verification:** Two prepared SELECT statements confirm the inserted status and its audit-event count.
10. **Cleanup:** Statements are closed after use and the connection is closed in finally.
11. **Safe errors:** Technical details go to the error log; the command line receives a generic failure message.

## Important notes

- This script is a learning example and assumes the schema already exists.
- The MySQLi connection must reach a MySQL-compatible server, and the tables must use InnoDB for the transaction behavior shown.
- Provision the database account with least privilege. Do not use an administrator account for an application.
- Keep environment variables protected and configure TLS where required by the deployment.
- This example uses bind_result() so it does not depend on mysqlnd. mysqli_stmt::get_result() is a convenient alternative when mysqlnd is available.
- The prepared statement values are safe from changing SQL syntax, but the application still needs authorization and business validation.
- The example logs exception details. Protect logs and do not include credentials or sensitive data in log messages.
- Database DDL and some other MySQL statements can implicitly commit; the example creates its tables separately from the application transaction.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [102 - Filtering, sorting, and paginating database results](../lessons/102-filtering-sorting-and-paginating-database-results.md) | [Q&A](../qnas/103-using-the-mysqli-extension.md) | Example | [104 - Schema design and database migrations](../lessons/104-schema-design-and-database-migrations.md) |
