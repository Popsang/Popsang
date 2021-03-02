# Redis

# 数据格式

https://www.jianshu.com/p/f09480c05e42

Redis是典型的Key-Value类型数据库，Key为字符类型，Value的类型常用的为五种类型：String、Hash 、List 、 Set 、 Ordered Set

##### redisObject 核心对象

> Redis 内部使用一个 redisObject 对象来表示所有的 key 和 value。

1. type ：代表一个 value 对象具体是何种数据类型。
2. encoding ：是不同数据类型在 redis 内部的存储方式，比如：type=string 代表 value 存储的是一个普通字符串，那么对应的 encoding 可以是 raw 或者是 int，如果是 int 则代表实际 redis 内部是按数值型类存储和表示这个字符串的，当然前提是这个字符串本身可以用数值表示，比如："123" "456"这样的字符串。
3. vm 字段：只有打开了 Redis 的虚拟内存功能，此字段才会真正的分配内存，该功能默认是关闭状态的。 Redis 使用 redisObject 来表示所有的 key/value 数据是比较浪费内存的，当然这些内存管理成本的付出主要也是为了给 Redis 不同数据类型提供一个统一的管理接口，实际作者也提供了多种方法帮助我们尽量节省内存使用。

## key

### 过期删除

过期数据的清除从来不容易，为每一条key设置一个timer，到点立刻删除的消耗太大，每秒遍历所有数据消耗也大，**Redis使用了一种相对务实的做法： 当client主动访问key会先对key进行超时判断，过时的key会立刻删除。 如果clien永远都不再get那条key呢？ 它会在Master的后台，每秒10次的执行如下操作： 随机选取100个key校验是否过期，如果有25个以上的key过期了，立刻额外随机选取下100个key(不计算在10次之内)。**可见，如果过期的key不多，它最多每秒回收200条左右，如果有超过25%的key过期了，它就会做得更多，但只要key不被主动get，它占用的内存什么时候最终被清理掉只有天知道。



### key存储方式

返回 key 的数据类型，数据类型有：

- none (key不存在)
- string (字符串)
- list (列表)
- set (集合)
- zset (有序集)
- hash (哈希表)



常用操作

1. Key的长度限制：Key的最大长度不能超过1024字节，在实际开发时不要超过这个长度，但是Key的命名不能太短，要能唯一标识缓存的对，作者建议按照在关系型数据库中的库表唯一标识字段的方式来命令Key的值，用分号分割不同数据域，用点号作为单词连接。
2. Key的查询：Keys，返回匹配的key，支持通配符如 “keys a*” 、 “keys a?c”，但不建议在生产环境大数据量下使用。
3. 对Key对应的Value进行的排序：Sort命令对集合按数字或字母顺序排序后返回或另存为list，还可以关联到外部key等。因为复杂度是最高的O(N+Mlog(M))*(N是集合大小，M 为返回元素的数量)，有时会安排到slave上执行。官网链接https://redis.io/commands/sort
4. Key的超时操作：Expire（指定失效的秒数）/ExpireAt（指定失效的时间戳）/Persist（持久化）/TTL（返回还可存活的秒数），关于Key超时的操作。默认以秒为单位，也有p字头的以毫秒为单位的版本

## String（字符串类型的Value）

可以是String，也可是是任意的byte[]类型的数组，如图片等。**String 在 redis 内部存储默认就是一个字符串，被 redisObject 所引用，当遇到 incr,decr 等操作时会转成数值型进行计算，此时 redisObject 的 encoding 字段为int。**
[https://redis.io/commands#string](https://link.jianshu.com?t=https://redis.io/commands#string)

1. 大小限制：最大为512Mb，基本可以**存储任意图片**啦。
2. 常用命令的时间复杂度为O(1)，读写一样的快。
3. 对String代表的数字进行增减操作（没有指定的Key则设置为0值，然后在进行操作）：Incr/IncrBy/IncrByFloat/Decr/DecrBy（原子性）， 可以用来做计数器，做自增序列，也可以用于限流，令牌桶计数等。key不存在时会创建并贴心的设原值为0。IncrByFloat专门针对float。
4. 设置Value的安全性：SetNx命令仅当key不存在时才Set（原子性操作）。**可以用来选举Master或做分布式锁：所有Client不断尝试使用SetNx master myName抢注Master，成功的那位不断使用Expire刷新它的过期时间。如果Master倒掉了key就会失效，剩下的节点又会发生新一轮抢夺。**SetEx， Set + Expire 的简便写法，p字头版本以毫秒为单位。
5. 获取：GetSet（原子性）， 设置新值，返回旧值。比如一个按小时计算的计数器，可以用GetSet获取计数并重置为0。这种指令在服务端做起来是举手之劳，客户端便方便很多。MGet/MSet/MSetNx， 一次get/set多个key。
6. 其他操作：Append/SetRange/GetRange/StrLen，对文本进行扩展、替换、截取和求长度，只对特定数据格式如字段定长的有用，json就没什么用。
7. BitMap的用法：GetBit/SetBit/BitOp,与或非/BitCount， **BitMap的玩法，比如统计今天的独立访问用户数时，每个注册用户都有一个offset，他今天进来的话就把他那个位设为1，用BitCount就可以得出今天的总人数**。

##  Hash（HashMap，哈希映射表）

Redis 的 Hash 实际是内部存储的 Value 为一个 HashMap，并提供了直接存取这个 Map 成员的接口。Hash将对象的各个属性存入Map里，可以只读取/更新对象的某些属性。另外不同的模块可以只更新自己关心的属性而不会互相并发覆盖冲突。


不同程序通过 key（用户 ID） + field（属性标签）就可以并发操作各自关心的属性数据
[https://redis.io/commands#hash](https://link.jianshu.com?t=https://redis.io/commands#hash)

### 实现原理

Redis Hash 对应 Value 内部实际就是一个 HashMap，实际这里会有2种不同实现， 这个 Hash 的成员比较少时 Redis 为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的 HashMap 结构，对应的 value redisObject 的 encoding 为 zipmap，当成员数量增大时会自动转成真正的 HashMap，此时 encoding 为 ht。一般操作复杂度是O(1)，要同时操作多个field时就是O(N)，N是field的数量。

##### 常用操作

1. O(1)操作：hget、hset等等
2. O(n)操作：hgetallRedis 可以直接取到全部的属性数据，但是如果内部 Map 的成员很多，那么涉及到遍历整个内部 Map 的操作，**由于 Redis 单线程模型的缘故，这个遍历操作可能会比较耗时，而另其它客户端的请求完全不响应**，这点需要格外注意。


![胡硕 > Redis > 5064562-b81e7afd35978bcf.jpg](http://km.vivo.xyz/download/attachments/107940909/5064562-b81e7afd35978bcf.jpg?version=1&modificationDate=1582803283000&api=v2)

##  List（双向链表）（Todo）

> Redis list 的应用场景非常多，也是 Redis 最重要的数据结构之一，**比如 twitter 的关注列表，粉丝列表等都可以用 Redis 的 list 结构来实现，还提供了生产者消费者阻塞模式（B开头的命令），常用于任务队列，消息队列等**。



## set（HashSet）

### set（HashSet）

> Set就是HashSet，可以将重复的元素随便放入而Set会自动去重，底层实现也是HashMap，并且 set 提供了判断某个成员是否在一个 set 集合内的重要接口，这个也是 list 所不能提供的。

##### 实现原理

set 的内部实现是一个 value 永远为 null 的 HashMap，实际就是通过计算 hash 的方式来快速排重的，这也是 set 能提供判断一个成员是否在集合内的原因。

##### 常用操作

1. 增删改查：SAdd/SRem/SIsMember/SCard/SMove/SMembers等等。除了SMembers都是O(1)。
2. 集合操作：SInter/SInterStore/SUnion/SUnionStore/SDiff/SDiffStore，各种集合操作。交集运算可以用来**显示在线好友(在线用户 交集 好友列表)，共同关注(两个用户的关注列表的交集)**。O(N)，并集和差集的N是集合大小之和，交集的N是小的那个集合的大小的2倍。



##  Sorted Set（插入有序Set集合）

> set 不是自动有序的，而** sorted set 可以通过用户额外提供一个优先级（score）的参数来为成员排序**，**并且是插入有序的，即自动排序**。当你需要一个有序的并且不重复的集合列表，那么可以选择 sorted set 数据结构，比如 twitter 的 public **timeline 可以以发表时间作为 score 来存储，这样获取时就是自动按时间排好序的**。

##### 实现方式

> **内部使用 HashMap 和跳跃表（SkipList）来保证数据的存储和有序**

Sorted Set的实现是HashMap(element->score, 用于实现ZScore及判断element是否在集合内)，和SkipList(score->element,按score排序)的混合体。SkipList有点像平衡二叉树那样，不同范围的score被分成一层一层，每层是一个按score排序的链表。

##### 常用操作

> ZAdd/ZRem是O(log(N))；ZRangeByScore/ZRemRangeByScore是O(log(N)+M)，N是Set大小，M是结果/操作元素的个数。复杂度的log取对数很关键，可以使，1000万大小的Set，复杂度也只是几十不到。但是，如果一次命中很多元素M很大则复杂度很高。

1. ZRange/ZRevRange，按排序结果的范围返回元素列表，可以为正数与倒数。
2. ZRangeByScore/ZRevRangeByScore，按score的范围返回元素，可以为正数与倒数。
3. ZRemRangeByRank/ZRemRangeByScore，按排序/按score的范围限删除元素。
4. ZCount，统计按score的范围的元素个数。
5. ZRank/ZRevRank ，显示某个元素的正/倒序的排名。
6. ZScore/ZIncrby，显示元素的Score值/增加元素的Score。
7. ZAdd(Add)/ZRem(Remove)/ZCard(Count)，ZInsertStore(交集)/ZUnionStore(并集)，与Set相比，少了IsMember和差集运算。

# redis命令

## TYPE KEY_NAME 

命令用于返回 key 所储存的值的类型

## HASH

[HGET key field](https://www.runoob.com/redis/hashes-hget.html)：

# SMEMBERS key

Redis Smembers 命令返回集合中的所有的成员。 不存在的集合 key 被视为空集合。

# Redisson

Redisson是架设在Redis基础上的一个Java驻内存数据网格（In-Memory Data Grid）。充分的利用了Redis键值数据库提供的一系列优势，基于Java实用工具包中常用接口，为使用者提供了一系列具有分布式特性的常用工具类。使得原本作为协调单机多线程并发程序的工具包获得了协调分布式多机多线程并发系统的能力，大大降低了设计和研发大规模分布式系统的难度。同时结合各富特色的分布式服务，更进一步简化了分布式环境中程序相互之间的协作。

## 场景

分布式应用，分布式缓存，分布式回话管理，分布式服务（任务，延迟任务，执行器），分布式redis客户端

## 功能

- 支持同步/异步/异步流/管道流方式连接
- 多样化数据序列化
- 集合数据分片
- 分布式对象
- 分布式集合
- 分布式锁和同步器
- 分布式服务
- 独立节点模式
- 三方框架整合



# redis 锁

## reddison实现

可重入锁

```
public void testReentrantLock(RedissonClient redisson){         RLock lock = redisson.getLock("anyLock");        try{            // 1. 最常见的使用方法            //lock.lock();             // 2. 支持过期解锁功能,10秒钟以后自动解锁, 无需调用unlock方法手动解锁            //lock.lock(10, TimeUnit.SECONDS);             // 3. 尝试加锁，最多等待3秒，上锁以后10秒自动解锁            boolean res = lock.tryLock(3, 10, TimeUnit.SECONDS);            if(res){    //成功                // do your business             }        } catch (InterruptedException e) {            e.printStackTrace();        } finally {            lock.unlock();        }     }
```



boolean tryLock(long waitTime, long leaseTime, [TimeUnit](https://docs.oracle.com/javase/6/docs/api/java/util/concurrent/TimeUnit.html?is-external=true) unit) throws [InterruptedException](https://docs.oracle.com/javase/6/docs/api/java/lang/InterruptedException.html?is-external=true)

`waitTime` - the maximum time to aquire the lock 等待时间

`leaseTime` - lease time 释放时间

`unit` - time unit 单位



# redission 与 StringRedisTemplate 

推荐 redission

# Redis性能提升-Pipeline和Batch操作

Redis是一个c/s模式的TCP Server,本身是基于TCP协议的一个Request/Response protocol模式,使用和HTTP类似的请求响应协议.在OSI七层协议(七层结构：物理层、数据链路层、网络层、传输层、会话层、表示层、应用层)模型中,TCP属于传输层,HTTP属于应用层,越接近底层的协议一般来说传输速度会更快,所以Redis才有高速缓存,高效的读写功能.
  Redis Client和Redis Server之间使用TCP进行连接,一个Client可以通过一个Socket连接发起多个请求命令,每个请求命令发出后Client通常会阻塞等待Redis服务处理,Redis Server处理完后会将结果以响应报文返回给Client,因此当执行多条命令的时候,如果命令的发送是一条一条的发送,那么每一条命令的执行都需要等待上一条命令执行成功.如：get ‘0’,get ‘1’,get ‘2’,执行过程如下图:



![这里写图片描述](https://img-blog.csdn.net/20171217193410587?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRzBfaHc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



*1.使用Batch操作*
利用Batch(mget,mset之类的单条命令)处理多个key的命令,一次性将多个命令提交过去，极大的较少了在网络传输方面带来的损耗.

伪代码

> String productID = String.format("productID_{0}", 1);
> List<Integer> list = new List<>();
> for (int i = 0; i < 10; i++)
> {
> list.Add(i);
> }
> redis.SetAdd(productID,list.toArray(new int[list.size()]));

使用PipeLine管道
  Pipeline可以从client打包多条命令后一起发出,redis服务端会在处理完多条命令后将多条命令的处理结果打包到一起返回给客户端,与单条命令顺序执行相比,使用Pipeline极大的减少了客户端与redis server的通信次数,从而降低往返延时的时间.其过程如下图所示:

![这里写图片描述](https://img-blog.csdn.net/20171217200741206?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvRzBfaHc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



# reddsion cache等 数据结构