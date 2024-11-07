Home Sales Data Analysis with PySpark
This project involves using PySpark to load, analyze, and optimize queries on home sales data. We work with data from an AWS S3 bucket, cache it for performance improvement, and perform various transformations and calculations, including partitioning and Parquet formatting for efficient storage and access.

Prerequisites
Python and PySpark installed.
Spark environment initialized with findspark.
A working internet connection to access the dataset from AWS S3.
Steps
Step 1: Initialize Spark
We start by initializing Spark using findspark and setting up a SparkSession.

python
Copy code
import findspark
findspark.init()

from pyspark.sql import SparkSession
import time

spark = SparkSession.builder.appName("SparkSQL").getOrCreate()
Step 2: Load Data from AWS S3 into a DataFrame
We load a CSV file containing home sales data from an AWS S3 bucket into a PySpark DataFrame.

python
Copy code
from pyspark import SparkFiles

url = "https://2u-data-curriculum-team.s3.amazonaws.com/dataviz-classroom/v1.2/22-big-data/home_sales_revised.csv"
spark.sparkContext.addFile(url)
df = spark.read.csv(SparkFiles.get("home_sales_revised.csv"), header=True, inferSchema=True)
df.show(5)
Step 3: Create a Temporary View
We create a temporary view called home_sales to allow SQL queries on the DataFrame.

python
Copy code
df.createOrReplaceTempView("home_sales")
Step 4-6: Run Queries
We run several queries to gather insights:

Average Price of 4-Bedroom Homes Sold Per Year: Calculates the average price for homes with 4 bedrooms, grouped by year.
Average Price of 3-Bedroom, 3-Bathroom Homes by Year Built: Filters homes with 3 bedrooms and 3 bathrooms and calculates the average price by the year built.
Average Price for Specific Home Features by Year Built: Calculates the average price for homes with 3 bedrooms, 3 bathrooms, 2 floors, and at least 2,000 square feet.
Average Price by View Rating: Calculates the average price per view rating, filtered for view ratings with an average home price of at least $350,000. This query is timed to observe the runtime.
python
Copy code
# Sample Query
query = """
SELECT 
    view, 
    ROUND(AVG(price), 2) AS avg_price 
FROM 
    home_sales 
GROUP BY 
    view 
HAVING 
    avg_price >= 350000 
ORDER BY 
    view DESC
"""

start_time = time.time()
result = spark.sql(query)
result.show()
print("--- %s seconds ---" % (time.time() - start_time))
Step 7: Cache the Table
We cache the home_sales temporary table to improve query performance for repeated access.

python
Copy code
spark.catalog.cacheTable("home_sales")
Step 8-9: Rerun the View Rating Query with Cached Data
We rerun the view rating query on the cached table and measure the runtime to compare with the uncached runtime.

Step 10-12: Partition and Save as Parquet
We partition the data by date_built and save it in Parquet format to enable efficient storage and access.

python
Copy code
df.write.partitionBy("date_built").parquet("/mnt/data/home_sales_partitioned")
parquet_df = spark.read.parquet("/mnt/data/home_sales_partitioned")
parquet_df.createOrReplaceTempView("home_sales_parquet")
Step 13: Run Query on Parquet Data
Using the Parquet-based temporary table, we rerun the view rating query and measure the runtime to compare it with both cached and uncached versions.

Step 14: Uncache the Table
Finally, we uncache the home_sales table to free up resources.

python
Copy code
spark.catalog.uncacheTable("home_sales")
Conclusion
This project demonstrates efficient data handling and querying using PySpark. By leveraging temporary views, caching, partitioning, and Parquet storage, we improved query performance and explored how different data management techniques impact runtime.
