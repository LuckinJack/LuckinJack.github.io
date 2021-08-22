# 常用函数

## 流程控制函数

### IF()

- 功能：表达式 expr 得到不同的结果，当 expr 为真是返回 v1 的值，否则返回 v2
- 语法：`IF(expr, v1, v2)`

```sql
-- 结果为 yes
SELECT IF(1<2, 'yes', 'no');
-- 结果为 no
SELECT IF(STRCMP('abc','abcd'), 'yes', 'no');
-- 结果为 空字串
SELECT IF(LENGTH(TRIM(NULL))>0, '非空字串', '空字串');
```
### IFNULL()

- 功能：如果 v1 不为 NULL，则 IFNULL 函数返回 v1；否则返回 v2 
- 语法：`IFNULL(v1, v2)`
- 示例：

```sql
-- 结果为 8
SELECT IFNULL(0, 8);
-- 结果为 8
SELECT IFNULL(NULL, 8);
-- 结果为 8
SELECT IFNULL(1/0, 8);
```

### CASE WHEN

- 功能：除了 IF 函数，MySQL 还提供了一个替代的条件语句 CASE WHEN。CASE 语句有两种形式，简单函数和搜索函数。

- 语法：

  - 简单函数 - 枚举这个字段所有可能的值

    ```sql
    CASE case_value
    WHEN when_value THEN statement_list
    [WHEN when_value THEN statement_list] ...
    [ELSE statement_list]
    END CASE
    ```

  - 搜索函数 - 搜索函数可以写判断，并且搜索函数只会返回第一个符合条件的值，其他case被忽略

    ```sql
    CASE
    WHEN search_condition THEN statement_list
    [WHEN search_condition THEN statement_list] ...
    [ELSE statement_list]
END CASE
    CASE WHEN [expr] THEN [result1] ... ELSE [default] END
    ```
- 示例：
  ```sql
  -- 结果为 星期一
  SELECT CASE WEEKDAY(NOW()) 
  WHEN 0 THEN '星期一' WHEN 1 THEN '星期二' WHEN 2 THEN '星期三' 
  WHEN 3 THEN '星期四' WHEN 4 THEN '星期五' WHEN 5 THEN '星期六'
  ELSE '星期天' END AS today;
  /*
  查看各个ip下的query查询有多慢，结果为：
  ip            slowness  
  ------------  ----------     
  192.168.2.25  0
  192.168.2.26  10      
  192.168.2.27  110                  
  */
  SELECT SUBSTRING_INDEX(a.HOST,':', 1) AS ip, 
  SUM(CASE
  WHEN a.INFO IS NOT NULL AND a.TIME > 2 AND a.TIME <= 5 THEN 1
  WHEN a.INFO IS NOT NULL AND a.TIME > 5 AND a.TIME <= 8 THEN 10
  WHEN a.INFO IS NOT NULL AND a.TIME > 8 THEN 100
  ELSE 0 END) AS slowness
  FROM information_schema.processlist a GROUP BY SUBSTRING_INDEX(a.HOST,':', 1)
  ```
  
  

## 聚合函数

### MAX()



### MIN()



### COUNT()



### SUM()



### AVG()



## 数值操作

### ABS()



### CEIL()



### FLOOR()



### MOD()



### RAND()



### ROUND()



### TRUNCATE()



## 字符串函数
https://dev.mysql.com/doc/refman/8.0/en/string-functions.html

### CONCAT()

- 功能：将多个字符串连接成一个字符串。
- 语法：`concat(str1, str2,...)`

> **[!TIP]**
>
> `concat()`连接的字符串中，如果出现null，整个返回结果就为null。

### CONCAT_WS()

- 功能：用制定的分隔符，将多个字符串连接成一个字符串。
- 语法：`concat_ws(seperator, str1, str2,...)`

> **[!TIP]**
> 
> `concat_ws()`连接的字符串中，如果出现null，则忽略它不拼接到结果集中。

### GROUP_CONCAT()

- 功能：将group by产生的同一个分组中的值连接起来，返回一个字符串结果。

- 语法：`group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )`

> **[!TIP]**
>
> **1.** `group_concat()`会过滤掉NULL值。
>
> **2.** `group_concat()`有默认长度限制，当前长度限制可以这样查出：
>
> ```sql
> SELECT @@global. group_concat_max_len;
> ```
>
> 默认长度为1024，超出部分将被丢弃，不过我们可以对它的默认长度进行修改，这里有2种方法：
>
>    - 1）执行如下语句设置作用范围（在MySQL重启前一直有效）
>
>     `SET GLOBAL group_concat_max_len=1024000;`（全局group_concat长度）
>    
>     `SET SESSION group_concat_max_len = 1024000;`（当前session的group_concat长度）
>
>    - 2）在MySQL配置文件中添加一行以下语句（全局，MySQL重启后永久生效）
>
>    `group_concat_max_len = 1024000 `
>
> **3.** 返回的字符串的默认分隔符为逗号","，若要改为其他分隔符，可使用`SEPARATOR`子句指定
>
> ```sql
> ## GROUP_CONCAT设置指定分隔符
> SELECT month,
> GROUP_CONCAT(article SEPARATOR '|') AS articles
> FROM user_table 
> WHERE author = 'luckinjack'
> GROUP BY month
> ```

### LEFT()

- 功能：从左开始截取字符串。
- 语法：`left(被截取字符串, 截取长度)`
- 示例：

```sql
-- 结果为：www.yuan
SELECT LEFT('www.yuanrengu.com', 8)
```

### RIGHT()

- 功能：从右开始截取字符串。
- 语法：`right(被截取字符串, 截取长度)`
- 示例：

```sql
-- 结果为：gu.com
SELECT RIGHT('www.yuanrengu.com', 6)
```

### SUBSTRING()

- 功能：截取特定长度的字符串。
- 语法：`substring(被截取字符串， 从第几位开始截取)`、`substring(被截取字符串，从第几位开始截取，截取长度)`
- 示例：

```sql
## 从字符串的第9个字符开始读取直至结束
-- 结果为：rengu.com
SELECT SUBSTRING('www.yuanrengu.com', 9)

## 从字符串的第9个字符开始，只取3个字符
-- 结果为：ren
SELECT SUBSTRING('www.yuanrengu.com', 9, 3)

## 从字符串的倒数第6个字符开始截取，只取2个字符
-- 结果为 gu
SELECT SUBSTRING('www.yuanrengu.com', -6, 2)
```

### SUBSTRING_INDEX()

- 功能：按关键字进行截取。
- 语法：`substring_index(被截取字符串，关键字，关键字出现的次数)`
- 示例：

```sql
## 截取第二个“.”之前的所有字符
-- 结果为：www.yuanrengu
SELECT SUBSTRING_INDEX('www.yuanrengu.com', '.', 2)

## 截取倒数第二个“.”之后的所有字符
-- 结果为 yuanrengu.com
SELECT SUBSTRING_INDEX('www.yuanrengu.com', '.', -2)

## 如果关键字不存在，则返回整个字符串
-- 结果为：www.yuanrengu.com
SELECT SUBSTRING_INDEX('www.yuanrengu.com', 'sprite', 1);

```



### INSTR()

- 功能：获取子串第一次出现的下标位置，如果没有找到，则返回0（下标从1开始）。
- 语法：`instr(str, substr)`，`str`是到哪个字符串中去搜索，`substr`是要搜素的子字符串。

> **[!TIP]**
>
> 1. `INSTR()`函数不区分大小写
> 2. `INSTR('LuckinJack') > 0` 等效于使用运算符 `Like '%LuckinJack%'` 进行模糊查询



### FIND_IN_SET()

- 功能：获取字符串再字符串列表第一次出现的下标位置，如果没有找到，则返回0。
- 语法：`find_in_set(str, strlist)`，`strlist字符串列表` 就是由一些被逗号分隔开的字符串合并组成的单一字符串
- 示例：

```sql
SELECT FIND_IN_SET('1','1,2,3,4,5,6');    # 返回下标1
SELECT FIND_IN_SET('2','1,2,3,4,5,6');    # 返回下标2
SELECT FIND_IN_SET('1','');               # 第一个字符串不存在于第二个字符串列表中，所以返回0
SELECT FIND_IN_SET(null,'1,2,3,4,5,6');   # 任意一个参数为NULL，则返回值为 NULL
SELECT FIND_IN_SET('1',null);             # 任意一个参数为NULL，则返回值为 NULL
```

> [!NOTE]
>
> **注意区分 `FIND_IN_SET` 、 `IN` 、`INSTR`、`LIKE` 之间的区别**
>
> 





## 日期和时间函数

**MySQL 中最重要的内建日期函数**

函数| 描述
---|---
`NOW()`|	返回当前的日期和时间
`CURDATE()`	|返回当前的日期
`CURTIME()`|	返回当前的时间
`DATE()`|	提取日期或日期/时间表达式的日期部分
`EXTRACT()`	|返回日期/时间按的单独部分
`DATE_ADD()`|	给日期添加指定的时间间隔
`DATE_SUB()`| 从日期减去指定的时间间隔 
`DATEDIFF()`|	返回两个日期之间的天数
`DATE_FORMAT()`|	用不同的格式显示日期/时间

**MySQL Date 数据类型**

1. DATE - 格式 `YYYY-MM-DD`
2. DATETIME - 格式: `YYYY-MM-DD HH:MM:SS`
3. TIMESTAMP - 格式: `YYYY-MM-DD HH:MM:SS`
4. YEAR - 格式 `YYYY` 或 `YY`

Linux/Unix时间戳 转换为 MySQL时间日期类型：

UNIX_TIMESTAMP(NOW())                   //将mysql的datetime转换成linux/unix的时间戳；日期时间

UNIX_TIMESTAMP(DATE(NOW()))       //将mysql的date转换成linux/unix的日期。

UNIX_TIMESTAMP(TIME(NOW()))       
//将mysql的time转换成linux/unix的时间。(用问题)

FROM_UNIXTIME(time_t)          //将unix的时间戳转换成mysql的datetime；日期时间

DATE(FROM_UNIXTIME(time_t))          //日期

TIME(FROM_UNIXTIME(time_t))           //时间


- DATE_FORMAT(date,format)



## 转换函数和运算符

### convert()



### cast()





## 加密函数

### AES_ENCRYPT()



### AES_DECRYPT()



# MySQL 8 新增函数



## JSON函数

https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html



## 窗口函数

https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html




# 参考

- [MySQL函数大全，MySQL常用函数汇总 (biancheng.net)](http://c.biancheng.net/mysql/function/)

- [MySQL 8.0 参考手册 - 第 12 章函数和运算符](https://dev.mysql.com/doc/refman/8.0/en/functions.html)