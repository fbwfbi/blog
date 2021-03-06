# SQL 语句

本文记录目前可能需要用到的 SQL 语句。大部分摘抄自《MySQL 必知必会》。

## 使用 MySQL 数据库

### 连接到数据库服务器

可以用 `mysql` 命令行工具连接到 MySQL 数据库服务器。需要知道用户名、服务器 IP、端口号，可能还需要验证身份需要的密码。

```mysql
$ mysql -u 用户名 -p密码 -h ip地址
```

注意 -p 和密码之间没有空格。

也可以用有界面的工具，连接时需要填入的信息也差不多。

### 列出当前服务器中包含的数据库

```mysql
SHOW DATABASES
```

列出的库中有一些是 MySQL 内部使用的数据库，用来保存数据库、表、列、用户、权限等元信息。

### 选择要操作的数据库

```mysql
USE database_name
```

### 列出当前数据库中的表

```mysql
SHOW TABLES
```

### 列出表的所有字段

```mysql
SHOW COLUMNS FROM [tablename]
```

MySQL 支持用 `DESCRIBE` 作为 `SHOW COLUMS FROM` 的快捷命令。

### 查看创建数据库和表时使用的 SQL 语句

```mysql
SHOW CREATE DATABASE database_name
```

```mysql
SHOW CREATE TABLE table_name
```

## 创建数据库

```mysql
CREATE DATABASE base_name
```

## 删除数据库

```mysql
DROP DATABASE base_name
```

## 重命名表

```mysql
RENAME TABLE table_name TO new_name
```

或

```mysql
ALTER TABLE table_name RENAME [TO|AS] new_name
```

## 删除表

```mysql
DROP TABLE table_name
```

## 添加新字段

```mysql
ALTER TABLE [表名] ADD COLUMN `字段名` [数据类型] [约束] COMMENT '注释'
```

## 删除字段

```mysql
ALTER TABLE [表名] DROP COLUMN [字段名]
```

## 修改字段名

```mysql
ALTER TABLE [表名] CHANGE [原字段名] [新字段名] [数据类型] [约束] COMMENT '注释'
```

## 修改字段类型

```mysql
ALTER TABLE [表名] MODIFY COLUMN [字段名] 类型
```

## 添加索引

```mysql
ALTER TABLE [表名] ADD [INDEX | UNIQUE | PRIMARY KEY | FULLTEXT] [索引名](索引字段列)
```

UNIQUE 表示唯一索引，PRIMARY KEY 表示主键索引，FULLTEXT 表示全文索引。

## 删除索引名

```mysql
ALTER TABLE [表名] DROP INDEX [索引名]
```

## 删除 PRIMARY KEY

```mysql
ALTER TABLE [表名] DROP PRIMARY KEY
```

一个表只有一个 PRIMARY KEY，所以不需要知道主键名。

## 修改索引名

对于 MySQL 5.7 以下的版本，必须得先删除原索引，再添加新索引，以达到修改索引的目的。

```mysql
ALTER TABLE [表名] DROP INDEX [原索引名]
ALTER TABLE [表名] ADD INDEX [新索引名](索引字段列)
```

对于 MySQL 5.7 及以上的版本

```mysql
ALTER TABLE [表名] RENAME INDEX [原索引名] TO [新索引名]
```

## 查询数据

数据查询使用 `SELECT` 语句。`SELECT` 语句用于在指定的表格中查询记录，并可以指定想要查询的字段。格式如下:

```mysql
SELECT 字段1,字段2,... FROM 表名;
```

SELECT 后跟想要查询的字段名，如果要查询多个字段，则字段间用逗号隔开。表名表示要查询的表。

如果想要查询所有字段，则字段名用 `*` 代替，一般不建议这么做，最好指定所有想要查询的字段，这样即使数据库新增了字段，查询结果也不会出现应用不能解析的字段。

```mysql
SELECT * FROM 表名;
```

### 只列出不同的行

虽然同一张表中不存在所有字段都相同的多条记录，但使用 SELECT 查询时，如果只查询了部分字段，那查询到的结果完全有可能会出现重复的行。如果只想得到不同的行，可以使用 `DISTINCT` 关键字。格式如下：

```mysql
SELECT DISTINCT 字段名1,字段名2... FROM 表名;
```

使用 `DISTINCT` 关键字后，查询到的结果中不包含完全相同的两行记录，该关键字常用来统计某个字段所有不同值的个数。

### 限制结果

使用 `LIMIT` 关键字可以限制查询语句返回的行数。例如，下面的查询语句只返回匹配结果的前5行。

```mysql
SELECT 字段名1,字段名2,... FROM 表名 LIMIT 5;
```

可以继续查询后面的行，需要在 `LIMIT` 关键字后指定开始的行数和要查询的行数。例如下面语句将继续查询从第5行开始的10行记录。

```mysql
SELECT 字段名1,字段名2,... FROM 表名 LIMIT 5,10；
```

`LIMIT` 子句中的行下标从0开始。

### 使用完全限定名

`SELECT`语句中可以使用 `表名.列名` 的方式来指定要查询的字段。如下：

```mysql
SELECT 表名.列名 FROM 表名;
```

也可以使用 `数据库名.表名` 的方式指定要查询表。如下：

```mysql
SELECT 表名.列名 FROM 数据库名.表名
```

## 使用WHERE子句过滤数据

一般的应用场景并不需要检索出表中所有的记录行，而只需要从表中检索出一个满足特定条件的记录子集，这就需要用到 `WHERE` 字句加过滤条件。

如下语句将查询满足字段1为某固定值的所有记录：

```mysql
SELECT 字段名1,字段名2,... FROM 表名 WHERE 字段1=某固定值;
```

### WHERE 子句操作符

在 `WHERE` 子句中使用的过滤条件中可以使用的操作符如下表。

| 操作符   | 说明               |
| -------- | ------------------ |
| =        | 等于               |
| <> 或 != | 不等于             |
| <        | 小于               |
| <=       | 小于等于           |
| >        | 大于               |
| >=       | 大于等于           |
| BETWEEN  | 在指定的两个值之间 |

### 组合多个过滤条件

如果需要在 `WHERE` 子句查询满足多个条件的记录，可以使用 `AND` 和 `OR` 操作符组合多个条件。`AND` 表示同时满足多个条件，`OR` 表示满足条件中的一个。

```mysql
SELECT 字段1,字段2,... FROM 表名 WHERE 字段1=固定值1 AND 字段2=固定值2;
```

`AND` 的优先级比 `OR` 的优先级更高，建议使用括号来明确计算顺序。

### 范围值检查

为了查询满足某个字段的值在某个范围内的记录，可以使用范围值检查。范围值检查子句包括 `BETWEEN...AND...`、`IN`。

比如查询价格在5美元到10美元之间的产品名和对应价格。

`SELECT prod_name, prod_price FROM products WHERE prod_price BETWEEN 5 AND 10;`

查询产品 ID 为1002,1003,1005的所有产品。

```mysql
SELECT prod_name, prod_price FROM products WHERE vend_id IN (1002,1003,1005);
```

`BETWEEN` 适合匹配理论上值会在连续范围内的字段，而 `IN` 适合匹配值为离散的某个枚举值的字段。

### 空值检查

在创建表结构时，可以指定某个字段的值可以为 NULL，这和0、空字符串、空白字符是不同的，前者表示该字段为空，后者字段还是有实际值的。

查询 email 为空的用户：

```mysql
SELECT cust_id FROM customers WHERE cust_email IS NULL;
```