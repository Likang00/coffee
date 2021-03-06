---
title: SQL必知必会
date: 2019-07-08 10:43:53
tags:
---

# MySQL 必知必会

## 命令行登录mysql
添加mysql.exe所在路径到系统环境变量

-h : 指定客户端所要登录的 MySQL 主机名, 登录本机(localhost 或 127.0.0.1)该参数可以省略;
-P : 端口号(大写)
-u : 登录的用户名;
-p : 告诉服务器将会使用一个密码来登录, 如果所要登录的用户名密码为空, 可以忽略此选项。

*本地*
`mysql -u root -p`

*腾讯云mysql*
`mysql -h cdb-a5oi3xsc.bj.tencentcdb.com -P 10087 -u root -p`

## 了解数据库和表
```sql
-- 获取数据库和表信息
SHOW DATABASES;
USE likang;
SHOW TABLES;
SHOW COLUMNS FROM customers;
DESCRIBE customers; -- 同上
SHOW STATUS;
SHOW CREATE TABLE customers;
SHOW GRANTS;
```

## 检索数据
```sql
-- 检索单个列 
select columnName from tableName;
-- 检索多个列 
select columnName1, columnName2 ... from tableName;
-- 检索所有列 
select * from tableName;
-- 检索不同的行 
select distinct from tableName;
-- 限制结果 
select * from tableName limit 5;
-- 使用完全限定的表名
select tableName.columnName from tableName;
```

## 排序检索数据
```sql
-- 排序数据 (用非检索的列排序数据是完全合法的)
select columnName from tableName order by columnName;
-- 按多个列排序
select columnName1, columnName2 from tableName order by columnName1, columnName2;
-- 指定排序方向 (若不指定 则默认为升序asc)
select columnName from tableName order by columnName desc;
```

## 过滤数据
```sql
-- where 子句
select columnName from tableName where columnName = 'xxx';
-- where 子句操作符
= <> != < <= > >= between 
```

## 数据过滤
```sql
-- 组合where子句
and or 计算次序 in not
```

## 用通配符进行过滤
```sql
-- LIKE操作符
-- % 表示 任何字符出现任意次数
select * from tableName where column like 'xxx%';
-- _表示 任何字符出现一次
select * from tableName where column like 'xxx_';
```

## 用正则表达式进行搜素
```sql
select * from tableName where column regexp 'xxx';
```

## 计算字段
```sql
-- 拼接
concat()
-- 算数
+ - * /
```

## 函数 
可移植性不强 几乎每个DBMS都支持特有的函数
大体分为四类
1.字符串处理函数
2.日期时间函数
3.数值处理函数
4.自定义函数

## 汇总数据
**聚集函数**
```sql
-- MySQL聚集函数
AVG() 平均
COUNT() 行数
MAX() 最大
MIN() 最小
SUM() 求和
-- 聚集不同的值
select AVG(DISTINCT prod_pric) as avg_price
from products where vend_id = 1003;
```

## 分组数据
过滤分组使用 *HAVING*
```sql
-- HAVING 过滤分组
SELECT cust_id, COUNT(*) AS orders
FROM orders 
GROUP BY cust_id
HAVING COUNT(*) >= 2;
-- 列出具有2个及以上、价格为10及以上的产品的供应商
SELECT vend_id, COUNT(*) AS num_prods
FROM products
WHERE prod_price >= 10
GROUP BY vend_id
HAVING COUNT(*) >= 2;
-- where 在数据分组前进行过滤
-- having 在数据分组后进行过滤
```

## 子查询
## 联结表
```sql
-- 没有联结条件的表关系 返回的结果为笛卡尔积 行数将为第一张表的行数*第二张表的行数
-- 内部联结
/* 用三种方法实现 */

select cust_name, cust_contact
from customers
where cust_id in (
	select cust_id
	from orders
	where order_num in (
		select order_num
		from orderitems
		where prod_id = 'TNT2'
		)
);

SELECT c.cust_name, c.cust_contact
FROM customers AS c, orders AS o, orderitems AS i
WHERE c.cust_id = o.cust_id
AND i.order_num = o.order_num
AND i.prod_id = 'TNT2';

select c.cust_name, c.cust_contact
from customers as c
inner join orders as o on c.cust_id = o.cust_id
inner join orderitems as i on o.order_num = i.order_num
where i.prod_id = 'TNT2';

-- 自联结
select p1.prod_id, p1.prod_name
from products as p1, products as p2
where p1.vend_id = p2.vend_id
and p2.prod_id = 'DTNTR';

-- 使用带聚集函数的联结
select c.cust_id, c.cust_name, count(o.order_num) as num_ord
from customers as c
left join orders as o
on c.cust_id = o.cust_id
group by c.cust_id
order by num_ord;
```

## 组合查询
**UNION**
```sql
-- 默认取消重复的行
-- 若要包含 使用 union all
-- 对组合查询进行排序
select vend_id, prod_id, prod_price
from products
where prod_price <= 5
union 
select vend_id, prod_id, prod_price
from products
where vend_id in (1001,1002)
order by vend_id, prod_price;
```

## 全文检索
```sql
CREATE TABLE `productnotes` (
  `note_id` int(11) NOT NULL AUTO_INCREMENT,
  `prod_id` char(10) COLLATE utf8_bin NOT NULL,
  `note_date` datetime NOT NULL,
  `note_text` text COLLATE utf8_bin,
  PRIMARY KEY (`note_id`),
  FULLTEXT KEY `note_text` (`note_text`)
) ENGINE=InnoDB AUTO_INCREMENT=115 DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

## 存储过程
```sql

```

## 游标
MySQL游标只能用于存储过程(和函数)
使用游标设计到的几个步骤: 声明游标 打开游标 检索数据 关闭游标




# MySQL数据类型

## 串数据类型
数据类型 | 说明 
:-: | :-: 
CHAR | 1~255个字符的定长串
VARCHAR | 长度可变 最多不超过255字节
ENUM | 接受最多64K个串组成的一个预定义集合的某个串
SET | 接受最多64K个串组成的一个预定义集合的零个或多个串
TEXT | 最大长度为64K的长文本
MEDIUMTEXT | 最大长度16K
TINYTEXT | 最大长度255字节
LONGTEXT | 最大长度4GB

## 数值数据类型
数据类型 | 说明 
:-: | :-: 
BIT | 位字段，1～64位。
BIGINT | 整数值，支持-9223372036854775808～9223372036854775807的数 
BOOLEAN | （或BOOL） 布尔标志，或者为0或者为1，主要用于开/关（on/off）标志 
DECIMAL | （或DEC） 精度可变的浮点值 
DOUBLE |  双精度浮点值 
FLOAT |  单精度浮点值 
INT | （或INTEGER） 整数值，支持-2147483648～2147483647的数 
MEDIUMINT |  整数值，支持-8388608～8388607（如果是UNSIGNED，为0～ 16777215）的数 
REAL |  4字节的浮点值 
SMALLINT |  整数值，支持-32768～32767（如果是UNSIGNED，为0～ 65535）的数 
TINYINT |  整数值，支持-128～127（如果为UNSIGNED，为0～255）的数 

## 日期时间类型
数据类型 | 说明 
:-: | :-: 
DATE | 表示1000-01-01～9999-12-31的日期，格式为 YYYY-MM-DD 
DATETIME |  DATE和TIME的组合 TIMESTAMP 功能和DATETIME相同（但范围较小） 
TIME | 格式为HH:MM:SS 
YEAR | 用2位数字表示，范围是70（1970年）～69（2069 年），用4位数字表示，范围是1901年～2155年 D

## 二进制数据类型
数据类型 | 说明 
:-: | :-: 
BLOB | Blob最大长度为64 KB 
MEDIUMBLOB | Blob最大长度为16 MB 
LONGBLOB | Blob最大长度为4 GB 
TINYBLOB | Blob最大长度为255字节 
