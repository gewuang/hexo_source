---
layout: mypost
title: 寻找两个有序数组的中位数 
categories: [leetcode]
tags: "算法" 
---

### 题目 

给定两个大小为 m 和 n 的有序数组 nums1 和 nums2。

请你找出这两个有序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。

你可以假设 nums1 和 nums2 不会同时为空。

示例1:
```
nums1 = [1, 3]
nums2 = [2]

则中位数是 2.0
```
示例2:
```
nums1 = [1, 2]
nums2 = [3, 4]

则中位数是 (2 + 3)/2 = 2.5
```

### anwser 

```c
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size){
    double ret = 0; // 返回值
    int x = -1;     // 已去掉x下标
    int y = -1;     // 已去掉y下标
    int m = nums1Size + nums2Size;     // 还需要去掉的数
    int kx = 0;     // x移动后的位置
    int ky = 0;     // y移动后的位置
    int km = 0;      // 需要移动的位数
    int a = 19999999;
    int b = 0;
    
    if(m != 1) {
        m /= 2;
    }
    km = m;
    while(km != 0 && x != nums1Size-1 && y != nums2Size-1) {
        km = (km == 1) ? 1:(km/2);
        kx = (x+km)>(nums1Size-1) ? (nums1Size-1):(x+km);
        ky = (y+km)>(nums2Size-1) ? (nums2Size-1):(y+km);
        if(nums1[kx] > nums2[ky]) {
            m = m - ky + y;
            km = m;
            y = ky;
            a = nums2[y];
        }
        else {
            m = m - kx + x;
            km = m;
            x = kx;
            a = nums1[x];
        }
    }
    if(x != nums1Size-1) {
        kx = (x+1+m)>(nums1Size-1) ? (nums1Size-1):(x+1+m);
    }
    if(y != nums2Size-1) {
        ky = (y+1+m)>(nums2Size-1) ? (nums2Size-1):(y+1+m);
    }
    b = (nums1Size + nums2Size) % 2;
    
    if(x == nums1Size-1) {
        //x已经走到头了，只有y了
        if(b) {
            ret = nums2[ky];
        }
        else {
            if (a == 19999999 || m > 0) {
                a = nums2[ky-1];
            }
            ret = (double)(nums2[ky]+a)/2;
        }
    }
    else if(y == nums2Size-1) {
        if(b) {
            ret = nums1[kx];
        }
        else {
            if (a == 19999999 || m > 0) {
                a = nums1[kx-1];
            }
            ret = (double)(nums1[kx]+a)/2;
        }
    }
    else {
        // 两者都有
        ret = nums1[kx] > nums2[ky] ? nums2[ky] : nums1[kx];
        // 为偶数时，取x/y中较大的和kx/ky中较小的相加/2
        if(b == 0){
           ret = (double)(ret+a)/2;
        }
    }
    
    return ret;
}
```
没有力气解读了，到现在最难的一个算法了，处理各种边界问题到心态爆炸，而且还是在没有解决所以去参考了解法自己实现的情况下，具体解法采用的是最小k值法，详细算法请参考下列链接，我就不赘述了:[median-of-two-sorted-arrays](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/solution/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-w-2/)
# 总结

即使知道了这么巧妙的解法，实现起来也花了好久好久，错了很多遍，不过自己最终挺了过来，坚持完成了，超过了88%的c语言提交者，虽然是因为算法优秀，但是毕竟是自己实现的，也希望自己后面遇到困难坚持下去，不放弃，总会成功的。

