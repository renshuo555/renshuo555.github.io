---
layout: post
title: Java排序总结
date: 2018-2-27 09:25:57
categories: Java
tags: Java Sort
author: renshuo
mathjax: true
---

* content
{:toc}

对Java常用排序算法总结，主要是八大内部排序。

<!--more-->

## 概要总结

![java排序](/pictures/java/sort-algorithm-summary/java排序.png)

### 稳定性

* **稳定**：冒泡排序、插入排序、归并排序和基数排序
* **不稳定**：选择排序、快速排序、希尔排序、堆排序

### 算法选择

1. **数据规模较小**

* 待排序列基本序的情况下，可以选择**直接插入排序**；
* 对稳定性不作要求宜用简单选择排序，对稳定性有要求宜用插入或冒泡

2. **数据规模不是很大**

* 完全可以用内存空间，序列杂乱无序，对稳定性没有要求，快速排序，此时要付出log（N）的额外空间。
* 序列本身可能有序，对稳定性有要求，空间允许下，宜用归并排序

3. **数据规模很大**

* 对稳定性有求，则可考虑归并排序。
* 对稳定性没要求，宜用堆排序

4. **序列初始基本有序（正序）**

* 宜用直接插入，冒泡


### 复杂度分析

![排序算法对比图](/pictures/java/sort-algorithm-summary/排序算法对比图.jpg)

O(n^2):直接插入排序，简单选择排序，冒泡排序。

在数据规模较小时（9W内），直接插入排序，简单选择排序差不多。当数据较大时，冒泡排序算法的时间代价最高。性能为O(n^2)的算法基本上是相邻元素进行比较，基本上都是稳定的。

O(nlogn):快速排序，归并排序，希尔排序，堆排序。

其中，快排是最好的，其次是归并和希尔，堆排序在数据量很大时效果明显。

## 内部排序

内部排序是数据记录在内存中进行排序，本文介绍的八个基本排序算法也都属于内部排序。

### 插入排序

每一趟将一个待排序的元素，按其关键字值的大小插入到已经排过序的有序区中的适当位置，直到全部插入完成为止。不同插入排序算法的最根本的不同点是根据什么规则去找新元素的插入点，直接插入排序是依次寻找，希尔排序缩小增量再使用直接插入排序。

#### 直接插入排序

依次将无序区中的每一个元素插入到一个有序区中去。

``` java
/**
 * 直接插入排序
 *
 * @param arr 待排序数组
 */
private static void insertSort(int[] arr) {
    for (int i = 0; i < arr.length; i++) {
        int temp = arr[i];
        int j = i - 1;
        while (j >= 0 && temp < arr[j]) {
            // 将大于temp的值整体后移一个单位
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = temp;
    }
}
```

**二分查找插入排序**：上面的插入排序算法，是使用从后往前的顺序查找进行比较的，为了减少比较次数，可以使用二分查找进行改进。

**2-路插入排序算法**是在折半插入排序的基础之上进行的改进，主要贡献是减少了排序过程中移动记录的次数，不足的地方是需要辅助空间。

#### 希尔排序

在要排序的一组数中，根据某一增量分为若干子序列，并对子序列分别进行插入排序。然后逐渐将增量减小,并重复上述过程。直至增量为1,此时数据序列基本有序,最后进行插入排序。

关于希尔排序中增量序列的深入可以看这篇文章：[希尔排序增量序列简介](http://blog.csdn.net/foliciatarier/article/details/53891144)

``` java
/**
 * 希尔排序
 * 增量使用Knuth增量序列
 *
 * @param arr 待排序数组
 */
private static void shellSort(int[] arr) {
    int gap = 1;
    int temp;
    int i, j;
    while (gap < arr.length / 3) {
        gap = gap * 3 + 1;
    }
    for (; gap > 0; gap /= 3) {
        for (i = gap; i < arr.length; i++) {
            temp = arr[i];
            for (j = i - gap; j >= 0 && arr[j] > temp; j -= gap) {
                arr[j + gap] = arr[j];
            }
            arr[j + gap] = temp;
        }
    }
}
```

### 选择排序

每趟从待排序的记录序列中选择关键字最小的记录放置到已排序表的最前位置，直到全部排完。常用于选择序列中最大或者最小的几个元素。

#### 简单选择排序

在需要排序的一组元素中，选择最小的与未排序序列中第一个位置的元素进行交换，重复操作，直到倒数第二个数和最后一个数比较为止。

``` java
/**
 * 简单选择排序
 *
 * @param arr 待排序数组
 */
private static void selectSort(int[] arr) {
    int min, temp;
    for (int i = 0; i < arr.length - 1; i++) {
        min = i;
        for (int j = i + 1; j < arr.length; j++) {
            if (arr[j] < arr[min]) {
                min = j;
            }
        }
        temp = arr[i];
        arr[i] = arr[min];
        arr[min] = temp;
    }
}
```

#### 堆排序

堆排序是对简单选择排序的一种优化，将序列构建成大顶堆，将根结点与最后一个结点交换，然后断开最后一个结点，重复操作，直到所有结点断开。

堆排序有两个主要问题，第一个是*如何将n个待排序元素建立成堆*，第二个是*输出堆顶元素后，怎样调整剩下的n-1个元素，使之成为一个新堆*。

1. **建立堆**

* n个结点的完全二叉树，最后一个结点的父结点是 `n/2` (向下取整)。
* 从第`n/2` (向下取整) 结点为根的子树开始，构建堆结构，(与左右孩子结点比较、交换，直到叶子节点)。
* 之后往前依次对各结点为根的子树进行构建堆结构操作，直到根结点。

2. **调整堆的方法**

- 设有m 个元素的堆，输出堆顶元素后，剩下m-1 个元素。将堆底元素送入堆顶（最后一个元素与堆顶进行交换），堆被破坏，其原因仅是根结点不满足堆的性质。
- 将根结点与左、右子树中较大或者较小元素的进行交换。
- 若与左子树交换：如果左子树堆被破坏，即左子树的根结点不满足堆的性质，则重复第二步。
- 若与右子树交换，如果右子树堆被破坏，即右子树的根结点不满足堆的性质，则重复第二步。
- 继续对不满足堆性质的子树进行上述交换操作，直到叶子结点，堆被建成。

可以发现在建立堆的过程中，是可以使用调整堆方法来逐步进行的。

``` java
/**
 * 堆排序
 *
 * @param arr 待排序数组
 */
private static void heapSort(int[] arr) {
    // 首先构建大顶堆
    for (int i = arr.length / 2 - 1; i >= 0; i--) {
        buildMaxHeap(arr, i, arr.length - 1);
    }

    // System.out.println("构建堆结构：");
    // System.out.println(Arrays.toString(arr));

    for (int i = arr.length - 1; i >= 0; i--) {
        // 交换堆顶元素与最后一个记录元素
        int temp = arr[0];
        arr[0] = arr[i];
        arr[i] = temp;
        // 调整堆结构
        buildMaxHeap(arr, 0, i - 1);
    }
}

/**
 * 构建大顶堆
 *
 * @param a          元素序列
 * @param firstIndex 第一个结点所在位置
 * @param lastIndex  最后一个结点所在位置
 */
private static void buildMaxHeap(int[] a, int firstIndex, int lastIndex) {
    int temp = a[firstIndex];
    // 恢复堆结构的过程，从根结点向下
    // 以0开始的数组，每个结点左孩子index应该是奇数，右孩子是偶数
    for (int i = firstIndex * 2 + 1; i <= lastIndex; i *= 2) {
        // 获取左右孩子中较大元素的结点下标
        if (i < lastIndex && a[i] < a[i + 1]) {
            i++;
        }
        if (temp < a[i]) {
            a[firstIndex] = a[i];
            firstIndex = i;
        }
    }
    a[firstIndex] = temp;
}
```

### 交换排序

边比较边调整元素的位置

``` java
/**
 * 交换数组中两个位置的元素
 *
 * @param arr 数组
 * @param i   位置1
 * @param j   位置2
 */
private static void swap(int[] arr, int i, int j) {
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

#### 冒泡排序

在要排序的一组数中，对当前还未排好序的范围内的全部数，对相邻的两个数依次进行比较和调整，让较大的数往下沉，较小的往上冒。即：每当两相邻的数比较后发现它们的排序与排序要求相反时，就将它们互换。

每一次冒泡的过程，都会将最大或者最小的元素放在指定的位置上，和选择冒泡的方向有关。

``` java
/**
 * 冒泡排序
 *
 * @param arr 待排序序列
 */
private static void bubbleSort(int[] arr) {
    // 当剩下最后一个元素时，不需要再操作了
    for (int i = 0; i < arr.length - 1; i++) {
        // 每趟冒泡将未排序序列中最小的放在了前面
        for (int j = arr.length - 1; j > i; j--) {
            if (arr[j] < arr[j - 1]) {
                swap(arr, j, j - 1);
            }
        }
    }
}
```

**冒泡排序的改进**：

- 设置标志变量，当一轮比较结束后，没有发生交换，表示已经有序。
- 设置标志位`p`，记录最后一次交换的位置，下一趟排序时只需要扫描到`p`位置即可

#### 快速排序

选择一个基准元素，将序列中其它元素与基准元素进行比较，分为比它大和比它小的两部分，此时基准元素处在排序后的正确位置上，用同样的方法递归排序划分的两部分。

基准元素可以选择第一个或者最后一个或者其它方式选取。

``` java
/**
 * 快速排序
 *
 * @param arr   待排序序列
 * @param start 开始位置
 * @param end   结束位置
 */
private static void quickSort(int[] arr, int start, int end) {
    if (start < end) {
        int middle = partition(arr, start, end);
        quickSort(arr, start, middle - 1);
        quickSort(arr, middle + 1, end);
    }
}

/**
 * 快速排序划分函数
 *
 * @param arr  待排序序列
 * @param low  划分开始部分
 * @param high 划分结束部分
 * @return 基准元素所在位置
 */
private static int partition(int[] arr, int low, int high) {
    // 取第一个元素为基准元素
    int key = arr[low];
    while (low < high) {
        while (arr[high] >= key && high > low) {
            high--;
        }
        arr[low] = arr[high];

        while (arr[low] <= key && low < high) {
            low++;
        }
        arr[high] = arr[low];
    }
    // 此时low == high
    arr[low] = key;
    return low;
}
```

**快速排序的改进**：

- 优化基准元素的选择，如三者取中等方法。
- 只对长度大于k的序列使用快速排序，当序列基本有序时，使用其它排序方法(如插入排序)进一步排序。

### 归并排序

将两个或两个以上的有序序列合并成一个新的有序序列，以此类推，直到全部元素归并到一个序列中。

``` java
/**
 * 二路归并排序
 *
 * @param arr   待排序序列
 * @param left  起始位置
 * @param right 末尾位置
 */
private static void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = (left + right) >>> 1;

        mergeSort(arr, left, mid); //递归排序左边序列
        mergeSort(arr, mid + 1, right); //递归排序右边序列
        merge(arr, left, mid, right); // 合并
    }
}

/**
 * 合并两个相邻的有序序列
 *
 * @param arr   待合并序列
 * @param left  起始位置
 * @param mid   中间位置
 * @param right 末尾位置
 */
private static void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left;
    int j = mid + 1;
    int k = 0;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) {
            temp[k++] = arr[i++];
        } else {
            temp[k++] = arr[j++];
        }
    }

    while (i <= mid) {
        temp[k++] = arr[i++];
    }

    while (j <= right) {
        temp[k++] = arr[j++];
    }

    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

### 基数排序

基本思想：将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后,数列就变成一个有序序列。

## 参考资料:

- [面试中的排序算法总结](http://www.cnblogs.com/wxisme/p/5243631.html)
- [八大排序算法](http://blog.csdn.net/hguisu/article/details/7776068)
- [Java常用排序算法/程序员必须掌握的8大排序算法](http://blog.csdn.net/a125138/article/details/7752973)
- [Java常用的八种排序算法与代码实现](http://www.cnblogs.com/10158wsj/p/6782124.html)
