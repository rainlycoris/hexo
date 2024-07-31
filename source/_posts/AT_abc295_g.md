---
title: ABC295G 题解
date: 2023-03-27 00:00:00
# categories: 全新体验
tags:
- 题解
- atcoder
- 并查集
- 强连通
sticky: 1
#cover:
comments: true
toc: true
author: DaydreamWarrior
---

“「あるべき人間の姿へ」 「正しい人間の姿へ」 そう思えばなんだか 人間全てが汚く思えてくるな”—— 《不可解》

## 题意简述

给定一张点数为 $N$ 的有向图，初始 $p_i(1\leq p_i \leq i,1 \leq i < N)$ 连向 $i+1$。

$Q$ 次操作，有两种：
- `1 u v`：$u$ 向 $v$ 连一条有向边，保证最开始时 $v$ 能到达 $u$，$u \ne v$。
- `2 x`：询问 $x$ 能到达的点中编号最小的点。

## 分析

最开始时，$u$ 能到达的所有点都比 $u$ 大。而操作 $1$ 形成了一个强联通分量，走强联通分量内部的点才可能达到更小的点。

一个点 $x$ 能达到的最小的点在 $x$ 所在的强连通分量里，用并查集维护即可。

## 代码
```cpp
const int N = 200005;
int fa[N],p[N];
int n;

int find(int x){
    if(fa[x]==x)
        return fa[x];
    return fa[x]=find(fa[x]);
}

int main(){
    cin >> n;
    for(int k=2;k<=n;k++)
        cin >> p[k];
    for(int k=1;k<=n;k++)
        fa[k] = k;
    int q;
    cin >> q;
    while(q--){
        int t;
        cin >> t;
        if(t==1){
            int u,v;
            cin >> u >> v;
            v = find(v);
            while(find(u)!=v){
                fa[find(u)] = find(p[u]);
                u = p[u];
            }
            fa[find(u)] = v;
        }
        else{
            int x;
            cin >> x;
            cout << find(x) << endl;
        }
    }
    return 0;
}
```

## 闲话

警惕 abc 打普及 G 牌。