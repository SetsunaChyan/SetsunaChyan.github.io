---
title: "2020 HDU Multi-University Training Contest 2"
categories: [Tutorial]
tags: [Union Set,Flows,DP]
toc: true
classes: []
excerpt: "2020杭电多校第二场"
---



Rank 125.



|      | A    | B    | C    | D         | E         | F         | G    | H    | I         | J    | K         | L         |
| ---- | ---- | ---- | ---- | --------- | --------- | --------- | ---- | ---- | --------- | ---- | --------- | --------- |
| 赛时 |  :balloon:  |      | :eyes: |  | :balloon: | :balloon:   | :eyes: |      | :eyes: | :balloon: |      | :cloud: |
| 赛后 |      |      |      |           |           | |      |      |           | :balloon: |  | :balloon: |



### A. Total Eclipse (并查集)

$n$ 点 $m$ 边无向图，有点权，每次可以挑选一个极大连通分量使得所有点权减一，问最少要多少次操作能使点权全为零。

一开始题面里面没有要求是极大的，看那么多人过了就直接糊过了，事实上这个做法在非极大时是不正确的。

考虑倒着搞，最后删的一定是点权最大的。最后的形态一定是 $n$ 个连通块，每个连通块里只有一个点。从大到小每次把相同大小的点往比它更大的点合并，更新连通块数量，然后把每个连通块上的点权都减到比目前点权小的最大点权一样大(如果目前点权是最小的那么就减到 $0$ )。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n,m,fa[100005],org[100005],cnt;
struct node
{
    int pos,v;
}a[100005];
vector<int> e[100005];

int _find(int x){return x==fa[x]?x:fa[x]=_find(fa[x]);}
void _merge(int x,int y)
{
    x=_find(x);y=_find(y);
    if(x!=y) cnt--,fa[x]=y;
}

bool cmp(node a,node b)
{
    return a.v>b.v;
}

void solve()
{
    cnt=0;
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++) scanf("%d",&a[i].v),a[i].pos=i,org[i]=a[i].v,fa[i]=i,e[i].clear();
    sort(a+1,a+1+n,cmp);
    for(int i=1,x,y;i<=m;i++)
    {
        scanf("%d%d",&x,&y);
        e[x].push_back(y);
        e[y].push_back(x);
    }
    long long ans=0;
    for(int i=1;i<=n;i++)
    {
        cnt++;
        for(auto to:e[a[i].pos])
            if(org[to]>=a[i].v) _merge(a[i].pos,to);
        if(i==n||a[i].v!=a[i+1].v)
        {
            int tmp=a[i].v;
            if(i!=n) tmp-=a[i+1].v;
            ans+=(long long)cnt*tmp;
        }
    }
    printf("%lld\n",ans);
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



### E. New Equipments (最小费用最大流)

$n$ 条开口向上的抛物线，横坐标只能选 $[1,m]$ 内的整数，对于所有 $i \in [1,n]$ 求选 $i$ 条抛物线的最小代价。代价是每条抛物线在自选横坐标上的取值，$i$ 条抛物线选的横坐标必须是两两不同的。  

题意就是一个最小代价的匹配，虽然 $m$ 很大，但有用的横坐标对于每条抛物线来说只有 $n$ 个，所以右侧点最多 $n^2$ 个。由于 $n$ 很小，直接跑 $n$ 次最小费用最大流就行了。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const ll inf=0x3f3f3f3f3f3f3f3fll;
ll msmf,dis[5005];
int s,t,n,m,cur[5005],vis[5005];
struct edge
{
    int to,rev;
    ll cap,cost;
    edge(){}
    edge(int to,ll cap,ll cost,int rev):to(to),cap(cap),cost(cost),rev(rev){}
};
vector<edge> E[5005];
map<int,vector<int>> mp;

inline void add_edge(int x,int y,ll f,ll c)
{
    E[x].emplace_back(y,f,c,E[y].size());
    E[y].emplace_back(x,0,-c,E[x].size()-1);
}

int spfa()
{
    for(int i=s;i<=t;i++) vis[i]=0,dis[i]=inf;
    dis[s]=0;
    queue<int> q;
    q.push(s);
    while(!q.empty())
    {
        int p=q.front();q.pop();
        vis[p]=0;
        for(auto e:E[p])
            if(e.cap&&dis[p]+e.cost<dis[e.to])
            {
                dis[e.to]=dis[p]+e.cost;
                if(!vis[e.to])
                    vis[e.to]=1,q.push(e.to);
            }
    }
    return dis[t]!=inf;
}

ll dfs(int now,ll flow)
{
    if(now==t) return flow;
    ll rest=flow,k;
    vis[now]=1;
    for(int i=cur[now];i<E[now].size();i++)
    {
        edge &e=E[now][i];
        if(e.cap&&dis[now]+e.cost==dis[e.to]&&!vis[e.to])
        {
            cur[now]=i;
            k=dfs(e.to,min(rest,e.cap));
            e.cap-=k;
            E[e.to][e.rev].cap+=k;
            msmf+=k*e.cost;
            rest-=k;
        }
    }
    vis[now]=0;
    return flow-rest;
}

ll dinic()
{
    msmf=0;
    ll ret=0,delta;
    while(spfa())
    {
        for(int i=s;i<=t;i++) cur[i]=vis[i]=0;
        while(delta=dfs(s,inf)) ret+=delta;
    }
    return ret;
}

ll a[55],b[55],c[55];

void solve()
{
    scanf("%d%d",&n,&m);
    for(int i=1;i<=n;i++) scanf("%lld%lld%lld",a+i,b+i,c+i);
    for(int T=1;T<=n;T++)
    {
        mp.clear();
        s=0,t=61*n+2;
        for(int i=s;i<=t;i++) E[i].clear();
        add_edge(s,s+1,T,0);
        for(int i=1;i<=n;i++)
        {
            add_edge(s+1,s+1+i,1,0);
            int mid=max(1ll,-b[i]/(2*a[i])),cur=0,l=mid,r=mid+1;
            while(cur<min(60,m)&&(l>=1||r<=m))
            {
                if(l>=1)
                {
                    cur++;
                    mp[l].push_back(i);
                    l--;
                }
                if(cur>=min(60,m)) break;
                if(r<=m)
                {
                    cur++;
                    mp[r].push_back(i);
                    r++;
                }
            }
        }
        int tot=s+1+n;
        for(auto it:mp)
        {
            ll x=it.first;
            tot++;
            for(auto from:it.second)
                add_edge(s+1+from,tot,1,a[from]*x*x+b[from]*x+c[from]);
            add_edge(tot,t,1,0);
        }
        dinic();
        printf("%lld%c",msmf," \n"[T==n]);
    }
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



### F. The Oculus (乱搞)

给出用斐波那契数系表示的两个数 $A,B$ 和它们的乘积 $C$ ，$C$ 的斐波那契表示恰好有一位 $1$ 变成了 $0$ ，求它是第几位。

找到一个合适的模数使得斐波那契数列前 $2e5$ 项两两不同余即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

ll mod[2]={998244353,1000000007},fib[2000005][2],A[2],B[2],C[2],CC[2];
int an,bn,cn,c[2000005],x;

inline int read()
{
    char ch=getchar();int s=0;
    while(ch<48||ch>57){ch=getchar();}
    while(ch>=48&&ch<=57){s=(s<<1)+(s<<3)+ch-48;ch=getchar();}
    return s;
}

void solve()
{
    A[0]=A[1]=B[0]=B[1]=CC[0]=CC[1]=0;
    an=read();
    for(int i=1;i<=an;i++)
    {
        x=read();
        if(!x) continue;
        A[0]=(A[0]+fib[i][0])%mod[0];
        A[1]=(A[1]+fib[i][1])%mod[1];
    }
    bn=read();
    for(int i=1;i<=bn;i++)
    {
        x=read();
        if(!x) continue;
        B[0]=(B[0]+fib[i][0])%mod[0];
        B[1]=(B[1]+fib[i][1])%mod[1];
    }
    C[0]=A[0]*B[0]%mod[0];
    C[1]=A[1]*B[1]%mod[1];
    cn=read();
    for(int i=1;i<=cn;i++)
    {
        c[i]=x=read();
        if(!x) continue;
        CC[0]=(CC[0]+fib[i][0])%mod[0];
        CC[1]=(CC[1]+fib[i][1])%mod[1];
    }
    for(int i=1;i<=cn;i++)
    {
        if(c[i]==0&&(i==1||c[i-1]==0)&&(i==cn||c[i+1]==0))
            if((CC[0]+fib[i][0])%mod[0]==C[0]&&(CC[1]+fib[i][1])%mod[1]==C[1])
            {
                printf("%d\n",i);
                return;
            }
    }
}

int main()
{
    fib[0][0]=fib[0][1]=1;
    fib[1][0]=fib[1][1]=1;
    for(int i=2;i<=2000001;i++)
    {
        fib[i][0]=(fib[i-1][0]+fib[i-2][0])%mod[0];
        fib[i][1]=(fib[i-1][1]+fib[i-2][1])%mod[1];
    }
    int _;
    _=read();
    while(_--) solve();
    return 0;
}
```



### J. Lead of Wisdom (搜索)

$n$ 物品，每个物品有个 $type$ 和 $a,b,c,d$ 四种属性，每个 $type$ 只能选最多一个。

要求最大化 $(100+\sum a)(100+\sum b)(100+\sum c)(100+\sum d)$。

显然只能爆搜，注意空的 $type$ 在搜的时候要跳掉，否则在大量的空 $type$ 时复杂度会多个 $k$。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

struct node
{
    int a,b,c,d;
};
int n,k,nxt[55];
ll ans;
vector<node> v[55];

void dfs(int x,int A,int B,int C,int D)
{
    if(x==0)
    {
        ans=max(ans,(100ll+A)*(100ll+B)*(100ll+C)*(100ll+D));
        return;
    }
    for(auto it:v[x])
        dfs(nxt[x],A+it.a,B+it.b,C+it.c,D+it.d);
}

void solve()
{
    ans=0;
    scanf("%d%d",&n,&k);
    for(int i=0;i<=k;i++) v[i].clear(),nxt[i]=0;
    for(int i=0,t,a,b,c,d;i<n;i++)
    {
        scanf("%d%d%d%d%d",&t,&a,&b,&c,&d);
        v[t].push_back({a,b,c,d});
    }
    for(int i=k-1;i>=0;i--)
        if(v[i+1].size()!=0) nxt[i]=i+1;
        else nxt[i]=nxt[i+1];
    dfs(nxt[0],0,0,0,0);
    printf("%lld\n",ans);
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



### L. String Distance (DP)

给长度分别为 $n,m$ 的两串 $A,B$ ，多个询问，询问 $A[l...r]$ 和 $B$ 之间的距离，距离定义是在任意一个串种删除或添加一个任意一个字符的最少操作次数，$m \leq 20$。

这个距离等价于两串长减去它们的 $\text{LCS}$ ，由于 $m$ 很小，可以先暴力预处理出 $A$ 串中每个下标开始往后第一个碰到的字母 $c$ 在哪里，记为 $nxt$ 。

设 $\text{DP}_{i,j}$ 表示和 $B$ 的前 $i$ 个字符匹配了恰好 $j$ 个字符时最靠左的下标，那么就可以用 $nxt$ 数组快速转移了，单次询问 $O(m^2)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

int q,n,m,nxt[100005][26],dp[22][22];
char A[100005],B[22];

void solve()
{
    scanf("%s%s",A+1,B+1);
    n=strlen(A+1),m=strlen(B+1);
    for(int i=0;i<26;i++) nxt[n][i]=0x3f3f3f3f;
    for(int i=n-1;i>=0;i--)
    {
        for(int j=0;j<26;j++)
            nxt[i][j]=nxt[i+1][j];
        nxt[i][A[i+1]-'a']=i+1;
    }
    scanf("%d",&q);
    while(q--)
    {
        int l,r,ans=0;
        scanf("%d%d",&l,&r);
        memset(dp,0x3f3f3f3f,sizeof(dp));
        dp[0][0]=l-1;
        for(int i=0;i<m;i++)
            for(int j=0;j<=i;j++)
            {
                dp[i+1][j]=min(dp[i+1][j],dp[i][j]);
                if(dp[i][j]<=r)
                    dp[i+1][j+1]=min(dp[i+1][j+1],nxt[dp[i][j]][B[i+1]-'a']);
            }
        for(int i=m;i>=0;i--)
            if(dp[m][i]<=r)
            {
                ans=i;
                break;
            }
        printf("%d\n",m+r-l+1-2*ans);
    }
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```

