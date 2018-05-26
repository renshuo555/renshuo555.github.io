---
layout: post
title: 《MySQL必知必会》读书笔记（三）
date: 2018-5-24 22:33:00
categories: 数据库
tags: MySQL 数据库
author: renshuo
mathjax: true
---

* content
{:toc}
《MySQL必知必会》读书笔记，对基础数据库语句做了总结，方便以后查看

本篇包含19 ~ 23章内容

<!--more-->

## 插入数据

`INSERT` 是用来插入（或添加）行到数据库表的，插入可以分为以下几种方式，插入完整行；插入行的一部分；插入多行；插入某些查询结果；

### 插入完整的行

``` sql
INSERT INTO <表> VALUES(<按照列的顺序输入值，不能忽略，可以给null>)

INSERT INTO <表名>(<列1>, <列2>, ...) VALUES(<前面列名对应的值>, ...)
```

### 插入多个行

可以使用多条INSERT语句，甚至一次提交它们，每条语句用一个分号结束。

或者，只要每条`INSERT`语句中的列名（和次序）相同，可以在VELUES后面使用逗号分隔不同的行值，每个行的值是括在小括号里面的

### 插入检索出的数据

``` sql
INSERT INTO <表>(<列>, <列>, ...) SELECT <列>, <列>, ... FROM <表>
```

`INSERT SELECT`中`SELECT`语句可包含WHERE子句以过滤插入的数据。

## 更新和删除数据

### 更新数据

使用UPDATE语句更新（修改）表中的数据，可以更新特定行或者所有行。

UPDATE语句由三部分组成：要更新的表；列名和它们的新值；确定要更新行的过滤条件。

``` sql
UPDATE customers SET cust_name = 'The Fudds', cust_email = 'elemer@fudd.com' WHERE cust_id = '10005';
```

**`IGNORE`关键字** 如果用UPDATE语句更新多行，并且在更新这些行中的一行或多行时出一个现错误，则整个UPDATE操作被取消（错误发生前更新的所有行被恢复到它们原来的值）。为即使是发生错误，也继续进行更新，可使用IGNORE关键字，如下所示：`UPDATE IGNORE customers ...`

### 删除数据

使用DELETE可以冲一个表中删除数据，可以删除特定的行，也可以删除所有行。

**不要省略`WHELE`子句！不要省略`WHELE`子句！不要省略`WHELE`子句！**

``` sql
DELETE FROM customers WHERE cust_id = 10006;
```

如果想从表中删除所有行，不要使用`DELETE`。可使用`TRUNCATE TABLE`语句，它完成相同的工作，但速度更快（`TRUNCATE`实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据）。

### 更新和删除的指导原则

- 除非确实打算更新和删除每一行，否则绝对不要使用不带`WHERE`子句的`UPDATE`或`DELETE`语句
- 保证每个表都有主键，尽可能像WHERE子句那样使用它（可以指定各主键、多个值或值的范围）
- 在对`UPDATE`或`DELETE`语句使用`WHERE`子句前，应该先用`SELECT`进行测试，保证它过滤的是正确的记录，以防编写的`WHERE`子句不正确
- 使用强制实施引用完整性的数据库，这样MySQL将不允许删除具有与其他表相关联的数据的行

## 创建和操纵表

### 创建表

利用`CREATE TABLE` 创建表，需要给出以下信息：

- 新表的名字，在关键字`CREATE TABLE之`后给出；
- 表列的名字和定义，用逗号分隔。

```sql
CREATE TABLE customers
(
    cust_id int NOT NULL AUTO_INCTEMENT,
    quantity int NOT NULL DEFAULT 1,
    <列名> <数据类型> <描述>,
    ...
    PRIMARY KEY(cust_id)
)ENGING=InnoDB;
```

每个表列或者是`NULL`列，或者是`NOT NULL`列，这种状态在创建时由表的定义规定。

为创建由多个列组成的主键，应该以逗号分隔的列表给出各列名，例如： `PRIMARY KEY(order_num, order_item)`

`AUTO_INCREMENT`告诉MySQL，本列每当增加一行时自动增量。每次执行一个`INSERT`操作时， MySQL自动对该列增量（从而才有这个关键字`AUTO_INCREMENT`），给该列赋予下一个可用的值。这样给每个行分配一个唯一的cust_id，从而可以用作主键值。

**每个表只允许一个`AUTO_INCREMENT`列，而且它必须被索引（如，通过使它成为主键）。**

**指定默认值：** 给该列的描述添加文本`DEFAULT 1`指示MySQL，在未给出数量的情况下使用数量1。

### 引擎类型

如果省略ENGINE=语句，则使用默认引擎（很可能是MyISAM），多数SQL语句都会默认使用它。但并不是所有语句都默认使用它，这就是为什么ENGINE=语句很重要的原因。

- InnoDB是一个可靠的事务处理引擎，它不支持全文本搜索；
- MEMORY在功能等同于MyISAM， 但由于数据存储在内存（不是磁盘）中，速度很快（特别适合于临时表）；
- MyISAM是一个性能极高的引擎，它支持全文本搜索，但不支持事务处理。

外键不能跨引擎，即使用一个引擎的表不能引用具有使用不同引擎的表的外键。

### 更新表

为更新表定义，可使用`ALTER TABLE`语句。

```sql
ALTER TABLE vendors ADD vend_phone CHAR(20);

ALTER TABLE vendors DROP COLUMN vend_phone;
```

ALTER TABLE的一种常见用途是定义外键。

```sql
ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);
```

### 删除表

```sql
DROP TABLE customers;
```

### 重命名表

```sql
RENAME TABLE customers2 TO customers, vendors2 TO vendors;
```

## 使用视图

视图是虚拟的表。与包含数据的表不一样，视图只包含使用时动态检索数据的查询。视图的常见应用：

- 重用SQL语句。
- 简化复杂的SQL操作。在编写查询后，可以方便地重用它而不必知道它的基本查询细节。
-  使用表的组成部分而不是整个表。
- 保护数据。可以给用户授予表的特定部分的访问权限而不是整个表的访问权限。
- 更改数据格式和表示。视图可返回与底层表的表示和格式不同的数据。

### 视图的规则和限制

- 与表一样，视图必须唯一命名（不能给视图取与别的视图或表相同的名字）。
- 对于可以创建的视图数目没有限制。
- 为了创建视图，必须具有足够的访问权限。这些限制通常由数据库管理人员授予。
- 视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造一个视图。
- ORDER BY可以用在视图中，但如果从该视图检索数据SELECT中也含有ORDER BY，那么该视图中的ORDER BY将被覆盖。
- 视图不能索引，也不能有关联的触发器或默认值。
- 视图可以和表一起使用。例如，编写一条联结表和视图的SELECT语句。

### 使用视图

- 视图用CREATE VIEW语句来创建。
- 使用`SHOW CREATE VIEW viewname;`来查看创建视图的语句。
- 用DROP删除视图，其语法为`DROP VIEW viewname;`。
- 更新视图时，可以先用DROP再用CREATE，也可以直接用CREATE OR REPLACE VIEW。如果要更新的视图不存在，则第2条更新语句会创建一个视图；如果要更新的视图存在，则第2条更新语句会替换原有视图。

使用视图简化复杂的联结

``` sql
CREATE VIEW productcustomers AS
SELECT cust_name, cust_contact, prod_id
FROM customers, orders, orderitems
WHERE customers.cust_id = orders.cust_id
	AND orderitems.order_num = orders.order_num;
```

用视图重新格式化检索出的数据

```sql
CREATE VIEW vendorlocations AS 
SELECT Concat(RTrim(vend_name), '(', RTrim(vend_country),')') AS vend_title
FROM vendors
ORDER BY vend_name;
```

用视图过滤不想要的数据

```sql
CREATE VIEW customermaillist AS
SELECT cust_id, cust_name, cust_mail
FROM customers
WHERE cust_mail IS NOT NULL;
```

使用视图与计算字段

```sql
CREATE VIEW orderitemsexpanded AS
SELECT order_num, prod_id, quantity*item_price AS expanded_price
FROM orderitems;
```

### 更新视图

通常，视图是可更新的，使用INSERT、 UPDATE和DELETE。但是并非所有的视图都是可更新的。如果视图定义中又以下操作，则不能更新

- 分组（使用GROUP BY和HAVING）
- 联结
- 子查询
- 并
- 聚集函数（Min()、Count()、Sum() 等）
- DISTINCT
- 导出（计算）列

上面列出的限制自MySQL 5以来是正确的。不过，未来的MySQL很可能会取消某些限制。

## 使用存储过程

存储过程简单来说，就是为以后的使用而保存的一条或多条MySQL语句的集合。可将其视为批文件，虽然它们的作用不仅限于批处理。存储过程有3个主要的好处，简单、安全、高性能。

``` sql
CALL productpricing(@pricelow, @pricehigh, @priceaverage);  --执行名为productpricing的存储过程，它计算并返回产品的最低、最高和平均价格创建
```

所有MySQL变量都必须以@开始。

在调用时，这条语句并不显示任何数据。它返回以后可以显示（或在其他处理中使用）的变量。

**创建存储过程**

```sql
CREATE PROCEDURE productpricing()
BEGIN
	SELECT Avg(prod_price) AS priceaverage
	FROM products;
END;
```

**删除存储过程**

```sql
DROP PROCEDURE productpricing;
```

**使用参数**

```sql
CREATE PROCEDURE productpricing(
	OUT pl DECIMAL(8,2),
    OUT ph DECIMAL(8,2),
    OUT pa DECIMAL(8,2)
)
BEGIN
	SELECT Min(prod_price)
	INTO pl
	FROM products;
	SELECT Max(prod_price)
	INTO ph
	FROM products;
	SELECT Avg(prod_price)
	INTO pa
	FROM products;
END;
```

关键字OUT指出相应的参数用来从存储过程传出一个值（返回给调用者）。 MySQL支持`IN`（传递给存储过程）、 `OUT`（从存储过程传出，如这里所用）和`INOUT`（对存储过程传入和传出）类型的参数。

```sql
CREATE PROCEDURE ordertotal(
	IN onumber INT,
    OUT ototal DECIMAL(8,2)
)
BEGIN
	SELECT SUM(item_price*quantity)
	FROM orderitems
	WHERE order_num = onumber
	INTO ototal;
END;    -- 创建

CALL ordertatal(20005, @total);   -- 调用
```

**建立智能存储过程**

``` sql
-- Name:ordertotal
-- Parmeters:onumber = order number
--           taxable = 0 if not taxable
--           ototal = order total variable
CREATE PROCEDURE ordretotal(
IN onumber INT,
IN taxable BOOLEAN,
OUT ototal DECIMAL(8,2)
)COMMENT 'obtain order total,optiomally adding tax'
BEGIN
	-- dexlare variable for total
	DECLARE total DECIMAL(8,2);
	-- delcare tax percentage
	DECLARE taxtate INT DEFAULT 6;
	-- get the ordertotal
	SELECT Sum(item_price*quantity)
	FROM orderitems
	WHERE order_num = onumber INTO total;
	-- is this taxable?
	IF taxable THEN
		-- yes,so add taxrate to the total
		SELECT total+(total/100*taxrate) INTO total;
	END IF;
	-- and finally,save to out variable
	SELECT total INTO ototal;
END;
```

本例子中的存储过程在`CREATE PROCEDURE`语句中包含了一个`COMMENT`值。它不是必需的，但如果给出，将在`SHOW PROCEDURE STATUS`的结果中显示。

这个例子给出了MySQL的IF语句的基本用法。 IF语句还支持ELSEIF和ELSE子句（前者还使用THEN子句，后者不使用）。

**检查存储过程**

``` sql
SHOW CREATE PROCEDURE ordertotal;  -- 显示用来创建一个存储过程的CREATE语句

SHOW PROCEDURE STATUS;   -- 获得包括何时、由谁创建等详细信息的存储过程列表
```

