# Data Integration
## Snowflake Openflow
Snowflake Openflow is an integration service that connects any data source and any destination with hundreds of processors supporting structured and unstructured text, images, audio, video and sensor data.

## Apache Iceberg
Apache Iceberg tables. Combine the performance and query semantics of typical Snowflake tables with external cloud storage that you manage. They are ideal for existing data lakes that you cannot, or choose not to, store in Snowflake.

External storage. Not part of Snowflake. Snowflake does not provide Fail-safe storage.

External volume. A named, account-level Snowflake object to connect Snowflake to your external cloud storage. Stores an identity, and IAM entity. Used to access table data, Iceberg metadata, and manifest files (containing schema/partitions/metadata). S3 bucket name cannot contain dots. 

Catalog. Allows the compute to manage/load tables:
- current metadata pointer: table name -> location of table's current metadata file
- perform atomic operations, to update the current metadata pointer for a table. 

Catalog integration. A named, account-level Snowflake object, that stores info about how your table metadata is organized, whether you table is managed by AWS Glue, or Snowflake Open Catalog, etc. Support many tables. 

Billing. Compute, cloud services, automated refresh, etc. 

you can create multiple external volumes to secure various storage locations differently. e.g.:
- A read-only external volume, for externally managed Iceberg tables.
- An external volume configured with read and write access, for Snowflake-managed tables.

Perform frequent refreshes on Iceberg tables that use an external catalog.

Refresh the metadata each time you perform a maintenance operation, such as snapshot expiration, or compaction. 

Make sure your Parquet file statistics are as complete as possible. Missing statistics like the following degrade query performance:
- Minimum and maximum values.
- Number of distinct values (NDV). The number of distinct values is used to determine the join order in complex joins. Missing NDV statistics can lead to join explosion.
- Number of NULL counts.

Snowflake attempts to read statistics from the table manifest files, to provide faster performance. In some situations, such as when there are missing/incorrect statistics in the manifest files, Snowflake scans the table data files for statistics. Scanning a large number of data files can slow down table creation. 
















