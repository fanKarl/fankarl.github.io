---
title: LeetCode 反转链表
date: 2020-03-02 21:45:01
categories: 
    - Algorithm
    - LeetCode
tags: 
    - 链表
    - todo
---

### 反转一个链表

>输入: 1->2->3->4->5->NULL
>输出: 5->4->3->2->1->NULL

进阶： 迭代|递归反转链表 

<!--more-->

前置条件：链表数据结构

```java
    //前置条件
    public class ListNode {
        public int val;
        public ListNode next;

        public ListNode(int x) {
            val = x;
        }
    }
```


#### 解法一：暴力解法

```java
    //解法
    public ListNode reverseList(ListNode head) {
        ListNode node = null;
        while (head != null) {
            ListNode temp = head.next;
            head.next = node;
            node = head;
            head = temp;
        }
        return node;
    }
```

解题思路：

链表反转思路就是反转每一个节点的指针就可以了，以此为思路步骤如下：
1. 声明一个新的链表用于装载反转后的链表，即目标链表
2. 循环遍历原链表
    1. 每次从原链表取出一个元素，将该元素指向目标链表，成为目标链表新的头部节点
    2. 原链表指向下一个节点，也就是少了一个元素
    3. 重复循环1和2步骤直到原链表为null,跳出循环
3. 此时目标链表就实现了对原链表的反转


该方法时间复杂度是：O(n) 


#### 解法二：

考虑思路是递归，但是还没想到，待补充！！！
