# Hive Task:
- Load videogame sales data from S3 into Hadoop
- Use Hive to query data
- Make sure to account for the header row in the data and commas in the 'Name' field

### Copy data from S3 into HDFS (in master node)
```
hdfs dfs -cp s3://data-eng-resources/big-data/vgsales.csv vgsales.csv
```

### Using Hive (via Hue)
```
CREATE DATABASE IF NOT EXISTS sales;

CREATE TABLE IF NOT EXISTS sales.vgsales(
Rank INT,
Name STRING,
Platform STRING,
Year INT,
Genre STRING,
Publisher STRING,
NA_Sales FLOAT,
EU_Sales FLOAT,
JP_Sales FLOAT,
Other_Sales FLOAT,
Global_Sales FLOAT)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
tblproperties("skip.header.line.count"="1");

LOAD DATA INPATH '/user/hadoop/vgsales.csv'
OVERWRITE INTO TABLE sales.vgsales;

SELECT * FROM sales.vgsales;
```
