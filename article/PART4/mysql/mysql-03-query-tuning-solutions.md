# 使用视图

## 什么是视图？
> **[!NOTE]**
>
> 1. 视图是一张虚拟的表，它本身不存储也不包含数据，但和数据库的表存在着对应的关系，行和列的数据来自于定义视图的查询中所使用的表，并且还是在查询视图时（`SELECT * FROM <视图名>`）去查表后动态生成的。
> 2. 视图可用来简化数据处理以及重新格式化基础数据或保护基础数据。
> 3. 视图的名称不能与其他的视图名以及表名相同。
> 4. 不能在 MySQL 的视图上创建索引，只能在其基表上创建索引。

❑ 视图的优点如下：
 - 重用 SQL 语句。
 - 更改数据格式，简化数据操作。
 - 定制用户数据，聚焦特定数据。
 - 提高数据安全性。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。

❑ 视图同样也有缺点：
- 性能一般。用视图查询数据的效率并不比其对应的 SQL 高：① MySQL 不支持在视图上创建索引，② 如果视图与视图之间存在嵌套，查询效率会很低。
- 独立性差。与基表耦合，每当更改与视图相关联的基表的表结构时，都必须更改视图。


## 视图操作：创建、查询、修改、删除

- 创建视图：`CREATE VIEW <视图名> AS <SELECT语句>`

- 查询视图：`SELECT * FROM <视图名>`

- 修改视图：`CREATE OR REPLACE VIEW <视图名> AS <SELECT语句>`

- 删除视图：`DROP VIEW <视图名>`

```sql
# 创建视图（全表视图）
CREATE VIEW view_article_info 
AS SELECT * FROM t_article_info;

# 删除视图
DROP VIEW view_article_info;

# 创建视图（指定部分列创建视图，两者的对应列数应该相同）
CREATE VIEW view_article_info(a_id,a_title,a_content,a_author) 
AS SELECT (id, title, content_str, author_name) FROM t_article_info;

# 查询视图
SELECT a_title, a_author FROM view_article_info

# 修改视图
CREATE OR REPLACE VIEW view_article_info(a_id,a_upload_time,a_click_count) 
AS SELECT (id,create_time,user_click_count) FROM t_article_info;
```

视图是可更新的，虽然视图本身不包含数据，但可对它们使用`INSERT`、`UPDATE`和`DELETE`，更新一个视图将更新其基表。
如果你对视图增加或删除行，实际上是对其基表增加或删除行。当然在一般情况下，**最好仅将视图作为查询数据的虚拟表，不推荐用视图来更新基表的数据**。

> **[!WARNING]**
> 如果视图包含下述结构中的任何一种，那么它就是不可更新的：
>
> - 聚合函数（`SUM()`, `MIN()`, `MAX()`, `COUNT()`等）。
> - `DISTINCT`
> - `GROUP BY`
> - `HAVING`
> - `UNION`或`UNION ALL`
> - 位于选择列表中的子查询
> - `JOIN`
> - FROM子句中的不可更新视图
> - WHERE子句中的子查询，引用FROM子句中的表。
> - 仅引用文字值（在该情况下，没有要更新的基本表）。
> - `ALGORITHM = TEMPTABLE`（使用临时表总会使视图成为不可更新的）
>

最后，**不建议 MySQL 上使用视图，尽量转化为普通查询的方式来获取对应需要的数据**（对表进行 ETL，再在 ETL 产生的派生表上进行查询）。

# 使用索引

## 什么是索引

## 索引操作：创建、修改、删除

##  索引失效的场景

### or连接的查询条件并未都加上索引

 否则索引失效，全表扫描

### 复合索引未使用最左列字段

 复合索引（又叫多列索引或者联合索引），必须使用复合索引的第一列字段作为Where后的查询条件，否则索引失效。

 例如，表table 的复合索引为（age，year，state）

```SQL
 SELECT * FROM table WHERE age = 18                   -- 会使用索引
 SELECT * FROM table WHERE age = 18 AND year = 2002   -- 会使用索引
 SELECT * FROM table WHERE year = 2002                -- 不会使用索引
 SELECT * FROM table WHERE year = 2002 AND state = 1  -- 不会使用索引
```
### LIKE 以 % 开头

 例如，表table 的索引为 （name）

```SQL
 SELECT * FROM table WHERE name LIKE 'LuckInJack%'  -- 会使用索引
 SELECT * FROM table WHERE name LIKE '%LuckInJack%' -- 不会使用索引
 SELECT * FROM table WHERE name LIKE '%LuckInJack'  -- 不会使用索引
```

### 字段存在类型转换


### where后的索引列有运算


### where后的索引列使用了函数

```SQL
YEAR(t.createTime)
```

### 数据量少，全表扫描比走索引更快时

### 在非where 和 on 条件下使用索引字段

 在where和on后面才可以利用到索引，在`case when xxx then `中使用无效

### 对派生表进行条件查询或连接查询

 对原表取索引字段生成的中间表进行查询时，将不会使用索引

 从大表（几十个字段）中取几个关键字段转化为派生表再进行查询，理论上看，查的字段减少提高了效率和速度，但是派生表索引失效，整体速度是严重下降的


### 使用 is null 或 is not null

 建表时把需要索引的列定义为非空(not null)


# 定位低效率SQL并优化

## SQL缓慢的常见原因

### 无索引或索引失效

### 锁等待

### SQL语句写法不恰当

1. 张大宽表中直接使用`SELECT *`：应只查询关键字段，否则一次查询出的数据量过大，占用数据库IO太多，响应会十分缓慢。
2. MySQL常用的存储引擎有  `InnoDB`  和  `MyISAM`，前者支持行锁和表锁，后者只支持表锁。



## 定位

我们可以通过慢查询日志、`SHOW PROCESSLIST`来定位执行效率较低的SQL语句。

找到执行效率低的SQL语句后，就可以通过`SHOW PROFILE FOR QUERY`、`EXPLAIN`或`trace`等方法来优化这些SQL语句

### 慢查询日志

可以通过慢查询日志定位那些已经执行完毕的SQL语句。

### SHOW PROCESSLIST

慢查询日志在查询结束以后才记录，所以，在应用反应执行效率出现问题的时候查询慢查询日志并不能定位问题。

此时，可以使用`SHOW PROCESSLIST`命令查看当前MySQL正在进行的线程，包括线程的状态、是否锁表等，可以实时地查看SQL的执行情况，同时对一些锁表操作进行优化。




## 分析

### EXPLAIN

通过`EXPLAIN`，我们可以查看到如下的信息

- 表之间的引用
- 哪些索引被使用
- 哪些索引可以被使用

> **[!NOTE]**
> - **id**：每个执行计划都有一个 id，如果是一个联合查询，这里还将有多个 id。
> 
> - **select_type**：表示 SELECT 查询类型，常见的有以下几种。
>
> 	- `SIMPLE`：普通查询，即没有联合查询、子查询
>
> 	- `PRIMARY`：主查询
>	
> 	- `UNION`：UNION 中后面的查询
>
> 	- `SUBQUERY`：子查询
> 
> - **table**：当前执行计划查询的表，如果给表起别名了，则显示别名信息。
> 
> - **partitions**：访问的分区表信息。
> 
> - **type**：表示从表中查询到行所执行的方式，查询方式是 SQL 优化中一个很重要的指标，结果值从好到差依次是：`system` > `const` > `eq_ref` > `ref` > `range` > `index` > `ALL`。
> 
> 	- `system/const`：表中只有一行数据匹配，此时根据索引查询一次就能找到对应的数据。
>
> 	- `eq_ref`：使用唯一索引扫描，常见于多表连接中使用主键和唯一索引作为关联条件
>
> 	- `ref`：非唯一索引扫描，还可见于唯一索引最左原则匹配扫描。
>
> 	- `range`：索引范围扫描，比如，`<`，`>`，`between` 等操作。
>
> 	- `index`：索引全表扫描，此时遍历整个索引树。
>
> 	- `ALL`：表示全表扫描，需要遍历全表来找到对应的行。
>
> - **possible_keys**：可能使用到的索引。
> 
> - **key**：实际使用到的索引。
> 
> - **key_len**：当前使用的索引的长度。
> 
> - **ref**：关联 id 等信息。
> 
> - **rows**：查找到记录所扫描的行数。
> 
> - **filtered**：查找到所需记录占总扫描记录数的比例。
> 
> - **Extra**：额外的信息。

### `SHOW PROFILE FOR QUERY`

### `trace`
