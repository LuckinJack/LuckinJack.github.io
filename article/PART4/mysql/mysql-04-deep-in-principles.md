
# 事务

## 事务基本要素：ACID

事务是由一组SQL语句组成的逻辑处理单元，具有4个属性，通常简称为事务的ACID属性。

❑ **A (Atomicity) 原子性**：整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。
- 事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

❑ **C (Consistency) 一致性**：在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。

❑ **I (Isolation) 隔离性**：一个事务的执行不能其它事务干扰。
- 即一个事务内部的操作及使用的数据对其它并发事务是隔离的，并发执行的各个事务之间不能互相干扰。

❑ **D (Durability) 持久性**：在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

## 并发事务处理带来的问题

- **更新丢失(Lost Update)**：事务A和事务B选择同一行，然后基于最初选定的值更新该行时，由于两个事务都不知道彼此的存在，就会发生丢失更新问题。
- **脏读(Dirty Reads)**：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据。
- **不可重复读（Non-Repeatable Reads)**：事务 A 多次读取同一数据，事务B在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果不一致。
- **幻读（Phantom Reads)**：幻读与不可重复读类似。它发生在一个事务A读取了几行数据，接着另一个并发事务B插入了一些数据时。在随后的查询中，事务A就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。

## 事务隔离级别

> **[!NOTE]**
> 查看当前数据库的事务隔离级别的命令：
> 
> `show variables like 'tx_isolation';`

数据库标准的事务的隔离级别有4种，由低到高分别为：

❑ **READ-UNCOMMITTED(读未提交)**： 最低的隔离级别，允许一个事务可以读取另一个未提交事务的数据。
- 可能会导致脏读、幻读或不可重复读。

❑ **READ-COMMITTED(读已提交)**： 允许一个事务读取另一个事务已经提交的数据。
- 可以阻止脏读，但是幻读或不可重复读仍有可能发生。

❑ **REPEATABLE-READ(可重复读)**： 开始读取数据（事务开启）时，不再允许修改操作。
- 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

❑ **SERIALIZABLE(可串行化)**： 最高的隔离级别，完全服从ACID的隔离级别。
- 事务在读操作时，先加表级别的共享锁，直到事务结束才释放
- 事务在写操作时，先加表级别的排它锁，直到事务结束才释放
- 所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，该隔离级别可以防止脏读、不可重复读以及幻读。

数据库的事务隔离越严格，并发副作用越小，但付出的代价就越大，因为事务隔离实质上就是使事务在一定程度上“串行化”进行，而“串行化”可能导致大量的超时和锁竞争问题，这显然与“并发”是矛盾的。同时，不同的应用对读一致性和事务隔离程度的要求也是不同的，比如许多应用对“不可重复读”和“幻读”并不敏感，可能更关心数据并发访问的能力。


> **[!WARNING]**
> ❑ **REPEATABLE-READ（可重读）** 是存储引擎 ** InnoDB 的默认隔离级别**。
>
> ❑ InnoDB 存储引擎，与基于 SQL 标准（比如SQL Server等）实现的其他数据库引擎有一些不同：
>
> - InnoDB 存储引擎在 **REPEATABLE-READ（可重读）** 事务隔离级别下使用了 `Next-Key Lock` 算法，可以避免幻读的产生。
> 也就是说 InnoDB 存储引擎的默认隔离级别 **REPEATABLE-READ（可重读）** 已经可以完全保证事务的隔离性要求，达到了 SQL标准的 **SERIALIZABLE(可串行化) ** 隔离级别，而且保留了比较好的并发性能。
> - `Next-Key Lock` 算法避免幻读的方式是：锁住该记录行本身，也锁住一段范围的行，锁住的地方其他事务无法插入和删除数据，从而保证在where的条件下读到的数据是一致的。



## 事务操作语句

下面是关于事务处理需要知道的几个术语：

❑ **事务（transaction）** ：一组SQL语句。
- 一般的MySQL语句都是直接针对数据库表执行和编写，提交（写或保存）操作是隐式自动进行的

❑ **回滚（rollback）** ：撤销指定SQL语句的过程。
- 可回滚INSERT、UPDATE和DELETE语句。不能回滚SELECT语句
- 当COMMIT或ROLLBACK语句执行后，事务会自动关闭

❑ **提交（commit）** ：将未存储的SQL语句结果写入数据库表；
- 当COMMIT或ROLLBACK语句执行后，事务会自动关闭

❑ **保存点（savepoint）** ：事务处理中设置的临时占位符（place-holder），你可以对它发布回滚
- 保存点会因为提交事务和新开事务而失效，保存点只能使用在当前事务中
- 一个保存点如果不被删除，则可以存在一个事务中被多次使用
- 同名保存点会覆盖

具体的用法如下：

```sql
# 查询一张有数据的表trade_records
SELECT * FROM trade_records;
# 开启事务
START TRANSACTIONAL;
# 删除trade_records的所有数据
DELETE FROM trade_records;
# 再次查询trade_records无数据
SELECT * FROM trade_records;
# 回滚
ROLLBACK;
# 回滚后trade_records又能查出数据了
SELECT * FROM trade_records;
```

```sql
# 开启事务
START TRANSACTIONAL;
# 查询一张有数据的表trade_records
SELECT * FROM trade_records;
# 删除数据
DELETE FROM trade_records WHERE id = 123456;
# 标记保存点delete1
SAVEPOINT delete1;
# 删除数据
DELETE FROM trade_records WHERE id = 654321;
# 标记保存点delete2
SAVEPOINT delete2;
# 回滚到保存点delete1
ROLLBACK TO delete1;
# 再次查询，仅id = 123456的这条数据不见了
SELECT * FROM trade_records;
# 删除数据
DELETE FROM trade_records WHERE id = 111111;
# 删除保存点delete2，删除后该保存点不能再使用
RELEASE SAVEPOINT delete2;
# 再次回滚到保存点delete1（delete1保存点在同一事务中可复用）
ROLLBACK TO delete1;
# 再次查询，仅id = 123456的这条数据不见了
SELECT * FROM trade_records;
# 提交，所有保存点失效
COMMIT;
# 这时再回滚会报错
ROLLBACK TO delete1;
```

## 事务的实现

事务的实现是基于数据库的存储引擎。不同的存储引擎对事务的支持程度不一样。MySQL 中支持事务的存储引擎有 InnoDB 和 NDB。

事务的实现就是如何实现ACID特性。

事务的隔离性是通过锁实现，而事务的原子性、一致性和持久性则是通过事务日志实现。

事务日志包括：**重做日志redo** 和 **回滚日志undo** 。

那么事务日志是如何实现事务的原子性、一致性和持久性的？

> **[!NOTE]**
> ❑ **redo log（重做日志）** ：实现持久化和原子性
>
> - 在InnoDB存储引擎中，事务日志通过重做(redo)日志和innoDB存储引擎的 **日志缓冲(InnoDB Log Buffer)** 实现。事务开启时，事务中的操作，都会先写入存储引擎的日志缓冲中，在事务提交之前，这些缓冲的日志都需要提前刷新到磁盘上持久化，这就是DBA们口中常说的 **日志先行(Write-Ahead Logging)** 。当事务提交之后，在 *Buffer Pool* 中映射的数据文件才会慢慢刷新到磁盘。此时如果数据库崩溃或者宕机，那么当系统重启进行恢复时，就可以根据redo log中记录的日志，把数据库恢复到崩溃前的一个状态。未完成的事务，可以继续提交，也可以选择回滚，这基于恢复的策略而定。
> - 在系统启动的时候，就已经为redo log分配了一块连续的存储空间，以顺序追加的方式记录Redo Log，通过顺序IO来改善性能。所有的事务共享redo log的存储空间，它们的Redo Log按语句的执行顺序，依次交替的记录在一起。
>
> ❑ **undo log（回滚日志）** ：实现一致性
> - undo log 主要为事务的回滚服务。在事务执行的过程中，除了记录redo log，还会记录一定量的undo log。undo log记录了数据在每个操作前的状态，如果事务执行过程中需要回滚，就可以根据undo log进行回滚操作。单个事务的回滚，只会回滚当前事务做的操作，并不会影响到其他的事务做的操作。
> - Undo记录的是已部分完成并且写入硬盘的未完成的事务，默认情况下回滚日志是记录下表空间中的（共享表空间或者独享表空间）


# 存储过程






# MySQL引擎

MySQL的存储引擎有 `InnoDB`、`MyISAM`、`Memory`、`NDB`等等，这里我们只讨论最常用的两种：`InnoDB`、`MyISAM`。

## InnoDB 与 MyISAM的对比

> **[!NOTE]**
> - `InnoDB` 是一个可靠的事务处理引擎， MySQL 目前（≥ 5.5）默认的存储引擎，支持**事务、行级锁定和外键**。
> - `MyISAM` 是一个性能极高的引擎，支持全文本搜索，但**不支持事务**。

可以通过下面这个表格了解两者各自的特点：

引擎 |	MyISAM	 | InnoDB
---|---|---
主外键 |	不支持	| 支持
事务 |	不支持|	支持
行表锁|	表锁，即使操作一条记录也会锁住整个表，不适合高并发的操作|	行锁,操作时只锁某一行，不对其它行有影响，适合高并发的操作
缓存|	只缓存索引，不缓存真实数据	| 不仅缓存索引还要缓存真实数据，对内存要求较高，而且内存大小对性能有决定性的影响
表空间	|小	|大
关注点	|性能	|事务
默认安装	|是	|是


- **InnoDB 支持事务，MyISAM 不支持事务**。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一。

- **InnoDB 支持外键，而 MyISAM 不支持**。对一个包含外键的 InnoDB 表转为 MyISAM 会失败。

- **InnoDB 是聚簇索引，MyISAM 是非聚簇索引**。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。

- **InnoDB 不保存表的具体行数，MyISAM 保存了表具体行数**。InnoDB 在执行 `select count(*) from table` 时需要全表扫描。而 MyISAM 执行上述语句时只需要读出保存了表具体行数的变量，速度更快。

- **InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁**。在 MyISAM 下执行一个更新语句会锁住整张表，导致其他查询和更新都会被阻塞，因此并发访问受限。这也是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一。



## 设置和查看存储引擎

设置和查看存储引擎的命令如下：

```sql
# 查看支持的存储引擎
SHOW ENGINES;

# 查看默认存储引擎
SHOW VARIABLES LIKE 'storage_engine';

# 查看某个数据库中的某一表所使用的存储引擎
show table status like 'tablename';
show table status from database where name="tablename";

# 建表时指定存储引擎。默认的就是INNODB，不需要设置
CREATE TABLE t1 (i INT) ENGINE = INNODB;
CREATE TABLE t2 (i INT) ENGINE = CSV;
CREATE TABLE t3 (i INT) ENGINE = MEMORY;

# 修改存储引擎
ALTER TABLE t ENGINE = InnoDB;

# 修改默认存储引擎，也可以在配置文件my.cnf中修改默认引擎
SET default_storage_engine=NDBCLUSTER;
```

> **[!WARNING]**
> 默认情况下，每当 `CREATE TABLE` 或 `ALTER TABLE` 不能使用默认存储引擎时，都会生成一个警告。
> 为了防止在所需的引擎不可用时出现意外的错误操作，可以启用 `NO_ENGINE_SUBSTITUTION SQL` 模式。如果所需的引擎不可用，则此设置将产生错误而不是警告，并且不会创建或更改表。

## InnoDB锁机制

InnoDB有三种行锁的算法：

> **[!NOTE]**
> ❑  **记录锁(Record Locks)**：单个行记录上的锁
>
> - 对索引项加锁，锁定符合条件的行
> - 通过 主键索引 或 唯一索引 对数据行进行 UPDATE 操作时，也会对该行数据加记录锁
> ```sql
> # id 列为主键列或唯一索引列
> UPDATE trade_records SET price = 7895 WHERE id = 1;
> ```
>
> ❑  **间隙锁（Gap Locks）**：锁定一个范围，但不包括记录本身
> - 使用间隙锁锁住的是一个开区间，而不仅仅是这个区间中的每一条数据
> - 间隙锁基于非唯一索引，它锁定一段范围内的索引记录
>
> ❑  **临键锁(Next-key Locks)**：：临键锁，是记录锁与间隙锁的组合，它的封锁范围，既包含索引记录，又包含索引区间
> - 每个数据行上的非唯一索引列上都会存在一把临键锁，当某个事务持有该数据行的临键锁时，会锁住一段左开右闭区间的数据
> - 临键锁的主要目的，也是为了避免 **幻读(Phantom Read)** 。如果把事务的隔离级别降级为 **READ-COMMITTED(读已提交)** ，临键锁则也会失效
> - InnoDB 中行级锁是基于索引实现的，临键锁只与非唯一索引列有关，在唯一索引列（包括主键列）上不存在临键锁



# MySQL的四种日志

日志是 MySQL数据库的重要组成部分，记录着数据库运行期间各种状态信息。MySQL 日志主要包括错误日志、查询日志、慢查询日志、事务日志、二进制日志几大类，这里我们重点介绍四种日志

- 二进制日志（binlog）
- 事务日志（包括redo log 和 undo log）
- 中继日志（relay log）



##  Binary Log
> **[!NOTE]**
> *Binary Log* 也就是常说的 **bin-log（二进制日志）** 。
>
> - 对MySQL数据库执行写入性操作（不包括查询）以二进制日志文件保存在磁盘中
> - 属于Server 层的日志
>
> ❑  **Binary Log的主要用途**
>
> - **备份恢复**：在某个时间点a做了一次备份，然后利用binary log记录从这个时间点a后的所有数据库的改动，然后下一次还原的时候，利用时间点a的备份文件和这个binary log文件，就可以将数据还原。
> - **主从复制（Replication）**：在master端开启binary log后，binary log会记录所有数据库的改动，然后slave端获取这个Log文件内容就可以在slave端进行同样的操作。

### Binary Log日志结构



### Binary Log参数说明





## Relay Log

*Relay  log* 中继日志是连接 master 和 slave 的核心。

- 从服务器I/O线程将主服务器的二进制日志读取过来记录到从服务器本地文件，然后从服务器SQL线程会读取relay-log日志的内容并应用到从服务器，从而使从服务器和主服务器的数据保持一致



1）主服务器将更新写入二进制日志文件，并维护文件的一个索引以跟踪日志循环。

2）从库复制主库的二进制日志事件到本地的中继日志（relay log）。

3）从库重放中继日志。



### Relay Log日志结构



### Relay Log参数说明

```sql
mysql > show variables like '%relay%';
 
/* 结果 */
+---------------------------+----------------------------------+
| Variable_name             | Value                            |
+---------------------------+----------------------------------+
| max_relay_log_size        | 0                                |
| relay_log                 | relay-mysql                      |
| relay_log_basename        | /var/lib/mysql/relay-mysql       |
| relay_log_index           | /var/lib/mysql/relay-mysql.index |
| relay_log_info_file       | relay-log.info                   |
| relay_log_info_repository | FILE                             |
| relay_log_purge           | ON                               |
| relay_log_recovery        | ON                               |
| relay_log_space_limit     | 0                                |
| sync_relay_log            | 10000                            |
| sync_relay_log_info       | 10000                            |
+---------------------------+----------------------------------+
```





##  Redo Log

❑  

*Redo log* 是重做日志文件，属于 InnoDB 存储引擎层的日志，redo log 是循环写的，redo log 不是记录数据页更新之后的状态，而是记录这个页做了什么改动

❑  主要用途

##  Undo Log






# 参考

- [MySQL Innodb 中的锁](https://zhuanlan.zhihu.com/p/31875702)

- [MySQL 中继日志](https://blog.csdn.net/mshxuyi/article/details/100652769)