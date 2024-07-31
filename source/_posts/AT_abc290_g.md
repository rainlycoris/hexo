---
title: ABC290G 题解
date: 2023-02-20 00:00:00
# categories: 全新体验
tags:
- 题解
- atcoder
sticky: 1
#cover:
comments: true
toc: true
author: DaydreamWarrior
---

“空高く舞う鳥へ かさねたハート”—— 《Jump up HIGH!!》

## 题意简述

有一颗深度为 $D$ 的满 $K$ 叉树，你需要剪掉一些边使其存在大小为 $X$ 的联通分量，求最少剪掉的边的条数。

## 分析

考虑选择 __所有__ 大小大于等于 $X$ 的一颗深度为 $d$ 的满 $K$ 叉树，在剪掉后大小仍超过 $X$ 的情况下剪掉所有与根直接相连的边，也就是剪掉若干个深度为 $d-1$ 的满 $K$ 叉树，再递归处理深度 $d-1$ 的满 $K$ 叉树。注意如果 $d \ne D$，在树根与树根父节点的连边也要剪掉。

感性理解一下，考虑为什么是对的。在原树上的任意一个子树都可以看成一棵树，树的根节点往上有一条链。先不考虑链，原问题变成了有一颗满 $K$ 叉树，剪掉一些边使得 __包含根节点__ 的联通分量大小为 $x$，求剪掉的边的最少条数。这个问题贪心地剪若干个最大的子树再递归下去显然是对的。上面的做发选择所有大小大于等于 $X$ 的子树就实现了枚举链的长度，所以原贪心也是对的。

复杂度 $O(D^2)$。因为 $1 \leq \displaystyle\sum_{i=0}^D{K^i} \leq 10^{18}$，所以 $D \leq 60$，复杂度足以通过。

## 代码
```cpp
const int N = 70;
int a[N];
int D,K,X;

signed main(){
    int t;
    cin >> t;
    while(t--){
        cin >> D >> K >> X;
        a[0] = 1;
        for(int k=1;k<=D;k++)
            a[k] = a[k-1]*K+1;
        int ans = 1e18;
        for(int k=0;k<=D;k++){
            if(a[k]>=X){
                int tmpans = !(k==D);
                int x = a[k]-X,j = k-1;
                while(x&&~j){
                    tmpans += x/a[j];
                    x %= a[j];
                    j--;
                }
                ans = min(ans,tmpans);
            }
        }
        cout << ans << endl;
    }
    return 0;
}
```

## 闲话

考场写了个只选择最小的大小大于等于 $X$ 的满 $K$ 叉树，WA 了一发，然后在不知道为啥的情况下写了这个过了。