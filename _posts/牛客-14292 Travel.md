## 牛客-14292 Travel 最短路

[传送门](https://ac.nowcoder.com/acm/problem/14292)

$n$ 个点 $n$ 条边的无向图构成一个简单环，有额外 $m\leq20$ 条无向边在环中连接某两点，接下来 $q$ 个询问，问点对 $(x,y)$ 的距离。

由于 $n$ 不小，没有办法处理出所有点对之间的距离，这引导我们考虑在线的做法。

容易发现，点对在环上的距离是可以 $O(1)$ 得到的，我们只关心 $m$ 条额外的边所涉及到的端点之间的距离即可。

方法 $1$ ，对 $m$ 条边的最多 $2m$ 个端点为起点做单源最短路，这样预处理 $O(m(n+m)\log n)$ ，每次询问只要枚举 $x$ 在环上走到 $2m$ 个端点中的哪一个即可，可以 $O(m)$ 回答，总时间复杂度 $O(m(n+m)\log n+qm)$ 。

方法 $2$ ，对 $2m$ 个端点做任意点对最短路，回答时枚举 $x$ 和 $y$ 分别在环上走到 $2m$ 个端点中的哪一个。总时间复杂度 $O(n+m^3+qm^2)$ 。因为可能有重边所以用这种方法存图的时候要小心处理取最小值。

两种都能过，写哪个看喜好，下面代码是方法 $2$ 的。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef pair<int,int> pii;

int n,m,q,id[100005],inv[100005];
ll a[100005],pre[100005],tot,d[55][55];
set<int> v;
map<pii,ll> dis;

inline ll DIS(int x,int y)
{
    return min(abs(pre[x]-pre[y]),tot-abs(pre[x]-pre[y]));
}

int main()
{
    scanf("%d%d",&n,&m);
    for(int i=2;i<=n+1;i++) scanf("%lld",a+i),tot+=a[i];
    for(int i=2;i<=n+1;i++) pre[i]=pre[i-1]+a[i];
    for(int i=1,x,y,c;i<=m;i++)
    {
        scanf("%d%d%d",&x,&y,&c);
        v.insert(x),v.insert(y);
        if(dis.count({x,y}))
            dis[{x,y}]=dis[{y,x}]=min((long long)c,dis[{y,x}]);
        else
            dis[{x,y}]=dis[{y,x}]=c;
    }
    for(auto x:v) dis[{x,x}]=0;
    for(auto x:v) for(auto y:v)
        if(x!=y)
            if(dis.count({x,y}))
                dis[{x,y}]=dis[{y,x}]=min(dis[{y,x}],DIS(x,y));
            else
                dis[{x,y}]=dis[{y,x}]=DIS(x,y);
    for(auto k:v) for(auto i:v) for(auto j:v)
        dis[{i,j}]=min(dis[{i,j}],dis[{i,k}]+dis[{k,j}]);
    int cur=0;
    for(auto it:v) id[++cur]=it,inv[it]=cur;
    for(auto X:v) for(auto Y:v) d[inv[X]][inv[Y]]=dis[{X,Y}];
    scanf("%d",&q);
    for(int i=0,x,y;i<q;i++)
    {
        scanf("%d%d",&x,&y);
        ll ans=DIS(x,y);
        for(int X=1;X<=cur;X++) for(int Y=1;Y<=cur;Y++)
            ans=min(ans,DIS(x,id[X])+DIS(y,id[Y])+d[X][Y]);
        printf("%lld\n",ans);
    }
    return 0;
}
```

