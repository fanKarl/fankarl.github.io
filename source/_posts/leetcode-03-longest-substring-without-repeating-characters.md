---
title: LeetCode 无重复字符的最长子串
date: 2020-03-30 23:08:48
categories:
    - Algorithm
    - LeetCode
tags:
    - 字符串
    - 窗口滑动
---

[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

>给定一个字符串，请你找出其中不含有重复字符的 最长子串 的长度。
>示例 1:
>输入: "abcabcbb"
>输出: 3 
>解释: 因为无重复字符的最长子串是 "abc"，所以其长度为 3。
>示例 2:
>输入: "bbbbb"
>输出: 1
>解释: 因为无重复字符的最长子串是 "b"，所以其长度为 1。
>示例 3:
>输入: "pwwkew"
>输出: 3
>解释: 因为无重复字符的最长子串是 "wke"，所以其长度为 3。
>请注意，你的答案必须是 子串 的长度，"pwke" 是一个子序列，不是子串。

<!--more-->

### 题解

##### 思路一：暴力破解法

根据题设，一个最简单直接的思路就是从左到右遍历每一种子串，思路如下：
1. 从左向右遍历字符串，以经过的字符为起点，向右扩散验证子串是否重复，这里遍历只需要到n-1即可，因为最后一个字符没必要再验证
2. 向右扩散实际上也是一个循环遍历，只不过最大长度是 i 到 n-2。i为起点的子串每次向右扩展一位，判断新加入元素是否已存在，若不存在，则可以继续扩大子串，否则跳出循环，同时比较记录已出现的最大长度。
3. 验证完所有子串得到的最大长度就是无重复最长字符串长度

```java
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        int maxLength = 1;
        for (int i = 0; i < s.length() - 1; i++) {
            int step = i + 1;
            while (step < s.length()) {
                String tempStr = s.substring(i, step);
                char temp = s.charAt(step);
                if (tempStr.indexOf(temp) == -1) {
                    step++;
                } else {
                    break;
                }
            }
            maxLength = Math.max(step - i, maxLength);
        }
        return maxLength;
    }
```

时间复杂度 O(n^2)

#### 思路二：窗口滑动法

思路一中实际上是存在浪费的情况，比如bacabcbb，当内循环子串加入第二个a的时候，a已经存在，这时候i增加一位，结果还是从a开始，这一次遍历就浪费了。因此可以对算法进行改进，利用窗口滑动，如果遇到上述这种情况，可以直接将子串起点移到第二个a下一位。大致思路如下：

1. 需要准备 max 记录可能存在的最大子串长度， start记录子串当前起点，hashmap，用来存储已经出现的字符和它的位置，key=字符，vlaue=下标
2. 开始遍历字符串，循环变量end作为子串准备加入的字符，也就是在源字符串中的位置，用map中是否已经存在该字符
    1. 不存在该字符，计算当前构成新子串，比较记录是否产生新的最大长度。同时map存入该字符的值和下标
    2. 如果存在该字符，取出mao中记录的该字符的位置，将start移动到该下标位置下一位 map.get(ch) + 1，说明将原来的重复字符移出了窗口外。然后也需要计算比较最大长度、更新已出现该字符新的下标位置。
3. 遍历结束，得到的max就是目标结果

代码如下：

```java
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }

        HashMap<Character, Integer> map = new HashMap<>();
        int max = 0, start = 0;

        for (int end = 0; end < s.length(); end++) {
            char ch = s.charAt(end);
            if (map.containsKey(ch)) {
                start = Math.max(map.get(ch) + 1, start);
            }

            max = Math.max(max, end - start + 1);
            map.put(ch, end);
        }

        return max;
    }
```

使用这种办法的时间复杂度是O(n),比思路一要快很多。

PS: 之所以选择hashMap 是因为它存取数据的时间复杂度是O(1)，即使最差也是O(n)
