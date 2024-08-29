# Databricks Lakehouse Platform

The Databricks Lakehouse Platform is a commercial-grade, cloud-native implementation of the lakehouse model. It provides a data storage layer on top of the data lake, offering:

- Performance and reliability of a data warehouse
- Flexibility, cost-effectiveness, and scalability of a data lake
- Simple, open, and collaborative platform

![[Pasted image 20240820120947.png]]

## Architecture

### Data Plane

- Where data is processed
- Resources are on the customer's cloud account
- Holds compute resources (clusters)
- Can connect to external data sources
- Provides scalable compute for big data processing

### Control Plane

- Backend services managed by Databricks on its own cloud account
- Aligned with the cloud service used by the customer
- Handles job scheduling, cluster management, and web application

### Unity Catalog

- Provides data governance
- Centralized metadata management
- Fine-grained access control
- Audit logging for compliance

## Key Concepts

### Personas

1. Engineering/Data Science: Build data pipelines, perform advanced analytics
2. Machine Learning: Develop and deploy ML models
3. SQL Analyst: Perform data analysis using SQL queries

### Clusters

A set of computational resources and configurations for running workloads.

![[Pasted image 20240820122755.png]]

#### Types

1. **All-purpose Clusters**

   - For interactive development
   - Can be manually started and terminated
   - Retains configuration information (70 clusters for 30 days)
   - Ideal for data exploration and ad-hoc analysis


2. **Job Clusters**

   - For automated jobs/pipelines
   - Created and terminated by the Databricks scheduler
   - Cannot be restarted manually
   - Retains recent 30 clusters
   - Cost-effective for scheduled workloads


#### Modes

1. **Single Node**

   - One VM instance (driver only)
   - Low cost
   - Suitable for single-node ML workloads and lightweight exploratory analysis
   - Good for learning and small-scale development


2. **Standard (Multi-node)**

   - One VM for driver, at least one VM for worker(s)
   - General-purpose configuration
   - Scalable for large datasets and complex computations

#### Runtime Versions

- Standard: Spark + components for big data analytics
- ML: Includes ML libraries (e.g., TensorFlow, PyTorch, scikit-learn)
- Photon: Optimized for SQL workloads, uses Photon engine for faster query processing

#### Access Modes

- Specifies the overall security model of the cluster
- 2 out of 4 modes useful for Unity Catalog
- Includes options for user isolation and shared access

#### Cluster Policies

![[Pasted image 20240820123800.png]]

- Define and enforce rules for cluster creation
- Control costs and ensure compliance
- Can set limits on instance types, autoscaling, and more


### Databricks Notebooks

- Multi-language support (Python, Scala, R, SQL)
- Collaborative features (real-time co-presence, co-editing)
- Visual exploration capabilities
- Adaptable (can install libraries and local modules)
- Version control for reproducibility
- Easy to schedule as jobs
- Access controls, management, and auditability
- Supports Markdown for documentation

#### Magic Commands

- `%python`: Use Python
- `%scala`: Use Scala
- `%r`: Use R
- `%sql`: Use SQL
- `%sh`: Use shell commands
- `%fs`: Shortcut for dbutils
- `%md`: Markdown
- `%run`: Run other notebooks
- `%pip`: Install libraries


#### dbutils (Databricks Utilities)

- File system manipulation (`dbutils.fs`)
- Secrets management (`dbutils.secrets`)
- Notebook control flow (`dbutils.notebook`)
- Widgets for interactive parameters
- Job features for automation


## Data Objects in a Lakehouse
  

![[Pasted image 20240821161646.png]]

1. Metastore: Central repository for all metadata
2. Catalog: Grouping of databases, highest level container
3. Schema/Database: Grouping of objects, datalogs
4. Table: Structured data storage (managed or external)
5. View: Saved query, can be used like a virtual table
6. Function: Saved logic, can be User-Defined Functions (UDFs)

  
### Key Concepts

- **Lakehouse**: Architecture enabling efficient AI and BI on vast amounts of data stored in Data Lakes
- **Data Lake**: Central location holding large amounts of data in native, raw format using flat architecture and object storage
- **Managed Table**: Based on a file stored in the managed store location (metastore)
  - Databricks manages the lifecycle of the data
  - Dropping the table deletes the data
- **External Table**: File stored outside the managed store location
  - Data persists when table is dropped
  - Useful for accessing data in existing storage systems
- **View**: Saved query
  - Can be used to simplify complex queries
  - Does not store data, only the query definition
- **Function**: Saved logic
  - Can be built-in or user-defined
  - Extends SQL capabilities

## SQL Operations

### SELECT

- Can query JSON: `SELECT * FROM json.'path_to_json'`
- Can query directories: `SELECT * FROM json.'directory'`
- Works with CSV as well
- Supports complex operations like window functions and subqueries

  
### CREATE TABLE

```sql
CREATE TABLE IF NOT EXISTS table_name
(
  column1 datatype1,
  column2 datatype2,
  ...
)
USING CSV
OPTIONS (
  header = "true",
  delimiter = ","
)
LOCATION 'path/to/csv/files'
```


### DESCRIBE

- `DESCRIBE`: Shows basic metadata (column names, data types)
- `DESCRIBE EXTENDED`: Shows more detailed metadata including storage information and statistics


### VIEWS

- Temporary views are only visible in the current Spark session
- Permanent views are stored in the metastore and available across sessions

```sql
CREATE OR REPLACE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition
```

  
### CTEs (Common Table Expressions)

- Must be run in the same cell to work
- Useful for breaking down complex queries into manageable parts

```sql
WITH cte_name AS (
  SELECT column1, column2
  FROM table_name
  WHERE condition
)
SELECT * FROM cte_name
```


### Binary File Metadata

`SELECT * FROM binaryFile.'file/directory'` provides metadata like:
- Last update date
- Content
- Length
- Path
- File name

### Delta Lake

- Open-source storage layer that brings ACID transactions to Apache Spark and big data workloads
- Provides snapshots, rollbacks, and schema enforcement
- Create a Delta table:

```sql
CREATE TABLE events (
  id BIGINT,
  date DATE,
  country STRING,
  confirmed BIGINT,
  deaths BIGINT,
  recovered BIGINT,
  active BIGINT
)
USING DELTA
LOCATION '/mnt/delta/events'
```

### Performance Optimization

- Use appropriate file formats (Parquet, ORC) for better query performance
- Partition large tables based on frequently filtered columns
- Use caching for frequently accessed data
- Utilize Databricks Runtime optimizations like Photon engine

### Caching

- Spark automatically caches underlying data for optimal performance in subsequent queries.
- Manually refresh cache:
  ```sql
  REFRESH TABLE sales_csv
  ```
  - Useful for time testing as it invalidates the cache.

### Reading from JDBC Tables

```python
import pyspark.sql.functions as F

location = spark.sql("DESCRIBE EXTENDED ...").filter(F.col("col_name") == "Location").first()["data_type"]

files = dbutils.fs.location(location)
```

### Custom Drivers in SQL Systems

Some SQL systems have custom drivers, which can affect how Spark interacts with the data:
1. Copy all data into Databricks, then execute the query.
2. Execute the query, then move the data.

## SQL and PySpark Equivalents

### Counting Null Values

SQL:
```sql
SELECT COUNT_IF(blabla IS NULL)
SELECT COUNT() FROM A WHERE blabla IS NULL
```

PySpark:
```python
table = spark.read.table(...)
table.selectExpr("SELECT COUNT_IF(blabla IS NULL)")
# OR
table.where(col("email").isNull()).count()
```

### Distinct Values

PySpark:
```python
table.distinct().count()
```

### Deduplication

SQL (using GROUP BY):
```sql
SELECT MAX(column) FROM table GROUP BY key_column
```

PySpark:
```python
new_Table = (
    table
    .where(col("user_id").isNotNull())
    .groupBy("key_column")
    .agg(max("column").alias("column_alias"))
)
```

### Counting Duplicates

SQL:
```sql
SELECT MAX(row_count) <= 1 AS no_duplicates 
FROM (
    SELECT user_id, COUNT(*) AS row_count 
    FROM table 
    GROUP BY user_id
)
```

> [!NOTE]
> When converting from SQL to PySpark, the order of operations is often reversed.

### Date Formatting

SQL:
```sql
SELECT DATE_FORMAT(column, 'mmddyy')
```

PySpark:
```python
withColumn("new_column", date_format("column", 'mmddyy').cast("timestamp"))
```

## Working with JSON Data

### Querying JSON Columns
```sql
SELECT * FROM table WHERE column:column_2 = 'oh'
```

### Creating Schema from JSON
Use `schema_of_json` to create a schema from a JSON row, then use `from_json` to create a table accordingly.

SQL:
```sql
-- Create a temporary view with sample JSON data
CREATE OR REPLACE TEMPORARY VIEW json_data AS
SELECT '
[
  {
    "name": "Alice",
    "age": 30,
    "city": "New York",
    "hobbies": ["reading", "hiking"]
  },
  {
    "name": "Bob",
    "age": 25,
    "city": "San Francisco",
    "hobbies": ["gaming", "cooking"]
  }
]' AS json_column;

-- Infer the schema from the JSON data
CREATE OR REPLACE TEMPORARY VIEW inferred_schema AS
SELECT schema_of_json(json_column) AS json_schema
FROM json_data;

-- Show the inferred schema
SELECT * FROM inferred_schema;

-- Parse the JSON data using the inferred schema
CREATE OR REPLACE TEMPORARY VIEW parsed_data AS
SELECT from_json(json_column, (SELECT json_schema FROM inferred_schema)) AS parsed
FROM json_data;

-- Explode the array to create individual rows and select all fields
CREATE OR REPLACE TABLE json_table AS
SELECT inline(parsed).*
FROM parsed_data;

-- Show the resulting table
SELECT * FROM json_table;
```

PySpark:
```python
from pyspark.sql.functions import col, schema_of_json, from_json

# Sample JSON data
json_data = """
[
  {
    "name": "Alice",
    "age": 30,
    "city": "New York",
    "hobbies": ["reading", "hiking"]
  },
  {
    "name": "Bob",
    "age": 25,
    "city": "San Francisco",
    "hobbies": ["gaming", "cooking"]
  }
]
"""

# Create a DataFrame with a single column containing the JSON string
df = spark.createDataFrame([(json_data,)], ["json_column"])

# Infer the schema from the JSON data
json_schema = schema_of_json(df.select("json_column").first()[0])

# Print the inferred schema
print("Inferred Schema:")
print(json_schema)

# Parse the JSON data using the inferred schema
parsed_df = df.withColumn("parsed_data", from_json(col("json_column"), json_schema))

# Explode the array to create individual rows
exploded_df = parsed_df.select("parsed_data.*").selectExpr("inline(parsed_data)")

# Show the resulting table
print("\nResulting Table:")
exploded_df.show(truncate=False)
```


### JSON Array Operations

- Check JSON array size:
  ```sql
  SELECT * FROM table WHERE SIZE(column) > 1
  ```
- `collect_set`: Collects unique values for a field
- `flatten`: Combines arrays into a single array
- `array_distinct`: Removes distinct values

![[Pasted image 20240821170521.png]]

![[Pasted image 20240821170509.png]]

> [!TIP]
> Use `column.whatever` to navigate down the array tree.

## Pivot Tables

A **pivot table** is a table of aggregated values from a more extensive dataset (e.g., database, spreadsheet, or business intelligence program) within one or more discrete categories.

Example of pivot table transformation:

![[Pasted image 20240821171333.png]]

Goes to:

![[Pasted image 20240821171341.png]]

For more information, see [Pivot table - Wikipedia](https://en.wikipedia.org/wiki/Pivot_table).

## User-Defined Functions (UDFs)

### SQL UDFs in Spark

SQL UDFs allow you to register custom SQL logic as functions in the database, making these methods reusable anywhere SQL can be run on Databricks. They are registered locally and take advantage of Spark optimizations when executed.

Example:
```sql
CREATE OR REPLACE FUNCTION sales_announcement(item_name, item_price)
RETURNS STRING
RETURN CONCAT('The ', item_name, ' is on sale for ', item_price);

SELECT *, sales_announcement(name, price) AS message FROM item_lookup
```

- Use `DESCRIBE FUNCTION` to view the SQL logic in the function body.
- Useful for complex `CASE WHEN` statements.

### Python UDFs

Python UDFs are custom column transformation functions with some limitations:
- Cannot be optimized by Spark
- Function is serialized and sent to executors
- Requires serialization and deserialization to use the UDF (performance impact)
- Extra communication between the driver and executors

> [!WARNING]
> Python UDFs can have significant performance overhead due to serialization/deserialization and additional communication.

# Delta Lake

## Overview

Delta Lake is an open-source project that enables building a lakehouse on top of existing cloud storage.

```
Delta Lake is an open-source project that enables building a lakehouse on top of an existing cloud storage
```

### What Delta Lake is NOT:
- A storage format
- A database service or data warehouse

### What Delta Lake IS:
- Built upon standard data formats (mainly Parquet)
- Optimized for cloud object storage (though it runs on various mediums)
- Built for scalable metadata handling (quickly return point data)

## ACID Guarantees

Delta Lake guarantees ACID properties:

1. **Atomicity**: All transactions succeed or fail completely.
2. **Consistency**: Guarantees the state of the data between simultaneous operations.
3. **Isolation**: Manages how simultaneous operations conflict with one another.
4. **Durability**: Ensures that committed changes are permanent.

### ACID Solves:
- Difficulty in appending data
- Modification of existing data
- Job failures midway (changes are not committed until success)
- Real-time operations thanks to atomic micro-operations
- Historical data preservation through logs and snapshot queries, allowing time travel

> **Note**: Delta is the standard format in Databricks.

## Schema and Table Management

### Creating a Schema
- Custom location vs. non-custom location
  - Non-custom goes to DBFS shared warehouse

```sql
DESCRIBE SCHEMA [EXTENDED]
```
This command also works and will show information about the tables in that schema.

### Creating Tables

After creating a schema, you can create tables within it.

- If a table is created as external on a schema and the table is dropped, the data will remain in the schema.
- `CASCADE` - deletes from parent to child.

#### Table Creation Methods:
1. `CREATE TABLE AS` (CTAs)
   - Limiting because you can't be very specific
   - To counter this, create a temp view with configurations and CTA from the view
2. `CREATE OR REPLACE TABLE AS SELECT *`
   - Creates if not exists, overwrites if it exists

### Generated Columns

Generated columns are a special type of column whose values are automatically generated based on the values of other columns.

### MERGE Operations

MERGE can combine two tables with configurations for update, insert, or delete operations.

![[Pasted image 20240828113847.png]]
![[Pasted image 20240828113855.png]]

> **Note**: Can have problems if a column is generated and the values do not match.

```
There is no need to use REFRESH TABLE on Delta tables!
```

### Table Constraints

You can add constraints for columns using `ADD CONSTRAINT`.

### Cloning Tables

- **DEEP CLONE**: Fully copies data and metadata.
- **SHALLOW CLONE**: Only copies Delta transaction logs.

## Delta Transaction Log

- An ordered record of every transaction performed on a Delta Lake table since its inception.
- Stores a single source of truth, a repository with all changes users make to the table.
- When a user performs a SELECT on a table, the Spark engine checks the Delta log repo and updates the table to the last version.

### Handling Concurrent Reads and Writes

Delta Lake uses **optimistic concurrency control**:
- Assumes transactions from different users can often complete without conflict.
- Has a protocol for resolving conflicts when they occur.

![[Pasted image 20240828115734.png]]

1. Delta starts with version 0.
2. Users 1 and 2 attempt to append data, creating a conflict.
3. Mutual exclusion is applied, allowing only User 1 to write.
4. If version 001 and 002 don't differ, User 2 automatically applies 002.
5. If they differ, User 2's reading version is silently updated to 001, then commits changes.

```
Note: Errors can occur when conflicts can't be resolved optimistically, e.g., if both users delete the same file.
```

## Table Operations

### Overwrite vs. DELETE + Create
- Faster
- Preserves old versions for time travel
- Atomic operation
- If it fails, the table is not updated

### Useful Commands

- `DESCRIBE HISTORY`: View Delta table history
- `COPY INTO`: Efficient for adding small files multiple times to a table
- `OPTIMIZE`: Compact small files
- `ZORDER`: Index by a specific column
- `VACUUM`: Clean files from older versions of a Delta table (use `RETAIN X HOURS`)
- `RESTORE TABLE`: Restore the table to a previous version

## Additional Resources

For more in-depth information on Delta Lake and its features, consider exploring:
1. [Delta Lake Official Documentation](https://docs.delta.io/latest/index.html)
2. [Databricks Delta Lake Guide](https://docs.databricks.com/delta/index.html)
3. [Delta Lake GitHub Repository](https://github.com/delta-io/delta)

