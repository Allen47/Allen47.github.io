---
title: Maxinum product of Splitted Binary Tree
date: 2020-03-07 20:50:51
tags: [tree,Java,medium]
categories: 每日刷题
---

## [【LeetCode】1339. 分裂二叉树的最大乘积](https://leetcode-cn.com/problems/maximum-product-of-splitted-binary-tree/)

给你一棵二叉树，它的根为 root 。请你删除 1 条边，使二叉树分裂成两棵子树，且它们子树和的乘积尽可能大。

由于答案可能会很大，请你将结果对 10^9 + 7 取模后再返回。

<!-- more -->

示例 1：

![image1](/images/tr1.png)

> 输入：root = [1,2,3,4,5,6]
> 输出：110
> 解释：删除红色的边，得到 2 棵子树，和分别为 11 和 10 。它们的乘积是 110 （11*10）

示例 2：

![image1](/images/tr2.png)

> 输入：root = [1,null,2,3,4,null,null,5,6]
> 输出：90
> 解释：移除红色的边，得到 2 棵子树，和分别是 15 和 6 。它们的乘积为 90 （15*6）

示例 3：

> 输入：root = [2,3,9,10,7,8,6,5,4,11,1]
> 输出：1025



示例 4：

> 输入：root = [1,1]
> 输出：1



提示：

每棵树最多有 50000 个节点，且至少有 2 个节点。
每个节点的值在 [1, 10000] 之间



---

## 思路

1. 定义一个求和函数求整棵树的和 S 【这里的树随着遍历的过程在不断的改变】；
2. 定义一个遍历的函数，在遍历的过程中求划分后左部分树的和 Sl ，则右边剩余部分的和为 (S - Sl)；用一个全局遍历 ans ，在遍历过程中不断更新 ans = max{ (S - Sl)* Sl ,(S -Sr )*Sr }；
3. return (r -> val + Sl + Sr);

4. 最后答案 ans = ans % (10^9 + 7)。



## Code

```java
class Solution {

    final double Kmod =  Math.pow(10,9)+7 ;
    long s = 0;
    long ans = 0;
    
    public int TreeSum(TreeNode r){  //求以r为根的树上所有节点值之和
        if(r == null)    return 0;
        return r.val + TreeSum(r.left)+TreeSum(r.right);
    }

    public int solve(TreeNode r){ //以r为根节点的树，
        if(r == null)
            return 0;

        int sl = solve(r.left);
        int sr = solve(r.right);
        ans = Math.max(Math.max((s-sl)*sl,(s-sr)*sr),ans);
        return r.val + sl + sr;
    }

    public int maxProduct(TreeNode root) { //主函数
        // long s = 0;
        // long ans = 0;
        this.s = TreeSum(root);
        solve(root);
        return (int)(ans % Kmod );

    }
}
```

时间复杂度：遍历了整棵树，T(n) = O(n)

空间复杂度：保存了整n棵树的信息，S(n) = O(n)



## 难点

我个人第一次看这个代码时没想通的点，为啥要定义两个求和函数，而不是共用一个就行。

后来我想了一下，TreeSum（root）是求出整棵树的节点和后，将结果保存在了 s 中；分裂子树的操作放在了 Solve(sum) 中，而 s 的值一直是不变的，不断更新 ans 的操作也放在 Sovle(root) 中。(ans = { ans , (s-sl)\*sl,(s-sr)*sr )

俺也不知道这样讲说清楚没，等以后技术精进了再重新修改一下吧，各位看官大人勉强看看。

