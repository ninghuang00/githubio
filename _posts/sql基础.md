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

