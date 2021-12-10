# Hive (and Hue)

## Hive
- Apache Hive is a data warehouse software project built on top of Apache Hadoop for providing data query and analysis. Hive gives an SQL-like interface to query data stored in various databases and file systems that integrate with Hadoop.

### Hive - Schema On Read
- Hive has a schema on read system
- In schema on read, data is applied to a schema as it is pulled out of a stored location.

## Hue
- Hue is a web user interface which provides a number of services including Hive. It is an open-source SQL Cloud Editor.

### Accessing Hue

#### Configure proxy settings
- To access Hue we must first configure proxy settings:
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-connect-master-node-proxy.html

#### Setting up ssh port forwarding tunnel
ssh -i ~/.ssh/<private_key> -N -D 8157 hadoop@<public_DNS>

#### Access Hue in web browser
- In the address bar go to: `<private_dns_name>:8888`

### Hive Simple Example:
```
CREATE DATABASE IF NOT EXISTS academy;

CREATE TABLE IF NOT EXISTS academy.spartans (
spartan_id INT,
client_name STRING,
placement_start DATE )
ROW FORMAT DELIMITED FIELDS TERMINATED BY ',';

LOAD DATA LOCAL INPATH 'home/hadoop/spartans.csv'
OVERWRITE INTO TABLE academy.spartans;

SELECT * FROM academy.spartans;
```
