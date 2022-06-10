---
title: "2020 HDU Multi-University Training Contest 3"
categories: [Tutorial]
tags: [Geometry,Greedy,Union Set]
toc: true
classes: []
excerpt: "2020杭电多校第三场"
---



Rank 101，前期大胜利，一度进过前 $15$ ，中后期雪崩...



|      | A    | B    | C    | D         | E         | F       | G      | H         | I         | J       | K    |
| ---- | ---- | ---- | ---- | --------- | --------- | ------- | ------ | --------- | --------- | ------- | ---- |
| 赛时 |      |      |      | :balloon: | :balloon: | :cloud: | :bulb: | :balloon: | :balloon: | :cloud: |      |
| 赛后 |      |      |      |           |           |         |        |           |           |         |      |



### D. Tokitsukaze and Multiple (签到)

$n$ 长数组，每次可以挑选相邻的两个合并成它们的和，最大化数组里 $p$ 的倍数的个数。

把前缀和模 $p$ 存下来贪心地合并即可。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n,p,ans,pre;
unordered_set<int> s;

void solve()
{
    ans=pre=0;
    scanf("%d%d",&n,&p);
    s.insert(0);
    for(int i=1,x;i<=n;i++)
    {
        scanf("%d",&x);
        pre=(pre+x)%p;
        if(!s.insert(pre).second)
        {
            s.clear();
            s.insert(0);
            pre=0;
            ans++;
        }
    }
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



### E. Little W and Contest (并查集+简单计数)

$n$ 个人，分为两类，三人可组一队。一队至少有 $2$ 个第二类人，且所有人互相不认识。一开始大家都不认识，然后 $n-1$ 个操作，每次介绍两个人认识，求过程中能组出一个队伍的方案数，认识具有传递性。

队友推的公式，设连通块 $x,y$ 各有 $A_x,B_x,A_y,B_y$ 个两类人，而第二类人总共有 $B$ 个，那么合并它们会使得答案减小 
$$
B_xB_y(n-A_x-B_x-A_y-B_y) + (A_xB_y+A_yB_x)(B-B_x-B_y)
$$

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const ll mod=1e9+7;
int n,fa[100005];
ll ans,A[100005],B[100005],totA,totB;

int _find(int x){return x==fa[x]?x:fa[x]=_find(fa[x]);}
void _merge(int x,int y)
{
    x=_find(x),y=_find(y);

    ans-=B[x]*B[y]%mod*(n-A[x]-B[x]-A[y]-B[y])%mod;
    ans=(ans+mod)%mod;
    ans-=(A[x]*B[y]+A[y]*B[x])*(totB-B[x]-B[y])%mod;
    ans=(ans+mod)%mod;

    fa[x]=y;A[y]+=A[x],B[y]+=B[x];
}

void solve()
{
    scanf("%d",&n);
    totA=totB=0;
    for(int i=1,x;i<=n;i++)
    {
        fa[i]=i;
        scanf("%d",&x);
        A[i]=B[i]=0;
        if(x==1) A[i]=1,totA++; else B[i]=1,totB++;
    }
    ans=totB*(totB-1)*(totB-2)/6%mod+totB*(totB-1)/2*totA%mod;
    ans%=mod;
    printf("%lld\n",ans);
    for(int i=1,x,y;i<n;i++)
    {
        scanf("%d%d",&x,&y);
        _merge(x,y);
        printf("%lld\n",ans);
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



### H. Triangle Collision (几何)

给定初始坐标和运动方向与速度，问质点在边长 $L$ 的等边三角形内第 $k$ 次碰撞的时间。

把等边三角形平铺平面，第 $k$ 次碰撞等价于第 $k$ 次与直线相交，二分答案即可。

写的比较丑，事实上完全不需要几何的板子...

```cpp
#include <bits/stdc++.h>
using namespace std;

#define db double
const db EPS=1e-9;
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

const db SQRT3=sqrt(3);
const db SQRT32=sqrt(3)/2;
int k;
db L,x,y,vx,vy;
P P00,P01,P10,P11,P20,P21,st,ed,dir,tmp;

inline int gao(double dis,P &a,P &b,int id)
{
    ed=st+dir*dis;
    if(!chkLL(a,b,st,ed)) return 0;
    db QAQ=st.distTo(proj(a,b,st));
    tmp=isLL(a,b,st,ed);
    if((tmp-st).dot(ed-st)<0) QAQ=SQRT32-QAQ;
    db WTF=st.distTo(proj(ed,ed+b-a,st));
    return int((WTF-QAQ)/SQRT32+0.000001)+1;
}

inline bool check(double dis)
{
    return gao(dis,P00,P01,0)+gao(dis,P10,P11,1)+gao(dis,P20,P21,2)>=k;
}

void solve()
{
    scanf("%lf%lf%lf%lf%lf%d",&L,&x,&y,&vx,&vy,&k);
    st=P(x,y)/L;dir=P(vx,vy).unit();
    db l=0,r=10*k,mid;
    for(int i=0;i<100;i++)
    {
        mid=(l+r)/2;
        ed=st+dir*mid;
        if(!isSS(P00,P01,st,ed)&&!isSS(P10,P11,st,ed)&&!isSS(P20,P21,st,ed)) l=mid;
        else r=mid;
    }
    r=10*k;
    for(int i=0;i<100;i++)
    {
        mid=(l+r)/2;
        if(check(mid)) r=mid;
        else l=mid;
    }
    db ans=(l+r)/2;
    ans=ans*L/sqrt(vx*vx+vy*vy);
    printf("%.10f\n",ans);
}

int main()
{
    P00=P10=P(-0.5,0);
    P11=P21=P(0,SQRT32);
    P01=P20=P(0.5,0);
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



### I. Parentheses Matching (贪心)

$n$ 长字符串，有左右括号和通配符，把它变成最短时字典序最小的合法括号序列。

尽量把左括号往左放，右括号往右放，所以左右各扫一遍就做完了。

```cpp
#include <bits/stdc++.h>
using namespace std;

int n;
char s[100005];

void solve()
{
    scanf("%s",s);
    n=strlen(s);
    int tot=0;

    for(int i=0,cur=0;i<n;i++)
    {
        if(s[i]=='(') tot++;
        else if(s[i]==')') tot--;
        if(tot<0)
        {
            while(cur<i&&s[cur]!='*') cur++;
            if(s[cur]!='*')
            {
                printf("No solution!\n");
                return;
            }
            s[cur]='(';tot++;
        }
    }

    tot=0;
    for(int i=n-1,cur=n-1;i>=0;i--)
    {
        if(s[i]=='(') tot++;
        else if(s[i]==')') tot--;
        if(tot>0)
        {
            while(cur>i&&s[cur]!='*') cur--;
            if(s[cur]!='*')
            {
                printf("No solution!\n");
                return;
            }
            s[cur]=')';tot--;
        }
    }

    tot=0;
    for(int i=0;i<n;i++)
        if(s[i]=='(') tot++;
        else if(s[i]==')') tot--;

    if(tot!=0)
    {
        printf("No solution!\n");
        return;
    }

    for(int i=0;i<n;i++)
        if(s[i]!='*') printf("%c",s[i]);
    printf("\n");
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```

