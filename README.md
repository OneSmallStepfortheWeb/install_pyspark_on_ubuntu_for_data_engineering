# Step by step tuts to setup Apache Spark ( pySpark ) on Ubuntu and setup an environment for data engineering.


## Step 1 : Installing Python 3.10 on Ubuntu with Apt
Installing Python 3.10 on Ubuntu with apt is a relatively straightforward process and takes only a few minutes to complete.

Update the packages list and install the prerequisites:
``` console
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common
```

Add the deadsnakes PPA to your system’s sources list:
``` console
sudo add-apt-repository ppa:deadsnakes/ppa
```
When prompted, press [Enter] to continue.

Once the repository is enabled, you can install Python 3.9 by executing:
``` console
sudo apt install python3.10
```

Verify that the installation was successful by typing:
``` console
python3.10 --version
```
Output
```
Python 3.10.12
```

Install pip3
``` console
sudo apt install python3-pip
```

Verify that the installation was successful by typing:
``` console
pip3 --version
```
Output
```
pip 24.0 from /home/<username>/.local/lib/python3.10/site-packages/pip (python 3.10)
```

## Step 2 : Install Java

Run the following command. After installation, we can check it by running
``` console
java -version
```
To run the PySpark application, you would need Java 8/11/17 or a later version.
``` console
sudo apt install openjdk-8-jre-headless -y
```

## Step 3 : Install Apache Spark
``` console
pip3 install pyspark==3.5
```

## Step 4 : Install the delta-spark package
``` console
pip3 install delta-spark==3.2
```

## Step 5 : Set up project with Python
``` Python
import pyspark

from pyspark.sql import SparkSession

from delta import *

from delta.tables import *

builder = pyspark.sql.SparkSession.builder.appName("MyApp").enableHiveSupport() \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")

spark = configure_spark_with_delta_pip(builder).getOrCreate()

spark
```
## Step 6: Create a database *default*
``` Python
spark.sql("""
CREATE DATABASE IF NOT EXISTS default
LOCATION '/mnt/silver/default/'
""")
```

## Step 7: Create a *customer* Delta Lake table in the *default* database
``` Python
spark.sql("""
CREATE TABLE  default.customer
(
customer_id INT,
last_name STRING,
first_name STRING
)
USING delta
LOCATION '/mnt/silver/default/customer/';
""")
```

## Step 8: Add some data to the table
``` Python
spark.sql("""
INSERT INTO default.customer (customer_id, last_name, first_name) 
VALUES (1, 'Wilson', 'John'),
       (2, 'Smith', 'Mary'),
       (3, 'Jones', 'David'),
       (4, 'John', 'David');
""")
```

## Step 9: Query the table
``` Python
spark.sql("""
SELECT * FROM default.customer;
""").show()
```
Result
```
+-----------+---------+----------+
|customer_id|last_name|first_name|
+-----------+---------+----------+
|          1|   Wilson|      John|
|          3|    Jones|     David|
|          4|     John|     David|
|          2|    Smith|      Mary|
+-----------+---------+----------+
```

## Step 10: Get description of the table
``` Python
spark.sql("""
DESCRIBE TABLE EXTENDED default.customer;
""").show(50, False)
```
Result
```
+----------------------------+-------------------------------------------------------+-------+
|col_name                    |data_type                                              |comment|
+----------------------------+-------------------------------------------------------+-------+
|customer_id                 |int                                                    |NULL   |
|last_name                   |string                                                 |NULL   |
|first_name                  |string                                                 |NULL   |
|                            |                                                       |       |
|# Detailed Table Information|                                                       |       |
|Name                        |spark_catalog.default.customer                         |       |
|Type                        |EXTERNAL                                               |       |
|Location                    |file:/mnt/silver/default/customer                      |       |
|Provider                    |delta                                                  |       |
|Owner                       |<username>                                             |       |
|Table Properties            |[delta.minReaderVersion=1,delta.minWriterVersion=2]    |       |
+----------------------------+-------------------------------------------------------+-------+
```

