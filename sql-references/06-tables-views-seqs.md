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

When specifying the select statement, you cannot specify a HAVING clause or an ORDER BY clause.

```sql
create materialized view 
  mymv
comment='test view'
as
  select col1, col2 from mytable;
```

When creating a materialized view with a masking policy, or a row access policy, use the POLICY_CONTEXT() function to simulate a query to validate.


## create external table
When queried, an external table reads data from a set of files in an external stage and outputs the data in a single VARIANT column.

For partitions computed from expressions: 
```sql
create [ or replace ] external table [if not exists]
  <table_name> (
    [<part_col_name> <col_type> as <part_expr> ] [inlineconstraint], 
    ... 
  )
cloudproviderparams -- notification integration
[partition by (<part_col_name>, ... )] 
location = externalstage
[refresh_on_create =  true | false] -- default: true
[auto_refresh = true | false] -- default: true
[pattern = '<regex_pattern>']
file_format = ( 
  format_name = '<file_format_name>' | 
  type = { csv | json | avro | orc | parquet } [formattypeoptions] 
)
[aws_sns_topic = '<string>'] -- for auto-refresh
[copy grants] -- when recreate the table, copy original grants
[row access policy <policy_name> on (value) ]
[tag (<tag_name> = '<tag_value>', ...) ]
[comment = '<string_literal>' ]
```

The external table does not inherit the file format in the stage definition. You must explicitly specify any file format options for the external table using the FILE_FORMAT parameter.

Refreshing the external table metadata synchronizes the metadata with the current list of data files in the specified stage path. If the specified location contains close to 1 million files or more, we recommend that you set REFRESH_ON_CREATE = FALSE. After creating the external table, refresh the metadata incrementally by executing ALTER EXTERNAL TABLE ... REFRESH statements that specify subpaths in the location (i.e. subsets of files to include in the refresh) until the metadata includes all of the files in the location.

AUTO_REFRESH Specifies whether Snowflake enable automatic refreshes of the external table metadata when new or updated data files are available in the named external stage specified in the LOCATION = .... You must configure an event notification for your storage location to notify Snowflake when new or updated data is available to read into the external table metadata.

PARTITION_TYPE = USER_SPECIFIED Defines the partition type for the external table as user-defined. The owner of the external table must add partitions to the external metadata manually by executing ALTER EXTERNAL TABLE ... ADD PARTITION statements.

External tables support external stages only; internal stages are not supported.

External tables include the following metadata column:
- METADATA$FILENAME: Name of each staged data file included in the external table. Includes the path to the data file in the stage.
- METADATA$FILE_ROW_NUMBER: Row number for each record in the staged data file.

The following are not supported for external tables:
- Clustering keys
- Cloning
- Data in XML format

Time Travel is not supported for external tables.

You cannot add a masking policy to an external table column other than the VALUE column, because a masking policy cannot be attached to a virtual column.

You can add a row access policy to an external table while creating the external table.

```sql
-- add partitions automatically
create stage s1 url='s3://mybucket/files/logs/' ...;
select metadata$filename from @s1/;
create external table et1(
  date_part date as 
    to_date(
      split_part(metadata$filename, '/', 3)
      || '/' || split_part(metadata$filename, '/', 4)
      || '/' || split_part(metadata$filename, '/', 5), 'yyyy/mm/dd'
    ),
  timestamp bigint as (value:timestamp::bigint),
  col2 varchar as (value:col2::varchar))
partition by (date_part)
location=@s1/logs/
auto_refresh = true
file_format = (type = parquet)
aws_sns_topic = 'arn:aws:sns:us-west-2:001234567890:s3_mybucket'
;
alter external table et1 refresh;
select timestamp, col2 from et1 where date_part = to_date('08/05/2018');

-- add partitions manually
create stage s1 url='s3://mybucket/files/logs/' ...;
create external table et2(
  col1 date as (parse_json(metadata$external_table_partition):col1::date),
  col2 varchar as (parse_json(metadata$external_table_partition):col2::varchar),
  col3 number as (parse_json(metadata$external_table_partition):col3::number))
partition by (col1,col2,col3)
location=@s2/logs/
partition_type = user_specified
file_format = (type = parquet)
;
alter external table et2 add partition(col1='2022-01-24', col2='a', col3='12') location '2022/01';
select col1, col2, col3 from et1 where col1 = to_date('2022-01-24') and col2 = 'a' order by metadata$file_row_number;

create materialized view 
  et1_mv -- materialized view on a external table
as
  select col2 from et1;

create external table mytable -- detect col definitions
using template (
  select 
    array_agg(object_construct(*))
  from table(
    infer_schema(
      location=>'@mystage',
      file_format=>'my_parquet_format'
    )
  )
)
location=@mystage
file_format=my_parquet_format
auto_refresh=false;
```






















