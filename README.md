Apache Hive is a data warehouse style SQL engine for Apache Hadoop. Apache Phoenix is a SQL interface on top of Apache HBase. While Hive is great at executing batch and large-scale analytical queries, HBase's indexed lookups will be faster for record counts of up to a few million rows (specific count depends on a variety of factors).

The HiveToPhoenix artifact is intended to be used as a reusable application for executing queries against Hive tables and saving results to Phoenix tables. It requires an input properties file and relies on SparkSQL's DataFrames to make Hive->Phoenix data movement easy.

The job properties (job.props) file takes the typical Java Properties file format:
```
srcUser=
srcPass=
srcClass=org.apache.hive.jdbc.HiveDriver
srcConnStr=jdbc:hive2://localhost:10000
srcTable=test
srcScript=test/test_in.sql

dstTable=test
dstUser=
dstPass=
dstClass=org.apache.phoenix.jdbc.PhoenixDriver
dstConnStr=jdbc:phoenix:localhost:2181:/hbase-unsecure
dstZkUrl=localhost:2181:/hbase-unsecure
dstPk=id

typeMap=string|varchar,int|integer
```

If srcScript is null, the Phoenix table will be loaded with the result of "select * from srcTable".

If srcScript is not null, the contents of the file at the specified path will be executed by SparkSQL and results will be loaded.

Note: srcScript is provided as a convenience the allows the user to specify filters, but the query results currently must have the same schema as srcTable.

typeMap exists because Hive types and Phoenix types are not 1 to 1. "string|varchar" means that "string" types in the source table will be mapped to "varchar" types in the Phoenix table. Hive and Phoenix types are evolving, so the user is free to update this typeMap field.

**Note**: The following example assumes there are no existing Hive or Phoenix tables named "test".
Example:
```
hadoop fs -put test/input/ .
hive --hiveconf PATH=/user/root/input/ -f test/test.ddl
export SPARK_HOME=/usr/hdp/current/spark
export HADOOP_CONF_DIR=/etc/hadoop/conf
$SPARK_HOME/bin/spark-submit --class com.github.randerzander.HiveToPhoenix --master yarn-client --num-executors 1 --executor-memory 512M target/HiveToPhoenix-0.0.1-SNAPSHOT.jar test/job.props
/usr/hdp/current/phoenix-client/bin/psql.py localhost:/hbase-unsecure test/test_out.sql

Output:
ID                                                                            VAL 
---------------------------------------- ---------------------------------------- 
a                                                                               1 
Time: 0.039 sec(s)
```

Here we've put data in HDFS & defined a Hive table for reading it. Then we ran a Spark job to load the Hive table into a Phoenix table.

test/input/test.txt contained 2 records, but the Phoenix table contains only one: the test/test_in.sql script included a "where" clause filter, demonstrating the ability to load specific subsets of records.


ToDo:

1. Support derived expressions, e.g. select id, count(*) as count from srcTable

To build manually:
```
mvn clean package
```
