PQL
===

\
 PQL is an SQL-like language that can be used for quering and filtering
data in PaperTrail it can be used in the following places:

-   Anywhere where a query expression is used e.g. Advanced Search
    Reports, Queues, Scheduled Rules, Document linking etc..
-   As an alternative to groovy based expressions on node rule filters

e.g. instead of:\
 `filename == 'filename && a == 'b'`\
 Use:\
 `WHERE filename = 'filename' AND a = 'b'`\
\
 PQL filters are fully case-insensitive and null safe so instead of:
`index1 != null && index1.toLowerCase() == 'abc'  `\
 Use:\
 `WHERE index1 = 'abc'`

Syntax
------

The SELECT clause MUST contain exactly one of the following:

1.  A comma-separated list of one or more column names (node or document
    index names).\
     You can use aliases to rename returned columns, e.g.
    `column AS alias`.
    -   If an explicit column list is provided: Only the columns listed
        will be returned and only in the order supplied

     
    -   **\*** : All standard indexes and any custom indexes will be
        returned in the default order
    -   **\*\***: Only custom indexes will be returned 

2.  One or more calls to aggregate functions.

    Aggregate functions produce a single row output from multiple rows
    in a group.\
      

Column Expressions
------------------

Groovy expressions can be used to format data returned e.g. \
\
 `SELECT '${name[0]}' as Initial FROM Clients`{.sql}

Multiple columns can also be referenced:\
 `SELECT '${LastName}, ${FirstName}' as FullName FROM Clients`{.sql} \
\
 As well as arithmetic on *Number* and *Double* indexes:\
 `SELECT '${total + vat}' as GrandTotal FROM 'Sales'`{.sql}

\
 FROM Clause
------------

The FROM clause identifies which Virtual Table (Node) the query will be
run against, as described in the previous section.

The FROM clause MUST contain the full path of a node, and MUST be single
quoted if there are any spaces .e.g\
\
 FROM parent/division\
 FROM 'parent/sub folder/folder'\
 FROM '@{VirtualDataSource}

FROM @WorkflowHistory
---------------------

\
 The workflow history virtual data source will return details about the
unassigned and human tasks of a one or more documents e.g. \
\
 `SELECT * FROM '@WorkflowHistory' WHERE docId = 1`\
 Will return the following special columns:\
  

  ------------------------------------ ------------------------------------
  **Column**                           allocation
  **Description**                      The final user allocated
  ------------------------------------ ------------------------------------

Any standard and custom indexes can also be merged into the results by
adding them into the column list.\
 Any standard search criteria can also be used

FROM '@Activity'
----------------

 

FROM '@QueueHistory"
--------------------

WHERE
-----

This clause identifies the constraints that rows MUST satisfy to be
considered a result for the query.

All column names MUST be valid “queryName” or their aliased values for
properties that are defined as “queryable” in the Object-Type(s) whose
Virtual Tables are listed in the FROM clause.

Properties are defined to not support a “null” value, therefore the
\<null predicate\> MUST be interpreted as testing the not set or set
state of the specified property.

 Comparisons permitted in the WHERE clause.

PQL supports the following predicates:

-   = (equals)
-   \> (bigger than, after)
-   \< (smaller than, before)

    Note: Bigger than / Less than and end equal to (\>=, \<=) are not
    supported

-   contains
-   startsWith
-   endWith
-   BETWEEN and NOT BETWEEN predicates to compare on ranges

    e.g. `column BETWEEN a AND b`, is equivalent to
    `column >= a AND column <= b` 

-   IN
-   LIKE 
-   IS NULL
-   IS NOT NULL
-   all\_empty

    e.g. a filter that matches when all 4 indexes are populated\
     `WHERE not all_empty (invoice_no,total,date,approved)`{.sql}

-   not all\_empty
-   before, after\
     `WHERE createdDate BEFORE '2015-01-01'`{.sql}

WHERE Expressions
-----------------

\
 Expressions can also be used within WHERE clauses. e.g.\
\
 `SELECT name FROM Customers WHERE '${name[0]}' = 'A'`{.sql}

ORDER BY
--------

This clause MUST a single column to order by\
 All column names referenced in this clause MUST be valid “queryName” or
alias (either for an aggregate function result or a column).\
 Only columns in the SELECT clause MAY be in the ORDER BY clause.

GROUP BY
--------

This clause specifies one or many columns to group by.

Supported functions:

-   `COUNT(column)`, `COUNT(*)` - returns a number of entries in a
    group. `COUNT(column)` skips null values.
-   `AVG(column)` - returns an average value of a column in a group.
-   `MIN(column), MAX(column)` - return a minimal or maximal value of a
    certain column in a group.
-   `SUM(column)` - returns a sum of a column in a group.

Escaping
--------

Repositories MUST support the escaping of characters using a backslash
(\\) in the query statement.  The backslash character (\\) will be used
to escape characters within quoted strings in the query as follows:

1.  \\’ will represent a single-quote(‘) character
2.  \\ \\ will represent a backslash (\\) character
3.  Within a LIKE string, \\% and \\\_ will represent the literal
    characters % and \_, respectively.
4.  All other instances of a \\ are errors.

