---
title: mysql explain
date: 2021-1-6 16:59:11
tags: [Sql,Mysql]
categories: sql
description: mysql explain的使用，索引应该怎么建立
---

参考：

https://www.cnblogs.com/tufujie/p/9413852.html   Explain详解

https://www.cnblogs.com/duanxz/p/5244737.html   多列索引



## Explain详解

可以使用explain这个命令来查看一个这些SQL语句的执行计划，查看该SQL语句有没有使用上了索引，有没有做全表扫描

expain出来的信息有12列，分别是

> id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra

**概要描述：**
id:选择标识符
select_type:表示查询的类型。
table:输出结果集的表
partitions:匹配的分区
type:表示表的连接类型
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明

### 一、 **id**

SELECT识别符。这是SELECT的查询序列号

**我的理解是SQL执行的顺序的标识，SQL从大到小的执行**

1. id相同时，执行顺序由上至下

2. 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行

3. id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

```
-- 查看在研发部并且名字以Jef开头的员工，经典查询
explain select e.no, e.name from emp e left join dept d on e.dept_no = d.no where e.name like 'Jef%' and d.name = '研发部';
```

![](https://gitee.com/wowoa/typoraPic/raw/master/image/2021/01/20210108165444.png)

### **二、select_type**

   **示查询中每个select子句的类型**

(1) SIMPLE(简单SELECT，不使用UNION或子查询等)

(2) PRIMARY(子查询中最外层查询，查询中若包含任何复杂的子部分，最外层的select被标记为PRIMARY)

(3) UNION(UNION中的第二个或后面的SELECT语句)

(4) DEPENDENT UNION(UNION中的第二个或后面的SELECT语句，取决于外面的查询)

(5) UNION RESULT(UNION的结果，union语句中第二个select开始后面所有select)

(6) SUBQUERY(子查询中的第一个SELECT，结果不依赖于外部查询)

(7) DEPENDENT SUBQUERY(子查询中的第一个SELECT，依赖于外部查询)

(8) DERIVED(派生表的SELECT, FROM子句的子查询)

(9) UNCACHEABLE SUBQUERY(一个子查询的结果不能被缓存，必须重新评估外链接的第一行)

 

### **三、table**

显示这一步所访问数据库中表名称（显示这一行的数据是关于哪张表的），有时不是真实的表名字，可能是简称，例如上面的e，d，也可能是第几步执行的结果的简称



### **四、type**

对表访问方式，表示MySQL在表中找到所需行的方式，又称“访问类型”。

常用的类型有： **ALL、index、range、 ref、eq_ref、const、system、****NULL（从左到右，性能从差到好）**

ALL：Full Table Scan， MySQL将遍历全表以找到匹配的行

index: Full Index Scan，index与ALL区别为index类型只遍历索引树

range:只检索给定范围的行，使用一个索引来选择行

ref: 表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

eq_ref: 类似ref，区别就在使用的索引是唯一索引，对于每个索引键值，表中只有一条记录匹配，简单来说，就是多表连接中使用primary key或者 unique key作为关联条件

const、system: 当MySQL对查询某部分进行优化，并转换为一个常量时，使用这些类型访问。如将主键置于where列表中，MySQL就能将该查询转换为一个常量，system是const类型的特例，当查询的表只有一行的情况下，使用system

NULL: MySQL在优化过程中分解语句，执行时甚至不用访问表或索引，例如从一个索引列里选取最小值可以通过单独索引查找完成。

 

### **五、possible_keys**

**指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用（该查询可以利用的索引，如果没有任何索引显示 null）**

该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。
如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查WHERE子句看是否它引用某些列或适合索引的列来提高你的查询性能。如果是这样，创造一个适当的索引并且再次用EXPLAIN检查查询

 

### **六、Key**

**key列显示MySQL实际决定使用的键（索引），必然包含在possible_keys中**

如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

 

### **七、key_len**

**表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度（key_len显示的值为索引字段的最大可能长度，并非实际使用长度，即key_len是根据表定义计算而得，不是通过表内检索出的）**

不损失精确性的情况下，长度越短越好 

 

### **八、ref**

**列与索引的比较，表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值**

 

### **九、rows**

 **估算出结果集行数，表示MySQL根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数**

 

### **十、Extra**

**该列包含MySQL解决查询的详细信息,有以下几种情况：**

Using where:不用读取表中所有信息，仅通过索引就可以获取所需数据，这发生在对表的全部的请求列都是同一个索引的部分的时候，表示mysql服务器将在存储引擎检索行后再进行过滤

Using temporary：表示MySQL需要使用临时表来存储结果集，常见于排序和分组查询，常见 group by ; order by

Using filesort：当Query中包含 order by 操作，而且无法利用索引完成的排序操作称为“文件排序”

```
-- 测试Extra的filesort
explain select * from emp order by name;
```

Using join buffer：改值强调了在获取连接条件时没有使用索引，并且需要连接缓冲区来存储中间结果。如果出现了这个值，那应该注意，根据查询的具体情况可能需要添加索引来改进能。

Impossible where：这个值强调了where语句会导致没有符合条件的行（通过收集统计信息不可能存在结果）。

Select tables optimized away：这个值意味着仅通过使用索引，优化器可能仅从聚合函数结果中返回一行

No tables used：Query语句中使用from dual 或不含任何from子句

```
-- explain select now() from dual;
```

 

**总结：**

- EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
- EXPLAIN不考虑各种Cache
- EXPLAIN不能显示MySQL在执行查询时所作的优化工作
- 部分统计信息是估算的，并非精确值
- EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划。

通过收集统计信息不可能存在结果



## 索引设计

| sql                                  | note           | id   | select_type | table | partitions | type  | possible_keys                   | key          | key_len | ref         | rows | filtered | Extra                             |
| ------------------------------------ | -------------- | ---- | ----------- | ----- | ---------- | ----- | ------------------------------- | ------------ | ------- | ----------- | ---- | -------- | --------------------------------- |
| id=1                                 | 主键           | 1    | SIMPLE      | demo  |            | const | PRIMARY                         | PRIMARY      | 8       | const       | 1    | 100      | Directly search via Primary Index |
| user_id=111111                       | 没有索引       | 1    | SIMPLE      | demo  |            | ALL   |                                 |              |         |             | 3    | 33.33    | Using where                       |
| status=1                             | 普通索引       | 1    | SIMPLE      | demo  |            | ref   | index_status                    | index_status | 1       | const       | 2    | 100      |                                   |
| status=1 and user_id=111111          | 普通+多列      | 1    | SIMPLE      | demo  |            | ref   | index_demo,index_status         | index_demo   | 9       | const,const | 1    | 100      |                                   |
| status=1 and user_id=111111          | 普通索引       | 1    | SIMPLE      | demo  |            | ref   | index_status                    | index_status | 1       | const       | 2    | 33.33    | Using where                       |
| id=1 amd status=1 and user_id=111111 | 主键+普通+多列 | 1    | SIMPLE      | demo  |            | const | PRIMARY,index_demo,index_status | PRIMARY      | 8       | const       | 1    | 100      |                                   |

### 索引的三星原则

1. 索引将相关的记录放到一起，则获得一星

2. 如果索引中的数据顺序和查找中的排列顺序一致则获得二星

3. 如果索引中的列包含了查询中的需要的全部列则获得三星



### 多个单列索引

为每个列创建独立的索引，从SHOW CREATE TABLE 中很容易看到这种情况：

```sql
CREATE TABLE t(
 　c1 INT,c2 INT , c3 INT ,KEY(c1),KEY(c2),KEY(c3)
);
```

**这样一来最好的情况也只能是“一星”索引，其性能比起真正最有效的索引可能差几个数量级。有时如果无法设计出一个“三星”索引，那么不如忽略掉WHERE 子句，集中精力优化索引列的顺序，或者创建一个全覆盖索引。**

### **索引合并**

　在多个列上建立独立的单列索引大部分情况下不能提高MySQL的查询性能。MySQL5.0和更高的版本医用了一种叫“**索引合并**”策略，一定程度上可以使用表上的多个单列索引来定位指定的行。更早版本的MySQL只能使用其中某一个单列索引，然而这种情况下没有哪一个独立索引是非常有效的。例如在film_actor在字段film_id和actor_id上各有一个单列索引。但是对于这个查询WHERE 条件，这两个单列索引都不是好的选择：

```
SELECT film_id ,actor_id FROM film_actor WHERE actor_id=1 or film_id =1;
```

　　在老的MySQL版本中，MySQL对于这个查询是会使用全表扫描的，除非改写成如下的两个查询UNION的方式：

```
SELECT film_id ,actor_id FROM film_actor WHERE actor_id=1
　　UNION ALL
　　SELECT film_id ,actor_id FROM film_actor WHERE film_id=1；
```

但是在MySQL5.0 和更高的版本中，查询能够同时使用者两个单列索引进行扫扫描，并将结果进行合并。这种算法有三个变种：OR条件的联合（union），AND条件的相交（intersection），组合前面两种情况的联合及相交。下面的查询就是使用了两个索引扫描的联合，通过EXPLAIN中的Extra列可以看出这点：

```
EXPLAIN SELECT film_id,actor_id FROM film_actor WHERE actor_id=1 or film_id = 1 
```



### 多列索引

```
ALTER TABLE people ADD INDEX fname_lname_age (firstname,lastname,age)； 
```

由于索引文件以B-树格式保存，MySQL能够立即转到合适的firstname，然后再转到合适的lastname，最后转到合适的age。在没有扫描数据文件任何一个记录的情况下，MySQL就正确地找出了搜索的目标记录！

那么，如果在firstname、lastname、age这三个列上分别创建单列索引，效果是否和创建一个firstname、lastname、age的多列MySQL数据库索引一样呢？答案是否定的，两者完全不同。**当我们执行查询的时候，MySQL只能使用一个索引。如果你有三个单列的MySQL数据库索引，MySQL会试图选择一个限制最严格的索引。**

但是，即使是限制最严格的单列索引，它的限制能力也肯定远远低于firstname、lastname、age这三个列上的多列索引。

**Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。例如索引是key index （a,b,c）。 可以支持a | a,b| a,b,c 3种组合进行查找，但不支持 b,c进行查找 .当最左侧字段是常量引用时，索引就十分有效。两个或更多个列上的索引被称作复合索引。**



### 前缀索引

有一种与索引选择性有关的索引优化策略叫做**前缀索引**，就是用列的前缀代替整个列作为索引key，当前缀长度合适时，可以做到既使得前缀索引的选择性接近全列索引，同时因为索引key变短而减少了索引文件的大小和维护开销。

如果频繁按名字搜索员工，这样显然效率很低，因此我们可以考虑建索引。有两种选择，建<first_name>或<first_name, last_name>

使用公式查看选择性

```
SELECT count(DISTINCT(first_name))/count(*) AS Selectivity FROM employees.employees;
```

<first_name>显然选择性太低，<first_name, last_name>选择性很好，但是first_name和last_name加起来长度为30，有没有兼顾长度和选择性的办法？可以考虑用first_name和last_name的前几个字符建立索引，例如<first_name, left(last_name, 3)>

前缀索引兼顾索引大小和查询速度，但是其缺点是不能用于ORDER BY和GROUP BY操作，也不能用于Covering index（即当索引本身包含查询所需全部数据时，不再访问数据文件本身）。



