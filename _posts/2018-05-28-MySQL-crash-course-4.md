---
layout: post
title: 《MySQL必知必会》读书笔记（四）
date: 2018-5-24 22:33:00
categories: 数据库
tags: MySQL 数据库
author: renshuo
mathjax: true
---

* content
{:toc}
《MySQL必知必会》读书笔记，对基础数据库语句做了总结，方便以后查看

本篇包含25 ~ 28章内容

第 29 章 数据库维护 和 第 30 章 改善性能 主要以文字为主，请阅读原书。

<!--more-->

## 使用触发器

### 触发器

需要在某个**表**发生更改时自动处理。

触发器是MySQL响应以下任意语句而自动执行的一条MySQL语句，或位于`BEGIN`和`END`语句之间的一组语句。最好保持每个数据库的触发器名称唯一，每个表可以建立6个触发器（3条语句，之前之后）。

**`DELETE`；`INSERT`；`UPDATE`**

### 创建触发器

在创建触发器时，需要给出4条信息：

- 唯一的触发器名；
- 触发器关联的表；
- 触发器应该响应的活动（`DELETE`、 `INSERT`或`UPDATE`）；
- 触发器何时执行（处理之前或之后）。

```sql
CREATE TRIGGER newporduct AFTER INSERT ON products
FOR EACH ROW SELECT 'Product added';
```

### 删除触发器

```sql
DROP TRIGGER newproduct;
```

触发器不能更新或者覆盖，只能先删除，再重现创建。

### 使用触发器

#### INSERT 触发器

- 在INSERT触发器代码内，可引用一个名为NEW的虚拟表，访问被插入的行；
- 在BEFORE INSERT触发器中， NEW中的值也可以被更新（允许更改被插入的值）；
- 对于AUTO_INCREMENT列， NEW在INSERT执行之前包含0，在INSERT执行之后包含新的自动生成值。

```sql
CREATE TRIGGER neworder AFTER INSERT ON orders
FOR EACH ROW SELECT NEW.order_num;
```

#### DELETE 触发器

- 在DELETE触发器代码内，你可以引用一个名为OLD的虚拟表，访问被删除的行；
- OLD中的值全都是只读的，不能更新。

```sql
CREATE TRIGGER deleteorder BEFORE DELETE ON order
FOR EACH ROW
BEGIN
	INSERT INTO archive_order(order_num, order_date, cust_id)
	VALUES(OLD.order_num, OLD.order_date, OLD.cust_id)
END;
```

**多语句触发**：触发器deleteorder使用`BEGIN`和`END`语句标记触发器体。这在此例子中并不是必需的，不过也没有害处。使用BEGIN END块的好处是触发器能容纳多条SQL语句（在BEGIN END块中一条挨着一条）。

#### UPDATE 触发器

- 在UPDATE触发器代码中，你可以引用一个名为OLD的虚拟表访问以前（UPDATE语句前）的值，引用一个名为NEW的虚拟表访问新更新的值；
- 在BEFORE UPDATE触发器中， NEW中的值可能也被更新（允许更改将要用于UPDATE语句中的值）；
- OLD中的值全都是只读的，不能更新。

```sql
CREATE TRIGGER updatevendor BEFORE UPDATE ON vendors
FOR EACH ROW SET NEW.vend_state = Upper(NEW.vend_state);
```

## 管理事务处理

### 事务处理

**并非所有引擎都支持明确的事务处理管理**。 MyISAM和InnoDB是两种最常使用的引擎。前者不支持明确的事务处理管理，而后者支持。

事务处理（transaction processing）可以用来维护数据库的完整性，它保证成批的MySQL操作要么完全执行，要么完全不执行。

- **事务**（transaction）指一组SQL语句；
- **回退**（rollback）指撤销指定SQL语句的过程；
-  **提交**（commit）指将未存储的SQL语句结果写入数据库表；
- **保留点**（savepoint）指事务处理中设置的临时占位符（placeholder），你可以对它发布回退（与回退整个事务处理不同）。

### 控制事务处理

管理事务处理的关键在于将SQL语句组分解为逻辑块，并明确规定数据何时应该回退，何时不应该回退。

```sql
START TRANSACTION   --标识事务的开始
```

#### 使用ROLLBACK

MySQL的`ROLLBACK`命令用来回退（撤销） MySQL语句

```sql
SELECT * FROM ordertotals;
START TRANSACTION;
DELETE FROM ordertotals;
SELECT * FROM ordertotals;
ROLLBACK;
SELECT * FROM ordertatals；
```

`ROLLBACK`只能在一个事务处理内使用（在执行一条`START TRANSACTION`命令之后）

事务处理用来管理`INSERT`、 `UPDATE`和`DELETE`语句。你不能回退`SELECT`语句，不能回退`CREATE`或`DROP`操作。

#### 使用COMMIT

```sql
START TRANSACTION;
DELETE FROM orderitems WHERE order_num = 20010;
DELETE FROM order WHERE order_num = 20010;
COMMIT;    -- 最后的COMMIT语句仅在不出错时写出更改
```

#### 使用保留点

简单的ROLLBACK和COMMIT语句就可以写入或撤销整个事务处理，更复杂的事务处理可能需要部分提交或回退。

``` sql
SAVEPOINT delete1;   -- 创建占位符,标识名为 delete1

ROLLBACK TO delete1;
```

可以在MySQL代码中设置任意多的保留点，越多越好。

保留点在事务处理完成（执行一条`ROLLBACK`或`COMMIT`）后自动释放。自MySQL 5以来，也可以用`RELEASE SAVEPOINT`明确地释放保留点。

#### 更改默认的提交行为

```sql
SET autocommit = 0;    -- 指示MySQL不自动提交更改
```

`autocommit`标志决定是否自动提交更改，设置autocommit为0（假）指示MySQL不自动提交更改（直到`autocommit`被设置为真为止）。

`autocommit`标志是**针对每个连接**而不是服务器的。

## 全球化和本地化

### 字符集和校对顺序

- **字符集**为字母和符号的集合；
- **编码**为某个字符集成员的内部表示；
- **校对**为规定字符如何比较的指令。

### 使用字符集和校对顺序

``` sql
SHOW CHARACTER SET;    -- 显示所有可用的字符集以及每个字符集的描述和默认校对

SHOW COLLATION;    -- 查看所支持校对的完整列表
```

为了确定所用的字符集和校对，可以使用以下语句

```sql
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';
```

实际上，字符集很少是服务器范围（甚至数据库范围）的设置。不同的表，甚至不同的列都可能需要不同的字符集，而且两者都可以在创建表时指定。

```sql
CREATE TABLE mytable
(
    columnn1 INT,
    columnn2 VARCHAR(10),
    columnn3 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_general_ci
) DEFAULT CHARACTER SET hebrew
  COLLATE hebrew_general_ci;
```

如果你需要用与创建表时不同的校对顺序排序特定的`SELECT`语句，可以在SELECT语句自身中进行：

```sql
SELECT * FROM CUSTOMERS
ORDER BY lastname, firstname COLLATE latin1_general_cs;
```

除了这里看到的在`ORDER BY`子句中使用以外， `COLLATE`还可以用于`GROUP BY`、 `HAVING`、聚集函数、别名等。

## 安全管理

### 访问控制

MySQL服务器的安全基础是： *用户应该对他们需要的数据具有适当的访问权，既不能多也不能少。*

### 管理用户

MySQL用户账号和信息存储在名为mysql的MySQL数据库中。

```sql
CREATE USER ben IDENTIFIED BY 'p@$$w0rd';    -- 创建用户账号

RENAME USER ben TO bforta;    -- 重命名
```

在创建用户账号时不一定需要口令。

```sql
DROP USER bforta;    -- 删除用户账号
```

#### 设置访问权限

```sql
SHOW GRANTS FOR bforta;    -- 查看用户账号权限
```

为设置权限，使用`GRANT`语句。 `GRANT`要求你至少给出以下信息：

- 要授予的权限；
- 被授予访问权限的数据库或表；
- 用户名。

```sql
GRANT SELECT ON crashcourse.* TO bforta;    -- 允许用户在crashcourse.*（crashcourse数据库的所有表）上使用SELEC
```

`GRANT`的反操作为`REVOKE`，用它来撤销特定的权限。

`GRANT`和`REVOKE`可在几个层次上控制访问权限：

- 整个服务器，使用`GRANT ALL`和`REVOKE ALL`；
- 整个数据库，使用`ON database.*`；
- 特定的表，使用`ON database.table`；
- 特定的列；
- 特定的存储过程。

#### 更改口令

为了更改用户口令，可使用`SET PASSWORD`语句。

```sql
SET PASSWORD FOR bforta = Password('n3w p@$$w0rd');
```

在不指定用户名时， SET PASSWORD更新当前登录用户的口令。

## 后记

关于第29章数据库维护和第30章改善性能，准备不写笔记了，有兴趣的还是去看书吧。这两章的内容，大多以文字为主。