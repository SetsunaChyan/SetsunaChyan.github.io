---
title: "CF1198E Rectangle Painting 2 网络流"
categories: [Tutorial]
tags: [Flows]
excerpt: "从棋盘覆盖变形的一道最大流"
---

[传送门](https://codeforces.com/problemset/problem/1198/E)

问题是网格图中有若干个黑点，每次可以选择一个 $w\times h$ 的矩阵消除黑点($w,h$ 任意)，代价是 $\min(w,h)$ ，问消除所有黑点的最小代价。

首先有个观察，我们填某个矩阵显然不会优于用 $1 \times n$ 和 $n \times 1$ 填，而且代价是一样的。

那么问题变成了最少用几个 $1 \times n$ 和 $n \times 1$ 能把棋盘上所有的黑格子覆盖。

这是一个经典问题。

考虑这样一张二分图，左部点是行，右部点是列，边存在当且仅当这行与这列的交点是黑格子，那么这张图的最小点覆盖就是答案了。因为二分图最小点覆盖等于二分图最大匹配，直接跑匹配就行。

等等，坐标范围 $1e9$ 。

由于 $m$ 很小，我们可以把边相同的点并在一块，像离散化那样搞一搞。

也就是说 $x$ 轴上会被分成最多 $2*m-1$ 段，$y$ 轴同理，我们就得到了一个点数 $O(m)$ 的图。

这样的图由于边权不再是 $1$ ，直接匈牙利肯定⑧太行，上最大带权匹配或者直接网络流应该都行。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const ll inf=0x3f3f3f3f3f3f3f3fll;
int n,m,s,t,tot,dis[505],cur[505];
struct edge
{
    int to,cap,rev;
    edge(){}
    edge(int to,int cap,int rev):to(to),cap(cap),rev(rev){}
};
vector<edge> E[505];

inline void add_edge(int x,int y,int f)
{
    E[x].emplace_back(y,f,E[y].size());
    E[y].emplace_back(x,0,E[x].size()-1);
}

int bfs()
{
    for(int i=0;i<=tot;i++) dis[i]=0x3f3f3f3f;
    dis[s]=0;
    queue<int> q;
    q.push(s);
    while(!q.empty())
    {
        int now=q.front();q.pop();
        for(int i=0;i<E[now].size();i++)
        {
            edge &e=E[now][i];
            if(dis[e.to]>dis[now]+1&&e.cap)
            {
                dis[e.to]=dis[now]+1;
                if(e.to==t) return 1;
                q.push(e.to);
            }
        }
    }
    return 0;
}

ll dfs(int now,ll flow)
{
    if(now==t) return flow;
    ll rest=flow,k;
    for(int i=cur[now];i<E[now].size();i++)
    {
        edge &e=E[now][i];
        if(e.cap&&dis[e.to]==dis[now]+1)
        {
            cur[now]=i;
            k=dfs(e.to,min(rest,(long long)e.cap));
            e.cap-=k;
            E[e.to][e.rev].cap+=k;
            rest-=k;
        }
    }
    return flow-rest;
}

ll dinic()
{
    ll ret=0,delta;
    while(bfs())
    {
        for(int i=0;i<=tot;i++) cur[i]=0;
        while(delta=dfs(s,inf)) ret+=delta;
    }
    return ret;
}

int x1[55],x2[55],y1[55],y2[55];
map<int,int> idx,idy;
vector<int> x,y;

int main()
{
    scanf("%d%d",&n,&m);
    if(m==0) return printf("0"),0;
    s=0,t=1,tot=1;
    for(int i=0;i<m;i++)
    {
        scanf("%d%d%d%d",x1+i,y1+i,x2+i,y2+i);
        x.push_back(x1[i]),x.push_back(x2[i]+1);
        y.push_back(y1[i]),y.push_back(y2[i]+1);
    }
    sort(x.begin(),x.end());
    sort(y.begin(),y.end());
    for(int i=0;i<x.size()-1;i++) if(x[i]!=x[i+1])
        for(int j=0;j<y.size()-1;j++) if(y[j]!=y[j+1])
            for(int k=0;k<m;k++)
                if(x[i]>=x1[k]&&x[i+1]-1<=x2[k]&&y[j]>=y1[k]&&y[j+1]-1<=y2[k])
                {
                    if(!idx.count(x[i])) idx[x[i]]=++tot,add_edge(s,tot,x[i+1]-x[i]);
                    if(!idy.count(y[j])) idy[y[j]]=++tot,add_edge(tot,t,y[j+1]-y[j]);
                    //printf("%d~%d  ->  %d~%d  %d\n",x[i],x[i+1]-1,y[j],y[j+1]-1,min(x[i+1]-x[i],y[j+1]-y[j]));
                    add_edge(idx[x[i]],idy[y[j]],min(x[i+1]-x[i],y[j+1]-y[j]));
                    break;
                }
    printf("%lld",dinic());
    return 0;
}
```