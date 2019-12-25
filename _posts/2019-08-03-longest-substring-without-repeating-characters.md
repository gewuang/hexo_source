---
layout: mypost
title: longest-substring-without-repeating-characters 
categories: [leetcode]
tags: "算法" 
---

### Title 

Given a string, find the length of the longest substring without repeating characters.

Example 1:
```
Input: "abcabcbb"
Output: 3 
Explanation: The answer is "abc", with the length of 3. 
```
Example 2:
```
Input: "bbbbb"
Output: 1
Explanation: The answer is "b", with the length of 1.
```
Example 3:
```
Input: "pwwkew"
Output: 3
Explanation: The answer is "wke", with the length of 3. 
             Note that the answer must be a substring, "pwke" is a subsequence and not a substring.
```

### my anwser 

```c++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        set<char> s_str;
        int i = 0;
        int j = 0;
        int len = s.length();
        int ret = 0;
        for(i = 0; i < len; i++) {
            s_str.insert(s[i]);
            for(j = 1; j < len - i; j++) {
                if(s_str.count(s[i+j]) == 0) {
                    s_str.insert(s[i+j]);
                }
                else {
                    s_str.clear();
                    break;
                }
            }
            if(ret < j) {
                ret = j;
            }
        }
        return ret;
    }
};
```
实打实的暴力破解法，这次没用c语言是因为考虑到查看新增的字符是不是出现在前面，又要再加个循环，跑到O(n^3)，实在是花费太多时间了，那就用set代替，虽然知道这样做在set内部也要用插入删除的操作，但总归是比for看起来舒服很多的，结果也是相当的不理想，不过因为自己这次在优化过程中一直想着是不是能用动态规划而把自己限制死了，私下在研究下动态规划吧🌚。
不过，在看了标准答案后真的是豁然开朗，原来是这么做的，其实自己如果一步步优化很多东西完全是可以想到的，以后要注意尽量不要眼高手低，下面先来欣赏标准答案第一个精彩的解法（直接忽略掉暴力解法）：

**解法一**

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length();
        Set<Character> set = new HashSet<>();
        int ans = 0, i = 0, j = 0;
        while (i < n && j < n) {
            // try to extend the range [i, j]
            if (!set.contains(s.charAt(j))){
                set.add(s.charAt(j++));
                ans = Math.max(ans, j - i);
            }
            else {
                set.remove(s.charAt(i++));
            }
        }
        return ans;
    }
    }
```

精彩的滑动窗口解法，一次遍历字符串，遇到重复的就移动左边届，直到没有重复内容，其实就是在暴力破解的基础上做了一些巧妙的改动，这个代码比较简单，比较好理解，重点放在后面两个优化后的算法上。

**解法二**

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        Map<Character, Integer> map = new HashMap<>(); // current index of character
        // try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            if (map.containsKey(s.charAt(j))) {
                i = Math.max(map.get(s.charAt(j)), i);
            }
            ans = Math.max(ans, j - i + 1);
            map.put(s.charAt(j), j + 1);
        }
        return ans;
    }
    }
```

非常精彩的解法，让我想到了kmp算法，这里的map就相当于next数组，在之前滑动窗口的基础上进行了改进，之前左边界需要逐步移动，这里通过map获取到当前字符对应的value值直接得到下一步左边届的位置，与next数组有异曲同工之妙，下次我也找个时间细细研究下kmp算法，更新一篇关于kmp算法的文章，这里就不多说了，先介绍上面的算法。

从上面聊下来，其实这里最难理解的就是i作为左边届(上面i是当前匹配字符串的左边界，j是右边界)，是怎么快速得到的。

首先，map在这里用的非常巧妙，map对应的值始终是下一跳的位置，我们以*abcaa*为例，看map中k以及v及i的变化:

| j | 0 | 1 | 2 | 3 | 4 | 
|----|----|----|----|----|----|
| k | a | b | c | a | a | 
| v | 1 | 2 | 3 | 4 | 5 | 
| i | 0 | 0 | 0 | 1 | 4 | 

从表中可以看出，当遇到相同的k时，递增的v就会改变对应的值为下次下标所在的位置，i此时获得的时之前value的值，也就是对应的左边届的位置是根据上次字符出现时的next位置决定的，因为在这之间的字符我们已经可以判断没有在轮询的必要了（每次都跳到上次出现的后一个位置）。

下面是针对上面这种解法的又一次优化：

**解法三**

```java
public class Solution {
    public int lengthOfLongestSubstring(String s) {
        int n = s.length(), ans = 0;
        int[] index = new int[128]; // current index of character
        // try to extend the range [i, j]
        for (int j = 0, i = 0; j < n; j++) {
            i = Math.max(index[s.charAt(j)], i);
            ans = Math.max(ans, j - i + 1);
            index[s.charAt(j)] = j + 1;
        }
        return ans;
    }
    }
```

这里针对ascii码是有限的（而且边界比较小），用数组替换了map，完成了这个精彩的算法，上面我们说的挺多的了，这里就不重复说了。


### 总结

这次完全是自己眼高手低了，算法应该先用自己能想到的方法实现，然后针对该算饭进行一步步优化，而不是刚开始就想着一口吃个胖子，有个多优秀的算法解决。同时，动态规划也要好好学习，首先，最起码应该知道什么情况下使用动态规划。
