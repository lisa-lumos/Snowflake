# Snowflake Horizon Catalog
## Use Snowsight to generate object descriptions
Can automatically generate descriptions for a column/table/view, and save it in the comment property. 

The Cortex Powered Object Descriptions feature uses Snowflake-hosted LLMs to evaluate object metadata, and optionally sample data to generate the description.

## Use SQL to generate object descriptions

```sql
call ai_generate_table_desc( 'my_table');
```

To set a generated description as a comment on a table/view/column, you must manually execute a SQL statement that includes the SET COMMENT parameter. 






























