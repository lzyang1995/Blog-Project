---
title: "LeetCode 128: Longest Consecutive Sequence"
date: 2020-05-26T20:33:56+08:00
draft: false
tags: ["LeetCode", "Algorithm"]
categories: ["算法"]
lightgallery: true

math:
  enable: true
---

原题链接：<https://leetcode.com/problems/longest-consecutive-sequence/>

这道题要求我们在$O(n)$的时间内找出一个无序数组中最长的连续数字的长度。这道题我也没有做出来，这里介绍一下Discuss板块中其他人分享的一个解法（<https://leetcode.com/problems/longest-consecutive-sequence/discuss/41057/Simple-O(n)-with-Explanation-Just-walk-each-streak>）：首先把输入数组nums中的所有数都放入一个集合（set）中，然后再把nums遍历一次。遍历到某个数$n$时，如果$n-1$也在集合中，说明$n$不是某一段连续数字的开头，因此直接跳过这个数访问下一个。如果$n-1$不在集合中，说明$n$是某一段连续数字的开头，那么我们就进一步去判断$n+1, n+2, n+3, \cdots$等等是否在集合中，直到到达某个数$m$不在这个集合中，那么这段连续数字的长度就是$m-n$。我们只需要在遍历过程中保存$m-n$的最大值即可。

关于这个算法的时间复杂度，首先第一步将所有数都放入集合的时间复杂度是$O(n)$。对于后续的遍历，如果$n$不是某一段连续数字的开头，我们只会访问$n$和$n-1$各一次，否则我们会访问从$n$到$m$的所有数各一次，最终每个数最多被访问3次（被作为$n$访问、被作为$n-1$访问、被作为$n$到$m$中的数访问），所以时间复杂度仍然是$O(n)$。因此该算法总的时间复杂度是$O(n)$。

下面是我参考该思路的代码实现：

```cpp
// Author: Zhiyang Li
// Date: 2020.05.26
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        unordered_set<int> set;
        int result = 0;
        for (int i = 0;i < nums.size();i++) {
            set.insert(nums[i]);
        }
        
        for (int i = 0;i < nums.size();i++) {
            int curNum = nums[i];
            if (set.count(curNum - 1) == 0) {
                int tail = curNum + 1;
                while (set.count(tail) != 0) {
                    tail++;
                }
                
                if (tail - curNum > result) {
                    result = tail - curNum;
                }
            }
        }
        
        return result;
    }
};
```

## 参考资料

1. <https://leetcode.com/problems/longest-consecutive-sequence/discuss/41057/Simple-O(n)-with-Explanation-Just-walk-each-streak>
