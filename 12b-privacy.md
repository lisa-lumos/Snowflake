# Privacy
## Aggregation policies
An aggregation policy is a schema-level object, that controls what type of query can access data from a table/view. 

When an aggregation policy is applied to a table, queries against that table must aggregate data into groups of a minimum size in order to return results, thereby preventing a query from returning information from an individual record. A table or view with an aggregation policy assigned to it is said to be aggregation-constrained.

Aggregation policies allow a provider (data owner) to exercise control over what can be done with their data, even after it is shared with a consumer.

### entity-level privacy
Entity-level privacy strengthens the privacy protections provided by aggregation policies. With entity-level privacy, Snowflake can ensure that an aggregation group contains a certain number of entities, not just a certain number of rows.

To achieve entity-level privacy, Snowflake allows you to specify which attributes can be used to identify an entity (an entity key). This lets Snowflake identify all of the records that belong to a particular entity within a dataset. 

For example, if the entity key is defined as the column email, then Snowflake can determine that all records where email=joe.smith@example.com belong to the same entity.

## Projection policies




























