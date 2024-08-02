---
title: ABC279F 并查集题解
date: 2023-03-20 00:00:00
# categories: 全新体验
tags:
- 题解
- atcoder
- 并查集
sticky: 1
#cover
comments: true
toc: true
author: DaydreamWarrior
---

“我仍然在无人问津的阴雨霉湿之地”   —— 《世末歌者》

## 题意

高橋君有 $N$ 瓶糖水，青木君有 $M$ 瓶糖水。

高橋君的第 $i$ 瓶糖水有 $A_i$ 份糖 $B_i$ 份水。

青木君的第 $i$ 瓶糖水有 $C_i$ 份糖 $D_i$ 份水。

将两人的糖水各选一瓶混合有 $NM$ 种可能，求其中浓度第 $k$ 大的糖水浓度是多少。

有 $x$ 份糖和 $y$ 份水的糖水浓度是 $\dfrac{100x}{x+y}\%$。

## 分析

二分浓度 $c$ 后，我们只需要得到混合后浓度大于等于 $c$ 的个数。

有 $a$ 份糖 $b$ 份水的糖水，再加 $(a+b)c-a$ 份糖就能变成浓度 $c$，也可能是减掉糖。

令 $(a+b)c-a$ 为 $s$，$s$ 的正负可以判断浓度和 $c$ 的关系。

那么两瓶糖水 $x,y$ 混合后，判断 $s_x+s_y$ 的正负即可。

因为 $(A_x+B_x+C_y+D_y)c-A_x-C_y=(A_x+B_x)c-A_x+(C_y+D_y)c-C_y=s_x+s_y$，所以 $s$ 是可加的。

二分浓度，将青木君糖水的 $s$ 排序，枚举高橋君的糖水，二分计算混合后浓度大于等于 $c$ 的个数。

复杂度 $O(n \log n \log v)$。

## 代码
```cpp
const int N = 50005;
const double eps = 1e-12;
vector<pair<int,int>> a,b;
int n,m,K;

inline int check(double c){
    vector<double> w1,w2;
    for(auto [x,y]:a)
        w1.push_back(x-(x+y)*c);
    for(auto [x,y]:b)
        w2.push_back(x-(x+y)*c);
    sort(w2.begin(),w2.end());
    int ans = 0;
    for(auto c:w1)
        ans += lower_bound(w2.begin(),w2.end(),-c+eps)-w2.begin();
    return ans;
}

signed main(){
    cin >> n >> m >> K;
    a.resize(n);
    b.resize(m);
    for(auto &[x,y]:a)
        cin >> x >> y;
    for(auto &[x,y]:b)
        cin >> x >> y;
    double l = 0,r = 1;
    K = n*m-K+1;
    while(abs(r-l)>eps){
        double mid = (l+r)/2;
        if(check(mid)>=K)
            r = mid;
        else
            l = mid;
    }
    printf("%.10lf",r*100);
    return 0;
}
```