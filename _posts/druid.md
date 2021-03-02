

------

# 简介

https://druid.apache.org/

Apache Druid is a real-time analytics database designed for fast slice-and-dice analytics ("[OLAP](https://zh.wikipedia.org/wiki/線上分析處理)" queries) on large data sets. Druid is most often used as a database for powering use cases where real-time ingest, fast query performance, and high uptime are important. As such, Druid is commonly used for powering GUIs of analytical applications, or as a backend for highly-concurrent APIs that need fast aggregations. Druid works best with event-oriented data 面向事件的数据.





Druid的常见应用领域包括：

点击流分析（网络和移动分析）

网络遥测分析（网络性能监测）

服务器度量存储

供应链分析（制造指标）

应用程序性能度量

数字营销/广告分析

商业智能/OLAP



1. Columnar storage format. （列存储）Druid uses column-oriented storage, meaning it only needs to load the exact columns needed for a particular query. This gives a huge speed boost to queries that only hit a few columns. In addition, each column is stored optimized for its particular data type, which supports fast scans and aggregations.
2. Scalable distributed system（可扩展的分布式系统）. Druid is typically deployed in clusters of tens to hundreds of servers, and can offer ingest rates of millions of records/sec, retention of trillions of records, and query latencies of sub-second to a few seconds.
3. Massively parallel processing（大规模并行处理）. Druid can process a query in parallel across the entire cluster.
4. Realtime or batch ingestion（实时或批量摄取）. Druid can ingest data either real-time (ingested data is immediately available for querying) or in batches.
5. Self-healing, self-balancing, easy to operate（自我修复，自我平衡，操作简单）. As an operator, to scale the cluster out or in, simply add or remove servers and the cluster will rebalance itself automatically, in the background, without any downtime. If any Druid servers fail, the system will automatically route around the damage until those servers can be replaced. Druid is designed to run 24/7 with no need for planned downtimes for any reason, including configuration changes and software updates.
6. Cloud-native, fault-tolerant architecture that won't lose data（不会丢失数据的云本机容错体系结构）. Once Druid has ingested your data, a copy is stored safely in [deep storage](https://druid.apache.org/docs/latest/design/architecture.html#deep-storage) (typically cloud storage, HDFS, or a shared filesystem). Your data can be recovered from deep storage even if every single Druid server fails. For more limited failures affecting just a few Druid servers, replication ensures that queries are still possible while the system recovers.
7. Indexes for quick filtering（用于快速筛选的索引）. Druid uses [CONCISE](https://arxiv.org/pdf/1004.0403) or [Roaring](https://roaringbitmap.org/) compressed bitmap indexes to create indexes that power fast filtering and searching across multiple columns.
8. Time-based partitioning（基于时间的分区）. Druid first partitions data by time, and can additionally partition based on other fields. This means time-based queries will only access the partitions that match the time range of the query. This leads to significant performance improvements for time-based data.
9. Approximate algorithms（近似算法）. Druid includes algorithms for approximate count-distinct, approximate ranking, and computation of approximate histograms and quantiles. These algorithms offer bounded memory usage and are often substantially faster than exact computations. For situations where accuracy is more important than speed, Druid also offers exact count-distinct and exact ranking.
10. Automatic summarization at ingest time（摄取时自动聚合）. Druid optionally supports data summarization at ingestion time. This summarization partially pre-aggregates your data, and can lead to big costs savings and performance boosts.



使用场景：

- 插入率很高，但更新不太常见。
- 大多数查询都是聚合和报告查询（“group by”查询）。您还可以进行搜索和扫描查询。
- 您的目标查询延迟为100毫秒到几秒。
- 您的数据有一个时间组件（Druid包括与时间相关的优化和设计选择）。
- 您可能有多个表，但每个查询只访问一个大的分布式表。查询可能会命中多个较小的“查找”表。
- 您拥有高基数的数据列（如url、用户id），并且需要快速计数和排名。
- 您希望从Kafka、HDFS、平面文件或类似Amazon S3的对象存储中加载数据。

不想推荐使用：

- 您需要使用主键对现有记录进行低延迟更新。Druid支持流式插入，但不支持流式更新（更新是使用后台批处理作业完成的）。
- 您正在构建一个离线报告系统，其中查询延迟不是非常重要。
- 您希望执行“大”连接（将一个大事实表连接到另一个大事实表），并且您可以接受这些查询需要很长时间才能完成。

# 表结构

列式存储：



### A segment file's core data structures

Here we describe the internal structure of segment files, which is essentially *columnar*: the data for each column is laid out in separate data structures. By storing each column separately, Druid can decrease query latency by scanning only those columns actually needed for a query. There are three basic column types: the timestamp column, dimension columns, and metric columns, as illustrated in the image below:

在这里，我们描述段文件的内部结构，它本质上是柱状的：每列的数据在单独的数据结构中布局。通过分别存储每一列，Druid可以通过只扫描查询实际需要的列来减少查询延迟。有三种基本列类型：时间戳列、维度列和度量列，

![Druid column types](https://druid.apache.org/docs/latest/assets/druid-column-types.png)

The timestamp and metric columns are simple: behind the scenes each of these is an array of integer or floating point values compressed with LZ4. Once a query knows which rows it needs to select, it simply decompresses these, pulls out the relevant rows, and applies the desired aggregation operator. As with all columns, if a query doesn’t require a column, then that column’s data is just skipped over.

timestamp和metric列很简单：在幕后，每个列都是用LZ4压缩的整数或浮点值数组。一旦查询知道需要选择哪些行，它只需解压缩这些行，提取相关行，然后应用所需的聚合运算符。与所有列一样，如果查询不需要列，则只跳过该列的数据。



Dimensions columns are different because they support filter and group-by operations, so each dimension requires the following three data structures:

1. A dictionary that maps values (which are always treated as strings) to integer IDs,
2. A list of the column’s values, encoded using the dictionary in 1, and
3. For each distinct value in the column, a bitmap that indicates which rows contain that value.



维度列不同，因为它们支持筛选和按操作分组，因此每个维度都需要以下三种数据结构：

- 一个将值（通常被当作字符串）映射到整数id的字典，
- 列值的列表，使用1中的字典进行编码，以及
- 对于列中的每个不同值，指示哪些行包含该值的位图。

Why these three data structures? The dictionary simply maps string values to integer ids so that the values in (2) and (3) can be represented compactly. The bitmaps in (3) -- also known as *inverted indexes* allow for quick filtering operations (specifically, bitmaps are convenient for quickly applying AND and OR operators). Finally, the list of values in (2) is needed for *group by* and *TopN* queries. In other words, queries that solely aggregate metrics based on filters do not need to touch the list of dimension values stored in (2).

介绍列存储优点，详见行存储和列存储的部分笔记



# 查询

查询方式：

1.聚合查询：时间序列查询(Timeseroes),Top查询(TopN),GroupBy查询(GroupBy)
2.元数据查询:时间范围(Time Boundary),段元数据(Segment Metadata)，数据源(DataSource)
2.Search查询(Search)



## Timeseries queries

These types of queries take a timeseries query object and return an array of JSON objects where each object represents a value asked for by the timeseries query.



query属性说明



| 属性             | 描述                                                         | 是否必填 |
| ---------------- | ------------------------------------------------------------ | -------- |
| querytype        | 字符串类型，时间序列 "timeseries"                            | 是       |
| dataSource       | 字符串类型，[数据源](http://druid.io/docs/0.9.2/querying/datasource.html) | 是       |
| descending       | 排序方式，默认false                                          | 否       |
| intervals        | 查询时间范围                                                 | 是       |
| granularity      | 聚合粒度，[说明](http://druid.io/docs/0.9.2/querying/granularities.html) | 是       |
| filter           | 过滤条件，[说明](http://druid.io/docs/0.9.2/querying/filters.html) | 否       |
| aggregations     | 聚合，[说明](http://druid.io/docs/0.9.2/querying/aggregations.html) | 是       |
| postAggregations | 后聚合，[说明](http://druid.io/docs/0.9.2/querying/post-aggregations.html) | 否       |
| context          | 上下文，[`说明`](http://druid.io/docs/0.9.2/querying/query-context.html) | 否       |

# TopN queries



Apache Druid TopN queries return a sorted set of results for the values in a given dimension according to some criteria. Conceptually, they can be thought of as an approximate [GroupByQuery](https://druid.apache.org/docs/latest/querying/groupbyquery.html) over a single dimension with an [Ordering](https://druid.apache.org/docs/latest/querying/limitspec.html) spec. TopNs are much faster and resource efficient than GroupBys for this use case. These types of queries take a topN query object and return an array of JSON objects where each object represents a value asked for by the topN query.



Apache Druid TopN查询根据某些条件返回给定维度中值的一组排序结果。从概念上讲，它们可以看作是一个具有排序规范的一维上的近似GroupByQuery。对于这个用例，topn比GroupBys快得多，而且资源效率也高。这些类型的查询接受一个topN查询对象并返回一个JSON对象数组，其中每个对象表示topN查询所要求的值。







topn是近似值，因为每个数据进程将对其前K个结果进行排序，并且只将前K个结果返回给代理。K、 在Druid中，默认为max（1000，阈值）。实际上，这意味着如果您要求订购前1000个项目，则前900个项目的正确率将为100%，并且不保证订购后的结果。增加阈值可以使topn更加精确。



query属性说明



| 属性             | 描述                                                         | 是否必填 |
| ---------------- | ------------------------------------------------------------ | -------- |
| querytype        | 字符串类型，时间序列 "topN"                                  | 是       |
| dataSource       | 字符串类型，数据源，[说明](http://druid.io/docs/0.9.2/querying/datasource.html) | 是       |
| dimension        | groupBy的维度，[说明](http://druid.io/docs/0.9.2/querying/dimensionspecs.html) | 是       |
| intervals        | 查询时间范围                                                 | 是       |
| granularity      | 聚合粒度，[说明](http://druid.io/docs/0.9.2/querying/granularities.html) | 是       |
| filter           | 过滤条件，[说明](http://druid.io/docs/0.9.2/querying/filters.html) | 否       |
| aggregations     | 聚合，[说明](http://druid.io/docs/0.9.2/querying/aggregations.html) | 是       |
| postAggregations | 后聚合，[说明](http://druid.io/docs/0.9.2/querying/post-aggregations.html) | 否       |
| threshold        | topN的N值                                                    | 是       |
| metric           | 字符串或Json对象指定度量对Top N个结果排序，[说明](http://druid.io/docs/0.9.2/querying/topnmetricspec.html) | 否       |
| context          | 上下文，[说明](http://druid.io/docs/0.9.2/querying/query-context.html) | 否       |





The current TopN algorithm is an approximate algorithm. The top 1000 local results from each segment are returned for merging to determine the global topN. As such, the topN algorithm is approximate in both rank and results. Approximate results *ONLY APPLY WHEN THERE ARE MORE THAN 1000 DIM VALUES*. A topN over a dimension with fewer than 1000 unique dimension values can be considered accurate in rank and accurate in aggregates.

The threshold can be modified from it's default 1000 via the server parameter `druid.query.topN.minTopNThreshold` which need to restart servers to take effect or set `minTopNThreshold` in query context which take effect per query.

If you are wanting the top 100 of a high cardinality, uniformly distributed dimension ordered by some low-cardinality, uniformly distributed dimension, you are potentially going to get aggregates back that are missing data.（如果您想要一个高基数、均匀分布维度的前100个维度按某个低基数、均匀分布的维度排序，那么您可能会得到丢失数据的聚合）

To put it another way, the best use cases for topN are when you can have confidence that the overall results are uniformly in the top. For example, if a particular site ID is in the top 10 for some metric for every hour of every day, then it will probably be accurate in the topN over multiple days. But if a site barely in the top 1000 for any given hour, but over the whole query granularity is in the top 500 (example: a site which gets highly uniform traffic co-mingling in the dataset with sites with highly periodic data), then a top500 query may not have that particular site a the exact rank, and may not be accurate for that particular site's aggregates.

（换言之，topN的最佳用例是当您能够确信总体结果一致地位于顶层时。例如，如果某个特定站点的ID在某个指标中每天每小时都在前10位，那么它可能会在多天内精确到topN。但是，如果一个站点在任何给定的小时内几乎不在前1000名之内，但在整个查询粒度上却在前500名（例如：一个站点在数据集中获得高度一致的流量，并且站点具有高度周期性的数据），则top500查询可能没有该特定站点的确切排名，对于那个特定站点的聚合可能并不准确。）

Before continuing in this section, please consider if you really need exact results. Getting exact results is a very resource intensive process. For the vast majority of "useful" data results, an approximate topN algorithm supplies plenty of accuracy.

Users wishing to get an *exact rank and exact aggregates* topN over a dimension with greater than 1000 unique values should issue a groupBy query and sort the results themselves. This is very computationally expensive for high-cardinality dimensions.

（如果用户希望在一个具有大于1000个唯一值维度上获得精确的排名和精确的topN聚合，那么应该发出groupBy查询并自行对结果进行排序。对于高基数维，这在计算上非常昂贵。）

Users who can tolerate *approximate rank* topN over a dimension with greater than 1000 unique values, but require *exact aggregates* can issue two queries. One to get the approximate topN dimension values, and another topN with dimension selection filters which only use the topN results of the first.





# GroupBy queries



These types of Apache Druid queries take a groupBy query object and return an array of JSON objects where each object represents a grouping asked for by the query.



> Note: If you are doing aggregations with time as your only grouping, or an ordered groupBy over a single dimension, consider [Timeseries](https://druid.apache.org/docs/latest/querying/timeseriesquery.html) and [TopN](https://druid.apache.org/docs/latest/querying/topnquery.html) queries as well as groupBy. Their performance may be better in some cases. See [Alternatives](https://druid.apache.org/docs/latest/querying/groupbyquery.html#alternatives) below for more details.



query属性说明



| 属性             | 描述                                                         | 是否必填 |
| ---------------- | ------------------------------------------------------------ | -------- |
| querytype        | 字符串类型，时间序列 "topN"                                  | 是       |
| dataSource       | 字符串类型，数据源，[说明](http://druid.io/docs/0.9.2/querying/datasource.html) | 是       |
| dimensions       | groupBy的维度，[说明](http://druid.io/docs/0.9.2/querying/dimensionspecs.html) | 是       |
| intervals        | 查询时间范围                                                 | 是       |
| granularity      | 聚合粒度，[说明](http://druid.io/docs/0.9.2/querying/granularities.html)简单粒度支持粒度字符串是：`all`，`none`，`second`，`minute`，`fifteen_minute`，`thirty_minute`，`hour`，`day`，`week`，`month`，`quarter`和`year`。持续时间`{"type": "duration", "duration": 7200000}`周期粒度周期粒度以[ISO8601](https://en.wikipedia.org/wiki/ISO_8601)格式指定为年，月，周，小时，分钟和秒的任意周期组合（例如P2W，P3M，PT1H30M，PT0.750S）。它们支持指定时区，该时区确定周期边界的起始位置以及返回的时间戳的时区。默认情况下，除非指定来源，否则年份从1月1日开始，月份从月份的1月开始，而周则从星期一开始。时区是可选的（默认为UTC）。原点是可选的（在给定的时区中默认为1970-01-01T00：00：00）。`{"type": "period", "period": "P2D", "timeZone": "America/Los_Angeles"}` | 是       |
| filter           | 过滤条件，[说明](http://druid.io/docs/0.9.2/querying/filters.html) | 否       |
| aggregations     | 聚合，[说明](http://druid.io/docs/0.9.2/querying/aggregations.html) | 是       |
| postAggregations | 后聚合，[说明](http://druid.io/docs/0.9.2/querying/post-aggregations.html) | 否       |
| limitSpec        | 返回指定数量的查询结果，类似mysql中的limit字句，[说明](http://druid.io/docs/0.9.2/querying/limitspec.html) | 否       |
| having           | 类似mysql中的having字句，[说明](http://druid.io/docs/0.9.2/querying/having.html) | 否       |
| context          | 上下文，[说明](http://druid.io/docs/0.9.2/querying/query-context.html) | 否       |



GroupBy queries can be executed using two different strategies. The default strategy for a cluster is determined by the "druid.query.groupBy.defaultStrategy" runtime property on the Broker. This can be overridden using "groupByStrategy" in the query context. If neither the context field nor the property is set, the "v2" strategy will be used.

- "v2", the default, is designed to offer better performance and memory management. This strategy generates per-segment results using a fully off-heap map. Data processes merge the per-segment results using a fully off-heap concurrent facts map combined with an on-heap string dictionary. This may optionally involve spilling to disk. Data processes return sorted results to the Broker, which merges result streams using an N-way merge. The broker materializes the results if necessary (e.g. if the query sorts on columns other than its dimensions). Otherwise, it streams results back as they are merged.
- "v1", a legacy engine, generates per-segment results on data processes (Historical, realtime, MiddleManager) using a map which is partially on-heap (dimension keys and the map itself) and partially off-heap (the aggregated values). Data processes then merge the per-segment results using Druid's indexing mechanism. This merging is multi-threaded by default, but can optionally be single-threaded. The Broker merges the final result set using Druid's indexing mechanism again. The broker merging is always single-threaded. Because the Broker merges results using the indexing mechanism, it must materialize the full result set before returning any results. On both the data processes and the Broker, the merging index is fully on-heap by default, but it can optionally store aggregated values off-heap.

### Differences between v1 and v2

Query API and results are compatible between the two engines; however, there are some differences from a cluster configuration perspective:

- groupBy v1 controls resource usage using a row-based limit (maxResults) whereas groupBy v2 uses bytes-based limits. In addition, groupBy v1 merges results on-heap, whereas groupBy v2 merges results off-heap. These factors mean that memory tuning and resource limits behave differently between v1 and v2. In particular, due to this, some queries that can complete successfully in one engine may exceed resource limits and fail with the other engine. See the "Memory tuning and resource limits" section for more details.
- groupBy v1 imposes no limit on the number of concurrently running queries, whereas groupBy v2 controls memory usage by using a finite-sized merge buffer pool. By default, the number of merge buffers is 1/4 the number of processing threads. You can adjust this as necessary to balance concurrency and memory usage.
- groupBy v1 supports caching on either the Broker or Historical processes, whereas groupBy v2 only supports caching on Historical processes.
- groupBy v2 supports both array-based aggregation and hash-based aggregation. The array-based aggregation is used only when the grouping key is a single indexed string column. In array-based aggregation, the dictionary-encoded value is used as the index, so the aggregated values in the array can be accessed directly without finding buckets based on hashing.

### Memory tuning and resource limits

When using groupBy v2, three parameters control resource usage and limits:

- `druid.processing.buffer.sizeBytes`: size of the off-heap hash table used for aggregation, per query, in bytes. At most `druid.processing.numMergeBuffers` of these will be created at once, which also serves as an upper limit on the number of concurrently running groupBy queries.
- `druid.query.groupBy.maxMergingDictionarySize`: size of the on-heap dictionary used when grouping on strings, per query, in bytes. Note that this is based on a rough estimate of the dictionary size, not the actual size.
- `druid.query.groupBy.maxOnDiskStorage`: amount of space on disk used for aggregation, per query, in bytes. By default, this is 0, which means aggregation will not use disk.

If `maxOnDiskStorage` is 0 (the default) then a query that exceeds either the on-heap dictionary limit, or the off-heap aggregation table limit, will fail with a "Resource limit exceeded" error describing the limit that was exceeded.

If `maxOnDiskStorage` is greater than 0, queries that exceed the in-memory limits will start using disk for aggregation. In this case, when either the on-heap dictionary or off-heap hash table fills up, partially aggregated records will be sorted and flushed to disk. Then, both in-memory structures will be cleared out for further aggregation. Queries that then go on to exceed `maxOnDiskStorage` will fail with a "Resource limit exceeded" error indicating that they ran out of disk space.

With groupBy v2, cluster operators should make sure that the off-heap hash tables and on-heap merging dictionaries will not exceed available memory for the maximum possible concurrent query load (given by `druid.processing.numMergeBuffers`). See the [basic cluster tuning guide](https://druid.apache.org/docs/latest/operations/basic-cluster-tuning.html) for more details about direct memory usage, organized by Druid process type.

Brokers do not need merge buffers for basic groupBy queries. Queries with subqueries (using a `query` dataSource) require one merge buffer if there is a single subquery, or two merge buffers if there is more than one layer of nested subqueries. Queries with [subtotals](https://druid.apache.org/docs/latest/querying/groupbyquery.html#more-on-subtotalsspec) need one merge buffer. These can stack on top of each other: a groupBy query with multiple layers of nested subqueries, and that also uses subtotals, will need three merge buffers.

Historicals and ingestion tasks need one merge buffer for each groupBy query, unless [parallel combination](https://druid.apache.org/docs/latest/querying/groupbyquery.html#parallel-combine) is enabled, in which case they need two merge buffers per query.

When using groupBy v1, all aggregation is done on-heap, and resource limits are done through the parameter `druid.query.groupBy.maxResults`. This is a cap on the maximum number of results in a result set. Queries that exceed this limit will fail with a "Resource limit exceeded" error indicating they exceeded their row limit. Cluster operators should make sure that the on-heap aggregations will not exceed available JVM heap space for the expected concurrent query load.

### Performance tuning for groupBy v2

#### Limit pushdown optimization

Druid pushes down the `limit` spec in groupBy queries to the segments on Historicals wherever possible to early prune unnecessary intermediate results and minimize the amount of data transferred to Brokers. By default, this technique is applied only when all fields in the `orderBy` spec is a subset of the grouping keys. This is because the `limitPushDown` doesn't guarantee the exact results if the `orderBy` spec includes any fields that are not in the grouping keys. However, you can enable this technique even in such cases if you can sacrifice some accuracy for fast query processing like in topN queries. See `forceLimitPushDown` in [advanced groupBy v2 configurations](https://druid.apache.org/docs/latest/querying/groupbyquery.html#groupby-v2-configurations).

#### Optimizing hash table

The groupBy v2 engine uses an open addressing hash table for aggregation. The hash table is initialized with a given initial bucket number and gradually grows on buffer full. On hash collisions, the linear probing technique is used.

The default number of initial buckets is 1024 and the default max load factor of the hash table is 0.7. If you can see too many collisions in the hash table, you can adjust these numbers. See `bufferGrouperInitialBuckets` and `bufferGrouperMaxLoadFactor` in [Advanced groupBy v2 configurations](https://druid.apache.org/docs/latest/querying/groupbyquery.html#groupby-v2-configurations).

#### Parallel combine

Once a Historical finishes aggregation using the hash table, it sorts the aggregated results and merges them before sending to the Broker for N-way merge aggregation in the broker. By default, Historicals use all their available processing threads (configured by `druid.processing.numThreads`) for aggregation, but use a single thread for sorting and merging aggregates which is an http thread to send data to Brokers.

This is to prevent some heavy groupBy queries from blocking other queries. In Druid, the processing threads are shared between all submitted queries and they are *not interruptible*. It means, if a heavy query takes all available processing threads, all other queries might be blocked until the heavy query is finished. GroupBy queries usually take longer time than timeseries or topN queries, they should release processing threads as soon as possible.

However, you might care about the performance of some really heavy groupBy queries. Usually, the performance bottleneck of heavy groupBy queries is merging sorted aggregates. In such cases, you can use processing threads for it as well. This is called *parallel combine*. To enable parallel combine, see `numParallelCombineThreads` in [Advanced groupBy v2 configurations](https://druid.apache.org/docs/latest/querying/groupbyquery.html#groupby-v2-configurations). Note that parallel combine can be enabled only when data is actually spilled (see [Memory tuning and resource limits](https://druid.apache.org/docs/latest/querying/groupbyquery.html#memory-tuning-and-resource-limits)).

Once parallel combine is enabled, the groupBy v2 engine can create a combining tree for merging sorted aggregates. Each intermediate node of the tree is a thread merging aggregates from the child nodes. The leaf node threads read and merge aggregates from hash tables including spilled ones. Usually, leaf processes are slower than intermediate nodes because they need to read data from disk. As a result, less threads are used for intermediate nodes by default. You can change the degree of intermediate nodes. See `intermediateCombineDegree` in [Advanced groupBy v2 configurations](https://druid.apache.org/docs/latest/querying/groupbyquery.html#groupby-v2-configurations).

Please note that each Historical needs two merge buffers to process a groupBy v2 query with parallel combine: one for computing intermediate aggregates from each segment and another for combining intermediate aggregates in parallel.

#  

#  

# scan模式



对比

| Timeseries     | TopN                   | GroupBy （推荐）       | scan                   |
| -------------- | ---------------------- | ---------------------- | ---------------------- |
| 无维度，可聚合 | 单维度，可聚合，可去重 | 多维度，可聚合，可去重 | 多维度，不去重，不聚合 |