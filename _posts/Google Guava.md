

------

# Google Guava http://ifeve.com/google-guava/

## 静态方法声明Collections

```
List<TypeThatsTooLongForItsOwnGood> list = ``new` `ArrayList<TypeThatsTooLongForItsOwnGood>();
```

可变为

List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();

Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();



直接元素初始化：

Set<Type> copySet = Sets.newHashSet(elements);

```
List<String> theseElements = Lists.newArrayList(``"alpha"``, ``"beta"``, ``"gamma"``);
List<Type> exactly100 = Lists.newArrayListWithCapacity(``100``);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(``100``);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(``100``);
Map<String, Integer> left = ImmutableMap.of(``"a"``, ``1``, ``"b"``, ``2``, ``"c"``, ``3``);
```

## Lists

[partition(List, int)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#partition(java.util.List, int)):把List按指定大小分割

[reverse(List)](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Lists.html#reverse(java.util.List)):返回给定List的反转视图。注: 如果List是不可变的，考虑改用[ImmutableList.reverse()](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/ImmutableList.html#reverse())。

```
List countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}List countUp = Ints.asList(``1``, ``2``, ``3``, ``4``, ``5``);
```



```
List<List> parts = Lists.partition(countUp, ``2``);``//{{1,2}, {3,4}, {5}}
```

## Sets

我们提供了很多标准的集合运算（Set-Theoretic）方法，这些方法接受Set参数并返回[SetView](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/Sets.SetView.html)，可用于：

union(Set, Set) --集合
intersection(Set, Set) --暂不知道
difference(Set, Set) --比较
symmetricDifference(Set, Set) --暂不知道



> cartesianProduct(List<Set>) --返回所有集合的笛卡儿积 
> powerSet(Set) --返回给定集合的所有子集
>
> Set<String> animals = ImmutableSet.of("gerbil", "hamster");
> Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");
> Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
> // {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
> // {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}
> Set<Set<String>> animalSets = Sets.powerSet(animals);
> // {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}

## Maps

### difference

Maps.difference(Map, Map)用来比较两个Map以获取所有不同点。该方法返回MapDifference对象，把不同点的维恩图分解为：

entriesInCommon() --两个Map中都有的映射项，包括匹配的键与值
entriesDiffering() --键相同但是值不同值映射项。返回的Map的值类型为MapDifference.ValueDifference，以表示左右两个不同的值
entriesOnlyOnLeft() --键只存在于左边Map的映射项
entriesOnlyOnRight() --键只存在于右边Map的映射项

> Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
> Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
> MapDifference<String, Integer> diff = Maps.difference(left, right);
> diff.entriesInCommon(); // {"b" => 2}
> diff.entriesInCommon(); // {"b" => 2}
> diff.entriesOnlyOnLeft(); // {"a" => 1}
> diff.entriesOnlyOnRight(); // {"d" => 5}

### ImmutableMap：

> - ImmutableMap, as suggested by the name, is a type of Map which is immutable. It means that the content of the map are fixed or constant after declaration, that is, they are **read-only**. 只读
> - If any attempt made to add, delete and update elements in the Map, **UnsupportedOperationException** is thrown. 任何操作都会报错
> - An ImmutableMap does not allow null element either. 不允许空
> - If any attempt made to create an ImmutableMap with null element, **NullPointerException** is thrown. If any attempt is made to add null element in Map, **UnsupportedOperationException** is thrown.
>
> **Advantages of ImmutableMap**
>
> - They are **thread safe. 现成安全**
> - They are **memory efficient. 内存有效？**
> - Since they are immutable, hence they can be **passed over to third party libraries** without any problem. 第三方有效

> 

# GUAVA Cache 缓存

## 使用场景

1. 愿意消耗一些内存空间来提升速度。
2. 预料到某些键会被多次查询。
3. 缓存中存放的数据总量不会超出内存容量。

*注*：如果你不需要Cache中的特性，使用ConcurrentHashMap有更好的内存效率——但Cache的大多数特性都很难基于旧有的ConcurrentMap复制，甚至根本不可能做到。

## 源码

```
// // Source code recreated from a .class file by IntelliJ IDEA // (powered by Fernflower decompiler) // @GwtCompatible public interface Cache<K, V> {    @Nullable    V getIfPresent(@CompatibleWith("K") Object var1);     V get(K var1, Callable<? extends V> var2) throws ExecutionException;     ImmutableMap<K, V> getAllPresent(Iterable<?> var1);     void put(K var1, V var2);     void putAll(Map<? extends K, ? extends V> var1);     void invalidate(@CompatibleWith("K") Object var1);     void invalidateAll(Iterable<?> var1);     void invalidateAll();     long size();     CacheStats stats();     ConcurrentMap<K, V> asMap();     void cleanUp(); }  
```



具体实现：

```
private static Cache<String, String> clientAndModule = CacheBuilder.newBuilder()        .expireAfterAccess(10, TimeUnit.MINUTES) // 设置缓存过期时间         .concurrencyLevel(40) // 设置并发级别为40        .recordStats() // 开启缓存统计        .build();
```

参数：

## CacheLoader

LoadingCache是附带CacheLoader构建而成的缓存实现。创建自己的CacheLoader通常只需要简单地实现V load(K key) throws Exception方法。例如，你可以用下面的代码构建LoadingCache：

```
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .build(
            new CacheLoader<Key, Graph>() {
                public Graph load(Key key) throws AnyException {
                    return createExpensiveGraph(key);
                }
            });
 
...
try {
    return graphs.get(key);
} catch (ExecutionException e) {
    throw new OtherException(e.getCause());
}
```

### `Callable`

所有类型的Guava Cache，不管有没有自动加载功能，都支持[get(K, Callable)](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/Cache.html#get(java.lang.Object,java.util.concurrent.Callable))方法。这个方法返回缓存中相应的值，或者用给定的Callable运算并把结果加入到缓存中。在整个加载方法完成前，缓存项相关的可观察状态都不会更改。这个方法简便地实现了模式"如果有缓存则返回；否则运算、缓存、然后返回"。

```
Cache<Key, Graph> cache = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .build(); // look Ma, no CacheLoader
...
try {
    // If the key wasn't in the "easy to compute" group, we need to
    // do things the hard way.
    cache.get(key, new Callable<Key, Graph>() {
        @Override
        public Value call() throws AnyException {
            return doThingsTheHardWay(key);
        }
    });
} catch (ExecutionException e) {
    throw new OtherException(e.getCause());
}
```

### 定时回收（Timed Eviction）

```
CacheBuilder``提供两种定时回收的方法：
```

> - [`expireAfterAccess(long, TimeUnit)`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/…eBuilder.html#expireAfterAccess(long, java.util.concurrent.TimeUnit))：缓存项在给定时间内没有被读/写访问，则回收。请注意这种缓存的回收顺序和基于大小回收一样。
> - [`expireAfterWrite(long, TimeUnit)`](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/…heBuilder.html#expireAfterWrite(long, java.util.concurrent.TimeUnit))：缓存项在给定时间内没有被写访问（创建或覆盖），则回收。如果认为缓存数据总是在固定时候后变得陈旧不可用，这种回收方式是可取的。

```

```

### 基于引用的回收（Reference-based Eviction）

通过使用弱引用的键、或弱引用的值、或软引用的值，Guava Cache可以把缓存设置为允许垃圾回收：

> - [`CacheBuilder.weakKeys()`](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilder.html#weakKeys())：使用弱引用存储键。当键没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用键的缓存用==而不是equals比较键。
> - [`CacheBuilder.weakValues()`](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilder.html#weakValues())：使用弱引用存储值。当值没有其它（强或软）引用时，缓存项可以被垃圾回收。因为垃圾回收仅依赖恒等式（==），使用弱引用值的缓存用==而不是equals比较值。
> - [`CacheBuilder.softValues()`](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilder.html#softValues())：使用软引用存储值。软引用只有在响应内存需要时，才按照全局最近最少使用的顺序回收。考虑到使用软引用的性能影响，我们通常建议使用更有性能预测性的缓存大小限定（见上文，基于容量回收）。使用软引用值的缓存同样用==而不是equals比较值。

### 显式清除

任何时候，你都可以显式地清除缓存项，而不是等到它被回收：

> - 个别清除：[`Cache.invalidate(key)`](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/Cache.html#invalidate(java.lang.Object))
> - 批量清除：`Cache.invalidateAll(keys)`

## Spring 配置Guava

```
@Configuration
@EnableCaching
public class GuavaCacheConfig {
 
    @Bean
    public CacheManager cacheManager() {
        GuavaCacheManager cacheManager = new GuavaCacheManager();
        cacheManager.setCacheBuilder(
                CacheBuilder.newBuilder().
                        expireAfterWrite(10, TimeUnit.SECONDS).
                        maximumSize(1000));
        return cacheManager;
    }
}
```



```
@Service
public class UserService {
 
    @Autowired
    private UserMapper userMapper;
 
    public List<User> getUsers() {
        return userMapper.getUsers();
    }
 
    public int addUser( User user ) {
        return userMapper.addUser(user);
    }
 
    @Cacheable(value = "user", key = "#userName")
    public List<User> getUsersByName( String userName ) {
        List<User> users = userMapper.getUsersByName( userName );
        System.out.println( "从数据库读取，而非读取缓存！" );
        return users;
    }
}
```

1. `@Cacheable`：配置在 `getUsersByName`方法上表示其返回值将被加入缓存。同时在查询时，会先从缓存中获取，若不存在才再发起对数据库的访问
2. `@CachePut`：配置于方法上时，能够根据参数定义条件来进行缓存，其与 `@Cacheable`不同的是使用 `@CachePut`标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中，所以主要用于数据新增和修改操作上
3. `@CacheEvict`：配置于方法上时，表示从缓存中移除相应数据。

```
@RestController
public class UserController {
 
    @Autowired
    private UserService userService;
 
    @Autowired
    CacheManager cacheManager;
 
    @RequestMapping( value = "/getusersbyname", method = RequestMethod.POST)
    public List<User> geUsersByName( @RequestBody User user ) {
        System.out.println( "-------------------------------------------" );
        System.out.println("call /getusersbyname");
        System.out.println(cacheManager.toString());
        List<User> users = userService.getUsersByName( user.getUserName() );
        return users;
    }
 
}
```