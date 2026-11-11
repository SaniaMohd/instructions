# Big Data Analytics (BDA) Lab Manual

**Environment:** Cloudera QuickStart VM 5.13.0, Hadoop 2.6.0 (CDH
5.13.0), Java 7/8-era Hadoop environment\
**Purpose:** Quick practical reference for Units I--V. Commands and
programs are based on the supplied syllabus, practical record, and the
Word Count program successfully executed in the Cloudera VM.

> **Important:** Some syllabus tasks do not have a complete
> implementation in the supplied practical material. They are clearly
> marked below instead of being replaced with invented faculty code.

------------------------------------------------------------------------

# UNIT I -- HDFS (Storage)

## A. Create directories and move local files to HDFS

### Create HDFS directories

``` bash
hadoop fs -mkdir /user/cloudera/data
hadoop fs -mkdir -p /user/cloudera/data/input
```

-   `-mkdir`: creates an HDFS directory.
-   `-p`: creates parent directories if needed.

### Create a local file

``` bash
cd ~/Desktop
cat > students.txt
```

Enter:

``` text
1,kriti,cse,45
2,neha,ece,56
3,jyothi,ce,78
4,priya,mechanical,89
```

Press `Ctrl+D` to save.

### Upload local file to HDFS

``` bash
hadoop fs -put students.txt /user/cloudera/data/
hadoop fs -copyFromLocal students.txt /user/cloudera/data/
```

Both commands copy a local file into HDFS. Use one of them for a given
upload.

## B. View files and directories

``` bash
hadoop fs -ls /user/cloudera/
hadoop fs -ls -R /user/cloudera/
hadoop fs -cat /user/cloudera/data/students.txt
hadoop fs -tail /user/cloudera/data/students.txt
```

-   `-ls`: lists files and directories.
-   `-ls -R`: lists recursively.
-   `-cat`: displays file contents.
-   `-tail`: displays the end of a file.

## C. Copy files from HDFS to local disk

``` bash
hadoop fs -get /user/cloudera/data/students.txt ~/Desktop/
hadoop fs -copyToLocal /user/cloudera/data/students.txt ~/Desktop/
```

Both copy a file from HDFS to the local filesystem.

## Other useful HDFS commands

``` bash
hadoop fs -cp source destination
hadoop fs -mv source destination
hadoop fs -rm /user/cloudera/file.txt
hadoop fs -rm -r /user/cloudera/directory
hadoop fs -touchz /user/cloudera/empty.txt
hadoop fs -du -s /user/cloudera/
hadoop fs -count /user/cloudera/
hadoop fs -chmod 777 /user/cloudera/file.txt
hadoop fs -df
hdfs fsck /
hadoop fs -appendToFile local.txt /user/cloudera/target.txt
hadoop fs -help
hdfs dfsadmin -safemode get
hdfs dfsadmin -safemode leave
```

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `-cp`                               Copies a file within HDFS

  `-mv`                               Moves or renames an HDFS path

  `-rm`                               Deletes a file

  `-rm -r`                            Deletes a directory recursively

  `-touchz`                           Creates a zero-length file

  `-du -s`                            Shows total space used by a path

  `-count`                            Shows directory count, file count
                                      and content size

  `-chmod 777`                        Gives read/write/execute permission
                                      to owner, group and others

  `-df`                               Shows filesystem capacity and free
                                      space

  `hdfs fsck /`                       Checks HDFS filesystem health

  `-appendToFile`                     Appends local file content to an
                                      HDFS file

  `safemode get/leave`                Checks/leaves HDFS safe mode
  -----------------------------------------------------------------------

## Local Linux commands used in practicals

  Command                   Purpose
  ------------------------- ----------------------------------------
  `pwd`                     Shows current directory
  `ls`                      Lists local files
  `cd Desktop`              Changes directory
  `cat > file.txt`          Creates a file and accepts typed input
  `cat file.txt`            Displays a local file
  `cp source destination`   Copies local file
  `mv source destination`   Moves/renames local file
  `rm file.txt`             Deletes local file

------------------------------------------------------------------------

# UNIT II -- MapReduce Programming

## A. Word Count

**Aim:** Count the occurrence/frequency of each word in a text file.

### 1. Input file

``` bash
cd ~/Desktop
cat > count.txt
```

Enter:

``` text
hi how are you
hope you are fine
we are all good
hi we will meet
see you soon
sure soon we will meet
```

Press `Ctrl+D`.

### 2. Upload and verify input

``` bash
hadoop fs -put count.txt /user/cloudera/
hadoop fs -cat /user/cloudera/count.txt
```

### 3. Java source -- `Classword.java`

Package: `word`\
The public class name must match the filename: `Classword`.

``` java
package word;

import java.io.IOException;
import java.util.StringTokenizer;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Classword {

    public static class Map extends Mapper<LongWritable, Text, Text, IntWritable> {
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                String word = itr.nextToken();
                context.write(new Text(word), new IntWritable(1));
            }
        }
    }

    public static class Reduce extends Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values, Context context)
                throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable x : values) {
                sum += x.get();
            }
            context.write(key, new IntWritable(sum));
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = new Job(conf, "My Word Count Program");

        job.setJarByClass(Classword.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

**Note:** The `LongWritable` import is required by this exact source.
Add this import with the other Hadoop imports:

``` java
import org.apache.hadoop.io.LongWritable;
```

### 4. Compile and package in the Cloudera VM

The following commands reflect the successful local execution setup used
in the VM. Run from the project directory, where
`src/word/Classword.java` exists:

``` bash
cd /home/cloudera/workspace/mapreduce
mkdir -p bin
javac -cp "$(hadoop classpath)" -d bin src/word/Classword.java
jar -cvf mapreduce.jar -C bin .
jar tf mapreduce.jar
```

-   `hadoop classpath`: supplies Hadoop library paths to the Java
    compiler.
-   `-d bin`: writes compiled `.class` files to `bin`.
-   `jar -cvf`: creates a JAR.
-   `jar tf`: lists JAR contents.

### 5. Run the job

``` bash
hadoop jar mapreduce.jar word.Classword /user/cloudera/count.txt /user/cloudera/result
```

If the output directory already exists, delete it before rerunning:

``` bash
hadoop fs -rm -r /user/cloudera/result
```

### 6. View output

``` bash
hadoop fs -cat /user/cloudera/result/part-r-00000
```

Expected output for the input above:

``` text
all     1
are     3
fine    1
good    1
hi      2
hope    1
how     1
meet    2
see     1
soon    2
sure    1
we      3
will    2
you     3
```

### Word Count code meaning

  Code/concept                Purpose
  --------------------------- ---------------------------------------
  `StringTokenizer`           Splits a line into words
  `hasMoreTokens()`           Checks if more words remain
  `nextToken()`               Gets next word
  `context.write(word, 1)`    Emits a word with count 1
  Mapper                      Produces intermediate key/value pairs
  Reducer                     Adds counts for each word
  `job.setMapperClass()`      Sets Mapper class
  `job.setReducerClass()`     Sets Reducer class
  `waitForCompletion(true)`   Runs and waits for job

**Important correction:** The source above uses `LongWritable`; ensure
the `LongWritable` import is present. The working code in your VM had
`Map` and `Reduce` inner classes and successfully produced the expected
output.

## B. Most Frequent Words Count

**Syllabus task:** Use the Word Count output to list the most frequent
words with their counts. The syllabus also mentions loading data using a
third filter.

The practical record gives these commands:

``` bash
hadoop jar /home/cloudera/MostFrequentWords.jar MostFrequentWords /user/cloudera/input/count.txt /user/cloudera/output_mostfreq
```

If output directory exists:

``` bash
hadoop fs -rm -r /user/cloudera/output_mostfreq
```

Run again:

``` bash
hadoop jar /home/cloudera/MostFrequentWords.jar MostFrequentWords /user/cloudera/input/count.txt /user/cloudera/output_mostfreq
```

View output:

``` bash
hadoop fs -cat /user/cloudera/output_mostfreq/part-r-00000
```

Faculty-record sample output:

``` text
1 bigdata
2 mapreduce
2 hdfs
3 hadoop
```

**Source limitation:** The supplied record gives the execution commands
and sample output, but the complete `MostFrequentWords.java` source and
the exact "third filter" implementation were not present in the
retrieved material. Do not treat the command above as guaranteed
runnable until the matching JAR/class and input paths are supplied.

## C. K-Means clustering using MapReduce

**Syllabus task:** Implement K-Means using Hadoop MapReduce.

Conceptual workflow: 1. Store input data points and initial centroids.
2. Mapper assigns each point to its nearest centroid. 3. Mapper emits
`(centroid → data point)`. 4. Reducer averages points assigned to each
centroid to compute new centroids. 5. Repeat until centroids stabilize
(convergence). 6. Use HDFS to store iteration inputs/outputs.

**Source limitation:** The syllabus/record explains the algorithm and
Mapper/Reducer responsibilities, but a complete Java implementation,
input dataset and exact execution commands were not available in the
supplied material. This section is the faculty-supported algorithm
outline, not a complete executable program.

## Additional practical-record program: Maximum Temperature

The practical record lists: - `MaxTempMapper.java`: extracts year and
temperature and filters invalid readings/quality flags. -
`MaxTempReducer.java`: finds maximum temperature for each year. -
`MaxTemp86.java`: configures and runs the job.

The record's sample output:

``` text
2020    45
2021    50
```

Execution command shown in the notes:

``` bash
hadoop jar MaxTemp86.jar MaxTemp86 /user/cloudera/maxtempinput /user/cloudera/maxtempoutput
```

View output:

``` bash
hdfs dfs -ls /user/cloudera/maxtempoutput/
hdfs dfs -cat /user/cloudera/maxtempoutput/part-r-00000
```

**Source limitation:** The full three Java source files were not present
in the material retrieved for this manual. The commands and sample
output above are the available record details.

------------------------------------------------------------------------

# UNIT III -- Data Processing Tools: Hive and Pig

## A. Hive -- HQL

### 1. Start Hive and create database

``` bash
hive
```

``` sql
CREATE DATABASE emp_details;
SHOW DATABASES;
USE emp_details;
```

  Command             Purpose
  ------------------- ------------------
  `hive`              Opens Hive CLI
  `CREATE DATABASE`   Creates database
  `SHOW DATABASES`    Lists databases
  `USE`               Selects database

### 2. Create employee table

``` sql
CREATE TABLE emp(
    empno INT,
    ename STRING,
    job STRING,
    sal INT,
    deptno INT
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

-   `INT`: integer datatype.
-   `STRING`: text datatype.
-   `FIELDS TERMINATED BY ','`: comma-separated columns.

### 3. Create input file

``` bash
cd ~/Desktop
cat > empdetails.txt
```

Enter:

``` text
1,A,clerk,4000,10
2,A,clerk,4000,30
3,B,mgr,8000,20
4,C,peon,2000,40
5,D,clerk,4000,10
6,E,mgr,8000,50
```

Press `Ctrl+D`.

### 4. Load data

``` sql
LOAD DATA LOCAL INPATH '/home/cloudera/Desktop/empdetails.txt'
OVERWRITE INTO TABLE emp;
```

`LOCAL` means the source file is on the local filesystem. `OVERWRITE`
replaces existing table data.

### 5. Hive commands and queries

``` sql
SHOW TABLES;
DESCRIBE emp;
DESCRIBE EXTENDED emp;
SELECT * FROM emp;
SELECT ename FROM emp;
SELECT * FROM emp WHERE ename='A';
SELECT COUNT(1) FROM emp;
SELECT deptno, SUM(sal) FROM emp GROUP BY deptno;
SELECT * FROM emp WHERE sal BETWEEN 1000 AND 2000;
SELECT ename FROM emp WHERE job LIKE '_____';
SELECT ename FROM emp WHERE job LIKE '_l%';
ALTER TABLE emp ADD COLUMNS(lastname STRING);
ALTER TABLE emp RENAME TO employee;
DROP TABLE emp;
DROP DATABASE emp_details;
```

  Query/command               Purpose
  --------------------------- ---------------------------
  `SHOW TABLES`               Lists tables
  `DESCRIBE`                  Displays table schema
  `SELECT *`                  Displays all columns/rows
  `WHERE`                     Filters rows
  `COUNT(1)`                  Counts rows
  `SUM(sal)`                  Adds salary values
  `GROUP BY`                  Groups records
  `BETWEEN`                   Checks inclusive range
  `LIKE '_____'`              Matches five characters
  `LIKE '_l%'`                Second character is `l`
  `ALTER TABLE ADD COLUMNS`   Adds column
  `RENAME TO`                 Renames table
  `DROP TABLE`                Deletes table
  `DROP DATABASE`             Deletes database

**Caution:** `ALTER TABLE emp RENAME TO employee` changes the table
name. Subsequent queries must use `employee`, not `emp`, unless `emp` is
recreated.

## B. Hive -- User Data Analysis (syllabus task)

Dataset fields: - `data_date` - `user_id` - `properties` (key/value text
such as `Age=21;state=CA;gender=M;`)

### 1. Create database and external table

``` sql
CREATE DATABASE IF NOT EXISTS user_analysis;
USE user_analysis;

CREATE EXTERNAL TABLE IF NOT EXISTS userdata (
    data_date STRING,
    user_id STRING,
    properties STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/cloudera/hive_lab/input';
```

The external table points to data in the specified HDFS location. The
source files must already exist there.

### 2. Example input records

``` text
2025-11-03,User1,Age=21;state=CA;gender=M;
2025-11-03,User2,Age=25;state=TX;gender=F;
2025-11-03,User3,Age=32;state=CA;gender=M;
2025-11-03,User4,Age=18;state=NY;gender=F;
2025-11-03,User5,Age=40;state=TX;gender=M;
```

### 3. Extract properties

``` sql
CREATE TABLE userdata_parsed AS
SELECT
    data_date,
    user_id,
    regexp_extract(properties, 'Age=([0-9]+)', 1) AS age,
    regexp_extract(properties, 'state=([A-Z]+)', 1) AS state,
    regexp_extract(properties, 'gender=([A-Z]+)', 1) AS gender
FROM userdata;
```

`regexp_extract()` extracts a matching value from the properties string.

### 4. Minimum, maximum and unique values

``` sql
SELECT MIN(CAST(age AS INT)) AS min_age,
       MAX(CAST(age AS INT)) AS max_age,
       COUNT(DISTINCT age) AS unique_ages
FROM userdata_parsed;
```

-   `MIN`: minimum value.
-   `MAX`: maximum value.
-   `COUNT(DISTINCT ...)`: number of unique values.

### 5. Count records per state

``` sql
SELECT state, COUNT(*) AS user_count
FROM userdata_parsed
WHERE state IS NOT NULL
GROUP BY state;
```

This groups users by state and counts records in each state.

**Note:** The sample data above is illustrative. Use the exact
faculty-provided dataset if one is supplied. The syllabus mentions
minimum, maximum and unique values for properties; confirm with faculty
whether the intended aggregation is over age or another extracted
property.

## C. Pig -- Pig Latin

### 1. Start Pig

``` bash
pig
```

Opens the Grunt shell.

### 2. Create input files

`pigfile.txt`

``` text
1,2,3
4,5,6
5,7,0
```

`pigfile1.txt`

``` text
2,4,5
2,4,5
4,7,9
```

Create using Linux:

``` bash
cat > pigfile.txt
cat > pigfile1.txt
```

Paste each file's content and press `Ctrl+D`.

### 3. Upload to HDFS

``` bash
hadoop fs -put pigfile.txt /user/cloudera/
hadoop fs -put pigfile1.txt /user/cloudera/
```

### 4. Load data into Pig

``` pig
a = LOAD '/user/cloudera/pigfile.txt' USING PigStorage(',');
b = LOAD '/user/cloudera/pigfile1.txt' USING PigStorage(',');
DUMP a;
DUMP b;
```

-   `LOAD`: reads data.
-   `PigStorage(',')`: comma delimiter.
-   `DUMP`: displays relation.

### 5. Load with schema

``` pig
a = LOAD '/user/cloudera/pigfile.txt'
USING PigStorage(',')
AS (a1:int, a2:int, a3:int);

b = LOAD '/user/cloudera/pigfile1.txt'
USING PigStorage(',')
AS (b1:int, b2:int, b3:int);

DESCRIBE a;
DESCRIBE b;
```

`AS` defines column names and datatypes. `DESCRIBE` displays schema.

### 6. UNION

``` pig
c = UNION a,b;
DUMP c;
```

Combines two relations with compatible schemas.

### 7. SPLIT

``` pig
SPLIT c INTO d IF $0 == 1, e IF $0 == 4;
DUMP d;
DUMP e;
```

Splits relation based on conditions. `$0` is the first column, `$1`
second, `$2` third.

### 8. FILTER

``` pig
f = FILTER c BY $1 > 6;
DUMP f;
```

Keeps records where the second column is greater than 6.

### 9. GROUP

``` pig
g = GROUP c BY $2;
DUMP g;
```

Groups records by the third column.

### 10. FOREACH and GENERATE

``` pig
s1 = FOREACH a GENERATE $1,$2;
DUMP s1;
```

Selects the second and third columns.

Expected output:

``` text
(2,3)
(5,6)
(7,0)
```

### 11. STORE

``` pig
STORE s1 INTO '/user/cloudera/pigresult';
```

Saves the relation in HDFS.

Verify in Linux terminal:

``` bash
hadoop fs -ls /user/cloudera/pigresult/
hadoop fs -cat /user/cloudera/pigresult/part-m-00000
```

## D. Pig -- Simple Logs (syllabus task)

Input fields:

``` text
site
hour_of_day
page_views
data_date
```

The syllabus says the log files are tab-delimited and are located in the
`prepare` folder. It specifically mentions:

``` pig
PigStorage('\u0001')
```

Required outputs: 1. Total views per hour per day. 2. Total views per
day. 3. Total views for each hour across all days. 4. Word Count script
using text files as input.

**Source limitation:** The supplied syllabus specifies the fields,
delimiter instruction and required calculations, but the complete
faculty Simple Logs input and program were not available in the
retrieved practical material. Do not assume comma-separated input for
this task.

------------------------------------------------------------------------

# UNIT IV -- SQOOP

**Aim:** Transfer data between MySQL and Hadoop/HDFS.

## A. Import MySQL table into HDFS

### 1. Open MySQL

``` bash
mysql -uroot -pcloudera
```

### 2. Create and select database

``` sql
CREATE DATABASE sqoop_db;
USE sqoop_db;
```

### 3. Create employee table

``` sql
CREATE TABLE emp(
    empno INT PRIMARY KEY,
    ename VARCHAR(10),
    job VARCHAR(9),
    mgr INT,
    hiredate DATE,
    sal INT,
    deptno INT
);
```

### 4. Insert records

``` sql
INSERT INTO emp VALUES
(1,'ABC','teacher',100,'2013-01-12',50000,10);

INSERT INTO emp VALUES
(2,'XYZ','teacher',200,'2014-10-02',650000,20);

INSERT INTO emp VALUES
(3,'PQRS','teacher',200,'2010-10-02',640000,30);

SELECT * FROM emp;
EXIT;
```

### 5. Import using Sqoop

Run in Linux terminal:

``` bash
sqoop import --connect jdbc:mysql://localhost/sqoop_db --username root --password cloudera --table emp --target-dir /user/cloudera/scoop_data
```

  Option           Purpose
  ---------------- ----------------------------------
  `sqoop import`   Imports database table to Hadoop
  `--connect`      JDBC connection URL
  `--username`     Database username
  `--password`     Database password
  `--table`        Source table
  `--target-dir`   HDFS destination

### 6. Verify import

``` bash
hadoop fs -ls /user/cloudera/scoop_data
hadoop fs -cat /user/cloudera/scoop_data/part-m-00000
```

The actual number of part files can vary with Sqoop's mapper
configuration.

## B. Export HDFS file to MySQL

### 1. Create local file

``` bash
cd ~/Desktop
cat > abc.txt
```

Enter:

``` text
1,2,3
4,5,6
7,8,9
```

Press `Ctrl+D`.

### 2. Upload to HDFS

``` bash
hadoop fs -put abc.txt /user/cloudera/
```

### 3. Create destination table in MySQL

``` bash
mysql -uroot -pcloudera
```

``` sql
USE sqoop_db;

CREATE TABLE abc_table(
    a INT,
    b INT,
    c INT
);

EXIT;
```

### 4. Export

``` bash
sqoop export --connect jdbc:mysql://localhost/sqoop_db --username root --password cloudera --table abc_table --export-dir /user/cloudera/abc.txt
```

-   `sqoop export`: exports HDFS data to relational database.
-   `--table`: destination MySQL table.
-   `--export-dir`: source path in HDFS.

### 5. Verify export

``` bash
mysql -uroot -pcloudera
```

``` sql
USE sqoop_db;
SELECT * FROM abc_table;
```

Expected:

``` text
1  2  3
4  5  6
7  8  9
```

**Direction to remember:** Import = MySQL → HDFS; Export = HDFS → MySQL.

------------------------------------------------------------------------

# UNIT V -- Data Visualization and Machine Learning

## A. Data Visualization using Python

**Dataset:** Iris

### Complete program

``` python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px

# Load Dataset
data = sns.load_dataset('iris')

# 1. Histogram
plt.figure(figsize=(8, 5))
sns.histplot(data['sepal_length'], kde=True, color='purple')
plt.title('Histogram of Sepal Length')
plt.xlabel('Sepal Length')
plt.ylabel('Frequency')
plt.show()

# 2. Box Plot
plt.figure(figsize=(8, 5))
sns.boxplot(x='species', y='sepal_length', data=data, palette='Set2')
plt.title('Box Plot of Sepal Length by Species')
plt.show()

# 3. Scatter Plot with Regression Line
plt.figure(figsize=(8, 5))
sns.regplot(x='sepal_length', y='petal_length', data=data, color='blue')
plt.title('Scatter Plot with Regression Line (Sepal vs Petal Length)')
plt.show()

# 4. Correlation Heatmap
plt.figure(figsize=(8, 6))
numeric_df = data.drop(columns=['species'])
sns.heatmap(numeric_df.corr(), annot=True, cmap='coolwarm', fmt='.2f')
plt.title('Correlation Heatmap')
plt.show()

# 5. Interactive 3D Scatter Plot
fig = px.scatter_3d(
    data,
    x='sepal_length',
    y='sepal_width',
    z='petal_length',
    color='species',
    title='3D Interactive Plot of Iris Dataset'
)
fig.show()
```

If needed in Google Colab:

``` python
!pip install seaborn plotly
```

  Function            Purpose
  ------------------- -----------------------------------
  `sns.histplot()`    Histogram
  `sns.boxplot()`     Box plot
  `sns.regplot()`     Scatter plot with regression line
  `sns.heatmap()`     Correlation heatmap
  `px.scatter_3d()`   Interactive 3D scatter plot
  `plt.show()`        Displays plot
  `kde=True`          Adds density curve
  `annot=True`        Displays values on heatmap

## B. K-Means using PySpark MLlib (practical-record program)

**Aim:** Implement an unsupervised K-Means clustering pipeline using
PySpark MLlib on the Iris dataset.

### Google Colab setup

``` python
!pip install pyspark
```

### Complete program

``` python
# Import libraries
import seaborn as sns
import matplotlib.pyplot as plt
from pyspark.sql import SparkSession
from pyspark.ml.feature import VectorAssembler
from pyspark.ml.clustering import KMeans
from pyspark.ml.evaluation import ClusteringEvaluator

# Initialize Spark Session
spark = SparkSession.builder.appName("KMeans").getOrCreate()

# Load Dataset
data = sns.load_dataset('iris')
spark_df = spark.createDataFrame(data)

# Display Schema
print("Spark Schema:")
spark_df.printSchema()

# Assemble Features
feature_cols = [
    'sepal_length',
    'sepal_width',
    'petal_length',
    'petal_width'
]

assembler = VectorAssembler(
    inputCols=feature_cols,
    outputCol="features"
)

dataset = assembler.transform(spark_df)
dataset.show(5, truncate=False)

# Train K-Means
kmeans = KMeans(featuresCol="features", k=3, seed=42)
model = kmeans.fit(dataset)
predictions = model.transform(dataset)

predictions.select('species', 'prediction').show(10, truncate=False)

# Evaluate using Silhouette Score
evaluator = ClusteringEvaluator()
silhouette = evaluator.evaluate(predictions)
print("Silhouette Score:", silhouette)

# Display Cluster Centers
centers = model.clusterCenters()
print("Cluster Centers:")
for center in centers:
    print(center)

# Cross-Tabulation
predictions.crosstab('prediction', 'species').show()

# Visualize Predictions
pred_pd = predictions.select(
    "sepal_length",
    "sepal_width",
    "prediction",
    "species"
).toPandas()

plt.figure(figsize=(9, 6))
sns.scatterplot(
    data=pred_pd,
    x='sepal_length',
    y='sepal_width',
    hue='prediction',
    style='species',
    palette='Set1',
    s=100
)

plt.title('K-Means Clustering Visualization (Sepal Length vs. Sepal Width)')
plt.xlabel('Sepal Length')
plt.ylabel('Sepal Width')
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
plt.tight_layout()
plt.show()

spark.stop()
```

### Short version for recall

``` python
assembler = VectorAssembler(inputCols=feature_cols, outputCol="features")
dataset = assembler.transform(spark_df)

kmeans = KMeans(featuresCol="features", k=3, seed=42)
model = kmeans.fit(dataset)
predictions = model.transform(dataset)

evaluator = ClusteringEvaluator()
silhouette = evaluator.evaluate(predictions)
print(silhouette)

centers = model.clusterCenters()
predictions.crosstab('prediction', 'species').show()
```

  Function                  Purpose
  ------------------------- ------------------------------------------
  `SparkSession`            Starts Spark
  `VectorAssembler`         Combines feature columns into a vector
  `KMeans`                  Creates clustering model
  `k=3`                     Specifies three clusters
  `fit()`                   Trains model
  `transform()`             Produces cluster predictions
  `ClusteringEvaluator()`   Evaluates clustering
  `silhouette`              Measures cluster quality
  `clusterCenters()`        Returns cluster centers
  `crosstab()`              Compares predicted clusters with species

**Syllabus/record distinction:** The syllabus mentions Mahout Machine
Learning Library, while the practical record's implementation uses
PySpark MLlib. This program follows the actual practical-record
implementation; it is not a Mahout Java program.

------------------------------------------------------------------------

# FINAL QUICK REFERENCE

## Important HDFS commands

``` bash
hadoop fs -mkdir
hadoop fs -put
hadoop fs -get
hadoop fs -ls
hadoop fs -cat
hadoop fs -cp
hadoop fs -mv
hadoop fs -rm
hadoop fs -rm -r
hdfs fsck /
```

## Important Hive commands

``` sql
CREATE DATABASE;
USE;
CREATE TABLE;
LOAD DATA;
SHOW TABLES;
DESCRIBE;
SELECT;
WHERE;
GROUP BY;
COUNT();
SUM();
MIN();
MAX();
COUNT(DISTINCT column);
ALTER TABLE;
DROP TABLE;
```

## Important Pig commands

``` pig
LOAD
PigStorage()
DUMP
DESCRIBE
UNION
SPLIT
FILTER
GROUP
FOREACH
GENERATE
STORE
```

## Important MapReduce concepts

-   Mapper: generates intermediate key/value pairs.
-   Reducer: aggregates values.
-   Driver: configures and runs the job.
-   JAR: packages compiled Java classes.
-   HDFS: stores input and output.

## Important Sqoop commands

``` bash
sqoop import
sqoop export
```

## Important Python/PySpark functions

``` python
sns.histplot()
sns.boxplot()
sns.regplot()
sns.heatmap()
px.scatter_3d()
VectorAssembler()
KMeans()
ClusteringEvaluator()
model.fit()
model.transform()
model.clusterCenters()
```

------------------------------------------------------------------------

# Practical completion checklist

-   [ ] Unit I -- HDFS commands
-   [ ] Unit II -- Word Count
-   [ ] Unit II -- Most Frequent Words
-   [ ] Unit II -- K-Means MapReduce
-   [ ] Unit II -- Maximum Temperature (additional record practical)
-   [ ] Unit III -- Hive employee queries
-   [ ] Unit III -- Hive user data analysis
-   [ ] Unit III -- Pig basic operations
-   [ ] Unit III -- Pig Simple Logs
-   [ ] Unit IV -- Sqoop import/export
-   [ ] Unit V -- Python visualization
-   [ ] Unit V -- PySpark K-Means

**Manual note:** This README combines the supplied syllabus,
practical-record commands, and programs already worked through. Sections
explicitly marked as source limitations still need the corresponding
faculty source code/input before they can be treated as complete
executable practicals.
