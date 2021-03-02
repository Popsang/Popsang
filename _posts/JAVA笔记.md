# 比较好的写法

## map->map

Map<Long, Long> finalMap = nodeIdAndParentId.stream().collect(Collectors.toMap(NodeDO::getNodeId,NodeDO::getParentId));



# 常用stream

### List转String

string.join(","List);

//List.stream().collect(Collectors.joining(",")





------

# Streams

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- - 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
  - 数据源 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
  - 聚合操作 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- - Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
  - 内部迭代： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## Arrays.stream()

## forEach

## map:

List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);

// 获取对应的平方数

List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());

## distinct() ： 去重

## collect(Collectors.toList());  变为列表

**Collector****使用(有时间可以看)**

## Filter ：写过滤条件

List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");

// 获取空字符串的数量

int count = strings.stream().filter(string -> string.isEmpty()).count();



## limit

## sorted

## 统计：

int[] intArray = {12,3,34,67,100,99};

/** 第一种构造intStream **/

IntStream intStream = IntStream.of(intArray);

/** 第二种构造intStream **/

//IntStream intStream2 = IntStream.of(12,3,34,67,100,99);

/** 这个是重点，获得当前int数组的统计信息，包括 **/

IntSummaryStatistics statistics = intStream.summaryStatistics();

/** 计算出最大，最小，平均等等，是不是很好用，赶紧get起来 **/

System.out.println("the max:" + statistics.getMax());

System.out.println("the min:" + statistics.getMin());

System.out.println("the average:" + statistics.getAverage());

System.out.println("the sum:" + statistics.getSum());

System.out.println("the count:" + statistics.getCount());



https://blog.csdn.net/u014042066/article/details/76360380

Java8新特性，用于过滤字符串等，使用stream.filter()去过滤集合，使用collect()将stream转化为一个集合。:



格式：

List<String> result = lines.stream()        // convert list to stream

​        .filter(line -> !"ricky".equals(line))   // we dont like ricky

​        .collect(Collectors.toList());       // collect the output and convert streams to a List



等价于：

 List<String> result = new ArrayList<>();

 for (String line : lines) {

   if (!"ricky".equals(line)) { // we dont like ricky

​     result.add(line);

   }

 }

## findFirst

Optional<T> min(Comparator<? super T> comparator); 
Optional<T> max(Comparator<? super T> comparator);
Optional<T> findFirst(); 
Optional<T> findAny();



这4个函数，都是返回的Optional对象，关于这个对象，如果有不清楚的，后期我们会做详细的介绍，现在只需要知道，这个类是对null做处理的，就可以了；

findFirst和findAny，通过名字，就可以看到，对这个集合的流，做一系列的中间操作后，可以调用findFirst，返回集合的第一个对象，findAny返回这个集合中，取到的任何一个对象；通过这样的描述，我们也可以知道，在串行的流中，findAny和findFirst返回的，都是第一个对象；而在并行的流中，findAny返回的是最快处理完的那个线程的数据，所以说，在并行操作中，对数据没有顺序上的要求，那么findAny的效率会比findFirst要快的；



## parallelStream的作用


Stream具有平行处理能力，处理的过程会分而治之，也就是将一个大任务切分成多个小任务，这表示每个任务都是一个操作，因此像以下的程式片段：

> GuavaList<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
> numbers.parallelStream()
> .forEach(out::println); 
>
> 
>
> 你得到的展示顺序不一定会是1、2、3、4、5、6、7、8、9，而可能是任意的顺序，就forEach()这个操作來讲，如果平行处理时，希望最后顺序是按照原来Stream的数据顺序，那可以调用forEachOrdered()。例如

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
numbers.parallelStream()
.forEachOrdered(out::println); 



##  groupingBy():进行分组 

是Stream API中最强大的收集器Collector之一，提供与SQL的GROUP BY子句类似的功能。

例1：



> List<String> strings = List.of("a", "bb", "cc", "ddd");
>
> Map<Integer, List<String>> result = strings.stream() .collect(groupingBy(String::length));
>
> System.out.println(result); *// {1=[a], 2=[bb, cc], 3=[ddd]}*

例2：



> ```
> list<String> strings = List.of("a", "bb", "cc", "ddd");
> 
> Map<Integer, TreeSet<String>> result = strings.stream()
>   .collect(groupingBy(String::length, toCollection(TreeSet::new)));
> 
> System.out.println(result); // {1=[a], 2=[bb, cc], 3=[ddd]}
> ```

## anyMatch 

anyMatch表示，判断的条件里，任意一个元素成功，返回true

## allMatch 

allMatch表示，判断条件里的元素，所有的都是，返回true

# Lambda表达式 



https://juejin.im/post/5abc9ccc6fb9a028d6643eea

 Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。

Java8 新特性

- - 可选类型声明：不需要声明参数类型，编译器可以统一识别参数值。
  - 可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号。
  - 可选的大括号：如果主体包含了一个语句，就不需要使用大括号。
  - 可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。



lambda 表达式只能引用标记了 final 的外层局部变量，这就是说不能在 lambda 内部修改定义在域外的局部变量，否则会编译错误。

TODO:

lambda 表达式的局部变量可以不用声明为 final，但是必须不可被后面的代码修改（即隐性的具有 final 的语义）



int num = 1; 

Converter<Integer, String> s = (param) -> System.out.println(String.valueOf(param + num));

s.convert(2);

num = 5; 

//报错信息：Local variable num defined in an enclosing scope must be final or effectively

 final



在 Lambda 表达式当中不允许声明一个与局部变量同名的参数或者局部变量。



注：Java中的“::”

person -> person.getAge();

可以替换成

Person::getAge

这种[方法引用]或者说[双冒号运算]对应的参数类型是Function<T,R> T表示传入类型，R表示返回类型。比如表达式person -> person.getAge(); 传入参数是person，返回值是person.getAge()，那么方法引用Person::getAge就对应着Function<Person,Integer>类型。





# Atomic属性

用来确保线程安全的，属于concurrent包

有时间看一下



# Builder（）用法

析构函数

#  Java 值引用与引用传递

**无论是基本类型和是引用类型，在实参传入形参时，都是值传递，也就是说传递的都是一个副本，而不是内容本身。**

测试一：

```
public static void PersonCrossTest(Person person){
        System.out.println("传入的person的name："+person.getName());
        person.setName("我是张小龙");
        System.out.println("方法内重新赋值后的name："+person.getName());
    }
//测试
public static void main(String[] args) {
        Person p=new Person();
        p.setName("我是马化腾");
        p.setAge(45);
        PersonCrossTest(p);
        System.out.println("方法执行后的name："+p.getName());
}
 
 
 
 
//输出
传入的person的name：我是马化腾
方法内重新赋值后的name：我是张小龙
方法执行后的name：我是张小龙
```

测试二

```
public static void PersonCrossTest(Person person){
        System.out.println("传入的person的name："+person.getName());
        person=new Person();//加多此行代码
        person.setName("我是张小龙");
        System.out.println("方法内重新赋值后的name："+person.getName());
 }
 
 
//输出
传入的person的name：我是马化腾
方法内重新赋值后的name：我是张小龙
方法执行后的name：我是马化腾
```

# Java List安全删除方法

https://juejin.im/post/5b92844a6fb9a05d290ed46c

list源码：

可以看到这个 remove() 方法被重载了，一种是根据下标删除，一种是根据元素删除，这也都很好理解。

根据下标删除的 remove() 方法，大致的步骤如下：

- 1、检查有没有下标越界，就是检查一下当前的下标有没有大于等于数组的长度
- 2、列表被修改（add和remove操作）的次数加1
- 3、保存要删除的值
- 4、计算移动的元素数量
- 5、删除位置后面的元素向左移动，这里是用数组拷贝实现的
- 6、将最后一个位置引用设为 null，使垃圾回收器回收这块内存
- 7、返回删除元素的值

根据元素删除的 remove() 方法，大致的步骤如下：

- 1、元素值分为null和非null值
- 2、循环遍历判等
- 3、调用 fastRemove(i) 函数
  - 3.1、修改次数加1
  - 3.2、计算移动的元素数量
  - 3.3、数组拷贝实现元素向左移动
  - 3.4、将最后一个位置引用设为 null
  - 3.5、返回 fase
- 4、返回 true



安全写法：

```
// 方法五：迭代器，使用迭代器的remove()方法删除，可以删除重复的元素，但不推荐
Iterator iterator = list.iterator();
while (iterator.hasNext()) {
  if(iterator.next().equals(elem)) {
    iterator.remove();
  }
}
 
 
//removeIf写法
List list = new ArrayList(Arrays.asList(1,2,3,4,5));
long last = System.currentTimeMillis();
list.removeIf(a -> a.equals(2));
 
 
default boolean removeIf(Predicate<? super E> filter) {
        //判断是否为null
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            //迭代出现运行时异常或者错误由由Predicate被转发给调用者
            if (filter.test(each.next())) {
                //remove底层调用的是System.arraycopy方法，是个C++编写的native方法，操作的是指针，所有比较快
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
```





# 代码中比较经典写法

l List<WeightUrl> urls = getUrlsByGroupName(groupName);

urls.removeIf(weightUrl -> weightUrl.getUrl().equalsIgnoreCase(url));

解析：

可以看见 urls 是一个list，通过removeIf函数，进行一个过滤操作，删除与url相同的操作。

> ```
>  statsMap.compute(AVG, (k, v) -> {
>         if (-1 == v) {
>             return delay;
>         }
>         return (v + delay) / 2;
>     });
>     statsMap.compute(MAX, (k, v) -> Math.max(v, delay));
> 
> }
> ```



 key不管存在不在都会执行后面的函数，并保存到map中

# enum

枚举类型

# 重写equals以及HashCode

1.equals()的所属以及内部原理（即Object中equals方法的实现原理）

说起equals方法，我们都知道是超类Object中的一个基本方法，用于检测一个对象是否与另外一个对象相等。而在Object类中这个方法实际上是判断两个对象是否具有相同的引用，如果有，它们就一定相等。其源码如下：



```
public boolean equals(Object obj) { return (this == obj); }
```

实际上我们知道所有的对象都拥有标识(内存地址)和状态(数据)，同时“==”比较两个对象的的内存地址，所以说 Object 的 equals() 方法是比较两个对象的内存地址是否相等，即若 object1.equals(object2) 为 true，则表示 equals1 和 equals2 实际上是引用同一个对象。



equals重写规则

> - 自反性。对于任何非null的引用值x，x.equals(x)应返回true。
> - 对称性。对于任何非null的引用值x与y，当且仅当：y.equals(x)返回true时，x.equals(y)才返回true。
> - 传递性。对于任何非null的引用值x、y与z，如果y.equals(x)返回true，y.equals(z)返回true，那么x.equals(z)也应返回true。
> - 一致性。对于任何非null的引用值x与y，假设对象上equals比较中的信息没有被修改，则多次调用x.equals(y)始终返回true或者始终返回false。
> - 对于任何非空引用值x，x.equal(null)应返回false。

重写HashCode原因

> 学过数据结构的同学都知道Map接口的类会使用到键对象的哈希码，当我们调用put方法或者get方法对Map容器进行操作时，都是根据键对象的哈希码来计算存储位置的，因此如果我们对哈希码的获取没有相关保证，就可能会得不到预期的结果。在java中，我们可以使用hashCode()来获取对象的哈希码，其值就是对象的存储地址，这个方法在Object类中声明，因此所有的子类都含有该方法。那我们先来认识一下hashCode()这个方法吧。

字符串s与t拥有相同的散列码，这是因为字符串的散列码是由内容导出的。而字符串缓冲sb与tb却有着不同的散列码，这是因为StringBuilder没有重写hashCode方法，它的散列码是由Object类默认的hashCode方法计算出来的对象存储地址，所以散列码自然也就不同了。那么我们该如何重写出一个较好的hashCode方法呢，其实并不难，我们只要合理地组织对象的散列码，就能够让不同的对象产生比较均匀的散列码。例如下面的例子：



附：String hashcode计算方式

```
public int hashCode() {    int h = hash;    if (h == 0 && value.length > 0) {        char val[] = value;         for (int i = 0; i < value.length; i++) {            h = 31 * h + val[i];        }        hash = h;    }    return h; }
```

#  BigDecimal

Java在java.math包中提供的API类BigDecimal，用来对超过16位有效位的数进行精确的运算。双精度浮点型变量double可以处理16位有效数。在实际应用中，需要对更大或者更小的数进行运算和处理。float和double只能用来做科学计算或者是工程计算，在商业计算中要用java.math.BigDecimal。BigDecimal所创建的是对象，我们不能使用传统的+、-、*、/等算术运算符直接对其对象进行数学运算，而必须调用其相对应的方法。方法中的参数也必须是BigDecimal的对象。构造器是类的特殊方法，专门用来创建对象，特别是带有参数的对象。

## 构造器描述

 BigDecimal(int)    创建一个具有参数所指定整数值的对象。 
BigDecimal(double) 创建一个具有参数所指定双精度值的对象。 
BigDecimal(long)   创建一个具有参数所指定长整数值的对象。 
BigDecimal(String) 创建一个具有参数所指定以字符串表示的数值的对象。

## 方法描述 

add(BigDecimal)     BigDecimal对象中的值相加，然后返回这个对象。 
subtract(BigDecimal) BigDecimal对象中的值相减，然后返回这个对象。 
multiply(BigDecimal)  BigDecimal对象中的值相乘，然后返回这个对象。 
divide(BigDecimal)   BigDecimal对象中的值相除，然后返回这个对象。 
toString()         将BigDecimal对象的数值转换成字符串。 
doubleValue()      将BigDecimal对象中的值以双精度数返回。 
floatValue()       将BigDecimal对象中的值以单精度数返回。 
longValue()       将BigDecimal对象中的值以长整数返回。 
intValue()        将BigDecimal对象中的值以整数返回。

# NumberFormat



# String 比较大小：compareTo



- 如果参数字符串等于此字符串，则返回值 0；
- 如果此字符串小于字符串参数，则返回一个小于 0 的值；
- 如果此字符串大于字符串参数，则返回一个大于 0 的值。
- 注意：字符长度不在考虑范围内，他会从前到后比，如：

> ```
> String a = "05";
> String b = "10";
> String c = "5";
> System.out.println(a.compareTo(b));
> System.out.println(c.compareTo(b));
> ```

结果：

> -1
> 4
>
> Process finished with exit code 0



# Java 8 Optional 类

Optional 类是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

Optional 是个容器：它可以保存类型T的值，或者仅仅保存null。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

Optional 类的引入很好的解决空指针异常。

# Java 常用API

## append

Stringbuffer 有append()方法 
Stringbuffer其实是动态字符串数组 
append()是往动态字符串数组添加，跟“xxxx”+“yyyy”相当那个‘+’号 
跟String不同的是Stringbuffer是放一起的 
String1+String2 和Stringbuffer1.append("yyyy")虽然打印效果一样，但在内存中表示却不一样 
String1+String2 存在于不同的两个地址内存 
Stringbuffer1.append(Stringbuffer2)放再一起

# Function.identity() 自己传自己本身

> ```
> // 将Stream转换成容器或Map Stream<String> stream = Stream.of("I", "love", "you", "too"); Map<String, Integer> map = stream.collect(Collectors.toMap(Function.identity(), String::length)); 
> ```

Function是一个接口，那么Function.identity()是什么意思呢？解释如下：

Java 8允许在接口中加入具体方法。接口中的具体方法有两种，default方法和static方法，identity()就是Function接口的一个静态方法。
Function.identity()返回一个输出跟输入一样的Lambda表达式对象，等价于形如`t -> t`形式的Lambda表达式。

identity() 方法JDK源码如下：

> ```php
> static  Function identity() {
>     return t -> t;
> }
> ```



#  instanceof 

instanceof 是 Java 的保留关键字。它的作用是测试它左边的对象是否是它右边的类的实例，返回 boolean 的数据类型。

>  if (o instanceof Vector) 
>
> System.out.println("对象是 java.util.Vector 类的实例");
>
>  else if (o instanceof ArrayList) 
>
> System.out.println("对象是 java.util.ArrayList 类的实例"); 
>
> else 
>
> System.out.println("对象是 " + o.getClass() + " 类的实例");

# Java判空

## 集合类

ArrayUtils.isNotEmpty();
StringUtils.isNotBlank();
CollectionUtils.isNotEmpty(list)
MapUtil.isEmpty();





# JAVA 时间

## Java 8日期/时间类

Java 8的日期和时间类包含`LocalDate`、`LocalTime`、`Instant`、`Duration`以及`Period`，这些类都包含在`java.time`包中，下面我们看看这些类的用法。

### `LocalDate`和`LocalTime`

`LocalDate`类表示一个具体的日期，但不包含具体时间，也不包含时区信息。可以通过`LocalDate`的静态方法`of()`创建一个实例，`LocalDate`也包含一些方法用来获取年份，月份，天，星期几等：



> ```
> LocalDate localDate = LocalDate.of(2017, 1, 4); // 初始化一个日期：2017-01-04
> int year = localDate.getYear(); // 年份：2017
> Month month = localDate.getMonth(); // 月份：JANUARY
> int dayOfMonth = localDate.getDayOfMonth(); // 月份中的第几天：4
> DayOfWeek dayOfWeek = localDate.getDayOfWeek(); // 一周的第几天：WEDNESDAY
> int length = localDate.lengthOfMonth(); // 月份的天数：31
> boolean leapYear = localDate.isLeapYear(); // 是否为闰年：false
> ```



```
LocalDate now = LocalDate.now();
```

`LocalTime`和`LocalDate`类似，他们之间的区别在于`LocalDate`不包含具体时间，而`LocalTime`包含具体时间，例如：



> ```
> LocalTime localTime = LocalTime.of(17, 23, 52); // 初始化一个时间：17:23:52
> int hour = localTime.getHour(); // 时：17
> int minute = localTime.getMinute(); // 分：23
> int second = localTime.getSecond(); 
> ```



### `LocalDateTime`



`LocalDateTime`类是`LocalDate`和`LocalTime`的结合体，可以通过`of()`方法直接创建，也可以调用`LocalDate`的`atTime()`方法或`LocalTime`的`atDate()`方法将`LocalDate`或`LocalTime`合并成一个`LocalDateTime`：

### 加减操作



> ```
> LocalDate date = LocalDate.of(2017, 1, 5); // 2017-01-05
> 
> LocalDate date1 = date.withYear(2016); // 修改为 2016-01-05
> LocalDate date2 = date.withMonth(2); // 修改为 2017-02-05
> LocalDate date3 = date.withDayOfMonth(1); // 修改为 2017-01-01
> 
> LocalDate date4 = date.plusYears(1); // 增加一年 2018-01-05
> LocalDate date5 = date.minusMonths(2); // 减少两个月 2016-11-05
> LocalDate date6 = date.plus(5, ChronoUnit.DAYS); // 增加5天 2017-01-10
> ```

LocalDateTime 转 date

> ZoneId zoneId = ZoneId.systemDefault();
> LocalDateTime localDateTime = LocalDateTime.now();
> ZonedDateTime zdt = localDateTime.atZone(zoneId);
>
> 
>
> Date date = Date.from(zdt.toInstant());
>
> 
>
> System.out.println("LocalDateTime = " + localDateTime);
> System.out.println("Date = " + date);

# JVM相关

![com.atlassian.confluence.content.render.xhtml.XhtmlException: Missing required attribute: {http://atlassian.com/resource/identifier}value](http://km.vivo.xyz/plugins/servlet/confluence/placeholder/error?i18nKey=editor.placeholder.broken.image&locale=zh_CN&version=2)

## `jmap命令`

参数：

- **option：** 选项参数。
- **pid：** 需要打印配置信息的进程ID。
- **executable：** 产生核心dump的Java可执行文件。
- **core：** 需要打印配置信息的核心文件。
- **server-id** 可选的唯一id，如果相同的远程主机上运行了多台调试服务器，用此选项参数标识服务器。
- **remote server IP or hostname** 远程调试服务器的IP地址或主机名。

option

- **no option：** 查看进程的内存映像信息,类似 Solaris pmap 命令。
- **heap：** 显示Java堆详细信息
- **histo[:live]：** 显示堆中对象的统计信息
- **clstats：**打印类加载器信息
- **finalizerinfo：** 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
- **dump:<dump-options>：**生成堆转储快照
- **F：** 强制,当-dump或者-histo参数没有响应下使用. 在这个模式下,live子参数无效.
- **help：**打印帮助信息
- **J<flag>：**指定传递给运行jmap的JVM的参数



# 静态内部类

**一般程序开发人员可以这么理解，非静态的内部类对象隐式地在外部类中保存了一个引用，指向创建它的外部类对象**。

牢记两个差别：

一、如是否可以创建静态的成员方法与成员变量(静态内部类可以创建静态的成员，而非静态的内部类不可以)

二、对于访问外部类的成员的限制(静态内部类只可以访问外部类中的静态成员变量与成员方法，而非静态的内部类即可以访问所有的外部类成员方法与成员变量)。

这两个差异是静态内部类与非静态外部类最大的差异，也是静态内部类之所以存在的原因。了解了这个差异之后，程序开发人员还需要知道，在什么情况下该使用静态内部类。如在程序测试的时候，为了避免在各个Java源文件中书写主方法的代码，可以将主方法写入到静态内部类中，以减少代码的书写量，让代码更加的简洁。

总 之，静态内部类在Java语言中是一个很特殊的类，跟普通的静态类以及非静态的内部类都有很大的差异。作为程序开发人员，必须要知道他们之间的差异，并在 实际工作中在合适的地方采用合适的类。不过总的来说，静态内部类的使用频率并不是很高。但是在有一些场合，如果没有这个内部静态类的话，可能会起到事倍功半的反面效果。

```
//静态类
 
package common.lang;
 
public class Student {
 
    private String name;
    private int age;
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public int getAge() {
        return age;
    }
 
    public void setAge(int age) {
        this.age = age;
    }
 
    public class Child{
        private String name1;
        private int age1;
 
        public String getName1() {
            return name1;
        }
        public void setName1(String name1) {
            this.name1 = name1;
        }
        public int getAge1() {
            System.out.println(age);
            return age1;
        }
        public void setAge1(int age1) {
            this.age1 = age1;
        }
 
 
    }
 
    public static void main(String[] args) {
        Student s = new Student();
        s.setAge(12);
        s.setName("yao");
        Child c = s.new Child();
        System.out.println(c.getAge1());
    }
}
 
//静态内部类
 
package common.lang;
 
public class Student {
 
 
    private String name;
    private static int age;
 
    public String getName() {
        return name;
    }
 
    public void setName(String name) {
        this.name = name;
    }
    public static int getAge() {
        return age;
    }
 
    public static void setAge(int age) {
        Student.age = age;
    }
 
 
    public static class Child{
        private String name1;
        private int age1;
 
        public String getName1() {
            return name1;
        }
        public void setName1(String name1) {
            this.name1 = name1;
        }
        public int getAge1() {
            System.out.println(age);
            return age1;
        }
        public void setAge1(int age1) {
            this.age1 = age1;
        }
 
 
    }
 
    public static void main(String[] args) {
        Student s = new Student();
        Child c = new Child();
    }
}

```