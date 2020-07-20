---
title: "Leetcode 138: Copy List with Random Pointer"
date: 2020-07-20T23:16:27+08:00
draft: false
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接：<https://leetcode.com/problems/copy-list-with-random-pointer/>

这道题要求对一个链表进行深拷贝，其中每个节点有一个额外的随机指针，随机指向链表中某一个节点或者NULL。之前在春招找实习的时候，被问了N次实现Javascript中的深拷贝，所以对这个也比较熟悉。方法就是通过递归来进行节点的复制，坑点在于链表中可能存在循环引用，因此单纯的递归会陷入死循环。解决方法是用一个Hashmap来保存从原链表的节点到新链表的节点的映射，如果我们现在要复制的节点已经在Hashmap中作为key存在了，那么就直接返回它对应的value，而不再创建新节点。代码实现如下：

```cpp
// Author: Zhiyang Li
// Date: 2020.07.20

/*
// Definition for a Node.
class Node {
public:
    int val;
    Node* next;
    Node* random;
    
    Node(int _val) {
        val = _val;
        next = NULL;
        random = NULL;
    }
};
*/

class Solution {
private:
    Node * copy(Node *p, unordered_map<Node *, Node *> &map) {
        if (p == nullptr) {
            return nullptr;
        }
        
        if (map.count(p) > 0) {
            return map[p];
        }
        
        Node *cur = new Node(p->val);
        map[p] = cur;
        
        cur->next = copy(p->next, map);
        cur->random = copy(p->random, map);
        
        return cur;
    }
public:
    Node* copyRandomList(Node* head) {
        unordered_map<Node *, Node *> map;
        
        return copy(head, map);
    }
};
```

该算法的空间复杂度为$O(n)$，因为Hashmap中保存了$n$个节点的信息，时间复杂度也是$O(n)$，因为每个指针被访问了一次，而该链表中有$2n$个指针。

