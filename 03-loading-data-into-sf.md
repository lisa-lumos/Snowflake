# 3. Loading data into SF

Snowflake supports following data transformations while loading it into a table:
- Column reordering
- Column omission
- Casts
- Truncating text strings that exceed the target column length

For semi-structured data (Apache Parquet, Apache Avro, and ORC files), snowflake supports auto detection of schema. 

For files to load to SF, the default encoding character set is UTF-8. gzip is the default compression format for uncompressed files to be loaded to SF stage. Compression format can be auto detected for certain formats, if the files are  compressed from the beginning. The files sits in the SF stage encrypted using 128-bit keys by default, if the files came unencrypted at the beginning. 

## Best practices for data loading







































