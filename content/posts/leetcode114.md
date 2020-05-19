---
title: "LeetCode 114: Flatten Binary Tree to Linked List"
date: 2020-05-19T21:43:40+08:00
draft: false
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接：<https://leetcode.com/problems/flatten-binary-tree-to-linked-list/>

这道题的题意是要实现一个flatten函数，把一棵二叉树按照先序遍历的顺序变成一个单链表。看到二叉树就想到递归，这道题也是可以通过递归来做的。我们要flatten一棵树，实际上就是把它的左子树和右子树都进行flatten操作，然后把左子树flatten的结果接到根节点上，右子树flatten的结果接到左子树flatten之后的最后一个节点上。下面的代码就是基于这个思路：

```cpp
// Author: Zhiyang Li
// Date: 2020.05.19
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    void flatten(TreeNode* root) {
        if (root == nullptr) return;
        
        TreeNode *l = root->left;
        TreeNode *r = root->right;
        
        root->left = nullptr;
        root->right = l;
        
        flatten(l);
        flatten(r);
        
        TreeNode *p = root;
        while (p->right != nullptr) {
            p = p->right;
        }
        
        p->right = r;
    }
};
```

然而，上面的算法效率并不是很高，主要问题在于需要寻找左子树flatten的结果的最后一个节点。这个操作会把左子树的所有节点都访问一次，导致在递归过程中，有很多节点被重复访问了很多次，越往左下角访问的次数越多。

论坛中有人提出了[另一种递归的算法](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/discuss/36977/My-short-post-order-traversal-Java-solution-for-share)。这个算法不是很好理解，不过它的大致思路是按照先序遍历的逆序来构造单链表。先序遍历的访问顺序是根节点-左子树-右子树，因此该算法是按照右子树-左子树-根节点的顺序来进行访问和单链表构造的。这也是一个比较常用的思路。例如，对一棵二叉树进行后序遍历时，如果要求不使用递归，那么一个比较简单的方法是按照后序遍历的逆序，即根节点-右子树-左子树来进行遍历，再把结果倒过来。这个的话用一个栈就可以轻松做到。

论坛中还有另一个[不使用递归的算法](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/discuss/37010/Share-my-simple-NON-recursive-solution-O(1)-space-complexity!)。该算法的思路是，对于当前访问的节点N，找到N的左子树的最右端的节点X，然后把X的right指针指向N的右子树。即相当于把N的右子树进行移动，成为X的右子树。这样操作前后，以N为根的子树的先序遍历的结果是不变的。所以我们可以继续只对N的左子树进行下一步的同样的操作，直到把所有的节点都遍历完。这个思路类似于Morris Traversal，时间复杂度应该是$O(n)$。

## 参考资料
1. <https://leetcode.com/problems/flatten-binary-tree-to-linked-list/discuss/36977/My-short-post-order-traversal-Java-solution-for-share>
2. <https://leetcode.com/problems/flatten-binary-tree-to-linked-list/discuss/37010/Share-my-simple-NON-recursive-solution-O(1)-space-complexity!>
