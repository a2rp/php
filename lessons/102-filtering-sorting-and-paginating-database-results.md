# 102 - Filtering, sorting, and paginating database results

## Learning goals

By the end of this lesson, you will be able to:

- Build optional filters from fixed SQL fragments and bind every filter value.
- Validate numeric filters, page numbers, and page sizes.
- Map sort choices to a fixed allowlist of SQL order clauses.
- Add a unique tie-breaker so paginated results have a predictable order.
- Use LIMIT and OFFSET with integer parameters.
- Calculate result counts and page metadata.
- Explain the limits of offset pagination and when keyset pagination can help.
- Recognize query performance and concurrent-data considerations.

## 1. Why filter, sort, and paginate in the database?

A list page often lets a visitor narrow records, choose a sort order, and move through pages. The database should do this work before returning rows to PHP:

1. **Filter** with WHERE conditions.
2. **Sort** with ORDER BY.
3. **Select a page** with LIMIT and OFFSET, or an equivalent supported by the database.

Fetching every row and then filtering or sorting in PHP wastes network, database, and application memory. It can also expose records to the application before access checks have been applied.

Pagination limits how many rows one request loads. A page size should be bounded so a request cannot ask the application to return an unreasonably large result set.

## 2. Build optional filters from fixed SQL

The SQL structure should come from application code. Add a condition only when its filter is present, and pass its value separately:

~~~php
$conditions = [];
$parameters = [];

if ($categoryId !== null) {
    $conditions[] = 'category_id = :category_id';
    $parameters['category_id'] = $categoryId;
}

if ($searchText !== '') {
    $conditions[] = 'name LIKE :search_pattern';
    $parameters['search_pattern'] = '%' . $searchText . '%';
}

$whereSql = $conditions === []
    ? ''
    : ' WHERE ' . implode(' AND ', $conditions);

$sql = 'SELECT id, name, category_id
        FROM products'
    . $whereSql
    . ' ORDER BY id';

$statement = $pdo->prepare($sql);
$statement->execute($parameters);
$products = $statement->fetchAll();
~~~

Only fixed condition strings are added to SQL. Filter contents stay in the parameter array.

For multiple filters, decide which combinations are valid. An omitted filter may mean “do not restrict by this field”; an invalid value should usually be rejected rather than silently treated as omitted.

## 3. Validate request values before using them

Request parameters arrive as untrusted strings and can also have unexpected shapes. Validate their type and meaning before using them:

~~~php
$categoryInput = $_GET['category_id'] ?? null;

if ($categoryInput !== null && $categoryInput !== '') {
    if (!is_string($categoryInput) && !is_int($categoryInput)) {
        throw new InvalidArgumentException('Category must be a single value.');
    }

    $categoryId = filter_var($categoryInput, FILTER_VALIDATE_INT);

    if ($categoryId === false || $categoryId < 1) {
        throw new InvalidArgumentException('Category must be a positive integer.');
    }
} else {
    $categoryId = null;
}
~~~

Validation answers whether a value is allowed by application rules. Binding answers whether the value is kept separate from SQL syntax. Use both.

For text search, require a single string, trim it if appropriate, and apply a maximum length. Decide whether an empty search means no text filter or should be rejected.

A bound LIKE pattern still treats percent and underscore as wildcards. If the interface promises literal substring matching, escape those characters using the target database's LIKE rules and a matching ESCAPE clause.

## 4. Keep sort choices on an allowlist

A placeholder can bind a value, but it cannot stand for an SQL identifier or keyword. Do not place a raw request value in ORDER BY:

~~~php
$sortOptions = [
    'newest' => 'created_at DESC, id DESC',
    'oldest' => 'created_at ASC, id ASC',
    'name' => 'name ASC, id ASC',
];

$sortKey = $_GET['sort'] ?? 'newest';
$orderBy = $sortOptions[$sortKey] ?? $sortOptions['newest'];

$sql = 'SELECT id, name, created_at
        FROM products
        ORDER BY ' . $orderBy;
~~~

The map contains only developer-written SQL fragments. A request chooses a key; it cannot insert its own column, direction, or SQL expression. Reject unsupported choices instead of falling back if the interface needs to tell users their choice was invalid.

Include a unique tie-breaker such as the primary key. If several products have the same created_at timestamp or name, ordering only by that column leaves their relative order unspecified. The additional id makes the selected ordering unique.

## 5. Calculate and bound page parameters

Page numbers normally begin at 1. A simple offset formula is:

~~~text
offset = (page - 1) * page_size
~~~

For example, page 4 with 20 rows per page skips 60 rows.

Validate page and page-size inputs, and impose reasonable maximums:

~~~php
function boundedPositiveInt(
    mixed $input,
    int $default,
    int $maximum
): int {
    if (!is_string($input) && !is_int($input)) {
        return $default;
    }

    $value = filter_var($input, FILTER_VALIDATE_INT);

    if ($value === false) {
        return $default;
    }

    return max(1, min($maximum, $value));
}

$page = boundedPositiveInt($_GET['page'] ?? null, 1, 10000);
$pageSize = boundedPositiveInt($_GET['per_page'] ?? null, 20, 100);

$offset = ($page - 1) * $pageSize;
~~~

This helper accepts one scalar integer-like value, substitutes a default for invalid values, and clamps valid values to the configured maximum. The exact page and page-size limits are product decisions. They help control resource use; they do not replace rate limiting or authorization.

If the product rejects invalid page values instead of normalizing them, return a clear validation error rather than silently changing the request.

## 6. Bind LIMIT and OFFSET as integers

LIMIT and OFFSET are part of the SQL grammar, but common database drivers allow them to be supplied as integer parameters. Bind them as PDO::PARAM_INT:

~~~php
$statement = $pdo->prepare(
    'SELECT id, name
     FROM products
     ORDER BY created_at DESC, id DESC
     LIMIT :page_size OFFSET :row_offset'
);

$statement->bindValue(':page_size', $pageSize, PDO::PARAM_INT);
$statement->bindValue(':row_offset', $offset, PDO::PARAM_INT);
$statement->execute();

$products = $statement->fetchAll();
~~~

Parameter support and exact syntax can differ among database drivers. Check the target driver's behavior. If a driver does not support a placeholder in a particular position, validate and clamp the integer first, then use the driver's documented safe approach; never insert an unvalidated request string into SQL.

## 7. Count matching rows for page controls

If the interface displays a total count or number of pages, run a COUNT query with the same filters as the result query:

~~~php
$countStatement = $pdo->prepare(
    'SELECT COUNT(*)
     FROM products'
    . $whereSql
);
$countStatement->execute($parameters);
$totalRows = (int) $countStatement->fetchColumn();

$totalPages = (int) ceil($totalRows / $pageSize);
~~~

Use the same WHERE conditions and parameter values so the count describes the filtered result set. If the count is zero, totalPages is zero; the interface can still display an empty state without trying to fetch a nonexistent page.

A separate COUNT can be expensive for large or complex queries. Calculate it only when the interface needs it. If records change between the count and result queries, their results may describe slightly different snapshots, depending on the database and transaction isolation.

When there are no results, show an intentional empty state. Do not remove filters merely because the current page is empty.

## 8. Use deterministic ordering with pagination

LIMIT without ORDER BY does not identify a predictable subset. Database row order is not guaranteed unless a sort is specified.

Even a non-unique sort is incomplete for pagination. If multiple rows have equal values, add a unique column:

~~~sql
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40
~~~

This sorts by creation time and then by ID. It gives the query a stable order for a fixed dataset.

A stable order does not freeze the dataset between requests. If rows are inserted or deleted while someone is moving between offset pages, records may shift and appear twice or be skipped. The application should decide whether that behavior is acceptable for a browsable list.

## 9. Know when offset pagination is not a good fit

Offset pagination is simple and works well for small or moderate page numbers. For a very large offset, the database may need to walk past many earlier rows before returning the requested page.

Keyset pagination, also called cursor pagination, continues from the last sort key of the previous page instead of skipping a large offset.

For a descending, unique ordering by created_at and id, a later page can use a condition such as:

~~~sql
WHERE created_at < :last_created_at
   OR (created_at = :last_created_at AND id < :last_id)
ORDER BY created_at DESC, id DESC
LIMIT :page_size
~~~

Bind both cursor values. The cursor must contain every ordering value needed to continue uniquely, including the tie-breaker. Validate or authenticate client-provided cursor data according to the application's needs.

Keyset pagination is useful for sequential browsing of large or changing datasets. It does not naturally support jumping directly to an arbitrary page number or showing an exact page count. It also does not by itself provide a frozen snapshot of changing data.

## 10. Keep filter, count, and result queries aligned

A practical list operation often needs:

- The filtered result query.
- A count query for those same filters, if the interface needs totals.
- The same validated filter values in both queries.
- A consistent ordering and pagination rule.
- Authorization conditions that restrict which records the current user may see.

Authorization belongs in the query or in a carefully enforced application check; a safely bound filter does not grant permission to see a record.

When query construction becomes complex, keep the SQL fragments and their parameter values together in a small, well-tested repository method. Avoid duplicating filter logic in separate count and result code paths, where one can accidentally omit a condition.

## 11. Consider indexes and query plans

Filtering and sorting can become slow as a table grows. Indexes can help when their columns and order match frequent filters and sorts, but indexes also consume storage and make writes more expensive.

Use the database's query-plan tools with representative data. Check actual query patterns rather than adding indexes for every possible combination. Composite indexes may help a common filter-and-sort combination, but their usefulness depends on the database and the order of columns.

Searches with a leading wildcard pattern such as percent followed by text often cannot use a simple B-tree index efficiently. Large text-search features may need a database-specific full-text index or search system.

## Common mistakes

### Fetching every row and slicing in PHP

This uses unnecessary memory and work. Apply filters, ordering, and page limits in the database.

### Using raw request data in ORDER BY

Placeholders represent values, not identifiers or SQL keywords. Map allowed sort keys to fixed SQL fragments.

### Paginating without a unique order

Rows can move between pages or appear in an unspecified order. Sort by the requested field and a unique tie-breaker.

### Accepting an unlimited page size

A very large request can make the application perform expensive queries and return too much data. Validate and clamp page size.

### Reusing a count query with different filters

The total can then disagree with the displayed rows. Build the filter conditions once and use the same values for both queries.

### Assuming OFFSET is free

Large offsets can require the database to scan past many rows. Consider keyset pagination for deep sequential browsing.

### Assuming a page stays unchanged while the user navigates

Inserts and deletes can shift offset-based pages. Decide whether live changing data is acceptable or whether a snapshot/cursor experience is needed.

## Key takeaways

- Filter in SQL with fixed conditions and bound values.
- Validate request values and enforce sensible page and page-size bounds.
- Map sort keys to developer-written SQL fragments; never interpolate raw sort input.
- Add a unique tie-breaker to every paginated sort order.
- Use LIMIT and OFFSET with validated integer parameters where the driver supports them.
- Reuse the same filters for total counts and result rows.
- Offset pagination is straightforward but large offsets can be expensive and changing rows can shift pages.
- Consider keyset pagination for deep, sequential navigation through large or frequently changing data.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [101 - Database transactions and error handling](101-database-transactions-and-error-handling.md) | [Q&A](../qnas/102-filtering-sorting-and-paginating-database-results.md) | [Example](../examples/102-filtering-sorting-and-paginating-database-results.md) | [103 - Using the MySQLi extension](103-using-the-mysqli-extension.md) |
