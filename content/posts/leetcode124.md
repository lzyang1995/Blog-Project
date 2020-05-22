---
title: "LeetCode 124: Binary Tree Maximum Path Sum"
date: 2020-05-22T20:56:40+08:00
draft: false
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接：<https://leetcode.com/problems/binary-tree-maximum-path-sum/>

这道题是要在一棵二叉树中寻找一条路径，使得该路径上的节点的和（Path Sum）最大。这里的路径是图论中的路径，即可以从任意一点开始到任意一点结束，而不是通常见到的从根节点开始的路径。

在二叉树里面，Path Sum最大的路径必然包含一个高度最高的节点（即最接近根节点的那个节点），该路径以该节点为中心，分别向该节点的左子树和右子树延伸。因此，我们只需要对于树中的每个节点，都计算一下以该节点为最高节点的Path Sum的最大值，这些最大值中的最大值就是最终的结果（即该二叉树的最大Path Sum）。为此，我们可以定义一个递归函数recur(node)，去计算以node为最高节点并且只向一边延伸（左子树或者右子树）的Path Sum的最大值，这个值可以根据recur(node.left)和recur(node.right)来计算。与此同时，以node为最高节点的Path Sum的最大值（不限制延伸的方向）也可以根据recur(node.left)和recur(node.right)计算出来。具体代码如下：

```js
// Author: Zhiyang Li
// Date: 2020.05.22
/**
 * Definition for a binary tree node.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */

// 返回以node为最高节点，并且只向一边延伸的Path Sum的最大值。
// result用于存放最终结果
function recur(node, result) {
    if (node === null) return 0;
    
    let l1 = recur(node.left, result);
    let r1 = recur(node.right, result);
    
    let cur1 = Math.max(l1, r1, 0);
    
    let cur1l = Math.max(l1, 0);
    let cur1r = Math.max(r1, 0);
    
    // 计算以node为最高节点的Path Sum的最大值，如果大于result.value
    // 就更新result.value
    if (node.val + cur1l + cur1r > result.value) {
        result.value = node.val + cur1l + cur1r;
    }
    
    return node.val + cur1;
}

var maxPathSum = function(root) {
    let result = {value: -Infinity};
    recur(root, result);
    return result.value;
};
```

由于该算法只访问了每个节点1次，因此时间复杂度为$O(n)$。
