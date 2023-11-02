---
title: '「CometOJ Round #2 F」真实无妄她们的人生之路'
category: OI 题解
cover: 36.webp
tags:
  - 转置原理
  - 多项式多点求值
date: 2020-05-20 10:34:00
---

> 有 $n$ 种操作，第 $i$ 种操作使用后有 $p_i$ 的概率升级，$(1-p_i)$ 的概率不升级。
> 
> 进行若干次操作后，如果主人公的等级为 $i$，就能产生 $a_i$ 的贡献。
> 
> 对于每个 $i \in [1;n]$ 求出，使用 $j \neq i$ 的所有操作 $j$，主人公产生等级贡献的期望。
> 
> $n \leq 10^5$。

<!--more-->

## 题解

设 $C_i(x) = (1 - p_i + p_i x)$，$C(x) = \prod_{i=0}^{n-1} C_i(x)$

$$
ans_m = \sum_{i=0}^{n-1} a_i [x^i] \prod_{j \neq m} C_j(x)
$$

这是一个线性算法，其转移矩阵为 $D_{m,i} = [x^i] \prod_{j \neq m} C_j(x)$

考虑其转置 $D^T_{m,i} = [x^m] \prod_{j \neq i} C_j(x)$

我们有做法


$$
ans_m = \prod_{i=0}^{n-1} a_i [x^m] \prod_{j \neq i} C_j(x)
$$

$$
ANS(x) = \prod_{i=0}^n a_i \prod_{j \neq i} C_j(x)
$$

有类似 $k$ 次幂和的 $O(n \log^2 n)$ 做法，可以改写成其转置。

具体的，我们令 $A(x) = \sum_{i \ge 0} a_i x^i$，为线段树根节点，$\operatorname{mul}^T$ 到叶子节点即可。

一般做法可以参见 zsy 博客：https://www.cnblogs.com/zhoushuyu/p/10777808.html

大概是利用多项式点乘的性质把贡献拆开然后多点求值（

## 代码

```cpp
#include<bits/stdc++.h>
template<class T> inline void read(T &x){
  x=0; register char c=getchar(); register bool f=0;
  while(!isdigit(c))f^=c=='-',c=getchar();
  while(isdigit(c))x=x*10+c-'0',c=getchar(); if(f)x=-x;
}
template<class T> inline void print(T x){
  if(x<0)putchar('-'),x=-x;
  if(x>9)print(x/10); putchar(x%10+'0');
}
template<class T> inline void print(T x,char c){print(x),putchar(c);}
const int N=1e5+10,mod=998244353;
int n,m;
struct z {
  int x;
  z(int x=0):x(x){}
  friend inline z operator*(z a,z b){return (long long)a.x*b.x%mod;}
  friend inline z operator-(z a,z b){return (a.x-=b.x)<0?a.x+mod:a.x;}
  friend inline z operator+(z a,z b){return (a.x+=b.x)>=mod?a.x-mod:a.x;}
}a[N],p[N],ans[N],fac[N],ifac[N],inv[N];
inline z fpow(z a,int b){z s=1;for(;b;b>>=1,a=a*a)if(b&1)s=s*a;return s;}
void init(int n){
  n+=5,inv[0]=inv[1]=fac[0]=ifac[0]=1;
  for(int i=2;i<n;i++)inv[i]=(mod-mod/i)*inv[mod%i];
  for(int i=1;i<n;i++)fac[i]=fac[i-1]*i,ifac[i]=ifac[i-1]*inv[i];
}
namespace poly{//polynomial
  int len=1,rev[N<<2]; z w[N<<2];
  struct vec:std::vector<z>{
    using std::vector<z>::vector;
    inline void input(int n){resize(n);for(int i=0;i<n;i++)read(this->operator[](i).x);}
    inline void output(){for(int i=0;i<size();i++)print(this->operator[](i).x," \n"[i+1==size()]);}
  };
  int init(int n){
    int lim=1,k=0; while(lim<n)lim<<=1,++k;
    for(int i=0;i<lim;i++)rev[i]=(rev[i>>1]>>1)|((i&1)<<(k-1));
    for(;len<lim;len<<=1){
      z wn=fpow(3,(mod-1)/(len<<1)); w[len]=1;
      for(int i=1;i<len;i++)w[i+len]=w[i+len-1]*wn;
    }return lim;
  }
  void dft(vec &a,int lim){
    a.resize(lim);
    for(int i=0;i<lim;i++)if(i<rev[i])std::swap(a[i],a[rev[i]]);
    for(int len=1;len<lim;len<<=1)
      for(int i=0;i<lim;i+=(len<<1))
        for(int j=0;j<len;j++){
          z x=a[i+j],y=a[i+j+len]*w[j+len];
          a[i+j]=x+y,a[i+j+len]=x-y;
        }
  }
  void idft(vec &a,int lim){
    dft(a,lim),std::reverse(&a[1],&a[lim]); z inv=fpow(lim,mod-2);
    for(int i=0;i<lim;i++)a[i]=a[i]*inv;
  }
  inline vec mul(vec a,vec b,int l=-1){
    int len=~l?l:a.size()+b.size()-1,lim=init(a.size()+b.size()-1);
    dft(a,lim),dft(b,lim);
    for(int i=0;i<lim;i++)a[i]=a[i]*b[i];
    return idft(a,lim),a.resize(len),a;
  }
  namespace extra{
    using namespace poly;
    vec mulT(vec a,vec b){
      int al=a.size(),bl=b.size(),lim=init(al);
      dft(a,lim),std::reverse(b.begin(),b.end()),dft(b,lim);
      for(int i=0;i<lim;i++)a[i]=a[i]*b[i];
      return idft(a,lim),vec(&a[bl-1],&a[al]);
    }
  }
}
poly::vec F[N<<2],G[N<<2];
void dfs1(int u,int l,int r){
  if(l==r){F[u]={1-p[l],p[l]}; return;}
  dfs1(u<<1,l,(l+r)>>1),dfs1(u<<1|1,((l+r)>>1)+1,r);
  F[u]=poly::mul(F[u<<1],F[u<<1|1]);
}
void dfs2(int u,int l,int r){
  if(l==r){ans[l]=G[u][0]; return;}
  G[u<<1]=poly::extra::mulT(G[u],F[u<<1|1]);
  G[u<<1|1]=poly::extra::mulT(G[u],F[u<<1]);
  dfs2(u<<1,l,(l+r)>>1),dfs2(u<<1|1,((l+r)>>1)+1,r);
}
int main(){
#ifdef memset0
  freopen("1.in","r",stdin);
#endif
  read(n);
  for(int i=0;i<n;i++)read((int&)a[i]);
  for(int i=0,x,y;i<n;i++)read(x),read(y),p[i]=x*fpow(y,mod-2);
  dfs1(1,0,n-1),G[1]=poly::vec(a,a+n),dfs2(1,0,n-1);
  for(int i=0;i<n;i++)print(ans[i].x," \n"[i==n]);
}
```