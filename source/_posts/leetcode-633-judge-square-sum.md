---
title: LeetCode 平方数之和
date: 2020-05-21 22:57:47
categories:
    - Algorithm
    - LeetCode
tags:
    - 二分查找
    - todo
---

[平方数之和](https://leetcode-cn.com/problems/sum-of-square-numbers/)

> 给定一个非负整数 c ，你要判断是否存在两个整数 a 和 b，使得 a^2 + b^2 = c。
> 
> 示例1：
> 输入: 5
> 输出: True
> 解释: 1 * 1 + 2 * 2 = 5
> 
> 示例2
> 输入: 3
> 输出: False

<!--more-->

#### 解法一：暴力破解

暴力破解思路比较简单：
1. 首先求出 c 的平方数 sqrt，那么 a 或者 b 都不会大于这个平方数，由此确定 取值范围[0,sqrt]
2. 给 a 依次设定目标范围内的取值，求出 a 的二次方和 c 的差值 m
3. 理论来说 m 开平方得到的就是b，但是需要判断 m 是不是一个平方数，也就是 b的二次方取值是不是正好等于m
4. 找到符合条件的 b 即可返回 true，循环结束没有符合条件，返回false

```java
    public boolean judgeSquareSum(int c) {
        int sqrt = (int) Math.sqrt(c);

        for (int i = 0; i <= sqrt; i++) {
            int m = c - i * i;
            int a = (int) Math.sqrt(m);
            if (a * a == m){
                return true;
            }
        }
        return false;
    }
```

这种解法的时间复杂度是 O(n)，测试结果：超过 22% 的提交，显然是低效的，而且需要借助 Math 相关API，更不完美了。

#### 解法二：二分查找法

如果不借助 Math API，那么我们就得扩大搜索范围，a 取值从0 开始直到平方值大于 c 为止，然后差值就是 b 的目标平方值，而要寻找 是否有符合条件的 b，可以用到二分查找方法，思路如下：
1. 循环遍历直到 a * a >= c，每次循环得到 b 的目标平方值 m
2. 从[0,m] 范围对筛选是否有符合条件的b，下面来看如何使用二分查找
3. 取范围start 和end 的中值mid，判断mid * mid 与 m 的关系
4. 如果比 m 大，说明中值过大，往小的范围去找[mid+1,end]
5. 如果比 m 小，说明中值太小，去大的范围找 [start,mid-1]
6. 设定终止条件，如果 start 已经大于 end 了，那就返回false

```java
    public boolean judgeSquareSum2(int c) {

        for (long a = 0; a * a <= c; a++) {
            int m = c - (int) (a * a);
            if (binarySearch(0, m, m)) {
                return true;
            }
        }
        return false;
    }

    
    public boolean binarySearch(long start, long end, int m) {
        if (start > end) {
            return false;
        }

        long mid = (start + end) / 2;
        if (mid * mid == m) {
            return true;
        }

        if (mid * mid > m) {
            return binarySearch(start, mid - 1, m);
        }

        return binarySearch(mid + 1, end, m);
    }
```

时间复杂度：
首先外层循环的复杂度是：O(n), 二分查找的时间复杂度是 Logn,
所以总的时间复杂度应该是 nLog n

二分查找，虽然提高了查询效率，但是抵不过范围太大，因此效率还不如sqrt 函数。这种方法也不完美。

#### 方法三：费马平方和定理

费马平方和 这个涉及到知识盲区了，没看懂，待补充




