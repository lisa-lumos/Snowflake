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



## Spark Connector
### Overview
















