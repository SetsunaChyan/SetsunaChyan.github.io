## HDU-5710 Digit-Sum 思维

设 $S(n)$ 为 $n$ 的数位和，给定 $a,b$ 求最小的正整数 $n$ 使得 $aS(n)=bS(2n)$ 。

很有意思的一道题。

首先有个很显然的观察，如果 $S(n)$ 是个定值，但 $n$ 是任意的，则有

$$
S(2n)=2S(n)-9k, 0 \leq k \leq \lfloor\frac{S(n)}{5}\rfloor
$$

其中 $k$ 是 $n$ 在乘以 $2$ 后会进位的数字个数，也即 $n$ 中大于等于 $5$ 的数位的个数。

然后我们把它带入 $aS(n)=bS(2n)$ 中得

$$
aS(n)=b[2S(n)-9k]
$$

变形得

$$
(2b-a)S(n)=9bk
$$

讨论一下，如果 $2b-a \lt 0$ ，那么原方程显然无解，因为右边恒非负。

如果 $2b-a=0$ ，那么令 $n=S(n)=1$ 就行了。

接下来考虑 $2b-a \gt 0$ 时，这个方程的通解如下

$$
S(n)=\frac{\text{lcm}(2b-a,9b)}{2b-a}d
$$

$$
k=\frac{\text{lcm}(2b-a,9b)}{9b}d
$$

其中 $d \geq 1$ ，$\text{lcm}(x,y)$ 表示 $x$ 和 $y$ 的最小公倍数。

又观察发现，若要求 $n$ 最小，则 $d$ 必须取 $1$ ，因为 $S(n)$ 和 $k$ 总是一起成倍增加的，很容易证明 $n$ 的位数也是和 $d$ 成正比例的。

问题转化成了给定数位和 $S$ ，以及这个数内大于等于 $5$ 的数位 $k$ ，构造一个最小的数。

从后往前贪心地填 $9$ 就行了，如果填不了则填更小的，贪心的正确性比较显然。

这部分可以参考代码中的细节。

```cpp
#include <bits/stdc++.h>
using namespace std;

int x,k,a,b;
vector<int> ans;

inline int lcm(int a,int b)
{
    return a/__gcd(a,b)*b;
}

void solve()
{
    scanf("%d%d",&a,&b);
    if(2*b-a<0)
    {
        printf("0\n");
        return;
    }
    if(2*b==a)
    {
        printf("1\n");
        return;
    }
    x=lcm(2*b-a,9*b)/(2*b-a);
    k=lcm(2*b-a,9*b)/(9*b);
    if(x<5*k)
    {
        printf("0\n");
        return;
    }
    ans.clear();
    while(x)
        for(int i=9;i>=1;i--)
        {
            if(!k&&i>=5) continue; //如果k被用完了
            if((k-1)*5>x-i) continue; //如果这位填了i后无论如何也填不满k个了
            if(x<i) continue; //如果不够填i了
            if(i>=5) k--;
            ans.push_back(i),x-=i;
            break;
        }
    for(int i=ans.size()-1;i>=0;i--) printf("%d",ans[i]);
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