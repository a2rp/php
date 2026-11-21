# 098 - Relational databases and SQL fundamentals: Q&A

## Multiple-choice questions

### 1. What does a row in a relational table usually represent?

- **A.** One record about the table's subject
- **B.** A database connection
- **C.** A SQL keyword
- **D.** A relationship between every table

**Answer: A.** A row is one record, such as one customer or one order. Columns describe the properties stored for each record.

### 2. What is a primary key used for?

- **A.** To uniquely identify a row
- **B.** To encrypt every column
- **C.** To sort every query automatically
- **D.** To connect to PHP

**Answer: A.** A primary key gives each row a stable unique identity. It does not automatically sort query results.

### 3. What does a foreign key help enforce?

- **A.** That a reference points to a corresponding row in another table
- **B.** That every query uses an index
- **C.** That a table can have only one row
- **D.** That a column is always public

**Answer: A.** A foreign key represents and can enforce a relationship, such as an order referring to an existing customer.

### 4. Which constraint requires a column to have a value?

- **A.** NOT NULL
- **B.** ORDER BY
- **C.** GROUP BY
- **D.** LIMIT

**Answer: A.** NOT NULL prevents a row from storing NULL in that column. It does not mean the value must be a non-empty string unless another rule enforces that.

### 5. What is the difference between NULL and an empty string?

- **A.** They are always identical
- **B.** NULL means missing or unknown; an empty string is a present value with zero characters
- **C.** An empty string is a database constraint
- **D.** NULL is the same as the number zero

**Answer: B.** These values have different meanings. A query checks for NULL with IS NULL rather than equals NULL.

### 6. How should a query find rows where a column is NULL?

- **A.** WHERE column = NULL
- **B.** WHERE column IS NULL
- **C.** WHERE column == ''
- **D.** WHERE column IS EMPTY

**Answer: B.** SQL uses IS NULL and IS NOT NULL for null checks. Comparisons with NULL do not behave like ordinary equality comparisons.

### 7. Which relationship is commonly modeled with a foreign key on the child table?

- **A.** One-to-many
- **B.** Many-to-many without a junction table
- **C.** Every-to-every
- **D.** No relationship

**Answer: A.** For example, each order row can store a customer_id that references one customer. One customer can then have many order rows.

### 8. How is a many-to-many relationship commonly represented?

- **A.** By repeating comma-separated IDs in one column
- **B.** With a junction table that references both related tables
- **C.** By removing primary keys
- **D.** By storing every relation in a comment

**Answer: B.** A junction table stores one row per association and can include relationship attributes such as quantity or the price at the time of an order.

### 9. What does an INNER JOIN return?

- **A.** Only rows that match the join condition
- **B.** Every possible row combination
- **C.** Only columns with NULL values
- **D.** A table's schema

**Answer: A.** An INNER JOIN includes matching rows from both sides. Rows without a match are omitted.

### 10. What does a LEFT JOIN preserve?

- **A.** Every row from the left table, even when no right-side row matches
- **B.** Only right-side rows
- **C.** Every table in the database
- **D.** Only rows with a primary key of 1

**Answer: A.** A LEFT JOIN returns all left-side rows and uses NULL for right-side columns where there is no match.

### 11. Which clause defines a predictable order for query results?

- **A.** ORDER BY
- **B.** WHERE
- **C.** INSERT
- **D.** CHECK

**Answer: A.** Without ORDER BY, row order is not guaranteed. State the order explicitly when the application depends on it.

### 12. What is the difference between WHERE and HAVING?

- **A.** WHERE filters rows before grouping; HAVING filters groups after aggregation
- **B.** WHERE creates a table; HAVING deletes one
- **C.** They are identical in every query
- **D.** HAVING sorts text alphabetically

**Answer: A.** Use WHERE to filter input rows and HAVING to apply conditions to grouped aggregate results.

### 13. Why should an UPDATE statement usually include a WHERE clause?

- **A.** To limit which rows are changed
- **B.** To create an index automatically
- **C.** To make the statement read-only
- **D.** To disable constraints

**Answer: A.** Without WHERE, UPDATE can change every row in the table. Confirm the condition before executing a destructive statement.

### 14. What is a database transaction?

- **A.** A group of operations handled as one logical unit
- **B.** A table alias
- **C.** A database column type
- **D.** A text search operator

**Answer: A.** Transactions let related changes commit together or be rolled back together when an operation fails.

### 15. Which is a tradeoff of adding an index?

- **A.** It may speed up reads but takes space and adds work to writes
- **B.** It makes every query instant
- **C.** It prevents all invalid data
- **D.** It replaces a primary key

**Answer: A.** Indexes can improve access paths, but they consume storage and need maintenance when data changes. Add them based on measured queries.

### 16. What is the preferred way to include untrusted values in SQL?

- **A.** Concatenate the values into the query string
- **B.** Use a parameterized query and bind the values
- **C.** Convert every value to uppercase
- **D.** Remove all punctuation

**Answer: B.** Parameter binding keeps values separate from SQL syntax. Manually escaping or filtering values is not a replacement for parameterized queries.

### 17. Can a placeholder usually represent a table name or ORDER BY direction?

- **A.** Yes, placeholders can safely replace every part of SQL syntax
- **B.** No, placeholders are for data values; dynamic identifiers or directions need a fixed allowlist
- **C.** Only when the value comes from a form
- **D.** Only with SQLite

**Answer: B.** SQL parameters generally bind values, not syntax such as a table name or ASC/DESC. Map such structural choices to application-owned options.

### 18. What does least privilege mean for an application database account?

- **A.** Grant only the database permissions that application needs
- **B.** Use an administrator account for convenience
- **C.** Store the password in every PHP file
- **D.** Disable every database constraint

**Answer: A.** Limiting permissions reduces the damage possible if a query or application path is compromised.

## Short-answer questions

### 19. What is a database schema?

**Answer:** The schema describes database structures such as tables, columns, data types, keys, and constraints. **Explanation:** It defines how data is organized and which structural rules the database should enforce.

### 20. Why are database constraints useful if PHP already validates input?

**Answer:** Constraints protect integrity for every code path that writes to the database, not only one form or endpoint. **Explanation:** Application validation improves feedback; database rules provide a final integrity boundary.

### 21. Why should a query list the columns it needs instead of selecting every column?

**Answer:** Explicit columns make the query's data requirements clear and reduce unnecessary retrieval or accidental exposure of newly added sensitive columns. **Explanation:** It also makes returned result shapes easier to maintain.

### 22. Why should data that belongs to an order item, such as quantity, be stored in the junction table?

**Answer:** Quantity describes the relationship between a particular order and a particular product. **Explanation:** A junction table represents that association and can hold attributes specific to it.

### 23. What is normalization intended to reduce?

**Answer:** Unnecessary repetition of the same facts across records. **Explanation:** Duplication can cause update, insertion, and deletion anomalies, such as inconsistent copies of a customer's email.

### 24. Why should a query use ORDER BY when the user expects newest records first?

**Answer:** Databases do not promise a useful default row order. **Explanation:** Include an explicit timestamp sort and a tie-breaker such as the primary key for stable ordering.

### 25. What does a transaction rollback do?

**Answer:** It cancels the uncommitted changes in the active transaction, subject to the database's transaction support and behavior. **Explanation:** This helps avoid partially applying a group of changes that the application treats as one unit.

### 26. Does an index enforce that a value is unique?

**Answer:** An ordinary index does not necessarily enforce uniqueness; a UNIQUE constraint or unique index is needed. **Explanation:** Indexes support access paths, while unique rules protect a data property.

### 27. Why can an index make an application slower in some cases?

**Answer:** Indexes consume storage and must be maintained during inserts, updates, and deletes. **Explanation:** An index that does not help important queries adds overhead without enough benefit.

### 28. Why is input validation not a substitute for parameterized SQL?

**Answer:** Validation checks whether a value meets application rules, while parameterization keeps the value out of SQL syntax. **Explanation:** Even validated strings can contain characters that change a concatenated query.

### 29. What should an application do when a sort column comes from a request parameter?

**Answer:** Map the requested option to a fixed allowlist of known column expressions. **Explanation:** A normal value placeholder cannot bind an identifier or SQL keyword such as ASC or DESC.

## Practical scenarios

### 30. A customer row has several phone numbers. Should they be stored as one comma-separated string?

**Answer:** Usually no. Store phone numbers as related rows if they need independent validation, querying, or updates. **Explanation:** One value per field keeps data easier to search, constrain, and maintain.

### 31. A report must show every customer, including customers with no orders. Which join should it start with?

**Answer:** Start from customers and use a LEFT JOIN to orders. **Explanation:** LEFT JOIN preserves all rows from the left-side customer table and supplies NULL for missing order matches.

### 32. The application updates a customer's display name, but a test changes every customer's name. What is the likely query issue?

**Answer:** The UPDATE statement probably omitted or incorrectly built its WHERE clause. **Explanation:** Review the condition and test it against a safe dataset before running a change against important data.

### 33. A web request controls the value used for a query's WHERE condition. What is the safe approach?

**Answer:** Use a prepared, parameterized query and bind the request value. **Explanation:** Binding treats the value as data instead of allowing it to alter the SQL statement's structure.

### 34. A web request controls the ORDER BY column. Can the application bind that column as a parameter?

**Answer:** Usually no. Validate the requested sort key against a fixed allowlist and select the matching application-owned SQL fragment. **Explanation:** Parameters bind values, not SQL identifiers.

### 35. A workflow subtracts money from one account and adds it to another. Why is a transaction appropriate?

**Answer:** Both updates form one logical operation and should not be left partially applied. **Explanation:** A transaction allows the application to commit both changes or roll them back if a required step fails.

### 36. A query works in one SQL database but fails in another. Does that necessarily mean one database is broken?

**Answer:** No. SQL dialects differ in data types, functions, generated-key syntax, pagination, and other features. **Explanation:** Check the selected engine's syntax and behavior rather than assuming every system implements every feature identically.

### 37. The database query is slow, so a developer adds an index to every column. Is this a sound plan?

**Answer:** No. First inspect the slow query and its query plan, then add targeted indexes that support measured access patterns. **Explanation:** Indexes have storage and write costs and are not guaranteed to improve every query.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [097 - Building JSON and REST APIs](../lessons/097-building-json-and-rest-apis.md) | [Lesson](../lessons/098-relational-databases-and-sql-fundamentals.md) | [Example](../examples/098-relational-databases-and-sql-fundamentals.md) | [099 - Connecting to databases with PDO](../lessons/099-connecting-to-databases-with-pdo.md) |
