# Apache Pig

- **Pig Latin** is the data flow language used by Apache Pig to analyse the data in Hadoop.
- Tasks in Apache Pig are converted into MapReduce jobs automatically

## Apache Pig - Grunt Shell
- Accessing the pig shell (grunt):
- Local mode: `pig -x local`
- Hadoop mode: `pig -x mapreduce` or `pig`

<br />
- **Local Mode** - To run Pig in local mode, you need access to a single machine; all files are installed and run using your local host and file system. Specify local mode using the -x flag (pig -x local).
- **Mapreduce Mode** - To run Pig in mapreduce mode, you need access to a Hadoop cluster and HDFS installation. Mapreduce mode is the default mode; you can, but don't need to, specify it using the -x flag (pig OR pig -x mapreduce).

## Keywords
- **DUMP**	Prints results to the screen. `DUMP var;`
- **LIMIT**	Restricts number of records.	`LIMIT var 10;`
- **GROUP** BY	Groups data on a column.	`GROUP var BY col;`
- **ORDER BY**	Orders the data by a column.	`ORDER var BY col [ASC / DESC]``
- **FILTER**	Filters data.	`FILTER var BY col > 10`
- **FOREACH**	Applies transformation on each record.	`FOREACH var GENERATE col1, col2, col3 * 10`
- **JOIN**	Joins two variables together.	`JOIN var1 BY col, var2 BY col;``
- **STORE**	Stores results of MapReduce job.	`STORE var INTO 'name'`
- **DESCRIBE**	Shows schema of variable.	`DESCRIBE var`
- **EXPLAIN**	Shows execution plan.	`EXPLAIN var`
- **ILLUSTRATE**	Shows step-by-step breakdown of script.	`ILLUSTRATE var`

## Disambiguate Operator
- After JOIN, COGROUP, CROSS, or FLATTEN operations, the field names have the orginial alias and the disambiguate operator ( :: ) prepended in the schema. The disambiguate operator is used to identify field names in case there is a ambiguity.
- In this example, to disambiguate y, use A::y or B::y. In cases where there is no ambiguity, such as z, the :: is not necessary but is still supported.
```
A = load 'data1' as (x, y);
B = load 'data2' as (x, y, z);
C = join A by x, B by x;
D = foreach C generate A::y, z; -- Cannot simply refer to y as it can refer to A::y or B::y
```

## Examples:
- **Loading in data from S3:**
```
index_raw = LOAD '<s3_uri>' USING PigStorage(',') AS (date: chararray, open: float, high: float, low: float, close: float, adjclose: float, volume:int);
```
- `PigStorage(',')` - ',' refers to delimiter
- `date: chararray` - i.e. <column_name>:<data_type>
<br />

- **Filtering Data examples:**
- Retrieve specific columns: `index_values = FOREACH index_raw GENERATE date, high, low, volume`
- Filter data where high column  greater than 3000: (3000.0F - the F indicates data type is a float) `filtered_high = FILTER index_values BY high > 3000.0F`
- Limit to 10 rows: `results_10 = LIMIT filtered_high 10;`

<br />

- **Grouping data by month and calculating total volume per month:**
```
months = FOREACH index_values GENERATE SUBSTRING(date, 3, 10) AS month, volume;

month_groups = GROUP months BY month

month_total = FOREACH month_groups GENERATE group, SUM(months.volume) AS total;
```

<br />

- **Sorting and Storing data:**
```
subset = FOREACH index_raw GENERATE date, high, volume;

ordered = ORDER subset BY high DESC;

STORE ordered INTO 'ordered_tesla' USING PigStorage(',');

```

<br />

- **Remove first 10 Rows of data:**
```
filterHeader = STREAM fullFile THROUGH `tail -n +10`;

```

## Count No. of words in file (Creating Pig Script)
- Load data: `curl -O norvig.com/big.txt`

- `hdfs dfs -mkdir wordcount`

- Move data into hdfs: `hdfs dfs -put big.txt wordcount`

- Create Pig script: `nano wordcount.pig`
- Contents of wordcount.pig:
```
lines = LOAD 'wordcount/big.txt' AS (line: chararray);
words = FOREACH lines GENERATE FLATTEN(TOKENIZE(line)) as word;
grouped = GROUP words BY word;
wordcount = FOREACH grouped GENERATE group, COUNT(words) AS count;
STORE wordcount INTO 'pig_wordcount';
```
- Run script: `pig -x mapreduce wordcount.pig`
- File is stored in hdfs: `hdfs dfs -ls pig_wordcount`
- Move file to local node: `hdfs dfs -cat pig_wordcount/part-r-* > output.txt`
- Copy file to local machine from node: `scp -i ~/.ssh/<public_key>hadoop@3.64.149.60:/home/hadoop/output.txt .`

## Task
**Task to complete:**
```
1. Load CAT file in using Pig
- Same as with TSLA file

2. Calculate difference between stock values at close every day
- Between TSLA and CAT

3. Make sure you have both tsla and cat files loaded into own variables.
- Grab only relevant columns, date and close.
- Join them using date column.
- Use ILLUSTRATE to look at how to access dates in joint file.
```
**Answer:**
- Loading in data:
```
tesla_raw = LOAD 's3:/<file_path>/TSLA.csv' USING PigStorage(',') AS (date: chararray, open: float, high: float, low: float, close: float, adjclose: float, volume:int);

cat_raw = LOAD 's3:/<file_path>/CAT.csv' USING PigStorage(',') AS (date: chararray, open: float, high: float, low: float, close: float, adjclose: float, volume:int);
```
- Grabbing only relevant columns:
```
close_values_tesla = FOREACH tesla_raw GENERATE date, close;

close_values_cat = FOREACH tesla_cat GENERATE date, close;
```
- Join data using data column:
```
combined_data = JOIN close_values_tesla BY date, close_values_cat BY date;

combined_data_final = FOREACH combined_data GENERATE close_values_cat::date AS date, close_values_cat::close AS cat_close, close_values_tesla::close AS tesla_close;
```
- Add difference column to data:
```
diff = FOREACH combined_data_final GENERATE date, (tesla_close - cat_close) AS diff;
```
- Using Illustrate: `ILLUSTRATE diff;`
```
ILLUSTRATE diff;
------------------------------------------------------------------------------------------------------------------------------------------------
| tesla_raw     | date:chararray     | open:float     | high:float     | low:float     | close:float     | adjclose:float     | volume:int     |
------------------------------------------------------------------------------------------------------------------------------------------------
|               | 09/04/2019         | 271.65         | 275.0          | 269.61        | 272.31          | 272.31             | 5904000        |
|               | 09/04/2019         | 271.65         | 275.0          | 269.61        | 272.31          | 272.31             | 5904000        |
------------------------------------------------------------------------------------------------------------------------------------------------
-----------------------------------------------------------------
| close_values_tesla     | date:chararray     | close:float     |
-----------------------------------------------------------------
|                        | 09/04/2019         | 272.31          |
|                        | 09/04/2019         | 272.31          |
-----------------------------------------------------------------
----------------------------------------------------------------------------------------------------------------------------------------------
| cat_raw     | date:chararray     | open:float     | high:float     | low:float     | close:float     | adjclose:float     | volume:int     |
----------------------------------------------------------------------------------------------------------------------------------------------
|             | 09/04/2019         | 138.65         | 138.91         | 136.07        | 136.35          | 132.51521          | 3316500        |
|             | 09/04/2019         | 138.65         | 138.91         | 136.07        | 136.35          | 132.51521          | 3316500        |
----------------------------------------------------------------------------------------------------------------------------------------------
---------------------------------------------------------------
| close_values_cat     | date:chararray     | close:float     |
---------------------------------------------------------------
|                      | 09/04/2019         | 136.35          |
|                      | 09/04/2019         | 136.35          |
---------------------------------------------------------------
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
| combined_data     | close_values_tesla::date:chararray     | close_values_tesla::close:float     | close_values_cat::date:chararray     | close_values_cat::close:float     |
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                   | 09/04/2019                             | 272.31                              | 09/04/2019                           | 136.35                            |
|                   | 09/04/2019                             | 272.31                              | 09/04/2019                           | 136.35                            |
|                   | 09/04/2019                             | 272.31                              | 09/04/2019                           | 136.35                            |
|                   | 09/04/2019                             | 272.31                              | 09/04/2019                           | 136.35                            |
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
----------------------------------------------------------------------------------------------
| combined_data_final     | date:chararray     | cat_close:float     | tesla_close:float     |
----------------------------------------------------------------------------------------------
|                         | 09/04/2019         | 136.35              | 272.31                |
|                         | 09/04/2019         | 136.35              | 272.31                |
|                         | 09/04/2019         | 136.35              | 272.31                |
|                         | 09/04/2019         | 136.35              | 272.31                |
----------------------------------------------------------------------------------------------
--------------------------------------------------
| diff     | date:chararray     | diff:float     |
--------------------------------------------------
|          | 09/04/2019         | 135.95999      |
|          | 09/04/2019         | 135.95999      |
|          | 09/04/2019         | 135.95999      |
|          | 09/04/2019         | 135.95999      |
--------------------------------------------------
```
