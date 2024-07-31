---
title: ABC241G 题解
date: 2023-05-24 00:00:00
# categories: 全新体验
tags:
- 题解笔记
- 网络流
- 最大流
sticky: 1
#cover:
comments: true
toc: true
author: DaydreamWarrior
---

## 分析

枚举第一名，假设以后的比赛该名玩家都获胜，然后判定是否合法。

数据范围很网络流，考虑最大流，流量代表得分，然后是建图：
- 源点和每场比赛连容量为 $1$ 的边
- 比赛如果有胜负就向胜者连边，否则就向参与比赛的两个人连边
- 设第一名的得分为 $w$，第一名向汇点连容量为 $w$ 的边，其余的人得分不能超过第一名，所以其余的人向汇点连容量为 $w-1$ 的边

由于总场次为 $\frac{n(n+1)}{2}$，如果最大流为 $\frac{n(n+1)}{2}$ 就说明每场比赛都分出了胜负，这种情况合法。

## 代码

```cpp
const int N = 3000,M = 10000;
int h[N],ne[M],e[M],w[M],idx;
int win[N][N];
int n,m,S,T;

void add(int a,int b,int c){
    w[idx] = c,e[idx] = b,ne[idx] = h[a],h[a] = idx++;
    w[idx] = 0,e[idx] = a,ne[idx] = h[b],h[b] = idx++;
}

namespace Dinic{
    int q[N],d[N],cur[N];

    bool bfs(){
        int hh = 0,tt = -1;
        memset(d,-1,sizeof(d));
        d[S] = 0,cur[S] = h[S];
        q[++tt] = S;
        while(hh<=tt){
            int u = q[hh++];
            for(int k=h[u];~k;k=ne[k]){
                int v = e[k];
                if(d[v]==-1&&w[k]){
                    d[v] = d[u]+1;
                    cur[v] = h[v];
                    if(v==T)
                        return true;
                    q[++tt] = v;
                }
            }
        }
        return false;
    }

    int find(int u,int lim){
        if(u==T)
            return lim;
        int flow = 0;
        for(int k=cur[u];~k&&flow<lim;k=ne[k]){
            cur[u] = k;
            int v = e[k];
            if(d[v]==d[u]+1&&w[k]){
                int t = find(v,min(w[k],lim-flow));
                if(!t)
                    d[v] = -1;
                w[k] -= t;
                w[k^1] += t;
                flow += t;
            }
        }
        return flow;
    }

    int dinic(){
        int r = 0,flow;
        while(bfs())
            if((flow=find(S,1e9)))
                r += flow;
        return r;
    }
}

signed main(){
    n = in(),m = in();
    for(int k=1;k<=m;k++){
        int a = in(),b = in();
        win[a][b] = 1;
        win[b][a] = 2;
    }
    S = 0,T = n+n*(n-1)/2+1;
    for(int k=1;k<=n;k++){
        memset(h,-1,sizeof(h));
        idx = 0;
        int cnt = n,wnt = 0;
        for(int j=1;j<=n;j++)
            for(int i=j+1;i<=n;i++){
                add(S,++cnt,1);
                if(win[i][j]){
                    if(win[j][i]==1){
                        if(j==k)
                            wnt++;
                        add(S,j,1);
                    }
                    else{
                        if(i==k)
                            wnt++;
                        add(S,i,1);
                    }
                }
                else{
                    if(j==k||i==k){
                        add(cnt,k,1);
                        wnt++;
                    }
                    else{
                        add(cnt,j,1);
                        add(cnt,i,1);
                    }
                }
            }
        for(int j=1;j<=n;j++)
            if(j==k)
                add(j,T,wnt);
            else
                add(j,T,wnt-1);
        if(Dinic::dinic()==n*(n-1)/2)
            out(k,' ');
    }
    return 0;
}
```