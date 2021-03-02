### [Stream Partitions and Tasks](https://kafka.apache.org/26/documentation/streams/architecture#streams_architecture_tasks)

Slightly simplified, the maximum parallelism at which your application may run is bounded by the maximum number of stream tasks, which itself is determined by maximum number of partitions of the input topic(s) the application is reading from. For example, if your input topic has 5 partitions, then you can run up to 5 applications instances. These instances will collaboratively process the topic’s data. If you run a larger number of app instances than partitions of the input topic, the “excess” app instances will launch but remain idle; however, if one of the busy instances goes down, one of the idle instances will resume the former’s work.

### [Threading Model](https://kafka.apache.org/26/documentation/streams/architecture#streams_architecture_threads)

Kafka Streams allows the user to configure the number of **threads** that the library can use to parallelize processing within an application instance. Each thread can execute one or more tasks with their processor topologies independently. For example, the following diagram shows one stream thread running two stream tasks.

![img](https://kafka.apache.org/26/images/streams-architecture-threads.jpg)



Starting more stream threads or more instances of the application merely amounts to replicating the topology and having it process a different subset of Kafka partitions, effectively parallelizing processing. It is worth noting that there is no shared state amongst the threads, so no inter-thread coordination is necessary. This makes it very simple to run topologies in parallel across the application instances and threads. The assignment of Kafka topic partitions amongst the various stream threads is transparently handled by Kafka Streams leveraging [Kafka's coordination](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+Client-side+Assignment+Proposal) functionality.

As we described above, scaling your stream processing application with Kafka Streams is easy: you merely need to start additional instances of your application, and Kafka Streams takes care of distributing partitions amongst tasks that run in the application instances. You can start as many threads of the application as there are input Kafka topic partitions so that, across all running instances of an application, every thread (or rather, the tasks it runs) has at least one input partition to process.(您可以启动与输入Kafka主题分区一样多的应用程序线程，以便在应用程序的所有运行实例中，每个线程（或更确切地说，它运行的任务）至少具有一个要处理的输入分区。)





API:https://kafka.apache.org/26/documentation/streams/developer-guide/dsl-api.html#writing-streams-back-to-kafka



#### 1.KStream

​    只有DSL中有KStream的概念。KStream是一个流式记录的抽象，是一个无界的数据集。用一个表来类比，数据记录在一个流中可以理解为一直在进行“INSERT”的动作。只进行追加。

#### 2.KTable

​    只有DSL中有KTable的概念。KTable是一个changelog stream的抽象，每个数据记录都被表示为一个update。如果key在KTable中已经存在，则表示为一个“UPDATE”；如果不存在，则表示为一个“INSERT”。

#### 3.GlobalKTable

​    只有DSL中有GlobalKTable的概念。和KTable一样，GlobalKTable也是一个changelog stream的抽象，每个数据记录都被表示为一个update。
​    KTable存储的数据根据key进行分区的，GlobalKTable是不分区的，且存储的数据足够小，能完全装入内存，因此保证每个流任务都有所有数据的完整副本，而不关心传入record的key是什么。
​    基于以上的特性，GlobalKTable有如下优点：

- 更高效的join操作：当链接多个join操作时，使用GlobalKTable效率更高，它不需要co-partitioned（类似shuffle的一种操作）的发生。
- 可用于将信息“广播”到应用程序的所有实例。