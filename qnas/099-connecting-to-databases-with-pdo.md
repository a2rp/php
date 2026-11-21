# 099 - Connecting to databases with PDO: Q&A

## Multiple-choice questions

### 1. What is PDO?

- **A.** A relational database server
- **B.** A common PHP interface for database connections and statements
- **C.** An ORM that removes the need to write SQL
- **D.** A database backup format

**Answer: B.** PDO provides a common API in PHP. A database-specific PDO driver is still required to connect to a particular database system.

### 2. What does a PDO driver do?

- **A.** Connects PDO to a specific database system
- **B.** Automatically designs the database schema
- **C.** Replaces the database server
- **D.** Converts every SQL query into HTML

**Answer: A.** Drivers such as PDO_SQLITE and PDO_MYSQL implement PDO's interface for their respective databases.

### 3. Which method lists drivers available to the current PHP runtime?

- **A.** PDO::getAvailableDrivers()
- **B.** PDO::listTables()
- **C.** PDO::getUsers()
- **D.** PDO::queryDrivers()

**Answer: A.** The method reports PDO drivers loaded by that runtime. The CLI and web server may use different PHP configurations.

### 4. What is a DSN?

- **A.** A database connection description whose format depends on the driver
- **B.** A SQL result set
- **C.** A password hashing algorithm
- **D.** A PHP session identifier

**Answer: A.** A data source name identifies the driver and connection information, such as an SQLite database path or MySQL host and database name.

### 5. Which DSN connects PDO_SQLITE to an in-memory database?

- **A.** sqlite::memory:
- **B.** mysql::memory:
- **C.** pgsql::memory:
- **D.** sqlite://memory

**Answer: A.** PDO_SQLITE uses the sqlite: prefix. Appending :memory: creates a temporary in-memory database for that connection.

### 6. Why pass a database username and password separately from the DSN when the driver supports it?

- **A.** It keeps credentials separate from connection syntax
- **B.** It makes passwords safe to commit
- **C.** It disables database authentication
- **D.** It converts the DSN into a prepared statement

**Answer: A.** Separate constructor arguments are clearer and reduce the chance of mixing credentials into a connection string. Secrets still need protected runtime configuration.

### 7. What does PDO::ERRMODE_EXCEPTION do?

- **A.** Makes many database failures throw PDOException
- **B.** Disables all database errors
- **C.** Prints every exception to the browser safely
- **D.** Automatically retries every query

**Answer: A.** Exception mode gives application code a consistent way to catch statement errors. It does not make it safe to expose exception details to visitors.

### 8. What does PDO::FETCH_ASSOC return for each row?

- **A.** An array indexed by column name
- **B.** Only the column count
- **C.** A random row object
- **D.** The original SQL string

**Answer: A.** Associative rows use column names as keys. This avoids the duplicate numeric and named keys of PDO::FETCH_BOTH.

### 9. When is PDO::query() appropriate?

- **A.** For a static SQL statement with no input values
- **B.** For concatenating untrusted request data into SQL
- **C.** For connecting to the database
- **D.** Only for HTTP requests

**Answer: A.** query() prepares and executes SQL without bound placeholders. Use prepare() and execute() for dynamic values; the next lesson explains that pattern.

### 10. What does PDOStatement::fetch() return when there are no more rows?

- **A.** false
- **B.** The database password
- **C.** A new connection
- **D.** Always an empty string

**Answer: A.** Fetch one row at a time until fetch() returns false. The returned row format depends on the fetch mode.

### 11. Why should fetchAll() be used carefully?

- **A.** It loads all remaining rows into memory
- **B.** It deletes every fetched row
- **C.** It closes the database server
- **D.** It only works with UPDATE

**Answer: A.** fetchAll() is convenient for small bounded results. Large results should generally be iterated or paginated to limit memory use.

### 12. Should rowCount() be used to count rows returned by SELECT across all PDO drivers?

- **A.** Yes, it is identical for every database
- **B.** No, SELECT row-count behavior varies by driver
- **C.** Only when the table has an index
- **D.** Only when the result is empty

**Answer: B.** Use fetches or an explicit COUNT query when a SELECT row count is needed. rowCount() is commonly useful for affected rows, but exact behavior can still depend on the driver and statement.

### 13. When is PDO::exec() commonly used?

- **A.** For a statement that does not return a result set
- **B.** To fetch the next row
- **C.** To list loaded extensions
- **D.** To render a PHP template

**Answer: A.** exec() can run statements such as fixed schema commands or updates without returned rows. It is not for SQL assembled from untrusted input.

### 14. What is the purpose of PDO::beginTransaction()?

- **A.** To start a transaction for related database operations
- **B.** To install a PDO driver
- **C.** To set a fetch mode
- **D.** To commit every future request automatically

**Answer: A.** It starts a transaction so a related group of operations can later be committed or rolled back as a unit.

### 15. What should code do after a transaction operation fails?

- **A.** Roll back if a transaction is active, then handle or rethrow the error
- **B.** Report success without checking
- **C.** Commit the partial changes
- **D.** Print the database password

**Answer: A.** Rollback prevents incomplete changes from being treated as a successful operation. Do not swallow the error and claim success.

### 16. Why might a PDO connection work in the CLI but fail in a web request?

- **A.** The CLI and web server can use different PHP configurations or loaded drivers
- **B.** PDO works only for terminal users
- **C.** Browsers cannot use databases
- **D.** SQL cannot be sent through HTTP

**Answer: A.** Different SAPIs may load different php.ini files and extensions. Check the actual runtime serving the request.

### 17. What is a good default for PDO persistent connections?

- **A.** Enable them only after measuring and understanding connection reuse
- **B.** Always enable them for every application
- **C.** Use them to store passwords
- **D.** They are required for transactions

**Answer: A.** Persistent connections can reuse state across requests and change connection assumptions. They are not a universal performance fix.

### 18. How should a production application respond to a database connection failure?

- **A.** Return a generic service error and log useful diagnostics securely
- **B.** Return the full PDOException and credentials
- **C.** Pretend the database operation succeeded
- **D.** Show the entire DSN to every visitor

**Answer: A.** Logs can contain diagnostic details for authorized operators, but public responses should not expose internal connection or query information.

## Short-answer questions

### 19. Does PDO itself install SQLite, MySQL, or PostgreSQL?

**Answer:** No. PDO needs the matching database-specific driver and a reachable database. **Explanation:** The PDO extension provides the common API, while a driver implements communication with a specific database system.

### 20. Why can checking PDO drivers from the terminal be insufficient?

**Answer:** A web server can use a different PHP version, php.ini file, or extension configuration. **Explanation:** Check the runtime that actually executes the application, not only a separate CLI runtime.

### 21. What is the difference between a file-backed and in-memory SQLite DSN?

**Answer:** A file-backed DSN stores data in a database file; an in-memory DSN creates temporary data associated with the connection. **Explanation:** The in-memory database normally disappears when the connection closes.

### 22. Why set the PDO error mode explicitly?

**Answer:** It makes the application's intended error behavior clear and consistent. **Explanation:** With exception mode, database errors can be handled through a catch boundary instead of being silently ignored.

### 23. Why is FETCH_ASSOC useful?

**Answer:** It returns rows indexed by column names. **Explanation:** Code can use readable keys and avoid duplicated numeric indexes from PDO::FETCH_BOTH.

### 24. Why should a statement cursor be fetched or closed?

**Answer:** An active result may hold database resources and can prevent another query on some drivers. **Explanation:** Iterate to completion or call closeCursor() when the result is no longer needed.

### 25. Why should rowCount() not be used as a portable SELECT count?

**Answer:** The number of rows reported for a SELECT is not consistent across PDO drivers. **Explanation:** Fetch rows or run a deliberate COUNT query for a result count.

### 26. What does a DSN typically contain?

**Answer:** A driver name and driver-specific connection details. **Explanation:** For example, SQLite uses a file path, while a server database DSN may include host, database name, port, or character set.

### 27. Why should application database credentials have limited permissions?

**Answer:** A restricted account limits what an application error or compromise can do to the database. **Explanation:** Grant only the privileges needed for that application's tasks.

### 28. Why should a database exception not be sent directly to a visitor?

**Answer:** It may reveal SQL, schema names, host details, paths, or other internal information. **Explanation:** Log useful details securely and return a generic response that does not disclose implementation internals.

## Practical scenarios

### 29. PDO exists, but PDO::getAvailableDrivers() does not list mysql. What should be checked?

**Answer:** Confirm that the PDO_MYSQL driver is installed and enabled for the same PHP runtime used by the application, then restart the relevant service if configuration changed. **Explanation:** The PDO core does not guarantee that every database driver is loaded.

### 30. A query returns thousands of rows and the code calls fetchAll(). What should be considered?

**Answer:** Iterate through the statement or paginate the query rather than loading the entire result set at once. **Explanation:** Holding every row in memory can increase memory use substantially.

### 31. A web handler catches PDOException and sends getMessage() to the browser. What should change?

**Answer:** Log the diagnostic details in a protected log, then return a generic public error response. **Explanation:** Database exception text may reveal internal implementation and operational data.

### 32. Two database changes must both succeed to represent a completed operation. What PDO feature should be used?

**Answer:** Wrap them in a transaction with beginTransaction(), commit(), and rollback handling. **Explanation:** This prevents the application from leaving only one half of a logically related operation applied.

### 33. Code calls query() with a value concatenated from a search form. What is the safer direction?

**Answer:** Use a prepared statement and bind or execute with the search value. **Explanation:** The next chapter covers parameterized queries; concatenating request data into SQL can allow the input to alter query syntax.

### 34. A development database works with PDO_SQLITE but production uses MySQL. What might need to change?

**Answer:** The DSN, driver configuration, some SQL syntax, data types, transaction behavior, and possibly value conversions. **Explanation:** PDO offers a common PHP interface, not identical database behavior across engines.

### 35. An update statement returns zero from rowCount(). Does that always prove an error occurred?

**Answer:** No. It can mean no rows matched or that the database reports only rows whose values changed; behavior can depend on the driver. **Explanation:** Check the application's expected outcome and understand the selected database's affected-row semantics.

### 36. A transaction fails and the catch block calls rollBack() unconditionally. What should be improved?

**Answer:** Check inTransaction() before rolling back, then preserve or handle the original failure. **Explanation:** The failure may happen before a transaction starts or after it has already ended.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [098 - Relational databases and SQL fundamentals](../lessons/098-relational-databases-and-sql-fundamentals.md) | [Lesson](../lessons/099-connecting-to-databases-with-pdo.md) | [Example](../examples/099-connecting-to-databases-with-pdo.md) | [100 - Prepared statements and safe database queries](../lessons/100-prepared-statements-and-safe-database-queries.md) |
