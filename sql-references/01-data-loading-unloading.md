# 1. Data Loading & Unloading

## COPY INTO table
Standard data load: 
```sql
copy into 
  [<namespace>.]<table_name>
from 
  internalStage | externalStage | externalLocation
[files = ('<file_name>', ...)]
[pattern = '<regex_pattern>']
[file_format = ( 
  format_name = '[<namespace>.]<file_format_name>' |
  type = csv | json | avro | orc | parquet | xml [formattypeoptions]  
)]
[copyOptions]
[validation_mode = return_<n>_rows | return_errors | return_all_errors]
```

Data load with transformation: 
```sql
copy into 
  [<namespace>.]<table_name> [(<col_name>, ...)]
from ( 
  select 
    [<alias>.]$<file_col_num>[.<element>], ...
  from 
    internalStage | externalStage [<alias>]
)
[files = ('<file_name>', ...)]
[pattern = '<regex_pattern>']
[file_format = ( 
  format_name = '[<namespace>.]<file_format_name>' |
  type = csv | json | avro | orc | parquet | xml [formattypeoptions] 
)]
[copyOptions]
```
Note that in the target table, any columns excluded from this column list are populated by their default value (NULL, if not specified). However, excluded columns cannot have a sequence as their default value.

In the FILES list, the max num of files names that can be specified is 1000. For external stages, the file path is set by concatenating the URL in the stage definition and the list of resolved file names. However, Snowflake doesn't insert a separator implicitly between the path and file names. You must explicitly include a separator (/), either at the end of the URL in the stage definition, or at the beginning of each file name specified in this parameter.

For the best performance, try to avoid applying patterns that filter on a large number of files. Regular expression is applied differently to bulk data loads versus Snowpipe data loads: 
- Snowpipe ignores any path info contained in the stage definition, and applies the regular expression to any remaining path segments and filenames. DESCRIBE STAGE for the stage shows its stage definition. Eg, if the FROM clause has @s/path1/path2/, and the URL value for stage @s is s3://mybucket/path1/, then Snowpipe trims /path1/ from the FROM clause and applies the regular expression to path2/ plus the filenames.
- Bulk data load operations apply the regular expression to the entire storage location in the FROM clause.

FORMAT_NAME and TYPE are mutually exclusive; specifying both in the same COPY command might result in unexpected behavior.

validation_mode: instructs the COPY command to validate the data files, instead of loading them into the specified table:
- RETURN_n_ROWS (eg, RETURN_10_ROWS): Validates first n rows, fails at the first error encountered in them. 
- RETURN_ERRORS: Returns all errors across all files specified in the COPY statement.
- RETURN_ALL_ERRORS: Returns all errors across all files specified in the COPY statement, including files with errors that were partially loaded in an earlier load, when its ON_ERROR copy option was set to CONTINUE.

VALIDATION_MODE does not support COPY statements that transform data during a load.

Use the VALIDATE table function to view all errors encountered during a previous load.

```sql
formatTypeOptions ::=
  -- if file_format = ( type = csv ... )
  compression = auto | gzip | bz2 | brotli | zstd | deflate | raw_deflate | none
  record_delimiter = '<character>' | none -- default: new line character
  field_delimiter = '<character>' | none -- default: ;
  skip_header = <integer> -- num of lines to skip. default: 0
  skip_blank_lines = true | false -- default: false
  date_format = '<string>' | auto -- default: auto (use date_input_format session param)
  time_format = '<string>' | auto-- default: auto (use time_input_format session param)
  timestamp_format = '<string>' | auto -- default: auto (use timestamp_input_format session param)
  binary_format = hex | base64 | utf8 -- encoding format for binary io when load data into bineary columns. default: hex
  escape = '<character>' | none -- escape char for enclosed field vals. default: none
  escape_unenclosed_field = '<character>' | none -- escape char for un-enclosed field vals. default: \\ 
  trim_space = true | false -- rmv white space before&after field delimiters. default: false
  field_optionally_enclosed_by = '<character>' | none -- character that encloses strings. default: none
  null_if = ( '<string>' [ , '<string>' ... ] ) -- string to convert to/from sql NULL. default: \\n
  error_on_column_count_mismatch = true | false -- whether generate error. default: true
  replace_invalid_characters = true | false -- default: false
  empty_field_as_null = true | false -- default: true
  skip_byte_order_mark = true | false -- default: true
  encoding = '<string>' | utf8 -- default: utf8

  -- if file_format = ( type = json ... )
  compression = auto | gzip | bz2 | brotli | zstd | deflate | raw_deflate | none
  date_format = '<string>' | auto
  time_format = '<string>' | auto
  timestamp_format = '<string>' | auto
  binary_format = hex | base64 | utf8
  trim_space = true | false
  null_if = ( '<string>' [ , '<string>' ... ] )
  enable_octal = true | false -- default: false
  allow_duplicate = true | false -- allow duplicate oject field names. default: false
  strip_outer_array = true | false -- rmv outer []. default: false
  strip_null_values = true | false -- treat null as sql NULL. default: false
  replace_invalid_characters = true | false
  ignore_utf8_errors = true | false -- default: false
  skip_byte_order_mark = true | false

  -- if file_format = ( type = avro ... )
  compression = auto | gzip | brotli | zstd | deflate | raw_deflate | none
  trim_space = true | false
  null_if = ( '<string>' [ , '<string>' ... ] )

  -- if file_format = ( type = orc ... )
  trim_space = true | false
  null_if = ( '<string>' [ , '<string>' ... ] )

  -- if file_format = ( type = parquet ... )
  compression = auto | snappy | none
  binary_as_text = true | false
  trim_space = true | false
  null_if = ( '<string>' [ , '<string>' ... ] )

  -- if file_format = ( type = xml ... )
  compression = auto | gzip | bz2 | brotli | zstd | deflate | raw_deflate | none
  ignore_utf8_errors = true | false
  preserve_space = true | false -- default: false
  strip_outer_element = true | false -- default: false
  disable_snowflake_data = true | false -- default: false
  disable_auto_convert = true | false -- default: false
  skip_byte_order_mark = true | false
```

```sql
copyOptions ::=
  on_error = 
    continue -- continue to load the file
    | skip_file          -- skip this file (default for snowpipe)
    | skip_file_<n>      -- skip this file if >= n rows have error
    | 'skip_file_<num>%' -- skip this file if >= n% rows have error
    | abort_statement -- abort the query (default for bulk loading)
  size_limit = <num> -- max bytes of files to load. default: null (no limit)
  purge = true | false -- rmv data files after load. default: false
  return_failed_only = true | false -- whether only return failed files. default: false
  match_by_column_name = case_sensitive | case_insensitive | none -- supports JSON, Avro, ORC, Parquet formats. default: none
  enforce_length = true | false -- truncate strings exceed target col length. default: true
  truncatecolumns = true | false -- same as above. default: false
  force = true | false -- ignore metadata for loaded files. default: false
  load_uncertain_files = true | false -- load files with unknown status. default: false
```

Examples:
```sql
copy into mytable
from @my_internal_stage;

copy into mytable -- use table stage
file_format = (type = csv);

copy into mytable from @~/staged -- use user stage
file_format = (format_name = 'mycsv');

copy into mycsvtable -- use external stage
  from @my_ext_stage/tutorials/dataloading/contacts1.csv;

copy into mytable
  from @my_ext_stage/tutorials/dataloading/sales.json.gz
  file_format = (type = 'json')
  match_by_column_name='case_insensitive';

copy into mytable
  from s3://mybucket/data/files
  storage_integration = myint
  encryption=(master_key = 'es...')
  file_format = (format_name = my_csv_format);

copy into mytable
  from s3://mybucket/data/files
  credentials=(aws_key_id='$aws_access_key_id' aws_secret_key='$aws_secret_access_key')
  encryption=(master_key = 'es...')
  file_format = (format_name = my_csv_format);

copy into mytable
  file_format = (type = 'csv')
  pattern='.*/.*/.*[.]csv[.]gz';

copy into mytable
  file_format = (format_name = myformat)
  pattern='.*sales.*[.]csv';

create or replace file format json_format
  type = 'json'
  strip_outer_array = true;
create or replace stage mystage
  file_format = json_format;
put file:///tmp/sales.json @mystage auto_compress=true;
create or replace table house_sales (src variant);
copy into house_sales
   from @mystage/sales.json.gz;

copy into load1 from @%load1/data1/
    files=('test1.csv', 'test2.csv');
copy into load1 from @%load1/data1/
    files=('test1.csv', 'test2.csv')
    force=true;

alter table mytable set stage_copy_options = (purge = true);
copy into mytable;

copy into mytable validation_mode = 'return_errors';

copy into mytable validation_mode = 'return_2_rows';

```













































