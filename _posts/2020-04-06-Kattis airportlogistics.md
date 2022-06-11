## Airport Logistics 几何+最短路

[传送门](<https://open.kattis.com/problems/airportlogistics>)

$\text{2D}$ 平面给若干有向线段，在线段上的速度为 $2$ 倍，求平面上两点之间的最短路。

几何寻路问题当中比较简单的一类。

线段端点一定是关键点。

考虑线段外一点到线段上的最短路和线段上到线段外一点的最短路，容易发现射入和射出一定和线段成 $60°$ ，可以求一阶导的零点或公式 $\frac{sin\theta_1}{v_1}=\frac{sin\theta_2}{v_2}$ 得到。所以经过每个点，沿着线段方向顺时针和逆时针旋转 $60°$ 方向的直线和线段的交点也一定是关键点。

然后直接跑 $\text{Dij}$ 即可。

```cpp
#include<bits/stdc++.h>
using namespace std;

#define db double
#define y2 fuck
const db EPS=1e-6;
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

const double pi=acos(-1.0);
int n,tot;
P p[205];
vector<int> e[405*405];
vector<double> v[405*405];
typedef pair<db,int> pdi;

void addEdge(int x,int y,double dis)
{
	e[x].push_back(y),e[y].push_back(x);
	v[x].push_back(dis),v[y].push_back(dis);
}

void gao(int num)
{
	vector<pdi> tv;
	P A=p[num*2],B=p[num*2+1],dir1,dir2;
	dir1=(B-A).rot(pi/3);
	dir2=(B-A).rot(-pi/3);
	tv.emplace_back(0.0,num*2);
	tv.emplace_back(B.distTo(A),num*2+1);
	for(int i=0;i<=2*n+1;i++)
	{
		if(i==num*2||i==num*2+1) continue;
		P p1=isLL(A,B,p[i],p[i]+dir1);
		P p2=isLL(A,B,p[i],p[i]+dir2);
		if(onSeg(A,B,p1))
		{
			++tot;
			tv.emplace_back(p1.distTo(A),tot);
			addEdge(i,tot,p[i].distTo(p1));
		}
		if(onSeg(A,B,p2))
		{
			++tot;
			tv.emplace_back(p2.distTo(A),tot);
			addEdge(i,tot,p[i].distTo(p2));
		}
	}
	sort(tv.begin(),tv.end());
	for(int i=1;i<tv.size();i++)
	{
		e[tv[i-1].second].push_back(tv[i].second);
		v[tv[i-1].second].push_back((tv[i].first-tv[i-1].first)/2);
	}
}

db dis[405*405];
int vis[405*405];

void dij(int st)
{
	priority_queue<pdi,vector<pdi>,greater<pdi>> q;
	for(int i=0;i<=tot;i++) dis[i]=1e9;
	dis[st]=0;
	q.emplace(0.0,st);
	while(!q.empty())
	{
		int now=q.top().second;q.pop();
		if(vis[now]) continue;
		vis[now]=1;
		for(int i=0,to;i<e[now].size();i++)
		{
			db d=v[now][i];
			to=e[now][i];
			if(dis[to]>dis[now]+d)
			{
				dis[to]=dis[now]+d;
				q.emplace(dis[to],to);
			}
		}
	}
}

int main()
{
	p[0].read(),p[1].read();
	scanf("%d",&n);tot=2*n+1;
	for(int i=1;i<=n;i++) p[i*2].read(),p[i*2+1].read();
	for(int i=0;i<=tot;i++)
		for(int j=0;j<=tot;j++)
			if(i!=j) addEdge(i,j,p[i].distTo(p[j]));
	for(int i=1;i<=n;i++) gao(i);
	dij(0);
	printf("%.10f",dis[1]);
	return 0;
}
```



