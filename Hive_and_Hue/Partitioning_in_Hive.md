# Partitioning

## Partition using 2 separate files as source
```
CREATE DATABASE IF NOT EXISTS academy;

CREATE TABLE academy.spartitions (
spartan_id INT,
client_name STRING,
placement_date DATE )
PARTITIONED BY (
city STRING )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH '/home/hadoop/spartans-london.csv'
INTO TABLE academy.spartitions PARTITION (city='London');

LOAD DATA LOCAL INPATH '/home/hadoop/spartans-bham.csv'
INTO TABLE academy.spartitions PARTITION (city='Birmingham');

SELECT * FROM academy.spartitions;
```

## Partition using 1 file as source
```
CREATE DATABASE IF NOT EXISTS academy;
DROP TABLE IF EXISTS academy.spartitions;

CREATE TABLE academy.spartitions (
spartan_id INT,
client_name STRING,
placement_date DATE )
PARTITIONED BY (
city STRING )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

DROP TABLE IF EXISTS academy.spartitions_placeholder;
CREATE TABLE academy.spartitions_placeholder (
spartan_id INT,
client_name STRING,
placement_date DATE,
city STRING )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH '/home/hadoop/spartans-all.csv'
OVERWRITE INTO TABLE academy.spartitions_placeholder;

SELECT * FROM academy.spartitions_placeholder;

INSERT INTO TABLE academy.spartitions PARTITION (city='London')
SELECT spartan_id, client_name, placement_date FROM academy.spartitions_placeholder
WHERE city='London';

INSERT INTO TABLE academy.spartitions PARTITION (city='Bham')
SELECT spartan_id, client_name, placement_date FROM academy.spartitions_placeholder
WHERE city='Bham';

SELECT * FROM academy.spartitions;
```

## Dynamic Partitioning
```
SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;

CREATE DATABASE IF NOT EXISTS academy;
DROP TABLE IF EXISTS academy.spartitions;

CREATE TABLE academy.spartitions (
spartan_id INT,
client_name STRING,
placement_date DATE )
PARTITIONED BY (
city STRING )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

DROP TABLE IF EXISTS academy.spartitions_placeholder;
CREATE TABLE academy.spartitions_placeholder (
spartan_id INT,
client_name STRING,
placement_date DATE,
city STRING )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH '/home/hadoop/spartans-all.csv'
OVERWRITE INTO TABLE academy.spartitions_placeholder;

SELECT * FROM academy.spartitions_placeholder;

INSERT INTO TABLE academy.spartitions PARTITION (city)
SELECT spartan_id, client_name, placement_date, city FROM academy.spartitions_placeholder;

```
- Note: The last column will be the one that is partitioned in this statement (in this case city):
```
INSERT INTO TABLE academy.spartitions PARTITION (city)
SELECT sparta_id, client_name, placement_date, city FROM academy.spartitions_placeholder;
```

## Task
- Load in the contents of vgsales.csv into a Hive table.
- and dynamically partition by Platform

### Load data from S3:
- ssh into master node and run:
`hdfs dfs -cp s3://data-eng-resources/big-data/vgsales.csv vgsales.csv`


### Using Hive (via Hue) and dynamically partitioning data by platform
```
CREATE DATABASE IF NOT EXISTS sales;

SET hive.exec.dynamic.partition=true;
SET hive.exec.dynamic.partition.mode=nonstrict;

CREATE DATABASE IF NOT EXISTS sales;

CREATE TABLE IF NOT EXISTS sales.vgsales_partition(
rank INT,
name STRING,
year INT,
genre STRING,
publisher STRING,
NA_Sales FLOAT,
EU_sales FLOAT,
JP_sales FLOAT,
other_sales FLOAT,
global_sales FLOAT)
PARTITIONED BY (
platform STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
tblproperties("skip.header.line.count"="1");

CREATE TABLE IF NOT EXISTS sales.vgsales_partition_placeholder(
rank INT,
name STRING,
platform STRING,
year INT,
genre STRING,
publisher STRING,
NA_sales FLOAT,
EU_sales FLOAT,
JP_sales FLOAT,
other_sales FLOAT,
global_sales FLOAT)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
tblproperties("skip.header.line.count"="1");

LOAD DATA INPATH '/user/hadoop/vgsales.csv'
OVERWRITE INTO TABLE sales.vgsales_partition_placeholder;

INSERT INTO TABLE sales.vgsales_partition PARTITION (platform)
SELECT rank, name, year, genre, publisher, NA_Sales, EU_Sales, JP_Sales, Other_sales, Global_sales, platform FROM sales.vgsales_partition_placeholder;

```
