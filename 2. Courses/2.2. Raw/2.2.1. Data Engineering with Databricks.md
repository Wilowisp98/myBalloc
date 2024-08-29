
The Databricks Lakehouse Plataform is a comercial grade cloud native implementation of the lakehouse model.

A data storage layer that resides on top of the datalake that provides the performance and reliability of a data warehoues, combined with the flex/cost/scale of a datalake all in one simple open and colaborative plataform.

![[Pasted image 20240820120947.png]]

Data Plane.
- Is where the data is processed, the resources are on my cloud account. The data plane holds compute resources, which are called clusters. They are connect to the datastore but can also be connected to external data sources, doesnt need to be on the same account.
COntrol plane.
- Consists of backend services that databricks manages on its own cloud account aligned with the cloud service used by the costumer.

Unity Catalog.
- Data Governance.

3 Personas:
- Engineering/DS.
- ML.
- SQL Analyst.

Cluster is a set of computational resources and configurations on which I can run engineering/science or analytics workloads. Can run them as a set of commands in a notebook or on a job.


dbc - databricks compression file.


Cluster consists of a set of one or more VM instances in which computation workload is distributed. Likelike Spark, with a driver and workers.
![[Pasted image 20240820122755.png]]

Two types of clusters: 
- All-purpose: Intended for interactive development. Can manually and terminate an all purpose cluster. Keeps more configuration information (70clusters 30days)
- Job clusters: For running automated jobs/pipelines. The db scheduler creates the job cluster when I run a job and terminates it at the end, cant be restarted.  (recent 30clusters)

Cluster mode:
- Single Node: Only one VM instance, hosting the driver and there are no drivers. It's low cost, good for single node machine learning workloads that use spark to load and save data and lightweight exploratory analysis.
- Standard (Multi-node): General purpose config of a VM instance hosting the driver and at least one VM instance for a worker.

DB Runtime Version.
- Standard - Spark + other components and updates for big data analytics.
- ML - ML libraries.
- Photon - Optimize SQL workloads.

Access mode:
- Specifies the overall security model of the cluster. (2 out of 4 useful for Unity Catalog)

Cluster Policies:
- Policies that can be made on a cluster.
![[Pasted image 20240820123800.png]]


LTS - long time support

DB Notebooks:
- Multi-language: python, scala, r
- collaborative: real time co presence, co editing etc...
- Good for exploration, can have visuals.
- Adaptable, can install libraries and local modules.
- Reproducibale bc they keep versions.
- Easy to schedule notebooks as jobs.
- Access controls, management and auditability.


%python -> use pyhton
%sh -> use sh
%fs -> shortcute for dbutils
%md -> markdown
%run -> run other noteboks
%pip -> install libraries

dbutils (databricks utilities)
- manipulate filesystem (dbutils.fs)
- secrets within notebooks (dbutils.secrets)
- utilities for the control flow of the notebook (dbutils.notebook)
- widgets
- jobs to leverage job features


Data objects in a lakehouse:
- Metastore.
- Catalog. - grouping of DBs
- Schema. - DATABASE, grouping of objects, datalogs. can contain:
- Table. (can be managed or external)
- View
- Function
![[Pasted image 20240821161646.png]]

what is a lakehouse?\
- Data Lakehouse is an architecture that enables efficient and secure Artificial Intelligence (AI) and Business Intelligence (BI) directly on vast amounts of data stored in Data Lakes.

what is a datalake?
- A data lake is a central location that holds a large amount of data in its native, raw format. Compared to a hierarchical data warehouse, which stores data in files or folders, a data lake uses a flat architecture and object storage to store the data.‍ Object storage stores data with metadata tags and a unique identifier, which makes it easier to locate and retrieve data across regions, and improves performance.

what is a managed table?
- a managed table is based on a file which is stored on the managed store location which is on the metastore.
- external table file is stored outside of the managed store location. (table is dropped but data is saved)

what is a view?
- is a saved query.

function
- saved logic.

SELECT:
- you can do SELECT * FROM json.*path to json*
- you can do SELECT * FROM json.*directory*
- can also do csv.

CREATE TABLE IF NOT EXISTS ...
(columns)
USING CSV
OPTIONS (
header = ?
delimiteR = ?
)
LOCATION ...

DESCRIBE/DESCRIBE EXTENDED shows more metadata.

VIEWS:
- TEMP VW are only seen on the current spark session.

CTEs:
- dont work if not run on the same cell.

SELECT * FROM binaryFile.*file/directory* gives METADATA, LUD, content, length, path, etc.


When a query is ran on spark, it will cache the underlying data to assure that subsequent queries will be optimal in terms of perfomance. However, we can manually refresh the cache of our table using:
- REFRESH TABLE sales_csv (it will invalidate the cache, its good when time testing)



here we are from a jdbc table, finding where its saved and reading from that location instead.

import pyspark.sql.functions as F

location = spark.sql("DESCRIBE EXTENDED ...").filter(F.col("col_name") == "Location").first()["data_type"]

files = dbutils.fs.location(location)


Some SQL systems ahave custom drivers, what does that mean? When spark runs over it it will have one of the two approaches:
- it will copy all the data into db and then execute the query.
- it will execute the query and then move the data.




SELECT COUNT_IF(blabla is null) == SELECT COUNT() FROM A WHERE blabla is null
which is equal to

table = spark.read.table(...)
table.selectExpr("SELECT COUNT_IF(blabla is null)")
OR
table.where(col(email).isNull()).count()


distinct:
- table.distinct().count()


how to deduplicate?
- group by (max for example)
vs
new_Table = (
table
.where(col("user_id").isNotNull())
.groupby("blablabla")
.agg(max(blablabla).alias(ablablabla))
)



count duplicates?

select max(row_count) <= 1 no_duplicates? from (
select user_id, count() from able group by user_id
)


```
basicamente, de SQL para spark troca a ordem
```


select date_format(blablabla, "mmddyy)
<=>
withColumn("blablabla_NEW"), date_format("blablabla, 'mmddyy').cast(timestamp)



if we have a column looking like this:
{
"column1": 'hey'
"column2": 'oh'
"column3": 'lets'
"column4": 'go'
}

i can select it like this: select ** from table where column:column_2 = 'oh' 

i can also create a schema using a row of a json, (schema_of_json) like on spark, and use that schema to create a table accordingly (from_json)

using select ** from table where size(column) > 1, will return the rows where the json has more that 1 value on that column
![[Pasted image 20240821170521.png]]

![[Pasted image 20240821170509.png]]

collect_set = colects unique values for a field.
flatten = combines arrays into a single array
array_distinct = removes distincts

using column.whatever will go down on the array tree.

pivot tables:
A **pivot table** is a [table](https://en.wikipedia.org/wiki/Table_(information) "Table (information)") of values which are aggregations of groups of individual values from a more extensive table (such as from a [database](https://en.wikipedia.org/wiki/Database "Database"), [spreadsheet](https://en.wikipedia.org/wiki/Spreadsheet "Spreadsheet"), or [business intelligence program](https://en.wikipedia.org/wiki/Business_intelligence_software "Business intelligence software")) within one or more discrete categories [1] [Pivot table - Wikipedia](https://en.wikipedia.org/wiki/Pivot_table)

![[Pasted image 20240821171333.png]] 

goes to this:

![[Pasted image 20240821171341.png]]

what are SQL UDFS in spark?

allow you to registar custom sql logic as8 functions in database, making these methdos reusable anywhere sql and can be run on databricks. they are registered locally and take all the spark optimizations when executed.

for example:
create or replace function sales_announcement(item_name, item_price)
returns string <- type
return concat('the", item_name, "is on sale for", item price.); <- what it does
 
select ALL, sales_announcement(name, price) as message from item_lookup

you can use describe function, the body has the sql logic.

its nice for case whens.

---

what are Python UDFS?

- custom column transformation function.
- cant be optimized.
- functions is serialized and sent to executors.
- serialize and deserialize to use the udf (WHICH IS VERY BAD)
- extra communitication between the driver and executors.



---

Delta lake 

```
Delta lake is an open source project that enables building a lakehouse on top of an existing cloud storage
```

IS NOT:
- storage format
- database service or data warehouse

IS:
- builds upon standard data formats. (mainly in parquet)
- optimized for cloud object storage. (even tho it runs of various mediums)
- built for scalable matadata handling. (quicky return point data)

delta guarantees ACID:
- atomicity: all transactions succeed or fail completly.
- consistencty: guarantees the state of the data between simultaenous operations.
- isolation: how simultaneous operations conflict with one another.
- durability: means that commited changes are permanent.

ACID solves:
- hard to append data.
- modficication of existing data.
- jobs failing midway means that changes are not commited until it suceeds.
- real time operations thanks to atomic micro operations.
- keep historical through logs and snapchat queries, which allows timetravel

DELTA Is the standard format in databricks.

creating a schema with a custom locaiton vs not a custom location:
-> non custom goes to dbfs shared warehouse.

DESCRIBE SHCEMA [EXTENDED] also works, will also show the information of the tables on that schema.

Then, having the schema created, we can create a table on that schema.

if a table is created as external on a schema and the table is dropped the data will remain on the schema

*CASCADE* - deletes from father to child.

CREATE TABLE AS - CTAs (is limitating cause you cant be very specific)
TO counter that, we cna create a temp view with configurations and CTA from the view and it will gather the configurations.

CREATE OR REPLACE TABLE AS SELECT * - create if not exists and if it exists, it overwrites the data.

GENERATED COLUMNS ARE A SPECIAL TYPE OF COLUMN WHOSE VALUES ARE AUTOMATICALLY GENERATED BASED ON THE VALUES OF OTHER COLUMNS

MERGE - can merge 2 tables with configurations as update, insert or delete.

![[Pasted image 20240828113847.png]]![[Pasted image 20240828113855.png]]

Can have problems if a column is generated and the values do not match.

```
There is no need to use refresh tables on delta tables!! 
```


Can add constraints for the columns using ADD CONTRAINT.

DEEP CLONE -> Fully copies data and metadata.

SHALLOW CLONE -> Only copies delta transaction logs.

What is a delta transaction log?
- Is an ordered record of every transaction that has ever been performed on a delta lake table since its inception.
- In order to allow multiple read/writes of the same information deta tables store a single source of truth, a repository with all the changes that users make to the table.
- When a user does a select on a table, the spark engine will check the delta log repo and update the table to the last version.

How does it deal with multiple concurrent reads and writes?
- Since it usess Spark, its expected to be submited to multiple reads/writes at the same time and because of that Delta Lake assumes *optimistic concurrency control*. What does that mean? It means that transactions made by different users to the same table can probably be completed without confliction without one or another. (Building a puzzle, if I build the center and someone else the corners its totally fine)
- However, sometimes they can interfer. For that Delta as a protocol to determine what to do when two or more commits are made at the same time. The engine tries to apply mutual exclusion aand then tries to solve it optimistically. 
![[Pasted image 20240828115734.png]]

1. Delta has the version 0.
2. Users 1 and 2 both attempt to append data from that version, which is a conflict since both want to write on 001.
3. It applies mutual exclusion, and only user 1 is allwoed to write.
4. Instead of sending an error, it checks if 001 differs from 002, if it dont user 2 automatically applies 002, if not, the version that user 2 is reading is updated to 001 *SILENTLY* and then user 2 simply commits his changes.
``` 
SOmetimes it can throw errors when it can not be solved optimistically. If User 1 deletes a files that User 2 also deleted.

```


Overwrite vs DELETE + Create
- Its faster.
- The old version still exists, so you can time travel.
- It's an atomic operation.
- It it fails, the table is not updated.

DESCRIBE HISTORY - delta table history.

COPY INTO - Is nice when we are adding small files multiple times to a table.

OPTIMIZE - Compact small files.

ZORDER - Indexes by a column that we want.

VACUUM - Clean files from older versions of a delta table. (RETAIN X HOURS)

RESTORE TABLE - Allows to restore the version of the table to a previous version.

