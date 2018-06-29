Basics of Spark on HDInsight
Apache Spark is an open-source parallel processing framework that supports in-memory processing to boost the performance of big-data analytic applications. When you provision a Spark cluster in HDInsight, you provision Azure compute resources with Spark installed and configured. The data to be processed is stored in Azure Blob storage (WASB).

Spark on HDInsight

Now that you have created a Spark cluster, let us understand some basics of working with Spark on HDInsight. For detailed discussion on working with Spark, see Spark Programming Guide.

Notebook setup
When using PySpark kernel notebooks on HDInsight, there is no need to create a SparkContext or a SparkSession; a SparkSession which has the SparkContext is created for you automatically when you run the first code cell, and you'll be able to see the progress printed. The contexts are created with the following variable names:

SparkSession (spark)
To run the cells below, place the cursor in the cell and then press SHIFT + ENTER.

What is an RDD?
Big Data applications rely on iterative, distributed computing for faster processing of large data sets. To distribute data processing over multiple jobs, the data is typically reused or shared across jobs. To share data between existing distributed computing systems you need to store data in some intermediate stable distributed store such as HDFS. This makes the overall computations of jobs slower.

Resilient Distributed Datasets or RDDs address this by enabling fault-tolerant, distributed, in-memory computations.

How do I make an RDD?
RDDs can be created from stable storage or by transforming other RDDs. Run the cells below to create RDDs from the sample data files available in the storage container associated with your Spark cluster. One such sample data file is available on the cluster at wasb:///example/data/fruits.txt.

fruits = spark.sparkContext.textFile('wasb:///example/data/fruits.txt')
yellowThings = spark.sparkContext.textFile('wasb:///example/data/yellowthings.txt')
For more examples on how to create RDDs see the following notebooks available with your Spark cluster:

Read and write data from Azure Storage Blobs (WASB)
Read and write data from Hive tables
What are RDD operations?
RDDs support two types of operations: transformations and actions.

Transformations create a new dataset from an existing one. Transformations are lazy, meaning that no transformation is executed until you execute an action.
Actions return a value to the driver program after running a computation on the dataset.
RDD transformations
Following are examples of some of the common transformations available. For a detailed list, see RDD Transformations

Run some transformations below to understand this better. Place the cursor in the cell and press SHIFT + ENTER.

# map
fruitsReversed = fruits.map(lambda fruit: fruit[::-1])
# filter
shortFruits = fruits.filter(lambda fruit: len(fruit) <= 5)
# flatMap
characters = fruits.flatMap(lambda fruit: list(fruit))
# union
fruitsAndYellowThings = fruits.union(yellowThings)
# intersection
yellowFruits = fruits.intersection(yellowThings)
# distinct
distinctFruitsAndYellowThings = fruitsAndYellowThings.distinct()
distinctFruitsAndYellowThings
# groupByKey
yellowThingsByFirstLetter = yellowThings.map(lambda thing: (thing[0], thing)).groupByKey()
# reduceByKey
numFruitsByLength = fruits.map(lambda fruit: (len(fruit), 1)).reduceByKey(lambda x, y: x + y)
RDD actions
Following are examples of some of the common actions available. For a detailed list, see RDD Actions.

Run some transformations below to understand this better. Place the cursor in the cell and press SHIFT + ENTER.

# collect
fruitsArray = fruits.collect()
yellowThingsArray = yellowThings.collect()
fruitsArray
# count
numFruits = fruits.count()
numFruits
# take
first3Fruits = fruits.take(3)
first3Fruits
# reduce
letterSet = fruits.map(lambda fruit: set(fruit)).reduce(lambda x, y: x.union(y))
letterSet
IMPORTANT: Another important RDD action is saving the output to a file. See the Read and write data from Azure Storage Blobs (WASB) notebook for more information.

What is a dataframe?
The pyspark.sql library provides an alternative API for manipulating structured datasets, known as "dataframes". (Dataframes are not a Spark-specific concept but pyspark provides its own dedicated dataframe library.) These are different from RDDs, but you can convert an RDD into a dataframe or vice-versa, if required.

See Spark SQL and DataFrame Guide for more information.

How do I make a dataframe?
You can load a dataframe directly from an input data source. See the following notebooks included with your Spark cluster for more information.

Read and write data from Azure Storage Blobs (WASB)
Read and write data from Hive tables
You can also create a dataframe from a CSV file as shown below.

df = spark.read.csv('wasb:///HdiSamples/HdiSamples/SensorSampleData/building/building.csv', header=True, inferSchema=True)
Dataframe operations
Run the cells below to see examples of some of the the operations that you can perform on dataframes.

# show the content of the dataframe
df.show()
# Print the dataframe schema in a tree format
df.printSchema()
# Create an RDD from the dataframe
dfrdd = df.rdd
dfrdd.take(3)
dfrdd = df.rdd
dfrdd.take(3)
# Retrieve a given number of rows from the dataframe
df.limit(3).show()
df.limit(3).show()
# Retrieve specific columns from the dataframe
df.select('BuildingID', 'Country').limit(3).show()
# Use GroupBy clause with dataframe 
df.groupBy('HVACProduct').count().select('HVACProduct', 'count').show()
IMPORTANT: Many of the methods available on normal RDDs are also available on dataframes. For example, distinct, count, collect, filter, map, and take are all methods on dataframes as well as on RDDs.

Spark SQL and dataframes
You can also run SQL queries over dataframes once you register them as temporary tables within the SparkSession. Run the snippet below to see an example.

# Register the dataframe as a temporary table called HVAC
df.registerTempTable('HVAC')
%%sql
SELECT * FROM HVAC WHERE BuildingAge >= 10
%%sql 
SELECT BuildingID, Country FROM HVAC LIMIT 3