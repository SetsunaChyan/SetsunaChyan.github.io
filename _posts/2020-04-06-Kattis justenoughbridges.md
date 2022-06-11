---
title: "Just Enough Bridges 二分图最大匹配+强连通分量"
categories: [Tutorial]
tags: [Tarjan,Flows]
excerpt: "求多少条边是完美匹配的必选边"
---

[传送门](https://open.kattis.com/problems/justenoughbridges)

一个两侧点数都为 $n$ 的二分图，$m$ 条边，保证存在完美匹配，问有多少条边满足原图删去它后最大匹配仍然是完美匹配。

双倍经验 $\text{bzoj 2140}$ 。

假如我们已经知道了任意一组最大匹配，那么没被匹配上的边一定是可以被删除的，也就是答案至少是 $m-n$ 。

考虑哪些匹配边也可以删。对于每条边边 $u \to v$ ，其中 $u$ 是左侧点，$v$ 是右侧点。我们建一张新图，如果 $u \to v$ 是匹配边那么保留(当然如果有重边我们只保留一条，其余的反向)，如果不是则反向变成 $v \to u$ 。如果匹配 $(u,v)$ 在新图中是在同一个 $\text{SCC}$ 内，那么这条边删除后一定存在完美匹配。因为新图依然是一张二分图，那么如果在同一个 $\text{SCC}$ 内说明这两个点在一个偶环上，将偶环上的所有边取反就是新的一组完美匹配了。反之，如果在两个 $\text{SCC}$ 内，删掉这条边一定会使匹配数恰好减小 $1$ 。

那么直接跑 $\text{TarjanSCC}$ 就行，这部分的复杂度为 $O(n+m)$ 。

然后本题的点边是 $1e5$ ，匈牙利跑最大匹配就 $\text{T}$ 飞了，有一种非常神奇的最大匹配算法 $\text{HopcroftKarp}$ ，于是整个算法的复杂度为 $O(m\sqrt n+n+m)$ 。

$\text{HopcroftKarp}$ 可以参考 $\text{CLRS}$ 或者 $\text{Wikipedia}$ ，代码里这部分抄的 $\text{std}$ 。

另，越南语大佬可以参考官方题解 http://acmicpc-vietnam.github.io/2019/national/editorial.pdf

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN=200005;
const int INF=0x7FFFFFFF;

int n,m,ans,gao[MAXN];

namespace HopcroftCarp
{
    struct edge
    {
        int from,to;
        void read()
        {
            scanf("%d%d",&from,&to);
        }
        int other(int x)
        {
            return from^to^x;
        }
    }e[MAXN];
	vector<int> V[MAXN],unmatched,E[MAXN];
	int matchLeft[MAXN],matchRight[MAXN],d[MAXN],ansRight[MAXN];
	bool was[MAXN],found;
	void init()
	{
		for(int i=0;i<=MAXN;i++) V[i].clear();
		for(int i=1;i<=m;i++) V[e[i].from].push_back(i),E[e[i].from].push_back(e[i].to);
		memset(matchLeft,0,sizeof(matchLeft));
		memset(matchRight,0,sizeof(matchRight));
		unmatched.resize(n);
		for(int i=0;i<n;i++) unmatched[i]=i+1;
		memset(ansRight,0,sizeof(ansRight));
	}
	void dfs(int u)
	{
		for(auto id:V[u])
        {
			int v=e[id].other(u);
			if(matchRight[v]==0) found=true;
			else
			{
				if(was[v]) continue;
				was[v]=true,dfs(matchRight[v]);
			}
			if(found)
			{
				matchLeft[u]=v,matchRight[v]=u;
				return;
			}
		}
	}
	void run()
	{
	    init();
		while(true)
        {
			memset(was,false,sizeof(was));
			int lastSz=unmatched.size();
			for(int i=int(unmatched.size())-1;i>=0;i--)
            {
				int u=unmatched[i];
				found=false;
				dfs(u);
				if(found)
					swap(unmatched.back(),unmatched[i]),unmatched.pop_back();
			}
			if(unmatched.size()==lastSz) break;
		}
	}
}

namespace TarjanSCC
{
    vector<int> e[MAXN];
    int num,top,low[MAXN],c[MAXN],dfn[MAXN],stk[MAXN],cnt,ins[MAXN];
    void tarjan(int x)
    {
        dfn[x]=low[x]=++num;
        stk[++top]=x,ins[x]=1;
        for(auto to:e[x])
            if(!dfn[to])
            {
                tarjan(to);
                low[x]=min(low[x],low[to]);
            }
            else if(ins[to])
                low[x]=min(low[x],dfn[to]);
        if(dfn[x]==low[x])
        {
            cnt++;int y;
            do{y=stk[top--],ins[y]=0,c[y]=cnt;}while(x!=y);
        }
    }
}

int main()
{
    scanf("%d%d",&n,&m);
    ans=m-n;
    using namespace HopcroftCarp;
    for(int i=1;i<=m;i++) e[i].read();
    run();
    for(int i=1;i<=n;i++)
        for(auto to:E[i])
            if(matchLeft[i]==to&&!gao[to]) gao[to]=1,TarjanSCC::e[i].push_back(to+n);
            else TarjanSCC::e[to+n].push_back(i);
    for(int i=1;i<=2*n;i++)
        if(!TarjanSCC::dfn[i]) TarjanSCC::tarjan(i);
    for(int i=1;i<=n;i++)
        for(auto to:E[i])
            if(matchLeft[i]==to&&gao[to]&&TarjanSCC::c[i]==TarjanSCC::c[to+n]) ans++,gao[to]=0;
    printf("%d",ans);
    return 0;
}
```

