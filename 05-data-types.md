# 5. Data Types
## Binary data input and output
3 supported binary encoding formats: hex (default, base16), base64, and UTF-8.

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
Semi-structured data:
- does not require a prior definition of a schema and can constantly evolve.
- entities within the same class may have different attributes, the order of the attributes is not important.
- can contain N-level hierarchies of nested information. 

An OBJECT contains key-value pairs like a Java HashMap. An ARRAY contains a list of things like a Java ArrayList. VARIANT is used to build and store hierarchical data, can also contain ARRAY and OBJECT.

Snowflake can import semi-structured data from JSON, Avro, ORC, Parquet, and XML formats. 

When loading semi-structured data, you can
- if the data is key-value pairs, you can load it into a col of type OBJECT.
- if the data is is an array, you can load it into a column of type ARRAY.
- if the data is hierarchical, you can split the data across diff columns, or store it in one col as VARIANT. 

### Supported formats
JSON, Avro, ORC, Parquet, XML. 

### Querying VARIANT
Refer to chapter 0. 

### Considerations for VARIANT
The VARIANT data type imposes a `16 MB size limit` on individual rows. If the data exceeds 16 MB, enable the STRIP_OUTER_ARRAY file format option for the COPY INTO command to remove the outer array brackets and load the records inside into separate table rows. 

To convert a VARIANT "null" value to SQL NULL, cast it as a string.

To improve query performance, recommend to extract semi-structured data elements containing "null" values into relational columns before loading them. Or, if the "null" values in your files simply mean missing values, recommend setting the file format option `STRIP_NULL_VALUES` to TRUE when loading the semi-structured data files. 

When working with unfamiliar semi-structured data, you might not know the key names in an OBJECT. You can use the `FLATTEN function with the RECURSIVE argument to return the list of distinct key names in all nested elements in an OBJECT`. You can also use this to retrieve all keys and paths in an OBJECT.

## Unstructured data
### Intro
Includes form responses, social media conversations, images, video, audio, industry-specific file types such as VCF (genomics), KDF (semiconductors), or HDF5 (aeronautics).

Snowflake supports:
- Securely access data files located in cloud storage.
- Share file access URLs with collaborators and partners.
- Load file access URLs and other file metadata into Snowflake tables.

Both external and internal stages support unstructured data.

Types of URLs to access files: 
- **Scoped URL**: encoded, `temp access of 24 hrs`, need role privileges. Good for custom apps that provides files to other accounts via a `share`, for download for ad hoc analysis
- **File URL**: identifies `permanent full path` of the files, need role privileges. Good for custom apps with authorization token. Format: `https://<account_identifier>/api/files/<db_name>.<schema_name>.<stage_name>/<relative_path>`
- **Pre-signed URL**: a https url for web browser, `anyone can access`, configurable expiration time. Good for BI tools to display the file contents. Need stages with server-side encryption, NOT client-side encryption (because other users need the encryption key to read the files). 

### Directory tables
Directory tables store file-level metadata about the files in a stage. Roles with sufficient privileges can get file URLs & metadata from them.

A directory table can be added explicitly to a stage at any time. It is an implicit object layered on a stage. 

The metadata for a directory table can be refreshed manually, or automatically using the event notification service for your cloud storage service. The overhead charge for event notification management appears as Snowpipe charges in your billing statement.

Automatic refresh the metadata is not available for:
- internal stages
- external stages that refers Google Cloud Storage buckets

You can use streams on a directory table to track added/dropped files in the referenced cloud storage location.

### REST API
`GET /api/files/`

Python example:
```py
import requests
response = requests.get(url,
    headers={
      "User-Agent": "reg-tests",
      "Accept": "*/*",
      "X-Snowflake-Authorization-Token-Type": "OAUTH",
      "Authorization": """Bearer {}""".format(token)
      },
    allow_redirects=True)
print(response.status_code)
print(response.content)
```

### Processing with Java UDFs or UDTFs (preview)
To make your code resilient to file injection attacks, always use a scoped URL when passing a file's location to a UDF. 

example:
```java
CREATE FUNCTION process_pdf(file string)
RETURNS string
LANGUAGE java
RUNTIME_VERSION = 11
IMPORTS = ('@jars_stage/pdfbox-app-2.0.27.jar')
HANDLER = 'PdfParser.readFile'
as
$$
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.text.PDFTextStripper;
import org.apache.pdfbox.text.PDFTextStripperByArea;
import com.snowflake.snowpark_java.types.SnowflakeFile;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

public class PdfParser {
    public static String readFile(String fileURL) throws IOException {
        SnowflakeFile file = SnowflakeFile.newInstance(fileURL);
        try (PDDocument document = PDDocument.load(file.getInputStream())) {
            document.getClass();
            if (!document.isEncrypted()) {
                PDFTextStripperByArea stripper = new PDFTextStripperByArea();
                stripper.setSortByPosition(true);
                PDFTextStripper tStripper = new PDFTextStripper();
                String pdfFileInText = tStripper.getText(document);
                return pdfFileInText;
            }
        }
        return null;
    }
}
$$;
```
```sql
-- Input a scoped URL.
SELECT process_pdf(build_scoped_file_url('@data_stage', '/myfile.pdf'));
```

### Sharing unstructured data
Data providers can use Secure Data Sharing to share unstructured data files with data consumers.

Create secure views that allow data consumers to retrieve scoped/pre-signed URLs for unstructured data files.

### Troubleshooting
Make sure to use Server Side Encryption (SSE). 
