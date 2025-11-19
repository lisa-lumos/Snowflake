# Storage Lifecycle Policies
A storage lifecycle policy is a schema-level object that automatically manages the data lifecycle for standard Snowflake tables. Use these policies to archive/expire specific table rows based on conditions that you define, such as data age or other criteria. Snowflake automatically executes these policies daily using shared compute resources.

When the policy runs, it checks each row against your expression and either archives data to COOL or COLD storage, or expires it (deletes it permanently).

Archive storage tiers:
- Cool: fast retrieval time, min archival period is 90d. 
- Cold: cost savings than cool, long retrieval time. 

When you attach a storage lifecycle policy to a table, Snowflake waits approximately 24 hours before running it for the first time.

Snowflake runs storage lifecycle policies every day by using shared compute resources.

After Snowflake archives rows, you can't query them directly.

Truncating a table doesn't affect archived data for that table. You can still retrieve data from archive storage after truncating the table.





















