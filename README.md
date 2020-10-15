Bases de données avancées - Big Data: Hadoop/MapReduce
===

# News

2020/10/01 TP Parts 1-2 instructions added

# Prerequisites

There are many issues to build and run this code (and Hadoop in general) in Windows so please use the recommended VM.

## Recommended Virtual Machine

Install [Virtualbox](https://www.virtualbox.org/wiki/Downloads) and the provided Virtual Machine.

You can download the Virtual Machine also from this link (5.3 GB), remember that you need to
allocate at least 4GB to the VM:
[https://downloads.cloudera.com/demo_vm/virtualbox/cloudera-quickstart-vm-5.13.0-0-virtualbox.zip]

## Do it yourself on your own linux machine (for those struggling with the Hadoop VM)

You can download a recent Linux image here and install it locally or in a VM:

https://releases.ubuntu.com/20.04.1/ubuntu-20.04.1-desktop-amd64.iso

After installing open a terminal and run:

    sudo apt install openjdk-8-jdk mvn
    sudo apt install python3

WARNING: Please validate that you are using Java 8!

- Verify you have a correct Java installation with the JAVA_HOME variable configured.
- Install a Java IDE (IntelliJ or Eclipse), or a decent editor.
- Install Maven in case you don't have it, if you are using Eclipse verify that you have the latest
  version or install the m2e plugin to import the maven project.

# Part 1

Clone this repo.

    git clone https://github.com/iemejia/formation-bigdata

1. Finish the implementation of Wordcount as seen in the class and validate that it works well.

You can use a file of your own or download a book from the gutenberg project, you find also the example
file dataset/hamlet.txt

2. Modify the implementation to return only the words that start with the letter 'm'.

- Where did you change it, in the mapper or in the reducer ?
- What are the consequences of changing the code in each one ?
- Compare the Hadoop counters results and explain which one is the best strategy.

## Hadoop HDFS first steps (Inside the Cloudera VM)

We are going to run the wordcount example of the course, so we need first to add the file to the distributed file system.
So first we must connect to the master server (ssh like in 1) and do:

    hadoop fs -mkdir hdfs:///user/id##/dataset/wordcount/

or

    hadoop fs -mkdir hdfs:///user/gcn##/dataset/wordcount/

depending on your master and group (##).

Upload the file

    hadoop fs -put hamlet.txt hdfs:///user/id##/dataset/wordcount/

Verify the upload

    hadoop fs -ls hdfs:///user/id##/dataset/wordcount/

This one for remote IPs with the corresponding mapping (port 8020 or 9000)

    hadoop fs -ls hdfs://172.17.0.2/tp1/

You can browse the cluster web interfaces:

Resource Manager - http://172.17.0.2:8088/
HDFS Name Node - http://172.17.0.2:50070/

## Running a mapreduce job in the YARN cluster

Copy the program to the cluster

The structure is:

    hadoop jar JAR_FILE CLASS input output

    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples*.jar wordcount hdfs:///user/id00/dataset/wordcount/hamlet.txt hdfs:///user/id00/output/wordcount/

The hadoop mapreduce examples is available in the following path in the cluster:

    /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar

Verify the output

    hadoop fs -cat hdfs:///user/id00/output/wordcount/*

For example if the IP is 172.17.0.2 The following are the addresses for the different web interfaces (this applies for the VM or the Cluster):

- NameNode http://172.17.0.2:50070/
- ResourceManager http://172.17.0.2:8088/
- MapReduce JobHistory Server http://172.17.0.2:19888/
- Hue http://172.17.0.2:8888/

## References

- https://developer.yahoo.com/hadoop/tutorial/
- http://blog.cloudera.com/blog/2009/08/hadoop-default-ports-quick-reference/


# Part 2

3. Use the GDELT dataset and implement a Map Reduce job to find the top 10 countries that have more
relevance in the news for a given time period (one week, one day, one month).

For more info about gdelt column format you can find more info at:
http://data.gdeltproject.org/documentation/GDELT-Data_Format_Codebook.pdf

You can find a small test sample of the dataset at dataset/gdeltmini/20160529.export.csv
that you can use to test your implementation.

You can also download the full datasets from:
http://data.gdeltproject.org/events/index.html

We will consider the country code as the three letter identifier represented as Actor1CountryCode, we
will count the relevance of an event based on its NumMentions column. So this become likes a
WordCount where we count the NumMentions of each event per country to determine the Top 20.

For ref. Actor1CountryCode is column 7 and NumMentions is column 31 on the GDELT file format.

If using Hadoop, add a combiner to the job? Do you see any improvement in the counters? Explain and compare the values. What could go wrong?


# Part 3 (Spark)

You will need a recent Linux installation. You can install a VM with Ubuntu >= 18.04 (available in the class) or use the lab computers (if you choose this route pay attention to paths that are unreadable in the machine in particular the one where you install the project).

## Environment setup

Create a python virtualenv

    python -m venv spark-env

Activate the venv

    source spark-env/bin/activate

Upgrade setup tools

    pip install --upgrade pip setuptools wheel

Install dependencies

    pip install pyspark jupyter ipython black

## You can now work with pyspark for that you can use one of three options (you choose one):

- Interactive Python (ipython) REPL [RECOMMENDED]

    ipython

- Interactive Notebook

    jupyter notebook

- Python editor / IDE

If you use Visual Studio Code you must install the Python extension

    code .

## Getting familiar with pyspark

Take a quick look at the Spark quick start and RDD guides to get familiar with Spark
https://spark.apache.org/docs/latest/quick-start.html
https://spark.apache.org/docs/latest/rdd-programming-guide.html

## Implement wordcount filtering the words that start with 'm' on pyspark

Spark has two main APIs, for this exercise we use the RDD one.

    ```python
    from pyspark.sql import SparkSession
    from pyspark.sql import functions as f

    spark = SparkSession.builder.master('local').appName('BigDataTP').getOrCreate()
    sc = spark.sparkContext

    text_file = sc.textFile("dataset/wordcount/hamlet.txt")
    text_file.collect()

    # Add your implementation here
    ```

Once you have the current results, take a look at the running jobs in the spark console:
http://localhost:4040/jobs/


# Homework

Implement the GDELT countries with most mentions analysis in PySpark using the Dataframe/Dataset API.
You have to do the analysis for the month code assigned for your code, e.g. 202001 for group 01, etc.
(if you are in group 13 take 201901, group 14 then 201902, etc.)

You may find the identifiers for other columns here:
https://github.com/aamend/spark-gdelt/blob/master/src/main/scala/com/aamend/spark/gdelt/GdeltParser.scala

1. Download the files corresponding to the month of your pair e.g. 201901...

    wget http://data.gdeltproject.org/events/20200101.export.CSV.zip
    wget http://data.gdeltproject.org/events/20200102.export.CSV.zip
    ...

Unzip them and put them all the CSVs in the same directory

    unzip "*.zip"
    mv *.CSV dataset/gdelt

2. Take a look at the dataframe/dataset/sql pyspark guides in particular the cheatsheet and get familiar with the different operations:

- https://spark.apache.org/docs/latest/api/python/pyspark.sql.html
- https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html
- https://s3.amazonaws.com/assets.datacamp.com/blog_assets/PySpark_SQL_Cheat_Sheet_Python.pdf

    ```python
    gdelt = spark.read.option("delimiter", "\\t").csv('dataset/gdelt')
    gdelt.printSchema

    # Add your implementation
    # 1. Filter the rows where Actor1CountryCode does not have a value
    # 2. Group them by Actor1CountryCode
    # 3. Sum them by NumMentions
    # 4. Order by NumMentions and get the 10 more mentioned
    # 5. Write the results to a CSV file
    ```

3. Convert the CSV data into Parquet format

```python
gdelt.write.parquet('dataset/gdelt-parquet')
gdelt_parquet = spark.read.parquet('dataset/parquet')
# 6. Run the same analysis of points (1-5) on the new `gdelt_parquet` dataset.
```

4. Do the SQL version of the analysis

```python
gdelt.createTempView('gdelt')
spark.sql("SELECT ...").collect()
```

Your results should be the same than in the Dataset version if not double check.

5. Do an interesting analysis from the GDELT dataset using Spark's Dataset API.

Take a look at the code book format, in particular the Actor/CAMEO codes, and do an interesting new analysis, some ideas:

- Compare the number of news for a given pair of actors, do you remark some bias.
- Compare religions, is news reporting biased for certain religions (see tone).
- How much influence had some country vs another (in pure count terms).
- Top of organizations (group code) mentions per month.

Use joins for this, you can find some reference tables for some codes e.g. ethnic, religion, knowngroup here:
https://github.com/carrillo/Gdelt/tree/master/resources/staticTables

You can find more info on join types on pyspark here:
https://luminousmen.com/post/introduction-to-pyspark-join-types


## Deliverable

You have to send me a document with:

1. The source code to do GDELT's countries with most mentions analysis using the Dataframe/Dataset API (python file) and the output of the execution (csv file) [2]
2. A screenshot of the DAG visualization of the job with a short explanation of the execution plan. Take a look at the final dataset + `.explain()` for ideas [2].
3. A screenshot where I can see the duration of the CSV based job vs the Parquet based job. Add a short explanation to explain the differences (if any) [3].
4. The SQL query that makes the same analysis (countries with most mentions) with the gdelt dataset [4].
5. The explanation, code and results of the interesting analysis you did from the GDELT dataset [5].
What strategy was used by the join in your analysis (see explain() and detail).

Extra points:
If you did the GDELT analysis with Hadoop, please send me the code and the Hadoop counters of your execution.
Are the results different? if so why?
