---
title: 「Ynoi2019」美好的每一天~ 不连续的存在
category: OI 题解
cover: 17.webp
tags:
  - 分块
  - 莫队
date: 2020-06-26 00:07:00
publish: true
---

> 给数组 $A$ 和 $n$ 个节点的树，每个点有一个 $1$ 到 $x$ 颜色。
>
> $m$ 次查询，每次查询树上只保留 $[l,r]$ 内的所有节点，设一个极大连通块中出现奇数次数的颜色个数为 $t$，则其对答案的贡献为 $A_t$ ，即答案是所有连通块贡献的和，询问相互独立。
>
> $1\leq n,m\leq 10^5$，$1\leq x,A_i \leq 10^4$。

<!-- more -->

## 题解

~~退役选手被 lxl 抓过来写题解~~

考虑用单增莫队维护，想了想容易发现复杂度不对。

莫队的端点移动需要启发式合并维护信息，而启发式合并的复杂度基于我们可以把势能均摊，如果我们在分块时基于势能呢？

单增莫队的复杂度主要产生于以下两个部分（假设把询问分为 $S$ 块）：

- 左端点左移：对于每个端点都会贡献 $O(S)$，故这一部分的总势能是 $O(S n \sqrt n)$ 的
- 右端点右移：每个块内的询问会对每个块内的端点贡献 $O(1)$ 次，可以通过构造询问分布，卡满这部分的势能和。
- 回滚后缀的右端点右移操作：回滚操作于增加操作复杂度相同。

~~有没有救呢？当然是有的。~~注意左端点的移动势能是只和块数相关的。瓶颈在于右端点的移动：对于一个区间 $[l,r]$，端点 $i$ 在右端点移动时贡献的势能为 $A_i$，则一次经过该区间的询问产生的势能至多为 $\sum_{i=l+1}^r A_i$。

理清思路后题解就很显然了，我们先处理出每个端点的势能 $B_i$（在 $1 \ldots (i-1)$ 后插入 $i$ 贡献的势能，上文提到的势能 $A_i$ 是一定有 $A_i \leq B_i$ 的）。然后从右到左对势能分块。

如果加入当前端点后当前栈内端点的势能和大于 $\sqrt {n} \log n$ 就将栈内元素分为一块。由于所有被分配到这个块的询问都是经过块的不会贡献势能的左端点，而其余端点贡献的势能和是一定小于 $\sqrt n \log n$ 的。

总时间复杂度 $O((n+Q)\sqrt n \log n)$。

## 卡常技巧

本题卡常的一比，毒瘤 lxl（（

1. 手动调整块大小
2. 作栈/队列功能的 `std::vector<T>` 换成手写（不知道为什么我做到这这步就过了）
3. 手写 bitset（我鸽了）
4. 手写可合并的 vector（然而我手写了一个还打不过 `std::vector<T>`，我已经报警了）

## 代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#pragma GCC target("avx")
#pragma GCC target("popcnt")
#pragma GCC optimize(2)
#pragma GCC optimize(3)
#pragma GCC optimize("Ofast")
#pragma GCC optimize("unroll-loops")
#pragma GCC optimize("inline-functions")
#pragma GCC optimize("no-stack-protector")
#pragma GCC optimize("inline-functions-called-once")
#define log(...) fprintf(stderr, __VA_ARGS__)
#define debug log("\33[2mPassing [%s] in LINE %d\33[0m\n", __FUNCTION__, __LINE__);
const int S = 1 << 21;
char ibuf[S], *iS, *iT, obuf[S], *oS = obuf, *oT = oS + S - 1;
#define flush() (fwrite(obuf, 1, oS - obuf, stdout), oS = obuf, void())
#define getchar() (iS == iT ? (iT = (iS = ibuf) + fread(ibuf, 1, S, stdin), (iS == iT ? EOF : *iS++)) : *iS++)
#define putchar(x) (*oS++ = (x), oS == oT ? flush() : void())
struct Flusher_ {
  ~Flusher_() { flush(); }
} flusher_;
template <class T> inline void read(T &x) {
  x = 0;
  int c = getchar(), f = 0;
  while (!isdigit(c)) f ^= c == '-', c = getchar();
  while (isdigit(c)) x = x * 10 + c - '0', c = getchar();
  if (f) x = -x;
}
template <class T> inline void print(T x) {
  if (x < 0) putchar('-'), x = -x;
  if (x > 9) print(x / 10);
  putchar(x % 10 + '0');
}
template <class T> inline void print(T x, char c) { print(x), putchar(c); }
const int N = 1e5 + 9, M = 1e4 + 9, S1 = 256, S2 = 4;
int n, m, x, blocks, a[N], c[N], bln[N], anc[N], pre[N], siz[N], ans[N];
vector<int> G[N];
template <class T, int N> struct static_vector {
  T a[N];
  size_t top;
  inline T &operator[](size_t k) { return a[k]; }
  inline T operator[](size_t k) const { return a[k]; }
  inline T &back() { return a[top - 1]; }
  inline T back() const { return a[top - 1]; }
  inline void clear() { top = 0; }
  inline void pop_back() { top--; }
  inline void push_back(const T &e) { a[top++] = e; }
  inline size_t size() { return top; }
};
struct query {
  int l, r, id;
};
vector<query> q;
namespace tree {
int ans, delta, anc[N];
bool roll, mrk[N];
bitset<M> mem[N / S2 + 5];
static_vector<size_t, (N / S2 + 5)> rub;
static_vector<tuple<int, int, int>, (N << 1)> history;
struct unicom_block {
  int id, cnt, bitset, key[S2];
  vector<int> vec;
  inline unicom_block() { bitset = -1, memset(key, -1, sizeof(key)); }
  inline void clear() {
    if (~bitset) mem[bitset].reset(), rub.push_back(bitset), bitset = -1;
    cnt = 0, vec.clear(), memset(key, -1, sizeof(key));
  }
  inline void pushup(int k) {
    ans -= a[cnt];
    if (~bitset) {
      if (mem[bitset][k]) {
        mem[bitset][k] = 0, --cnt;
      } else {
        mem[bitset][k] = 1, ++cnt;
      }
    } else {
      for (int i = 0; i < S2; i++)
        if (key[i] == k) {
          key[i] = -1, --cnt;
          goto out;
        }
      for (int i = 0; i < S2; i++)
        if (key[i] == -1) {
          key[i] = k, ++cnt;
          goto out;
        }
      bitset = rub.back(), rub.pop_back();
      mem[bitset][k] = 1, ++cnt;
      for (int i = 0; i < S2; i++) mem[bitset][key[i]] = 1;
    }
  out:
    ans += a[cnt];
  }
} uni[N];
inline void merge(unicom_block &u, unicom_block &v) {
  for (int i : u.vec) v.pushup(i);
}
int find(int x) { return x == anc[x] ? x : find(anc[x]); }
void reset() {
  ans = roll = 0;
  for (int i = 1; i <= n; i++) {
    anc[i] = i;
    if (mrk[i]) uni[i].clear(), mrk[i] = 0;
  }
}
void add(int x) {
  ans += a[0];
  if (roll) delta += a[0];
  if (roll) history.push_back({0, x, 0});
  mrk[x] = 1;
  uni[x].vec.push_back(c[x]), uni[x].pushup(c[x]);
  for (int i = 0; i < G[x].size(); i++)
    if (mrk[G[x][i]]) {
      int y = G[x][i], u = find(x), v = find(y);
      if (uni[u].vec.size() > uni[v].vec.size()) swap(u, v);
      if (roll) history.push_back({u, v, anc[u]});
      merge(uni[u], uni[v]);
      uni[v].vec.insert(uni[v].vec.end(), uni[u].vec.begin(), uni[u].vec.end()), anc[u] = v;
      ans -= a[uni[u].cnt];
      if (roll) delta -= a[uni[u].cnt];
    }
}
int query() { return ans; }
void rollback() {
  ans -= delta, delta = 0;
  for (int u, v, t, i = (int)history.size() - 1; i >= 0; i--) {
    tie(u, v, t) = history[i];
    if (u) {
      anc[u] = t;
      for (int i : uni[u].vec) uni[v].vec.pop_back();
      merge(uni[u], uni[v]);
    } else {
      mrk[v] = 0, uni[v].pushup(c[v]), uni[v].clear();
    }
  }
  history.clear();
}
} // namespace tree
struct block {
  int id, l, r;
  vector<query> q;
  void init() {
    for (int i = l; i <= r; i++) bln[i] = id;
  }
  void solve() {
    sort(q.begin(), q.end(), [](const query &a, const query &b) { return a.l > b.l; });
    tree::reset();
    int cur = l, i;
    for (const auto &it : q) {
      while (it.l <= cur) tree::add(cur--);
      for (tree::roll ^= 1, i = l + 1; i <= it.r; i++) tree::add(i);
      ans[it.id] = tree::query();
      tree::roll ^= 1, tree::rollback();
    }
  }
} block[S1 << 1];
void build() {
  function<int(int)> find = [&](int x) { return anc[x] == x ? x : anc[x] = find(anc[x]); };
  for (int i = 1; i <= n; i++) anc[i] = i, siz[i] = 1;
  for (int u = 1; u <= n; u++) {
    for (int v : G[u])
      if (v < u) {
        int fu = find(u), fv = find(v);
        if (fu == fv) continue;
        if (siz[fu] > siz[fv]) swap(fu, fv);
        pre[u] += siz[fu], anc[fu] = fv, siz[fv] += siz[fu];
      }
    pre[u] += G[u].size();
  }
  int S = accumulate(pre + 1, pre + n + 1, 0) / S1 + 1;
  fprintf(stderr, "> block limit = %d\n", S);
  vector<tuple<int, int, int>> seq = {{n + 1, n, 0}};
  for (int i = n; i >= 1; i--) {
    get<0>(seq.back())--, get<2>(seq.back()) += pre[i];
    if (i == 1 || get<2>(seq.back()) + pre[i] > S) seq.push_back({i, i - 1, 0});
  }
  seq.pop_back(), reverse(seq.begin(), seq.end()), blocks = seq.size();
  for (int _, i = 0; i < blocks; i++) {
    tie(block[i].l, block[i].r, _) = seq[i];
    block[i].id = i, block[i].init();
  }
}
void solve() {
  for (int i = 1; i <= n; i++) tree::uni[i].id = i;
  for (const auto &it : q)
    if (bln[it.l] == bln[it.r]) {
      tree::reset();
      for (int i = it.l; i <= it.r; i++) tree::add(i);
      ans[it.id] = tree::query();
    } else {
      block[bln[it.r]].q.push_back(it);
    }
  for (int i = 0; i < blocks; i++) block[i].solve();
}
int main() {
#ifdef memset0
  freopen("1.in", "r", stdin);
  freopen("1.out", "w", stdout);
#endif
  for (int i = 0; i < N / S2 + 5; i++) tree::rub.push_back(i);
  read(n), read(m), read(x);
  for (int i = 1; i <= n; i++) read(c[i]), --c[i];
  for (int u, v, i = 1; i < n; i++) {
    read(u), read(v);
    G[u].push_back(v), G[v].push_back(u);
  }
  for (int i = 0; i <= x; i++) read(a[i]);
  build();
  for (int l, r, i = 1; i <= m; i++) {
    read(l), read(r);
    q.push_back({l, r, i});
  }
  solve();
  for (int i = 1; i <= m; i++) print(ans[i], '\n');
}
```

## 手写 vector 且没通过的代码

```cpp
#include <bits/stdc++.h>
using namespace std;
#pragma GCC target("avx")
#pragma GCC target("popcnt,tune=native")
#pragma GCC optimize(2)
#pragma GCC optimize(3)
#pragma GCC optimize("Ofast")
#pragma GCC optimize("unroll-loops")
#pragma GCC optimize("inline-functions")
#pragma GCC optimize("no-stack-protector")
#pragma GCC optimize("inline-functions-called-once")
const int S = 1 << 21;
char ibuf[S], *iS, *iT, obuf[S], *oS = obuf, *oT = oS + S - 1;
#define flush() (fwrite(obuf, 1, oS - obuf, stdout), oS = obuf, void())
#define getchar() (iS == iT ? (iT = (iS = ibuf) + fread(ibuf, 1, S, stdin), (iS == iT ? EOF : *iS++)) : *iS++)
#define putchar(x) (*oS++ = (x), oS == oT ? flush() : void())
struct Flusher_ {
  ~Flusher_() { flush(); }
} flusher_;
template <class T> inline void read(T &x) {
  x = 0;
  int c = getchar(), f = 0;
  while (!isdigit(c)) f ^= c == '-', c = getchar();
  while (isdigit(c)) x = x * 10 + c - '0', c = getchar();
  if (f) x = -x;
}
template <class T> inline void print(T x) {
  if (x < 0) putchar('-'), x = -x;
  if (x > 9) print(x / 10);
  putchar(x % 10 + '0');
}
template <class T> inline void print(T x, char c) { print(x), putchar(c); }
const int N = 1e5 + 9, M = 1e4 + 9, S1 = 256, S2 = 4;
int n, m, x, blocks, a[N], c[N], bln[N], anc[N], pre[N], siz[N], ans[N];
vector<int> G[N];
struct query {
  int l, r, id;
};
vector<query> q;
template <class T, int N> struct static_vector {
  T a[N];
  size_t top;
  inline T &operator[](size_t k) { return a[k]; }
  inline T operator[](size_t k) const { return a[k]; }
  inline T &back() { return a[top - 1]; }
  inline T back() const { return a[top - 1]; }
  inline void clear() { top = 0; }
  inline void pop_back() { top--; }
  inline void push_back(const T &e) { a[top++] = e; }
  inline size_t size() { return top; }
};
int mempool[N << 5], *mempointer = mempool;
int *memselect(int n) {
  int *res = mempointer;
  mempointer += n;
  return res;
}
struct combinable_vector {
  int lim;
  int *l, *r;
  inline size_t size() const { return r - l; }
  inline void clear() { r = l; }
  inline void assign() { l = r = memselect(lim = 8); }
  inline void re_assign() {
    int *tl = l, *tr = r;
    lim <<= 3, l = memselect(lim), r = l + (tr - tl);
    memcpy(l, tl, size() << 2);
  }
  inline void push_back(int x) {
    if (size() == lim) re_assign();
    *(r++) = x;
  }
  inline void pop_back() { r--; }
  inline void concat(const combinable_vector &rhs) {
    if (size() + rhs.size() > lim) re_assign();
    memcpy(r, rhs.l, rhs.size() << 2), r += rhs.size();
  }
  inline void concat_reverse(const combinable_vector &rhs) { r -= rhs.size(); }
};
namespace tree {
int ans, delta, anc[N];
bool roll, mrk[N];
bitset<M> mem[N / S2 + 5];
static_vector<size_t, (N / S2 + 5)> rub;
static_vector<tuple<int, int, int>, (N << 1)> history;
struct unicom_block {
  int id, cnt, bitset, key[S2];
  combinable_vector vec;
  inline unicom_block() { bitset = -1, memset(key, -1, sizeof(key)), vec.assign(); }
  inline void clear() {
    if (~bitset) mem[bitset].reset(), rub.push_back(bitset), bitset = -1;
    cnt = 0, vec.clear(), memset(key, -1, sizeof(key));
  }
  inline void pushup(int k) {
    ans -= a[cnt];
    if (~bitset) {
      if (mem[bitset][k]) {
        mem[bitset][k] = 0, --cnt;
      } else {
        mem[bitset][k] = 1, ++cnt;
      }
    } else {
      for (int i = 0; i < S2; i++)
        if (key[i] == k) {
          key[i] = -1, --cnt;
          goto out;
        }
      for (int i = 0; i < S2; i++)
        if (key[i] == -1) {
          key[i] = k, ++cnt;
          goto out;
        }
      bitset = rub.back(), rub.pop_back();
      mem[bitset][k] = 1, ++cnt;
      for (int i = 0; i < S2; i++) mem[bitset][key[i]] = 1;
    }
  out:
    ans += a[cnt];
  }
} uni[N];
inline void merge(unicom_block &u, unicom_block &v) {
  for (int *it = u.vec.l; it != u.vec.r; it++) v.pushup(*it);
}
int find(int x) { return x == anc[x] ? x : find(anc[x]); }
void reset() {
  ans = roll = 0;
  for (int i = 1; i <= n; i++) {
    anc[i] = i;
    if (mrk[i]) uni[i].clear(), mrk[i] = 0;
  }
  mempointer = mempool;
  for (int i = 1; i <= n; i++) uni[i].vec.assign();
}
void add(int x) {
  ans += a[0];
  if (roll) delta += a[0];
  if (roll) history.push_back({0, x, 0});
  mrk[x] = 1, uni[x].pushup(c[x]);
  uni[x].vec.push_back(c[x]);
  for (int i = 0; i < G[x].size(); i++)
    if (mrk[G[x][i]]) {
      int y = G[x][i], u = find(x), v = find(y);
      if (uni[u].vec.size() > uni[v].vec.size()) swap(u, v);
      if (roll) history.push_back({u, v, anc[u]});
      merge(uni[u], uni[v]);
      anc[u] = v;
      uni[v].vec.concat(uni[u].vec);
      ans -= a[uni[u].cnt];
      if (roll) delta -= a[uni[u].cnt];
    }
}
int query() { return ans; }
void rollback() {
  ans -= delta, delta = 0;
  for (int u, v, t, i = (int)history.size() - 1; i >= 0; i--) {
    tie(u, v, t) = history[i];
    if (u) {
      anc[u] = t;
      merge(uni[u], uni[v]);
      uni[v].vec.concat_reverse(uni[u].vec);
    } else {
      mrk[v] = 0, uni[v].pushup(c[v]), uni[v].clear();
    }
  }
  history.clear();
}
} // namespace tree
struct block {
  int id, l, r;
  vector<query> q;
  void init() {
    for (int i = l; i <= r; i++) bln[i] = id;
  }
  void solve() {
    sort(q.begin(), q.end(), [](const query &a, const query &b) { return a.l > b.l; });
    tree::reset();
    int cur = l, i;
    for (const auto &it : q) {
      while (it.l <= cur) tree::add(cur--);
      for (tree::roll ^= 1, i = l + 1; i <= it.r; i++) tree::add(i);
      ans[it.id] = tree::query();
      tree::roll ^= 1, tree::rollback();
    }
  }
} block[S1 << 1];
void build() {
  function<int(int)> find = [&](int x) { return anc[x] == x ? x : anc[x] = find(anc[x]); };
  for (int i = 1; i <= n; i++) anc[i] = i, siz[i] = 1;
  for (int u = 1; u <= n; u++) {
    for (int v : G[u])
      if (v < u) {
        int fu = find(u), fv = find(v);
        if (fu == fv) continue;
        if (siz[fu] > siz[fv]) swap(fu, fv);
        pre[u] += siz[fu], anc[fu] = fv, siz[fv] += siz[fu];
      }
    pre[u] += G[u].size();
  }
  int S = accumulate(pre + 1, pre + n + 1, 0) / S1 + 1;
  fprintf(stderr, "> block limit = %d\n", S);
  vector<tuple<int, int, int>> seq = {{n + 1, n, 0}};
  for (int i = n; i >= 1; i--) {
    get<0>(seq.back())--, get<2>(seq.back()) += pre[i];
    if (i == 1 || get<2>(seq.back()) + pre[i] > S) seq.push_back({i, i - 1, 0});
  }
  seq.pop_back(), reverse(seq.begin(), seq.end()), blocks = seq.size();
  for (int _, i = 0; i < blocks; i++) {
    tie(block[i].l, block[i].r, _) = seq[i];
    block[i].id = i, block[i].init();
  }
}
void solve() {
  for (int i = 1; i <= n; i++) tree::uni[i].id = i;
  for (const auto &it : q)
    if (bln[it.l] == bln[it.r]) {
      tree::reset();
      for (int i = it.l; i <= it.r; i++) tree::add(i);
      ans[it.id] = tree::query();
    } else {
      block[bln[it.r]].q.push_back(it);
    }
  for (int i = 0; i < blocks; i++) block[i].solve();
}
int main() {
#ifdef memset0
  freopen("1.in", "r", stdin);
  freopen("1.out", "w", stdout);
#endif
  for (int i = 0; i < N / S2 + 5; i++) tree::rub.push_back(i);
  read(n), read(m), read(x);
  for (int i = 1; i <= n; i++) read(c[i]), --c[i];
  for (int u, v, i = 1; i < n; i++) {
    read(u), read(v);
    G[u].push_back(v), G[v].push_back(u);
  }
  for (int i = 0; i <= x; i++) read(a[i]);
  build();
  for (int l, r, i = 1; i <= m; i++) {
    read(l), read(r);
    q.push_back({l, r, i});
  }
  solve();
  for (int i = 1; i <= m; i++) print(ans[i], '\n');
}
```
