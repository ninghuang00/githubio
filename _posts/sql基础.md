---
title: sql基础
categories:
  - 数据库
tags:
  - sql
date: 2018-09-02 10:23:52
---
 这是摘要
 <!-- more -->

# SQL基本知识
## select的基本能力
1. 投影
在二维表中,根据条件选择相应的列
2. 选择
在二维表中,根据条件选择相应的行
3. 连接
多个表中获取需要的行,并且将这些行结合在一起

## 列的别名
1. 可是在列名后面跟上别名,中间可以加`as`,也可以省略
2. 别命中有空格,特殊字符或者大小写敏感的需要加双引号

## 级联操作
如果需要将查询结果中每一列的字符串连接可以使用`||`,如`select firstname||lastname as fullname`

## 查询数据的限制和排序
1. where条件语句后面如果是字符串的话要加单引号
2. where后面如果要匹配null值,不是用`=`,用的是`is null`
3. `between and`的范围包含两个端点值
4. `order by a,b desc`,先按照a升序排列,排序结果相同的再按照b降序排列

## 常用函数
1. 字符串处理
CONCAT(str1,str2)连接字符串,substr(str,index,length)取出子字符串,instr(str,character)字符在字符串中的位置,length(str)字符串的长度
2. 数字处理
round(num,2)小数点后两位四舍五入,trunc(num,2)小数点后两位截取
3. 日期处理
`select sysdate from dual`取出当前系统的日期
4. 转换函数
将数字,日期转换成字符串`to_char(date,'YYYY-MM-DD')`,`to_char(num,'$99,999.00')`,将`$`换成`L`则表示使用本地流通的货币符号
将字符串转换成数字`to_num('str','999,99.99')`
将字符串转换成日期`to_date('str','yyyy-mm-dd hh24:mm:ss')`
5. 分组操作
avg(num)求平均,sum(num)求和,`count(*)`计数,max(num),min(num)
group by分组然后计算,条件使用having而不是where,having只能跟在group by后面,是对group by的结果进行筛选,但是最后还是可以跟where对`group by having`的结果进行筛选

## 多表查询
1. 自连接
自连接的第一步就是将同一张表按照需要查询的信息命名别名,然后就可以当成两张表俩进行连表查询
2. 等值连接
from where xx=yy
3. 外连接(左连接,右连接,全连接)
from join on xx=yy

## 子查询
1. 单行子查询
2. 多行子查询
in()和子查询返回列表中的任意一个相等,any()和子查询返回列表中的任何一个比较,有一个返回true就行,all()和子查询返回列表中的所有值比较,所有的都要返回true


## 事务
### 事务结束的标记
1. 发出commit或者rollback语句时
2. 使用DDL(define数据定义语言,建表,索引,视图等)或者DCL(control数据控制语言,赋权,提交,回滚等)语句的时候(隐式)
3. 用户退出的h时候(默认)
4. 数据库崩溃的时候(隐式)

## SQL语句简单优化
1. 避免使用`*`
2. 多表查询使用别名
3. where子句中连接顺序,执行顺序是从右到左,所以需要把能够迅速缩小范围的条件表达式放在后面
4. 尽量使用`>=`替代`>`
5. 确定需要删除的数据使用truncate而不是delete,truncate删除的数据无法回滚,delete可以
6. 多使用commit,以释放以下资源:回滚段中的空间,程序语句获得的锁,redo log buffer中的空间,数据库管理以上内容的资源
7. 不要使用索引字段进行运算,因为运算会使得索引失效
低效:`select name from temp where sal*2 > 20000`
高效:`select name from temp where sal > 20000/2`


# sql执行顺序
{% asset_img sql执行顺序.png sql执行顺序%}

# SQL编译基本过程
1. 直接写一堆的sql来进行数据的处理，和用一个存储过程来进行数据的处理，哪个性能更好一些。
存储过程是在数据库中已经编译过的,直接写sql还要先编译,效率会差一些


# sql语法
## 嵌套查询
> 参考地址:https://my.oschina.net/startphp/blog/123337

```sql
#查询“001”课程比“002”课程成绩高的所有学生的学号；
Select a.Sno 
from 
(select Sno,score from SC where Cno='001') a,
(select Sno,score from SC where Cno='002') b
Where a.Sno=b.Sno and a.score>b.score;
```





## 函数
* group by 
{% asset_img 演示数据库.png 演示数据库%}
GROUP BY 语句用于**结合聚合函数**，根据一个或多个列对结果集进行分组。

```sql
# 求学生平均成绩
SELECT id, AVG(score)
FROM student
GROUP BY id;
```

```sql 
# 统计各个site_id的访问次数
SELECT site_id, SUM(access_log.count) AS nums
FROM access_log GROUP BY site_id;
```
{% asset_img groupby1.png %}

```sql 
# 统计各个网站访问数量
SELECT Websites.name,COUNT(access_log.aid) AS nums 
FROM access_log
LEFT JOIN Websites
ON access_log.site_id=Websites.id
GROUP BY Websites.name;
```
{% asset_img groupby2.png %}

* having
1. 在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用。where是先于聚合语句执行的,
HAVING 子句可以让我们筛选分组后的各组数据。
2. 而聚合语句是先于having执行的
```sql 
# 查找总访问量大于 200 的网站。
SELECT Websites.name, Websites.url, SUM(access_log.count) AS nums 
FROM (access_log
INNER JOIN Websites
ON access_log.site_id=Websites.id)
GROUP BY Websites.name
HAVING SUM(access_log.count) > 200;
```
{% asset_img having1.png %}

```sql 
# 查找总访问量大于 200 的网站，并且 alexa 排名小于 200。
SELECT Websites.name, SUM(access_log.count) AS nums FROM Websites
INNER JOIN access_log
ON Websites.id=access_log.site_id AND Websites.alexa < 200 
#WHERE Websites.alexa < 200 
GROUP BY Websites.name
HAVING SUM(access_log.count) > 200;
```
{% asset_img having2.png %}

* where和having的区别
1. 找出部门平均薪资大于3000的
```java
select deparment, avg(salary) as average from salary_info 
group by deparment having average > 3000
```
2. 统计每个部门中薪资大于3000的人数
```java
select deparment, count(*) as c from salary_info 
where salary > 3000 group by deparment

```


## 基础语法
* distinct
```sql 
SELECT DISTINCT column_name,column_name
FROM table_name;
```
* limit
```sql
SELECT column_name(s)
FROM table_name
LIMIT number;
```

* order by
```sql 
SELECT column_name,column_name
FROM table_name
ORDER BY column_name,column_name ASC|DESC;
```

## sql通配符
```sql 
# MySQL支持
% : 代表0个或者多个字符
_ : 代表1个字符
# MySQL不支持
[abc] : a,b,c中的一个字符
[!abc]或者[^abc] : 不是a,b,c中的字符
```

## 高级语法
* like
```sql
#返回G开头的 
SELECT * FROM Websites
WHERE name LIKE 'G%';
```

* regexp
```sql 
#正则表达式,选取G或者F或者s开头的
SELECT * FROM Websites
WHERE name REGEXP '^[GFs]';
```

* between和in
```sql 
SELECT * FROM Websites
WHERE (alexa BETWEEN 1 AND 20)
AND NOT country IN ('USA', 'IND');
```

* 别名
```sql 
#列别名
SELECT name, CONCAT(url, ', ', alexa, ', ', country) AS site_info
FROM Websites;
```

```sql 
#表别名
SELECT w.name, w.url, a.count, a.date 
FROM Websites AS w, access_log AS a 
WHERE a.site_id=w.id and w.name="菜鸟教程";
```

* inner join(简写成join)
INNER JOIN 关键字在表中存在至少一个匹配时返回行。如果 "Websites" 表中的行在 "access_log" 中没有匹配，则不会列出这些行。
```sql 
SELECT Websites.id, Websites.name, access_log.count, access_log.date
FROM Websites
INNER JOIN access_log
ON Websites.id=access_log.site_id;
```

* left join
LEFT JOIN 关键字从左表（table1）返回所有的行，即使右表（table2）中没有匹配。如果右表中没有匹配，则结果为 NULL。
返回的结果数>=左表
```sql 
SELECT column_name(s)
FROM table1
LEFT JOIN table2
ON table1.column_name=table2.column_name;
```

* right join
RIGHT JOIN 关键字从右表（table2）返回所有的行，即使左表（table1）中没有匹配。如果左表中没有匹配，则结果为 NULL。
返回的结果数>=右表
```sql 
SELECT column_name(s)
FROM table1
RIGHT JOIN table2
ON table1.column_name=table2.column_name;
```

* full join
FULL OUTER JOIN 关键字只要左表（table1）和右表（table2）其中一个表中存在匹配，则返回行.
```sql 
SELECT column_name(s)
FROM table1
FULL OUTER JOIN table2
ON table1.column_name=table2.column_name;
```
* where和on的关系

>参考地址:http://www.runoob.com/w3cnote/sql-join-the-different-of-on-and-where.html

* union和union all
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。
UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。
UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。
```sql 
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```

* 
```sql 

```

* 
```sql 

```

* 连接的结果数
```
inner join <= min(left join, right join)
full join >= max(left join, right join)
当 inner join < min(left join, right join) 时， full join > max(left join, right join)
```

## 约束
```sql 
CREATE TABLE Persons
(
    Id_P int NOT NULL PRIMARY KEY,   //PRIMARY KEY约束
    LastName varchar(255) NOT NULL,
    FirstName varchar(255)
)
```
* unique
同一张表中可以有多个列为unique
创建表时
```sql
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
UNIQUE (P_Id)
)
```
修改
```sql
ALTER TABLE Persons
ADD UNIQUE (P_Id)

ALTER TABLE Persons
DROP INDEX uc_PersonID
```

* primary key 
同一张表中只能有一列
```sql 
CREATE TABLE Persons
(
P_Id int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
)
```
修改
```sql 
ALTER TABLE Persons
ADD PRIMARY KEY (P_Id)
```
联合主键
```SQL
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (P_Id,LastName)
```
* foreign key 
```sql 
CREATE TABLE Orders
(
O_Id int NOT NULL,
OrderNo int NOT NULL,
P_Id int,
PRIMARY KEY (O_Id),
FOREIGN KEY (P_Id) REFERENCES Persons(P_Id)
)
//修改
ALTER TABLE Orders
ADD FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)
//联合外键
ALTER TABLE Orders
ADD CONSTRAINT fk_PerOrders
FOREIGN KEY (P_Id)
REFERENCES Persons(P_Id)
```

* check
```sql 
CREATE TABLE Persons
(
P_Id int NOT NULL,
CHECK (P_Id>0)
)
//修改
ALTER TABLE Persons
ADD CHECK (P_Id>0)
//多个
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (P_Id>0 AND City='Sandnes')
```

* default
```sql 
CREATE TABLE Persons
(
    P_Id int NOT NULL,
    City varchar(255) DEFAULT 'Sandnes'
)
//修改
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

# NoSQL入门
## 发展历史
主要是为了应对传统结构性数据库对于海量数据处理速度慢,扩展困难的问题,半结构化的数据现在占数据量的大头

## CAP理论
### 背景
1. 单节点无法保存海量的数据
2. 同一份数据在在多个节点上保存以保证可靠性

### 问题
同一份数据在多个节点上保存时是否实时同步

### cap 
一个分布式系统不可能同时满足以下三点中的两点,但是每一个特性不是绝对的,可以灵活的适应需求:
1. 一致性(consistence)
每一个节点上的副本都是最新的,一致性可以是最终一致性
2. 可用性(availability)
对数据的更新具备高可用,可用性可以在0到100%之间实时变动
3. 分区容忍性(partition tolerance)
相当于是对网络通信的时限要求

### NoSQL的分类和代表产品
1. 键值型
redis,memcached,riak,不适合数据之间存在强关联性的数据存储,产品一般是对于特定应用的定制开发,缺乏通用性
	1. web应用的暂时性数据的追踪,如用户购物车
	2. 存储配置和用户信息数据的移动应用
	3. 关系型数据库查询的缓存
	4. 存储音频和图片的等较大的文件
2. 文档型
MongoDB,couchDB,将版本化的文档,半结构化的文档以特定的格式存储,不适合处理复杂事务,查询持续变化的聚合结构
	1. 适用于事件日志,web分析,博客,电子商务等
3. 表型
HBASE,Cassandra,
4. 图型
neo4j

## MongoDB简介
### 与关系型数据库的对比
关系型:字段-行-表-数据库
mongo:字段-文档-集合-数据库

|    |关系型 |文档型 |
|------ |---- |---- |
|表结构 |需要事先定义 |集合结构无需定义(无模式) |
|字段和属性 |同一行的字段必须一致 |同一个集合的文档属性可以有差异 |
|数据类型 |只有有限的几种 |很多,数组,嵌入式文档等复杂数据结构 |


















