---
layout: post
title: NumPy 基础学习
date: 2018-01-09 10:33:08
categories: Python
tags: Python NumPy
author: renshuo
mathjax: true
---

* content
{:toc}
`NumPy` 基础学习总结 [转]

是直接使用 `Jupyter Notebook` 导出的 `Markdown` 文件

<!--more-->

首先介绍`numpy.info`函数，它用于获取NumPy内函数的帮助：

```python
import numpy as np
```

`numpy.info` 可以获取 `NumPy` 内函数的帮助信息


```python
# np.info(np.array)
```

## NumPy的ndarray

### 创建ndarray

- array 将输入数据（列表、元组、数组或其它序列类型）转换为ndarray。要么推断出dtype，要么显示指定dtype。默认直接复制输入数据。

- asarray 将输入转换为ndarray，如果输入本身就是一个ndarray就不进行复制。

- arange 类似于Python内置的range，但返回一个ndarray而不是列表。

- ones, ones_like 根据指定形状和dtype创建一个全1的数组。ones_like以另一个数组为参数，并根据其形状和dtype创建一个全1数组。

- zeros, zeros_like 类似于ones和ones_like，只不过产生的是全0数组而已。

- empty, empty_like 创建数组，只分配内存空间但不填充任何值

- eye, identity 创建对角矩阵、单位矩阵。单位矩阵一定是方阵，但对角矩阵不一定


```python
z = np.array([2, 4, 1, 6, 10])
print(z)
z = np.array([2, 4, 1, 6, 10], dtype=float)
print(z)

z = np.eye(3)
print(z)
```

    [ 2  4  1  6 10]
    [  2.   4.   1.   6.  10.]
    [[ 1.  0.  0.]
     [ 0.  1.  0.]
     [ 0.  0.  1.]]


### 数据类型

- int8, uint8 有/无符号的8位整型

- int16, uint16 有/无符号的16位整型

- int32, uint32 有/无符号的32位整型

- int64, uint64 有/无符号的64位整型

- float16 半精度浮点数

- float32 标准的单精度浮点数，与C的float兼容

- float64 标准的双精度浮点数，与C的double和Python的float兼容

- float128 扩展精度浮点数

- complex, comples64, complex128 分别用128位、64位、128位表示的复数

- bool 存储True和Flase值的布尔类型

- object Python对象类型

- string_ 字符串

- unicode_ unicode类型

除了创建 `ndarray` 时使用 `dtype` 参数指定数据类型外，可以使用 `astype`函数显示的转换类型。

`itemsize` 函数可以查看数据类型的 `size` 。


```python
z = np.array([2, 4, 1, 6, 10], dtype=np.complex)
print(z.itemsize)
z = z.astype(np.complex64)
print(z.itemsize)
```

    16
    8


### ndarray与标量的运算
- 数组与标量（包括长度为1的数组）的算术运算：将标量值运用到数组的每个元素
- 数组与数组：大小相等才能运算，也是运用到数组的每个元素


```python
x = np.array([1, 2, 3])
y = np.array([3, 2, 5])
print(1 / x)
print(x * y)
```

    [ 1.          0.5         0.33333333]
    [ 3  4 15]


### 索引与切片
索引和切片跟Python默认的索引和切片出入不大，其强大之处需要慢慢体会，下面两个例子自己感受下：


```python
# 创建一个2维数组，使所有边界上的元素都为1，内部的元素都为0
z = np.ones((6, 6))
z[1:-1, 1:-1] = 0
print(z)

# 创建一个跳棋盘布局的数组
z = np.zeros((6, 6))
z[::2, 1::2] = 1
z[1::2, ::2] = 1
print(z)
```

    [[ 1.  1.  1.  1.  1.  1.]
     [ 1.  0.  0.  0.  0.  1.]
     [ 1.  0.  0.  0.  0.  1.]
     [ 1.  0.  0.  0.  0.  1.]
     [ 1.  0.  0.  0.  0.  1.]
     [ 1.  1.  1.  1.  1.  1.]]
    [[ 0.  1.  0.  1.  0.  1.]
     [ 1.  0.  1.  0.  1.  0.]
     [ 0.  1.  0.  1.  0.  1.]
     [ 1.  0.  1.  0.  1.  0.]
     [ 0.  1.  0.  1.  0.  1.]
     [ 1.  0.  1.  0.  1.  0.]]


### 布尔型索引

- 布尔型数组的长度必须跟被索引的轴长度一致。

- 可以将布尔型数组跟切片、整数（或整数序列）混合使用


```python
name_arr = np.array(['Bob', 'Joe', 'Will', 'Bob', 'Will', 'Joe', 'Joe'])
rnd_arr = np.random.randn(7, 4)

print(rnd_arr)
# 返回布尔数组，元素等于'Bob'为True，否则False
print(name_arr == 'Bob')
# 利用布尔数组选择行
print(rnd_arr[name_arr == 'Bob'])
# 增加限制打印列的范围
print(rnd_arr[name_arr == 'Bob', :2])
```

    [[ 0.55847192  0.45400539  0.66459502 -0.0745954 ]
     [ 1.19314101  0.80097929 -1.14151769 -0.02601475]
     [-0.77302773 -0.49514954  1.29349717  0.55751855]
     [ 0.78469097 -0.18242615 -0.3933802   0.84887575]
     [ 1.0757846  -0.51104367  0.53406883 -2.39047944]
     [-0.14250946  1.39838787 -0.40234123 -0.82047402]
     [ 0.95223314  0.8081477  -0.05766121  0.43086325]]
    [ True False False  True False False False]
    [[ 0.55847192  0.45400539  0.66459502 -0.0745954 ]
     [ 0.78469097 -0.18242615 -0.3933802   0.84887575]]
    [[ 0.55847192  0.45400539]
     [ 0.78469097 -0.18242615]]


### 花式索引
- 花式索引（Fancy indexing）是一个NumPy术语，它指的是利用整数数组进行索引。


```python
arr = np.tile(np.arange(8).reshape(8, -1), (1, 4)).astype((float))
print(arr)

print('打印arr[4]、arr[3]、arr[0]和arr[6]')
print(arr[[4, 3, 0, 6]])  # 打印arr[4]、arr[3]、arr[0]和arr[6]。
print('打印arr[1, 0]、arr[5, 3]，arr[7, 1]和arr[2, 2]')
print(arr[[1, 5, 7, 2], [0, 3, 1, 2]])  # 打印arr[1, 0]、arr[5, 3]，arr[7, 1]和arr[2, 2]
print('1572行的0312列')
print(arr[[1, 5, 7, 2]][:, [0, 3, 1, 2]])  # 1572行的0312列
print(arr[np.ix_([1, 5, 7, 2], [0, 3, 1, 2])])  # 可读性更好的写法
```

    [[ 0.  0.  0.  0.]
     [ 1.  1.  1.  1.]
     [ 2.  2.  2.  2.]
     [ 3.  3.  3.  3.]
     [ 4.  4.  4.  4.]
     [ 5.  5.  5.  5.]
     [ 6.  6.  6.  6.]
     [ 7.  7.  7.  7.]]
    打印arr[4]、arr[3]、arr[0]和arr[6]
    [[ 4.  4.  4.  4.]
     [ 3.  3.  3.  3.]
     [ 0.  0.  0.  0.]
     [ 6.  6.  6.  6.]]
    打印arr[1, 0]、arr[5, 3]，arr[7, 1]和arr[2, 2]
    [ 1.  5.  7.  2.]
    1572行的0312列
    [[ 1.  1.  1.  1.]
     [ 5.  5.  5.  5.]
     [ 7.  7.  7.  7.]
     [ 2.  2.  2.  2.]]
    [[ 1.  1.  1.  1.]
     [ 5.  5.  5.  5.]
     [ 7.  7.  7.  7.]
     [ 2.  2.  2.  2.]]


### 数组转置和轴对换
- 转置：arr.T比较简单，就是常规的转置，arr.transpose可以增加坐标作为参数
- 点积：numpy.dot
- 轴交换：swapaxes


```python
z = np.arange(9).reshape((3, 3))
print('转置')
print(z.T)
print(z.transpose())
```

    转置
    [[0 3 6]
     [1 4 7]
     [2 5 8]]
    [[0 3 6]
     [1 4 7]
     [2 5 8]]



```python
'''
详细解释：
arr数组的内容为
- a[0][0] = [0, 1, 2, 3]
- a[0][1] = [4, 5, 6, 7]
- a[1][0] = [8, 9, 10, 11]
- a[1][1] = [12, 13, 14, 15]
transpose的参数为坐标，正常顺序为(0, 1, 2, ... , n - 1)，
现在传入的为(1, 0, 2)代表a[x][y][z] = a[y][x][z]，第0个和第1个坐标互换。
- a'[0][0] = a[0][0] = [0, 1, 2, 3]
- a'[0][1] = a[1][0] = [8, 9, 10, 11]
- a'[1][0] = a[0][1] = [4, 5, 6, 7]
- a'[1][1] = a[1][1] = [12, 13, 14, 15]
'''
z = np.arange(16).reshape((2, 2, 4))
print(z)
print('transpose')
print(z.transpose((1, 0, 2)))

print('swapaxes')
print(z.swapaxes(1, 2))
```

    [[[ 0  1  2  3]
      [ 4  5  6  7]]
    
     [[ 8  9 10 11]
      [12 13 14 15]]]
    transpose
    [[[ 0  1  2  3]
      [ 8  9 10 11]]
    
     [[ 4  5  6  7]
      [12 13 14 15]]]
    swapaxes
    [[[ 0  4]
      [ 1  5]
      [ 2  6]
      [ 3  7]]
    
     [[ 8 12]
      [ 9 13]
      [10 14]
      [11 15]]]


### 快速的元素级函数

#### 一元函数：

- abs, fabs 计算整数、浮点数或复数的绝对值。对于非复数值，可以使用更快的fabs。
- sqrt 计算各元素的平方根。相当于arr^0.5
- sqare 计算各元素的平方。相当于arr^2
- exp 计算各元素的e^x
- log, log10, log2, log1p 分别为自然对数、底数为10的log、底数为2的log和log(1+x)
- sign 计算各元素的正负号：1（正数）、0（零）、-1（负数）
- ceil 计算各元素的ceiling值，即大于等于该值的最小整数
- floor 计算各元素的floor值，即小于等于该值的最小整数
- rint 将各元素值四舍五入到最接近的整数，保留dtype
- modf 将数组的小数部分与整数部分以两个独立数组的形式返还
- isnan 返回一个表示“哪些值时NaN”的布尔型数组
- isfinite, isinf 分别返回一个表示“哪些元素是有限的（非inf，非NaN）”或“哪些元素是无穷的”的布尔型数组
- cos, cosh, sin, sinh, tan, tanh 普通型或双曲型三角函数
- arccos, arccosh, arcsin, arcsinh, arctan, arctanh 反三角函数
- logical_not 计算各元素not x的真值，相当于~arr

#### 二元函数：

- add 将数组中对应元素相加
- subtract 从第一个数组中减去第二个数组中的元素
- multiply 数组元素相乘
- divide, floor_divide 除法或向下取整除法
- power 对第一个数组中的元素A和第二个数组中对应位置的元素B，计算A^B
- maximum, fmax 元素级的最大值计算。fmax将忽略NaN
- minimum, fmin 元素级的最小值计算。fmin将忽略NaN
- mod 元素级的求模计算
- copysign 将第二个数组中的符号复制给第一个数组中的值
- greater, greater_equal, less, less_equal, equal, not_equal 执行元素级的比较，最终产生布尔型数组
- logical_and, logical_or, logical_xor 执行元素级的真值逻辑运算，最终产生布尔型数组

## 数据处理

- NumPy数组使你可以将许多种数据处理任务表述为简洁的数组表达式（否则需要编写循环）。用数组表达式代替循环的做法，通常被称为矢量化。
- 矢量化数组运算要比等价的纯Python方式快上一两个数量级

### 将条件逻辑表述为数组运算

- numpy.where 根据条件来分别从两个数组中选择值。对比下面例子中的“列表推导”方法（速度不够快、无法应用于高维数组）


```python
'''
关于zip函数的一点解释，zip可以接受任意多参数，然后重新组合成1个tuple列表。
zip([1, 2, 3], [4, 5, 6], [7, 8, 9])
返回结果：[(1, 4, 7), (2, 5, 8), (3, 6, 9)]
'''
print('通过真值表选择元素')
x_arr = np.array([1.1, 1.2, 1.3, 1.4, 1.5])
y_arr = np.array([2.1, 2.2, 2.3, 2.4, 2.5])
cond = np.array([True, False, True, True, False])
result = [(x if c else y) for x, y, c in zip(x_arr, y_arr, cond)]  # 通过列表推导实现
print(result)
print(np.where(cond, x_arr, y_arr))  # 使用NumPy的where函数
print('where嵌套')
cond_1 = np.array([True, False, True, True, False])
cond_2 = np.array([False, True, False, True, False])
# 传统代码如下
result = []
for i in range(len(cond)):
    if cond_1[i] and cond_2[i]:
        result.append(0)
    elif cond_1[i]:
        result.append(1)
    elif cond_2[i]:
        result.append(2)
    else:
        result.append(3)
print(result)
# np版本代码
result = np.where(cond_1 & cond_2, 0, np.where(cond_1, 1, np.where(cond_2, 2, 3)))
print(result)
```

    通过真值表选择元素
    [1.1000000000000001, 2.2000000000000002, 1.3, 1.3999999999999999, 2.5]
    [ 1.1  2.2  1.3  1.4  2.5]
    where嵌套
    [1, 2, 1, 0, 3]
    [1 2 1 0 3]


### 数学和统计方法

- sum 对数组中全部或某轴向的元素求和。零长度的数组的sum为0。
- mean 算数平均数。零长度的数组的mean为NaN。
- std, var 分别为标准差和方差，自由度可调（默认为n）。
- min, max 最大值和最小值
- argmin, argmax 最大值、最小值的索引
- cumsum 所有元素的累计和
- cumprod 所有元素的累计积


```python
'''
cumsum:
- 按列操作：a[i][j] += a[i - 1][j]
- 按行操作：a[i][j] *= a[i][j - 1]
cumprod:
- 按列操作：a[i][j] += a[i - 1][j]
- 按行操作：a[i][j] *= a[i][j - 1]
'''
arr = np.array([[0, 1, 2], [3, 4, 5], [6, 7, 8]])
print(arr.cumsum(0))
print(arr.cumprod(1))
```

    [[ 0  1  2]
     [ 3  5  7]
     [ 9 12 15]]
    [[  0   0   0]
     [  3  12  60]
     [  6  42 336]]


### 用于布尔型数组的方法

- sum 对True值求和
- any， all 测试布尔型数组，对于非布尔型数组，所有非0元素将会被当做True


```python
print('对正数求和')
arr = np.random.randn(100)
print((arr > 0).sum())
print()
print('对数组逻辑操作')
bools = np.array([False, False, True, False])
print(bools.any())  # 有一个为True则返回True
print(bools.all())  # 有一个为False则返回False
```

    对正数求和
    39
    
    对数组逻辑操作
    True
    False


### 排序

- sort


```python
print('一维数组排序')
arr = np.random.randn(8)
arr.sort()
print(arr)
print()
print('二维数组排序')
arr = np.random.randn(5, 3)
print(arr)
arr.sort(1)  # 对每一行元素做排序
print(arr)
print('找位置在5%的数字')
large_arr = np.random.randn(1000)
large_arr.sort()
print(large_arr[int(0.05 * len(large_arr))])
```

    一维数组排序
    [-0.40045439 -0.26070464 -0.20473029  0.00889988  0.58376642  0.93275869
      0.98781935  1.17966278]
    
    二维数组排序
    [[-1.0094797  -1.35479776  0.64314258]
     [ 0.5179261   0.65697195  0.53006781]
     [-0.06161718 -0.81296897  0.83720279]
     [-1.4796964  -0.3654008   0.42282089]
     [ 0.14035657  0.73662839 -0.56868074]]
    [[-1.35479776 -1.0094797   0.64314258]
     [ 0.5179261   0.53006781  0.65697195]
     [-0.81296897 -0.06161718  0.83720279]
     [-1.4796964  -0.3654008   0.42282089]
     [-0.56868074  0.14035657  0.73662839]]
    找位置在5%的数字
    -1.70379756296


### 去重以及其它集合运算

- unique(x) 计算x中的唯一元素，并返回有序结果
- intersect1d(x, y) 计算x和y中的公共元素，并返回有序结果
- union1d(x, y) 计算x和y的并集，并返回有序结果
- in1d(x, y) 得到一个表述【x的元素是否包含于y】的布尔型数组
- setdiff1d(x, y) 集合的差，即元素在x中且不在y中
- setxor1d(x, y) 集合的异或，即存在于一个数组中但不同时存在于两个数组中的元素

## 数组文件的输入输出

- save
- load


```python
print('数组文件读写')
arr = np.arange(10)
np.save('some_array', arr)
print(np.load('some_array.npy'))
print()
print('多个数组压缩存储')
np.savez('array_archive.npz', a=arr, b=arr)
arch = np.load('array_archive.npz')
print(arch['b'])
```

    数组文件读写
    [0 1 2 3 4 5 6 7 8 9]
    
    多个数组压缩存储
    [0 1 2 3 4 5 6 7 8 9]


## 线性代数

常用的 `numpy.linalg` 函数

- diag 以一维数组的形式返回方针的对角线（或非对角线元素），或将一维数组转换为方针（非对角线元素为0）
- dot 矩阵乘法
- trace 迹：对角线元素的和
- det 计算矩阵行列式
- eig 计算方阵的特征值和特征向量
- inv 计算方针的逆
- pinv 计算矩阵的Moore-Penrose伪逆
- qr 计算QR分解
- svd 计算奇异值分解
- solve 解线性方程Ax=b，其中A为一个方阵
- lstsq 计算Ax=b的最小二乘解


```python
A = np.array([[5, -1], [4, -1]])
b = np.array([[-1], [3]])
print(np.linalg.solve(A, b))
```

    [[ -4.]
     [-19.]]


## 随机数生成

部分 `numpy.random` 函数：

- seed 确定随机数生成器的种子
- permutation 返回一个序列的随机排列或返回一个随机排列
- shuffle 对一个序列就地随机乱序
- rand 产生均匀分布的样本值
- randint 从给定的上下限范围内随机选取整数
- randn 产生正态分布（平均值为0，标准差为1）
- binomial 产生二项分布的样本值
- normal 产生正态（高斯）分布的样本值
- beta 产生Beta分布的样本值
- chisquare 产生卡方分布的样本值
- gamma 产生Gamma分布的样本值
- uniform 产生在[0,1]中均匀分布的样本值

## 高级应用

### 数组重塑

- reshape -1可以自动推导维度大小

### 数组的合并和拆分

数组连接函数：

- concatenate 最一般化的连接，沿一条轴连接一组数组
- vstack, row_stack 以面向行的方式对数组进行堆叠（沿轴0）
- hstack 以面向列的方式对数组进行堆叠（沿轴1）
- column_stack 类似于hstack，但是会先将一维数组转换为二维列向量
- split 沿指定轴在指定的位置拆分数组
- hsplit, vsplit, dsplit split的便捷化函数，分别沿着轴0、轴1和轴2进行拆分

`np.r_，np.c_`对象也可以用于堆叠


```python
print('连接两个二维数组')
arr1 = np.array([[1, 2, 3], [4, 5, 6]])
arr2 = np.array([[7, 8, 9], [10, 11, 12]])
print('按行连接')
print(np.concatenate([arr1, arr2], axis=0))
print('按列连接')
print(np.concatenate([arr1, arr2], axis=1))
# stack 堆叠，连接的另一种表述
print('垂直stack')
print(np.vstack((arr1, arr2)))
print('水平stack')
print(np.hstack((arr1, arr2)))
print('np.r_对象')
print(np.r_[arr1, arr2])
print('np.c_对象')
print(np.c_[arr1, arr2])
```

    连接两个二维数组
    按行连接
    [[ 1  2  3]
     [ 4  5  6]
     [ 7  8  9]
     [10 11 12]]
    按列连接
    [[ 1  2  3  7  8  9]
     [ 4  5  6 10 11 12]]
    垂直stack
    [[ 1  2  3]
     [ 4  5  6]
     [ 7  8  9]
     [10 11 12]]
    水平stack
    [[ 1  2  3  7  8  9]
     [ 4  5  6 10 11 12]]
    np.r_对象
    [[ 1  2  3]
     [ 4  5  6]
     [ 7  8  9]
     [10 11 12]]
    np.c_对象
    [[ 1  2  3  7  8  9]
     [ 4  5  6 10 11 12]]



```python
arr = np.random.randn(5, 5)
print(arr)
print('水平拆分')
first, second, third = np.split(arr, [1, 3], axis=0)
print('first')
print(first)
print('second')
print(second)
print('third')
print(third)
print()
print('垂直拆分')
first, second, third = np.split(arr, [1, 3], axis=1)
print('first')
print(first)
print('second')
print(second)
print('third')
print(third)
```

    [[ 0.8965679   0.96870178 -2.0924175  -0.13104816  0.3278197 ]
     [-1.09022272 -1.75083774 -0.54377748  0.65826762 -0.27585484]
     [ 0.99815258  2.68046451  0.6952748  -2.26008359 -1.02304938]
     [ 0.69853033  0.04944017  0.18489227 -0.03483205 -0.22297128]
     [ 1.64612181  0.46242576  0.07580387 -1.17421674 -0.12202246]]
    水平拆分
    first
    [[ 0.8965679   0.96870178 -2.0924175  -0.13104816  0.3278197 ]]
    second
    [[-1.09022272 -1.75083774 -0.54377748  0.65826762 -0.27585484]
     [ 0.99815258  2.68046451  0.6952748  -2.26008359 -1.02304938]]
    third
    [[ 0.69853033  0.04944017  0.18489227 -0.03483205 -0.22297128]
     [ 1.64612181  0.46242576  0.07580387 -1.17421674 -0.12202246]]
    
    垂直拆分
    first
    [[ 0.8965679 ]
     [-1.09022272]
     [ 0.99815258]
     [ 0.69853033]
     [ 1.64612181]]
    second
    [[ 0.96870178 -2.0924175 ]
     [-1.75083774 -0.54377748]
     [ 2.68046451  0.6952748 ]
     [ 0.04944017  0.18489227]
     [ 0.46242576  0.07580387]]
    third
    [[-0.13104816  0.3278197 ]
     [ 0.65826762 -0.27585484]
     [-2.26008359 -1.02304938]
     [-0.03483205 -0.22297128]
     [-1.17421674 -0.12202246]]


### 元素的重复操作

- tile
- repeat


```python
print('Repeat: 按元素')
arr = np.arange(3)
print(arr.repeat(3))
print(arr.repeat([2, 3, 4]))  # 3个元素，分别复制2, 3, 4次。长度要匹配！
print()
print('Repeat，指定轴')
arr = np.random.randn(2, 2)
print(arr)
print('按行repeat')
print(arr.repeat(2, axis=0))  # 按行repeat
print('按列repeat')
print(arr.repeat(2, axis=1))  # 按列repeat
print()
print('Tile: 参考贴瓷砖')
print(np.tile(arr, 2))
print(np.tile(arr, (2, 3)))  # 指定每个轴的tile次数
```

    Repeat: 按元素
    [0 0 0 1 1 1 2 2 2]
    [0 0 1 1 1 2 2 2 2]
    
    Repeat，指定轴
    [[-0.21639649  1.09229637]
     [-0.51345133  0.25951482]]
    按行repeat
    [[-0.21639649  1.09229637]
     [-0.21639649  1.09229637]
     [-0.51345133  0.25951482]
     [-0.51345133  0.25951482]]
    按列repeat
    [[-0.21639649 -0.21639649  1.09229637  1.09229637]
     [-0.51345133 -0.51345133  0.25951482  0.25951482]]
    
    Tile: 参考贴瓷砖
    [[-0.21639649  1.09229637 -0.21639649  1.09229637]
     [-0.51345133  0.25951482 -0.51345133  0.25951482]]
    [[-0.21639649  1.09229637 -0.21639649  1.09229637 -0.21639649  1.09229637]
     [-0.51345133  0.25951482 -0.51345133  0.25951482 -0.51345133  0.25951482]
     [-0.21639649  1.09229637 -0.21639649  1.09229637 -0.21639649  1.09229637]
     [-0.51345133  0.25951482 -0.51345133  0.25951482 -0.51345133  0.25951482]]

> 原文地址：[NumPy基础 | coder_ss's blog ](http://guoshangshu.com/2017/05/11/machine_learning/numpy_summary/index.html)