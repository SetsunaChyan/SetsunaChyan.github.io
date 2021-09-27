---
title: "统计分析方法攻击维吉尼亚密码"
categories: [Information Security]
toc: true
---



正好学校在做实验，就丢上来了。

设密文串为 $S$ 。

如果 $S$ 无实际英文意义或 $\lvert S \rvert$ 很小或 $\lvert S \rvert$ 和密钥长度接近，那多半是解不出来的。

### 1. Kasiski 测试法确定密钥的长度 

根据概率学和实际经验，密文中连续3个字符多次出现基本对应了它对应的明文也可能是相同的，那么密钥最长就可能是所有该串出现位置的最大公约数，不妨设它为 $len$ ，那么密钥长度极有可能是  $len$ 的因子。为了减小偶然性对攻击的影响，枚举所有可能的连续 $3$ 个字符的组合的话复杂度是 $O(\min(26^3,\lvert S\rvert)^2)$ 的。只枚举 $k$ 次的话复杂度就是 $O(k\lvert S\rvert)$ 了，代价就是解密成功率降低，所以这部分视需求和经验而定。即使我们枚举了所有的组合，对于一个有实际意义的明文来说，得到的合法的 $len$ 的个数也不会太多。

```java
private Vector<Integer> Kasiski(String s)
{
    Vector<Integer> r=new Vector<>();
    Vector<Integer> v=new Vector<>();
    for(int i=0;i<s.length()-3;i++)
    {
        v.clear();
        String t=s.substring(i,i+3);
        for(int j=0;j<s.length()-3;j++)
            if(t.equals(s.substring(j,j+3))) v.add(j);
        if(v.size()<2) continue;
        int ret=0;
        for(int j=1;j<v.size();j++)
            ret=__gcd(ret,v.get(j)-v.get(j-1));
		r.add(ret);
    }
    return r;
}
```



### 2. 重合指数攻击

设一门语言由 $\lvert \Sigma \rvert$ 个字母组成，每个字母出现的概率为 $P_i$ ，出现次数为 $T_i$ ，则重合指数是指两个元素随机相同的概率之和，记作
$$
CI=\frac{\sum_{i=1}^{\lvert \Sigma \rvert}\tbinom{T_i}{2}}{\tbinom{\lvert S\rvert}{2}}
$$
英文中，一段文字是随机的话，有 $CI \approx 0.0385$ ，如果这段文字是有意义的，那么有 $CI \approx 0.065$ 。于是对于一个固定的 $len$ ，我们只需要把密文 $S$ 拆成长度为 $len$ 的若干段，统计所有段的 $CI$ 的平均值，越接近 $0.065$ 的就越有可能是真实的密钥长度。$len$ 的因子个数有个很松的上界 $O(\sqrt {len})$ ，所以这部分的复杂度是 $O(\sqrt{len} \lvert S\rvert)$ 的。 

```java
 private double getCI(String s,int len)
 {
    double ret=0;
    int cnt=0;
    for(int i=0;i+len<=s.length();i+=len)
    {
        double tmp=0;
        int vis[]=new int[26];
        for(int j=i;j<i+len;j++) vis[s.charAt(j)-'a']++;
        for(int j=0;j<26;j++) tmp+=(double)vis[j]/len*(vis[j]-1)/(len-1);
        ret+=tmp;
        cnt++;
    }
    return Math.abs(ret/cnt-0.065);
}
```



### 3. 字母频率分析

根据统计学，我们能得到每个字母在英文中的频率。我们假设明文是有意义的，那么明文就应该服从这个分布。对于 $len$ 长度的密钥，它的 $len$ 位都是独立的，且它们每位在密文 $S$ 中对应的也是服从这个分布的，可以看做是由 $len$ 个独立的凯撒密码所组成的。考虑如何破解单个凯撒密码，设内积
$$
R=\sum_{i=1}^{\lvert \Sigma \rvert} P_i \times Q_i其中
$$
其中 $P_i$ 是第 $i$ 种字符服从的出现概率，$Q_i$ 是凯撒解密后第 $i$ 种字符的出现频率。遍历字符集，得到 $R$ 最大时的字符取值，就是密钥某一位的值了。做一轮是 $O(\lvert \Sigma \rvert (\lvert S \rvert + len))$ 的。

> 随着枚举字符的变化 $Q_i$ 应该是循环左移的，我没想明白为什么 $R$ 取最大的时候是最优的，尽管感性理解上很对。

```java
static final double p[]={
    0.08167,0.01492,0.02782,0.04253,0.12702,0.02228,0.02015,0.06094,
    0.06966,0.00153,0.00772,0.04025,0.02406,0.06749,0.07507,0.01929,0.00095,
    0.05987,0.06327,0.09056,0.02758,0.00978,0.02360,0.00150,0.01974,0.00074
};

private String gao(String s,int len)
{
    StringBuilder t=new StringBuilder();
    for(int i=0;i<len;i++)
    {
        double mx=0;
        char c='a';
        for(int j=0;j<26;j++)
        {
            int vis[]=new int[26];
            int cnt=0;
            double tmp=0;
            for(int k=i;k<s.length();k+=len)
            {
                vis[(s.charAt(k)-'a'+j)%26]++;
                cnt++;
            }
            for(int k=0;k<26;k++) tmp+=(double)vis[k]/cnt*p[k];
            if(tmp>mx)
            {
                mx=tmp;
                c=(char)((26-j)%26+'a');
            }
        }
        t.append(c);
    }
    return decrypt(s,t.toString());
}
```



### 4. 人眼识别

选 $CI$ 那一步得分较高的人工筛选一下就行了。

