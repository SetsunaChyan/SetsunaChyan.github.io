---
title: "2020 HDU Multi-University Training Contest 4"
categories: [Tutorial]
tags: [DP]
toc: true
classes: []
excerpt: "2020杭电多校第四场"
---



Rank 55，这场签到题比较多，运气也比较好。




|      | A    | B    | C         | D         | E         | F       | G      | H     | I         | J         | K       | L    |
| ---- | ---- | ---- | --------- | --------- | --------- | ------- | ------ | --------- | --------- | ------- | ---- | ---- |
| 赛时 |      |  :balloon:   | :balloon: | :balloon: | :balloon: | :eyes: | :balloon: | :eyes: | :eyes: |  | :balloon: | :balloon: |
| 赛后 |      |      |           |           |           |         |        |        |           |           |         |      |



### B. Blow up the Enemy 

队友写的。



### C. Contest of Rope Pulling (随机化+背包DP)

抽象后就是，两堆物品各 $n,m \leq 1000$ 个，每个物品有体积 ($\leq 1000$) 和价值 (正负都有) 两个维度，两堆中取等体积物品，使得价值最大。

比赛时队友直接暴力 $\text{DP}$ + 卡常硬搞过去了... 

事实上对物品 $\text{shuffle}$ 一下，前 $i$ 个物品中，第一组物品大小体积减去第二组物品大小体积的极值期望是 $O(\sqrt{n+m})$ 的，那么只记录体积差在 $[-\sqrt{n+m},\sqrt{n+m}]$ 内的状态就行了。

下面抄一个学军中学题解里给的证明学习一下。



设有 $2n$ 个整数 $a_1,a_2,\cdots,a_n$ ，满足 $\sum_{i=1}^n a_i=0 , \vert a_i\vert \leq V$ 。将数列 $\{a_i\}$ 等概率随机排列得到 $\{b_i\}$ 。

计算 $A=\max\{\sum_{i=1}^k b_i|k \in \{1,2,\cdots,2n\}\}$ 期望的上界。



$\textbf{Lemma 1.}$  若存在 $i,j$ 满足 $-V \lt a_i \leq a_j \lt V$ ，令 $a_i$ 减 $1$ 且 $a_j$ 加 $1$ 后，目标期望增加。

$\it{proof.}$    不妨设 $i$ 经过重排后位置为 $p$ ，$j$ 重排后位置为 $q$ ，有 $b_p \leq b_q$ 。

显然 $p \lt q$ 的排列数量和 $p \gt q$ 的排列数量是一样的。

考虑最大前缀和的下标 $k$ 。对于 $p \lt q$ ，若 $k \lt p$ 或 $k \geq p$ 时对目标期望都是没有影响的，$p \gt q$ 同理。

又有 $p \lt q$ 时，$k \in [p,q)$ 的概率小于 $p \gt q$ 时，$k \in [q,p)$ 的概率，增加的贡献大于减少的贡献，所以目标期望增加。

<div align="right">◼</div>  

那么经过不断的操作以后，$a_i$ 一定有恰好 $n$ 个 $V$ 和 $n$ 个 $-V$ ，所以我们可以只考虑 $a_i=\pm 1$ 的情况。

考虑如何计算目标值为 $m$ 的方案数。使用卡特兰数的对称法([解释戳这里](https://www.cnblogs.com/Tyouchie/p/11711540.html))，可以得到目标值 $\leq m$ 时的方案数为 $\binom{2n}{n}-\binom{2n}{n-m-1}$，目标值等于 $m$ 的方案数为 $\binom{2n}{n-m}-\binom{2n}{n-m-1}$。



### K. Kindergarten Physics (签到)

质量为 $m_1,m_2$ 的两个相距 $d$ 米的小球只受万有引力影响，求第 $t_0$ 秒时两球的距离。

注意到所有输入都非常小，$t$ 秒内移动的距离显然远小于精度误差，所以直接输出 $d$ 即可。

正常做的话做不来。

```cpp
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
typedef pair<int,int> pii;

double a,b,d,t;

void solve()
{
    scanf("%lf%lf%lf%lf",&a,&b,&d,&t);
    printf("%.10f\n",d);
}

int main()
{
    int _;
    scanf("%d",&_);
    while(_--) solve();
    return 0;
}
```



剩下的咕咕咕了，准备保研中。
