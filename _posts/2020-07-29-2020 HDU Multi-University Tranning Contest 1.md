---
title: "2020 HDU Multi-University Training Contest 1"
categories: [Tutorial]
tags: [Number Theory,SQRT Alogorithm,Convex,Lyndon Decomposition,Half-plane Intersection]
toc: true
classes: []
excerpt: "2020杭电多校第一场"
---



|      | A    | B    | C    | D         | E         | F         | G    | H    | I         | J    | K         | L         |
| ---- | ---- | ---- | ---- | --------- | --------- | --------- | ---- | ---- | --------- | ---- | --------- | --------- |
| 赛时 |      |      |      | :balloon: | :balloon: | :cloud:   |      |      | :balloon: |      |           | :bulb:    |
| 赛后 |      |      |      |           |           | :balloon: |      |      |           |      | :balloon: | :balloon: |



Rank 87，依旧菜的真实。



### D. Distinct Sub-palindromes (签到)

求仅包含小写英文字母的长度为 $n$ 的串中，满足本质不同的回文子串数量最少的有几种。

发现长度为 $1,2,3$ 时，任意组合的本质不同回文子串数都是一样的，而 $n \geq 4$ 时，形如 `xyzxyz...` 这样的 pattern 一定是最少的。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n;

int main()
{
    int _;
    scanf("%d",&_);
    while(_--)
    {
        scanf("%d",&n);
        if(n==1) printf("26\n");
        else if(n==2) printf("676\n");
        else if(n==3) printf("%d\n",26*26*26);
        else printf("%d\n",26*25*24);
    }
    return 0;
}
```



### E. Fibonacci Sum (二次剩余)

设 $F_i$ 是斐波那契数列第 $i$ 项，给定 $n,c,k$ 求 $\sum_{i=0}^n F_{ci}^K\ \text{mod}\ 1e9+9$。

赛时也是找到了原题直接抄了改了...

斐波那契通项
$$
\frac{1}{\sqrt{5}}[(\frac{1+\sqrt 5}{2})^n-(\frac{1-\sqrt 5}{2})^n]
$$

$5$ 又恰好是该模数的二次剩余，注意到 $383008016^2$ 与 $5$ 同余，那么我们就可以用 $383008016$ 来代替 $\sqrt{5}$ 了。

记 
$$
a_n=(\frac{1+\sqrt 5}{2})^n,b_n=(\frac{1-\sqrt 5}{2})^n
$$

那么我们有
$$
F_{ci}^k=\frac{1}{\sqrt 5}^{k} (a_{ci}-b_{ci})^{k}
$$
再用二项式展开一波
$$
(a_{ci}-b_{ci})^{k}=\sum_{i=0}^{k} (-1)^{i}\tbinom{i}{k}a_{ci}^{k-i}b_{ci}^{i}
$$
对于二项式展开后的同一个项 $(-1)^i\tbinom{i}{k}a^{k-i} b^i$，不同的 $i$ 就形成了一个等比数列，直接使用等比数列求和就行。

```cpp
#include<cstdio>
#include<cstdlib>
#include<algorithm>
using namespace std;
typedef long long ll;

const ll P=1000000009;
const ll INV2=500000005;
const ll SQRT5=383008016;
const ll INVSQRT5=276601605;
ll A=691504013,Z,ZZ,Ainv,B=308495997;

const int N=100005;

ll n,K,c,fac[N],inv[N],pa,pb;

inline void Pre(int n)
{
    fac[0]=1;
    for(int i=1;i<=n;i++) fac[i]=fac[i-1]*i%P;
    inv[1]=1;
    for(int i=2;i<=n;i++) inv[i]=(P-P/i)*inv[P%i]%P;
    inv[0]=1;
    for(int i=1;i<=n;i++) inv[i]=inv[i]*inv[i-1]%P;
}

inline ll C(int n,int m){return fac[n]*inv[m]%P*inv[n-m]%P;}

ll Pow(ll a,ll b)
{
    ll ret=1;
    while(b)
    {
        if(b&1) ret=ret*a%P;
        a=a*a%P;
        b>>=1;
    }
    return ret;
}

inline ll Inv(ll x){return Pow(x,P-2);}
inline void Add(ll &x,ll y)
{
    x+=y;
    if(x<0) x+=P;
    else if(x>=P) x-=P;
}

void solve()
{
    scanf("%lld%lld%lld",&n,&c,&K);
    ll Ans=0;
    A=Pow(691504013,c),B=Pow(308495997,c);
    ll t=Pow(A,K),tt=Pow(t,n),tem;
    Ainv=Inv(A);
    Z=Ainv*B%P,ZZ=Pow(Z,n);
    for(int j=0;j<=K;j++)
    {
        tem=t==1?n%P:t*(tt+P-1)%P*Inv(t-1)%P;
        if(~j&1) Add(Ans,C(K,j)*tem%P);
        else Add(Ans,P-C(K,j)*tem%P);
        t=t*Z%P,tt=tt*ZZ%P;
    }
    Ans=Ans*Pow(INVSQRT5,K)%P;
    printf("%lld\n",Ans);
}

int main()
{
    int _;
    Pre(100000);
    scanf("%d",&_);
    while(_--) solve();
}
```



### F. Finding a MEX (根号分类讨论+树状数组上二分)

$n$ 点 $m$ 边无向图，单点修改点权或者询问和某个点邻接的所有点的 $\text{MEX}$ 。

一个比较直接的思路就是对度数分类讨论，度数大于 $\sqrt{n}$ 的点数数量一定是小于 $2\sqrt{n}$ 的。而又注意到，一个点的 $\text{MEX}$ 值是不会超过其度数的，我们可以对每个点开度数长度的树状数组维护答案，总空间是 $O(n)$ 的。

修改操作中，对于度数小的点来说，直接暴力更改与它邻接的点的树状数组；对于度数大的点，只修改该点权值。

查询操作中，该点的树状数组中已经存下了所有度数小的点的答案，我们只需要暴力把大的点插进树状数组，统计答案，再删掉它们就行，注意到度数大的点是 $O(\sqrt{n})$ 的，单次操作也就 $O(\sqrt{n}\log n)$ 。

线段树上二分同样也是一个 $\log$ 的但常数太大卡不太进去，所以8太行。树状数组上二分早有耳闻但从来没写过，就当涨知识了。

```cpp
#include <bits/stdc++.h>
using namespace std;

const int GAO=350;
int n,m,a[100005],deg[100005];
vector<int> e[100005],big[100005],bit[100005],cnt[100005];

inline int read()
{
    char ch=getchar();int s=0,w=1;
    while(ch<48||ch>57){if(ch=='-')w=-1;ch=getchar();}
    while(ch>=48&&ch<=57){s=(s<<1)+(s<<3)+ch-48;ch=getchar();}
    return s*w;
}

inline void write(int x)
{
    if(x<0)putchar('-'),x=-x;
    if(x>9)write(x/10);
    putchar(x%10+48);
}

void modify(vector<int> &bit,vector<int> &cnt,int x,int y)
{
    cnt[x]+=y;
    if(y==1&&cnt[x]!=1) return;
    if(y==-1&&cnt[x]!=0) return;
    for(;x<bit.size();x+=x&-x) bit[x]+=y;
}

int query(vector<int> &bit)
{
    int cnt=0,ret=0,n=bit.size()-1;
    for(int i=log2(n);~i;i--)
    {
        ret+=1<<i;
        if(ret>=n||cnt+bit[ret]!=ret) ret-=1<<i;
        else cnt+=bit[ret];
    }
    return ret+1;
}

void solve()
{
    n=read(),m=read();
    for(int i=1;i<=n;i++)
    {
        a[i]=read()+1,deg[i]=0;
        e[i].clear(),big[i].clear();
    }
    for(int i=0,x,y;i<m;i++)
    {
        x=read(),y=read();
        e[x].push_back(y),e[y].push_back(x);
        deg[x]++,deg[y]++;
    }
    for(int i=1;i<=n;i++)
    {
        bit[i]=vector<int>(deg[i]+2),cnt[i]=vector<int>(deg[i]+2);
        for(auto to:e[i])
            if(deg[to]>GAO)
                big[i].push_back(to);
            else
                if(a[to]<=deg[i]+1) modify(bit[i],cnt[i],a[to],1);
    }
    int q,op,u,x;
    q=read();
    while(q--)
    {
        op=read(),u=read();
        if(op==1)
        {
            x=read()+1;
            if(deg[u]<=GAO)
                for(auto to:e[u])
                {
                    if(a[u]<=deg[to]+1) modify(bit[to],cnt[to],a[u],-1);
                    if(x<=deg[to]+1) modify(bit[to],cnt[to],x,1);
                }
            a[u]=x;
        }
        else
        {
            for(auto to:big[u])
                if(a[to]<=deg[u]+1)
                    modify(bit[u],cnt[u],a[to],1);
            int ans=query(bit[u]);
            for(auto to:big[u])
                if(a[to]<=deg[u]+1)
                    modify(bit[u],cnt[u],a[to],-1);
            write(ans-1);putchar('\n');
        }
    }
}

int main()
{
    int _=read();
    while(_--) solve();
    return 0;
}
```



### I. Leading Robots (凸包)

$x$ 轴上 $n$ 个点，每个点具有初始位置和自己的加速度，求在无穷的时间内，曾经成为过最靠右的点的个数。

做法很多，赛时过的是和题解不同的奇怪做法。

假设某个点 $(a,x)$ 目前是第一，第一个会和他齐头并进的点一定是 $\frac{x-x'}{a-a'}$ 最大的，且如果有多个最大的，下一个第一一定是其中 $a$ 最大的。

把所有点用 $(a,x)$ 表示到平面直角坐标系里，发现我们要求的就是严格上凸壳上点的个数，注意可能多个点使用同一个 $(a,x)$，当它们出现在凸包上的时候我们不计数即可。

```cpp
#include<bits/stdc++.h>
using namespace std;
typedef long long ll;

#define db ll
const db EPS=1e-9;
inline int sign(db a){return a<-EPS?-1:a>EPS;}
inline int cmp(db a,db b){return sign(a-b);}
struct P
{
    db x,y;
    bool ok;
    P(){ok=false;}
    P(db x,db y):x(x),y(y){ok=false;}
    P operator+(P p){return {x+p.x,y+p.y};}
    P operator-(P p){return {x-p.x,y-p.y};}
    P operator*(db d){return {x*d,y*d};}
    P operator/(db d){return {x/d,y/d};}
    bool operator<(P p) const
    {
        int c=cmp(x,p.x);
        if(c) return c==-1;
        return cmp(y,p.y)==-1;
    }
    bool operator==(P o) const
    {
        return cmp(x,o.x)==0&&cmp(y,o.y)==0;
    }
    db distTo(P p){return (*this-p).abs();}
    db alpha(){return atan2(y,x);}
    void read(){scanf("%lld%lld",&x,&y);}
    void write(){printf("(%.10f,%.10f)\n",x,y);}
    db abs(){return sqrt(abs2());}
    db abs2(){return x*x+y*y;}
    P rot90(){return P(-y,x);}
    P unit(){return *this/abs();}
    int quad() const {return sign(y)==1||(sign(y)==0&&sign(x)>=0);}
    db dot(P p){return x*p.x+y*p.y;}
    db det(P p){return x*p.y-y*p.x;}
};

//For segment
#define cross(p1,p2,p3) ((p2.x-p1.x)*(p3.y-p1.y)-(p3.x-p1.x)*(p2.y-p1.y))
#define crossOp(p1,p2,p3) sign(cross(p1,p2,p3))

vector<P> convexHull(vector<P> ps)
{
    int n=ps.size();if(n<=1) return ps;
    sort(ps.begin(),ps.end());
    vector<P> qs(n*2);int k=0;
    for(int i=0;i<n;qs[k++]=ps[i++])
        while(k>1&&crossOp(qs[k-2],qs[k-1],ps[i])<=0) --k;
    for(int i=n-2,t=k;i>=0;qs[k++]=ps[i--])
        while(k>t&&crossOp(qs[k-2],qs[k-1],ps[i])<=0) --k;
    qs.resize(k-1);
    return qs;
}

int n;
vector<P> v,con;
map<pair<ll,ll>,int> mp;

void solve()
{
    v.clear();
    mp.clear();
    scanf("%d",&n);
    ll mxx=-1,mxy=-1;
    for(int i=0;i<n;i++)
    {
        ll x,y;
        scanf("%lld%lld",&y,&x);
        mp[make_pair(x,y)]++;
        mxx=max(mxx,x);
        mxy=max(mxy,y);
    }
    v.push_back(P(-1,-1));
    v.push_back(P(-1,mxy));
    v.push_back(P(mxx,-1));
    for(auto it:mp)
    {
        P tmp(it.first.first,it.first.second);
        if(it.second==1) tmp.ok=true;
        v.push_back(tmp);
    }
    con=convexHull(v);
    int ans=0;
    for(auto it:con) if(it.ok) ans++;
    printf("%d\n",ans);
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



### K. Minimum Index (Lyndon分解)

求一个串的所有前缀的最小后缀。

考虑在 Lyndon 分解的过程中，我们已经维护住了某个相似 Lyndon 串为 $t,t,t \dots t,t'$。

如果 $S_j=S_k$ ，那么 $t'$ 就是最小后缀；

如果 $S_j<S_k$ ，那么这个相似 Lyndon 串就是当前前缀的最后一个 Lyndon 子串，它一定是最小的；

如果 $S_j>S_k$ ，说明该相似 Lyndon 串找完了，从最后一次周期之后开始找新的 Lyndon 串。

注意到长度为 $1$ 的 Lyndon 串会没被统计到，补上即可。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const ll mod=1e9+7;
char s[1000005];
int n;
ll ans,dp[1000005],f[1000005];

void solve()
{
    ans=0;
    scanf("%s",s);
    n=strlen(s);
    int i,j,k;
    while(i<n)
    {
        j=i,k=i+1;
        dp[i]=i;
        while(k<n&&s[j]<=s[k])
        {
            if(s[j]==s[k]) dp[k]=dp[j]+k-j,j++;
            else j=i,dp[k]=i;
            k++;
        }
        while(i<=j) i+=k-j;
    }
    for(int i=0;i<n;i++)
        ans=(ans+(dp[i]+1)*f[i])%mod;
    printf("%lld\n",ans);
}

int main()
{
    int _;
    scanf("%d",&_);
    f[0]=1;
    for(int i=1;i<=1000000;i++) f[i]=f[i-1]*1112%mod;
    while(_--) solve();
    return 0;
}
```



### L. Mow (半平面交)

给一个凸包和一个圆，圆只能完全放在凸包里，圆可以移动。圆覆盖的面积单位代价是 $B$ ，其余凸包内未被覆盖的单位代价是 $A$ ，最小化代价和。

显然如果 $A \leq B$ 的话就不需要用圆了，答案就是凸包的面积。

考虑 $B$ 大的时候，圆心能在的位置就是凸包每条边往里推个半径，然后求个半平面交所形成的新的凸包(也可能为空，那就是放不下圆)。当然也可以不用半平面交求，求一下推完之后的所有相邻边的交点，然后连起来也是一样的。

设这个凸包的边长为 $P$ 面积是 $S$ ，圆的半径为 $r$ ，那么圆能覆盖的面积就是 $S+Pr+\pi r^2$ 。

赛时的做法是把圆往凸包的每个角落里卡，然后这个角落圆覆盖不到的面积就是一个多边形面积减掉扇形面积，但是因为存在凸包放不进圆这一情况没判好 WA 飞了，赛后下载数据发现 $100$ 组就挂了一个点...

```cpp
#include<bits/stdc++.h>
using namespace std;

#define db double
const db EPS=1e-8;
const db pi=acos(-1.0);
inline int sign(db a){return a<-EPS?-1:a>EPS;}
inline int cmp(db a,db b){return sign(a-b);}
struct P
{
    db x,y;
    P(){}
    P(db x,db y):x(x),y(y){}
    P operator+(P p){return {x+p.x,y+p.y};}
    P operator-(P p){return {x-p.x,y-p.y};}
    P operator*(db d){return {x*d,y*d};}
    P operator/(db d){return {x/d,y/d};}
    bool operator<(P p) const
    {
        int c=cmp(x,p.x);
        if(c) return c==-1;
        return cmp(y,p.y)==-1;
    }
    bool operator==(P o) const
    {
        return cmp(x,o.x)==0&&cmp(y,o.y)==0;
    }
    db distTo(P p){return (*this-p).abs();}
    db alpha(){return atan2(y,x);}
    void read(){scanf("%lf%lf",&x,&y);}
    void write(){printf("(%.10f,%.10f)\n",x,y);}
    db abs(){return sqrt(abs2());}
    db abs2(){return x*x+y*y;}
    P rot90(){return P(-y,x);}
    P unit(){return *this/abs();}
    int quad() const {return sign(y)==1||(sign(y)==0&&sign(x)>=0);}
    db dot(P p){return x*p.x+y*p.y;}
    db det(P p){return x*p.y-y*p.x;}
    P rot(db an){return {x*cos(an)-y*sin(an),x*sin(an)+y*cos(an)};}
};

int compareAngle(P a,P b)
{
    if(a.quad()!=b.quad()) return a.quad()<b.quad();
    return sign(a.det(b))>0;
}

//For segment
#define cross(p1,p2,p3) ((p2.x-p1.x)*(p3.y-p1.y)-(p3.x-p1.x)*(p2.y-p1.y))
#define crossOp(p1,p2,p3) sign(cross(p1,p2,p3))

bool chkLL(P p1,P p2,P q1,P q2) //0:parallel
{
    db a1=cross(q1,q2,p1),a2=-cross(q1,q2,p2);
    return sign(a1+a2)!=0;
}

P isLL(P p1,P p2,P q1,P q2) //crossover point if chkLL()
{
    db a1=cross(q1,q2,p1),a2=-cross(q1,q2,p2);
    return (p1*a2+p2*a1)/(a1+a2);
}

bool intersect(db l1,db r1,db l2,db r2)
{
    if(l1>r1) swap(l1,r1);if(l2>r2) swap(l2,r2);
    return !(cmp(r1,l2)==-1||cmp(r2,l1)==-1);
}

bool isSS(P p1,P p2,P q1,P q2)
{
    return intersect(p1.x,p2.x,q1.x,q2.x)&&intersect(p1.y,p2.y,q1.y,q2.y)&&
    crossOp(p1,p2,q1)*crossOp(p1,p2,q2)<=0&&crossOp(q1,q2,p1)*crossOp(q1,q2,p2)<=0;
}

bool isSS_strict(P p1,P p2,P q1,P q2)
{
    return crossOp(p1,p2,q1)*crossOp(p1,p2,q2)<0
    &&crossOp(q1,q2,p1)*crossOp(q1,q2,p2)<0;
}

bool isMiddle(db a,db m,db b)
{
    return sign(a-m)==0||sign(b-m)==0||(a<m!=b<m);
}

bool isMiddle(P a,P m,P b)
{
    return isMiddle(a.x,m.x,b.x)&&isMiddle(a.y,m.y,b.y);
}

bool onSeg(P p1,P p2,P q)
{
    return crossOp(p1,p2,q)==0&&isMiddle(p1,q,p2);
}

bool onSeg_strict(P p1,P p2,P q)
{
    return crossOp(p1,p2,q)==0&&sign((q-p1).dot(p1-p2))*sign((q-p2).dot(p1-p2))<0;
}

P proj(P p1,P p2,P q)
{
    P dir=p2-p1;
    return p1+dir*(dir.dot(q-p1)/dir.abs2());
}

P reflect(P p1,P p2,P q)
{
    return proj(p1,p2,q)*2-q;
}

db nearest(P p1,P p2,P q)
{
    P h=proj(p1,p2,q);
    if(isMiddle(p1,h,p2))
        return q.distTo(h);
    return min(p1.distTo(q),p2.distTo(q));
}

db disSS(P p1,P p2,P q1,P q2) //dist of 2 segments
{
    if(isSS(p1,p2,q1,q2)) return 0;
    return min(min(nearest(p1,p2,q1),nearest(p1,p2,q2)),min(nearest(q1,q2,p1),nearest(q1,q2,p2)));
}

db rad(P p1,P p2)
{
    return atan2l(p1.det(p2),p1.dot(p2));
}

db area(vector<P> ps)
{
    db ret=0;
    for(int i=0;i<ps.size();i++)
        ret+=ps[i].det(ps[(i+1)%ps.size()]);
    return ret/2;
}

int contain(vector<P> ps,P p) //2:inside,1:on_seg,0:outside
{
    int n=ps.size(),ret=0;
    for(int i=0;i<n;i++)
    {
        P u=ps[i],v=ps[(i+1)%n];
        if(onSeg(u,v,p)) return 1;
        if(cmp(u.y,v.y)<=0) swap(u,v);
        if(cmp(p.y,u.y)>0||cmp(p.y,v.y)<=0) continue;
        ret^=crossOp(p,u,v)>0;
    }
    return ret*2;
}

int convexContain(vector<P> &ps,P p) //1:inside|on_seg,0:outside
{
    int n=ps.size(),l=1,r=n-1,mid;
    while(l<r)
    {
        mid=(l+r+1)>>1;
        if(sign(cross(ps[0],ps[mid],p))<0) r=mid-1; else l=mid;
    }
    if(l==1&&sign(cross(ps[0],ps[1],p))<0) return 0;
    if(l==n-1&&onSeg(ps[0],ps[n-1],p)) return 1;
    if(l!=n-1&&sign(cross(ps[l],ps[l+1],p))>=0) return 1;
    return 0;
}

vector<P> convexHull(vector<P> ps)
{
    int n=ps.size();if(n<=1) return ps;
    sort(ps.begin(),ps.end());
    vector<P> qs(n*2);int k=0;
    for(int i=0;i<n;qs[k++]=ps[i++])
        while(k>1&&crossOp(qs[k-2],qs[k-1],ps[i])<=0) --k;
    for(int i=n-2,t=k;i>=0;qs[k++]=ps[i--])
        while(k>t&&crossOp(qs[k-2],qs[k-1],ps[i])<=0) --k;
    qs.resize(k-1);
    return qs;
}

db convexDiameter(vector<P> ps)
{
    int n=ps.size();if(n<=1) return 0;
    int is=0,js=0;
    for(int k=1;k<n;k++) is=ps[k]<ps[is]?k:is,js=ps[js]<ps[k]?js:k;
    int i=is,j=js;
    db ret=ps[i].distTo(ps[j]);
    do{
        if((ps[(i+1)%n]-ps[i]).det(ps[(j+1)%n]-ps[j])>=0) (++j)%=n;
        else (++i)%=n;
        ret=max(ret,ps[i].distTo(ps[j]));
    }while(i!=is||j!=js);
    return ret;
}

struct L // p[0]->p[1]
{
    P p[2];
    L(P k1,P k2){p[0]=k1,p[1]=k2;}
    P& operator [] (int k){return p[k];}
    int include(P k){return sign((p[1]-p[0]).det(k-p[0]))>0;}
    P dir(){return p[1]-p[0];}
    L push(db dis) // push dis (left hand)
    {
        P delta=(p[1]-p[0]).rot90().unit()*dis;
        return {p[0]+delta,p[1]+delta};
    }
};

bool parallel(L l0,L l1){return sign(l0.dir().det(l1.dir()))==0;}

bool sameDir(L l0,L l1){return parallel(l0,l1)&&sign(l0.dir().dot(l1.dir()))==1;}

bool operator < (L l0,L l1)
{
    if(sameDir(l0,l1)) return l1.include(l0[0]);
    return compareAngle(l0.dir(),l1.dir());
}

P isLL(L l0,L l1){return isLL(l0[0],l0[1],l1[0],l1[1]);}

bool check(L u,L v,L w){return w.include(isLL(u,v));}

vector<P> halfPlaneIS(vector<L> &l)
{
    sort(l.begin(),l.end());
    deque<L> q;
    for(int i=0;i<(int)l.size();i++)
    {
        if(i&&sameDir(l[i],l[i-1])) continue;
        while(q.size()>1&&!check(q[q.size()-2],q[q.size()-1],l[i])) q.pop_back();
        while(q.size()>1&&!check(q[1],q[0],l[i])) q.pop_front();
        q.push_back(l[i]);
    }
    while(q.size()>2&&!check(q[q.size()-2],q[q.size()-1],q[0])) q.pop_back();
    while(q.size()>2&&!check(q[1],q[0],q[q.size()-1])) q.pop_front();
    vector<P> ret;
    for(int i=0;i<(int)q.size();i++) ret.push_back(isLL(q[i],q[(i+1)%q.size()]));
    return ret;
}

int n;
db A,B,r,totArea,Barea;
vector<P> con;
vector<L> v;

void solve()
{
    v.clear();
    con.clear();
    scanf("%d%lf",&n,&r);
    scanf("%lf%lf",&A,&B);
    for(int i=0;i<n;i++)
    {
        P tmp;
        tmp.read();
        con.push_back(tmp);
    }
    totArea=area(con),Barea=0;
    if(totArea<0) totArea=-totArea,reverse(con.begin(),con.end());
    if(A<=B)
    {
        printf("%.10f\n",A*totArea);
        return;
    }
    for(int i=0;i<n;i++) v.emplace_back(L(con[i],con[(i+1)%n]).push(r));
    con=halfPlaneIS(v);
    db tmp=area(con);
    if(tmp!=0)
    {
        db par=0;
        for(int i=0;i<con.size();i++) par+=con[i].distTo(con[(i+1)%con.size()]);
        Barea=tmp+r*r*pi+r*par;
    }
    printf("%.10f\n",A*(totArea-Barea)+B*Barea);
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```

