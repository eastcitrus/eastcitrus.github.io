---
layout: post
title: Sort
date:  2016-07-14 17:22:22 +0800
categories: [Algorithm]
tags: [sort]
published: true
---


# 排序

![排序算法](https://p9-tt-ipv6.byteimg.com/origin/pgc-image/407dc3c36843431389ed08ec9b6f1b59)

【Exchange sorts】

冒泡排序 BubbleSort

快速排序 Quicksort

【Selection sorts】

[Selection sort](https://houbb.github.io/2016/07/14/sort-03-select-sort)

Heapsort

【Insertion sorts】

Insertion sort

Shellsort

【Merge sorts】

Merge sort

【Distribution sorts】

Bucket sort

Counting sort

------------------------------------------------------------------------

【树形数据结构】

树：

树，BST

AVL 红黑树

B B+ B- 树



# Sort

Here are some tools for sort.

- RandomUtil.java

```java
public class RandomUtil {
    public static int[] randomArray(final int size) {
        int[] array = new int[size];

        Random random = new Random();
        for(int i = 0; i < size; i++) {
            array[i] = random.nextInt(100);
        }

        return array;
    }
}
```

- SortUtil.java

```java
public class SortUtil {
    private SortUtil(){}

    /**
     * swap array[index] & array[index+1]
     * @param array
     * @param index
     */
    private static void swap(int[] array, int index) {
        int temp = array[index];
        array[index] = array[index+1];
        array[index+1] = temp;
    }

    /**
     * swap array[dest] & array[target]
     * @param array
     * @param destIndex
     * @param targetIndex
     */
    private static void swap(int[] array, int destIndex, int targetIndex) {
        int temp = array[destIndex];
        array[destIndex] = array[targetIndex];
        array[targetIndex] = temp;
    }

    public static void show(int[] array) {
        for(int value : array) {
            System.out.print(value+"\t");
        }
        System.out.println();
    }
}
```

# Bubble sort

TODO优化：


> [bubble sort](https://en.wikipedia.org/wiki/Bubble_sort)

![example](https://raw.githubusercontent.com/houbb/resource/master/img/2016-07-16-bubble-sort.gif)

Starting from the beginning of the list, compare every adjacent pair, swap their position if they are not in the right order (the latter one is smaller than the former one).
After each iteration, one less element (the last one) is needed to be compared until there are no more elements left to be compared.

- bubbleSort()

```java
public static void bubbleSort(int[] array) {
    for(int i = 0; i < array.length-1; i++) {
        for(int j = 0; j < array.length-1-i; j++) {
            if(array[j] > array[j+1]) {
                SortUtil.swap(array, j);
            }
        }
    }
}
```

- test

```java
@Test
public void testSort() {
    int[] array = RandomUtil.randomArray(10);
    SortUtil.show(array);

    SortUtil.bubbleSort(array);

    SortUtil.show(array);
}
```

- result

```
39	97	71	51	39	54	13	7	90	39
7	13	39	39	39	51	54	71	90	97

Process finished with exit code 0
```


<label class="label label-info">Tips</label>

We can add a flag to improve the speed of bubble sort, it works well when array **has sorted**.

- bubbleSortFlag()

```java
public static void bubbleSortFlag(int[] array) {
    boolean flag = true;

    for(int i = 0; i < array.length-1; i++) {
        flag = false;
        for(int j = 0; j < array.length-1-i; j++) {
            if(array[j] > array[j+1]) {
                SortUtil.swap(array, j);

                flag = true;
            }
        }

        if(!flag) {
            break;
        }
    }
}
```

# Selection sort

> [selection sort](https://en.wikipedia.org/wiki/Selection_sort)

![example](https://raw.githubusercontent.com/houbb/resource/master/img/2016-07-16-selection-sort.gif)


- selectionSort()

```java
public static void selectionSort(int[] array) {
    for(int i = 0; i < array.length-1; i++) {
        int minIndex = i;
        for(int j = i+1; j < array.length; j++) {
            if(array[j] < array[minIndex]) {
                minIndex = j;    //获取最小下标
            }
        }

        if(minIndex != i) {
            swap(array, minIndex, i);
        }
    }
}
```

- test

```java
@Test
public void testSelectionSort() {
    int[] array = RandomUtil.randomArray(10);
    SortUtil.show(array);

    SortUtil.selectionSort(array);
    SortUtil.show(array);
}
```

- result

```
7	19	38	23	63	11	0	55	77	78
0	7	11	19	23	38	55	63	77	78

Process finished with exit code 0
```

# insertion sort

## 插入排序

插入排序（英语：Insertion Sort）是一种简单直观的排序算法。

它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

插入排序在实现上，通常采用in-place排序（即只需用到 O(1) 的额外空间的排序），因而在从后向前扫描过程中，需要反复把已排序元素逐步向后挪位，为最新元素提供插入空间。

## 算法描述

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

1. 从第一个元素开始，该元素可以认为已经被排序

2. 取出下一个元素，在已经排序的元素序列中从后向前扫描

3. 如果该元素（已排序）大于新元素，将该元素移到下一位置

4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置

5. 将新元素插入到该位置后

重复步骤 2~5

如果比较操作的代价比交换操作大的话，可以采用二分查找法来减少比较操作的数目。

该算法可以认为是插入排序的一个变种，称为二分查找插入排序。

## 实例代码

- 实现 1

```java
public void insertionSort(int[] array) {
	for (int i = 1; i < array.length; i++) {
		int key = array[i];
		int j = i - 1;
		while (j >= 0 && array[j] > key) {
			array[j + 1] = array[j];
			j--;
		}
		array[j + 1] = key;
	}
}
```

- 实现 2

```java
//将arr[i] 插入到arr[0]...arr[i - 1]中
public static void insertion_sort(int[] arr) {
	for (int i = 1; i < arr.length; i++ ) {
		int temp = arr[i];
		int j = i - 1;  
//如果将赋值放到下一行的for循环内, 会导致在第10行出现j未声明的错误
		for (; j >= 0 && arr[j] > temp; j-- ) {
			arr[j + 1] = arr[j];
		}
		arr[j + 1] = temp;
	}
}
```

## 算法复杂度

如果目标是把n个元素的序列升序排列，那么采用插入排序存在最好情况和最坏情况。

最好情况就是，序列已经是升序排列了，在这种情况下，需要进行的比较操作需 `(n-1)` 次即可。

最坏情况就是，序列是降序排列，那么此时需要进行的比较共有 `n*(n-2)/2` 次。

插入排序的赋值操作是比较操作的次数减去 `(n-1)` 次，（因为 `(n-1)` 次循环中，每一次循环的比较都比赋值多一个，多在最后那一次比较并不带来赋值）。

平均来说插入排序算法复杂度为 `O(n^2)`。

因而，插入排序不适合对于数据量比较大的排序应用。

但是，如果需要排序的数据量很小，例如，量级小于千；或者若已知输入元素大致上按照顺序排列，那么插入排序还是一个不错的选择。 

插入排序在工业级库中也有着广泛的应用，在STL的sort算法和stdlib的qsort算法中，都将插入排序作为快速排序的补充，用于少量元素的排序（通常为8个或以下）。

## 参考资料

[插入排序](https://zh.wikipedia.org/wiki/%E6%8F%92%E5%85%A5%E6%8E%92%E5%BA%8F)

[二分查找法](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%9F%A5%E6%89%BE%E6%B3%95)

* any list
{:toc}




