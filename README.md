# Hadoop Ecosystem

### This repo will contain notes on the Hadoop Ecosystem

- **Apache Pig:**  a high-level platform for creating programs that run on Apache Hadoop. The language for this platform is called Pig Latin. Pig can execute its Hadoop jobs in MapReduce, Apache Tez, or Apache Spark.
- **Apache Hive:** Hive gives an SQL-like interface for querying stored data
- **HDFS:** Storage layer for Hadoop (HDFS enables the rapid transfer of data between nodes.)
- **Flume:** Collects, aggregates and logs data.
- **YARN:** Resource manager for Hadoop
- **Spark:** Analytics engine for large-scale data processing.
- **Sqoop:** CLI for transferring data between relational databases and Hadoop.
- **Kafka:** Provides a framework for storing, reading and analysing streaming data.
- **MapReduce:** is a software framework for writing applications which process big data in-parallel on large clusters. This process involves splitting the input data-set into independent chunks which are then processed (Map) and then sorting the outputs of the maps, which are then input to the reduce tasks (Reduce).
<br />
<br />

### AWS EMR (Elastic Map Reduce)
- EMR is the Amazon EMR is the cloud big data platform provided by AWS
- Accessing node in AWS: `ssh -i <path_to_private_key> hadoop@<Master_public_DNS>`
