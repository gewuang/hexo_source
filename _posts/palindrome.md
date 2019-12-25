---
layout: mypost
title: 最长回文子序列 
categories: [leetcode]
tags: "算法" 
---

### 题目 

给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。

示例1:
```
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```
示例2:
```
输入: "cbbd"
输出: "bb"
```

### anwser 

```cpp
class Solution {
public:
    bool isPalindromic(string sub) {
        bool ret = true;
        stack<char> c;
        int len = sub.length();
        int i = 0;
        char a;
        
        for (; i<len; i++) {
            c.push(sub[i]);
        }
        for(i=0; i<len; i++) {
            a = c.top();
            c.pop();
            if(sub[i] != a) {
                ret = false;
                break;
            }
        }
        
        return ret;
    }
    string longestPalindrome(string s) {
        string ret = s.substr(0,1);
        int len = s.length();
        int i = 0;
        int j = 0;
        int max = 0;
        
        for(; i<len-1; i++) {
            for (j=len-1; j>0; j--) {
                if(isPalindromic(s.substr(i, j-i+1))) {
                    if(max < (j-i)) {
                        ret = s.substr(i, j-i+1);
                        max = j-i;
                    }
                    break;
                }
                if(max!=0 && max>(j-i)) {
                    break;
                }
            }
            if(max!=0 && max>(len-1-i)) {
                break;
            }
        }
        return ret;
    }
};
```
谨记大神的说法，想不出高级解法就先用暴力法，上面就是暴力法，不过超时了？题目没有说时间啊、、、不懂了，但是毕竟也是自己写的，所以还是贴出来吧，优秀解法如下：
动态规划法（Time:O(n^2)）
```java
class Solution {
    public String longestPalindrome(String s) {
        int n = s.length();
        boolean[][] dp = new boolean[n][n];
        int start = 0, maxLen = 0;
        
        for(int i=0; i<n;i++){
            dp[i][i] = true;
            if(maxLen < 1) { start = i; maxLen = 1;}
            if(i<n-1 && s.charAt(i) == s.charAt(i+1)) {
                dp[i][i+1] = true;
                start = i;
                maxLen = 2;
            }
        }
        
        for(int numChars=3; numChars<=n;numChars++){
            for(int j=0; j<n-numChars+1; j++){
                int pos = j+numChars-1;
                if(dp[j+1][pos-1] && s.charAt(j) == s.charAt(pos)) {
                    dp[j][pos] = true;
                    start = j;
                    maxLen = numChars;
                }
            }
        }
        return s.substring(start, start+maxLen);
    }
    }
```
Best solution:

```java
class Solution {
    
     private static class LongestPalindromeNode {

        String longestPalindrome;
        int length;

        LongestPalindromeNode() {
            this.longestPalindrome = "";
            this.length = 0;
        }
    }

    private static LongestPalindromeNode helper(String word, int left, int right, LongestPalindromeNode longestPalindromeNode) {

        int curr_max_length = 0;

        while(left >= 0 && right < word.length() && word.charAt(left) == word.charAt(right)) {
            left--;
            right++;
        }
        curr_max_length = right - left -1;
        if(longestPalindromeNode.length < curr_max_length) {
            longestPalindromeNode.length = curr_max_length;
            longestPalindromeNode.longestPalindrome = word.substring(left+1, right);
        }

        return longestPalindromeNode;
    }

    public String longestPalindrome(String word) {
        
        int length = word.length();
        if(length == 0 || length == 1)
            return word;

        LongestPalindromeNode longestPalindromeNode = new LongestPalindromeNode();

        for(int idx = 0; idx < word.length(); idx++) {
            int remaining_length = length - idx;
            int max_palindrome_length_can_be_formed = (remaining_length) + (remaining_length-1);
            if(max_palindrome_length_can_be_formed < longestPalindromeNode.length)
                break;
            
            longestPalindromeNode = helper(word, idx, idx, longestPalindromeNode);
            longestPalindromeNode = helper(word, idx, idx+1, longestPalindromeNode);

        }

        return longestPalindromeNode.longestPalindrome;
        
    }
    }
```


