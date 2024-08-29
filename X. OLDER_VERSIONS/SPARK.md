[1] [Diagnose cost and performance issues using the Spark UI | Databricks on AWS](https://docs.databricks.com/en/optimizations/spark-ui-guide/index.html)


"*Data is expensive to move so Spark focuses on performing computations over the data, no matter where it resides.*" - This way we can achieve data locality, we are performing operations over the data without moving it to a central place.

*Apache Hadoop* is a combination of a filesystem *HDFS* and a computing engine *MAPREDUCE*, this makes it harder for one to work without the other. However, Spark does run well with HDFS.

A single computer might not be powerful enough to do some computation. A cluster/group of computers might get the job done as they utilize the resources of the multiple machines and use it as a single computer. Now, a cluster/group of computers is not enough, we need a framework that is able to coordinate the work amongst them, _and that's what Spark does_. 

The cluster of machines that Spark will use to execute some task is managed by a _cluster manager_, in which a _Spark Application_ will be submitted.

What is a _Spark Application_ (Spark Session)?
	A _Spark Application_ consists of a _driver process_ and a set of executors. 

What does the _Driver Process_ do?
	The _Driver Process_ is responsible to:
		1. run the __main()__ function.
		2. Maintain information about the Application.
		3. Respond to user input.
		4. Analyze, distribute and schedule work across the executors.

```
				The Driver Process is the heart of Spark!
```
	
What do the _Executors_ do?
	The _executors_ are responsible for actually carrying out the operations that the driver assigns them. So, it makes the responsible of two things:
		1. Running code assigned by the driver.
		2. Report back.

![[Pasted image 20240807161159.png]]

To allow every _executor_ to do the work in parallel, Spark breaks the data into chunks called "partitions". 

What are _partitions_?
	A _partition_ is a collection of collection of data (namely rows of a DataFrame) that sits *PHYSICALLY*  on a machine of our cluster/group of computers.
	So, a DataFrame partition means how the data is physically distributed across the cluster of machines during the execution.

What does this mean?
	-> If you only have 1 partition, the process will have parallelism of one.
	-> If you have multiple partitions but only 1 executor, the process will have a parallelism of one as well. 

---
##### Transformations

There are 2 types of transformations:
- __Narrow Transformations:__ Transformations consisting of narrow dependencies are those for which each partition will contribute to __ONLY__ one output partition (example: Divide one column by another). When doing this operations, the operation will be done in memory (which is good!) because there is no need to exchange partitions across the cluster.

	 ![[Pasted image 20240807162454.png]]
- __Wide Transformations__: Transformations consisting of wide dependencies will have input partitions contributing to multiple output partitions. This is what is referred as a _SHUFFLE_, which means that Spark is exchanging partitions across the cluster. (Example: Joins, Aggregations, etc...) Since there is a need to exchange partitions across the cluster the result will be written on disk.

		![[Pasted image 20240807162841.png]]

##### Lazy Evaluation

_Lazy Evaluation_ means that Spark will wait until the very last moment to execute a specific operation. __Why?__ This way Spark can compile the best plan and ran it across the cluster. (Example: Filters with pushdown.)

##### Logical Plan

The logical plan only represents a set of abstract transformations that do not refer to executors or
drivers, it’s purely to convert the user’s set of expressions into the most optimized version. It
does this by converting user code into an unresolved logical plan. This plan is unresolved
because although your code might be valid, the tables or columns that it refers to might or might
not exist. (Example: Pushdowns)

![[Pasted image 20240807170806.png]]
##### Physical Plan

The _physical plan_,, specifies how the logical plan will execute on the cluster by generating different physical execution strategies and comparing them through a cost model.

![[Pasted image 20240807170636.png]]

---

#### Joins

##### Shuffle Join

In a shuffle join every node "talks" to every other node and they share data according to which node has a certain key or set of keys.

##### Broadcast Join

It will replicate our small DataFrame onto every worker node. What this does is that instead of the nodes talking to each others they will just execute the join operation on themselves. This will make the CPU the biggest bottleneck. Becoming a narrow transformation.

---


