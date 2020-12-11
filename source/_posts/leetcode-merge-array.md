---
title: LeetCode 合并数组
date: 2020-03-04 21:48:41
categories:
    - Algorithm
    - LeetCode
tags:
    - 数组
---
### 合并数组
给定两个排序后的数组 A 和 B，其中 A 的末端有足够的缓冲空间容纳 B。 编写一个方法，将 B 合并入 A 并排序。初始化 A 和 B 的元素数量分别为 m 和 n。示例如下:
>输入
>A = [1,2,3,0,0,0], m = 3
>B = [2,5,6],       n = 3
>输出: [1,2,2,3,5,6]

<!--more-->

#### 解法一

```java
public static void merge01(int[] A, int m, int[] B, int n) {
        if (n == 0) {
            return;
        }
        if (m == 0) {
            if (n >= 0) {
                System.arraycopy(B, 0, A, 0, n);
            }
            return;
        }
        int stepA = m - 1;
        int stepB = n - 1;
        int step = (m + n - 1);

        while (step >= 0) {
            if (stepB < 0) {
                A[step] = A[stepA];
                stepA--;
                step--;
                continue;
            }
            if (stepA < 0) {
                A[step] = B[stepB];
                stepB--;
                step--;
                continue;
            }
            if (A[stepA] > B[stepB]) {
                A[step] = A[stepA];
                stepA--;
            } else {
                A[step] = B[stepB];
                stepB--;
            }
            step--;
        }
    }
```

解题思路

首先已知条件是 A数组有足够的剩余空间容纳B数组，所以一个很简单的思路就是直接把B数组加入到A数组进行快排，但是因为AB都是有序数组，快排会带来大量的冗余操作。所以考虑优化一下：
AB都是已经排序的数组，又已经AB初始化长度，那可以在插入的时候就开始进行比较。思路如下：
1. 根据m、n计算出AB合并后总长度 m+n
2. 声明三个指针stepA、stepB、step，分别指向A、B末端，以及合并后A末端
3. 这一步是关键，根据stepA或者stepB从AB中取出元素进行比较，谁的元素大就把该元素取出来放到step指向位置，同时取出元素的数组指针(stepA 或者 stepB)前移一位,另一个指针不变，step也需要在元素插入后前移一位。
4. 重复步骤三直到stepA小于0或者stepB小于零，那就把没有取完的数组剩余元素都插入step循环到0为止指向的位置。当step也小于0的时候跳出循环，就完成了AB数组的合并

PS：
1. 这里需要考虑边界条件，A或者B为空的时候，以及while循环的跳出条件。
2. 因为m+n 提供了足够的长度，因此在比较过程中不必担心从A或者B中取出来的元素在插入的过程中会覆盖A中未用到的元素。

时间复杂度是O(n)

