---
title: 五边形数定理学习笔记
date: 2019-12-26 08:35:00
category: OI 算法
tags:
  - 五边形数
  - 组合数学
cover: 59.jpg
publish: true
---

五边形数生成函数即欧拉函数：

$$
\varphi(x) = \prod_{n=1}^\infty (1 - x^n)
$$

<!--more-->

## 推导

五边形数定理用来描述欧拉函数展开式的特性：

$$
\varphi(x) = \sum_{k=-\infty}^{\infty} (-1)^k x^{\frac {k(3k+1)} 2} = \sum_{k=0}^{\infty} (-1)^k x^{\frac {k(3k \pm 1)} 2}
$$

欧拉函数的倒数是划分数的生成函数：

$$
\frac 1 {\varphi(x)} = \sum_{i=0}^\infty p(i) x^i = \prod_{i=0}^\infty \sum_{j=0}^\infty x^{ij} = \prod_{i=0}^\infty \frac 1 {1 - x^i}
$$

<!--more-->

由定义我们可以得到：

$$
\varphi(x) \times \frac 1 {\varphi(x)} = 1 \Leftrightarrow
\left( \sum_{k=0}^{\infty} (-1)^k x^{\frac {k(3k \pm 1)} 2} \right) \left( \sum_{i=0}^\infty p(i) x^i \right) = 1
$$

通过这个我们可以得到划分数的递推式。

同时发现五边形数生成函数的项数是 $O(\sqrt n)$ 级别的，故我们可以在 $O(n \sqrt n)$ 的时间复杂度内求出划分数前 $n$ 项。

我们也可以通过这种方法求有限制时的划分数：现在限制划分的每个数的大小小于 $k$，则有

$$
F(x) = \prod_{i=0}^\infty \sum_{j=0}^{k-1} x^{ij} = \prod_{i=1}^\infty \frac {1-x^{ik}} {1 - x^i} = \frac {\varphi(x^k)} {\varphi(x)}
$$

则

$$
F(x) = \varphi(x^k) \times P(x)
$$

在先求出划分数的生成函数之后，我们同样可以在 $O(\sqrt n)$ 的时间复杂度内求出 $F(x)$ 的单项。

## 例题

#### HDU4651

```cpp
#include<bits/stdc++.h>
const int N=1e5+10,mod=1e9+7;
int T,n,m,f[N],g[N];
int main(){
	f[0]=1;
	for(int i=1;i*(3*i-1)/2<=100000;i++)g[m++]=i*(3*i-1)/2,g[m++]=i*(3*i+1)/2;
	for(int n=1;n<=100000;n++)for(int j=0;j<m&&g[j]<=n;j++)f[n]=(f[n]+(((j>>1)&1)?mod-1ll:1ll)*f[n-g[j]])%mod;
	for(scanf("%d",&T);T--;)scanf("%d",&n),printf("%d\n",f[n]);
}
```

#### HDU4658

```cpp
#include<bits/stdc++.h>
const int N=5e5+10,mod=1e9+7;
int T,n,m,k,res,f[N],g[N];
int main(){
	f[0]=1;
	for(int k=1;k*(3*k-1)/2<=100000;k++)g[m++]=k*(3*k-1)/2,g[m++]=k*(3*k+1)/2;
	for(int i=1;i<=100000;i++)for(int j=0;j<m&&g[j]<=i;j++)f[i]=(f[i]+f[i-g[j]]*((j>>1)&1?mod-1ll:1ll))%mod;
	for(scanf("%d",&T);T--;){
		scanf("%d%d",&n,&k),res=f[n];
		for(int i=0;i<m&&g[i]*k<=n;i++)res=(res+f[n-g[i]*k]*((i>>1)&1?1ll:mod-1ll))%mod;
		printf("%d\n",res);
	}
}
```

#### 证明

欧拉函数展开式的证明：https://blog.csdn.net/visit_world/article/details/52734860