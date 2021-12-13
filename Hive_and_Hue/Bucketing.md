# Bucketing

```
CREATE TABLE academy.spartans (
spartan_id INT,
client_name STRING )
PARTITIONED BY (
city STRING)
CLUSTERED BY (
spartan_id)
INTO 8 BUCKETS
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

CREATE TABLE academy.placeholder (
spartan_id INT,
client_name STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH '/home/hadoop/spartans.csv'
INTO TABLE academy.placeholder;

INSERT INTO TABLE academy.spartans PARTITION (city='London')
SELECT spartan_id, client_name FROM academy.placeholder;
```
Note: for Hive to do bucketing we need a placeholder table

<br />

Look at data in hdfs
`hdfs dfs -ls /user/hive/warehouse/academy.db/spartans/city=London`
