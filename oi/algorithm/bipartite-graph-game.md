---
title: 二分图博弈学习笔记
date: 2020-03-25 10:40:00
category: OI 算法
tags:
  - 二分图博弈
  - 网络流
cover: 54.jpg
publish: true
---

两人在一二分图上进行决策，初始状态为二分图的一个点，两人轮流沿边行动，不允许重复访问节点，无法移动者输。

这样的问题称为二分图博弈。

<!--more-->

## 二分图博弈

> **结论 1**
>
> 起点 $v$ 是先手必胜的当且仅当其在所有最大匹配上。

**必要性**：如果存在一个最大匹配 $M$ 使得 $v$ 不在其上，那么先手每次操作时，要么无路可走，要么走到了某个匹配点上（否则就找到了一条增广路），于是后手只需要每次走到 $M$ 中与之相匹配的节点即可。

**充分性**：当 $v$ 在所有最大匹配上时，我们取任意最大匹配 $M$，将 $v$ 移动到对应匹配点。可以证明此时 $M$ 删去点 $v$ 和走的匹配后仍是最大匹配，就回到了证明必要性中的情况。

有增广路定理的一下推论：对于无向图 $G$ 和任一最大匹配 $M$，点 $v$ 必定在最大匹配上当且仅当对于 $M$，$v$ 在匹配上且不存在以 $v$ 为一端的偶数长度交错路。（否则可以沿着偶数长度交错路平移匹配方案，就得到了一个 $v$ 不是匹配点的匹配 $M'$，矛盾）。

所以移动并删除点 $v$ 的图中也不存在增广路，充分性得到证明。

## Dinic 求解

考虑用一般网络流算法求解此类问题，我们需要优化判断一个点 $v$ 是否必定在所有最大匹配中的算法。

> **结论 2**
>
> 点 $v$ 必定在所有最大匹配中当且仅当在某个最大流中 $S$ 到 $v$ 的边有流量，且残量网络上不存在 $S$ 到 $i$ 的路径。

**充分性**：$v$ 在最大匹配 $M$ 中当且仅当 $S$ 到 $v$ 的边有流量。若残量网络上存在 $S$ 到 $v$ 的路径，则将这条路径和 $S$ 到 $v$ 的边取反即可得到一个新的，$S$ 到 $v$ 无流量的最大流。

**必要性**：假设存在一个 $S$ 到 $v$ 无流量的最大流，考虑该最大流与现在的最大流之差，每个点的流量必定平衡。于是由欧拉回路定理，该图必定可被分解为若干简单环的和，取其中包含 $S$ 到 $v$ 的边即可构造出一个残量网络上 $S$ 到 $v$ 的路径。

## 参考代码

```cpp
template<int N,int M> struct Dinic{
    int tot,s,t,cur[N],hed[N],dep[N],to[M<<1],val[M<<1],nxt[M<<1];
    Dinic(){tot=0,memset(hed,-1,sizeof(hed));}
    inline void add(int u,int v,int w){
        nxt[tot]=hed[u],to[tot]=v,val[tot]=w,hed[u]=tot++;
        nxt[tot]=hed[v],to[tot]=u,val[tot]=0,hed[v]=tot++;
    }
    bool bfs(){
        static int u,l,r,q[N]; memset(dep+1,0,t<<2),l=r=dep[s]=1,q[1]=s;
        while(l<=r&&(u=q[l++]))for(int i=hed[u];~i;i=nxt[i])
            if(val[i]&&!dep[to[i]])dep[to[i]]=dep[u]+1,q[++r]=to[i];
        return dep[t];
    }
    int dfs(int u,int d){
        if(u==t)return d; int s=0;
        for(int &i=cur[u];~i;i=nxt[i])if(val[i]&&dep[to[i]]==dep[u]+1)
            if(int e=dfs(to[i],std::min(d-s,val[i]))){s+=e,val[i]-=e,val[i^1]+=e;if(s==d)return s;}
        return s?s:dep[s]=0;
    }
    int dinic(){int r=0;while(bfs())memcpy(cur+1,hed+1,t<<2),r+=dfs(s,1e9);return r;}
};
template<int N,int M> struct Main:Dinic<N,M>{
    bool vis[N];
    void dfs(int u){
        vis[u]=0;
        for(int i=hed[u];~i;i=nxt[i])if(val[i]&&vis[to[i]])vis[to[i]]=1,dfs(to[i]);
    }
    void filter(){
        dfs(s);
        for(int i=hed[s];~i;i=nxt[i])vis[to[i]]&=!val[i];
    }
};
```



