# Spark-Inverted-Index
Regis University's Data Engineering (MSDS 610) Final Project

## Summary

For this project, we will create an inverted index using data collected from Stack Overflow, which is available on StackExchange. An inverted index is commonly used as an example for MapReduce analytics. In this case, we want to build a map of some term (keyword, unique ID) to a list of identifiers (keyword A, list of IDs). The purpose of creating an index from a data set is to allow for faster searches or data enrichment capabilities. We will query Stack Overflow’s data API to create and download a dataset. This is done by following the Stack Overflow link on StackExchange and then by clicking ‘Compose Query’. After the data is downloaded as a CSV, we will use Spark, Jupyter Notebook, and a Google Cloud Platform Dataproc instance (or cluster) to create an inverted index that lists all the Post ID’s where each tag appears. 

## Data

The data used for this project needs to be collected from the Stack Overflow website. This involves using SQL dialects to query the data. The query used to obtain the data is:

```sql
SELECT Id, Tags
FROM Posts
WHERE Tags!= ‘’ and CreationDate like '%2017%';
```

We add the `CreationData like %2017%` to reduce the amount of data we query, and to limit query time. The results of this query contains approximately 50,000 rows of data. From here, a `csv` file was created to be uploaded into Spark later. It should be noted that the original `csv` file `QueryResults_unedited.csv` contains the characters `<` and `>`. These were removed by manually replacing the `<` and `>` characters with `,`. This version of the data was saved in a file names `QueryResults_edited.csv`, which was used for this project.

## Using Spark

There are various options for running Spark. It can be run on your local machine, but the "easier" choice is to use a GCP cloudproc cluster because they have Spark and Pyspark installed by default. To run a Jupyter notebook with a GCP cloudproc instance, a SSH tunnel eeds to be set up. By using the GCP method like this one, we can work with a PySpark kernel in Jupyter notebook to avoid having to download Spark internally. From here, we can load Spark and SparkContext, which is the main entry point for Spark functionality.

### Uploading and Using the Data

To use our collected data, we need to upload our file to Google Bucket Storage, which is is the basic containers that hold your data in Cloud Storage. Everything that you store in Cloud Storage must be contained in a bucket, which can be used to organize your data and control access to your data. To upload our data, we can simply drag-and-drop the files into the bucket. 

Now, going back to the Jupyter notebook, we can use the command `SparkContext.getOrCreate()` to create the spark context and `SparkSession.builder.getOrCreate()` to create the main entry point for dataframe and SQL functionality. Next, we can read our data from Google Bucket storage and store the data in a dataframe. 

```python
df = spark.read.csv(‘gs://<file path>’) #where file path is the path to the dataproc bucket
df.take(5) #ensure data was loaded properly; shows an array of the first 5 elements of the dataframe
```

Next, we need to convert our dataframe into an RDD because we cannot do the necessary data transformations directly on a spark dataframe. An RDD is a “Resilient Distributed Dataset, which are elements that are run and operated on multiple nodes to do parallel processing on a cluster” (PySpark-Rdd-Tutorialspoint). They are immutable elements, which means that once you create an RDD you cannot change it. There are two operations that can be applied to RDD’s, which are transformations and actions. A transformation, also known as a lazy operation, is an operation applied on a RDD to create a new RDD. Examples include filter, groupByKey, and map. An action is an operation applied on an RDD that instructs Spark to perform a computation and send the result back to the driver. Examples of this include .collect(), .show(), and .take().

```python
rdd = df.rdd # convert dataframe to an RDD; calls the pyspark.sql.DataFrame.rdd() (each row is a list of key-value pairs)
type(rdd)
rdd.take(10) #view first 10 rows
```

After further review of our data, we find that our data contains rows with empty cells. Our query above filters out null tags, but not null ID's. To filter out rows with null values, we can utilize the `lambda` function. Using this data, we can create a function that uses `.split()` and `.strip()`, which splits the `Tags` to individual tags. This allows us to create the inverted index. The `.split()` method returns a list of strings after splitting on a specified separator and `.strip()` returns a copy of the string with both leading and trailing characters removed. Below, we strip on whitespaces and split on commas, and then use the `map()` function to return a new distributed dataset that is formed by passing each element of the source through a function. 


```python
no_null = rdd.filter(lambda x: x_c0 != '' and x.c1 != '')

def split_strip(x):
    return (x._c0.strip(), x._c1.split(',')) # .strip() strips whitespaces, and .split(',') splits on commas 

no.null.map(split_strip.take(5)) #ensure function above returns proper data

split_id = no.null.map(split_strip) # saves distributed dataset
```




## Resources

1. Apache spark groupbykey function—Javatpoint. (n.d.). Www.Javatpoint.Com. Retrieved March 7, 2020, from https://www.javatpoint.com/apache-spark-groupbykey-function
2. Apache spark take function—Javatpoint. (n.d.). Www.Javatpoint.Com. Retrieved March 7, 2020, from https://www.javatpoint.com/apache-spark-take-function
3. Gupta, A. (2016a, September 16). Comprehensive introduction—Apache spark, rdds & dataframes (Pyspark). Analytics Vidhya. https://www.analyticsvidhya.com/blog/2016/09/comprehensive-introduction-to-apache-spark-rdds-dataframes-using-pyspark/
4. Gupta, A. (2016b, October 5). Using pyspark to perform transformations and actions on rdd. Analytics Vidhya. https://www.analyticsvidhya.com/blog/2016/10/using-pyspark-to-perform-transformations-and-actions-on-rdd/
5. Khan, V. (2016, November 22). Difference Between flatMap() and map() on an RDD. https://github.com/vaquarkhan/Apache-Kafka-poc-and-notes/wiki/Difference-between-flatMap()-and-map()-on-an-RDD
6. Malone, J. (2016, June 9). Google Cloud Dataproc—The fast, easy and safe way to try Spark 2.0-preview. Google Cloud Blog. https://cloud.google.com/blog/
7. Ming, C. (2018, August 29). Learning Apache Spark. GitHub. https://github.com/MingChen0919/learning-apache-spark
8. Mishra, R. (2019, June 27). Lambda, map, and filter in python. Medium. https://medium.com/better-programming/lambda-map-and-filter-in-python-4935f248593
9. Pyspark—Rdd—Tutorialspoint. (n.d.). Retrieved March 7, 2020, from https://www.tutorialspoint.com/pyspark/pyspark_rdd.htm
10. PySpark.sql Module. (n.d.). Module Context. https://spark.apache.org/docs/latest/api/python/pyspark.sql.html?highlight=sparksession
11. Python for Data Science. (n.d.). https://s3.amazonaws.com/assets.datacamp.com/blog_assets/PySpark_SQL_Cheat_Sheet_Python.pdf
12. Python lambda. (2017, October 3). GeeksforGeeks. https://www.geeksforgeeks.org/python-lambda-anonymous-functions-filter-map-reduce/
13. Python string methods | programiz. (n.d.). Retrieved March 7, 2020, from https://www.programiz.com/python-programming/methods/string

