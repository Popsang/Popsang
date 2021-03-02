

# 字符编码

## utf8

varchar 1 = 3 字节

## utf8mb4

varchar 1 = 4 字节

# Alter

### MODIFY

```
MODIFY COLUMN rule_type tinyint(4) COMMENT '检测规则类型：0：恒值检测，1：突变检测， 2: 同比检测，3：阈值检测，4：无数据检测', modify    字段名     not     null;
```





### ADD



```
ADD `period` varchar(20) COMMENT '检测周期，目前支持:1s,1m,10m,1h' ,
```







# group by

GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

聚合函数包括：
求和函数——SUM()
计数函数——COUNT()
最大/最小值函数—MAX()/MIN()
均值函数——AVG()

对于group by来说一定要结合聚合函数，而且选择的字段除了聚合函数外，还必须在group by中出现，否则报错，

但是在mysql中扩展了这样的功能：
首先对于不加聚合函数的sql来说，它的功能结合了limit来得出结果，仔细想想的时候有点Oracle分析函数的味道，limit的时候得到的并不是 最大最小的值，而是某一下group by结果集中第一行，也就是刚才说的相当与先group by， 然后在每个group by下面进行limit 1。
其次，刚才还说了常规的group by结合聚合函数的时候，选择的字段除了聚合函数外，必须在group by中存在，但是在mysql中不是这样了，它具有隐含字段的功能。

# ON DUPLICATE KEY UPDATE

MySql避免重复插入记录方法。语句的作用：当insert已经存在的记录时，执行Update。判断是否重复的条件为主键或者UNIQUE（唯一索引）。

![胡硕 > MYSQL笔记 > image2019-9-16_15-1-7.png](http://km.vivo.xyz/download/attachments/102060307/image2019-9-16_15-1-7.png?version=1&modificationDate=1568617267000&api=v2)

上表中，两个部分——id主键以及shield_type和shield_obj的联合唯一索引。

> ```
> insert into moni_detect_rule_shield
> (<include refid="Base_Column_List"/>)
> values
> <foreach collection="list" item="rule" index="index" separator=",">
>  (#{rule.shieldObj},#{rule.shieldType},#{rule.beginShieldTime},#{rule.endShieldTime},#{rule.createBy},#{rule.createId},#{rule.createTime})
> </foreach>
> ON DUPLICATE KEY UPDATE
> begin_shield_time = VALUES(begin_shield_time),
> end_shield_time = VALUES(end_shield_time),
> create_by = VALUES(create_by),
> create_id = VALUES(create_id),
> create_time = VALUES(create_time)
> ```

所以，上面的SQL的含义为，当shield_type和shield_obj联合重复的情况下，insert修改为update，并更新begin_shield_time、end_shield_time、create_by、create_id、create_time。



# ignore

# Replace

```
UPDATE `table_name` SET `field_name` = replace (`field_name`,'from_str','to_str') WHERE `field_name` LIKE '%from_str%'
```

#  if

> SELECT * 
>
> FROM moni_detect_rule_shield
> WHERE
>
>  if(daily_begin_shield_time < daily_end_shield_time 
> , daily_begin_shield_time < '22:53:58' and daily_end_shield_time > '22:53:58'
> , daily_begin_shield_time > '22:53:58' AND daily_end_shield_time < '22:53:58')

WHERE  IF(条件,  true执行条件, false执行条件 )

# left（str, length） 

说明：left（被截取字段，截取长度） 
例：`select left(content,200) as abstract from my_content_t ` 

# right（str, length） 


说明：right（被截取字段，截取长度） 
例：select right(content,200) as abstract from my_content_t 

# substring（str, pos）  substring（str, pos, length） 

说明：

substring（被截取字段，从第几位开始截取） 
substring（被截取字段，从第几位开始截取，截取长度） 

例：

```
select substring(content,5) as abstract from my_content_t 
select substring(content,5,200) as abstract from my_content_t 
```

**（注：如果位数是负数 如-5 则是从后倒数位数，到字符串结束或截取的长度）** 

# substring_index（str,delim,count） 

说明：substring_index（被截取字段，关键字，关键字出现的次数） 

例：select substring_index("www.w3cschool.cn",".",2) as abstract from wiki_user 

结果：www.w3cschool
**（注：如果关键字出现的次数是负数 如-2 则是从后倒数，到字符串结束）** 

函数简介：

SUBSTRING(str,pos) , SUBSTRING(str FROM pos) SUBSTRING(str,pos,len) , SUBSTRING(str FROM pos FOR len)

不带有len 参数的格式从字符串str返回一个子字符串，起始于位置 pos。带有len参数的格式从字符串str返回一个长度同len字符相同的子字符串，起始于位置 pos。 使用 FROM的格式为标准 SQL 语法。也可能对pos使用一个负值。假若这样，则子字符串的位置起始于字符串结尾的pos 字符，而不是字符串的开头位置。在以下格式的函数中可以对pos 使用一个负值。

# 增加注释

> ```
> ALTER TABLE moni_detect_rule_shield ADD daily_end_shield_time  VARCHAR(10) comment '每日屏蔽结束时间';
> ```

# 常用指令

- show databases；查看所有数据库；
- show tables; 查看所有表
- use ‘数据库名’;使用指定数据库
- show columns from ‘表名’；查看表字段名
- mysql -h ‘IP’ -P‘端口’ -u ‘用户名’ -p‘密码’；登录

# 数据库表数据迁移

- 如果两张张表（导出表和目标表）的字段一致，并且希望插入全部数据，可以用这种方法：（此方法只适合导出两表在同一database）

INSERT INTO 目标表 SELECT * FROM 来源表;
例如，要将 articles 表插入到 newArticles 表中，则可以通过如下SQL语句实现：
INSERT INTO newArticles SELECT * FROM articles;

- 如果只希望导入指定字段，可以用这种方法：

 INSERT INTO 目标表 (字段1, 字段2, ...) SELECT 字段1, 字段2, ... FROM 来源表;
请注意以上两表的字段必须一致（字段类型），否则会出现数据转换错误。

# 索引

建立索引的条件：

- 自动建立唯一索引
- **表的字段唯一约束**
- **直接条件查询的字段**
- **查询中与其它表关联的字段**
- **查询中排序的字段**
- **查询中统计或分组统计的字段**

不应该建立索引的情况：

- **表记录太少**
- **经常插入、删除、修改的表**
- **数据重复且分布平均的表字段**
- **经常和主字段一块查询但主字段索引值比较多的表字段**

# 全文索引

1. MySQL 5.6 以前的版本，只有 MyISAM 存储引擎支持全文索引；
2. MySQL 5.6 及以后的版本，MyISAM 和 InnoDB 存储引擎均支持全文索引;
3. 只有字段的数据类型为 char、varchar、text 及其系列才可以建全文索引



https://zhuanlan.zhihu.com/p/35675553

```
create table fulltext_test (    id int(11) NOT NULL AUTO_INCREMENT,    content text NOT NULL,    tag varchar(255),    PRIMARY KEY (id),    FULLTEXT KEY content_tag_fulltext(content,tag)  // 创建联合全文索引列 ) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```



和常用的模糊匹配使用 like + % 不同，全文索引有自己的语法格式，使用 match 和 against 关键字

```
select * from fulltext_test     where match(content,tag) against('xxx xxx');
```

注意： match() 函数中指定的列必须和全文索引中指定的列完全相同，否则就会报错，无法使用全文索引，这是因为全文索引不会记录关键字来自哪一列。如果想要对某一列使用全文索引，请单独为该列创建全文索引。

# 模糊匹配

后模糊匹配可以用索引 like (‘abc%’)



# join on

## left join

mysql left join 语句格式

A LEFT JOIN B ON 条件表达式

left join 是以A表为基础，A表即左表，B表即右表。

左表(A)的记录会全部显示，而右表(B)只会显示符合条件表达式的记录，如果在右表(B)中没有符合条件的记录，则记录不足的地方为NULL。



member 表

| id   | username |
| ---- | -------- |
| 1    | fdipzone |
| 2    | terry    |



member_login_log 表



| id   | uid  | logindate  |
| ---- | ---- | ---------- |
| 1    | 1    | 2015-01-01 |
| 2    | 2    | 2015-01-0  |
| 3    | 1    | 2015-01-02 |
| 4    | 2    | 2015-01-02 |



1:n情况

但如果B表符合条件的记录数大于1条，就会出现1:n的情况，这样left join后的结果，记录数会多于A表的记录数。如下：



| id   | username | logindate  |
| ---- | -------- | ---------- |
| 1    | fdipzone | 2015-01-01 |
| 1    | fdipzone | 2015-01-02 |
| 2    | terry    | 2015-01-01 |
| 2    | terry    | 2015-01-02 |
| 2    | terry    | 2015-01-03 |