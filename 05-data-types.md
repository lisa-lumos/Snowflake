# 5. Data Types

## Binary data input and output
3 supported binary encoding formats: hex (default), base64, and UTF-8.

Two session parameters determines how binary values are passed in and out of Snowflake: BINARY_INPUT_FORMAT and BINARY_OUTPUT_FORMAT, the default val for both is 'hex'. 

BINARY_FORMAT file format option can be used to explicitly control binary formatting for data loading/unloading. This setting is more specific than the 2 session params. 

The BINARY data type can be used as an intermediate step when converting between hex and base64 strings.

## Date & Time input and output
These session parameters determine how date, time, and timestamp data is passed into and out of Snowflake, as well as the time zone
- DATE_INPUT_FORMAT
- TIME_INPUT_FORMAT
- TIMESTAMP_INPUT_FORMAT
- DATE_OUTPUT_FORMAT
- TIME_OUTPUT_FORMAT
- TIMESTAMP_OUTPUT_FORMAT
- TIMESTAMP_LTZ_OUTPUT_FORMAT
- TIMESTAMP_NTZ_OUTPUT_FORMAT
- TIMESTAMP_TZ_OUTPUT_FORMAT
- TIMESTAMP_TYPE_MAPPING
- TIMEZONE

DATE_FORMAT, TIME_FORMAT, TIMESTAMP_FORMAT file format option can be used for explicitly control for data loading/unloading. This setting is more specific than the session params. 

## Semi-structured data 
### Intro

### Supported formats

### Querying

### Considerations

## Unstructured data
### Intro

### Directory tables

### REST API

### Processing with Java UDFs or UDTFs

### Sharing

### Troubleshooting



































