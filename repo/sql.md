# SQL基础语句

创建和删除数据库

```sql
CREATE DATABASE name
DROP database name
```

备份数据和导入数据

```mysql
mysqldump -uroot -p name(--all-databases) > a.txt
mysql -uroot -p database_name < a.txt
```

创建和删除表

```
create table name(typename1 type1[not null] [primary key],typename2 type2)
drop table name
```

根据已有表创建新表

```sql
create table tab_new like tab_old (使用旧表创建新表)
create table tab_new as select col1,col2… from tab_old definition only //不插入数据
```

增加列

```sql
alter table tablename add column type
```

添加或删除主键

```sql
alter table tablename add primary key (colname)
Alter table tabname drop primary key(col)
```

insert into 插入记录，或把一个表中数据插入到另一个

```sql
insert into table_name values (col1,col2...)
insert into table_name (col1,col2) valluse (val1,val2)
INSERT INTO table2  (column_name(s)) SELECT column_name(s) FROM table1;
```

update

```sql
update tablename set column1=val1,column2=val2...where somecolumn=somevalue
```

delete 

```sql
delete from table_name where some_column=song_value
```



创建删除索引

UNIQUE: 建立唯一索引。

CLUSTERED: 建立聚集索引。

NONCLUSTERED: 建立非聚集索引。

*索引是不可更改的，想更改必须删除重新建立*

```mysql
create [unique|clustered|nonclusttered] index idxname on tablename(col...)
```

创建删除视图

```sql
create view viewname as select statement
drop view viewname
```

order by用于对结果集按照一个列或者多个列进行排序, 默认升序

```sql
select column1,column2 from table_name order by column1  ASC|DESC
```

group by

```sql
SELECT column_name, function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```



where用于提取满足条件的记录 (=|>|<|!=|BETWEEN|LIKE|IN)

```sql
select * from tablename where column =xx
select * from tablename where column =xx and column1>1
```

Having 是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作，在Having中可以使用聚合函数

```sql
select goods_category_id , avg(goods_price) as ag from sw_goods where ag>1000 group by goods_category //报错！！因为from sw_goods 这张数据表里面没有ag这个字段
```

where和having的区别

```c++
where表示条件如果条件里有 sum avg等集合函数 需要使用 having
Where 是一个/*约束声明*/，使用Where来约束来自数据库的数据，Where是在结果返回之前起作用的，且Where中不能使用聚合函数。
Having 是一个/*过滤声明*/，是在查询返回结果集以后对查询结果进行的过滤操作，在Having中可以使用聚合函数，一般和group by联合使用。
```



select top,limit,rownum //mysql 没有 top 只有limit

```sql
select * from table_name limit num /num为行数
select * from table_name limit i,n//i为索引，从0开始，n为查询结果返回的数量
select column from tablename where rownum < num
select top num* column from table_name
```

like

```sql
select column from tablename where column like pattern
```

通配符

```sql
% 替代0个或多个字符
_替代一个字符
[charlist]字符列中的任何一个单一字符
[^charlist]or[!charlist]不在字符列中的任何单一字符
```

case when 挑选数值

```sql
select (case when 语文 >=80 then ‘优秀’
       when 语文 >=60 then ‘及格’
        else ‘不及格’end) 语文,
    (case when 数学 >=80then‘优秀’
       when 数学 >=60 then‘及格’ 
     else ‘不及格’end) 数学,
    (case when 英语 >=80then ‘优秀’
       when 英语 >=60 then ‘及格’
     else ‘不及格’end) 英语
 
from A;
```



in 允许在where语句中规定多个值

```sql
select column_name from tabe_name where column_name in (value1,value2)
```

between操作符选取介于两个值之间的数据范围内的值

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name BETWEEN value1 AND value2;
```

别名

```sql
select column_name as alias_name from table_name//表的别名
select column_name(s) from table_name as alias_name//列的别名
```

![19](.\img\19.png)

JOIN  连接 用于把来自两个或多个表的行结合起来

![20](.\img\20.png)

inner join ==join     关键字在表中存在至少一个匹配时返回行

```sql
SELECT column_name(s) FROM table1
INNER JOIN table2 ON table1.column_name=table2.column_name;
```

LEFT(RIGHT) JOIN 关键字从左(右)表返回所有的行，即使右(左)表中没有匹配。

```sql
SELECT Websites.name, access_log.count, access_log.date
FROM Websites
LEFT JOIN access_log
ON Websites.id=access_log.site_id
ORDER BY access_log.count DESC;
```

FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.FULL OUTER JOIN 关键字结合了 LEFT JOIN 和 RIGHT JOIN 的结果。



UNION操作符合并两个或多个 SELECT 语句的结果,允许重复值需要UNION ALL 关键字

```
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2
```

# SQL FOREIGN KEY

一个表中的 FOREIGN KEY 指向另一个表中的 UNIQUE KEY(唯一约束的键)。

FOREIGN KEY 约束用于预防破坏表之间连接的行为。

FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

让我们通过一个实例来解释外键。请看下面两个表：

| P_Id | LastName  | FirstName | Address      | City      |
| ---- | --------- | --------- | ------------ | --------- |
| 1    | Hansen    | Ola       | Timoteivn 10 | Sandnes   |
| 2    | Svendson  | Tove      | Borgvn 23    | Sandnes   |
| 3    | Pettersen | Kari      | Storgt 20    | Stavanger |

"Orders" 表：

| O_Id | OrderNo | P_Id |
| ---- | ------- | ---- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 2    |
| 4    | 24562   | 1    |

"Persons" 表中的 "P_Id" 列是 "Persons" 表中的 PRIMARY KEY。

"Orders" 表中的 "P_Id" 列是 "Orders" 表中的 FOREIGN KEY。

```sql
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
PRIMARY KEY (O_Id),
FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
)
```



# SQL约束

SQL 约束用于规定表中的数据规则。

如果存在违反约束的数据行为，行为会被约束终止。

约束可以在创建表时规定（通过 CREATE TABLE 语句），或者在表创建之后规定（通过 ALTER TABLE 语句）。

```sql
CREATE TABLE table_name
(
column_name1 data_type(size) constraint_name,
column_name2 data_type(size) constraint_name,
column_name3 data_type(size) constraint_name,
....
);
```

- **NOT NULL** - 指示某列不能存储 NULL 值。
- **UNIQUE** - 保证某列的每行必须有唯一的值。
- **PRIMARY KEY** - NOT NULL 和 UNIQUE 的结合。确保某列（或两个列多个列的结合）有唯一标识，有助于更容易更快速地找到表中的一个特定的记录。
- **FOREIGN KEY** - 保证一个表中的数据匹配另一个表中的值的参照完整性。
- **CHECK** - 保证列中的值符合指定的条件。
- **DEFAULT** - 规定没有给列赋值时的默认值。
- **AUTO INCREMENT**  每次插入新记录时，自动地创建主键字段的值

创建主键约束

```sql
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (P_Id)
)
```

添加\删除约束

```sql
ALTER TABLE Persons ADD PRIMARY KEY (P_Id)
ALTER TABLE Persons DROP PRIMARY KEY
```



# 范式

select fromhttps://blog.csdn.net/qq_39390545/article/details/115037994 

设计关系型数据库时，要遵从不同的规范要求，设计出合理的关系型数据库，这些规范被称为范式，越高的范式数据库冗余度越小

第一范式(1NF):

- 对属性的原子性要求，要求属性具有原子性，不可再分解

例如，"中国河南枣庄"可拆成国家、省份、市，不符合第一范式

![23](.\img\23.png)

```sql
mysql> select * from employee;
+-------------+--------------+-----------+------+------+-----------------------+--------------+--------------+-----------------------+
| employee_id | dept_name    | name      | age  | sex  | address               | job          | job_desc     | dept_desc             |
+-------------+--------------+-----------+------+------+-----------------------+--------------+--------------+-----------------------+
|           1 | 研发一部     | 陈哈哈      |   27 | 男   | 中国山东枣庄          | java研发     | 做web        | 做公司门户网站        |
|           2 | 宣传部       | 川建国      |   72 | 男   | 美国纽约曼哈顿        | 宣传部长     | 吹牛逼       | 跟客户吹逼            |
|           3 | 保卫科       | 盲僧       |   30 | 男    | 中国河南嵩山          | 保安队长     | 练回旋踢     | 站岗                  |
+-------------+--------------+-----------+------+------+-----------------------+--------------+--------------+-----------------------+
3 rows in set (0.00 sec)


```



第二范式(2NF):

- 满足1NF
- 表必须有一个主键
- 对于非主键列，必须完全依赖于主键，而不能只依赖于主键的一部分
- 目的是进一步减少插入异常和更新异常

上表中的dept_desc由主键dept_name所决定，但却不是由主键employee_id决定，所以dept_desc只依赖于两个主键中的一个，故要解决dept_desc对主键是部分依赖，从而满足第二范式，则需将dept_name、dept_desc拆分出来

![24](.\img\24.png)

第三范式(3NF)

- 满足2NF
- 非主键列必须直接依赖于主键，**不能存在传递依赖**。即不能存在：非主键列 A 依赖于非主键列 B，非主键列 B 依赖于主键的情况,表内每一列数据都与主键直接相关，不能简介相关。

# SQL索引

## 什么是索引？

　　SQL索引有两种，**聚集索引**和**非聚集索引**，索引主要目的是提高了SQL Server系统的性能，加快数据的查询速度与减少系统的响应时间 

​	聚集索引存储记录是物理上连续存在，而非聚集索引是逻辑上的连续，物理存储并不连续。就像字段，聚集索引是连续的，a后面肯定是b，非聚集索引就不连续了，就像图书馆的某个作者的书，有可能在第1个货架上和第10个货架上。还有一个小知识点就是：聚集索引一个表只能有一个，而非聚集索引一个表可以存在多个。

​	聚集索引是有序映射，非聚集索引是无序映射

创建索引

```sql
CREATE INDEX index_name ON table_name (column_name)
CREATE INDEX PIndex ON Persons (LastName, FirstName)
CREATE UNIQUE INDEX index_name ON table_name (column_name)//两个行不能有相同索引值
```



## 索引的存储机制

  　　首先，无索引的表，查询时，是按照顺序存续的方法扫描每个记录来查找符合条件的记录，这样效率十分低下,举个例子，如果我们将字典的汉字随即打乱，没有前面的按照拼音或者部首查询，那么我们想找一个字，按照顺序的方式去一页页的找，这样效率有多底，大家可以想象。

​       聚集索引和非聚集索引的根本区别是表记录的排列顺序和与索引的排列顺序是否一致，其实理解起来非常简单，还是举字典的例子：如果按照拼音查询，那么都是从a-z的，是具有连续性的，a后面就是b，b后面就是c， 聚集索引就是这样的，他是和表的物理排列顺序是一样的，例如有id为聚集索引，那么1后面肯定是2,2后面肯定是3，所以说这样的搜索顺序的就是聚集索引。非聚集索引就和按照部首查询是一样是，可能按照偏房查询的时候，根据偏旁‘弓’字旁，索引出两个汉字，张和弘，但是这两个其实一个在100页，一个在1000页，（这里只是举个例子），他们的索引顺序和数据库表的排列顺序是不一样的，这个样的就是非聚集索引。

​      聚集索引就是在数据库被开辟一个物理空间存放他的排列的值，例如1-100，所以当插入数据时，他会重新排列整个整个物理空间，而非聚集索引其实可以看作是一个含有聚集索引的表，他只仅包含原表中非聚集索引的列和指向实际物理表的指针。他只记录一个指针，其实就有点和堆栈差不多的感觉了

|        动作描述        | 使用聚集索引 | 使用非聚集索引 |
| :----------------: | :----: | :-----: |
|        外键列         |   应    |    应    |
|        主键列         |   应    |    应    |
| 列经常被分组排序(order by) |   应    |    应    |
|     返回某范围内的数据      |   应    |   不应    |
|      小数目的不同值       |   应    |   不应    |
|      大数目的不同值       |   不应   |    应    |
|       频繁更新的列       |   不应   |    应    |
|      频繁修改索引列       |   不应   |    应    |
|      一个或极少不同值      |   不应   |   不应    |

建立索引的原则：

1) 定义主键的数据列一定要建立索引。

2) 定义有外键的数据列一定要建立索引。

3) 对于经常查询的数据列最好建立索引。

4) 对于需要在指定范围内的快速或频繁查询的数据列;

5) 经常用在WHERE子句中的数据列。

6) 经常出现在关键字order by、group by、distinct后面的字段，建立索引。如果建立的是复合索引，索引的字段顺序要和这些关键字后面的字段顺序一致，否则索引不会被使用。

7) 对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。

8) 对于定义为text、image和bit的数据类型的列不要建立索引。

9) 对于经常存取的列避免建立索引 

9) 限制表上的索引数目。对一个存在大量更新操作的表，所建索引的数目一般不要超过3个，最多不要超过5个。索引虽说提高了访问速度，但太多索引会影响数据的更新操作。

10) 对复合索引，按照字段在查询条件中出现的频度建立索引。在复合索引中，记录首先按照第一个字段排序。对于在第一个字段上取值相同的记录，系统再按照第二个字段的取值排序，以此类推。因此只有复合索引的第一个字段出现在查询条件中，该索引才可能被使用,因此将应用频度高的字段，放置在复合索引的前面，会使系统最大可能地使用此索引，发挥索引的作用。

# SQL视图(view)

## 什么是视图？

在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，我们也可以提交数据，就像这些来自于某个单一的表。

注释：数据库的设计和结构不会受到视图中的函数、where 或 join 语句的影响。

## SQL CREATE VIEW 语法

```sql
CREATE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```

注释：视图总是显示最近的数据。每当用户查询视图时，数据库引擎通过使用 SQL 语句来重建数据。且视图建立在表中，*SHOW FULL TABLES WHERE Table_type = 'VIEW'* 查看所有view