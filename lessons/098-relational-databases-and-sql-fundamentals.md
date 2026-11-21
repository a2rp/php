# 098 - Relational databases and SQL fundamentals

## Learning goals

By the end of this lesson, you will be able to:

- Explain how a relational database stores structured data.
- Design tables with suitable columns, data types, keys, and constraints.
- Model one-to-one, one-to-many, and many-to-many relationships.
- Read and write data with common SQL statements.
- Combine related data with joins and summarize it with aggregate functions.
- Explain transactions, indexes, and basic query performance.
- Recognize unsafe SQL construction and the role of parameterized queries.

## 1. What a database does

A database is an organized store of data. A database management system (DBMS) provides a way to define, read, add, update, and delete that data while coordinating access and enforcing rules.

A relational database organizes data into tables and represents relationships between them. Common relational database systems include SQLite, MySQL, MariaDB, PostgreSQL, and SQL Server. They share many SQL concepts, but their data types, functions, administrative commands, and some syntax differ.

An application usually sends SQL to the database system. The database checks the statement, reads or changes data, and returns a result or an error. PHP can connect to many database systems; the next lesson introduces PHP's PDO interface.

## 2. Tables, rows, and columns

A **table** stores records about one kind of thing, such as customers, products, or orders.

- A **row** is one record.
- A **column** is one named property of that record.
- A **schema** defines the tables, columns, types, and rules.

A small customer table could be described like this:

| id | email | display_name | created_at |
| --- | --- | --- | --- |
| 1 | ada@example.test | Ada | 2026-01-12 10:30:00 |
| 2 | lin@example.test | Lin | 2026-01-13 09:15:00 |

Each row represents one customer. The email belongs to the customer record, while an order should be stored as an order record linked to that customer.

Choose names that describe the data and use one consistent naming convention. Avoid storing unrelated concepts in one table just because they currently appear together on a screen.

## 3. Choose columns and data types carefully

A column's data type describes the values it can hold and how the database handles them. Common categories include:

- Integers for whole-number identifiers and counts.
- Decimal types for exact decimal quantities, such as money where supported.
- Text types for names, labels, and descriptions.
- Date and time types for temporal values, depending on the database.
- Boolean-like types for true/false state, represented differently by different systems.
- Binary types for bytes when the database is intended to store binary data.

The right type and limits depend on the database and application. Do not assume that all SQL systems use identical type names or conversion rules.

Store money as a decimal type designed for exact values or as an integer number of minor units, such as cents. Binary floating-point types can represent some decimal fractions approximately, which can produce surprising arithmetic.

Use a clear policy for date and time values. A common application policy is to store instants in UTC and convert them for display. A date such as a birthday is not an instant and should not be shifted across time zones.

## 4. Understand NULL

NULL represents missing or unknown data. It is not the same as an empty string, zero, or false.

SQL uses three-valued logic: a comparison involving NULL is generally unknown, not true or false. Therefore, use IS NULL or IS NOT NULL instead of comparing a value with equals NULL.

~~~sql
SELECT id, display_name
FROM customers
WHERE phone_number IS NULL;
~~~

Decide which columns must always have a value and declare them NOT NULL. Avoid making every column nullable by default. When a value can be absent, document what that absence means.

Use COALESCE when the query needs an explicit fallback value, but do not hide important data-quality problems by replacing NULL everywhere.

## 5. Use keys and constraints to protect data

A **primary key** uniquely identifies each row. It should be stable and not depend on a value that users may change.

A **foreign key** connects a row to a row in another table. It helps enforce that a referenced record exists.

Other useful constraints include:

- **NOT NULL**: a value is required.
- **UNIQUE**: duplicate values are not allowed within the constrained key.
- **CHECK**: a value must satisfy a condition.
- **DEFAULT**: the database supplies a value when the insert omits that column.

These rules belong in the database schema as well as in application validation. Application checks provide helpful messages, while database constraints protect the data when more than one code path writes to the database.

~~~sql
CREATE TABLE customers (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    display_name TEXT NOT NULL,
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    customer_id INTEGER NOT NULL,
    status TEXT NOT NULL CHECK (
        status IN ('draft', 'placed', 'cancelled')
    ),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(id)
);
~~~

Exact auto-generated key syntax varies by database. Confirm the schema syntax and foreign-key behavior for the chosen engine.

## 6. Model relationships

Tables can be related in different ways.

### One-to-one

One row in a table corresponds to at most one row in another table. For example, a user may have one optional profile. A unique foreign key can enforce that a profile belongs to no more than one user.

### One-to-many

One row in the parent table can relate to many rows in a child table. A customer can have many orders. Put the customer's foreign key on each order row.

~~~sql
SELECT id, customer_id, status
FROM orders
WHERE customer_id = 1;
~~~

### Many-to-many

Rows in either table can relate to multiple rows in the other table. Orders can contain many products, and products can appear in many orders. Use a junction table such as order_items:

~~~sql
CREATE TABLE order_items (
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price_cents INTEGER NOT NULL CHECK (unit_price_cents >= 0),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);
~~~

The junction table can hold attributes about the relationship, such as quantity and the price at the time of an order. Storing the price on the order item preserves what the customer was charged even if the product's current price later changes.

## 7. Reduce duplication with normalization

Normalization is a way to structure relational data so each fact has a clear place and changes do not create avoidable inconsistencies.

For example, copying a customer's email into every order duplicates the same fact. If the customer changes their email, many rows may need updates and some may be missed. Instead, store the email in the customer row and refer to that row using customer_id.

A well-structured schema generally:

- Gives each table one clear subject.
- Stores one value per field rather than a comma-separated list.
- Uses keys to relate records.
- Avoids repeating the same fact in many rows.
- Stores values that belong to a relationship on a junction table when appropriate.

Normalization is a design tool, not a requirement to split every value into a separate table. Design around the application's real data and access patterns while preserving clear ownership and integrity.

## 8. Read data with SELECT

SELECT retrieves columns from one or more tables. Use WHERE to filter rows and ORDER BY to define result order.

~~~sql
SELECT id, email, display_name
FROM customers
WHERE id = 1;
~~~

Without ORDER BY, row order is not guaranteed. If the application needs stable ordering, name the sort columns explicitly and include a tie-breaker where values may match.

~~~sql
SELECT id, display_name
FROM customers
ORDER BY display_name ASC, id ASC
LIMIT 20;
~~~

Avoid selecting every column when the code needs only a few. Explicit columns make the result easier to understand and reduce accidental exposure if a table later gains sensitive fields.

The LIMIT and pagination syntax differs between database systems. Use the syntax supported by the selected database.

## 9. Combine tables with JOIN

A JOIN combines related rows. An INNER JOIN returns rows where the join condition matches. A LEFT JOIN preserves every row from the left table and fills missing columns from the right table with NULL.

~~~sql
SELECT
    orders.id AS order_id,
    customers.display_name AS customer_name,
    orders.status
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id
ORDER BY orders.id;
~~~

Use LEFT JOIN when the result should include parent rows even when there is no matching child:

~~~sql
SELECT
    customers.id,
    customers.display_name,
    orders.id AS order_id
FROM customers
LEFT JOIN orders
    ON orders.customer_id = customers.id
ORDER BY customers.id, orders.id;
~~~

Join on the intended keys. Joining unrelated columns can multiply rows and produce incorrect totals. Give repeated column names clear aliases so application code can distinguish them.

## 10. Summarize with aggregate functions

Aggregate functions compute a value from a set of rows. Common examples include COUNT, SUM, AVG, MIN, and MAX.

Use GROUP BY to form groups, and HAVING to filter groups after aggregation.

~~~sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 2;
~~~

WHERE filters individual rows before grouping. HAVING filters the grouped results. NULL values are ignored by most aggregates, while COUNT(*) counts rows. Check the chosen database's behavior when the distinction matters.

## 11. Change data with INSERT, UPDATE, and DELETE

INSERT adds a row. Name the columns so the statement does not depend on an implicit column order.

~~~sql
INSERT INTO customers (email, display_name)
VALUES ('sam@example.test', 'Sam');
~~~

UPDATE changes matching rows. Include a WHERE condition that identifies the intended records:

~~~sql
UPDATE customers
SET display_name = 'Sam Rivera'
WHERE id = 3;
~~~

DELETE removes matching rows:

~~~sql
DELETE FROM orders
WHERE id = 12;
~~~

An UPDATE or DELETE without a WHERE clause can affect every row in the table. Before running a destructive statement, confirm its conditions and understand any related foreign-key behavior. Production changes often need a transaction and a reviewed migration or administrative process.

## 12. Use transactions for related changes

A transaction groups database operations into one logical unit. If every operation succeeds, commit the transaction. If a required operation fails, roll the changes back.

The ACID properties describe common transaction goals:

- **Atomicity:** the group succeeds as a unit or is rolled back as a unit.
- **Consistency:** constraints and rules remain satisfied.
- **Isolation:** concurrent transactions are controlled according to the database's isolation behavior.
- **Durability:** committed changes survive according to the database's durability guarantees.

A bank transfer is a useful example: subtract from one account and add to another. If only one side succeeds, the data is wrong. Both updates should be part of one transaction.

~~~sql
BEGIN;

UPDATE accounts
SET balance_cents = balance_cents - 500
WHERE id = 1;

UPDATE accounts
SET balance_cents = balance_cents + 500
WHERE id = 2;

COMMIT;
~~~

The SQL syntax for transaction commands is widely supported, but exact behavior and supported isolation levels vary. Check affected-row counts and business conditions in the application before committing. The next PHP lesson demonstrates transaction handling through PDO.

## 13. Add indexes for useful access paths

An index is a data structure the database can use to locate rows without scanning an entire table. Indexes can speed up lookups, joins, ordering, and some uniqueness checks.

Indexes also have costs:

- They take storage space.
- Inserts, updates, and deletes may need to maintain them.
- A poor or redundant index can make writes slower without helping important queries.

Index columns that support measured application queries, such as a foreign key frequently used to find a customer's orders. Some database engines automatically index primary and unique keys; foreign-key index requirements differ.

An index does not guarantee that the database will use it. A query planner chooses a plan based on the query and available statistics. Use the database's query-plan tools to investigate real slow queries instead of adding indexes blindly.

## 14. Keep SQL structure separate from data values

Never build a query by concatenating untrusted input into SQL syntax. Use parameterized statements so values remain data instead of becoming SQL code. The next lesson shows parameter binding with PDO.

~~~text
Unsafe idea:
"SELECT id FROM customers WHERE email = '" + userInput + "'"

Safe approach:
Prepare a query with a placeholder for email, then bind or execute with the value.
~~~

Parameter binding protects values. It generally cannot substitute structural SQL such as a table name, column name, or ASC/DESC keyword. If those choices need to be dynamic, map the request to a fixed allowlist of application-owned choices.

Input validation remains useful for product rules, but validation does not replace parameter binding. Use a database account with only the permissions the application requires.

## 15. Understand SQL dialects and operational care

SQL has a common core, but database systems differ in data types, generated keys, date functions, string functions, upsert syntax, pagination, transaction behavior, and migration tooling.

When choosing a database:

- Identify required database features and hosting constraints.
- Learn the selected engine's connection, backup, and restore procedures.
- Test schema migrations against a safe environment before production.
- Keep credentials out of source control and restrict database network access.
- Plan backups and test that restoration works.
- Use least privilege for application and administrative accounts.
- Avoid exposing raw database errors to visitors.

The relational concepts transfer between systems, but SQL examples may need changes for a different dialect.

## Common mistakes

### Storing multiple values in one field

A comma-separated list makes filtering, updating, validation, and relationships difficult. Use related rows and a junction table for many-to-many data.

### Using NULL as if it were an empty string

NULL means missing or unknown; an empty string is a present value with zero characters. Decide which meaning the application needs and query NULL with IS NULL.

### Constructing SQL from request strings

Escaping strings manually is unreliable. Bind values through a parameterized query, and allowlist dynamic identifiers or sort choices.

### Assuming returned rows are ordered

A database does not promise order without ORDER BY. Define the order the application requires.

### Adding indexes without checking queries

Indexes use space and add write overhead. Measure representative queries and inspect query plans before making indexing decisions.

### Treating several writes as automatically all-or-nothing

Separate statements may partially succeed. Use a transaction when the application requires a group of changes to succeed together.

## Key takeaways

- Relational databases organize structured data into tables connected by keys.
- Use primary keys, foreign keys, and constraints to preserve data integrity.
- Model one-to-many relationships with a foreign key and many-to-many relationships with a junction table.
- Use SELECT, JOIN, grouping, INSERT, UPDATE, and DELETE with deliberate conditions.
- NULL is different from an empty string and requires IS NULL comparisons.
- Use transactions when related changes must succeed or fail together.
- Index measured access patterns and understand their write and storage costs.
- Bind data values in parameterized queries; use fixed allowlists for dynamic SQL structure.
- SQL dialects differ, so check the documentation for the selected database.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [097 - Building JSON and REST APIs](097-building-json-and-rest-apis.md) | [Q&A](../qnas/098-relational-databases-and-sql-fundamentals.md) | [Example](../examples/098-relational-databases-and-sql-fundamentals.md) | [099 - Connecting to databases with PDO](099-connecting-to-databases-with-pdo.md) |
