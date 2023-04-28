# 8. Queries
## Joins
Snowflake supports the following types of joins:
- inner join (only return matched rows)
- outer join (= left/right/full-outer join. return all rows, including those without a match)
- cross join (return Cartesian product, rarely used)
- natural join (= inner join without on clause, so it implicitly use same-named cols from two tables, can be risky to use)

### Eliminate Redundant Joins
If you have UNIQUE, PRIMARY KEY, FOREIGN KEY constraints defined on tables, and your data does comply with these constraints, then if you specify `rely` on these constraints, sf can use them to eliminate unnecessary joins (where the data in the joined table is not actually used in what you select in the SELECT, or, even if used, could get the same result without a join). 

## Subqueries
A subquery is a query within another query. 

The only subquery that allows a LIMIT/FETCH clause is an uncorrelated scalar subquery. But, because an uncorrelated scalar subquery returns only 1 row, the LIMIT clause is basically meaningless. 

## Querying hierarchical data
Many types of data are best represented as a hierarchy, such as an organization tree. Employees are usually organized in a hierarchy, with a company President at the top of the hierarchy. Another example of a hierarchy is a "parts explosion". For example, a car contains an engine; an engine contains a fuel pump; and a fuel pump contains a hose.

You can store hierarchical data in:
- A hierarchy of tables.
- A single table with column(s) representing the hierarchy,e.g. indicating each employee's direct manager.
- semi-structured data

If the number of levels is unknown/changing, so that it is not possible to create a hierarchy with a known number of tables, then the hierarchical data can be stored in one table. But if the data at different levels doesn't share the same record structure, then storing all in one table might not be practical.

Ways to query them in relational table(s):
- if num of levels known, use joins
- if num of levels unknown, use recursive CTE or CONNECT BY

CONNECT BY only allow for self-joins. Recursive CTE is more flexible, allows any joins. 

You can use recursive CTEs and CONNECT BY on a single table that contains multiple trees, but you can only query one tree at a time, and that tree must be contiguous.

## CTEs
Common table expressions, a named subquery defined in a WITH clause. CTEs increase modularity and simplify maintenance.

If CTE has same name with a db object, then the CTE name takes precedence. But try not to use duplicate names to avoid confusion. 

Constructing a recursive CTE incorrectly can cause an infinite loop.

## Window functions
read, skipped. Queries see Chapter 0. 

## Match recognize




## Persisted query results

## Distinct counts

## Similarity estimation

## Frequency estimation

## Estimating percentile values

## Query profile

## Cancel statements





In query history, some queries do not have warehouse size, this is because they used cache, and never required a warehouse. If a client shows Go, it means this query started from SnowSight, because it was written in the Go language. If a query was initiated in the SnowSQL, it will show Python, because SnowSQL is written in python. If from Tableau, it will show JDBC. 

As to query tag, if you are from IT team, and need to debug/optimize queries, you can tag these queries using alter session, and put ticket/Jira number in it, so the prod department know where these additional compute cost came from. 



























