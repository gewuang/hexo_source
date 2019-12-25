---
layout: mypost
title: ADD TWO NUM 
tags: 算法
categories: [leetcode]
---

### Title 

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:
```
Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```

### anwser 

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    int up = 0;
    struct ListNode* list1 = l1;
    struct ListNode* list2 = l2;
    struct ListNode* tmp = NULL;
    struct ListNode* ret = NULL;    
    
    while (!(list1 == NULL && list2 == NULL && up == 0)) {
        if(tmp != NULL) {
            tmp->next = (struct ListNode*)malloc(sizeof(struct ListNode));
            tmp = tmp->next;
        }
        else {
            tmp = (struct ListNode*)malloc(sizeof(struct ListNode));
            ret = tmp;
        }
        tmp->val = up;
        tmp->next = NULL;
        if (list1 != NULL) {
            tmp->val += list1->val;
            list1 = list1->next;
        }
        if (list2 != NULL) {
            tmp->val += list2->val;
            list2 = list2->next;
        }
        if (tmp->val > 9) {
            tmp->val = tmp->val%10;
            up = 1;
        }
        else {
            up = 0;
        }
    }
    return ret;
}

```

这次的解题要比上次好了很多，不过前期也错误了几次，因为错误代码没有保留（大家应该也没兴趣）这里就不展示了，下面是这次提交后的成果:

> Runtime: 12 ms, faster than 87.29% of C online submissions for Add Two Numbers.
> Memory Usage: 8.8 MB, less than 89.63% of C online submissions for Add Two Numbers.

哈哈，小小的开心下，来说下思路，本来一开始想着去直接返回l1或者l2，不增添新的空间，但是发现这样做迈不过要检查这两个链表长度的坎，如果要查长度，每个链表轮训一次的话，花费的时间就太长了，于是new了一个新的tmp链表，考虑到要返回这个链表，所以需要把头指针保存下来，所以就在循环的一开始有个判断，找到第一次，既解决了malloc返回对象第一次能正确送达到tmp链表的问题，也解决了ret获取到头节点指针的麻烦。

另外一个麻烦的地方是进位，如果最后一位有进位按照一般逻辑来说要拿出来判断，然后malloc，既然前面省了这么多，怎么能在这一步失败，所以苦思冥想，通过修改循环内部结构和while的条件解决了多一个进位的情况，同时整理代码看起来也简洁了很多。

最后放出标准答案:

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
    ListNode dummyHead = new ListNode(0);
    ListNode p = l1, q = l2, curr = dummyHead;
    int carry = 0;
    while (p != null || q != null) {
        int x = (p != null) ? p.val : 0;
        int y = (q != null) ? q.val : 0;
        int sum = carry + x + y;
        carry = sum / 10;
        curr.next = new ListNode(sum % 10);
        curr = curr.next;
        if (p != null) p = p.next;
        if (q != null) q = q.next;
    }
    if (carry > 0) {
        curr.next = new ListNode(carry);
    }
    return dummyHead.next;
    }
```

### 总结

这道题也是一道简单的算法题，但是有了上一道题的经验，这里对他进行了多次优化，用尽量少的代码和尽量省时间的方式来完成算法，通过结果也是对自己算法之路的一点鼓励了。


