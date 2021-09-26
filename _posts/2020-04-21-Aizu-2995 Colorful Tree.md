---
​---
categories: tutorial
​---
---

[传送门](http://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=2995)

$n$ 个点的以 $1$ 为根的树，第 $i$ 个点可以涂成两种颜色 $c_i,d_i$ (可能相同)，问对于每个子树而言，树内最多能有几种颜色。

把颜色抽象成点，树上的点就代表着一条边，这样的图中，每个联通块对答案的贡献是 $min(V,E)$ 的。

在树上启发式合并的时候用并查集维护住每个联通块的点数和边数即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n,t,c1[100005],c2[100005],son[100005],siz[100005],sum,hson,ans[100005];
int pa[200005],edge[200005],ver[200005];
vector<int> e[100005];

int _find(int x){return pa[x]==x?x:pa[x]=_find(pa[x]);}
void _merge(int x,int y)
{
    x=_find(x),y=_find(y);
    sum-=min(ver[x],edge[x]);
    if(x!=y) sum-=min(ver[y],edge[y]);
    edge[x]++;
    if(x!=y) edge[x]+=edge[y],pa[y]=pa[x],ver[x]+=ver[y];
    sum+=min(ver[x],edge[x]);
}

void dfs1(int now,int fa)
{
    son[now]=0,siz[now]=1;
    for(auto to:e[now])
    {
        if(to==fa) continue;
        dfs1(to,now);
        siz[now]+=siz[to];
        if(siz[to]>siz[son[now]]) son[now]=to;
    }
}

void cal(int now,int fa,int op)
{
    if(op==1) _merge(c1[now],c2[now]);
    else
    {
        ver[c1[now]]=ver[c2[now]]=1;
        edge[c1[now]]=edge[c2[now]]=0;
        pa[c1[now]]=c1[now],pa[c2[now]]=c2[now];
    }
    for(auto to:e[now])
        if(to!=fa&&to!=hson) cal(to,now,op);
}

void dfs2(int now,int fa,int keep)
{
    for(auto to:e[now])
    {
        if(to==fa||to==son[now]) continue;
        dfs2(to,now,0);
    }
    if(son[now]) dfs2(son[now],now,1);
    hson=son[now];
    cal(now,fa,1);
    hson=0;
    ans[now]=sum;
    if(!keep) cal(now,fa,-1),sum=0;
}

int main()
{
    scanf("%d%d",&n,&t);
    for(int i=1,x,y;i<n;i++)
    {
        scanf("%d%d",&x,&y);
        e[x].push_back(y),e[y].push_back(x);
    }
    for(int i=1;i<=n;i++) scanf("%d%d",c1+i,c2+i);
    for(int i=1;i<=t;i++) pa[i]=i,ver[i]=1;
    dfs1(1,1);
    dfs2(1,1,1);
    for(int i=1;i<=n;i++) printf("%d\n",ans[i]);
    return 0;
}
```

 