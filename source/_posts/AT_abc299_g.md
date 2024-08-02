---
title: ABC299G 题解
date: 2023-04-25 00:00:00
# categories: 全新体验
tags:
- 题解
- atcoder
sticky: 1
#cover
comments: true
toc: true
author: DaydreamWarrior
---

“まだ動くまだ進む 物語の上を泳げ” —— 《スイマー》

## 分析

维护一个栈，每次加入一个值。

如果栈顶在之后也会出现并且比加入的值大就弹出。

这样使得每个值尽可能放在前面。

粗略的证明一下。假设有两个序列 $A,B$，$A$ 的字典序小于 $B$，且 $A$ 是字典序最小的。

$A$ 的第一个与 $B$ 不同的位置为 $x$，$A_x$ 在 $B$ 中出现的位置 $y$ 一定在 $x$ 之后。

$B_y$ 能移动到 $x$，在处理 $B$ 的时候 $B_y$ 会被移动到 $x$，所以不会找到 $B$ 序列。

那么找到的序列是字典序最小的。

不过一个排列的置换字典序最小不一定代表这个排列的字典序最小。

## 代码
```cpp
const int N = 200005;
int a[N],pos[N];
int stk[N],top;
bool b[N];
int n,m;

int main(){
    cin >> n >> m;
    for(int k=1;k<=n;k++){
        cin >> a[k];
        pos[a[k]] = k;
    }
    for(int k=1;k<=n;k++){
        if(b[a[k]])
            continue;
        while(top&&stk[top]>a[k]&&pos[stk[top]]>k){
            b[stk[top]] = false;
            top--;
        }
        stk[++top] = a[k];
        b[a[k]] = true;
    }
    for(int k=1;k<=m;k++)
        cout << stk[k] << " ";
    return 0;
}
```