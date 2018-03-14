---
layout: post
title: Java位运算方法总结[转]
date: 2018-3-14 09:55:56
categories: 算法
tags: 算法
author: renshuo
mathjax: true
---

* content
{:toc}

关于算法题目中有关**位运算**的方法总结，转自LeetCode评论 [A summary: how to use bit manipulation to solve problems easily and efficiently](https://leetcode.com/problems/sum-of-two-integers/discuss/84278/a-summary-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently)

<!--more-->

## Wiki

> 以下来自 Google 翻译

位操作是通过算术操作位或其他短于数据的数据段的操作。需要位操作的计算机编程任务包括低级设备控制，错误检测和纠正算法，数据压缩，加密算法和优化。 对于大多数其他任务，现代编程语言允许程序员直接使用抽象而不是代表抽象的位。 进行位操作的源代码使用按位操作：AND，OR，XOR，NOT和位移。

在一些情况下，位操作可以避免或减少循环遍历数据结构的需要，并且可以提供多倍的加速，因为位操作是并行处理的，但代码可能变得更难以编写和维护。

## 细节

### 基础

位运算的核心是位操作符 `&` (and), `|`(or), `~` (not) 和 `^` (XOR) 和移位运算符， `a << b` 和 `a >> b`.

java中的移位运算符有三种， `>>>` 使用 `0` 填充高位； `>>` 使用符号位填充高位； `<<` 左移操作。

| 操作               | 英文             | 举例                                        |
| ------------------ | ---------------- | ------------------------------------------- |
| 取并集             | Set Union        | A \| B                                      |
| 取交集             | Set intersection | A & B                                       |
| 取A中不属于B的部分 | Set subtraction  | A & ~B                                      |
| 按位取反           | Set negation     | ALL_BITS ^ A or ~A                          |
| 将某位设置为1      | Set bit          | A \|= 1 << bit                              |
| 将某位设置为0      | Clear bit        | A &= ~(1 << bit)                            |
| 测试某位的值       | Test bit         | (A & 1 << bit) != 0                         |
| 提取最后一个1      | Extract last bit | A & -A or A & ~(A - 1) or x ^ (x & (x - 1)) |
| 删除最后一个1      | Remove last bit  | A & (A - 1)                                 |
| 获取全部为1的字节  | Get all 1-bits   | ~0                                          |

### 实例

**计算给定数字二进制表示中1的个数**

``` java
private static int countOne(int n) {
    int count = 0;
    while (n != 0) {
        n = n & (n - 1);
        count++;
    }
    return count;
}
```

**判断是否是4的幂数（可以使用映射，迭代或者递归方法）**

``` java
private static boolean isPowerOfFour(int n){
    return (n & (n - 1)) == 0 && (n & 0x55555555) != 0;
}
```

首先，如果一个数字是4的幂数，那么它的二进制肯定只有一个1，也就是 `(n&(n-1))==0`。

然后在看`0x55555555`，它的二进制表示是： `0101 0101 0101 0101 0101 0101 0101 0101`也就是只有都是4的倍数的值拼凑起来的数字和它进行 `&` 运算才能得到不是0的结果。

所以一个数字，是4的倍数，并且它的二进制表示中只有一个 1，那么它是4的幂数。同理可以求8的幂数等。

#### `^` 技巧

使用 `^` 来删除二进制表示中数字完全相同的位从而保存不同的，或者保存不同的位删除相同的。

**计算两个整数的加法运算**

使用 `^` 和 `&` 将两个整数相加

``` java
private static int getSum(int a, int b) {
    if (a == 0) return b;
    if (b == 0) return a;
    while (b != 0) {
        int carry = a & b; // 得到有进位的位置
        a = a ^ b; // 直接相加，但是没有进位
        b = carry << 1; // 得到进位
    }
    return a;
}
```

**找到缺少的数字**

给出一个数组，这个数组都是由不同的数字组成的，这些数字的取值从 0 到 n，请找出那个丢失的数字。

``` java
private static int missingNumber(int[] nums) {
    int ret = 0;
    for (int i = 0; i < nums.length; i++) {
        ret ^= i;
        ret ^= nums[i];
    }
    return ret ^ nums.length; // 数组中少了一个数，所以这里要加上
}
```

#### `|` 技巧

尽量包容下所有的值为 1 的位。

**给定一个数字 N，找出最大的 2 的幂数小于或者等于 N 的值。**

``` java
private static long largestPower(long N) {
    //changing all right side bits to 1.
    N = N | (N >> 1);
    N = N | (N >> 2);
    N = N | (N >> 4);
    N = N | (N >> 8);
    N = N | (N >> 16); // java中要是int型，只需要到这步
    N = N | (N >> 32);
    return (N + 1) >> 1;
}
```

将当前数字的二进制表达式中的最高位的 1 右边的所有的数字全部置为 1。然后呢，我们再处理最高位的问题，怎么处理这个最高位呢？我们可以使之加 1，实现进位加 1，然后我们再右移 1 位，保证当前数字小于等于参数值。

**反转给出的32位无符号整数**

java 中没有无符号整数，下面给出c代码：

```c
uint32_t reverseBits(uint32_t n) {
    unsigned int mask = 1<<31, res = 0;
    for(int i = 0; i < 32; ++i) {
        if(n & 1) res |= mask; //从右往左
        mask >>= 1; // 从左往右
        n >>= 1;
    }
    return res;
}
```

``` c
uint32_t reverseBits(uint32_t n) {
	uint32_t mask = 1, ret = 0;
	for(int i = 0; i < 32; ++i){
		ret <<= 1;  // 低位向高位移动
		if(mask & n) ret |= 1;  // 判断低位上的数字
		mask <<= 1;  // 从低位到高位进行扫描
	}
	return ret;
}
```

#### `&` 技巧

用来选择一些需要的位

**反转一些指定的位**

``` java
x = ((x & 0xaaaaaaaa) >> 1) | ((x & 0x55555555) << 1);
x = ((x & 0xcccccccc) >> 2) | ((x & 0x33333333) << 2);
x = ((x & 0xf0f0f0f0) >> 4) | ((x & 0x0f0f0f0f) << 4);
x = ((x & 0xff00ff00) >> 8) | ((x & 0x00ff00ff) << 8);
x = ((x & 0xffff0000) >> 16) | ((x & 0x0000ffff) << 16);
```

| 十六进制 | 二进制    | 功能                 |
| -------- | --------- | -------------------- |
| 0xaa     | 1010 1010 | 选择指定 1 位        |
| 0x55     | 0101 0101 | 与0xaa的位置刚好相反 |
| 0xcc     | 1100 1100 | 选择指定 2 位        |
| 0x33     | 0011 0011 | 与0xcc的位置刚好相反 |
| 0xf0     | 1111 0000 | 选择指定 4 位        |
| 0x0f     | 0000 1111 | 与0x0f的位置刚好相反 |

**给定一个范围[m, n]，其中 0 <= m <= n <= 2147483647（0x7FFFFFFF），返回指定范围内所有数字的相与结果。**

比如给出 `[5,7]`,结果为4

``` java
private static int rangeBitwiseAnd(int m, int n) {
    int a = 0;
    while (m != n) {  // 找到高位相同的地方
        m >>= 1;
        n >>= 1;
        a++;
    }
    return m << a; // &运算，低位不同的，都变成0了
}
```

## 题目

后续会把LeetCode里面相关的题目链接总结在这里，持续更新中

[Easy - 136. Single Number](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-11-20-136-SingleNumber.md)

[Easy - 190. Reverse Bits](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-11-23-190-ReverseBits.md)

[Easy - 191. Number of 1 Bits](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-11-25-191-Numberof1Bits.md)

[Easy - 231. Power of Two](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-11-29-231-PowerofTwo.md)

[Easy - 268. Missing Number](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-12-03-268-MissintNumber.md)

[Easy - 342. Power of Four](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-12-04-342-PowerofFour.md)

[Easy - 371. Sum of Two Integers](https://github.com/renshuo555/LeetCode/blob/master/Easy/2017-12-06-371-SumofTwoIntegers.md)

## 参考资料

[A summary: how to use bit manipulation to solve problems easily and efficiently](https://leetcode.com/problems/sum-of-two-integers/discuss/84278/a-summary-how-to-use-bit-manipulation-to-solve-problems-easily-and-efficiently)

[（翻译整理）如何简单、高效地使用位操作解决问题](http://blog.csdn.net/u012814856/article/details/70902466)