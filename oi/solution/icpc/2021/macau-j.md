---
title: 「Macau 21 J」Colorful Tree
date: 2023-11-08 10:23:00
cover: 37.webp
tag:
  - 树分治
  - 线段树
  - 我的想法
publish: true
---

> 维护一个点有颜色的树，一开始只有一个编号为 $1$ 的节点，颜色为 $C$，要求支持以下操作 $q$ 次：
>
> 1. 给定 $x,c,d$，添加一个编号为 $n+1$ 颜色为 $c$ 的节点，向点 $x$ 连一条长度为 $d$ 的边
> 2. 给定 $x,c$，将点 $x$ 的颜色变成 $c$。
>
> 在每次操作后，你都需要在树上选两个颜色不同的点并最大化他们之间最短简单路径的长度，并输出这个长度。
>
> $1\le q\le 5 \times 10^5$。

<!-- more -->

## 题解

这题并不要求强制在线，添加节点并不会改变树的形状和边的长度，所以我们可以先离线所有操作，将两种操作都转化为对颜色的修改。

如果只要求求一次答案，我们可以考虑点分治，从每一个分治中心出发，搜索得到的信息是一个三元组 $(d,x,y)$，其中 $d$ 是分治中心到目标点的路径长度、$x$ 是目标点的颜色，$y$ 是目标点在 $d$ 的哪个子树里。这样问题就转化为，需要选择两个三元组 $(d_1,x_1,y_1)$ 和 $(d_2,x_2,y_2)$，最大化 $d_1+d_2$ 且满足 $x_1\neq x_2 \land y_1 \neq y_2$。这个问题比较经典，可以考虑类似 2023 年 CCPC Online F 的思路维护。

现在需要在每次修改操作后得到答案，可以把点分治换成点分树，每次修改颜色只需要重做 $O(\log n)$ 个分治中心的结果。注意到前面提到的问题我们只需要支持单点修改全局查询，且满足线段树信息的性质，可以考虑对于每个分治中心开一个线段树解决。

## 代码

```cpp
#include <bits/stdc++.h>
#define endl '\n'
using namespace std;
const int N = 5e5 + 9, M = 1 << 21;
int T, Q, C, n, root, all, fa[N], c[N], siz[N], mxp[N], par[N];
bool vis[N];
vector<int> que, op[N];
vector<pair<int, int>> G[N];
vector<pair<int, long long>> ins[N];
struct operation {
  int op, x, c, d;
} q[N];
void findroot(int u) {
  siz[u] = 1, mxp[u] = 0;
  for (auto [v, w] : G[u])
    if (v != fa[u] && !vis[v]) {
      fa[v] = u;
      findroot(v);
      siz[u] += siz[v];
      if (siz[v] > mxp[u]) mxp[u] = siz[v];
    }
  mxp[u] = max(mxp[u], all - siz[u]);
  if (root == 0 || mxp[u] < mxp[root]) root = u;
}
struct atom {
  long long w;
  int x, y;
  inline bool operator<(const atom &rhs) const { return w > rhs.w; }
};
vector<pair<int, atom>> dat;
struct atom_list {
  atom a[4];
  short len = 0;
} p[M];
long long calc(atom_list &u) {
  long long ans = 0;
  for (int i = 0; i < u.len; i++)
    for (int j = i + 1; j < u.len; j++)
      if (u.a[i].x != u.a[j].x && u.a[i].y != u.a[j].y && u.a[i].w >= 0 && u.a[j].w >= 0) {
        if (u.a[i].w + u.a[j].w > ans) ans = u.a[i].w + u.a[j].w;
      }
  return ans;
}
void maintain(atom_list &u, const atom_list &l, const atom_list &r) {
  int x1 = 0, x2 = 0, y1 = 0, y2 = 0, lstx = 0, lsty = 0;
  u.len = 0;
#define doi(_it)                             \
  atom it = _it;                             \
  if (!x1 && !y1) {                          \
    x1 = it.x, y1 = it.y, u.a[u.len++] = it; \
  } else {                                   \
    if (it.x != x1) {                        \
      if (!x2) x2 = it.y;                    \
      else if (x2 != it.y)                   \
        x2 = -1;                             \
    }                                        \
    if (it.y != y1) {                        \
      if (!y2) y2 = it.x;                    \
      else if (y2 != it.x)                   \
        y2 = -1;                             \
    }                                        \
    if (x2 != lstx || y2 != lsty) {          \
      u.a[u.len++] = it;                     \
    }                                        \
    lstx = x2, lsty = y2;                    \
    if (x2 == -1 && y2 == -1) break;         \
  }
  int i = 0, j = 0;
  while (i < l.len && j < r.len) {
    if (l.a[i].w > r.a[j].w) {
      doi(l.a[i]);
      i++;
    } else {
      doi(r.a[j]);
      j++;
    }
  }
  if (x2 != -1 || y2 != -1) {
    while (i < l.len) {
      doi(l.a[i]);
      i++;
    }
  }
  if (x2 != -1 || y2 != -1) {
    while (j < r.len) {
      doi(r.a[j]);
      j++;
    }
  }
}
void build(int u, int l, int r) {
  p[u].len = 0;
  if (l == r) {
    return;
  }
  int mid = (l + r) >> 1;
  build(u << 1, l, mid);
  build(u << 1 | 1, mid + 1, r);
}
void modify(int u, int k, const atom &x, int l, int r) {
  if (l == r) {
    p[u].a[0] = x;
    p[u].len = 1;
    return;
  }
  int mid = (l + r) >> 1;
  if (k <= mid) {
    modify(u << 1, k, x, l, mid);
  } else {
    modify(u << 1 | 1, k, x, mid + 1, r);
  }
  maintain(p[u], p[u << 1], p[u << 1 | 1]);
}
struct status {
  int len;
  long long ans;
  vector<int> ord;
  vector<atom> dat;
} f;
void dfs(int u, int cur) {
  siz[u] = 1;
  for (auto [v, w] : G[u])
    if (v != fa[u] && !vis[v]) {
      fa[v] = u;
      dfs(v, cur);
      siz[u] += siz[v];
    }
}
void collect(int u, long long len, int sub, int cur) {
  dat.push_back({u, {len, -1, sub}});
  siz[u] = 1;
  for (auto [v, w] : G[u])
    if (v != fa[u] && !vis[v]) {
      fa[v] = u;
      collect(v, len + w, sub, cur);
      siz[u] += siz[v];
    }
}
void build(int u, int src = 0) {
  par[u] = src;
  vis[u] = 1;
  que.push_back(u);
  for (auto [v, w] : G[u])
    if (!vis[v]) {
      fa[v] = u;
      dfs(v, u);
    }
  for (auto [v, w] : G[u])
    if (!vis[v]) {
      all = siz[v], root = 0, findroot(v), build(root, u);
    }
}
void solve(int u) {
  vis[u] = 1;
  vector<pair<int, atom>>{}.swap(dat);
  dat.push_back({u, {0, -1, u}});
  for (auto [v, w] : G[u])
    if (!vis[v]) {
      fa[v] = u;
      collect(v, w, v, u);
    }
  sort(dat.begin(), dat.end());
  f.ord.resize(dat.size());
  f.dat.resize(dat.size());
  f.len = dat.size();
  for (int i = 0; i < dat.size(); i++) {
    f.ord[i] = dat[i].first;
    f.dat[i] = dat[i].second;
  }
  build(1, 1, f.len);
  for (int i : op[u]) {
    int k = lower_bound(f.ord.begin(), f.ord.end(), q[i].x) - f.ord.begin() + 1;
    f.dat[k - 1].x = q[i].c;
    modify(1, k, f.dat[k - 1], 1, f.len);
    ins[i].emplace_back(u, calc(p[1]));
  }
  op[u].clear();
  op[u].shrink_to_fit();
}
namespace pool {
struct segment {
  long long ans;
} p[M];
void build(int u, int l, int r) {
  p[u].ans = 0;
  if (l == r) return;
  int mid = (l + r) >> 1;
  build(u << 1, l, mid);
  build(u << 1 | 1, mid + 1, r);
}
void modify(int u, int k, long long s, int l, int r) {
  if (l == r) {
    p[u].ans = s;
    return;
  }
  int mid = (l + r) >> 1;
  if (k <= mid) {
    modify(u << 1, k, s, l, mid);
  } else {
    modify(u << 1 | 1, k, s, mid + 1, r);
  }
  p[u].ans = max(p[u << 1].ans, p[u << 1 | 1].ans);
}
} // namespace pool
int main() {
#ifdef memset0
  freopen("1.in", "r", stdin);
#endif
  cin.tie(0)->sync_with_stdio(0);
  cin >> T;
  while (T--) {
    cin >> Q >> C;
    n = 1;
    G[1].clear();
    for (int i = 1; i <= Q; i++) {
      cin >> q[i].op >> q[i].x >> q[i].c;
      if (q[i].op == 0) {
        cin >> q[i].d;
        c[++n] = -1;
        G[n] = {{q[i].x, q[i].d}};
        G[q[i].x].emplace_back(n, q[i].d);
        q[i].x = n;
      }
    }
    fill_n(vis + 1, n, false);
    fa[1] = 0, root = 0, all = n, findroot(1), build(root);
    q[0].op = 1, q[0].c = C, q[0].x = 1;
    for (int i = 0; i <= Q; i++) {
      for (int u = q[i].x; u; u = par[u]) op[u].push_back(i);
    }
    fill_n(vis + 1, n, false);
    for (int u : que) solve(u);
    que.clear();
    pool::build(1, 1, n);
    for (int i = 0; i <= Q; i++) {
      for (const auto &[k, t] : ins[i]) pool::modify(1, k, t, 1, n);
      ins[i].clear();
      ins[i].shrink_to_fit();
      if (i) cout << pool::p[1].ans << endl;
    }
  }
  return 0;
}
```

