---
title: Mysql的sql_mode简单介绍
date: 2016-10-18
tags: 
	- mysql
categories: mysql
---

## 为什么了解sql_mode？

在写msql安装的博客时，cat /etc/my.cnf的时候，最后一行的sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES很是惹眼。既然这么惹眼就简单的看一下。

## sql_mode官方文档 [点击探索](http://dev.mysql.com/doc/refman/5.6/en/sql-mode.html)

## sql_mode的含义？

首先看一下sql_mode的简单介绍：
```
The MySQL server can operate in different SQL modes, and can apply these modes differently for different clients,
depending on the value of the sql_mode system variable. DBAs can set the global SQL mode to match site server operating
requirements, and each application can set its session SQL mode to its own requirements.
```
译文：
```
MySQL服务器可以在不同的SQL模式下运行，并且可以根据sql_mode系统变量的值对不同的客户端应用不同的模式。 DBA可以设置全局SQL模式以匹配站点服务器
操作要求，并且每个应用程序可以将其会话SQL模式设置为其自己的要求。
```
也就是说sql_mode就是mysql运行的sql模式。那么肯定和sql相关。

## 都有哪些sql_mode,它们的作用是什么？

###  比较重要的sql_mode：


#### STRICT_TRANS_TABLES（严格模式）
+ 严格模式控制MySQL如何处理数据更改语句（如INSERT或UPDATE）中的无效或缺失值。值可能由于几个原因无效。例如，它可能具有列的错误数据类型，或者它可能超出范围。当插入一条数据却没有设置一个非空且没有设置default数据的字段的值时。严格模式还会影响DDL语句，例如CREATE TABLE。
测试：
	1. 创建test库和test表：
```
    mysql> create database test;
    Query OK, 1 row affected (0.01 sec)
    mysql> use test
    Database changed
    mysql> create table test (id int(10),name varchar(10),birthday datetime);
    Query OK, 0 rows affected (0.27 sec)
```
	2. 设置sql_mode为严格模式，这里有一个知识点，这是知识点很简单我们就直接了解下，mysql的几种变量。目前我所了解的分为三种：session变量、用户变量、全局变量。当然这种情况适用于使用sql语句设置变量，修改my.cnf属于全局变量。
	设置变量：
```
设置session变量：
mysql> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)
设置用户变量：
mysql> set @sql_mode='';
Query OK, 0 rows affected (0.00 sec)
设置全局变量：
mysql> set @@sql_mode='';
Query OK, 0 rows affected (0.00 sec)
```
	查询变量：
```
查询session变量，注意，此时不能用select sql_mode：
mysql> select @@session.sql_mode;
+--------------------+
| @@session.sql_mode |
+--------------------+
|                    |
+--------------------+
1 row in set (0.00 sec)
查询用户变量：
mysql> select @sql_mode;
+-----------+
| @sql_mode |
+-----------+
|           |
+-----------+
1 row in set (0.00 sec)
查询全局变量：
mysql> select @@sql_mode;
+------------+
| @@sql_mode |
+------------+
|            |
+------------+
1 row in set (0.00 sec)
```
	注意：设置用户和全局变量不会影响已连接的session的变量，会影响新建的连接的变量。
    变量相关的话题到此结束，下面我们来设置sql_mode为严格模式，我们需要修改我们的session变量：
```
mysql> set sql_mode='STRICT_ALL_TABLES';
Query OK, 0 rows affected (0.00 sec)
mysql> select @@session.sql_mode;
+--------------------+
| @@session.sql_mode |
+--------------------+
| STRICT_ALL_TABLES  |
+--------------------+
1 row in set (0.00 sec)
```
	3. 验证插入'错误的数据类型'和'超出范围'
```
严格模式下：
mysql> insert into test values('xiaoguo','xiaoli',now());
ERROR 1366 (HY000): Incorrect integer value: 'xiaoguo' for column 'id' at row 1
mysql> insert into test values(1,'xiaolixiaoguo',now());
ERROR 1406 (22001): Data too long for column 'name' at row 1
非严格模式下：
mysql> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)
mysql> insert into test values('xiaoguo','xiaoli',now());
Query OK, 1 row affected, 1 warning (0.00 sec)
mysql> insert into test values(1,'xiaolixiaoguo',now());
Query OK, 1 row affected, 1 warning (0.00 sec)
看看插入了些什么：
mysql> select * from test;
+------+------------+---------------------+
| id   | name       | birthday            |
+------+------------+---------------------+
|    0 | xiaoli     | 2016-10-18 22:33:23 |
|    1 | xiaolixiao | 2016-10-18 22:33:30 |
+------+------------+---------------------+
我们看到，在类型错误时，插入了该类型的默认值，在类型超过规定长度时，mysql帮我们进行了截取。
```
	4. 验证当要插入的新行不包含其定义中没有显式DEFAULT子句的非NULL列的值时，缺少值时。
	```
修改id列为非空：
mysql> alter table test modify column id int(5) not null;
Query OK, 2 rows affected (0.26 sec)
Records: 2  Duplicates: 0  Warnings: 0
修改sql_mode为严格模式：
mysql> set sql_mode='STRICT_TRANS_TABLES';
Query OK, 0 rows affected (0.00 sec)
插入一条数据，不设置id字段
mysql> insert into test(name,birthday)values('xiaozhang',now());
ERROR 1364 (HY000): Field 'id' doesn't have a default value
设置sql_mode为非严格模式：
mysql> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)
插入一条数据，不设置id字段：
mysql> insert into test(name,birthday)values('xiaozhang',now());
Query OK, 1 row affected, 1 warning (0.00 sec)
查询插入了什么？
mysql> select * from test;
+----+------------+---------------------+
| id | name       | birthday            |
+----+------------+---------------------+
|  0 | xiaoli     | 2016-10-18 22:33:23 |
|  1 | xiaolixiao | 2016-10-18 22:33:30 |
|  0 | xiaozhang  | 2016-10-18 22:43:58 |
+----+------------+---------------------+
发现插入了int的默认值。
```
	5. 严格模式还会影响DDL语句，例如CREATE TABLE。（不知何以，切并未搜索到相关案例，如果读者知道请邮件：522054591@qq.com【码匠】）。

+ 如果严格模式没有生效，MySQL会将调整后的值插入无效或缺失值，并生成警告（请参见第13.7.5.41节“显示警告语法”）。在严格模式下，可以使用INSERT IGNORE或UPDATE IGNORE来产生此行为。
如下：
```
mysql> set sql_mode='';
Query OK, 0 rows affected (0.00 sec)
mysql> insert into test(id,name)values(2,'xiaoliuxiaoguo');
Query OK, 1 row affected, 1 warning (0.00 sec)
显示警告：
mysql> show warnings;
+---------+------+-------------------------------------------+
| Level   | Code | Message                                   |
+---------+------+-------------------------------------------+
| Warning | 1265 | Data truncated for column 'name' at row 1 |
+---------+------+-------------------------------------------+
1 row in set (0.00 sec)
```
	想了解更多关于show warnings如：显示warnings的数量等？ [点击探索](http://dev.mysql.com/doc/refman/5.6/en/show-warnings.html)

+ 对于诸如SELECT之类的不更改数据的语句，无效值会在严格模式下生成警告，而不是错误。
如下：
```
mysql> select * from test where id = 'test';
+----+-----------+---------------------+
| id | name      | birthday            |
+----+-----------+---------------------+
|  0 | xiaoli    | 2016-10-18 22:33:23 |
|  0 | xiaozhang | 2016-10-18 22:43:58 |
+----+-----------+---------------------+
2 rows in set, 1 warning (0.01 sec)
显示警告：
mysql> show warnings;
+---------+------+------------------------------------------+
| Level   | Code | Message                                  |
+---------+------+------------------------------------------+
| Warning | 1292 | Truncated incorrect DOUBLE value: 'test' |
+---------+------+------------------------------------------+
1 row in set (0.00 sec)
```

+ 从MySQL 5.6.11开始，严格模式会在尝试创建超过最大密钥长度的密钥时产生错误。以前，这导致警告和截断到最大密钥长度的密钥（与没有启用严格模式时相同）。不了解，暂时也没兴趣。

+ 严格模式不会影响是否检查外键约束。 foreign_key_checks可以用于。 （请参见第5.1.5节“服务器系统变量”。）

+ 如果启用了STRICT_ALL_TABLES或STRICT_TRANS_TABLES，则严格的SQL模式有效，但这些模式的效果有所不同：
	+ 对于事务表，当启用STRICT_ALL_TABLES或STRICT_TRANS_TABLES时，数据更改语句中的无效或缺失值将发生错误。语句被中止并回滚。
	+ 对于非事务表，如果错误值出现在要插入或更新的第一行中，则对任一模式的行为都是相同的：语句被中止，表保持不变。如果语句插入或修改多个行，并且第二行或更高版本中出现错误值，则结果取决于启用的严格模式：
		+ 对于【STRICT_ALL_TABLES】，MySQL返回一个错误并忽略其余行。但是，由于较早的行已插入或更新，结果是部分更新。要避免这种情况，请使用单行语句，可以在不更改表的情况下中止这些语句。
		+ 对于【STRICT_TRANS_TABLES】，MySQL将无效值转换为列的最接近的有效值，并插入调整后的值。如果缺少一个值，MySQL将为列数据类型插入隐式默认值。在任一情况下，MySQL都会生成警告，而不是错误，并继续处理语句。隐式默认值在第11.6节“数据类型默认值”中描述。
	+ 上面BB了那么多意思很简单【非事务表】即使在严格模式下也会出现问题：
    插入多条数据，第一条合格，其他条不合格
    如果严格模式是STRICT_ALL_TABLES：第一条数据插入，其他失败。
    如果严格模式是STRICT_TRANS_TABLES：第一条数据插入，其他无效的数据转为有效的插入！
	事务表：支持事务的表，取决于其存储引擎，比如存储引擎为InnoDB的表为事务表，MyISAM的表为非事务表。下面我们来验证上述说法：
    ```
        我们先看一下一个标准的建表语句：
        mysql> show create table test;
        +-------+------------------------------------------------------------------------------------------------------------------------------------------------------------+
        | Table | Create Table                                                                                                                                               |
        +-------+------------------------------------------------------------------------------------------------------------------------------------------------------------+
        | test  | CREATE TABLE `test` (
          `id` int(5) NOT NULL,
          `name` varchar(10) DEFAULT NULL,
          `birthday` datetime DEFAULT NULL
        ) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
        +-------+------------------------------------------------------------------------------------------------------------------------------------------------------------+
        1 row in set (0.00 sec)
        看看数据库支持的引擎
        mysql> show engines;
        +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
        | Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
        +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
        | MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
        | CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
        | MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
        | BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
        | MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
        | InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
        | ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
        | PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
        | FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
        +--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
        9 rows in set (0.00 sec)
        创建一个MyISAM表
        mysql> create table test1 (id int,name varchar(10),birthday datetime) ENGINE=MyISAM;
        Query OK, 0 rows affected (0.00 sec)
        查看当前的sql_mode
        mysql> select @@session.sql_mode;
        +---------------------+
        | @@session.sql_mode  |
        +---------------------+
        | STRICT_TRANS_TABLES |
        +---------------------+
        1 row in set (0.00 sec)
        插入一条id不合格的数据：
        mysql> insert into test1 values('name','xiaoli',now());
        报错是正常滴：
        ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '('name','xiaoli',now())' at line 1
        插入两条数据，第一条完全合格，第二条id不合格！
        mysql> insert into test1 values(1,'xiaoli',now()),('name','xiaoli',now());
        天啊！！！竟然插进去了！！！
        Query OK, 2 rows affected, 1 warning (0.00 sec)
        Records: 2  Duplicates: 0  Warnings: 1
        看一下警告吧
        mysql> show warnings;
        +---------+------+----------------------------------------------------------+
        | Level   | Code | Message                                                  |
        +---------+------+----------------------------------------------------------+
        | Warning | 1366 | Incorrect integer value: 'name' for column 'id' at row 2 |
        +---------+------+----------------------------------------------------------+
        1 row in set (0.00 sec)
        让我们看看插入了什么
        mysql> select * from test1;
        +------+--------+---------------------+
        | id   | name   | birthday            |
        +------+--------+---------------------+
        |    1 | xiaoli | 2016-10-18 23:35:55 |
        |    0 | xiaoli | 2016-10-18 23:35:55 |
        +------+--------+---------------------+
        2 rows in set (0.00 sec)
        将不合格的id设置为int的默认值0
        再来看看【STRICT_ALL_TABLES】
        mysql> set sql_mode='STRICT_ALL_TABLES';
        Query OK, 0 rows affected (0.00 sec)
        插入上面一样的数据
        mysql> insert into test1 values(1,'xiaoli',now()),('name','xiaoli',now());
        报错了，哈哈哈
        ERROR 1366 (HY000): Incorrect integer value: 'name' for column 'id' at row 2
        神奇的是对的那条没有回滚啊！
        mysql> select * from test1;
        +------+--------+---------------------+
        | id   | name   | birthday            |
        +------+--------+---------------------+
        |    1 | xiaoli | 2016-10-18 23:35:55 |
        |    0 | xiaoli | 2016-10-18 23:35:55 |
        |    1 | xiaoli | 2016-10-18 23:47:24 |
        +------+--------+---------------------+
        3 rows in set (0.00 sec)
```

+ 严格模式还会影响处理除以零，零日期和日期的零，以及ERROR_FOR_DIVISION_BY_ZERO，NO_ZERO_DATE和NO_ZERO_IN_DATE模式。有关详细信息，请参阅这些模式的说明。
+ 如何让严格模式对单个sql语句失效？很简单insert ignore或者update ignore

#### ANSI
+ 更改语法和行为，使其更符合标准SQL，相当于REAL_AS_FLOAT, PIPES_AS_CONCAT, ANSI_QUOTES, IGNORE_SPACE的集合。

#### TRADITIONAL

+ 让MySQL的行为像一个“传统的”SQL数据库系统。 在向列中插入不正确的值时，此模式的简单描述是“给出错误而不是警告”。 它是本节结尾列出的特殊组合模式之一。等同于STRICT_TRANS_TABLES, STRICT_ALL_TABLES, NO_ZERO_IN_DATE, NO_ZERO_DATE, ERROR_FOR_DIVISION_BY_ZERO, NO_AUTO_CREATE_USER, and NO_ENGINE_SUBSTITUTION的集合。

### 其它sql_mode:
#### ERROR_FOR_DIVISION_BY_ZERO
+ 在严格模式下，如果update或者insert的数据进行运算且除数为0，会出现警告然后插入NULL，加上该sql_mode会报错而不是警告。
从MySQL 5.6.17开始，NO_ZERO_DATE已弃用，并将sql_mode值设置为包含它会生成警告。
	如：
```
        mysql> set sql_mode='STRICT_TRANS_TABLES';
        Query OK, 0 rows affected (0.00 sec)
        mysql> insert into test1 values(1,10/0,now());
        Query OK, 1 row affected (0.00 sec)
        mysql> select * from test1;
        +------+--------+---------------------+
        | id   | name   | birthday            |
        +------+--------+---------------------+
        |    1 | NULL   | 2016-10-19 00:16:52 |
        +------+--------+---------------------+
        mysql> set sql_mode='STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO';
        Query OK, 0 rows affected, 1 warning (0.00 sec)

        mysql> insert into test1 values(1,10/0,now());
        ERROR 1365 (22012): Division by 0
```

#### NO_ZERO_DATE
+ NO_ZERO_DATE模式影响服务器是否允许“0000-00-00”作为有效日期。 其效果还取决于是否启用严格SQL模式。
如果未启用此模式，则允许“0000-00-00”，插入不会产生警告。
如果启用此模式，则允许“0000-00-00”，插入将产生警告。
如果启用此模式和严格模式，则不允许使用“0000-00-00”，并且插入会产生错误。
从MySQL 5.6.17开始，NO_ZERO_DATE已弃用，并将sql_mode值设置为包含它会生成警告。
测试:
```
        设置sql_mode为NO_ZERO_DATE，现在是非严格模式
        mysql> set sql_mode='NO_ZERO_DATE';
        Query OK, 0 rows affected, 1 warning (0.00 sec)

        产生警告：NO_ZERO_DATE在未来的版本会移除
        mysql> show warnings;
        +---------+------+-----------------------------------------------------------------------+
        | Level   | Code | Message                                                               |
        +---------+------+-----------------------------------------------------------------------+
        | Warning | 1681 | 'NO_ZERO_DATE' is deprecated and will be removed in a future release. |
        +---------+------+-----------------------------------------------------------------------+
        1 row in set (0.00 sec)

        mysql> insert  into test values(1,'xiaoguo','0000-00-00');
        Query OK, 1 row affected, 1 warning (0.01 sec)

        mysql> show warnings;
        +---------+------+---------------------------------------------------+
        | Level   | Code | Message                                           |
        +---------+------+---------------------------------------------------+
        | Warning | 1264 | Out of range value for column 'birthday' at row 1 |
        +---------+------+---------------------------------------------------+
        1 row in set (0.00 sec)

        设置为严格模式，再试
        mysql> set sql_mode='STRICT_TRANS_TABLES,NO_ZERO_DATE';
        Query OK, 0 rows affected, 1 warning (0.00 sec)
        报错！
        mysql> insert  into test values(1,'xiaoguo','0000-00-00');
        ERROR 1292 (22007): Incorrect datetime value: '0000-00-00' for column 'birthday' at row 1
```
#### NO_ZERO_IN_DATE
+ NO_ZERO_IN_DATE模式会影响服务器是否允许年份部分为非零，但月份或日期部分为0的日期。（此模式会影响日期，例如“2010-00-01”或“2010-01-00”，但不会影响日期 '0000-00-00'。要控制服务器是否允许'0000-00-00'，请使用NO_ZERO_DATE模式。）NO_ZERO_IN_DATE的效果也取决于是否启用严格SQL模式。
如果未启用此模式，则允许零零件的日期，插入不产生警告。
如果启用此模式，则零零件的日期将插入为“0000-00-00”，并产生警告。
如果启用此模式和严格模式，则不允许具有零部分的日期，并且插入会产生错误。
从MySQL 5.6.17开始，NO_ZERO_IN_DATE已弃用，并将sql_mode值设置为包含它会生成警告。
测试:
```
		没有设置sql_mode
		mysql> set sql_mode='';
		Query OK, 0 rows affected (0.00 sec)

        mysql> insert  into test values(1,'xiaoguo','1992-00-00');
        Query OK, 1 row affected (0.00 sec)
		设置ql_mode为NO_ZERO_IN_DATE
        mysql> set sql_mode='NO_ZERO_IN_DATE';
        Query OK, 0 rows affected, 1 warning (0.00 sec)

        mysql> insert  into test values(1,'xiaoguo','1992-00-00');
        Query OK, 1 row affected, 1 warning (0.00 sec)

        mysql> show warnings;
        +---------+------+---------------------------------------------------+
        | Level   | Code | Message                                           |
        +---------+------+---------------------------------------------------+
        | Warning | 1264 | Out of range value for column 'birthday' at row 1 |
        +---------+------+---------------------------------------------------+
        1 row in set (0.00 sec)
		设置sql_mode为严格模式
        mysql> set sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE';
        Query OK, 0 rows affected, 1 warning (0.00 sec)

        mysql> insert  into test values(1,'xiaoguo','1992-00-00');
        ERROR 1292 (22007): Incorrect datetime value: '1992-00-00' for column 'birthday' at row 1
```
#### 更多sql_mode请参考官方文档。sql_mode官方文档 [点击探索](http://dev.mysql.com/doc/refman/5.6/en/sql-mode.html)

