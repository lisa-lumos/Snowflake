# 6. Tables, Views, Sequences

## create materialized view
```sql
CREATE [OR REPLACE] [SECURE] MATERIALIZED VIEW [IF NOT EXISTS] <name>
[COPY GRANTS]
[<col1> 
  MASKING POLICY <policy_name> [USING (<col1>,  ... )]
  TAG (<tag_name> = '<tag_value>', ...)
]
[<col2> ...]
[ROW ACCESS POLICY <policy_name> ON (<col_name>, ...]
[TAG (<tag_name> = '<tag_value>', ...]
[COMMENT = '<string_literal>']
[CLUSTER BY (<expr1> ...)]
AS 
  <select_statement>
```

Creating a materialized view requires CREATE MATERIALIZED VIEW privilege on the schema, and SELECT privilege on the base table. 

When specifying the select_statement, you cannot specify a HAVING clause or an ORDER BY clause.

```sql
create materialized view 
  mymv
comment='test view'
as
  select col1, col2 from mytable;
```

When creating a materialized view with a masking policy, or a row access policy, use the POLICY_CONTEXT() function to simulate a query to validate.























