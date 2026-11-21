# 103 - Using the MySQLi extension: Q&A

## Multiple-choice questions

### 1. What is MySQLi?

- **A.** A PHP extension for connecting to MySQL-compatible servers
- **B.** A database schema language
- **C.** A general PHP API for every database engine
- **D.** A browser-side JavaScript library

**Answer: A.** MySQLi is a PHP database extension for MySQL. PDO is a separate API that can connect to multiple databases through drivers.

### 2. Which database API is designed to support multiple database systems through drivers?

- **A.** MySQLi
- **B.** PDO
- **C.** mysqli_result
- **D.** mysqli_stmt

**Answer: B.** PDO provides a common interface for multiple database drivers. MySQLi is focused on MySQL-compatible servers.

### 3. What does mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT) enable?

- **A.** Strict reporting that throws exceptions for MySQLi operation errors
- **B.** Automatic retry of every failed query
- **C.** Encryption of every table
- **D.** A default database password

**Answer: A.** Strict reporting lets the application handle MySQLi failures through exception flow. It does not validate input or make a query safe by itself.

### 4. Which method sets the MySQL connection character set?

- **A.** set_charset()
- **B.** set_character()
- **C.** prepare_charset()
- **D.** bind_charset()

**Answer: A.** Call set_charset(), commonly with utf8mb4, on the connection. This configures how the client sends and receives text.

### 5. When is mysqli::query() appropriate?

- **A.** For fixed SQL that contains no untrusted request values
- **B.** For concatenating raw form input into SQL
- **C.** To bind parameters
- **D.** To define the PHP runtime environment

**Answer: A.** Use query() for fixed statements without changing request values. Use a prepared statement and bind_param() for values that vary.

### 6. Which placeholder style does a MySQLi prepared statement use?

- **A.** Named markers such as :email
- **B.** Positional question marks
- **C.** Curly braces
- **D.** PHP variable names inside SQL

**Answer: B.** MySQLi prepared statements use question-mark markers. Bind one variable for each marker in the same order.

### 7. What does the type character i mean in bind_param()?

- **A.** Integer
- **B.** Image
- **C.** Identifier
- **D.** Index

**Answer: A.** MySQLi type characters are i for integer, d for float or double, s for string, and b for blob data.

### 8. Why does bind_param() require variables?

- **A.** It binds variables by reference and uses their values when execute() runs
- **B.** It can only bind global variables
- **C.** Variables automatically validate every value
- **D.** It does not allow values to change before execution

**Answer: A.** Bound variables are read at execution time. Assign computed expressions to variables before passing them to bind_param().

### 9. What must match the number of placeholders in a prepared statement?

- **A.** The length of the bind_param() type string and the number of bound variables
- **B.** The number of PHP files in the project
- **C.** The number of database tables
- **D.** The number of columns in every table

**Answer: A.** The type string must have one character per marker, and bind_param() must receive the same number of variables in corresponding order.

### 10. Can a placeholder represent a column name?

- **A.** Yes, it can represent any SQL fragment
- **B.** No, placeholders represent values; use a fixed allowlist for identifiers
- **C.** Yes, if the name is supplied by a trusted form
- **D.** Only for UPDATE statements

**Answer: B.** Parameters cannot act as table names, column names, or SQL keywords. Map supported choices to developer-written SQL fragments.

### 11. Which prepared-result method requires mysqlnd?

- **A.** get_result()
- **B.** bind_param()
- **C.** close()
- **D.** execute()

**Answer: A.** get_result() returns a mysqli_result and is available only when PHP uses mysqlnd.

### 12. What is an alternative to get_result() for fetching prepared-statement rows?

- **A.** bind_result() followed by fetch()
- **B.** query() followed by rollback()
- **C.** set_charset() followed by insert_id
- **D.** close() followed by execute()

**Answer: A.** bind_result() associates output variables with result columns. After execute(), call bind_result() and then fetch() rows.

### 13. What should the application do with a mysqli_result after finishing its rows?

- **A.** Free it when it is no longer needed
- **B.** Convert it to a table name
- **C.** Bind it as SQL
- **D.** Keep every result open indefinitely

**Answer: A.** Freeing results and closing statements makes resource use clearer and is especially useful in long-running processes.

### 14. What does mysqli_stmt::$affected_rows generally help report?

- **A.** Rows changed, inserted, deleted, or matched according to the statement and MySQL behavior
- **B.** The count of columns in the schema
- **C.** The number of bound variables
- **D.** The exact number of rows in every SELECT across all APIs

**Answer: A.** affected_rows is useful for write outcomes, but interpret it according to the statement and MySQL connection settings.

### 15. What does mysqli::$insert_id report after an auto-increment insert?

- **A.** The generated ID of the last insert on that connection
- **B.** The last SQL error
- **C.** The number of changed rows
- **D.** The server's port number

**Answer: A.** insert_id can retrieve the generated auto-increment ID after a successful insert on that connection.

### 16. What must be true for a MySQL table's changes to be rolled back in a transaction?

- **A.** The table must use a transactional storage engine such as InnoDB
- **B.** The table must have no columns
- **C.** The query must use query() instead of a statement
- **D.** The connection must use a numeric charset

**Answer: A.** A transaction cannot provide rollback for changes made to a non-transactional table.

### 17. Which MySQLi API methods control a transaction?

- **A.** begin_transaction(), commit(), and rollback()
- **B.** bind_param(), fetch(), and free()
- **C.** prepare(), get_result(), and close()
- **D.** set_charset(), insert_id, and affected_rows

**Answer: A.** Start the transaction, commit after every required write succeeds, and roll back active uncommitted work after failure.

### 18. What can happen if certain DDL statements run during a MySQL transaction?

- **A.** The server can implicitly commit
- **B.** MySQLi always undoes them on rollback
- **C.** The prepared statement becomes a PHP array
- **D.** The connection character set resets

**Answer: A.** Some schema statements cause implicit commits. Check MySQL behavior and keep migrations separate from ordinary transactional data work.

## Short-answer questions

### 19. Why choose either object-oriented or procedural MySQLi consistently?

**Answer:** Consistency makes connection, statement, result, and error-handling code easier to follow. **Explanation:** Both styles provide equivalent operations, but mixing them without a reason can make ownership and control flow harder to review.

### 20. Why keep MySQL credentials outside committed PHP source?

**Answer:** Source files can be copied, published, or shared; credentials should be supplied through protected runtime configuration. **Explanation:** Restrict database permissions as well and avoid logging secrets.

### 21. Why call set_charset('utf8mb4') after connecting?

**Answer:** It configures the client connection's character set for sending and receiving text. **Explanation:** Using the connection API keeps character-set configuration synchronized with the client library's behavior.

### 22. Why not replace prepared statements with mysqli_real_escape_string()?

**Answer:** Prepared statements keep changing values separate from SQL syntax and avoid manual quoting mistakes. **Explanation:** Escaping depends on context and connection settings; use parameter binding for values.

### 23. When does bind_param() read a variable's current value?

**Answer:** When the prepared statement executes. **Explanation:** The variables are passed by reference, so assigning them after binding and before execute() changes what is sent.

### 24. What is the b type character used for?

**Answer:** It identifies blob data sent in packets; large blob values can require mysqli_stmt_send_long_data(). **Explanation:** Follow the MySQLi API's blob handling rules and server packet limits for large values.

### 25. How should MySQLi bind a variable list for an IN clause?

**Answer:** Generate one question-mark marker per value, create a matching type string and variable list, then bind each value. **Explanation:** A single marker cannot represent an entire array. Handle an empty list explicitly.

### 26. What does a MySQLi transaction cover?

**Answer:** Transactional database operations on that connection and transactional tables. **Explanation:** It does not undo emails, files, HTTP calls, DDL that implicitly commits, or changes to non-transactional tables.

### 27. What should an application do with a database exception?

**Answer:** Log useful diagnostics securely and return a suitable generic response to a public caller. **Explanation:** Do not expose SQL, host details, credentials, file paths, or full exception messages.

### 28. Why should affected_rows be interpreted carefully?

**Answer:** Its value depends on the statement and MySQL behavior or connection options. **Explanation:** Zero can indicate no match or no changed value; define what the application expects and handle it intentionally.

## Practical scenarios

### 29. A statement has two question marks but bind_param() receives type string is and only one variable. What is wrong?

**Answer:** The marker count, type-string length, and bound-variable count do not match. **Explanation:** Provide one type character and one variable for each placeholder, in SQL order.

### 30. The query sorts by a request-provided column using ORDER BY ? and returns an unexpected order. Why?

**Answer:** The placeholder is a value, not a column identifier. **Explanation:** Map accepted sort choices to a fixed allowlist of SQL expressions.

### 31. A prepared SELECT works on one PHP server using get_result() but fails on another. What should be checked?

**Answer:** Check whether mysqlnd is available in the PHP runtime on the failing server. **Explanation:** get_result() requires mysqlnd; bind_result() provides another way to retrieve prepared results.

### 32. A developer calls bind_param('si', $name, 12). Why can this fail?

**Answer:** bind_param() expects variables by reference, but 12 is a literal value. **Explanation:** Assign the integer to a variable and pass that variable.

### 33. A developer binds a comma-separated string such as 4,9,15 to one marker in IN (?). Why does it not match three IDs?

**Answer:** One parameter represents one value; the comma-separated text is still one value. **Explanation:** Generate three question-mark markers and bind the three integers separately.

### 34. A transaction rollback does not undo a table change after CREATE TABLE ran. What might explain this?

**Answer:** MySQL can implicitly commit around some DDL statements. **Explanation:** Verify the statement's transaction behavior and keep schema migrations outside ordinary data transactions.

### 35. An UPDATE returns affected_rows equal to zero. Does that always mean an SQL error?

**Answer:** No. It may mean no row matched or that no value changed, depending on the query and MySQL settings. **Explanation:** Use strict error reporting to detect SQL failures and separately interpret the expected write outcome.

### 36. The CLI can connect through MySQLi, but a web request says the class or extension is missing. What should be checked?

**Answer:** Confirm the web server's PHP runtime has the mysqli extension enabled and is using the expected PHP configuration. **Explanation:** CLI and web-server PHP can load different versions, configuration files, or extensions.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [102 - Filtering, sorting, and paginating database results](../lessons/102-filtering-sorting-and-paginating-database-results.md) | [Lesson](../lessons/103-using-the-mysqli-extension.md) | [Example](../examples/103-using-the-mysqli-extension.md) | [104 - Schema design and database migrations](../lessons/104-schema-design-and-database-migrations.md) |
