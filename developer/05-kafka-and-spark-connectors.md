# Kafka and Spark Connectors
## Kafka Connector
The Snowflake Kafka connector reads data from one or more Apache Kafka topics, and loads the data into a Snowflake table.

### Overview
An application publishes messages to a topic, and an application subscribes to a topic to receive those messages. Topics can be divided into partitions, to increase scalability.

Kafka Connect is a framework for connecting Kafka with external systems, such as databases. A Kafka Connect cluster is a separate cluster from the Kafka cluster, and it supports running/scaling-out connectors, which are components that support reading/writing between external systems.

The sf Kafka connector runs in a Kafka Connect cluster, to read data from Kafka topics, and write the data into Snowflake tables.

In Snowflake's view, a Kafka topic produces a stream of rows to be inserted into a sf table. Each Kafka message contains one row. One topic supplies messages (rows) for one sf table.

Kafka topics can be mapped to existing Snowflake tables in the Kafka configuration. If the topics are not mapped, then the Kafka connector creates a new table for each topic using the topic name.

Every Snowflake table loaded by the Kafka connector has two VARIANT columns:
- RECORD_CONTENT. The Kafka message.
- RECORD_METADATA. Metadata about the message, eg, the source topic.

If Snowflake creates the table, then the table contains only these two columns. 

If the user creates the table, then the table can contain more than these two columns (any additional cols must allow NULL values, because data from the connector does not have values for those columns).

Typically, each message in a specific topic has the same basic structure. Different topics typically use different structure.

Each Kafka message is passed to Snowflake in JSON/Avro format.

The amount of metadata recorded in the RECORD_METADATA column is configurable, using optional Kafka configuration properties.

The Kafka connector does below stuff, to subscribe to Kafka topics and create Snowflake objects:
1. It subscribes to one/more Kafka topics, based on the config info, provided via the Kafka config file or command line.
2. It creates the following objects for each topic:
   - One internal stage, to temporarily store data files for each topic.
   - One pipe, to ingest the data files for each topic partition.
   - One table for each topic. If the table does not exist, it will be created; otherwise, it creates the RECORD_CONTENT and RECORD_METADATA columns in the existing table, and verifies the other cols are nullable.

The sf Kafka connector buffers messages from the Kafka topics. When a threshold is reached, the connector writes the messages to a temporary file in the internal stage. The connector then triggers Snowpipe to ingest the temporary file. After the file is loaded, the connector cleans it up in the stage. If a failure prevented the data from loading, the connector moves the file into the table stage and produces an error message.

Instances of the Kafka connector do not communicate with each other. If you start multiple instances of the connector on the same topics/partitions, then multiple copies of the same row might be inserted into the table. This is not recommended; each topic should be processed by only one instance of the connector.

There is no guarantee that rows are inserted in the order that they were originally published.

## Spark Connector
Allows Spark to read data from, and write data to, Snowflake. 
