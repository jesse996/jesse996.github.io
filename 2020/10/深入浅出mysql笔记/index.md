# 深入浅出MySql笔记


SQL 分类：

-   DDL：数据定义语言。定义数据库，表，列，索引等。create,drop,alter
-   DML：数据操纵语言。增删改查。insert,delete,update,select
-   DCL：数据控制语言。定义了访问权限和安全级别。grant,revoke

MySQL 操作所影响的记录行数，通常只对增删改生效，drop 等 DDL 操作通常显示 “0 rows affected”

HAVING 和 WHERE 的区别在于，HAVING 是对聚合后的结果进行条件的过滤，而 WHERE 是在聚合前就对记录进行过滤。

## SQL 优化

-   通过`show status`命令了解各种 SQL 的执行频率

### 可以通过下面两种方法定位执行效率低的 SQL：

-   通过慢日志查询。将`slow-query-log`设为 1，MySQL 会将执行时间超过`long_query_time`参数设定阈值的 SQL 写入`show_query_log_file`
-   慢日志日志在查询结束以后才记录，所以可以用`show processlist`命令查看当前 MySQL 在进行的线程

### 通过 EXPLAIN 分析低效 SQL 的执行计划

从上到下，越来越快：

-   type=ALl ，全表扫描
-   type=index ，索引全扫描，MySQL 遍历整个索引来查询匹配的行：`select username from user`
-   type=range，索引范围扫描
-   type=ref ,使用非`唯一索引扫描`或`唯一索引的前缀扫描`，返回匹配某个单独值的记录行，例如：`select * from user where username = '张三'`;
-   type=eq_ref，类似 ref，区别就在使用的索引是唯一索引
-   type=const/system ，单表中最多有一个匹配行，查询起来非常迅速，所以这个匹配行中的其他列的值可以被优化器在当前查询中当作常量来处理，例如根据主键或唯一索引进行的查询。
-   type=NULL,MySQL 不用访问表或索引，直接就能得到结果。

### 通过`show profile`或`show profile`分析 SQL

### 通过 trace 分析优化器如何选择执行计划

### MySQL 目前提供了一下 4 中索引：

-   B-Tree
-   HASH
-   R-Tree：空间索引，主要用于地理空间数据类型
-   Full-text：全文索引

### 存在索引但不能使用索引的典型场景：

-   以%开头的 LIKE 查询不能利用 B-tree 索引。
-   数据类型中出现隐式类型转换的时候也不会使用索引，特别是当列类型是字符串，那么一定要在 where 条件中把字符常量值用引号引起来
-   复合索引的情况下，不满足最左原则是不会使用索引的
-   如果 MySQL 估计使用索引比全表扫描更慢，则不使用索引
-   用 or 分割开的条件，如果 or 前的条件中的列有索引，而后面的列没有索引，那么涉及的索引都不会被用到。因为 or 后面的条件列中没有索引，那么后面的查询肯定要走全表扫描，在存在全表扫描的情况下，就没有必要多一次索引扫描增加 IO 访问。

### 两个简单实用的优化方法：

-   定期分期表和检查表

    分析表：
    `analyze table tablename`
    检查表：
    `check table tablename`

-   定期优化表

    `optimize table tablename`

要在数据库不繁忙的时候执行相关操作

### 优化 INSERT 语句：

-   尽量使用多个值表的 INSERT 语句
-   如果从不同客户插入很多行，可以通过 INSERT DELAYED 语句得到更高的速度。
-   当从文本文件装载一个表时，使用 LOAD DATA INFILE

### 优化 ORDER BY 语句

mysql 有两种排序方式

-   通过有序索引直接返回有序数据
-   对返回的数据进行排序，也就是常说的 Filesort 排序

所以，尽量减少额外的排序，通过索引直接返回有序数据。where 和 order by 使用相同的索引，并且 order by 的顺序和索引的顺序相同，并且 order by 的字段都是升序或都是降序。

### 优化 GROUP BY 语句

group by 会默认排序，通过 order by null 可以禁止排序

### 优化分页查询

1. 第一种优化思路

这是全表扫描：

```sql
select id,desc from film order by titile limit 50,5;
```

在索引上完成分页操作，最后根据主键关联回原表查询所需的其他列内容：

```sql
select a.id,a.desc from file a inner join (select id from file order by title limit 50,5)b on a.id=b.id;
```

这种方式让 MySQL 扫描尽可能少的页面来提高分页效率

2. 第二种优化思路
   把 limit 查询转换为某个位置的查询，把 LIMIT m,n 转换为 LIMIT n

### 直方图

利用直方图，用户可以对一张表的一列做数据分布的统计，特别是针对没有索引的字段。主要场景是用来计算字段选择性，即过滤效率。
可以帮助优化器找到更优的执行计划。

生成直方图：

```sql
analyze table tbname update histogram on clo_name [,col_name] [with n buckets];
```

删除直方图：

```sql
analyze table tbname drop histogram on clo_name [,col_name];
```

直方图的分离：

-   等宽直方图
-   等高直方图

列中不同值得个数小于等于 bucket 数，则为等宽直方图，否则为等高直方图

并不是所有大表的字段都需要创建直方图。通常在一些唯一值较少，数据分布不均衡，查询较为频繁，没有创建索引的字段上考虑创建直方图。虽然创建索引有时也可以达到优化效果，但由于使用率低，索引维护成本高，故通常不会创建索引
直方图只需要创建一次，对数据的变更不需要实时进行维护，代价较小，更适合此类查询。

### 利用 group by 的 with rollup 子句

with rollup 反映的是一种 OLAP 思想，就是说这个 group by 语句完成后可以满足用户想要得到的任何一个分组以及分组组合的聚合信息值。

当使用 rollup 时，不能同时使用 order by 字句进行排序。此外，limit 要在 rollup 后面。

### 大小写问题

在大多数 UNIX 环境中，由于操作系统对大小写敏感，导致数据库名和表名大小写敏感，而 Windows 则不敏感。
列，索引，存储子程序和触发器在任何平台都对大小写不敏感。默认情况下，表别名在 UNIX 中大小写敏感，在 windows 或 maxOS X 中大小写不敏感。

最好统一用小写。

MySQL 有两种锁：共享读锁，独占写锁。

在 5.7 及以前，用`select ... in share mode`获取共享锁，8.0 后改为`select for share`，兼容以前的。
用`select for update`获取独占锁。如遇到锁等待，默认等 50s，可以在 update 后加两个选项：skip locked 和 nowait。

innoDb 行锁分 3 种：

-   record lock：对索引项加锁
-   gap lock：对索引间的间隙加锁
-   next-key lock：前两种的组合，对记录及前面的间隙加锁

-   这意味着如果不通过索引检索数据，那么 innoDB 将对表中所有数据加锁，和锁表一样！！

-   由于 MySQL 的行锁是针对索引加的锁，不是针对记录加的锁，所以虽然是访问不同的记录，但是如果是使用的相同的索引键，就会出现锁冲突。

-   当表有多个索引的时候，不同的事务可以使用不同的索引锁定不同的行

-   即便在条件中使用了索引字段，但是否使用索引来检索数据是由 MySQL 通过判断不同执行计划的代价来决定的。

当使用范围条件检索数据，并请求共享或排他锁时，innoDB 会给符合条件的数据记录的索引项加锁，对于键值在条件范围内但并不存在的记录，叫做间隙，innoDB 也会对这个间隙加锁，这就是`next-key`锁。
除了通过范围条件外，如果视同相等条件给一个不存在的记录加锁，innoDB 也会使用 next-key 锁。

next-key 可以解决幻读。

insert ... select 和 create table ... select 可能会阻止对源表的并发更新，MySQL 将这种 sql 称为不确定的 sql， 属于“Unsafe SQL”，不推荐使用。

可以通过"select \* from source into outfile" 和"load data infile ..."语句来组合间接实现。或使用基于行的 BINLOG 格式和基于行数据的复制。

### 关于死锁

两个事务都需要获取对方持有的排他锁才能继续完成事务，这种循环锁等待就是典型的死锁。
发生死锁后，innoDB 一般都能自动检测到，并使一个事务释放锁并回退。当涉及外部锁或表锁，innoDB 并不能完全自动检测到死锁，需要设置超时等待参数 innodb_lock_wait_timeout 来解决。

通常来说，死锁都是应用设计的问题

-   在应用中，如果不同的程序会并发读取多个表，应尽量约定以相同的顺序来访问表，可以大大降低死锁的概率。
-   在程序以批量方式处理数据的时候，如果实现对数据排序，保证每个线程按固定的顺序来处理记录，也可以大大降低死锁的概率。
-   在事务中，如果要更新记录，应直接申请排他锁，而不应先申请共享锁再申请排他锁

用 `show innodb status` 分析死锁的原因

innoDB 采用 redo log 机制来保证事务更新的一致性和持久性。

当更新数据时，innoDB 内部的操作流程大致如下：

1. 将数据读入 innoDB buffer pool，并对 相关记录加独占锁
2. 将 undo 信息写入 undo 表空间的回滚段中
3. 更改缓存页中的数据，并将更新记录写入 redo buffer 中
4. 提交时，根据 innodb_flush_log_at_trx_commit 的设置，用不同的方式将 redo buffer 中的更新记录刷新到 innoDB rego log file 中，然后释放独占锁
5. 最后，后台 IO 线程根据需要择机将缓存中更新的数据刷新到磁盘文件中

set persist/set persist only 可以持久化参数设置，通过 reset persist 来清除某个配置

