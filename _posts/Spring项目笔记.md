Spring项目笔记

------

# spring.xml解析

如图：下图为spring的标准配置

![胡硕 > Spring项目笔记 > image2019-8-14_16-0-47.png](http://km.vivo.xyz/download/attachments/94908774/image2019-8-14_16-0-47.png?version=1&modificationDate=1565769647000&api=v2)

> ```
> <context:component-scan base-package="com.vivo.monitor.opentsdb"/>
> ```

这句话是说spring扫描的包是"com.vivo.monitor.opentsdb"。

> ```
> <context:annotation-config/>
> ```

这句话是说，配置了spring注解配置功能。

> ```
> <task:annotation-driven/>
> ```

启动task注解任务（命名空间也要进行修改）。



# @Scheduled注解

该注解是spring的定时任务功能。

首先，要开启task注解功能。

然后，写一段代码实现，在代码上面加上 @Scheduled(cron="0/5 * * * * ? ")。注：spring的@Scheduled注解需要写在实现上、定时器的任务方法不能有返回值、实现类上要有组件的注解@Component。

最后，编写cron表达式，详见[cron表达式](http://km.vivo.xyz/pages/viewpage.action?pageId=94909208)。写代码。

注意：

推荐配置线程池来使用task任务。

配置方法：spring.xml中增加

<task:scheduler id="myScheduler" pool-size="5"/>

这样写的好处：

在设置多个定时任务时，不使用线程池，会进行阻塞执行，也就是一个定时任务执行完后，在进行下一个定时任务执行。

# spring线程池

> ```
> <bean id="poolExecutor"
>  class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
>  <property name="corePoolSize" value="${pool.core.size}" />
>  <property name="maxPoolSize" value="${pool.max.size}" />
>  <property name="queueCapacity" value="${pool.queue.size}" />
>  <property name="keepAliveSeconds" value="${pool.keepAlive.seconds}" />
>  <property name="rejectedExecutionHandler">
>  <bean class="java.util.concurrent.ThreadPoolExecutor$AbortPolicy" />
>  </property>
> </bean>
> ```

如上段代码所示，配置对应参数。其中，后面的value值是spring从properties中读出的参数。对应各参数含义为：

- corePoolSize：线程池维护线程的最少数量
- keepAliveSeconds：允许的空闲时间
- maxPoolSize：线程池维护线程的最大数量
- queueCapacity：缓存队列
- rejectedExecutionHandler：对拒绝task的处理策略

执行方法：

- 如果此时线程池中的数量**小于**corePoolSize，即使线程池中的线程都处于空闲状态，也要**创建新**的线程来处理被添加的任务。
- 如果此时线程池中的数量**等于** corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。
- 如果此时线程池中的数量**大于**corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maxPoolSize，建新的线程来处理被添加的任务。
- 如果此时线程池中的数量**大于**corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maxPoolSize，那么通过handler所指定的策略来处理此任务。也就是：处理任务的优先级为：核心线程corePoolSize、任务队列workQueue、最大线程 maximumPoolSize，如果三者都满了，使用handler处理被拒绝的任务。
- 当线程池中的线程数量**大于**corePoolSize时，如果某线程空闲时间超过keepAliveTime，线程将被终止。这样，线程池可以动态的调整池中的线程数。

拒绝策略：

1. ThreadPoolExecutor.AbortPolicy()，抛出java.util.concurrent.RejectedExecutionException异常 。
2. ThreadPoolExecutor.CallerRunsPolicy()，用于被拒绝任务的处理程序，它直接在 execute 方法的调用线程中运行被拒绝的任务；如果执行程序已关闭，则会丢弃该任务。
3. ThreadPoolExecutor.DiscardOldestPolicy() 当任务呗拒绝添加时，会抛弃任务队列中最旧的任务也就是最先加入队列的，再把这个新任务添加进去。
4. ThreadPoolExecutor.DiscardPolicy() 这个东西什么都没干。因此采用这个拒绝策略，会让被线程池拒绝的任务直接抛弃，不会抛异常也不会执行。

#  @component

（把普通pojo实例化到spring容器中，相当于配置文件中的 <bean id="" class=""/>）。泛指各种组件，就是说当我们的类不属于各种归类的时候（不属于@Controller、@Services等的时候），我们就可以使用@Component来标注这个类。

# java中读取properties文件

## 通过context:property-placeholder加载配置文件jdbc.properties中的内容

> ```javascript
> <context:property-placeholder location="classpath:jdbc.properties" ignore-unresolvable="true"/>
> ```



## 自定义工具类PropertyUtil，并在该类的static静态代码块中读取properties文件内容保存在static属性中以供别的程序使用

![胡硕 > Spring项目笔记 > image2019-8-15_10-26-3.png](http://km.vivo.xyz/download/attachments/94908774/image2019-8-15_10-26-3.png?version=1&modificationDate=1565835964000&api=v2)



# PageHelper

PageHelper是一款好用的开源免费的Mybatis第三方物理分页插件

什么时候会导致不安全的分页？
PageHelper 方法使用了静态的 `ThreadLocal` 参数，分页参数和线程是绑定的。
只要你可以保证在 PageHelper 方法调用后紧跟 MyBatis 查询方法，这就是安全的。因为 PageHelper 在 `finally` 代码段中自动清除了 `ThreadLocal`存储的对象。

如果代码在进入 `Executor` 前发生异常，就会导致线程不可用，这属于人为的 Bug（例如接口方法和 XML 中的不匹配，导致找不到 `MappedStatement` 时）， 这种情况由于线程不可用，也不会导致 ThreadLocal 参数被错误的使用。

但是如果你写出下面这样的代码，就是不安全的用法：

> ```
> PageHelper.startPage(1, 10); List<Country> list; if(param1 != null){    list = countryMapper.selectIf(param1); } else {    list = new ArrayList<Country>(); } 
> ```

这种情况下由于 `param1` 存在`null`的情况，就会导致 PageHelper 生产了一个分页参数，但是没有被消费，这个参数就会一直保留在这个线程上。当这个线程再次被使用时，就可能导致不该分页的方法去消费这个分页参数，这就产生了莫名其妙的分页。
上面这个代码，应该写成下面这个样子：

> ```
> List<Country> list; if(param1 != null){    PageHelper.startPage(1, 10);    list = countryMapper.selectIf(param1); } else {    list = new ArrayList<Country>(); } 
> ```

这种写法就能保证安全。

# json解析问题　



Spring Boot支持与三种JSON mapping库集成：Gson、Jackson和JSON-B。Jackson是首选和默认的。



Jackson是spring-boot-starter-json依赖中的一部分，spring-boot-starter-web中包含spring-boot-starter-json。也就是说，当项目中引入spring-boot-starter-web后会自动引入spring-boot-starter-json。

所以，springboot默认的json转换方式是Jackson方式。

封装参数方式应为：

```
ObjectMapper mapper = new ObjectMapper();
String str = null;
try {
    str = mapper.writeValueAsString(object);
} catch (JsonProcessingException e) {
    e.printStackTrace();
}
```

如果使用fastjson的方式，可能会导致解析失败：

JSONObject.toJSONString(Javabean对象)

## fastjson转换

```
//fastjson String str1=JSON.toJSONString(new Person("xinghang",666));// 对象转JSON字符串 Person p1=JSONObject.parseObject(str1, Person.class);//JSON字符串转对象 String str=JSON.toJSONString(list1);//list转json字符串 List<Person> person = JSON.parseArray(str, Person.class); //json字符串转list  JSONObject.parseObject(task.getHeaders(), new TypeReference<Map<String, String>>() {}) //指定json字符串
```

## jackson

```
             //json串转对象 User user = JsonMapper.jsonToObject(json, new TypeReference<User>(){}); User user = JsonMapper.jsonToObject(json, User.class);  //对象转json String json = ObjectMapper .writeValueAsString(user);  //List转json List<User> list = Lists.newArrayList(); String json = JsonMapper.objectToJson(list);  //json转List List<User> list = JsonMapper.jsonToObject(json, new TypeReference<List<User>>() {}); List<User> list = JsonMapper.jsonToObject(json, List.class,User.class);     // json中的对象个数比java对象的属性个数少            JSONObject json1 = new JSONObject();            json1.put("name", "anne");            json1.put("age", 15);            String d1 = json1.toString();            Student s1 = mapper.readValue(d1, Student.class);            System.out.println(s1.toString());             // json中出现java对象中没有的属性            JSONObject json2 = new JSONObject();            json2.put("name", "anne");            json2.put("age", 15);            json2.put("sex", "boy");            String d2 = json1.toString();            Student s2 = mapper.readValue(d2, Student.class);            System.out.println(s2.toString());             /** java对象转换为json字符串 */            Student s3 = new Student();            s3.setAge(12);            s3.setHobby("sport");            s3.setName("anne");            String d3 = mapper.writeValueAsString(s3);            System.out.println(d3);
```



#  /error

SpringBoot在页面 发生异常的时候会自动把请求转到/error，SpringBoot内置了一个BasicErrorController对异常进行统一的处理。

https://juejin.im/post/5b2101716fb9a01e80785be2



# Spring 拦截器——HandlerInterceptor

（1）preHandle (HttpServletRequest request, HttpServletResponse response, Object handle) 方法，顾名思义，该方法将在请求处理之前进行调用。SpringMVC 中的Interceptor 是链式的调用的，在一个应用中或者说是在一个请求中可以同时存在多个Interceptor 。每个Interceptor 的调用会依据它的声明顺序依次执行，而且最先执行的都是Interceptor 中的preHandle 方法，所以可以在这个方法中进行一些前置初始化操作或者是对当前请求的一个预处理，也可以在这个方法中进行一些判断来决定请求是否要继续进行下去。该方法的返回值是布尔值Boolean 类型的，当它返回为false 时，表示请求结束，后续的Interceptor 和Controller 都不会再执行；当返回值为true 时就会继续调用下一个Interceptor 的preHandle 方法，如果已经是最后一个Interceptor 的时候就会是调用当前请求的Controller 方法。

（2）postHandle (HttpServletRequest request, HttpServletResponse response, Object handle, ModelAndView modelAndView) 方法，由preHandle 方法的解释我们知道这个方法包括后面要说到的afterCompletion 方法都只能是在当前所属的Interceptor 的preHandle 方法的返回值为true 时才能被调用。postHandle 方法，顾名思义就是在当前请求进行处理之后，也就是Controller 方法调用之后执行，但是它会在DispatcherServlet 进行视图返回渲染之前被调用，所以我们可以在这个方法中对Controller 处理之后的ModelAndView 对象进行操作。postHandle 方法被调用的方向跟preHandle 是相反的，也就是说先声明的Interceptor 的postHandle 方法反而会后执行。

（3）afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handle, Exception ex) 方法，该方法也是需要当前对应的Interceptor 的preHandle 方法的返回值为true 时才会执行。顾名思义，该方法将在整个请求结束之后，也就是在DispatcherServlet 渲染了对应的视图之后执行。这个方法的主要作用是用于进行资源清理工作的。