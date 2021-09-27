---
categories: [Tutorial]
tags: [DP]
---



[传送门](https://atcoder.jp/contests/abc159/tasks/abc159_f)

$n$ 个数，令 $f(l,r)$ 表示 $a$ 的子区间 $[l,r]$ 中，元素和为 $S$ 的子序列的个数，求 $\sum_{1\leq l \leq r \leq n}f(l,r)$ 。

如果朴素的搞只能得到一个 $O(n^3)$ 的做法。

仔细观察就能发现可以转化成一个普通的背包计数，只要每次 $dp_0=dp_0+1$ 即可，含义是每次都能新开一个 $L$ 作为开始。

反正爆踩官方题解了。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

const ll mod=998244353;
int n,s;
ll dp[3005],ans;

int main()
{
    scanf("%d%d",&n,&s);
    for(int i=1,x;i<=n;i++)
    {
        scanf("%d",&x);
        dp[0]++;
        for(int j=s;j>=x;j--)
            dp[j]=(dp[j]+dp[j-x])%mod;
        ans=(ans+dp[s])%mod;
    }
    printf("%lld",ans);
    return 0;
}
```

