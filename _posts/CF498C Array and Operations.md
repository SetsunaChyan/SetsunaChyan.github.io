## CF498C Array and Operations 网络流/二分图最大匹配

[传送门](https://codeforces.com/problemset/problem/498/C)

$n$ 个数字 $a[1],\dots,a[n]$ ，$m$ 对关系 $(i_1,j_1),\dots,(i_m,j_m)$ ，保证 $i_k+j_k$ 是奇数。

每次可以选择一对 $(i_k,j_k)$ 和一个整数 $v>1$ 使得 $v$ 是 $a[i_k]$ 和 $a[j_k]$ 的公因子，然后把 $a[i_k]$ 除以 $v$ ，把 $a[j_k]$ 除以 $v$ 。

问最多可以做这样的操作多少次。

显而易见的是，每次除以一个质因子一定是最优的。

条件 $i_k+j_k$ 是奇数明示了这是一张二分图，不妨把下标为奇数的看做左部，为偶数的看做右部。

每个数的各个质因子的决策之间是独立的，并且有 $2\times 3\times 5\times 7\times 11\times 13\times 17\times 19\times 23=223092870$ ，所以每个数最多只有 $9$ 个不相同的质因子。

考虑如下建图：

1. 建立源点 $s$ 和汇点 $t$ 。并且把每个数的每个质因子拆成入点和出点，来限制一个数中该质因子的出现次数。
2. 每个数的每个质因子的入点向出点连边，容量是这个数中这个质因子的出现次数。
3. 源点 $s$ 向每个左部数的每个质因子的入点连边，容量为正无穷。
4. 每个右部数的每个质因子的出点向汇点 $t$ 连边，容量为正无穷。
5. 如果关系 $(i,j)$ 存在($i$ 是奇数)，那么 $a_i$ 的每个质因子的出点向 $a_j$ 的对应质因子的入点连边，容量为该质因子在 $\gcd(a_i,a_j)$ 中的出现次数。

这个图的最大流就是答案。

![CF498C_1](/home/kirisame/Desktop/Article/pic/CF498C_1.png)

时间复杂度 $O(n\sqrt a+m+\text{max-flow}(|V|=18*n,|E|=18*n+9*m))$ ，正常的流都能过。

样例 $2$ 的建图如下。

![CF498C_2](/home/kirisame/Desktop/Article/pic/CF498C_2.png)

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef pair<int,int> pii;

const int MAXN=4*10*105;
const int inf=0x3f3f3f3f;
int n,m,s,t,tot,dis[MAXN],cur[MAXN],a[105];
pii e[105];
map<int,int> mp[105],id[105];
struct edge
{
    int to,cap,rev;
    edge(){}
    edge(int to,int cap,int rev):to(to),cap(cap),rev(rev){}
};
vector<edge> E[MAXN];

inline void add_edge(int x,int y,int f)
{
    E[x].emplace_back(y,f,E[y].size());
    E[y].emplace_back(x,0,E[x].size()-1);
}

int bfs()
{
    for(int i=s;i<=t;i++) dis[i]=inf;
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

int dfs(int now,ll flow)
{
    if(now==t) return flow;
    int rest=flow,k;
    for(int i=cur[now];i<E[now].size();i++)
    {
        edge &e=E[now][i];
        if(e.cap&&dis[e.to]==dis[now]+1)
        {
            cur[now]=i;
            k=dfs(e.to,min(rest,e.cap));
            e.cap-=k;
            E[e.to][e.rev].cap+=k;
            rest-=k;
        }
    }
    return flow-rest;
}

int dinic()
{
    int ret=0,delta;
    while(bfs())
    {
        for(int i=s;i<=t;i++) cur[i]=0;
        while(delta=dfs(s,inf)) ret+=delta;
    }
    return ret;
}

void gao(int x)
{
    int t=a[x];
    for(int i=2;i*i<=t;i++)
        if(t%i==0)
        {
            int k=0;
            while(t%i==0) t/=i,k++;
            mp[x][i]=k,id[x][i]=tot,tot+=2;
        }
    if(t!=1) mp[x][t]=1,id[x][t]=tot,tot+=2;
}

int main()
{
    s=0,tot=1;
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++) scanf("%d",a+i),gao(i);
    t=tot;
    for(int i=1;i<=n;i++)
        for(auto it:id[i])
        {
            if(i&1) add_edge(s,it.second,inf);
            else add_edge(it.second+1,t,inf);
            add_edge(it.second,it.second+1,mp[i][it.first]);
        }
    for(int i=0,x,y;i<m;i++)
    {
        scanf("%d%d",&x,&y);
        if(y&1) swap(x,y);
        for(auto it:mp[x])
        {
            if(mp[y].count(it.first)==0) continue;
            add_edge(id[x][it.first]+1,id[y][it.first],min(mp[y][it.first],it.second));
        }
    }
    printf("%d",dinic());
    return 0;
}
```