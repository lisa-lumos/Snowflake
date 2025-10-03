# Snowflake Horizon Catalog
## Use Snowsight to generate object descriptions
Can automatically generate descriptions for a column/table/view, and save it in the comment property. 

The Cortex Powered Object Descriptions feature uses Snowflake-hosted LLMs to evaluate object metadata, and optionally sample data to generate the description.

## Use SQL to generate object descriptions

```sql
call ai_generate_table_desc( 'my_table');
```

To set a generated description as a comment on a table/view/column, you must manually execute a SQL statement that includes the SET COMMENT parameter. 

## Using contacts
Contacts are schema-level objects, that contain details about how to communicate with a user, or group of users.

For example, one contact named data_stewards might include an email distribution list, while another named support_department might include the URL of the department's website.

The purpose of the contact the association between a contact and a specific object. An object can have more than one contact as long as the purpose of each contact is unique for the object. 
