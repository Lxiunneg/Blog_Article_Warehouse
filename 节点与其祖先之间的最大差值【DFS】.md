---
title: 节点与其祖先之间的最大差值【DFS】
top: false
mathjax: true
date: 2023-04-18 22:58:32
tags:
categories:
---
# 节点与其祖先之间的最大差值


<!--more-->


## 题目描述

给定二叉树的根节点 root，找出存在于 不同 节点 A 和 B 之间的最大值 V，其中 V = `|A.val - B.val|`，且 A 是 B 的祖先。  

（如果 A 的任何子节点之一为 B，或者 A 的任何子节点是 B 的祖先，那么我们认为 A 是 B 的祖先）
 
![](https://assets.leetcode.com/uploads/2020/11/09/tmp-tree.jpg)  

## 题目分析 

## AC代码

```cpp
class Solution {
public:

    void DFS(TreeNode* node,int cmax,int cmin,int& ans){
        if(node == nullptr) return;
        else{
            ans = max(ans, max(abs(node->val - cmax), abs(node->val - cmin)));
            cmax = max(cmax,node->val);
            cmin = min(cmin,node->val);
            DFS(node->left,cmax,cmin,ans);
            DFS(node->right,cmax,cmin,ans);
        }
    }

    int maxAncestorDiff(TreeNode* root) {
        int ans = 0;
        DFS(root,root->val,root->val,ans);
        return ans;
    }
};
```

## 思考
DFS更适用于树和图
