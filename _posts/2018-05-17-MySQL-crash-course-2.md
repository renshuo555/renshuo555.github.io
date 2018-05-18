---
layout: post
title: 《MySQL必知必会》读书笔记（二）
date: 2018-5-17 10:19:35
categories: 数据库
tags: MySQL 数据库
author: renshuo
mathjax: true
---

* content
{:toc}
《MySQL必知必会》读书笔记，对基础数据库语句做了总结，方便以后查看

本篇包含12 ~ 18章内容

<!--more-->

## 汇总数据

### 聚集函数

- 确定表中行数（或者满足某个条件或包含某个特定值的行数）
- 获得表中行组的和
-  找出表列（或所有行或某些特定的行）的最大值、最小值和平均值

|函 数| 说 明|
|-----|-----|
|`AVG()` |返回某列的平均值|
|`COUNT()` |返回某列的行数|
|`MAX()` |返回某列的最大值|
|`MIN()` |返回某列的最小值|
|`SUM()` |返回某列值之和|

#### AVG()

```sql
SELECT AVG(prod_price) AS avg_price FROM products WHERE ven_id = 1003;
```

`AVG()`只能用来确定特定数值列的平均值，而且列名必须作为函数参数给出。**为了获得多个列的平均值，必须使用多个`AVG()`函数**，`AVG()`函数忽略列值为NULL的行。

#### COUNT()

- 使用`COUNT(*)`对表中行的数目进行计数， 不管表列中包含的是空值（`NULL`）还是非空值
- 使用COUNT(column)对特定列中具有值的行进行计数，忽略NULL值

#### MAX() & MIN()

`MAX() `和 `MIN()` 函数忽略列值为NULL的行

#### SUM()

``` sql
SELECT SUM(item_price*quantity) AS total_price FROM orderitems WHERE order_num = 20005;
```

### 聚集不同值

对以上5个聚集函数都可以如下使用：

- 对所有的行执行计算，指定ALL参数或不给参数（因为ALL是默认行为）
- 只包含不同的值，指定DISTINCT参数

如果指定列名，则`DISTINCT`只能用于`COUNT()`。`DISTINCT`不能用于`COUNT(*)`，因此不允许使用`COUNT（ DISTINCT）`，否则会产生错误。类似地， `DISTINCT`必须使用列名，不能用于计算或表达式。

## 分组数据

### 创建分组

```sql
SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;
```

- GROUP BY子句可以包含任意数目的列。这使得能对分组进行嵌套，为数据分组提供更细致的控制。
- 如果在GROUP BY子句中嵌套了分组，数据将在最后规定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算（所以不能从个别的列取回数据）。
- GROUP BY子句中列出的每个列都必须是检索列或有效的表达式（但不能是聚集函数）。如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
- 除聚集计算语句外， SELECT语句中的每个列都必须在GROUP BY子句中给出。
- 如果分组列中具有NULL值，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。
- GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。

### 过滤分组

``` sql
SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*) >=2;
```

HAVING和WHERE的差别，可以有两种理解方式：

1. `WHERE`过滤行，`HAVING`过滤分组。
2. WHERE在数据分组前进行过滤， HAVING在数据分组后进行过滤。这是一个重要的区别， WHERE排除的行不包括在分组中。这可能会改变计算值，从而影响HAVING子句中基于这些值过滤掉的分组。

``` sql
SELECT vend_id, COUNT(*) AS num_prods FROM products WHERE prod_price >= 10 GROUP BY vend_id HAVING COUNT(*) >= 2;
```

### 分组和排序

ORDER BY与GROUP BY经常完成相同的工作，但是他们是非常不同的：

|ORDER BY |GROUP BY|
|-----------------------|---|
|排序产生的输出  | 分组行。但输出可能不是分组的顺序|
|任意列都可以使用（甚至非选择的列也可以使用）|只可能使用选择列或表达式列，而且必须使用每个选择列表达式|
|不一定需要      |如果与聚集函数一起使用列（或表达式），则必须使用|

**一般在使用GROUP BY子句时，应该也给出ORDER BY子句。**

### SELECT子句及其顺序

|子 句| 说 明 |是否必须使用|
|----|----|----|
|`SELECT` |要返回的列或表达式| 是|
|`FROM`| 从中检索数据的表 |仅在从表选择数据时使用|
|`WHERE` |行级过滤| 否|
|`GROUP BY`| 分组说明 |仅在按组计算聚集时使用|
|`HAVING` |组级过滤 |否|
|`ORDER BY` |输出排序顺序| 否|
|`LIMIT` |要检索的行数| 否|

## 使用子查询

子查询（ subquery） ，即嵌套在其他查询中的查询。

```sql
SELECT order_num FROM orderitems WHERE prod_id = 'TNT2';  --检索包含物品TNT2的所有订单的编号

SELECT cust_id FROM orders WHERE order_num IN (20005,20007); --检索具有前一步骤列出的订单编号的所有客户的ID
```

现在把上面两个组合起来

``` sql
SELECT cust_id FROM orders WHERE order_num IN (SELECT order_num FROM orderitems WHERE prod_id = 'TNT2');
```

```sql
SELECT cust_name, cust_contact FROM customers WHERE cust_id IN (10001,10004); --检索这些客户ID的客户信息
```

把上面的都组合起来

``` sql
SELECT cust_name, cust_contact 
FROM customers 
WHERE cust_id 
IN (SELECT cust_id 
    FROM orders 
    WHERE order_num 
    IN (SELECT order_num 
        FROM orderitems 
        WHERE prod_id = 'TNT2'));
```

### 作为计算字段使用子查询

使用子查询的另一方法是创建计算字段。

``` sql
SELECT cust_name, cust_state,(SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders FROM customers ORDER BY cust_name;
```

## 联结表

SQL最强大的功能之一就是能在数据检索查询的执行中联结（join）表

### 创建联结

``` sql
SELECT vend_name, prod_name, prod_price FROM vendors, products WHERE vendors.vend_id = products.vend_id ORDER BY vend_name, prod_name;
```

应该保证所有联结都有`WHERE`子句，否则MySQL将返回比想要的数据多得多的数据。同理，应该保证`WHERE`子句的正确性。不正确的过滤条件将导致MySQL返回不正确的数据。

### 内部联结

目前为止所用的联结称为等值联结（equijoin），它基于两个表之间的相等测试。这种联结也称为内部联结。

``` sql
SELECT vend_name, prod_name, prod_price FROM vendors INNER JOIN products ON vendors.vend_id = products.vend_id;
```

### 联结多个表

```sql
SELECT prod_name, vend_name, pro_price, quantity
FROM orderitems, products, vendors
WHERE products.vend_id = vendors.vend_id
	AND orderitems.prod_id = products.prod_id 
	AND order_num = 20005;
```

MySQL在运行时关联指定的每个表以处理联结。这种处理可能是非常耗费资源的，因此应该仔细，不要联结不必要的表。联结的表越多，性能下降越厉害。

## 创建高级联结

### 使用表别名

``` sql
SELECT cust_name, cust_contact
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.cust_id = o.cust_id
	AND oi.order_num = o.order_num
	AND prod_id = 'TNT2';
```

### 使用不同类型的联结

上面使用的都是内部联结或等值联结（equijoin）的简单联结，还有自联结、自然联结、外部联结三种。

#### 自联结

``` sql
SELECT prod_id, prod_name FROM products WHERE vend_id = (SELECT vend_id FROM products WHERE prod_id = 'DTNTR'); 	 -- 使用了子查询

SELECT p1.prod_id, p1.prod_name FROM products AS p1, products AS p2 WHERE p1.vend_id = p2.vend_id AND p2.prod_id = 'DTNTR';     -- 使用自联结
```

有时候处理联结远比处理子查询快得多

#### 自然联结

自然联结排除多次出现，使每个列只返回一次。事实上，迄今为止我们建立的每个内部联结都是自然联结，很可能我们永远都不会用到不是自然联结的内部联结。

#### 外部联结

``` sql
SELECT customers.cust_id, orders.order_num FROM customers LEFT OUTER JOIN orders on customers.cust_id = order.cust_id;
```

外部联结分为左外部联结(`LEFT OUTER JOIN`)和右外部联结(`RIGHT OUTER JOIN`)两种。外部联结就是把对应的左或者右边表中所以的数据都显示，另一边的只显示符合条件的数据，没有符合条件的就显示 `NULL`。

### 使用带聚集函数的联结

```sql
SELECT customers.cust_name, customers.cust_id, COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders ON customers.cust_id = orders.cust_id 
GROUP BY customers.cust_id;

SELECT customers.cust_name, customers.cust_id, COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders ON customers.cust_id = orders.cust_id 
GROUP BY customers.cust_id;
```

## 组合查询

- 在单个查询中从不同的表返回类似结构的数据；
- 对单个表执行多个查询，按单个查询返回数据。

多数情况下，组合相同表的两个查询完成的工作与具有多个WHERE子句条件的单条查询完成的工作相同。

### 创建组合查询

#### 使用UNION

`UNION`的使用很简单。所需做的只是给出每条SELECT语句，在各条语句之间放上关键字`UNION`。

``` sql
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001,1002);

SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5 OR vend_id IN (1001,1002);    -- 功能相同
```

#### UNION规则

- UNION必须由两条或两条以上的SELECT语句组成，语句之间用关键字UNION分隔
-  UNION中的每个查询必须包含相同的列、表达式或聚集函数（不过各个列不需要以相同的次序列出）
- 列数据类型必须兼容：类型不必完全相同，但必须是DBMS可以隐含地转换的类型（例如，不同的数值类型或不同的日期类型）

#### 包含或取消重复的行

`UNION`从查询结果集中自动去除了重复的行，如果想返回所有匹配行，可使用`UNION ALL`而不是`UNION`。

#### 对组合查询结果排序

`SELECT`语句的输出用`ORDER BY`子句排序。在用`UNION`组合查询时，只能使用一条`ORDER BY`子句，它必须出现在最后一条`SELECT`语句之后。对于结果集，**不存在用一种方式排序一部分，而又用另一种方式排序另一部分的情况**，因此不允许使用多条`ORDER BY`子句。

```sql
SELECT vend_id, prod_id, prod_price FROM products WHERE prod_price <= 5
UNION
SELECT vend_id, prod_id, prod_price FROM products WHERE vend_id IN (1001,1002)
ORDER BY vend_id, prod_price;
```

使用`UNION`的组合查询可以应用**不同的表**。

## 全文本搜索

并非所有引擎都支持全文本搜索，两个最常使用的引擎为MyISAM和InnoDB，前者支持全文本搜索，而后者不支持。

为了进行全文本搜索，必须索引被搜索的列，在索引之后， `SELECT`可与`Match()`和`Against()`一起使用以实际执行搜索。

### 启动并进行全文本搜索

在创建表时，使用 `FULLTEXT` 子句对列进行索引。

`Match()`指定被搜索的列， `Against()`指定要使用的搜索表达式。

```sql
SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit');
```

传递给 `Match()` 的值必须与`FULLTEXT()`定义中的相同。如果指定多个列，则必须列出它们（而且次序正确）。

除非使用`BINARY`方式，否则全文本搜索不区分大小写。

全文本搜索的一个重要部分就是对结果排序。具有较高等级的行先返回。比如在文本中，查找的次靠前的等级高。如果指定多个搜索项，则包含多数匹配词的那些行将具有比包含较少词（或仅有一个匹配）的那些行高的等级值。

### 使用查询扩展

- 首先，进行一个基本的全文本搜索，找出与搜索条件匹配的所有行；
- 其次， MySQL检查这些匹配行并选择所有有用的词。
- 再其次， MySQL再次进行全文本搜索，这次不仅使用原来的条件，而且还使用所有有用的词。

利用查询扩展，能找出可能相关的结果，即使它们并不精确包含所查找的词。

```sql
SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit' WITH QUERY EXPANSION);
```

表中的行越多（这些行中的文本就越多），使用查询扩展返回的结果越好。

### 布尔文本搜索

即使没有FULLTEXT索引也可以使用**布尔方式（boolean mode）**

- 要匹配的词；
- 要排斥的词（如果某行包含这个词，则不返回该行，即使它包含其他指定的词也是如此）；
- 排列提示（指定某些词比其他词更重要，更重要的词等级更高）；
- 表达式分组；
- 另外一些内容

```sql
SELECT note_text FROM productnotes WHERE Match(note_text) Against('heavy -rope*' IN BOOLEAN MODE);    -- 匹配包含heavy但不包含任意以rope开始的词的行
```

|布尔操作符 |说明|
|----|-----|
|`+` |包含，词必须存在|
|`-` |排除，词必须不出现|
|`>` |包含，而且增加等级值|
|`<` |包含，且减少等级值|
|`()` |把词组成子表达式（允许这些子表达式作为一个组被包含、排除、排列等）|
|`~` |取消一个词的排序值|
|`*` |词尾的通配符|
|`""` |定义一个短语（与单个词的列表不一样，它匹配整个短语以便包含或排除这个短语）|

在布尔方式中，不按等级值降序排序返回的行。

```sql
SELECT note_text FROM productnotes WHERE Match(note_text) Against('rabbit bait' IN BOOLEAN MODE);    -- 匹配包含rabbit和bait中的至少一个词的行
```

### 全文本搜索使用说明

- 在索引全文本数据时，短词被忽略且从索引中排除。短词定义为那些具有3个或3个以下字符的词（如果需要，这个数目可以更改）。
-  MySQL带有一个内建的非用词（stopword）列表，这些词在索引全文本数据时总是被忽略。如果需要，可以覆盖这个列表（请参阅MySQL文档以了解如何完成此工作）。
- 许多词出现的频率很高，搜索它们没有用处（返回太多的结果）。因此， MySQL规定了一条50%规则，如果一个词出现在50%以上的行中，则将它作为一个非用词忽略。 50%规则不用于IN BOOLEAN MODE。
- 如果表中的行数少于3行，则全文本搜索不返回结果（因为每个词或者不出现，或者至少出现在50%的行中）。
- 忽略词中的单引号。例如， `don't`索引为`dont`。
- 不具有词分隔符（包括日语和汉语）的语言不能恰当地返回全文本搜索结果。
- 如前所述，仅在MyISAM数据库引擎中支持全文本搜索。