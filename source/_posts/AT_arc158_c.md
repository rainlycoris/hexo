---
title: ARC158C 题解
date: 2023-03-13 00:00:00
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

“かけがえのない日々を ここで 積みかさねて ひとつひとつ いまさらわかった”—— 《想いよひとつになれ》

## 题意简述

设 $f(x)$ 为 $x$ 的数字和。例如 $f(158)=1+5+8=14$。

给定一个长度为 $N$ 的正整数序列 $A$，求 $\sum_{i=1}^{N}\sum_{j=1}^{N}f(A_i+A_j)$。

## 分析

设 $g(a,b)$ 为 $a+b$ 进位的次数，则 $f(a+b)=f(a)+f(b)-9 \times g(a,b)$，其中 $\sum_{i=1}^{N}\sum_{j=1}^{N}f(A_i)+f(A_j)=2 n \times \sum_{i=1}^{N} f(A_i)$。

考虑如何计算 $\sum_{i=1}^{N}\sum_{j=1}^{N}g(A_i,A_j)$。对于两个数 $x,y$，如果 $x+y$ 在第 $d$ 位上有进位，当且仅当 $x \bmod 10^d+y \bmod 10^d \geq 10^{d+1}$。将所有 $A_i \bmod 10^d$ 排序，枚举 $d$ 和 $A_j$ 去二分 $A_i \bmod 10^d$ 中大于 $10^{d+1}-A_j \bmod 10^d$ 的值的个数即可。


## 代码
```cpp
const int N = 200005,M = 17;
int a[M][N];
int n;

signed main(){
    cin >> n;
    int ans = 0;
    for(int k=1;k<=n;k++){
        int c;
        cin >> c;
        int d = 10;
        for(int j=1;j<M;j++){
            a[j][k] = c%d;
            d *= 10;
        }
        while(c){
            ans += c%10;
            c /= 10;
        }
    }
    ans *= 2*n;
    for(int k=1;k<M;k++)
        sort(a[k]+1,a[k]+1+n);
    int d = 10;
    int sum = 0;
    for(int k=1;k<M;k++){
        for(int j=1;j<=n;j++){
            int w = a[k]+n+1-lower_bound(a[k]+1,a[k]+1+n,d-a[k][j]);
            sum += w;
        }
        d *= 10;
    }
    cout << ans-sum*9;
    return 0;
}
```