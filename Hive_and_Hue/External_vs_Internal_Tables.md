# Hive External vs Internal Tables

## Hive Internal Table
- Hive owns the data for the internal tables.
- It is the default table in Hive. When the user creates a table in Hive without specifying it as external, then by default, an internal table gets created in a specific location in HDFS.
- By default, an internal table will be created in a folder path similar to /user/hive/warehouse directory of HDFS. We can override the default location by the location property during table creation.
- If we drop the managed table or partition, the table data and the metadata associated with that table will be deleted from the HDFS.

## Hive External Table
- Hive does not manage the data of the External table.
- We create an external table for external use as when we want to use the data outside the Hive.
- External tables are stored outside the warehouse directory. They can access data stored in sources such as remote HDFS locations or Azure Storage Volumes.
- Whenever we drop the external table, then only the metadata associated with the table will get deleted, the table data remains untouched by Hive.
- We can create the external table by specifying the EXTERNAL keyword in the Hive create table statement.

<br />

### Creating an external table:
```
CREATE DATABASE academy;

CREATE EXTERNAL TABLE academy.spartans_all (
spartans_id INT,
client_name STRING,
placement_date DATE,
city STRING)
ROW FORMAT DELIMITED FIELDS Terminated BY ','
LOCATION '/user/hadoop/spartans/';

SELECT * FROM academy.spartans_all;
```
