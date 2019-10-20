---
title: SparkRDD学习
date: 2019-10-09 17:30:01
tags:
---

RDD 操作

官方文档:

http://spark.apache.org/docs/latest/rdd-programming-guide.html

> RDD 操作分为 转换(Transformation)和行动(Action)

Transformations

| sformation                                                   | Meaning                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| **map**(*func*)                                              | Return a new distributed dataset formed by passing each element of the source through a function *func*. 将每个元素用`func`函数处理后返回一个新的RDD |
| **filter**(*func*)                                           | Return a new dataset formed by selecting those elements of the source on which *func* returns true. 将每个元素用func处理,将返回结果是true的结果组合成一个新的RDD返回 |
| **flatMap**(*func*)                                          | Similar to map, but each input item can be mapped to 0 or more output items (so *func* should return a Seq rather than a single item).  和map函数类似,每个元素可以输出1个以上的元素(所以func应该返回一个Seq,而不是一个值) |
| **mapPartitions**(*func*)                                    | Similar to map, but runs separately on each partition (block) of the RDD, so *func* must be of type     Iterator<T> => Iterator<U> when running on an RDD of type T.  和map函数类是,func接收的参数是RDD的每个分区,在类型是T的RDD上运行时,func的输入输出参数要是 Iterator<T>=>Iterator<T> |
| **mapPartitionsWithIndex**(*func*)                           | Similar to mapPartitions, but also provides *func* with an integer value representing the index of   the partition, so *func* must be of type (Int, Iterator<T>) => Iterator<U> when running on an RDD of type T.    和mapPartitions类似,func函数的参数多一个整数表示分区的索引,参数类型是 (Int,Iterator)=>(Iterator) |
| **sample**(*withReplacement*, *fraction*, *seed*)            | Sample a fraction *fraction* of the data, with or without replacement, using a given random number generator seed. |
| **union**(*otherDataset*)                                    | Return a new dataset that contains the union of the elements in the source dataset and the argument. 原RDD和参数的RDD联合 |
| **intersection**(*otherDataset*)                             | Return a new RDD that contains the intersection of elements in the source dataset and the argument. |
| **distinct**([*numPartitions*]))                             | Return a new dataset that contains the distinct elements of the source dataset. 返回一个没有重复元素的RDD |
| **groupByKey**([*numPartitions*])                            | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs.       **Note:** If you are grouping in order to perform an aggregation (such as a sum or       average) over each key, using `reduceByKey` or `aggregateByKey` will yield much better       performance.           **Note:** By default, the level of parallelism in the output depends on the number of partitions of the parent RDD.       You can pass an optional `numPartitions` argument to set a different number of tasks.   在(K,V)类型的键值对上调用,返回一个(K,Iterable<V>)的键值对 Note:如果为了在每个key上聚合(如求和,平均),使用`reduceByKey`或`aggregateBykey`,性能会更好 |
| **reduceByKey**(*func*, [*numPartitions*])                   | When called on a dataset of (K, V) pairs, returns a dataset of  (K, V) pairs where the values for each key are aggregated using the  given reduce function *func*, which must be of type (V,V) => V. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. 在(K,V)的键值对RDD上使用 ,返回一个RDD ,key的每个value用给定的reduce函数func聚合,聚合函数func的类型要是(V,V)=>V,向`groupByKey`一样,reduce任务通过可选的第二参数设置 |
| **aggregateByKey**(*zeroValue*)(*seqOp*, *combOp*, [*numPartitions*]) | When called on a dataset of (K, V) pairs, returns a dataset of  (K, U) pairs where the values for each key are aggregated using the  given combine functions and a neutral "zero" value. Allows an aggregated  value type that is different than the input value type, while avoiding  unnecessary allocations. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. 在(K,V)键值对的RDD上使用,返回(K,U)键值对,每个键的值使用指定的combine函数和一个值聚合.聚合的值的类型可以和输入的值类型不一样,和`groupByKey`函数一样,reduce task的数量通过第二个可选参数指定 |
| **sortByKey**([*ascending*], [*numPartitions*])              | When called on a dataset of (K, V) pairs where K implements  Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending  or descending order, as specified in the boolean `ascending` argument.在实现了Ordered的 (K,V)RDD上使用 |
| **join**(*otherDataset*, [*numPartitions*])                  | When called on datasets of type (K, V) and (K, W), returns a  dataset of (K, (V, W)) pairs with all pairs of elements for each key.     Outer joins are supported through `leftOuterJoin`, `rightOuterJoin`, and `fullOuterJoin`.   在(K,V)类型的RDD上使用,将键一样的放在一起 (K,V)+(K,W)=(K,(V,W) |
| **cogroup**(*otherDataset*, [*numPartitions*])               | When called on datasets of type (K, V) and (K, W), returns a  dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This  operation is also called `groupWith`. 在 (K,V)和(K,W)类型的RDD上调用,返回一个(K,(Iterable<V>,Iterable<W>))元祖的 |
| **cartesian**(*otherDataset*)                                | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements). |
| **pipe**(*command*, *[envVars]*)                             | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the     process's stdin and lines output to its stdout are returned as an RDD of strings. |
| **coalesce**(*numPartitions*)                                | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently     after filtering down a large dataset. |
| **repartition**(*numPartitions*)                             | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them.     This always shuffles all data over the network. |
| **repartitionAndSortWithinPartitions**(*partitioner*)        | Repartition the RDD according to the given partitioner and, within each resulting partition,   sort records by their keys. This is more efficient than calling `repartition` and then sorting within   each partition because it can push the sorting down into the shuffle machinery. |

<!-- more -->

Actions

| Action                                             | Meaning                                                      |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **reduce**(*func*)                                 | Aggregate the elements of the dataset using a function *func*  (which takes two arguments and returns one). The function should be  commutative and associative so that it can be computed correctly in  parallel. |
| **collect**()                                      | Return all the elements of the dataset as an array at the driver  program. This is usually useful after a filter or other operation that  returns a sufficiently small subset of the data. |
| **count**()                                        | Return the number of elements in the dataset.                |
| **first**()                                        | Return the first element of the dataset (similar to take(1)). |
| **take**(*n*)                                      | Return an array with the first *n* elements of the dataset.  |
| **takeSample**(*withReplacement*, *num*, [*seed*]) | Return an array with a random sample of *num* elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed. |
| **takeOrdered**(*n*, *[ordering]*)                 | Return the first *n* elements of the RDD using either their natural order or a custom comparator. |
| **saveAsTextFile**(*path*)                         | Write the elements of the dataset as a text file (or set of text  files) in a given directory in the local filesystem, HDFS or any other  Hadoop-supported file system. Spark will call toString on each element  to convert it to a line of text in the file. |
| **saveAsSequenceFile**(*path*)   (Java and Scala)  | Write the elements of the dataset as a Hadoop SequenceFile in a  given path in the local filesystem, HDFS or any other Hadoop-supported  file system. This is available on RDDs of key-value pairs that implement  Hadoop's Writable interface. In Scala, it is also    available on types that are implicitly convertible to Writable (Spark  includes conversions for basic types like Int, Double, String, etc). |
| **saveAsObjectFile**(*path*)   (Java and Scala)    | Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using     `SparkContext.objectFile()`. |
| **countByKey**()                                   | Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. |
| **foreach**(*func*)                                | Run a function *func* on each element of the dataset. This is usually done for side effects such as updating an [Accumulator](http://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators) or interacting with external storage systems.    **Note**: modifying variables other than Accumulators outside of the `foreach()` may result in undefined behavior. See [Understanding closures ](http://spark.apache.org/docs/latest/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka) for more details. |