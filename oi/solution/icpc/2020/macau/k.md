---
title: 「Macau 20 K」Candy Ads
sync: /oi/solution/icpc/2020/macau/k.md
date: 2024-03-26 21:01:56
link: https://qoj.ac/contest/1538/problem/8315
publish: true
tags:
  - KDTree
  - 2-SAT
  - 分块
  - 我的想法
cover: sakura_ev0015a.png
---

> 主办方将在一个二维平面中投放广告。共有 $n$ 个广告可被投放，其中每个广告的都是左上角为 $(x_i,y_i)$ 的 $w\times h$ 矩形且出现时间为 $[l_i,r_i]$。同一时间内，任意两个被投放的广告不能有重叠面积。此外还有 $m$ 条限制 $(u_i,v_i)$ 表示在广告 $u_i$ 和广告 $v_i$ 中至少选择投放一条。判断是否存在一组合法的投放方案，如果存在的话给出方案。
>
> $1\le n\le 5\times 10^4$，$1\le m\le 10^5$，$1\le w,h,x_i,y_i,l_i,r_i \le 2000$。

<!-- more -->

（本文将介绍一个理论复杂度更优的做法，可惜由于常数问题未能在 QOJ 上通过全部测试集。）

## 题解

考虑用基于 Tarjan 做法的 2-SAT 的解决问题：给定的 $m$ 条限制可以方便的用有向边 $\lnot u_i\rightarrow v_i$ 和 $\lnot v_i \rightarrow u_i$ 表示。

关于 $n$ 条广告不能重叠的约束，将 $n$ 条广告按照时间右端点 $r_i$ 排序后，做时间的扫描线。某条广告 $i$ 和时间上与他有重叠的广告集合是一段连续的区间。对原序列以 $S$ 为块大小分块，将连续区间基于分块进行处理，

- 零散块部分：直接比较每条广告是否与第 $i$ 条广告在空间（那个二维平面）上有重合。这一部分贡献的时间复杂度共为 $O(N S)$。
- 整块部分：将 $i$ 这一信息离线到每一整块中，作为一次带关于广告 $i$ 的“询问”。

之后枚举每个块，上面有若干条离线上来的询问。考虑计算矩形 $[x_i,x_i+w-1] \times [y_i,y_i+h-1]$ 与 $[x_j,x_j+w-1]\times [y_j,y_j+h-1]$ 是否有交，等价于查询 $(x_i,y_i)$ 是否在矩形 $[x_j-w+2,x_j+w-2]\times [y_j-h+2,y_j+h-2]$ 内。将块中的所有**询问**插入到 K-D Tree 中，用块中的 $S$ 个**广告**作为查询，考虑基于 K-D Tree 的结构优化建图。

利用插入的节点是 $(x_1,y_1),(x_2,y_2) \ldots (x_n,y_n)$ 的子集这一性质，通过预先 $O(n\log n)$ 建树，这样每个块的建树过程只需要在原结构上“开关”若干点即可，可以将复杂度降到单次 $O(n)$ 的，而查询贡献的复杂度为 $O(S\sqrt{n})$。这一部分总时间复杂度为 $O\left(\dfrac{n^2}S + n\sqrt{n}\right)$。

取 $S=\sqrt{n}$，就可以得到时空复杂度均为 $O(n\sqrt{n})$ 的做法。

## 评价

我们的做法的虽然时空复杂度为 $O(n\sqrt{n})$，渐进意义上优于官方题解给出的 $O\left(\dfrac{n^2}{\omega}\right)$ 做法。但由于 K-D Tree、2-SAT 的常数较大，尽管博主进行了很多尝试，均未能在同时满足题目的是时、空约束的条件下通过本题。欢迎正在阅读这篇文章的你提出改进建议。

## 代码

这份代码在本机上可过，但在 QOJ 上会 [TLE on #27](https://qoj.ac/submission/356348)。如果调整块大小则可能 MLE。

```cpp
#include <bits/stdc++.h>
using namespace std;

#define cho(u, x) ((u) << 1 | (x))

const int N = 5e4 + 9, M = 5e6, S = 370;
int n, m, w, h, bln[N], tot, tmp[N], now[N];
bool ans[N], use[N];
vector<vector<int>> G;

void myassert(int x) {
  if (!x) {
    cout << "assertion failed" << endl;
    exit(-1);
  }
}

int tim, col_tot;
vector<int> dfn, low, col;
vector<char> ins;
int top, stk[M];
void tarjan(int u) {
  dfn[u] = low[u] = ++tim;
  ins[u] = true, stk[++top] = u;
  for (int v : G[u])
    if (!dfn[v]) {
      tarjan(v);
      low[u] = min(low[u], low[v]);
    } else if (ins[v]) {
      low[u] = min(low[u], dfn[v]);
    }
  if (dfn[u] == low[u]) {
    ++col_tot;
    do {
      col[stk[top]] = col_tot;
      ins[stk[top]] = false;
    } while (stk[top--] != u);
  }
}

struct point {
  int x, y;
};

struct atom {
  int l, r, id;
  point u;
  inline bool operator<(const atom &rhs) const { return r == rhs.r ? l < rhs.l : r < rhs.r; }
} a[N];

int newnode() {
  ++tot;
  if (G.size() <= static_cast<size_t>(tot * 2 + 10)) {
    G.resize(G.size() + N);
  }
  return tot;
}

void addedge(int u, int v) {
  // fprintf(stderr, ">>> add edge %d[%d] %d[%d] \n", u >> 1, u & 1, v >> 1, v & 1);
  G[u].emplace_back(v);
}

void check(int i, int j) {
  if (a[i].u.x < a[j].u.x) {
    if (a[i].u.x + w <= a[j].u.x) return;
  } else {
    if (a[j].u.x + w <= a[i].u.x) return;
  }
  if (a[i].u.y < a[j].u.y) {
    if (a[i].u.y + h <= a[j].u.y) return;
  } else {
    if (a[j].u.y + h <= a[i].u.y) return;
  }
  addedge(cho(i, 1), cho(j, 0));
  addedge(cho(j, 1), cho(i, 0));
}

int d, rt;
struct node {
  int x[2], l[2], r[2], ch[2], src, nod;
  bool operator<(const node &rhs) const { return x[d] < rhs.x[d]; }
} kdt[N];

inline void maintain(int u) {
  kdt[u].l[0] = kdt[u].x[0];
  if (kdt[u].ch[0] && kdt[kdt[u].ch[0]].l[0] < kdt[u].l[0]) kdt[u].l[0] = kdt[kdt[u].ch[0]].l[0];
  if (kdt[u].ch[1] && kdt[kdt[u].ch[1]].l[0] < kdt[u].l[0]) kdt[u].l[0] = kdt[kdt[u].ch[1]].l[0];
  kdt[u].l[1] = kdt[u].x[1];
  if (kdt[u].ch[0] && kdt[kdt[u].ch[0]].l[1] < kdt[u].l[1]) kdt[u].l[1] = kdt[kdt[u].ch[0]].l[1];
  if (kdt[u].ch[1] && kdt[kdt[u].ch[1]].l[1] < kdt[u].l[1]) kdt[u].l[1] = kdt[kdt[u].ch[1]].l[1];
  kdt[u].r[0] = kdt[u].x[0];
  if (kdt[u].ch[0] && kdt[kdt[u].ch[0]].r[0] > kdt[u].r[0]) kdt[u].r[0] = kdt[kdt[u].ch[0]].r[0];
  if (kdt[u].ch[1] && kdt[kdt[u].ch[1]].r[0] > kdt[u].r[0]) kdt[u].r[0] = kdt[kdt[u].ch[1]].r[0];
  kdt[u].r[1] = kdt[u].x[1];
  if (kdt[u].ch[0] && kdt[kdt[u].ch[0]].r[1] > kdt[u].r[1]) kdt[u].r[1] = kdt[kdt[u].ch[0]].r[1];
  if (kdt[u].ch[1] && kdt[kdt[u].ch[1]].r[1] > kdt[u].r[1]) kdt[u].r[1] = kdt[kdt[u].ch[1]].r[1];
}

int build(int l, int r, int k) {
  if (l > r) return 0;
  int mid = (l + r) >> 1;
  d = k;
  nth_element(kdt + l, kdt + mid, kdt + r + 1);
  kdt[mid].ch[0] = build(l, mid - 1, k ^ 1);
  kdt[mid].ch[1] = build(mid + 1, r, k ^ 1);
  maintain(mid);
  // fprintf(stderr, "build tree u=%d lc=%d rc=%d\n", mid, kdt[mid].ch[0], kdt[mid].ch[1]);
  return mid;
}

void query(int u, const node &cur) {
  if (cur.l[0] <= kdt[u].l[0] && cur.l[1] <= kdt[u].l[1] && kdt[u].r[0] <= cur.r[0] && kdt[u].r[1] <= cur.r[1]) {
    if (kdt[u].nod) {
      addedge(cho(cur.src, 1), cho(kdt[u].nod, 0));
      addedge(cho(kdt[u].nod, 1), cho(cur.src, 0));
    }
    return;
  }
  if (cur.l[0] > kdt[u].r[0] || cur.l[1] > kdt[u].r[1] || cur.r[0] < kdt[u].l[0] || cur.r[1] < kdt[u].l[1]) {
    return;
  }
  if (use[kdt[u].src] && cur.l[0] <= kdt[u].x[0] && cur.l[1] <= kdt[u].x[1] && kdt[u].x[0] <= cur.r[0] && kdt[u].x[1] <= cur.r[1]) {
    addedge(cho(cur.src, 1), cho(kdt[u].src, 0));
    addedge(cho(kdt[u].src, 1), cho(cur.src, 0));
  }
  query(kdt[u].ch[0], cur);
  query(kdt[u].ch[1], cur);
}

void assign(int u) {
  if (kdt[u].ch[0]) assign(kdt[u].ch[0]);
  if (kdt[u].ch[1]) assign(kdt[u].ch[1]);
  int lc = kdt[u].ch[0], rc = kdt[u].ch[1];
  if (kdt[lc].nod && kdt[rc].nod) {
    kdt[u].nod = newnode();
    addedge(cho(kdt[u].nod, 0), cho(kdt[lc].nod, 0));
    addedge(cho(kdt[lc].nod, 1), cho(kdt[u].nod, 1));
    addedge(cho(kdt[u].nod, 0), cho(kdt[rc].nod, 0));
    addedge(cho(kdt[rc].nod, 1), cho(kdt[u].nod, 1));
    if (use[kdt[u].src]) {
      addedge(cho(kdt[u].nod, 0), cho(kdt[u].src, 0));
      addedge(cho(kdt[u].src, 1), cho(kdt[u].nod, 1));
    }
  } else if (kdt[lc].nod) {
    if (use[kdt[u].src]) {
      kdt[u].nod = newnode();
      G[cho(kdt[u].nod, 0)] = {cho(kdt[lc].nod, 0), cho(kdt[u].src, 0)};
      addedge(cho(kdt[lc].nod, 1), cho(kdt[u].nod, 1));
      addedge(cho(kdt[u].src, 1), cho(kdt[u].nod, 1));
    } else {
      kdt[u].nod = kdt[lc].nod;
    }
  } else if (kdt[rc].nod) {
    if (use[kdt[u].src]) {
      kdt[u].nod = newnode();
      G[cho(kdt[u].nod, 0)] = {cho(kdt[rc].nod, 0), cho(kdt[u].src, 0)};
      addedge(cho(kdt[rc].nod, 1), cho(kdt[u].nod, 1));
      addedge(cho(kdt[u].src, 1), cho(kdt[u].nod, 1));
    } else {
      kdt[u].nod = kdt[rc].nod;
    }
  } else {
    if (use[kdt[u].src]) {
      kdt[u].nod = kdt[u].src;
    } else {
      kdt[u].nod = 0;
    }
  }
  // fprintf(stderr, "assign %d use=%d src=%d nod=%d\n", u, use[kdt[u].src], kdt[u].src, kdt[u].nod);
}

struct block {
  int l, r;
  vector<int> q;
  void solve() {
    // fprintf(stderr, "solve block %d %d\n", l, r);
    for (int i = 1; i <= n; i++) use[i] = false;
    for (int x : q) use[x] = true;
    assign(rt);
    node cur;
    for (int i = l; i <= r; i++) {
      cur.l[0] = a[i].u.x - (w - 1), cur.r[0] = a[i].u.x + (w - 1);
      cur.l[1] = a[i].u.y - (h - 1), cur.r[1] = a[i].u.y + (h - 1);
      cur.src = i, cur.nod = -1;
      query(rt, cur);
    }
  }
} b[N / S + 9];

int main() {
#ifdef memset0
  // freopen("K.in", "r", stdin);
  // freopen("K-big.txt", "r", stdin);
  freopen("K-data.txt", "r", stdin);
#endif
  cin.tie(0)->sync_with_stdio(0);
  // vector<pair<int, int>> lim;
  cin >> n >> w >> h;
  for (int i = 1; i <= n; i++) {
    cin >> a[i].l >> a[i].r >> a[i].u.x >> a[i].u.y;
    a[i].id = i;
  }
  sort(a + 1, a + n + 1);
  // for (int i = 1; i <= n; i++) fprintf(stderr, "a[%d] => l=%d r=%d x=%d y=%d id=%d\n", i, a[i].l, a[i].r, a[i].u.x, a[i].u.y, a[i].id);
  for (int i = 1; i <= n; i++) {
    now[a[i].id] = i;
  }
  cin >> m;
  tot = n;
  G.resize(tot * 2 + N);
  for (int u, v, i = 1; i <= m; i++) {
    cin >> u >> v;
    // lim.emplace_back(u, v);
    u = now[u];
    v = now[v];
    addedge(cho(u, 0), cho(v, 1));
    addedge(cho(v, 0), cho(u, 1));
  }

  for (int i = 1; i <= n; i++) {
    kdt[i].src = i;
    kdt[i].x[0] = a[i].u.x;
    kdt[i].x[1] = a[i].u.y;
  }
  rt = build(1, n, 0);

  for (int i = 1; i <= n; i++) {
    bln[i] = i / S + 1;
    if (!b[bln[i]].l) b[bln[i]].l = i;
    b[bln[i]].r = i;
    // cout << bln[i] << " \n"[i == n];
  }
  for (int i = 1; i <= n; i++) tmp[i] = a[i].r;
  for (int ql, qr, i = 1; i <= n; i++) {
    ql = lower_bound(tmp + 1, tmp + i, a[i].l) - tmp;
    qr = i - 1;
    if (ql <= qr) {
      // fprintf(stderr, "solve [ql=%d[%d] qr=%d[%d]] i=%d\n", ql, bln[ql], qr, bln[qr], i);
      if (bln[ql] == bln[qr]) {
        for (int j = ql; j <= qr; j++) check(j, i);
      } else {
        for (int j = ql; j <= b[bln[ql]].r; j++) check(j, i);
        for (int j = b[bln[qr]].l; j <= qr; j++) check(j, i);
        for (int k = bln[ql] + 1; k < bln[qr]; k++) {
          b[k].q.emplace_back(i);
        }
      }
    }
  }
  cerr << "clock = " << clock() / (double)CLOCKS_PER_SEC << endl;
  for (int i = 1; i <= bln[n]; i++) {
    b[i].solve();
  }
#ifdef memset0
  int edg = 0;
  for (int i = cho(1, 0); i <= cho(tot, 1); i++) edg += G[i].size();
  cerr << "tot = " << tot << "; edg = " << edg << endl;
#endif

  dfn.resize((tot + 1) << 1);
  low.resize((tot + 1) << 1);
  col.resize((tot + 1) << 1);
  ins.resize((tot + 1) << 1);
  cerr << "clock = " << clock() / (double)CLOCKS_PER_SEC << endl;
  for (int i = cho(1, 0); i <= cho(tot, 1); i++)
    if (!dfn[i]) {
      tarjan(i);
    }
  cerr << "clock = " << clock() / (double)CLOCKS_PER_SEC << endl;
  for (int i = 1; i <= tot; i++)
    if (col[cho(i, 0)] == col[cho(i, 1)]) {
      cout << "No" << endl;
      return 0;
    }
  cout << "Yes" << endl;
  for (int i = 1; i <= n; i++) {
    ans[a[i].id] = col[cho(i, 0)] > col[cho(i, 1)];
  }
  // for (int i = 1; i <= n; i++) cerr << col[cho(i, 0)] << " \n"[i == n];
  // for (int i = 1; i <= n; i++) cerr << col[cho(i, 1)] << " \n"[i == n];
#ifdef ONLINE_JUDGE
  for (int i = 1; i <= n; i++) cout << ans[i];
  cout << endl;
#endif
#ifdef memset0
  cerr << "clock = " << clock() / (double)CLOCKS_PER_SEC << endl;
#endif
}
```
