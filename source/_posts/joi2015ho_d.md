---
title: 网络流常见建模
date: 2023-07-04 00:00:00
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

你眼中倒映的世界 一瞬永远 —— 《梦语》

## 分析

首先答案有单调性，考虑二分。

然后每次留下来的构成一颗三叉树，父亲是儿子的中位数，考虑树形 dp。

上很经典的中位数套路，设二分的值为 $v$，$\geq v$ 的 $g_v$ 为 $1$，$< v$ 的 $g_v$为 $-1$。

如果儿子的值之和 $>0$ 那么父亲就 $\geq v$。

考虑设 $f_u$ 为 $g_u > 0$ 时最小的子树的叶子中 $\geq v$ 的个数。

转移为 $f_u = \sum f_v -\max f_v$，因为如果 $g_u > 0$ 必须有两个及以上的 $g_v=1$，多了没用所以是选择最小的的两个 $f_v$。

如果 $f_{root} \leq$ 序列中 $\leq v$ 的个数就是可行的。

## 代码

```cpp
const int N = 200005;
int d[N],p[N],f[N];
vector<int> g[N];
int root;
int n,m;

void dfs(int u){
    int maxx = 0;
    for(int v:g[u]){
        dfs(v);
        f[u] += f[v];
        maxx = max(maxx,f[v]);
    }
    f[u] -= maxx;
}

bool check(int v){
    for(int k=1;k<=root;k++)
        f[k] = 0;
    for(int k=1;k<=n;k++){
        if(p[k])
            f[k] = d[p[k]]>=v?0:N+1;
        else
            f[k] = 1;
    }
    int cnt = 0;
    for(int k=m+1;k<=n;k++)
        cnt += d[k]>=v;
    dfs(root);
    return f[root]<=cnt;
}

int main(){
    n = in,m = in;
    queue<int> q;
    for(int k=1;k<=n;k++)
        q.push(k);
    root = n;
    while(q.size()>1){
        int a = q.front();
        q.pop();
        int b = q.front();
        q.pop();
        int c = q.front();
        q.pop();
        root++;
        g[root].push_back(a);
        g[root].push_back(b);
        g[root].push_back(c);
        q.push(root);
    }
    for(int k=1;k<=m;k++)
        d[k] = in,p[(int)in] = k;
    for(int k=m+1;k<=n;k++)
        d[k] = in;
    int L = 1,R = 1000000000,ans;
    while(L<=R){
        int mid = (L+R)>>1;
        if(check(mid)){
            ans = mid;
            L = mid+1;
        }
        else
            R = mid-1;
    }
    out(ans,'\n');
    return 0;
}
```