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




## Querying hierarchical data


## CTEs


## Window functions


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



























