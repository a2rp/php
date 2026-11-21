# 102 - Filtering, sorting, and paginating database results: Q&A

## Multiple-choice questions

### 1. Where should a large database result set usually be filtered and paginated?

- **A.** In the database query, before rows are returned to PHP
- **B.** After downloading every row into PHP
- **C.** In the browser only
- **D.** In a CSS stylesheet

**Answer: A.** WHERE, ORDER BY, and page limits let the database return only the rows needed for the requested view, reducing unnecessary transfer and application memory.

### 2. How should a request's category ID be used in a filter?

- **A.** Concatenate it into the SQL string
- **B.** Validate it according to application rules and bind it as a value
- **C.** Treat it as a column name
- **D.** Use it as raw SQL after trimming whitespace

**Answer: B.** Validation checks whether it is an acceptable category ID; parameter binding keeps the value separate from SQL syntax.

### 3. What should an application add to SQL when an optional filter is absent?

- **A.** A raw copy of the query parameter
- **B.** No condition for that filter, if omission means no restriction
- **C.** A WHERE clause with an empty identifier
- **D.** A condition that always deletes matching rows

**Answer: B.** Optional filters should be added only according to the application's defined semantics. Invalid values should not silently become omitted filters unless that is an intentional policy.

### 4. Can a PDO placeholder represent a column name in ORDER BY?

- **A.** Yes, a placeholder represents any SQL fragment
- **B.** No, placeholders represent values; use an allowlist for identifiers and sort expressions
- **C.** Only if the name is lowercase
- **D.** Only when the table has an index

**Answer: B.** A bound value is not substituted as an SQL identifier. Map accepted sort keys to fixed SQL fragments written by the application.

### 5. Why include a unique tie-breaker such as id in a paginated ORDER BY?

- **A.** To guarantee that equal primary sort values have a defined relative order
- **B.** To make every row have the same sort value
- **C.** To remove the need for WHERE
- **D.** To make LIMIT accept strings

**Answer: A.** If multiple rows tie on the requested sort field, a unique tie-breaker makes their ordering deterministic for a fixed dataset.

### 6. What is the offset for page 4 with 20 rows per page?

- **A.** 20
- **B.** 40
- **C.** 60
- **D.** 80

**Answer: C.** The offset formula is (page - 1) multiplied by page size: (4 - 1) × 20 = 60.

### 7. Why should page size be bounded?

- **A.** To prevent requests from asking for an unreasonably large result set
- **B.** To ensure all database rows have the same value
- **C.** Because page size is an SQL identifier
- **D.** To disable prepared statements

**Answer: A.** A cap helps control query and response resource use. It does not replace authorization, rate limits, or query performance work.

### 8. What does LIMIT do in a paginated query?

- **A.** Restricts how many rows are returned
- **B.** Specifies the database password
- **C.** Sorts rows alphabetically
- **D.** Counts all rows that match

**Answer: A.** LIMIT caps the rows returned by a query. OFFSET controls how many rows to skip first.

### 9. What does OFFSET do?

- **A.** Skips a number of rows from the ordered result before returning the page
- **B.** Changes a table name
- **C.** Adds a database index
- **D.** Counts matching rows

**Answer: A.** OFFSET is used with a row limit to select a later portion of a result set.

### 10. Which query gives a predictable page order?

- **A.** A query with LIMIT and no ORDER BY
- **B.** A query with ORDER BY on a unique ordering, followed by LIMIT and OFFSET
- **C.** A query with OFFSET but no filter
- **D.** A query with LIMIT and a random column

**Answer: B.** SQL does not promise row order without ORDER BY. Add a unique tie-breaker so rows with equal sort values are ordered consistently.

### 11. What should a COUNT query for pagination count?

- **A.** Rows matching the same filters as the displayed result query
- **B.** Every row in every table
- **C.** Only rows on the current page
- **D.** The number of SQL parameters

**Answer: A.** Reuse the same filter conditions and parameter values so the total describes the result set the page query can display.

### 12. If COUNT returns zero, what is the page count using the normal total / page-size calculation?

- **A.** Zero
- **B.** One hundred
- **C.** The page size
- **D.** An SQL identifier

**Answer: A.** No matching rows means zero total pages. The interface can still render an empty state without fetching a nonexistent result page.

### 13. What can happen to offset-based pages when rows are inserted or deleted between requests?

- **A.** Rows can shift, causing a record to appear twice or be skipped
- **B.** SQL automatically preserves a snapshot across all requests
- **C.** PDO prevents all concurrent writes
- **D.** The primary key is changed automatically

**Answer: A.** Each request normally queries the current data. New or removed rows can change which records occupy a given offset.

### 14. Why can a very large OFFSET be inefficient?

- **A.** The database may need to walk past many preceding rows before returning the page
- **B.** OFFSET always deletes skipped rows
- **C.** OFFSET prevents every index from being used
- **D.** PHP loads every database table into memory

**Answer: A.** Deep offset pages can require substantial database work. A keyset/cursor approach can be better for sequential navigation through large results.

### 15. What values should a keyset cursor include for a sort by created_at and id?

- **A.** The values needed to continue both sort keys
- **B.** Only the current page size
- **C.** The SQL password
- **D.** Only the search term

**Answer: A.** A cursor must include all ordering values needed to continue uniquely, including the tie-breaker ID.

### 16. Is keyset pagination a natural fit for jumping directly to arbitrary page 200?

- **A.** Yes, it always computes any page number directly
- **B.** No, it is designed to continue from a cursor, usually for sequential browsing
- **C.** Only when OFFSET is also omitted
- **D.** Only for tables with one row

**Answer: B.** Keyset pagination avoids skipping large offsets, but it does not naturally provide arbitrary page-number jumps or an exact page count.

### 17. How can a named filter value be supplied to both a count statement and a result statement?

- **A.** Bind or execute the matching parameter values on each separately prepared statement
- **B.** Bind it once to one statement and expect the other statement to inherit it
- **C.** Concatenate it into both SQL strings
- **D.** Send the value through ORDER BY

**Answer: A.** Each prepared statement needs its own parameter values. Reuse the same validated data and filter-building logic for both queries.

### 18. What is a reasonable first step if filtering and sorting become slow on a large table?

- **A.** Inspect representative query plans and consider indexes for frequent query patterns
- **B.** Add an index to every column without analysis
- **C.** Fetch every row into PHP
- **D.** Remove all filters

**Answer: A.** Query plans help show how the database executes a real query. Indexes have storage and write costs, so choose them based on actual access patterns.

## Short-answer questions

### 19. Why should filtering usually happen in SQL rather than after fetching all rows into PHP?

**Answer:** The database can return only rows matching the filters and requested page. **Explanation:** This reduces unnecessary data transfer, memory use, and application work.

### 20. What is the difference between validating and binding a request value?

**Answer:** Validation checks whether the value follows application rules; binding keeps it separate from SQL syntax. **Explanation:** A validated value should still be bound when used as query data.

### 21. How should an application distinguish an omitted optional filter from an invalid filter?

**Answer:** Define the semantics explicitly: omission may mean no restriction, while an invalid value should normally be rejected or handled deliberately. **Explanation:** Silently dropping an invalid filter can broaden a query unexpectedly.

### 22. Why might a COUNT query be skipped for some interfaces?

**Answer:** Counting can be expensive, and some interfaces do not need an exact total or page count. **Explanation:** A “load more” experience may only need to know whether another page exists.

### 23. What happens to percent and underscore in a bound LIKE search pattern?

**Answer:** They keep their wildcard meaning even though the pattern is safely bound. **Explanation:** Parameterization protects SQL structure; escape LIKE wildcards separately when the search should treat them literally.

### 24. What makes an ORDER BY deterministic for pagination?

**Answer:** It specifies a complete ordering that ends in a unique tie-breaker, commonly a primary key. **Explanation:** Equal values for the first sort field then have an explicit order.

### 25. What does keyset pagination improve, and what does it not provide naturally?

**Answer:** It can avoid scanning past a large OFFSET for sequential browsing. It does not naturally support arbitrary page jumps or an exact total page count. **Explanation:** A cursor continues from the last seen ordering values.

### 26. How should sort direction input such as asc or desc be used?

**Answer:** Map recognized choices to fixed SQL tokens from an allowlist and reject or default unsupported values. **Explanation:** SQL keywords cannot be supplied through ordinary value placeholders.

### 27. Why bind a LIMIT or OFFSET value with PDO::PARAM_INT when supported?

**Answer:** It makes the intended integer type explicit. **Explanation:** Validate and bound the value first, and check the target driver's support for placeholders in that SQL position.

### 28. How can page limits reduce resource risk?

**Answer:** Clamp page numbers and page sizes to sensible application-defined bounds. **Explanation:** This prevents unreasonable values and arithmetic ranges, but does not replace rate limiting or query tuning.

## Practical scenarios

### 29. A request sends sort=price desc and the code appends that string to ORDER BY. What should change?

**Answer:** Parse a recognized sort key and map it to a fixed, developer-written order expression. **Explanation:** Raw request text can alter SQL structure and cannot be made safe merely by trimming it.

### 30. Several rows share the same created_at timestamp and different pages sometimes reorder them. What should the query add?

**Answer:** Add a unique tie-breaker such as id to ORDER BY. **Explanation:** The timestamp alone does not define the relative order of tied rows.

### 31. A visitor requests per_page=999999. What should the application do?

**Answer:** Validate and clamp the page size to a reasonable maximum or reject the request. **Explanation:** Returning an enormous result can waste database and application resources.

### 32. The displayed rows use category and search filters, but the total count uses only category. What is wrong?

**Answer:** The count does not represent the same result set as the displayed page. **Explanation:** Build the filter conditions once and reuse them, with the same values, for count and result statements.

### 33. Users browse to very deep pages, and query plans show large offsets are costly. What alternative should be considered?

**Answer:** Keyset or cursor pagination using the last row's complete sort values. **Explanation:** This is suited to sequential navigation; it is not a direct replacement when users need arbitrary page-number jumps.

### 34. A requested page is greater than the number of pages after filters are applied. What should the application decide?

**Answer:** Choose a deliberate behavior, such as showing an empty state, redirecting to the last valid page, or returning a validation response. **Explanation:** Do not remove the filters to force the page to contain results.

### 35. The count query and page query run separately while other users are changing records. Why might totals and rows disagree?

**Answer:** The two statements may observe different database states depending on timing and isolation. **Explanation:** If an exact consistent view is required, use an appropriate transaction and isolation level supported by the database, while considering its cost.

### 36. A search filters name with a leading wildcard pattern and becomes slow as the table grows. What should be investigated?

**Answer:** Inspect the query plan and consider an appropriate search index or database full-text feature. **Explanation:** A normal B-tree index may not efficiently support patterns that begin with a wildcard; the right solution depends on the database and search requirements.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [101 - Database transactions and error handling](../lessons/101-database-transactions-and-error-handling.md) | [Lesson](../lessons/102-filtering-sorting-and-paginating-database-results.md) | [Example](../examples/102-filtering-sorting-and-paginating-database-results.md) | [103 - Using the MySQLi extension](../lessons/103-using-the-mysqli-extension.md) |
