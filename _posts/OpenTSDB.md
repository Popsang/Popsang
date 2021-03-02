# 定义

基于Hbase的分布式的，可伸缩的时间序列数据库。

主要用途，就是做监控系统；譬如收集大规模集群（包括网络设备、操作系统、应用程序）的监控数据并进行存储，查询。

# 核心概念

OpenTSDB存储的一些核心概念：

1. Metric：即平时我们所说的监控项。譬如上面的CPU使用率。
2. Tags：就是一些标签，在OpenTSDB里面，Tags由tagk和tagv组成，即tagk=takv。标签是用来描述Metric的，譬如上面为了标记是服务器A的CpuUsage，tags可为hostname=qatest。
3. Value：一个Value表示一个metric的实际数值，譬如上面的99%。
4. Timestamp：即时间戳，用来描述Value是什么时候的；譬如上面的21:00。
5. Data Point：即某个Metric在某个时间点的数值。Data Point包括以下部分：Metric、Tags、Value、Timestamp。

# 存储方案（核心存储有两张表）

## a 表tsdb-uid

![img](https://pic3.zhimg.com/80/v2-270fb74fa9e0cd5f7b00959230d4e086_hd.jpg)

Metric、TagKey和TagValue都会被分配一个相同的固定长度的UniqueID，默认是三个字节。tsdb-uid表使用两个ColumnFamily，存储了Metric、TagKey和TagValue与UniqueID的映射和反向映射，总共是6个Map的数据。从图中的例子可以解读出：

- - TagKey为'host'，对应的UniqueID为'001'
  - TagValue为'static'，对应的UniqueId为'001'
  - Metric为'proc.loadavg.1m'，对应的UniqueID为'052'

为每一个Metric、TagKey和TagValue都分配UniqueID的好处，一是大大降低了存储空间和传输数据量，每个值都只需要3个字节就可以表示，这个压缩率是很客观的；二是采用固定长度的字节，可以很方便的从row key中解析出所需要的值，并且能够大大减少Java堆内的内存占用（bytes相比String能节省很多的内存占用），降低GC的压力。
不过采用固定字节的UID编码后，对于UID的个数是有上限要求的，3个字节最多只允许有16777216个不同的值（2的24次方3*8），不过在大部分场景下都是够用的。当然这个长度是可以调整的，不过不支持动态更改。

## b 表tsdb



![胡硕 > OpenTSDB笔记 > image2019-8-16_15-32-56.png](http://km.vivo.xyz/download/attachments/94919409/image2019-8-16_15-32-56.png?version=1&modificationDate=1565940776000&api=v2)

该表中，同一个小时内的数据会存储在同一行，行中的每一列代表一个数据点。如果是秒级精度，那一行最多会有3600个点，如果是毫秒级精度，那一行最多会有3600000个点。

1. 1. RowKey的设计
      RowKey其实和上面的metric|timestamp|value|host=web42|pool=static类似；但是区别是，OpenTSDB为了节省存储空间，将每个部分都做了映射。在OpenTSDB里面有这样的映射，metric-->3字节整数、tagk-->3字节整数、tagv-->3字节整数上图的映射关系为，proc.loadavg.1m-->052、host-->001、web42-->028、pool-->047、static-->001
   2. column的设计为了方便后期更进一步的节省空间。OpenTSDB将一个小时的数据，保存在一行里面。所以上面的timestamp1234567890，会先模一下小时，得出1234566000，然后得到的余数为1890，表示的是它是在这个小时里面的第1890秒；然后将1890作为column name，而0.42即为column value

# 总体架构

![胡硕 > OpenTSDB笔记 > image2019-8-16_15-34-32.png](http://km.vivo.xyz/download/attachments/94919409/image2019-8-16_15-34-32.png?version=1&modificationDate=1565940873000&api=v2)

Servers：就是服务器了，上面的C就是指Collector，可以理解为OpenTSDB的agent，通过Collector收集数据，推送数据；TSD：

 TSD是对外通信的无状态的服务器，Collector可以通过TSD简单的RPC协议推送监控数据；另外TSD还提供了一个web UI页面供数据查询；另外也可以通过脚本查询监控数据，对监控数据做报警

 HBase：TSD收到监控数据后，是通过AsyncHbase这个库来将数据写入到HBase；AsyncHbase是完全异步、非阻塞、线程安全的Hbase客户端，使用更少的线程、锁以及内存，可以提供更高的吞吐量，特别对于大量的写操作。



# 官方文档学习

## / api / stats

此 endpoints 提供正在运行的TSD的统计信息列表。子 endpoints 返回有关其他TSD组件的详细信息，例如JVM，线程状态或存储客户端。所有统计数据都是只读的。

OpenTSDB提供了许多有关其性能的指标，可以通过各种API端点进行访问。可以通过“stats”选项卡从GUI、 通过/api/stats 的http api 或通过/stats 的传统api访问主要统计信息。telnet风格的api还支持通过cli获取的“stats”命令。这些可以很容易地以您喜欢的任何时间间隔发布回OpenTSDB。

其他可用的统计信息包括JVM信息、存储详细信息（例如每个区域的客户机HBase统计信息）和执行的查询详细信息。有关其他端点的详细信息，请参阅/api/stats。

来自主统计端点的所有指标都包含一个主机标记，其中包含TSD运行的主机的名称。如果设置了tsd.stats.canonical配置标志，则此标志将更改为fqdn，并且tsd将尝试解析其主机名以返回完全限定的域名。目前所有的统计数据都是整数值。每个统计请求将实时获取统计信息，因此时间戳将反映TSD主机上的当前时间。

总结：存储了各项OpenTSDB数据，包括主机状态等。



## / api / query

可能是API中最有用的 endpoints ， `/api/query` 允许以所选序列化程序确定的各种格式从存储系统中提取数据。可以通过1.0查询字符串格式或正文内容提交查询。从2.2数据匹配开始，可以使用 `DELETE` 动词删除查询。必须启用配置参数 `tsd.http.query.allow_delete` 才能允许删除。删除的数据将在查询结果中返回。第二次执行查询应该返回空结果。



警告

删除数据是永久性的。还要注意，删除时，可能会删除开始和结束时间边界之外的某些数据，因为数据是按小时存储的。

| 名称                 | 数据类型        | 必需 | 描述                                                         | 默认      | QS                 | RW           | 示例 |
| :------------------- | :-------------- | :--- | :----------------------------------------------------------- | :-------- | :----------------- | :----------- | :--- |
| start                | String，Integer | 必需 | 查询的开始时间。这可以是相对或绝对时间戳。有关详细信息，请参阅 [Querying or Reading Data](https://www.docs4dev.com/docs/zh/opentsdb/2.3/reference/api_http-..-user_guide-query-index.html) 。 | 开始      | 1小时前            |              |      |
| end                  | String，Integer | 可选 | 查询的结束时间。如果未提供，TSD将假定服务器上的本地系统时间。这可以是相对或绝对时间戳。有关详细信息，请参见 [Querying or Reading Data](https://www.docs4dev.com/docs/zh/opentsdb/2.3/reference/api_http-..-user_guide-query-index.html) 。 | 当前时间  | 结束               | 1s-ago       |      |
| queries              | Array           | 必需 | 用于选择要返回的时间序列的一个或多个子查询。这些可能是公制 `m` 或TSUID `tsuids` 查询 | m或tsuids | 见下文             |              |      |
| noAnnotations        | Boolean         | 可选 | 是否返回带有查询的注释。默认设置是返回请求的时间 Span 的注释，但此标志可以禁用返回。这会影响本地和全局注释和覆盖`globalAnnotations` | false     | no_annotations     | false        |      |
| globalAnnotations    | Boolean         | 可选 | 查询是否应检索所请求时间 Span 的全局注释                     | false     | global_annotations | true         |      |
| msResolution（或ms） | 布尔            | 可选 | 是否以毫秒或秒为单位输出数据点时间戳。建议使用msResolution标志。如果未提供此标志并且在一秒内存在多个数据点，则将使用查询的聚合函数对这些数据点进行下采样。 | false     | ms                 | true         |      |
| showTSUIDs           | Boolean         | 可选 | 是否输出与结果中的时间序列关联的TSUID。如果将多个时间序列聚合到一个集合中，则将以排序的方式返回多个TSUID | false     | show_tsuids        | true         |      |
| showSummary（2.2）   | Boolean         | 可选 | 是否在结果中显示查询周围的计时摘要。这将在 Map 中创建另一个与数据点对象不同的对象。见 [Query Details and Stats](https://www.docs4dev.com/docs/zh/opentsdb/2.3/reference/api_http-..-user_guide-query-stats.html) | false     | show_summary       | true         |      |
| showStats（2.2）     | Boolean         | 可选 | 是否在结果中显示查询的详细时序。这将在 Map 中创建另一个与数据点对象不同的对象。见 [Query Details and Stats](https://www.docs4dev.com/docs/zh/opentsdb/2.3/reference/api_http-..-user_guide-query-stats.html) | false     | show_stats         | true         |      |
| showQuery（2.2）     | Boolean         | 可选 | 是否使用查询结果返回原始子查询。如果请求包含许多子查询，那么这是确定哪些结果属于哪个子查询的好方法。请注意，对于 `*` 或通配符查询，这会产生大量重复输出。 | false     | show_query         | true         |      |
| delete               | Boolean         | 可选 | 可以使用POST传递给JSON，以删除与给定查询匹配的任何数据点。   | false     | W                  | true         |      |
| timezone（2.3）      | String          | 可选 | 基于日历的下采样的可选时区。必须是TSD服务器上安装的JRE支持的有效 [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) 数据库名称。 | UTC       | timezone           | Asia / Kabul |      |
| useCalendar（2.3）   | 布尔            | 可选 | 是否使用基于给定时区的日历进行下采样间隔                     | false     | true               |              |      |

## 子查询