# 11. Semi-structured data

## try_parse_json(<string>) function
A special version of parse_json() that returns a NULL, if an error occurs during parsing.

Takes a string as input. The returned value is of type VARIANT, and contains a JSON document.

Supports an input expression with a maximum size of 8 MB compressed.

```sql
create or replace temporary table 
  var_tbl (
    id integer, 
    v varchar
  );

insert into 
  var_tbl (id, v) 
values 
  (1, '[-1, 12, 289, 2188, false,]'), 
  (2, '{ "x" : "abc", "y" : false, "z": 10} '),
  (3, '{ "bad" : "json", "missing" : true, "close_brace": 10 ');

select id, try_parse_json(v) 
from var_tbl
order by id;
-- +----+-------------------+
-- | ID | TRY_PARSE_JSON(V) |
-- |----+-------------------|
-- |  1 | [                 |
-- |    |   -1,             |
-- |    |   12,             |
-- |    |   289,            |
-- |    |   2188,           |
-- |    |   false,          |
-- |    |   undefined       |
-- |    | ]                 |
-- |  2 | {                 |
-- |    |   "x": "abc",     |
-- |    |   "y": false,     |
-- |    |   "z": 10         |
-- |    | }                 |
-- |  3 | NULL              |
-- +----+-------------------+
```

























