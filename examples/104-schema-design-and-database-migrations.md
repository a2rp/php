# 104 - Schema design and database migrations: Example

## Goal

Design a small article database with authors, articles, and tags. Apply its schema through ordered SQL migrations, add a new column in a later migration, and read an article and its tags from PHP.

## Question

How can a PHP application store articles and tags without duplicating data, enforce their relationships in MySQL, and evolve the schema without treating a failed migration as if nothing changed?

## Assumptions

- MySQL 8.0.16 or newer with InnoDB tables.
- PHP has PDO MySQL enabled.
- The database session and application follow a documented UTC time-zone policy.
- The migration commands below are for a new practice database. In an existing environment, inspect the migration history and schema first.
- The examples show SQL and PHP source; they do not create or run a database in this Markdown document.

## Project layout

~~~text
article-app/
├── database/
│   ├── bootstrap.sql
│   └── migrations/
│       ├── 001-create-content-tables.sql
│       └── 002-add-article-summary.sql
└── article-list.php
~~~

Migration names use an ordered version and a short description. Keep applied migration files in version control. Do not change a migration after it has run in a shared environment.

## 1. Create a practice database

Connect with the MySQL command-line client. Use a database account that is allowed to create this practice database, and enter its password at the prompt:

~~~sh
mysql --user=your_database_user --password
~~~

At the MySQL prompt, create the database and select it:

~~~sql
CREATE DATABASE php_schema_demo
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE php_schema_demo;
~~~

Choose a collation supported by the installed MySQL version. A different database product or version may use different syntax.

## 2. Bootstrap the migration history table

Create database/bootstrap.sql:

~~~sql
USE php_schema_demo;

CREATE TABLE schema_migrations (
    version VARCHAR(20) NOT NULL,
    description VARCHAR(200) NOT NULL,
    applied_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (version)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
~~~

The migration history table records which versions the deployment has applied. Create it once before applying numbered migrations. Do not create it separately for every release.

From the MySQL prompt, load the file. Replace the path with the actual location of the project:

~~~sql
SOURCE C:/path/to/article-app/database/bootstrap.sql;
~~~

Confirm that the table exists:

~~~sql
SHOW TABLES;
~~~

Expected output includes:

~~~text
schema_migrations
~~~

## 3. Migration 001: create normalized content tables

Create database/migrations/001-create-content-tables.sql:

~~~sql
CREATE TABLE authors (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    display_name VARCHAR(120) NOT NULL,
    email VARCHAR(254) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT uq_authors_email UNIQUE (email)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE articles (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    author_id INT UNSIGNED NOT NULL,
    slug VARCHAR(180) NOT NULL,
    title VARCHAR(200) NOT NULL,
    body MEDIUMTEXT NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'draft',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    CONSTRAINT uq_articles_slug UNIQUE (slug),
    CONSTRAINT chk_articles_status
        CHECK (status IN ('draft', 'published', 'archived')),
    INDEX idx_articles_author_created (author_id, created_at),
    CONSTRAINT fk_articles_author
        FOREIGN KEY (author_id)
        REFERENCES authors (id)
        ON DELETE RESTRICT
        ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE tags (
    id INT UNSIGNED NOT NULL AUTO_INCREMENT,
    slug VARCHAR(80) NOT NULL,
    name VARCHAR(80) NOT NULL,
    PRIMARY KEY (id),
    CONSTRAINT uq_tags_slug UNIQUE (slug),
    CONSTRAINT uq_tags_name UNIQUE (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE article_tags (
    article_id INT UNSIGNED NOT NULL,
    tag_id INT UNSIGNED NOT NULL,
    PRIMARY KEY (article_id, tag_id),
    INDEX idx_article_tags_tag_article (tag_id, article_id),
    CONSTRAINT fk_article_tags_article
        FOREIGN KEY (article_id)
        REFERENCES articles (id)
        ON DELETE CASCADE
        ON UPDATE RESTRICT,
    CONSTRAINT fk_article_tags_tag
        FOREIGN KEY (tag_id)
        REFERENCES tags (id)
        ON DELETE CASCADE
        ON UPDATE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

INSERT INTO schema_migrations (version, description)
VALUES ('001', 'Create normalized article and tag tables');
~~~

The history insert is deliberately last. It records version 001 only after the preceding statements finish. MySQL DDL can commit independently, so if any statement fails, some tables may already exist while the version row is absent. Inspect the database and repair the state deliberately; do not blindly run the entire file again.

### Why these tables are separate

- authors stores each author's name and email once.
- articles references an author through author_id.
- tags stores each tag once.
- article_tags stores the many-to-many relationship.
- The primary key on article_tags prevents duplicate pairs.
- The reverse index supports queries that begin with a tag.
- The unique constraints prevent duplicate author emails, article slugs, and tag names or slugs.
- The foreign keys stop an article from referencing a missing author and stop a relationship from referencing a missing article or tag.
- Deleting an article removes its relationship rows, while deleting an author with articles is restricted.
- The status CHECK constraint permits only the three documented states. Verify CHECK enforcement on the database version you deploy.

The author_id and articles.id columns use the same integer size and signedness. Matching foreign-key types avoids common schema creation errors.

Apply the migration once from the MySQL prompt:

~~~sql
SOURCE C:/path/to/article-app/database/migrations/001-create-content-tables.sql;
~~~

Check the ledger and table definitions:

~~~sql
SELECT version, description, applied_at
FROM schema_migrations
ORDER BY version;

SHOW CREATE TABLE articles;
~~~

Expected ledger rows:

~~~text
001 | Create normalized article and tag tables
~~~

## 4. Migration 002: add an optional summary

Suppose the application now needs a short article summary. Add it as nullable first so existing article rows remain valid and old code can keep running.

Create database/migrations/002-add-article-summary.sql:

~~~sql
ALTER TABLE articles
    ADD COLUMN summary VARCHAR(500) NULL AFTER title;

INSERT INTO schema_migrations (version, description)
VALUES ('002', 'Add optional article summary');
~~~

Apply version 002 after verifying version 001 is recorded:

~~~sql
SELECT version
FROM schema_migrations
ORDER BY version;

SOURCE C:/path/to/article-app/database/migrations/002-add-article-summary.sql;
~~~

Then verify the column and migration record:

~~~sql
SHOW COLUMNS FROM articles LIKE 'summary';

SELECT version, description
FROM schema_migrations
ORDER BY version;
~~~

Expected result: the articles table contains a nullable summary column, and the history table contains versions 001 and 002.

This is an additive change. New PHP code can begin reading or writing summary while older code ignores it. If the field later must become required, use a later staged migration:

1. Keep the field nullable while compatible application code is deployed.
2. Backfill existing rows in controlled batches with a real domain-appropriate value.
3. Verify no required rows remain NULL.
4. Add NOT NULL in a later migration after old application code no longer depends on NULL.

Do not invent a fake summary just to satisfy a constraint. If the summary is optional, keep it nullable.

## 5. Add representative data

Run these statements once after versions 001 and 002 are applied:

~~~sql
INSERT INTO authors (display_name, email)
VALUES ('Mina Rao', 'mina@example.test');

INSERT INTO articles (
    author_id,
    slug,
    title,
    summary,
    body,
    status
)
SELECT
    id,
    'schema-design',
    'Schema design',
    'A practical introduction to database structure.',
    'Tables, keys, relationships, and migrations form a durable data contract.',
    'published'
FROM authors
WHERE email = 'mina@example.test';

INSERT INTO tags (slug, name)
VALUES
    ('database', 'Database'),
    ('php', 'PHP');

INSERT INTO article_tags (article_id, tag_id)
SELECT article_row.id, tag_row.id
FROM articles AS article_row
CROSS JOIN tags AS tag_row
WHERE article_row.slug = 'schema-design'
  AND tag_row.slug IN ('database', 'php');
~~~

The example email and slug are unique, so rerunning the seed statements will produce a uniqueness error. That is intentional: seed data and migrations have different roles. For repeatable automated seed scripts, define an explicit duplicate-handling strategy instead of hiding unexpected conflicts.

Check the stored relationships:

~~~sql
SELECT
    articles.title,
    authors.display_name AS author,
    tags.name AS tag
FROM articles
JOIN authors
    ON authors.id = articles.author_id
JOIN article_tags
    ON article_tags.article_id = articles.id
JOIN tags
    ON tags.id = article_tags.tag_id
WHERE articles.slug = 'schema-design'
ORDER BY tags.name;
~~~

Expected rows:

~~~text
Schema design | Mina Rao | Database
Schema design | Mina Rao | PHP
~~~

The article appears twice because the result has one row per matching tag. The PHP example below groups those tag values for display.

## 6. Read the article from PHP with PDO

Create article-list.php:

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

try {
    $host = requiredEnvironmentValue('DB_HOST');
    $database = requiredEnvironmentValue('DB_NAME');
    $username = requiredEnvironmentValue('DB_USER');
    $password = requiredEnvironmentValue('DB_PASSWORD');

    $dsn = 'mysql:host=' . $host
        . ';dbname=' . $database
        . ';charset=utf8mb4';

    $pdo = new PDO($dsn, $username, $password, [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ]);

    $sql = <<<'SQL'
        SELECT
            articles.title,
            articles.summary,
            authors.display_name AS author,
            tags.name AS tag
        FROM articles
        JOIN authors
            ON authors.id = articles.author_id
        LEFT JOIN article_tags
            ON article_tags.article_id = articles.id
        LEFT JOIN tags
            ON tags.id = article_tags.tag_id
        WHERE articles.slug = :slug
          AND articles.status = :status
        ORDER BY tags.name
        SQL;

    $statement = $pdo->prepare($sql);
    $statement->execute([
        'slug' => 'schema-design',
        'status' => 'published',
    ]);

    $rows = $statement->fetchAll();

    if ($rows === []) {
        echo "Article not found.\n";
        exit(0);
    }

    $firstRow = $rows[0];
    $tagNames = [];

    foreach ($rows as $row) {
        if ($row['tag'] !== null) {
            $tagNames[] = $row['tag'];
        }
    }

    echo 'Title: ' . $firstRow['title'] . PHP_EOL;
    echo 'Summary: ' . $firstRow['summary'] . PHP_EOL;
    echo 'Author: ' . $firstRow['author'] . PHP_EOL;
    echo 'Tags: ' . implode(', ', $tagNames) . PHP_EOL;
} catch (Throwable $exception) {
    error_log($exception->__toString());
    fwrite(STDERR, "The article could not be loaded.\n");
    exit(1);
}
~~~

Set DB_HOST, DB_NAME, DB_USER, and DB_PASSWORD in the runtime environment using the deployment's secret-management method. Do not commit real credentials or print them in logs. Use an application database account with only the required read permissions for this script.

Run it from the project directory:

~~~sh
php article-list.php
~~~

Expected output:

~~~text
Title: Schema design
Summary: A practical introduction to database structure.
Author: Mina Rao
Tags: Database, PHP
~~~

The named markers bind values, not table or column names. The query uses fixed identifiers and bound values for the slug and status. The LEFT JOINs allow a published article with no tags to appear. Because the SELECT returns one row per tag, PHP collects the tag values before printing them.

## Migration workflow in a real project

A deployment system or migration tool should automate the same safe principles:

1. Read the migration history.
2. Identify pending versions in order.
3. Apply each migration once.
4. Record success only after the migration's required work is confirmed.
5. Stop on failure and report the exact version.
6. Require a deliberate repair when the schema is partially changed.
7. Verify the final schema and application health.

This example uses a manual MySQL client to make the SQL visible. It is not a production migration runner: it does not automate ordering, lock coordination, retries, permissions, or partial-failure recovery. For production, use a reviewed migration system and a documented deployment procedure. Do not place schema-change commands inside a normal web request.

## How it works

1. The schema separates authors, articles, tags, and article-tag relationships.
2. Primary and unique keys give each row stable identity and prevent duplicate domain values.
3. Foreign keys protect article ownership and tag relationships.
4. A composite primary key prevents the same tag from being attached twice to one article.
5. Version 001 creates the initial schema and records its version at the end.
6. Version 002 adds a nullable column so existing rows remain valid.
7. The ledger is checked before migrations are applied.
8. The PHP script connects with PDO and reads only the fields required for the display.
9. Prepared statement markers bind the slug and status values.
10. Credentials come from the process environment, and technical errors are logged rather than shown to visitors.

## Important notes

- MySQL DDL statements such as CREATE TABLE and ALTER TABLE can cause implicit commits. A surrounding transaction does not necessarily undo a failed schema migration.
- If a migration fails, inspect both the schema and schema_migrations before retrying. A missing ledger row does not prove that no DDL ran.
- Back up important data and test restore procedures before production schema changes.
- Online DDL support and locking vary by operation, engine, version, table size, and workload. Check the exact operation on a representative database and schedule risky changes.
- Keep old and new application versions compatible during deployment. Add first, backfill, switch application behavior, and remove old structure later.
- Use CHECK constraints only after verifying that the target database version enforces them.
- This example has not been executed against a MySQL server; review version-specific behavior in your actual environment before deployment.
- The displayed example database and email values are test data, not credentials.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [103 - Using the MySQLi extension](../lessons/103-using-the-mysqli-extension.md) | [Q&A](../qnas/104-schema-design-and-database-migrations.md) | [Lesson](../lessons/104-schema-design-and-database-migrations.md) | [105 - Command-line applications and arguments](../lessons/105-command-line-applications-and-arguments.md) |
