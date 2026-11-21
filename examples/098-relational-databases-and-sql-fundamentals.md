# 098 - Relational databases and SQL fundamentals: Example

## Goal

Create a small relational database for customers, products, and orders. Use primary keys, foreign keys, constraints, SELECT queries, joins, aggregates, and a transaction that creates and removes an order safely.

## Question

How can related application data be represented in tables, queried with SQL, and changed without losing relationships or applying only half of a multi-step operation?

## Complete example

This example uses SQLite SQL so the database can be created locally without configuring a database server. The relational ideas apply to other systems, though SQL syntax and behavior can vary.

Create these files:

~~~text
project/
├── setup.sql
└── queries.sql
~~~

### 1. Create the schema and seed data: setup.sql

This script is for a disposable lesson database. It drops the lesson tables and recreates them so the example can be reset.

~~~sql
PRAGMA foreign_keys = ON;

DROP TABLE IF EXISTS order_items;
DROP TABLE IF EXISTS orders;
DROP TABLE IF EXISTS products;
DROP TABLE IF EXISTS customers;

CREATE TABLE customers (
    id INTEGER PRIMARY KEY,
    email TEXT NOT NULL UNIQUE,
    display_name TEXT NOT NULL CHECK (length(trim(display_name)) > 0),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
    id INTEGER PRIMARY KEY,
    sku TEXT NOT NULL UNIQUE,
    name TEXT NOT NULL CHECK (length(trim(name)) > 0),
    unit_price_cents INTEGER NOT NULL CHECK (unit_price_cents >= 0)
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

CREATE TABLE order_items (
    order_id INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    unit_price_cents INTEGER NOT NULL CHECK (unit_price_cents >= 0),
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id),
    FOREIGN KEY (product_id) REFERENCES products(id)
);

INSERT INTO customers (id, email, display_name) VALUES
    (1, 'ada@example.test', 'Ada'),
    (2, 'lin@example.test', 'Lin'),
    (3, 'sam@example.test', 'Sam');

INSERT INTO products (id, sku, name, unit_price_cents) VALUES
    (101, 'BAG-01', 'Canvas bag', 8990),
    (102, 'NOTE-01', 'Notebook', 2499),
    (103, 'PEN-01', 'Pen set', 1299);

INSERT INTO orders (id, customer_id, status) VALUES
    (1001, 1, 'placed'),
    (1002, 2, 'draft');

INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price_cents
) VALUES
    (1001, 101, 1, 8990),
    (1001, 102, 2, 2499),
    (1002, 103, 1, 1299);
~~~

The orders table references one customer, so one customer can have multiple orders. The order_items table connects orders and products, forming a many-to-many relationship. Its composite primary key prevents the same product from appearing twice as separate rows in one order.

The order item stores its own unit_price_cents. This preserves the price used for that order even if the product's current price changes later.

### 2. Query and update the data: queries.sql

~~~sql
PRAGMA foreign_keys = ON;

-- Read selected columns in an explicit order.
SELECT id, email, display_name
FROM customers
ORDER BY id;

-- Filter rows with a condition and sort the result.
SELECT id, name, unit_price_cents
FROM products
WHERE unit_price_cents >= 2000
ORDER BY unit_price_cents DESC, id ASC;

-- Join orders to their customer and item details.
SELECT
    orders.id AS order_id,
    customers.display_name AS customer_name,
    products.name AS product_name,
    order_items.quantity,
    order_items.unit_price_cents
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id
INNER JOIN order_items
    ON order_items.order_id = orders.id
INNER JOIN products
    ON products.id = order_items.product_id
ORDER BY orders.id, products.id;

-- Keep customers that have no orders.
SELECT
    customers.id,
    customers.display_name,
    COUNT(orders.id) AS order_count
FROM customers
LEFT JOIN orders
    ON orders.customer_id = customers.id
GROUP BY customers.id, customers.display_name
ORDER BY customers.id;

-- Calculate an order total from its item rows.
SELECT
    orders.id AS order_id,
    customers.display_name AS customer_name,
    SUM(order_items.quantity * order_items.unit_price_cents) AS total_cents
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id
INNER JOIN order_items
    ON order_items.order_id = orders.id
GROUP BY orders.id, customers.display_name
ORDER BY orders.id;

-- Update one customer only.
UPDATE customers
SET display_name = 'Sam Rivera'
WHERE id = 3;

SELECT id, display_name
FROM customers
WHERE id = 3;

-- Create an order and its item as one transaction.
BEGIN;

INSERT INTO orders (id, customer_id, status)
VALUES (1003, 3, 'placed');

INSERT INTO order_items (
    order_id,
    product_id,
    quantity,
    unit_price_cents
) VALUES (
    1003,
    102,
    1,
    2499
);

COMMIT;

SELECT
    orders.id AS order_id,
    customers.display_name AS customer_name,
    products.name AS product_name,
    order_items.quantity,
    order_items.unit_price_cents
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id
INNER JOIN order_items
    ON order_items.order_id = orders.id
INNER JOIN products
    ON products.id = order_items.product_id
WHERE orders.id = 1003;

-- Remove a test order. Delete its child rows before its parent row.
BEGIN;

DELETE FROM order_items
WHERE order_id = 1003;

DELETE FROM orders
WHERE id = 1003;

COMMIT;

SELECT id, customer_id, status
FROM orders
WHERE id = 1003;
~~~

### 3. Run the example

Install the SQLite command-line tool if it is not already available. From the project directory, run these commands in PowerShell:

~~~powershell
Get-Content .\setup.sql | sqlite3.exe .\lesson.db
Get-Content .\queries.sql | sqlite3.exe -header -column .\lesson.db
~~~

The first command builds a fresh lesson database. The second runs the queries and prints column headings. Re-running setup.sql drops and recreates only these lesson tables in lesson.db.

On a shell that supports input redirection, the equivalent commands are:

~~~bash
sqlite3 lesson.db < setup.sql
sqlite3 -header -column lesson.db < queries.sql
~~~

## Expected output

The customer query contains Ada, Lin, and Sam in ID order. The product filter returns Canvas bag and Notebook because both meet the 2,000-cent condition; Pen set does not.

The order detail query includes:

| order_id | customer_name | product_name | quantity | unit_price_cents |
| --- | --- | --- | --- | --- |
| 1001 | Ada | Canvas bag | 1 | 8990 |
| 1001 | Ada | Notebook | 2 | 2499 |
| 1002 | Lin | Pen set | 1 | 1299 |

The customer count query includes Sam with an order_count of 0. The LEFT JOIN preserves the customer even though there is no matching order.

The grouped total query returns:

| order_id | customer_name | total_cents |
| --- | --- | --- |
| 1001 | Ada | 13988 |
| 1002 | Lin | 1299 |

After the UPDATE, the customer with ID 3 is named Sam Rivera. The transaction creates order 1003 and one Notebook order item together. After the DELETE statements, selecting order 1003 returns no rows.

## How it works

1. **Schema:** Each table represents one subject. Columns have explicit types and rules.
2. **Primary keys:** Customer, product, and order IDs identify rows. The order item uses a composite key.
3. **Foreign keys:** Orders must reference an existing customer; order items must reference an existing order and product.
4. **Constraints:** NOT NULL, UNIQUE, and CHECK rules reject invalid values at the database layer.
5. **Relationships:** The customer-to-order relationship is one-to-many; orders-to-products is many-to-many through order_items.
6. **Queries:** SELECT names the needed columns, WHERE filters rows, ORDER BY makes the result order explicit, and JOIN combines related data.
7. **Aggregation:** GROUP BY forms per-order or per-customer groups, then COUNT and SUM calculate summaries.
8. **Transaction:** The new order and its item are committed together. A production application should roll back when a required step fails.
9. **Deletion order:** Child order_items are removed before their parent order because the foreign key keeps relationships valid.

## Important notes

- This example uses SQLite-specific schema syntax and the SQLite CLI. Another database may require different type names, generated-key syntax, or pagination syntax.
- The database is disposable learning data. The setup script drops and recreates its four named tables.
- The example uses integer cents for money to avoid binary floating-point rounding in this simple case. A production design should document currency and rounding rules.
- SQLite foreign-key enforcement is enabled per database connection, so setup.sql and queries.sql each enable it.
- Never construct SQL by concatenating untrusted request data. Use parameterized statements; the next lesson demonstrates this through PDO.
- Placeholders represent data values, not table names, column names, or sort directions. Map dynamic SQL structure to a fixed allowlist.
- A transaction's exact isolation and durability behavior depends on the database engine and configuration.
- Indexes should support measured queries. Primary and unique constraints may create indexes automatically; other index needs depend on the chosen database.
- This SQL-only example does not connect from PHP. PHP-to-database connections and prepared statements are the subject of the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [097 - Building JSON and REST APIs](../lessons/097-building-json-and-rest-apis.md) | [Q&A](../qnas/098-relational-databases-and-sql-fundamentals.md) | Example | [099 - Connecting to databases with PDO](../lessons/099-connecting-to-databases-with-pdo.md) |
