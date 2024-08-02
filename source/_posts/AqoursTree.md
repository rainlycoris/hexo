---
title: 「Aqours Tree 」无旋 2-3 leafy 平衡树
date: 2024-07-05 13:23:00
# categories: 全新体验
tags:
- 平衡树
sticky: 1
#cover:
comments: true
toc: true
author: DaydreamWarrior
---

## 概述

Aqours Tree 是我发明的无旋 2-3 leafy 平衡树。非叶子结点只会有 $2$ 个或 $3$ 个儿子。 

每个叶子的深度相同，树高严格 $\leq \log_2$ 且 $\geq \log_3$，重量平衡。

支持单次 $O(\log n)$ 的 split 和单次 $O(\log a-\log b)$ 的 merge，支持可持久化，支持带势能的值域有交合并。

## 合并

要合并 $x$ 和 $y$ 这两棵 Aqours 树，$x$ 的树高小于 $y$。

首先在 $y$ 上找到 __高度__ 和 $x$ 一样的点 $u$。
- $fa_u$ 有两个儿子，$x$ 直接成为 $y$ 的儿子
- $fa_u$ 有三个儿子，把最左边的儿子和 $fa_u$ 断掉，新建一个结点连接最左边的儿子和 $x$。这产生了一个新的 Aqours 树，树高为 $x+1$。$u=fa_u$ 继续操作直到 $fa_u$ 有两个儿子。如果直到根都不是二叉的，就产生了一个新的树高为 $y+1$ 的 Aqours 树，这就是合并的结果。

否则 $y$ 就是合并的结果。

对于 $x$ 的树高大于 $y$ 同理。

不难证明复杂度为 $O(\log n)$。

## 分裂

类似于 WBLT，大体来说就是把 split 路径上所有点拆开，按照分裂的标准分成两组组内合并。

由于合并的复杂度为 $O(\log a-\log b)$，以及 split 下来的点树高一定是有序的，所以复杂度为 $O(\log n)$。当然树高有时会 $+1$，但是 $+1$ 之后根是二叉的，所以相邻两次只会有一次 $+1$。相同高度的也只会出现最多两次，所以 $+1$ 永远追不上树高的增长。

## 代码
第一版的实现和这版的效率差距高达 4 倍，我仍在改进。

update 7.27：我不打算对这个版本做任何实现优化了，虽然这个版本的效率仍是目前最快的。
```cpp
class Aqours{
    public:
        class{
            private:
                int s[3];
            public:
                int v,d,siz;
                void init(int x,int y){s[0] = x,s[1] = y,s[2] = 0;}
                int operator [] (const int &x){return s[x];}
                int back(){return s[2]?s[2]:s[1];}
                bool size3(){return s[2];}
                void push_front(int v){s[2] = s[1];s[1] = s[0];s[0] = v;}
                void push_back(int v){s[2] = v;}
                void pop_front(){s[0] = s[1];s[1] = s[2];s[2] = 0;}
                void pop_back(){s[2] = 0;}
        }tr[N];
        int leaf[N],pos;

        void update(int u){
            if(tr[u][0]<0){
                tr[u].siz = tr[u].size3()?3:2;
                tr[u].d = 2;
                tr[u].v = leaf[-tr[u].back()];
                return;
            }
            tr[u].d = tr[tr[u][0]].d+1;
            tr[u].v = tr[tr[u].back()].v;
            tr[u].siz = tr[tr[u][0]].siz+tr[tr[u][1]].siz;
            if(tr[u].size3())
                tr[u].siz += tr[tr[u][2]].siz;
        }

        basic_string<int> cl;
        int idx;

        void recycle(int u){if(u)cl.push_back(u);}
        int allocate(){
            if(cl.empty())
                return ++idx;
            int u = cl.back();
            cl.pop_back();
            return u;
        }

        int create(int x,int y){
            int u = allocate();
            tr[u].d = tr[u].v = 0;
            tr[u].init(x,y);
            update(u);
            return u;
        }

        int create(int v){
            leaf[++pos] = v;
            return -pos;
        }

        int stk[20],tp;
    public:
        int rt;
        int merge(int x,int y){
            if(!x||!y)
                return x|y;
            if(x<0&&y<0)
                return create(x,y);
            tp = 0;
            if(x<0){
                while(y>0){
                    stk[tp++] = y;
                    y = tr[y][0];
                }
                goto yoshiko;
            }
            if(y<0){
                while(x>0){
                    stk[tp++] = x;
                    x = tr[x].back();
                }
                goto riko;
            }
            if(tr[x].d<=tr[y].d){
                while(tr[x].d<tr[y].d){
                    stk[tp++] = y;
                    y = tr[y][0];
                }
                yoshiko:
                while(tp){
                    y = stk[--tp];
                    if(!tr[y].size3()){
                        tr[y].push_front(x);
                        update(y);

                        while(tp)
                            update(stk[--tp]);
                        return stk[0];
                    }
                    x = create(x,tr[y][0]);
                    tr[y].pop_front();
                    update(y);
                }
                return create(x,y);
            }
            while(tr[x].d>tr[y].d){
                stk[tp++] = x;
                x = tr[x].back();
            }
            riko:
            while(tp){
                x = stk[--tp];
                if(!tr[x].size3()){
                    tr[x].push_back(y);
                    update(x);

                    while(tp)
                        update(stk[--tp]);
                    return stk[0];
                }
                y = create(tr[x][2],y);
                tr[x].pop_back();
                update(x);
            }
            return create(x,y);
        }

        void split(int u,int v,int &x,int &y){
            if(u<=0){
                if(leaf[-u]<=v)
                    x = u,y = 0;
                else
                    x = 0,y = u;
                return;
            }
            if(tr[u].d==2){
                if(leaf[-tr[u][0]]>v)
                    x = 0,y = u;
                else if(leaf[-tr[u][1]]>v){
                    x = tr[u][0],y = merge(tr[u][1],tr[u][2]);
                    recycle(u);
                }
                else if(leaf[-tr[u][2]]>v){
                    x = merge(tr[u][0],tr[u][1]),y = tr[u][2];
                    recycle(u);
                }
                else
                    x = u,y = 0;
                return;
            }
            if(tr[tr[u][0]].v>v){
                split(tr[u][0],v,x,y);
                y = merge(merge(y,tr[u][1]),tr[u][2]);
            }
            else if(tr[tr[u][1]].v>v){
                split(tr[u][1],v,x,y);
                x = merge(tr[u][0],x);
                y = merge(y,tr[u][2]);
            }
            else{
                split(tr[u][2],v,x,y);
                x = merge(merge(tr[u][0],tr[u][1]),x);
            }
            recycle(u);
        }

        void split_rk(int u,int v,int &x,int &y){
            if(u<=0){
                if(v)
                    x = u,y = 0;
                else
                    x = 0,y = u;
                return;
            }
            if(tr[u].d==2){
                if(!v)
                    x = 0,y = u;
                else if(v==1){
                    x = tr[u][0],y = merge(tr[u][1],tr[u][2]);
                    recycle(u);
                }
                else if(v==2){
                    x = merge(tr[u][0],tr[u][1]),y = tr[u][2];
                    recycle(u);
                }
                else
                    x = u,y = 0;
                return;
            }
            if(tr[tr[u][0]].siz>v){
                split_rk(tr[u][0],v,x,y);
                y = merge(y,merge(tr[u][1],tr[u][2]));
            }
            else if(tr[tr[u][0]].siz+tr[tr[u][1]].siz>v){
                split_rk(tr[u][1],v-tr[tr[u][0]].siz,x,y);
                x = merge(tr[u][0],x);
                y = merge(y,tr[u][2]);
            }
            else{
                split_rk(tr[u][2],v-tr[tr[u][0]].siz-tr[tr[u][1]].siz,x,y);
                x = merge(merge(tr[u][0],tr[u][1]),x);
            }
            recycle(u);
        }

        void insert(int v){
            int x,y;
            split(rt,v,x,y);
            rt = merge(merge(x,create(v)),y);
        }

        void erase(int v){
            int x,y,z;
            split(rt,v-1,x,y);
            split_rk(y,1,y,z);
            rt = merge(x,z);
        }

        int rank(int v){
            int x,y;
            split(rt,v-1,x,y);
            int ans;
            if(x==0)
                ans = 1;
            else if(x<0)
                ans = 2;
            else
                ans = tr[x].siz+1;
            rt = merge(x,y);
            return ans;
        }

        int at(int v){
            int x,y;
            split_rk(rt,v-1,x,y);
            int u = y;
            while(u>0)
                u = tr[u][0];
            rt = merge(x,y);
            return leaf[-u];
        }

        int prev(int v){
            int x,y;
            split(rt,v-1,x,y);
            int u = x;
            while(u>0)
                u = tr[u].back();
            rt = merge(x,y);
            return leaf[-u];
        }

        int next(int v){
            int x,y;
            split(rt,v,x,y);
            int u = y;
            while(u>0)
                u = tr[u][0];
            rt = merge(x,y);
            return leaf[-u];
        }
}tr;
```
[这份](https://www.luogu.com.cn/paste/4g9kb92e)是更加精细的实现，但是优化已经不大了。

## 改进

### 二度化

在对三叉结点操作的时候不可避免的要进行 2 次 merge，但是没被操作的两个叉是不需要重新 merge 的，考虑「缓存」这个值。

二度化之后就起到了储存这两个叉信息的效果，期望 merge 次数变成了 1.5，理论做到和 fhqtreap 一样的 merge 次数。

具体的，对于三叉的左边两个叉或者右边，新建一个虚点连接这两个叉，然后原来的三叉点连接这个虚点。

当然也可以降低代码复杂度，因为现在就是棵正常的二叉树了。

这也增加了一般查询的复杂度，但是主要的瓶颈在 split/merge。

### 左偏

强制虚点在左儿子，可以降低代码复杂度，同时太不改变理论的 merge 次数.

同时不需要维护实点虚点了，每个点高度等于右儿子 $+1$。如果左儿子高度等于自己，自己就是三度点，同时左儿子是虚点。

### 代码

更新于 7.27

```cpp

class Aqours{
    public:
        struct{int l,r,d,s,v;} tr[2*N];
        int stk[39],tp;
        
        int idx = N,trash[N],pos;
        void recycle(int u){trash[pos++]=u;}
        int allocate(){
            if(pos)
                return trash[--pos];
            return ++idx;
        }
        int create(int x,int y){
            int u = allocate();
            tr[u].l = x,tr[u].r = y;
            tr[u].d = tr[tr[u].r].d+1;
            pushup(u);
            return u;
        }
        int cnt;
        int create(int v){
            int u = ++cnt;
            tr[u].l = tr[u].r = 0;
            tr[u].d = tr[u].s = 1;
            tr[u].v = v;
            return u;
        }

        void pushup(int u){
            merge_cnt++;
            tr[u].v = tr[tr[u].r].v;
            tr[u].s = tr[tr[u].l].s+tr[tr[u].r].s;
        }
    public:
        int rt;

        int merge(int x,int y){
            if(!x||!y)
                return x|y;
            tp = 0;
            if(tr[x].d<tr[y].d){
                while(tr[x].d<tr[y].d){
                    stk[++tp] = y;
                    y = tr[y].l;
                }
                while(tp){
                    y = stk[tp--];
                    if(tr[y].d==tr[stk[tp]].d){
                        pushup(y);
                        continue;
                    }
                    if(tr[tr[y].l].d!=tr[y].d){
                        tr[y].l = create(x,tr[y].l);
                        pushup(y);
                        goto yohane;
                    }
                    int u = tr[y].l;
                    tr[y].l = tr[u].r;
                    tr[u].r = tr[u].l;
                    tr[u].l = x;
                    pushup(y);
                    pushup(u);
                    x = u;
                }
                return create(x,y);
            }
            while(tr[x].d>tr[y].d){
                stk[++tp] = x;
                x = tr[x].r;
            }
            while(tp){
                x = stk[tp--];
                if(tr[tr[x].l].d!=tr[x].d){
                    tr[x].l = create(tr[x].l,tr[x].r);
                    tr[x].r = y;
                    pushup(x);
                    goto yohane;
                }
                y = create(tr[x].r,y);
                recycle(tr[x].l);
                tr[x] = tr[tr[x].l];
            }
            return create(x,y);

            yohane:
            while(tp)
                pushup(stk[tp--]);
            return stk[1];
        }

        void split(int u,int v,int &x,int &y){
            if(tr[u].v<=v){
                x = u,y = 0;
                return;
            }
            if(!tr[u].l){
                x = 0,y = u;
                return;
            }
            if(v<tr[tr[u].l].v){
                split(tr[u].l,v,x,y);
                y = merge(y,tr[u].r);
            }
            else{
                split(tr[u].r,v,x,y);
                x = merge(tr[u].l,x);
            }
            recycle(u);
        }
}tr;
```

## 致谢 & 杂谈

这个东西效率并不优秀，虽然我还在改进吧，但是没有任何的希望。

我目前在尝试找到一种结点编号方式，来优化缓存性能，因为其实瓶颈完全在缓存。

效率上，fhqtreap << Aqours Tree <= WBLT，虽然 Aqours Tree 和 WBLT 的 pushup 远少于 fhqtreap，更不说一些情况下 nodely 的等效 pushup 次数还要 $\times 2$，而差距应该是来自缓存。

我目前其实没有做到最小化 pushup 次数。

---

感谢 _ChiFAN_ Kevin090228 对证明和实现提供的帮助。

也感谢 realskc 和 lxl 的审阅。

--- 

我之前就有个梦想啊，发明一个算法，然后以 Aqours 命名。

水团每个人的名字都要作为算法，永远存在下去。

---

更严谨的证明 [_ChiFAN_《Aqours 树 —— wblt 的替代品》](https://www.luogu.com/article/ob28wvo6)。