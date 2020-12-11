---
title: LeetCode 最长回文子串
date: 2020-03-30 10:41:37
categories:
    - Algorithm
    - LeetCode
tags:
    - 动态规划
    - todo
---

[LeetCode 计算最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring)
>给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。
>示例 1：
>输入: "babad"
>输出: "bab"
>注意: "aba" 也是一个有效答案。
>示例 2：
>输入: "cbbd"
>输出: "bb"

<!--more-->

### 解题思路

首先我们要了解什么是回文字符串：字符串正着读和反着读一模一样的字符串。比如 abbabb、bbabb， 但是 bd、ababc 这样不是回文字符串。

接着看一下题目，要找到字符串s中所有的回文子串，先排除字符串为空的情况，字符串长度为 1 直接就是最长回文字符串。然后来看一下剩余情况，要知道最长回文子串，最简单直接的方法就是把 字符串所有的子串都找出来，检查是不是回文子串。

#### 暴力破解法

暴力破解法比较简单，直接上代码：

```java
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 2) {
            return s;
        }

        int maxLength = 1;
        String res = s.substring(0, 1);
        for (int i = 0; i < s.length(); i++) {
            for (int j = i + 1; j < s.length(); j++) {
                if (j - i + 1 > maxLength && isPalindrome(s, i, j)) {
                    maxLength = j - i + 1;
                    res = s.substring(i, j + 1);
                }
            }
        }
        return res;
    }

    private boolean isPalindrome(String src, int left, int right) {
        while (left < right) {
            if (src.charAt(left) != src.charAt(right)) {
                return false;
            }

            left++;
            right--;
        }
        return true;
    }
```

暴力破解虽然比较简单，但是时间复杂度也比较高，需要三层循环遍历，也就是O(n^3)，空间复杂度O(1)

暴力解法在数据量较大的时候显然是不合适的，我们可以尝试对算法进行改进，这就需要用到中心扩散法

#### 中心扩散法

中心扩散法是根据回文字符串的特性来解决问题的。每个回文串都有其中心，奇数回文串中心是中间字符，偶数回文串的中心是最终将的空隙，根据这点我们可以去考虑，源字符串从左向右移动指针，以每个字符为中心，向两边扩散找出符合条件的回文串，记录最大回文串结果就是满足题设的答案。

这时候又有一个问题，偶数回文字符串中心是空隙，怎么办？也有两种解法可以考虑

##### 解法一：奇数+偶数一起判断

我们在指定中心字符进行扩散验证的时候，可以同时使用两个指针，如果两个指针指向一致就是在判断是否存在奇数回文串，如果指针相邻，则是在判断是否存在偶数回文串。

```java
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 2) {
            return s;
        }
        int maxLength = 1;
        String res = s.substring(0, 1);
        for (int i = 0; i < s.length() - 1; i++) {
            String oddStr = centerSpread(s, i, i);
            String evenStr = centerSpread(s, i, i + 1);
            String maxString = oddStr.length() > evenStr.length() ? oddStr : evenStr;
            if (maxString.length() > maxLength) {
                maxLength = maxString.length();
                res = maxString;
            }
        }
        return res;
    }

    private String centerSpread(String src, int left, int right) {
        int length = src.length();
        int i = left;
        int j = right;
        while (i >= 0 && j < length && src.charAt(i) == src.charAt(j)) {
            i--;
            j++;
        }
        return src.substring(i + 1, j);
    }
```
这种解法的时间复杂度是 O(n^2)空间复杂度O(1)

但是这种解法也有缺陷，就是每次还要考虑奇数和偶数回文串，太麻烦。因此引入第三种解法，插入特殊字符来避免考虑奇偶性

##### 解法二： 中心扩散法Plus版

如果我们给整个字符串前后以及字符间隙都插入特殊字符 “#” （也可以是别的特殊字符，但是不能是字符串中存在的字符），这样奇数回文串判断方式不变，偶数间隙有字符"#"成了跟奇数回文串一样了。缺点就是牺牲了空间效率。

代码如下：

```java
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 2) {
            return s;
        }
        String dstStr = addBoundaries(s, '#');
        int maxLength = 1;

        int start = 0;
        for (int i = 0; i < dstStr.length(); i++) {
            int step = centerSpread(dstStr, i);
            if (step > maxLength) {
                maxLength = step;
                //根据新串中的位置，和目标回文串的宽度计算原字符串中的起始位置
                start = (i - maxLength) / 2;
            }
        }

        return s.substring(start, start + maxLength);

    }

    private int centerSpread(String src, int center) {
        int length = src.length();
        int left = center - 1;
        int right = center + 1;

        int step = 0;
        while (left >= 0 && right < length && src.charAt(left) == src.charAt(right)) {
            left--;
            right++;
            step++;
        }

        return step;
    }

    /**
    * 构造含有特殊字符的新字符串，长度是源字符串的 2n+1 
    */
    private String addBoundaries(String src, char divider) {
        if (src == null || src.length() == 0) {
            return "";
        }

        if (src.indexOf(divider) != -1) {
            throw new IllegalArgumentException("参数错误，不可以传递已经存在的字符作为分隔符");
        }

        StringBuilder stringBuilder = new StringBuilder();
        stringBuilder.append(divider);
        for (char ch : src.toCharArray()) {
            stringBuilder.append(ch).append(divider);
        }
        return stringBuilder.toString();
    }
```
时间复杂度 O(n^2), 空间复杂度O(1)

代码比较简单，主要对比一下两种解法的区别，Plus版centerSpread()改进了传入参数，原来需要传入两个指针，现在只需要一个中心指针即可，另外返回的step实际上就是符合条件的回文串的长度，这里不直接返回字符串是因为带有特殊字符串，不如直接记录长度和中心位置，进一步计算出在源字符串中的起始和终点位置。


##### 解法三： Manacher解法 

更优解法：Manacher解法，时间复杂度只有O(n)，这里我还没考虑清楚，待补充！！！


#### 参考链接
[LeetCode 第五题 最长回文子串](https://www.cxyxiaowu.com/2869.html)
