

# SQL语言四大类DQL、DDL、DML、DCL
SQL语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

- **DQL（Data Query Languages）**：数据查询语言，是由SELECT子句，FROM子句，WHERE子句组成的查询块：SELECT <字段名表> FROM <表或视图名> WHERE <查询条件>。

- **DDL（Data Definition Languages）语句**：数据定义语言，这些语句定义了不同的数据段、数据库、表、列、索引等数据库对象的定义。常用的语句关键字主要包括 **CREATE**、**DROP**、**ALTER**等。

- **DML（Data Manipulation Language）语句**：数据操纵语句，用于添加、删除、更新和查询数据库记录，并检查数据完整性，常用的语句关键字主要包括 **INSERT**、**DELETE**和**UPDATE**

- **DCL（Data Control Language）语句**：数据控制语句，用于控制不同数据段直接的许可和访问级别的语句。这些语句定义了数据库、表、字段、用户的访问权限和安全级别。主要的语句关键字包括 **GRANT**、**REVOKE** 等。


# DQL（数据查询语言）

```sql
# [SELECT、WHERE - 选取数据]
# 1.* 表示所有的列
SELECT * FROM employee_table;
# 2.指定返回的列名
SELECT name,age,salary FROM employee_table;

# [WHERE - 条件过滤]
# 1. 操作符（>=：大于等于, <=：小于等于, !=：不等于, <>：不等于）
SELECT * FROM employee_table WHERE age > 35;
# 2. 空值检查（IS NOT NULL）
SELECT dept,name,salary,education FROM employee_table WHERE education IS NOT NULL;

# [ORDER BY - 排序]
# 1. ORDER BY（不写时默认是升序排序）：按照年龄升序排序
SELECT * FROM employee_table ORDER BY age;
# 2. ORDER BY 指定多个字段的排序方式：年龄升序（ASC）排序，薪水降序（DESC）排序
SELECT dept,name,age,salary FROM employee_table ORDER BY age ASC, salary DESC;

# [LIKE - 通配符匹配]
# 1. '%'百分号匹配任意个字符
SELECT dept,name,age,salary FROM employee_table WHERE name LIKE '张%';
# 2. '_'下划线匹配单个字符
SELECT dept,name,age,salary FROM employee_table WHERE name LIKE '_力给';

# [LIMIT - 限制返回的结果集范围]
# 1. LIMIT(N)：只返回前10行数据
SELECT * FROM employee_table LIMIT 10;
# 2. LIMIT(M,N)：从第10行开始的10行数据
SELECT * FROM employee_table LIMIT 10, 10;

# [IN - 范围查询]
# 1. IN：包括后面的列表项。IN后面可接子查询
SELECT * FROM article WHERE status IN (1,2,3)
SELECT * FROM article WHERE uid IN (SELECT uid FROM user WHERE status=0)
# 2. NOT IN：前面加上 NOT 运算符时，表示不包括IN后的列表项
SELECT * FROM article WHERE status NOT IN (-1,0)
```



# DDL（数据定义语言）

```sql
# 创建表
CREATE TABLE table_name
(
column_1 dataType1,
column_2 dataType2,
column_3 dataType3,
....
)

# 在表中新增列
ALTER TABLE table_name ADD COLUMN column_name dataType;

# 在表中修改列
ALTER TABLE table_name MODIFY COLUMN column_name dataType;

# 删除表
DROP TABLE table_name;

# 删除数据库
DROP DATABASE database_name;

# 清空表
TRUNCATE TABLE table_name;
```

# DML（数据操纵语句）

## INSERT

INSERT INTO 语句用于向表格中插入新的行。

```sql
# 插入的这行的数据必须与表结构的列个数、列顺序一致
INSERT INTO table_name VALUES (值1, 值2,....)
# 指定所要插入数据的列
INSERT INTO table_name (column1, column2,...) VALUES (值1, 值2,....)
```

## UPDATE

Update 语句用于修改表中的数据。

```sql
# 基本语法：UPDATE table_name SET column_name1 = 新值 WHERE column_name2 = 某值
UPDATE person SET active = 1 WHERE name_id = 'LuckinJack';

# 更新一行中的若干列，用逗号“,”隔开
UPDATE person SET age = '23', City = 'Nanjing' WHERE name_id = 'LuckinJack';
```

## DELETE

DELETE 语句用于删除表中的行。

```sql
# 基本语法：DELETE FROM table_name WHERE column_name = 值
DELETE person WHERE name_id = 'LuckinJack';

```

# DCL（数据控制语句）

## GRANT 

GRANT 语句可以为用户设置权限。

```sql
# 使用 GRANT 语句创建一个新用户 user1，密码为 password。用户 user1 对所有的数据有查询、插入权限，并授予 GRANT 权限
GRANT SELECT,INSERT ON *.* TO 'user1'@'localhost' IDENTIFIED BY 'password'

# 查看 user1用户的权限
SHOW GRANTS FOR 'user1'@'localhost';
```
## REVOKE

使用 REVOKE 语句删除某个用户的某些权限（此用户不会被删除）。

```sql
# 删除用户某些特定的权限：删除user1用户的插入权限
REVOKE INSERT ON *.* FROM 'user1'@'localhost'

# 删除特定用户的所有权限：删除user1用户的所有权限
REVOKE ALL PRIVILEGES FROM 'user1'@'localhost'

# 查看 user1用户的权限
SHOW GRANTS FOR 'user1'@'localhost';
```

