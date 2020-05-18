---
title: "LeetCode 105: Construct Binary Tree from Preorder and Inorder Traversal"
date: 2020-05-18T21:21:18+08:00
draft: true
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接：<https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/>

这道题要求根据一棵二叉树的先序遍历（Preorder）和中序遍历（Inorder）的结果重建这棵树。首先，我们知道，先序遍历是按照根节点-左子树-右子树的顺序遍历的，而中序遍历是按照左子树-根节点-右子树的顺序遍历的。所以，一棵树的先序遍历和中序遍历的结果实际上具有以下结构：

{{< style "text-align: center;" >}}
{{< image src="/images/leetcode105/_image1.png" caption="遍历结果的结构" alt="遍历结果的结构" title="遍历结果的结构" >}}
{{< /style >}}

看到这里，我们应该就清楚这道题应该用递归来做了。递归过程中，对于某棵子树的先序和中序遍历结果，先序遍历结果的第一个值（不妨称为x）是该子树的根节点，然后我们在中序遍历结果中去寻找x，x的左边就是该子树的左子树的中序遍历结果，右边就是该子树的右子树的中序遍历结果。知道了左子树的中序遍历结果，也就知道了左子树的节点数，这样再回到先序遍历结果，我们就知道左子树以及右子树的先序遍历结果了。然后对于左子树和右子树分别进行递归即可。具体代码如下：

```cpp
// Author: Zhiyang Li
// Date: 2020.05.18
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
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if (preorder.size() == 0 && inorder.size() == 0) return nullptr;
        
        int rootVal = preorder[0];
        TreeNode *root = new TreeNode(rootVal);
        
        int rootInd;
        for (int i = 0;i < inorder.size();i++) {
            if (inorder[i] == rootVal) {
                rootInd = i;
                break;
            }
        }
        
        vector<int> leftPre(preorder.begin() + 1, preorder.begin() + 1 + rootInd);
        vector<int> leftIn(inorder.begin(), inorder.begin() + rootInd);
        
        vector<int> rightPre(preorder.begin() + 1 + rootInd, preorder.end());
        vector<int> rightIn(inorder.begin() + rootInd + 1, inorder.end());
        
        TreeNode *left = buildTree(leftPre, leftIn);
        TreeNode *right = buildTree(rightPre, rightIn);
        
        root->left = left;
        root->right = right;
        
        return root;
    }
};
```

对于平衡二叉树，该算法的时间复杂度是$O(nlogn)$（可以按照$T(n) = 2T(\frac{n}{2}) + O(n)$计算）。这里我们在每一次递归过程中都需要$O(n)$的时间来寻找根节点在中序遍历结果中的位置。实际上，我们可以在一开始就利用一个Hash Map来存储每个节点在中序遍历结果中的位置，后面递归时只需要在这个Hash Map中查找即可，时间复杂度为$O(1)$，这样就把整个算法的时间复杂度降低到了$O(n)$（可以按照$T(n) = 2T(\frac{n}{2}) + O(1)$计算）。修改后的具体代码如下：

```cpp
// Author: Zhiyang Li
// Date: 2020.05.18
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
private:
    TreeNode * recur(vector<int>& preorder, vector<int>& inorder, unordered_map<int, int> &map, 
                     int preBegin, int preEnd, int inBegin, int inEnd) {
        if (preBegin == preEnd && inBegin == inEnd) return nullptr;
        
        int rootVal = preorder[preBegin];
        TreeNode *root = new TreeNode(rootVal);
        
        int rootInd = map[rootVal];
        int numLeft = rootInd - inBegin;
        
        TreeNode *left = recur(preorder, inorder, map, 
                               preBegin + 1, preBegin + 1 + numLeft, inBegin, rootInd);
        TreeNode *right = recur(preorder, inorder, map,
                                preBegin + 1 + numLeft, preEnd, rootInd + 1, inEnd);
        
        root->left = left;
        root->right = right;
        
        return root;
    }
public:
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        unordered_map<int, int> map;
        for (int i = 0;i < inorder.size();i++) {
            map[inorder[i]] = i;
        }
        
        int n = preorder.size();
        return recur(preorder, inorder, map, 0, n, 0, n);
    }
};
```

论坛中还有人提到说不用Hash Map也可以在$O(n)$时间内实现这个算法（<https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/discuss/34543/Simple-O(n)-without-map>），这个有空再来分析一下。

