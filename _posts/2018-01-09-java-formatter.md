---
layout: post
title: Java格式化输出
date: 2018-01-09 22:47:47
categories: Java
tags: Java
author: renshuo
mathjax: true
---

* content
{:toc}

对 `Java` 格式化输出的总结，主要是对 `printf` 和 `format` 方法的归纳。

设计到的主要类为 [`java.util.Formatter`](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html)，本文主要是对 `Java8` 中相关 `API` 的简单翻译。

<!--more-->

`Java` 中字符串的样式格式化主要是基于 `java.util.Formatter` ，这个类提供了布局和对齐，数字的常见格式，字符串，各种日期时间和基于特定语言环境的输出。

格式化程序的线程安全需要用户自己负责。

## 概要 (Summary)

### 格式化字符串语法 (Format String Syntax)

每种格式化输出的方法，都需要一个**格式化字符串**和一个**参数列表**，格式化字符串中包含固定文本和一个或多个**格式说明符**。

格式说明符的一般语法为： 
> `%[argument_index$][flags][width][.precision]conversion`

* `argument_index`：可选，是一个十进制整数，表示参数列表中参数的位置。第一个参数由 `1$`  引用，第二个由 `2$`引用等
* `flags`：可选，是一组修改输出格式的字符，有效标志集取决于转换类型
* `width`：可选，是一个正的十进制整数，表示要写入输出的最小字符数
* `.precision`：可选，是一个非负十进制整数，通常用来限制字符数。特定行为取决于转换类型
* `conversion`：必须，表明应该如何格式化参数的字符，给定参数的有效转换集取决于参数的数据类型


格式说明符对日期时间的语法为：

> `%[argument_index$][flags][width]conversion`

* `conversion`：由两个字符组成，第一个字符是 `t` 或者 `T` ，第二个字符表示使用的格式

不符合参数的格式说明符语法为：

> `%[flags][width]conversion`

* `conversion`：一个表示在输出中插入内容的字符

### 转换符 (Conversions)

`conversion` 主要分为以下几个类别：

* **General** ：应用于任何参数类型
* **Character** ：应用于表示 `Unicode` 字符的基本类型， `char, Character, byte, Byte, short, and Short`。当 `Character.isValidCodePoint(int)` 返回为 `true` 时，也可应用于 `int and Integer`
* **Numeric**：整型和浮点型
* **Date/Time**：应用于能够对日期或者时间进行编码的类型 `long, Long, Calendar, Date and TemporalAccessor`
* **Percent**：百分比，产生字面值 `% (\u0025)`
* **Line Separator**：行分隔符，产生特定平台的行分隔符

有的转换符有大写的形式，表示输出的也是转换成大写输出。

| 转换符          | 说明                           |
| ------------ | ---------------------------- |
| `'b'`, `'B'` | 布尔型输出，参数为 `null` 结果为 `false` |
| `'h'`, `'H'` | 哈希值，参数为 `null` 结果为 `null`    |
| `'s'`, `'S'` | 字符串                          |
| `'c'`, `'C'` | 字符                           |
| `'d'`        | 十进制整数                        |
| `'o'`        | 八进制整数                        |
| `'x'`, `'X'` | 十六进制整数                       |
| `'e'`, `'E'` | 指数浮点数                        |
| `'f'`        | 定点浮点数  `3.55000`             |
| `'g'`, `'G'` | 通用浮点数                        |
| `'a'`, `'A'` | 十六进制浮点数                      |
| `'t'`, `'T'` | 日期和时间                        |
| `'%'`        | 百分号 `%`                      |
| `'n'`        | 与平台有关的行分隔符                   |

### 日期时间转换符 (Date/Time Conversions)

#### 格式化时间转换符

| 转换符   | 类型                                   | 说明                                       |
| ----- | ------------------------------------ | ---------------------------------------- |
| `'H'` | 两位数 小时数                              | 00 ~ 23                                  |
| `'I'` | 两位数 小时数                              | 01 ~ 12                                  |
| `'k'` | 小时数                                  | 0 ~ 23                                   |
| `'l'` | 小时数                                  | 1 ~ 12                                   |
| `'M'` | 两位数 分钟                               | 00 ~ 59                                  |
| `'S'` | 两位数 秒                                | 00 ~ 60                                  |
| `'L'` | 毫秒                                   | 000 ~ 999                                |
| `'N'` | 纳秒                                   | 000000000 - 999999999                    |
| `'p'` | 上下午标志                                | "`am`" or "`pm`"                         |
| `'z'` | 从 `GMT` 起， `RFC822` 数字位移             | -0800                                    |
| `'Z'` | 时区                                   | `PST`                                    |
| `'s'` | 从 1 January 1970 `00:00:00` UTC起的秒数  | `Long.MIN_VALUE/1000` to `Long.MAX_VALUE/1000` |
| `'Q'` | 从 1 January 1970 `00:00:00` UTC起的毫秒数 | `Long.MIN_VALUE` to `Long.MAX_VALUE`     |

#### 格式化日期转换符

| 转换符         | 类型        | 说明                        |
| ----------- | --------- | ------------------------- |
| `'B'`       | 完整的月份名称   | `"January"`, `"February"` |
| `'b'`，`'h'` | 月份名称简写    | `"Jan"`, `"Feb"`          |
| `'A'`       | 星期的全称     | `"Sunday"`, `"Monday"`    |
| `'a'`       | 星期的简写     | `"Sun"`, `"Mon"`          |
| `'C'`       | 年的前两位数字   | `00 - 99`                 |
| `'Y'`       | 年份，四位数字   | `2018`                    |
| `'y'`       | 年份的后两位数字  | `00 - 99`                 |
| `'j'`       | 年中的日子     | `001 - 366`               |
| `'m'`       | 月份        | 01 - 13                   |
| `'d'`       | 月中的日子，两位数 | 01 - 31                   |
| `'e'`       | 月中的日子     | 1 - 31                    |

#### 常用日期格式

| 转换符   | 说明                                       |
| ----- | ---------------------------------------- |
| `'R'` | Time formatted for the 24-hour clock as `"%tH:%tM"` |
| `'T'` | Time formatted for the 24-hour clock as `"%tH:%tM:%tS"`. |
| `'r'` | Time formatted for the 12-hour clock as `"%tI:%tM:%tS %Tp"`. The location of the morning or afternoon marker (`'%Tp'`) may be locale-dependent. |
| `'D'` | Date formatted as `"%tm/%td/%ty"`.       |
| `'F'` | [ISO 8601](http://www.w3.org/TR/NOTE-datetime) complete date formatted as `"%tY-%tm-%td"`. |
| `'c'` | Date and time formatted as `"%ta %tb %td %tT %tZ %tY"`, e.g. `"Sun Jul 20 16:17:00 EDT 1969"`. |

### 标志位 (Flags)

| Flag | 应用范围                        | 说明                           |
| ---- | --------------------------- | ---------------------------- |
| '-'  | 全部                          | 左对齐                          |
| '#'  | `Integral`和`Floating Point` | 整型显示进制前缀(`0x或0`)，浮点型显示小数点    |
| '+'  | `Integral`和`Floating Point` | 显示正负符号                       |
| '  ' | `Integral`和`Floating Point` | 空格，正数前面补充空格                  |
| '0'  | `Integral`和`Floating Point` | 数字前面补`0`                     |
| ','  | `Integral`和`Floating Point` | 添加分组分隔符，如 `3,333.33`         |
| '('  | `Integral`和`Floating Point` | 将负数用小括号括起来，如 `-33` 变成 `(33)` |

### 宽度 (Width)

将输出的最少的字符数

### 精度 (Precision)

`general argument types`：对常规的参数，输出的最大字符数

`conversions 'a', 'A', 'e', 'E', and 'f'`：浮点型数据，小数点后的位数

`conversion 'g' or 'G'`：浮点型数据，四舍五入后的所有位数

### 参数索引 (Argument Index)

是一个从 `1` 开始的十进制整数，第一个是 `1$` ，依次类推

`<` 这个标志可以重用以前的参数

下面两条语句生成的字符串相同：

``` java
Calendar c = ...;
String s1 = String.format("Duke's Birthday: %1$tm %1$te,%1$tY", c);
String s2 = String.format("Duke's Birthday: %1$tm %<te,%<tY", c);
```
