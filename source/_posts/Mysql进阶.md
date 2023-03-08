---
title: Mysql进阶
date: 2022-08-30 17:34:47
tags: 数据库
category: 学习笔记-计网/计组/数据库
cover: https://images.pexels.com/photos/12185637/pexels-photo-12185637.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=2
---

参考教学视频：[黑马程序员 MySQL数据库入门到精通](https://www.bilibili.com/video/BV1Kr4y1i7ru?p=148&spm_id_from=pageDriver&vd_source=2ac60144f27a69b3bdd68d9716b5c9cd)

# MySQL进阶

## 一、存储引擎

### 1) MySQL体系结构

![](https://s1.ax1x.com/2022/08/30/vh7v1U.md.png)

[![vhHiA1.png](https://s1.ax1x.com/2022/08/30/vhHiA1.png)](https://imgse.com/i/vhHiA1)

> 存储引擎就是存储数据、建立索引、更新/查询数据等技术的实现方式。存储引擎是基于表而不是基于库的，所以存储引擎也可以被称为表引擎。
> 默认存储引擎是**InnoDB**。

相关操作：

```sql
-- 查询建表语句
show create table account;
-- 建表时指定存储引擎
CREATE TABLE 表名(    ...) ENGINE=INNODB;
-- 查看当前数据库支持的存储引擎
show engines;
```

### 2) InnoDB

InnoDB 是一种兼顾**高可靠性和高性能的通用存储引擎**，在 MySQL 5.5 之后，InnoDB 是默认的 MySQL 引擎。

特点：

- DML（增删改） 操作遵循 ACID 模型，支持**事务**
- **行级锁**，提高并发访问性能
- 支持**外键**约束，保证数据的完整性和正确性

文件：

- xxx.ibd: xxx代表表名，InnoDB 引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm、sdi）、数据和索引。

参数：innodb_file_per_table，决定多张表共享一个表空间还是每张表对应一个表空间

查看 Mysql 变量：
`show variables like 'innodb_file_per_table';`

如果是on代表开启，也就是每一张表对应一个idb文件

**InnoDB 逻辑存储结构：**

[![vhHUBj.png](https://s1.ax1x.com/2022/08/30/vhHUBj.png)](https://imgse.com/i/vhHUBj)

### 3) MyISAM

MyISAM 是 MySQL 早期的默认存储引擎。

特点：

- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

文件：

- xxx.sdi: 存储表结构信息
- xxx.MYD: 存储数据
- xxx.MYI: 存储索引

### 4) Memory

Memory 引擎的表数据是存储在内存中的，受硬件问题、断电问题的影响，只能将这些表作为临时表或缓存使用。

特点：

- 存放在内存中，速度快
- hash索引（默认）

文件：

- xxx.sdi: 存储表结构信息

### 5) 存储引擎特点

| 特点         | InnoDB              | MyISAM | Memory |
| ------------ | ------------------- | ------ | ------ |
| 存储限制     | 64TB                | 有     | 有     |
| 事务安全     | 支持                | -      | -      |
| 锁机制       | 行锁                | 表锁   | 表锁   |
| B+tree索引   | 支持                | 支持   | 支持   |
| Hash索引     | -                   | -      | 支持   |
| 全文索引     | 支持（5.6版本之后） | 支持   | -      |
| 空间使用     | 高                  | 低     | N/A    |
| 内存使用     | 高                  | 低     | 中等   |
| 批量插入速度 | 低                  | 高     | 高     |
| 支持外键     | 支持                | -      | -      |

### 6) 存储引擎的选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

- InnoDB: 如果应用对事物的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，则 InnoDB 是比较合适的选择
- MyISAM: 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不高，那这个存储引擎是非常合适的。
- Memory: 将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。Memory 的缺陷是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性

电商中的足迹和评论适合使用 MyISAM 引擎，缓存适合使用 Memory 引擎。

## 二、性能分析

### 1) 查看执行频次

查看当前数据库的 INSERT, UPDATE, DELETE, SELECT 访问频次：
`SHOW GLOBAL STATUS LIKE 'Com_______';` 或者 `SHOW SESSION STATUS LIKE 'Com_______';`

### 2) 慢查询日志

慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志。
MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：

```sql
# 开启慢查询日志开关
slow_query_log=1
# 设置慢查询日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
```

更改后记得重启MySQL服务，日志文件位置：/var/lib/mysql/localhost-slow.log 腾讯云在（/www/server/data/mysql-slow.log）

查看慢查询日志开关状态：
`show variables like 'slow_query_log';`

### 3) profile

show profile 能在做SQL优化时帮我们了解时间都耗费在哪里。通过 have_profiling 参数，能看到当前 MySQL 是否支持 profile 操作：
`SELECT @@have_profiling;`
profiling 默认关闭，可以通过set语句在session/global级别开启 profiling：
`SET profiling = 1;`
查看所有语句的耗时：
`show profiles;`
查看指定query_id的SQL语句各个阶段的耗时：
`show profile for query query_id;`
查看指定query_id的SQL语句CPU的使用情况
`show profile cpu for query query_id;`

### 3) explain

EXPLAIN 或者 DESC 命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序。
语法：

```sql
# 直接在select语句之前加上关键字 
explain / descEXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件;
```

EXPLAIN 各字段含义：

- id：select 查询的序列号，表示查询中执行 select 子句或者操作表的顺序（id相同，执行顺序从上到下；id不同，值越大越先执行）
- select_type：表示 SELECT 的类型，常见取值有  SIMPLE（简单表，即不适用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION中的第二个或者后面的查询语句）、SUBQUERY（SELECT/WHERE之后包含了子查询）等
- type：表示连接类型，性能由好到差的连接类型为 NULL、system、const、eq_ref、ref、range、index、all (Null一般不用)
- possible_key：可能应用在这张表上的索引，一个或多个
- Key：实际使用的索引，如果为 NULL，则没有使用索引
- Key_len：表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好
- rows：MySQL认为必须要执行的行数，在InnoDB引擎的表中，是一个估计值，可能并不总是准确的
- filtered：表示返回结果的行数占需读取行数的百分比，filtered的值越大越好

## 三、索引

索引是帮助 MySQL **高效获取数据**的**数据结构（有序）**。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查询算法，这种数据结构就是索引。

优缺点：

优点：

- 提高数据检索效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗

缺点：

- 索引列也是要占用空间的
- 索引大大提高了查询效率，但降低了更新的速度，比如 INSERT、UPDATE、DELETE

### 1）索引结构

| 索引结构            | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| B+Tree              | 最常见的索引类型，大部分引擎都支持B+树索引                   |
| Hash                | 底层数据结构是用哈希表实现，只有精确匹配索引列的查询才有效，不支持范围查询 |
| R-Tree(空间索引)    | 空间索引是 MyISAM 引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-Text(全文索引) | 是一种通过建立倒排索引，快速匹配文档的方式，类似于 Lucene, Solr, ES |

| 索引       | InnoDB        | MyISAM | Memory |
| ---------- | ------------- | ------ | ------ |
| B+Tree索引 | 支持          | 支持   | 支持   |
| Hash索引   | 不支持        | 不支持 | 支持   |
| R-Tree索引 | 不支持        | 支持   | 不支持 |
| Full-text  | 5.6版本后支持 | 支持   | 不支持 |

### 2）B树

B-Tree (多路平衡查找树)

B树能够解决二叉树退化为单向链表的情况

 以一棵最大度数（max-degree，指一个节点的子节点个数）为5（5阶）的 b-tree 为例（每个节点最多存储4个key，5个指针）

![](https://s1.ax1x.com/2022/08/31/v4MaqO.png)

演示地址：https://www.cs.usfca.edu/~galles/visualization/BTree.html

#### B+Tree

结构图：

![](https://s1.ax1x.com/2022/08/31/v4QSY9.png)

演示地址：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

> 与 B-Tree 的区别：
>
> - 所有的数据都会出现在叶子节点
> - 叶子节点形成一个单向链表

MySQL 索引数据结构对经典的 B+Tree 进行了优化。在原 B+Tree 的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的 B+Tree，提高区间访问的性能。

![](https://s1.ax1x.com/2022/08/31/v4Qiy6.png)

#### Hash

哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中。
如果两个（或多个）键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可以通过链表来解决。

![](https://s1.ax1x.com/2022/08/31/v4QEwD.png)

特点：

- Hash索引只能用于对等比较（=、in），不支持范围查询（betwwn、>、<、…）
- 无法利用索引完成排序操作
- 查询效率高，通常只需要一次检索就可以了，效率通常要高于 B+Tree 索引

存储引擎支持：

- Memory
- InnoDB: 具有自适应hash功能，hash索引是存储引擎根据 B+Tree 索引在指定条件下自动构建的

为什么 InnoDB 存储引擎选择使用 B+Tree 索引结构？

- 相对于二叉树，层级更少，搜索效率高
- 对于 B-Tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针也跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低
- 相对于 Hash 索引，B+Tree 支持范围匹配及排序操作

### 3）索引分类

| 分类     | 含义                                                 | 特点                     | 关键字   |
| -------- | ---------------------------------------------------- | ------------------------ | -------- |
| 主键索引 | 针对于表中主键创建的索引                             | 默认自动创建，只能有一个 | PRIMARY  |
| 唯一索引 | 避免同一个表中某数据列中的值重复                     | 可以有多个               | UNIQUE   |
| 常规索引 | 快速定位特定数据                                     | 可以有多个               |          |
| 全文索引 | 全文索引查找的是文本中的关键词，而不是比较索引中的值 | 可以有多个               | FULLTEXT |

在 InnoDB 存储引擎中，根据索引的存储形式，又可以分为以下两种：

| 分类                      | 含义                                                       | 特点                 |
| ------------------------- | ---------------------------------------------------------- | -------------------- |
| 聚集索引(Clustered Index) | 将数据存储与索引放一块，索引结构的叶子节点保存了行数据     | 必须有，而且只有一个 |
| 二级索引(Secondary Index) | 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键 | 可以存在多个         |



![](https://s1.ax1x.com/2022/08/31/v4lZuV.png)

聚集索引选取规则：

- 如果存在主键，主键索引就是聚集索引
- 如果不存在主键，将使用第一个唯一(UNIQUE)索引作为聚集索引
- 如果表没有主键或没有合适的唯一索引，则 InnoDB 会自动生成一个 rowid 作为隐藏的聚集索引

#### 思考题

1. 以下 SQL 语句，哪个执行效率高？为什么？

```sql
select * from user where id = 10;
select * from user where name = 'Arm';-- 备注：id为主键，name字段创建的有索引
```

答：第一条语句，因为第二条需要回表查询，相当于两个步骤。

2. InnoDB 主键索引的 B+Tree 高度为多少？

答：假设一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB 的指针占用6个字节的空间，主键假设为bigint，占用字节数为8.
可得公式：`n * 8 + (n + 1) * 6 = 16 * 1024`，其中 8 表示 bigint 占用的字节数，n 表示当前节点存储的key的数量，(n + 1) 表示指针数量（比key多一个）。算出n约为1170。

如果树的高度为2，那么他能存储的数据量大概为：`1171 * 16 = 18736`；
如果树的高度为3，那么他能存储的数据量大概为：`1171 * 1171 * 16 = 21939856`。

### 4）使用规则

#### 查看、建立索引

```sql
# 查看索引
SHOW INDEX FROM `user`
# 建立索引
CREATE INDEX idx_user_phone ON `user`(phone)
# 按照age升序，phone倒序建立索引
CREATE INDEX idx_user_age_phone_ad ON `user`(age ASC , phone DESC)
```



#### 最左前缀法则

如果索引关联了多列（联合索引），要遵守最左前缀法则，最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。
如果跳跃某一列，索引将部分失效（后面的字段索引失效）。

联合索引中，出现范围查询（<, >），范围查询右侧的列索引失效。可以用>=或者<=来规避索引失效问题。

#### 索引失效情况

1. 在索引列上进行运算操作，索引将失效。如：`explain select * from tb_user where substring(phone, 10, 2) = '15';`
2. 字符串类型字段使用时，不加引号，索引将失效。如：`explain select * from tb_user where phone = 17799990015;`，此处phone的值没有加引号
3. 模糊查询中，如果仅仅是尾部模糊匹配，索引不会是失效；如果是头部模糊匹配，索引失效。如：`explain select * from tb_user where profession like '%工程';`，前后都有 % 也会失效。
4. 用 or 分割开的条件，如果 or 其中一个条件的列没有索引，那么涉及的索引都不会被用到。
5. 如果 MySQL 评估使用索引比全表更慢，则不使用索引。

#### SQL 提示

是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

例如，使用索引：
`explain select * from tb_user use index(idx_user_pro) where profession="软件工程";`
不使用哪个索引：
`explain select * from tb_user ignore index(idx_user_pro) where profession="软件工程";`
必须使用哪个索引：
`explain select * from tb_user force index(idx_user_pro) where profession="软件工程";`

use 是建议，实际使用哪个索引 MySQL 还会自己权衡运行速度去更改，force就是无论如何都强制使用该索引。

#### 覆盖索引&回表查询

尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能找到），减少 select *。

explain 中 extra 字段含义：
`using index condition`：查找使用了索引，但是需要回表查询数据
`using where; using index;`：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询

如果在聚集索引中直接能找到对应的行，则直接返回行数据，只需要一次查询，哪怕是select *；如果在辅助索引中找聚集索引，如`select id, name from xxx where name='xxx';`，也只需要通过辅助索引(name)查找到对应的id，返回name和name索引对应的id即可，只需要一次查询；如果是通过辅助索引查找其他字段，则需要回表查询，如`select id, name, gender from xxx where name='xxx';`

所以尽量不要用`select *`，容易出现回表查询，降低效率，除非有联合索引包含了所有字段

面试题：一张表，有四个字段（id, username, password, status），由于数据量大，需要对以下SQL语句进行优化，该如何进行才是最优方案：
`select id, username, password from tb_user where username='itcast';`

解：给username和password字段建立联合索引，则不需要回表查询，直接覆盖索引

#### 前缀索引

当字段类型为字符串（varchar, text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO，影响查询效率，此时可以只降字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

语法：`create index idx_xxxx on table_name(columnn(n));`
前缀长度：可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高，唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。
求选择性公式：

```sql
select count(distinct email) / count(*) from tb_user;
select count(distinct substring(email, 1, 5)) / count(*) from tb_user;
```

show index 里面的sub_part可以看到接取的长度

#### 单列索引&联合索引

单列索引：即一个索引只包含单个列
联合索引：即一个索引包含了多个列
在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

单列索引情况：
`explain select id, phone, name from tb_user where phone = '17799990010' and name = '韩信';`
这句只会用到phone索引字段

##### 注意事项

- 多条件联合查询时，MySQL优化器会评估哪个字段的索引效率更高，会选择该索引完成本次查询

### 5）设计原则

1. 针对于数据量较大，且查询比较频繁的表建立索引
2. 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高
4. 如果是字符串类型的字段，字段长度较长，可以针对于字段的特点，建立前缀索引
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价就越大，会影响增删改的效率
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询

## 四、SQL 优化

### 1）插入数据

普通插入：

1. 采用批量插入（一次插入的数据不建议超过1000条）
2. 手动提交事务
3. 主键顺序插入

大批量插入：
如果一次性需要插入大批量数据，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令插入。

```sql
# 客户端连接服务端时，加上参数 --local-infile（这一行在bash/cmd界面输入）
mysql --local-infile -u root -p
# 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;
select @@local_infile;
# 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table 'tb_user' fields terminated by ',' lines terminated by '\n';
```

### 2）主键优化

数据组织方式：在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表（Index organized table, IOT）

- 页分裂：页可以为空，也可以填充一般，也可以填充100%，每个页包含了2-N行数据（如果一行数据过大，会行溢出），根据主键排列。

- 页合并：当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用。当页中删除的记录到达 MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最靠近的页（前后）看看是否可以将这两个页合并以优化空间使用。

MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或创建索引时指定

主键设计原则：

- 满足业务需求的情况下，尽量降低主键的长度
- 插入数据时，尽量选择顺序插入，选择使用 AUTO_INCREMENT 自增主键
- 尽量不要使用 UUID 做主键或者是其他的自然主键，如身份证号
- 业务操作时，避免对主键的修改

### 3）order by优化

1. **Using filesort**：通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区 sort buffer 中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序
2. **Using index**：通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高

如果order by字段全部使用升序排序或者降序排序，则都会走索引，但是如果一个字段升序排序，另一个字段降序排序，则不会走索引，explain的extra信息显示的是`Using index, Using filesort`，如果要优化掉Using filesort，则需要另外再创建一个索引，如：`create index idx_user_age_phone_ad on tb_user(age asc, phone desc);`，此时使用`select id, age, phone from tb_user order by age asc, phone desc;`会全部走索引

总结：

- 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则
- 尽量使用覆盖索引
- 多字段排序，一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）
- 如果不可避免出现filesort，大数据量排序时，可以适当增大排序缓冲区大小 sort_buffer_size（默认256k）

### 4）group by优化

- 在分组操作时，可以通过索引来提高效率
- 分组操作时，索引的使用也是满足最左前缀法则的

如索引为`idx_user_pro_age_stat`，则句式可以是`select ... where profession order by age`，这样也符合最左前缀法则

### 5）limit优化

常见的问题如`limit 2000000, 10`，此时需要 MySQL 排序前2000000条记录，但仅仅返回2000000 - 2000010的记录，其他记录丢弃，查询排序的代价非常大。
优化方案：一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化

```sql
-- 此语句耗时很长
select * from tb_sku limit 9000000, 10;
-- 通过覆盖索引加快速度，直接通过主键索引进行排序及查询
select id from tb_sku order by id limit 9000000, 10;
-- 下面的语句是错误的，因为 MySQL 不支持 in 里面使用 limit
-- select * from tb_sku where id in (select id from tb_sku order by id limit 9000000, 10);
-- 通过连表查询即可实现第一句的效果，并且能达到第二句的速度
select * from tb_sku as s, (select id from tb_sku order by id limit 9000000, 10) as a where s.id = a.id;
```

### 6）count优化

MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(*) 的时候会直接返回这个数，效率很高（前提是不适用where）；
InnoDB 在执行 count(*) 时，需要把数据一行一行地从引擎里面读出来，然后累计计数。
优化方案：自己计数，如创建key-value表存储在内存或硬盘，或者是用redis

count的几种用法：

- 如果count函数的参数（count里面写的那个字段）不是NULL（字段值不为NULL），累计值就加一，最后返回累计值
- 用法：count(*)、count(主键)、count(字段)、count(1)
- count(主键)跟count(*)一样，因为主键不能为空；count(字段)只计算字段值不为NULL的行；count(1)引擎会为每行添加一个1，然后就count这个1，返回结果也跟count(*)一样；count(null)返回0

各种用法的性能：

- count(主键)：InnoDB引擎会遍历整张表，把每行的主键id值都取出来，返回给服务层，服务层拿到主键后，直接按行进行累加（主键不可能为空）
- count(字段)：没有not  null约束的话，InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，服务层判断是否为null，不为null，计数累加；有not null约束的话，InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层，直接按行进行累加
- count(1)：InnoDB 引擎遍历整张表，但不取值。服务层对于返回的每一层，放一个数字 1 进去，直接按行进行累加
- count(*)：InnoDB 引擎并不会把全部字段取出来，而是专门做了优化，不取值，服务层直接按行进行累加

按效率排序：count(字段) < count(主键) < count(1) < count(*)，所以尽量使用 count(*)

### 7）update优化（避免行锁升级为表锁）

InnoDB 的行锁是针对索引加的锁，不是针对记录加的锁，并且该索引不能失效，否则会从行锁升级为表锁。

如以下两条语句：
`update student set no = '123' where id = 1;`，这句由于id有主键索引，所以只会锁这一行；
`update student set no = '123' where name = 'test';`，这句由于name没有索引，所以会把整张表都锁住进行数据更新，解决方法是给name字段添加索引

## 五、视图

### 1、创建/修改视图

```sql
#创建
CREATE OR REPLACE VIEW user_v_1 AS SELECT id,name FROM `user` WHERE id<=3
#修改
ALTER VIEW user_v_1 AS SELECT id,name FROM `user` WHERE id<=3
#删除
DROP VIEW IF EXISTS user_v_1
#插入
INSERT INTO user_v_1 VALUES(5,'TOM')
```

### 2、视图检查选项cascaded

但是，当在视图中插入一条不符合视图规则的语句时，仍然会执行成功

例如，执行`INSERT INTO user_v_1 VALUES(5,'TOM')`（视图中我们制定id小于等于3），但是执行后会发现，原表中出现该数据，而视图中并未发现该数据，为了避免此现象，在创建视图时要使用关键字`WITH CASCADED CHECK OPTION`

```sql
CREATE OR REPLACE VIEW user_v_1 AS SELECT id,name FROM `user` WHERE id<=3 WITH CASCADED CHECK OPTION
```

> `WITH CASCADED CHECK OPTION`也可以使用`WITH LOCALCHECK OPTION`

加上cascaded（级联）关键字后，对视图的增删改操作首先会检查是否满足视图的条件，也就是保证视图的操作只能在允许的范围之内

**基于视图创建新的视图**

基于视图创建新的视图(新视图使用了cascaded关键字)，新视图的增删改操作仍然会遵循原来视图的规则（**即使原来视图没有使用cascaded关键字**），同时也会遵循当前的视图规则

### 3、视图检查选项local

使用`WITH LOCALCHECK OPTION`基于视图创建新视图，只会检查当前视图的条件是否满足，上一级的条件是否检查，与上一级是否使用cascaded或者local有关

总结

- local只会限制当前视图的条件
- decaded会限制所有级联视图的条件

### 4、视图的更新

要使视图可更新，视图中的行必须与基础表中的行一一对应才可以。

如果视图包含以下任何一个选项，则该视图不可更新：

1. 聚合函数或者窗口函数(SUM()、MIN()、MAX、COUNT)
2. DISTINCT
3. GROUP BY
4. HAVING
5. UNION或者UNION ALL

/5、视图的作用

- 简化操作——经常被使用的查询可以被定义为视图
- 安全——通过视图让用户操作指定数据
- 数据独立——视图可以帮助用户屏蔽真实表结构变化带来的影响



## 六、存储过程

### 1、基本语法

```sql
#创建存储过程
CREATE PROCEDURE pro1()
BEGIN
	SELECT count(*) FROM `user`;
END;
#调用存储过程
CALL pro1()
#查看存储过程
SELECT * FROM information_schema.ROUTINES 
SHOW CREATE PROCEDURE pro1
#删除存储过程
DROP PROCEDURE IF EXISTS pro1
```

### 2、存储过程-系统变量

查看系统变量

```sql
#系统变量
SHOW SESSION VARIABLES 
SHOW SESSION VARIABLES LIKE 'basedir'
SHOW SESSION VARIABLES LIKE 'auto%'
#查看全局系统变量
SHOW GLOBAL VARIABLES LIKE 'auto%'
#另一种语法
SELECT @@session.autocommit
SELECT @@global.autocommit
```

设置系统变量

```sql
#设置系统变量-session级别
SET SESSION autocommit = 1
#设置系统变量-global级别
SET GLOBAL autocommit = 1
```

但是数据库重启时会恢复初始值，想要不失效，就得去/etc/my.cnf中配置

### 3、存储过程-用户自定义变量

`@@`代表系统变量，而`@`代表自定义变量

```sql
#设置用户自定义变量
SET @myname = 'wht'
SELECT count(*) INTO @myname FROM `user`
#推荐使用下面的语法，因为mysql的赋值和比较使用的都是'='，用':='以示区分
SET @myname := 'wht'
#使用自定义变量
SELECT @myname
```

### 4、存储过程-局部变量

```sql
#局部变量，只在begin和end之间生效，超出范围不生效
CREATE PROCEDURE p2()
BEGIN
	DECLARE user_count int DEFAULT(0); #声明局部变量user_count
	SELECT count(*) INTO user_count FROM `user`;
	SELECT user_count;
END;
CALL p2
```

### 5、存储过程-if判断

```sql
#if判断
CREATE PROCEDURE p3()
BEGIN
	DECLARE score int DEFAULT(58);
	DECLARE res VARCHAR(10);
	IF score>=60 THEN
		SET res := '及格';
	ELSEIF score>=80 THEN
		SET res := '优秀';
	ELSE
		SET res := '不及格';
	END IF;
	SELECT res;
END;

CALL p3()

```

### 6、存储过程-参数

```sql
#参数in和out
CREATE PROCEDURE p5(IN score INT, OUT res VARCHAR(10))#定义有参数的procedure
BEGIN
	IF score>=80 THEN
		SET res := '优秀';
	ELSEIF score>=60 THEN
		SET res := '及格';
	ELSE
		SET res := '不及格';
	END IF;
END;

CALL p5(98,@res);
SELECT @res;
```

```sql
#参数inout
CREATE PROCEDURE p6(INOUT score DOUBLE)
BEGIN
	SET score := score*0.5;
END;
SET @res = 200;
CALL p6(@res);
SELECT @res;
```

### 7、存储过程-case

```sql
#case
CREATE PROCEDURE p7(IN month INT)
BEGIN
	DECLARE res VARCHAR(10);
CASE
	WHEN month>=1 AND month<=3 THEN
		SET res := '春';
	WHEN 4<=month AND month<=6 THEN
		SET res := '夏';
	WHEN 6<=month AND month<=9 THEN
		SET res := '秋';
	WHEN 9<=month AND month<=12 THEN
		SET res := '冬';
END CASE;
SELECT res;
END;
CALL p7(3);
```

### 8、存储过程-循环

```sql
#while循环
CREATE PROCEDURE p8(IN n INT)
BEGIN
	DECLARE total INT DEFAULT(0);
	WHILE n>0 DO
	SET total := total+n;
	SET n := n-1;
END WHILE;
SELECT total;
END;
CALL p8(4);
```

repeat相当于do while

```sql
#repeat
CREATE PROCEDURE p9(IN n INT)
BEGIN
DECLARE total INT DEFAULT(0);
REPEAT
	SET total := total+n;
	SET n := n-1;
UNTIL  n<=0 END REPEAT;
SELECT total;
END;

CALL p9(3);
```

loop循环

```sql
#LOOP
CREATE PROCEDURE p10(IN n INT)
BEGIN
	DECLARE total INT DEFAULT(0);
	
	sum:LOOP
	IF n<=0 THEN
		LEAVE sum; 
	END IF; 
	SET total = total+n;
	SET n = n-1;
	END LOOP sum;
	SELECT total;
END;

CALL p10(3);
```

### 9、存储过程-游标

略

### 10、存储函数

```sql
#定义存储函数
CREATE FUNCTION fun1(n INT)
RETURNS INT DETERMINISTIC #DETERMINISTIC表示输入输出
BEGIN
	DECLARE total INT DEFAULT 0;
	
	WHILE n>0 DO
		SET total := total+n;
		SET n := n-1;
	END WHILE;
	RETURN total;
END;
#调用
SELECT fun1(3)
```

![](https://s1.ax1x.com/2022/09/01/v5WzgH.png)

## 七、触发器

trigger-触发器

### 创建触发器

创建插入时触发器

```sql
#触发器
-- 插入数据时
CREATE TRIGGER user_insert_trigger
	AFTER INSERT ON `user` FOR EACH ROW -- 在每一行插入时触发
BEGIN
	INSERT INTO user_logs VALUES
	(NULL,'插入',NOW(),new.id);
END;
# 查看触发器
SHOW TRIGGERS;
#删除触发器
DROP TRIGGER user_insert_trigger;
#触发触发器
INSERT INTO `user` VALUES(12,'赵刚','11110',NULL,NULL,NULL,1);
#检查结果
SELECT * FROM user_logs;
```

创建更新时触发器

```sql
#创建更新时触发器
CREATE TRIGGER user_update_trigger
	AFTER UPDATE ON `user` FOR EACH ROW
BEGIN
	INSERT INTO user_logs VALUES
	(NULL,CONCAT('更新了',old.id),NOW(),new.id);
END;
#执行更新操作
UPDATE `user` SET phone = '3330' WHERE `name`='赵刚';
```

## 八、锁

- 全局锁
- 表级锁
- 行级锁

### 1、全局锁

```sql
Flush tables with read lock
```

全局锁就是对整个数据库实例加锁。MySQL提供了一个加全局读锁的方法，命令是`Flush tables with read lock`。当需要让整个库处于只读状态的时候，可以使用这个命令，之后其他线程的以下语句会被阻塞：数据更新语句（数据的增删改）、数据定义语句（包括建表、修改表结构等）和更新类事务的提交语句

典型应用场景：全库的逻辑备份，对所有表进行锁定，从而获取一致性视图，保证数据的完整性

![](https://s1.ax1x.com/2022/09/01/v54x8U.png)

全局锁的特点

![](https://s1.ax1x.com/2022/09/01/v55arj.png)

数据库备份既然要全库只读，为什么不使用`set global readonly=true`的方式？

- 在有些系统中，readonly的值会被用来做其他逻辑，比如用来判断一个库是主库还是备库。因此修改global变量的方式影响面更大
- 在异常处理机制上有差异。如果执行Flush tables with read lock命令之后由于客户端发生异常断开，那么MySQL会自动释放这个全局锁，整个库回到可以正常更新的状态。而将整个库设置为readonly之后，如果客户端发生异常，则数据库会一直保持readonly状态，这样会导致整个库长时间处于不可写状态，风险较高
  

### 2、表级锁

**MySQL里面表级别的锁有两种：一种是表锁，一种是元数据锁（meta data lock，MDL）**

```sql
#加表锁
lock tables … read/write
#解锁
unlock tables
```

#### 表锁

如果在某个线程A中执行`lock tables t1 read,t2 wirte;`这个语句，则其他线程写t1、读写t2的语句都会被阻塞。同时，线程A在执行unlock tables之前，也只能执行读t1、读写t2的操作。连写t1都不允许

**读锁也叫共享读锁**

![](https://s1.ax1x.com/2022/09/01/v55WL9.png)

**写锁也叫表独占写锁**

![](https://s1.ax1x.com/2022/09/01/v55oi6.png)

#### 元数据锁

另一类表级的锁是**元数据锁MDL**。MDL不需要显式使用，在访问一个表的时候会被自动加上。MDL的作用是，**避免DML和DDL冲突，保证读写的正确性**。如果一个查询正在遍历一个表中的数据，而执行期间另一个线程对这个表结构做了变更，删了一列，那么查询线程拿到的结果跟表结构对不上，肯定不行。

**当对一个表做增删改查操作的时候，加MDL读锁（共享）；当要对表做结构变更操作的时候，加MDL写锁（排他）**

![](https://s1.ax1x.com/2022/09/01/v5zi4g.png)

**事务中的MDL锁，在语句执行开始时申请，但是语句结束后并不会马上释放，而会等到整个事务提交后再释放**

#### 意向锁

意向锁其实本质上不是锁，而是一种标记，是为了解决以下问题而生的

在一个语句执行插入时，innodb引擎会加行级锁，但是加锁之后如果有其他线程尝试加表锁，就需要逐行判断又没有加行锁，这是十分影响性能的，所以在加行级锁时直接给这个表一个标记不就好了吗，这样，当另一个线程加表锁时只需判断有没有该标记就好了。

这个标记就是意向锁。

![](https://s1.ax1x.com/2022/09/01/v5zgqP.md.png)

### 3、行级锁

在InNoDB中，行级锁是基于索引实现的，也就是当查询条件非索引时，会对整张表加锁

#### 行锁

- MySQL的行锁是在引擎层由各个引擎自己实现的。但不是所有的引擎都支持行锁，比如MyISAM引擎就不支持行锁

- RC（Read Commit）、RR（Repeatable Read ）两种隔离级别下都支持行级锁

![](https://s1.ax1x.com/2022/09/01/v5zzRJ.md.png)

行锁又分为两种：共享锁（S）和排它锁（X）

![](https://s1.ax1x.com/2022/09/01/vIS3o8.png)

不同操作对应的加锁情况

![](https://s1.ax1x.com/2022/09/01/vIStzj.png)

> **在InnoDB事务中，行锁是在需要的时候才加上的，但并不是不需要了就立刻释放，而是要等到事务结束时才释放。这个就是两阶段锁协议**

#### 间隙锁

- 为了解决幻读问题，InnoDB引入了间隙锁，锁的就是两个值之间的空隙

- 在RR（Repeatable Read）隔离级别下支持

![](https://s1.ax1x.com/2022/09/01/v5zLZV.png)



#### 临键锁

- 行锁和间隙锁的组合

![](https://s1.ax1x.com/2022/09/01/v5zxG4.md.png)

默认情况下，InnoDB在RR事务隔离级别运行，InnoDB使用临键锁进行搜索和索引扫描，以防止幻读。

- 索引上的等值查询（唯一索引），给不存在的记录加锁时，优化为间隙锁。

比如我们要插入一条id为19 的记录

![](https://s1.ax1x.com/2022/09/02/vIktAI.png)

- 索引上的等值查询（普通索引），向右遍历时最后一个值不满足查询需求时，临键锁退化为间隙锁。

比如要查询的是18，会在下面的位置加间隙锁，目的是为了防止幻读

![](https://s1.ax1x.com/2022/09/02/vIkEnJ.png)

- 索引上的范围查询（唯一索引），会访问到不满足条件的第一个值为止

比如访问的是id<=34的记录

![](https://s1.ax1x.com/2022/09/01/v5zxG4.md.png)

> 注：间隙锁可以共存，一个事务的间隙锁不会影响另一个事务的间隙锁



## 九、逻辑存储结构

### 1、逻辑存储结构

![](https://s1.ax1x.com/2022/09/02/vIKJHO.md.png)

**1). 表空间**

表空间是InnoDB存储引擎逻辑结构的最高层， 如果用户启用了参数 innodb_file_per_table(在 8.0版本中默认开启) ，则每张表都会有一个表空间（xxx.ibd），一个mysql实例可以对应多个表空 间，用于存储记录、索引等数据。

**2). 段** 

段，分为数据段（Leaf node segment）、索引段（Non-leaf node segment）、回滚段 （Rollback segment），InnoDB是索引组织表，数据段就是B+树的叶子节点， 索引段即为B+树的 非叶子节点。段用来管理多个Extent（区）。

**3). 区** 

区，表空间的单元结构，每个区的大小为1M。 默认情况下， InnoDB存储引擎页大小为16K， 即一 个区中一共有64个连续的页。

**4). 页** 

页，是InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性， InnoDB 存储引擎每次从磁盘申请 4-5 个区。

**5). 行** 

行，InnoDB 存储引擎数据是按行进行存放的。 在行中，默认有两个隐藏字段： 

- Trx_id：每次对某条记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列。 
- Roll_pointer：每次对某条引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个 隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息。

### 2、架构

整体架构

[![vIoDSK.md.png](https://s1.ax1x.com/2022/09/02/vIoDSK.md.png)](https://imgse.com/i/vIoDSK)

 #### 内存结构

![](https://s1.ax1x.com/2022/09/02/vIohfP.png)

在左侧的内存结构中，主要分为这么四大块儿： 

- Buffer Pool
- Change Buffer
- Adaptive Hash Index
- Log Buffer

**1). Buffer Pool**

InnoDB存储引擎基于磁盘文件存储，访问物理硬盘和在内存中进行访问，速度相差很大，为了尽可能 弥补这两者之间的I/O效率的差值，就需要把经常使用的数据加载到缓冲池中，避免每次访问都进行磁 盘I/O。

在InnoDB的缓冲池中不仅缓存了索引页和数据页，还包含了undo页、插入缓存、自适应哈希索引以及 InnoDB的锁信息等等。

缓冲池 Buffer Pool，是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增 删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频 率刷新到磁盘，从而减少磁盘IO，加快处理速度。

缓冲池**以Page页为单位**，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型：

-  free page：空闲page，未被使用。
- clean page：被使用page，数据没有被修改过。
-  dirty page：脏页，被使用page，数据被修改过，也中数据与磁盘的数据产生了不一致。

在专用服务器上，通常将多达80％的物理内存分配给缓冲池

查看缓冲池大小

```sql
show variables like 'innodb_buffer_pool_size';
```

**2). Change Buffer**

Change Buffer，更改缓冲区（针对于非唯一二级索引页），在执行DML语句时，如果这些数据Page 没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区 Change Buffer 中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中。

Change Buffer的作用是什么？

![](https://s1.ax1x.com/2022/09/02/vITtc8.png)

与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更新 可能会影响索引树中不相邻的二级索引页，如果每一次都操作磁盘，会造成大量的磁盘IO。有了 ChangeBuffer之后，我们可以在缓冲池中进行合并处理，**减少磁盘IO**。

**3). Adaptive Hash Index**

**自适应hash索引**，用于优化对Buffer Pool数据的查询。MySQL的innoDB引擎中虽然没有直接支持 hash索引，但是给我们提供了一个功能就是这个自适应hash索引。因为前面我们讲到过，hash索引在 进行等值匹配时，一般性能是要高于B+树的，因为hash索引一般只需要一次IO即可，而B+树，可能需 要几次匹配，所以hash索引的效率要高，但是hash索引又不适合做范围查询、模糊匹配等。

InnoDB存储引擎会监控对表上各索引页的查询，如果观察到在特定的条件下hash索引可以提升速度， 则建立hash索引，称之为自适应hash索引。

```sql
#参数： innodb_adaptive_hash_index
show variables like 'innodb_adaptive_hash_index';
```

**4). Log Buffer**

Log Buffer：日志缓冲区，用来保存要写入到磁盘中的log日志数据（redo log 、undo log）， 默认大小为 16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事 务，增加日志缓冲区的大小可以节省磁盘 I/O。

```sql
#innodb_log_buffer_size：缓冲区大小
#innodb_flush_log_at_trx_commit：日志刷新到磁盘时机，取值主要包含以下三个：
#1: 日志在每次事务提交时写入并刷新到磁盘，默认值。
#0: 每秒将日志写入并刷新到磁盘一次。
#2: 日志在每次事务提交后写入，并每秒刷新到磁盘一次。

show variables like 'innodb_log_buffer_size';
show variables like 'innodb_flush_log_at_trx_commit';
```

####  磁盘结构

接下来，再来看看InnoDB体系结构的右边部分，也就是磁盘结构：

![](https://s1.ax1x.com/2022/09/02/vI7pCt.png)

主要有以下几部分

- 1). System Tablespace
- 2). File-Per-Table Tablespaces
- 3). General Tablespaces
- 4). Undo Tablespaces
- 5). Temporary Tablespaces
- 6). Doublewrite Buffer Files
- 7). Redo Log

**1). System Tablespace**

系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建 的，它也可能包含表和索引数据。(在MySQL5.x版本中还包含InnoDB数据字典、undolog等) 

```sql
#参数：innodb_data_file_path
show variables like 'innodb_data_file_path';
```

系统表空间，默认的文件名叫 ibdata1。

**2). File-Per-Table Tablespaces**

如果开启了innodb_file_per_table开关 ，则每个表的文件表空间包含单个InnoDB表的数据和索引 ，并存储在文件系统上的单个数据文件中。 

```sql
#开关参数：innodb_file_per_table ，该参数默认开启。
```

那也就是说，我们每创建一个表，都会产生一个表空间文件

**3). General Tablespaces**

通用表空间，需要通过 CREATE TABLESPACE 语法创建通用表空间，在创建表时，可以指定该表空 间。

```sql
#A. 创建表空间
CREATE TABLESPACE ts_name ADD DATAFILE 'file_name' ENGINE = engine_name;
#B. 创建表时指定表空间
CREATE TABLE xxx ... TABLESPACE ts_name;
```

**4). Undo Tablespaces**

撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小16M），用于存储 undo log日志。

**5). Temporary Tablespaces** 

InnoDB 使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据。

**6). Doublewrite Buffer Files**

双写缓冲区，innoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件 中，便于系统异常时恢复数据。

**7). Redo Log**

重做日志，是用来**实现事务的持久性**。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log）,前者是在内存中，后者在磁盘中。当事务提交之后会把所 有修改信息都会存到该日志中, 用于在刷新脏页到磁盘时,发生错误时, 进行数据恢复使用。

![](https://s1.ax1x.com/2022/09/02/vI7U8x.png)

#### 后台线程

<img src="https://s1.ax1x.com/2022/09/02/vI7Dqe.png" width="50%;" />

在InnoDB的后台线程中，分为4类，分别是：

- Master Thread 
- IO Thread
- Purge Thread
- Page Cleaner Thread

**1). Master Thread** 

核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中, 保持数据的一致性， 还包括脏页的刷新、合并插入缓存、undo页的回收 。

**2). IO Thread** 

在InnoDB存储引擎中大量使用了AIO来处理IO请求, 这样可以极大地提高数据库的性能，而IO Thread主要负责这些IO请求的回调。

```sql
# 查看innoDB信息
show engine innodb status
```

**3). Purge Thread**

主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回 收。

**4). Page Cleaner Thread** 

协助 Master Thread 刷新脏页到磁盘的线程，它可以减轻 Master Thread 的工作压力，减少阻 塞。

###  3、事务原理

**1). 事务** 

事务 是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系 统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

**2). 特性ACID**

• 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。 

• 一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态。 

• 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环 境下运行。 

• 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

在InnoDB中，

而对于这四大特性，实际上分为两个部分。 其中的原子性、一致性、持久性，实际上是由InnoDB中的 两份日志来保证的，一份是redo log日志，一份是undo log日志。 

而持久性是通过数据库的锁， 加上MVCC来保证的。

![](https://s1.ax1x.com/2022/09/02/vIHFRx.png)

#### redo log

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。 

该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file）,前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中, 用于在刷新脏页到磁盘,发生错误时, 进行数据恢复使用。

**如果没有redolog，可能会存在什么问题的？** 

我们知道，在InnoDB引擎中的内存结构中，主要的内存区域就是缓冲池，在缓冲池中缓存了很多的数 据页。 当我们在一个事务中，执行多个增删改的操作时，InnoDB引擎会先操作缓冲池中的数据，如果 缓冲区没有对应的数据，会通过后台线程将磁盘中的数据加载出来，存放在缓冲区中，然后将缓冲池中 的数据修改，修改后的数据页我们称为脏页。 而脏页则会在一定的时机，通过后台线程刷新到磁盘 中，从而保证缓冲区与磁盘的数据一致。 而缓冲区的脏页数据并不是实时刷新的，而是一段时间之后 将缓冲区的数据刷新到磁盘中，假如刷新到磁盘的过程出错了，而提示给用户事务提交成功，而数据却 没有持久化下来，这就出现问题了，没有保证事务的持久性。

那么，如何解决上述的问题呢？ 在InnoDB中提供了一份日志 redo log，接下来我们再来分析一 下，通过redolog如何解决这个问题。

![](https://s1.ax1x.com/2022/09/02/vIHNwQ.png)

有了redolog之后，当对缓冲区的数据进行增删改之后，会首先将操作的数据页的变化，记录在redo log buffer中。在事务提交时，会将redo log buffer中的数据刷新到redo log磁盘文件中。 过一段时间之后，如果刷新缓冲区的脏页到磁盘时，发生错误，此时就可以借助于redo log进行数据 恢复，这样就保证了事务的持久性。 而如果脏页成功刷新到磁盘 或 或者涉及到的数据已经落盘，此 时redolog就没有作用了，就可以删除了，所以存在的两个redolog文件是循环写的。

> 那为什么每一次提交事务，要刷新redo log 到磁盘中呢，而不是直接将buffer pool中的脏页刷新 到磁盘呢 ? 
>
> 因为在业务操作中，我们操作数据一般都是随机读写磁盘的，而不是顺序读写磁盘。 而redo log在 往磁盘文件中写入数据，由于是日志文件，所以都是顺序写的。顺序写的效率，要远大于随机写。 这 种先写日志的方式，称之为 WAL（Write-Ahead Logging）。

####  undo log 

回滚日志，用于记录数据被修改前的信息 , 作用包含两个 : 提供回滚(保证事务的原子性) 和 MVCC(多版本并发控制) 。 

undo log和redo log记录物理日志不一样，它是逻辑日志。

可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的 update记录。

当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚。

Undo log销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些 日志可能还用于MVCC。 

Undo log存储：undo log采用段的方式进行管理和记录，存放在前面介绍的 rollback segment 回滚段中，内部包含1024个undo log segment。

###  4、MVCC

#### 基本概念

1). 当前读 

读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加 锁。对于我们日常的操作，如：`select ... lock in share mode`(共享锁)，`select ... for update、update、insert、delete`(排他锁)都是一种**当前读**。

2). 快照读 

简单的select（不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据， 不加锁，是非阻塞读。

- Read Committed：每次select，都生成一个快照读。
- Repeatable Read：开启事务后第一个select语句才是快照读的地方。 
- Serializable：快照读会退化为当前读。

3). MVCC

全称 **Multi-Version Concurrency Control**，多版本并发控制。指维护一个数据的多个版本， 使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需 要依赖于数据库记录中的三个隐式字段、undo log日志、readView。

#### 隐藏字段

实际上在使用InnoDB的表中除了显示的字段以外，InnoDB还会自动的给我们添加三个隐藏字段及其含义分别是

![](https://s1.ax1x.com/2022/09/02/vIHTOO.png)

> 而上述的前两个字段是肯定会添加的， 是否添加最后一个字段DB_ROW_ID，得看当前表有没有主键， 如果有主键，则不会添加该隐藏字段。

```bash
 #查看隐式字段需要到存储ibd文件的目录执行下面的命令
 ibd2sdi stu.ibd
```

#### undolog

回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。 当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。 而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即 被删除。

#####  版本链

![](https://s1.ax1x.com/2022/09/02/vIHz1P.png)

> DB_TRX_ID : 代表最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID，是自增的。 
>
> DB_ROLL_PTR ： 由于这条数据是才插入的，没有被更新过，所以该字段值为null。

有四个并发事务同时在访问这张表

![](https://s1.ax1x.com/2022/09/02/vIbi7Q.png)

左右两张图分别是执行事务3和事务4时的版本链

![](https://s1.ax1x.com/2022/09/02/vIbkkj.png)

> 最终我们发现，不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条 记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录。

#### readview

ReadView（读视图）是快照读 SQL执行时MVCC提取数据的依据，**记录并维护系统当前活跃的事务 （未提交的）id **。

ReadView中包含了四个核心字段：

![](https://s1.ax1x.com/2022/09/02/vIbY1x.png)

而在readview中就规定了版本链数据的访问规则： 

**trx_id** 代表当前undolog版本链对应事务ID。

![](https://s1.ax1x.com/2022/09/03/vIzz1H.png)

不同的隔离级别，生成ReadView的时机不同：

-  READ COMMITTED ：在事务中每一次执行快照读时生成ReadView。 
- REPEATABLE READ：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。

####  原理分析

##### RC隔离级别

RC隔离级别下，在事务中每一次执行快照读时生成ReadView。

例如：

在事务5中，查询了两次id为30的记录，由于隔离级别为Read Committed，所以每一次进行快照读 都会生成一个ReadView，那么两次生成的ReadView如下。

![](https://s1.ax1x.com/2022/09/03/voSCnI.png)

那么这两次快照读在获取数据时，就需要根据所生成的ReadView以及ReadView的版本链访问规则， 到**undolog版本链**中匹配数据，最终决定此次快照读返回的数据。

在第一次查询id为30的记录时，因为是RC隔离级别，所以只能读已提交事务，也就是事务2提交的事务，那么我们来看看具体是怎样在版本链中找到该提交事务的版本的。

![](https://s1.ax1x.com/2022/09/03/voSiHP.png)

首先会找到该行的隐藏字段`DB_TRX_ID`，也就是当前操作的事务ID，因为当前事务4正在操作该行，所以TRX_ID为4,4进入右侧比对

![](https://s1.ax1x.com/2022/09/03/voSz5T.png)

发现1234都不满足，所以事务5读取不到事务4当前的数据，需要沿着undo log的版本链回退，也就是TRX_ID为3（对应事务3当前的数据），3进入右侧比对，发现1234又不成立，所以继续回退，进入TRX_ID为2,2进入右侧比对，发现满足`trx_id<min_trx_id`，（逻辑上理解为当前读的事务ID小于最小的正在操作该行的事务ID），所以读取成功。

最终读到的数据是事务2当前的数据。

![](https://s1.ax1x.com/2022/09/03/vopNi8.png)

##### RR隔离级别

RR隔离级别下，仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。 

而RR 是可重复读，在一个事务中，执行两次相同的select语句，查询到的结果是一样的。

![](https://s1.ax1x.com/2022/09/03/vo9jhT.png)

我们看到，在RR隔离级别下，只是在事务中第一次快照读时生成ReadView，后续都是复用该 ReadView，那么既然ReadView都一样， ReadView的版本链匹配规则也一样， 那么最终快照读返 回的结果也是一样的。

所以呢，MVCC的实现原理就是通过 InnoDB表的隐藏字段、UndoLog 版本链、ReadView来实现的。 

而MVCC + 锁，则实现了事务的隔离性。 而一致性则是由redolog 与 undolog保证。























