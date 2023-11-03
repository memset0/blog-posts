---
title: "「CTT2018」esperar"
date: 2019-07-07 14:48:00
category: OI 题解
tags: [数论]
cover: 3.png
---

> 给定 $n$ 和长度为 $n$ 的数组 $\{a_i \} _{i=1}^n$ ，求满足 $\forall i \in [1, n], c_i | b_i, b_i | a_i$ 并且 $\prod_{i=1}^n c_i^2  \leq \prod_{i=1}^n b_i$ 的 $\{b_i\}_{i=1}^n$ 和 $\{c_i\}_{i=1}^n$ 的方案数。
> 
> $n \leq 100,\ a_i \leq 10^9$。

<!--more-->

假设 $B = \prod_{i=1}^n b_i, C = \prod_{i=1}^n c_i^2$ ，易证 $C < B$ 的方案与 $C > B$ 的方案是相同的，可以算出总共的方案加上等于的方案再除以二。

考虑等于的方案，可把每个质因数分开考虑，相当于做背包取第 $0$ 项；

考虑总共的方案，对于在 $a_i$ 中出现了 $k$ 次的质因数 $x$，贡献为 $\frac 1 2 (k + 1) (k + 2)$。

```cpp
// =================================
//   author: memset0
//   date: 2019.07.07 11:14:03
//   website: https://memset0.cn/
// =================================
#include <bits/stdc++.h>
#define ll long long
#define debug(...) ((void)0)
#ifndef debug
#define debug(...) fprintf(stderr,__VA_ARGS__)
#endif
namespace ringo {
template <class T> inline void read(T &x) {
	x = 0; char c = getchar(); bool f = 0;
	while (!isdigit(c)) f ^= c == '-', c = getchar();
	while (isdigit(c)) x = x * 10 + c - '0', c = getchar();
	if (f) x = -x;
}
template <class T> inline void print(T x) {
	if (x < 0) putchar('-'), x = -x;
	if (x > 9) print(x / 10);
	putchar('0' + x % 10);
}
template <class T> inline void print(T x, char c) { print(x), putchar(c); }
template <class T> inline void print(T a, int l, int r, std::string s = "") {
	if (s != "") std::cout << s << ": ";
	for (int i = l; i <= r; i++) print(a[i], " \n"[i == r]);
}

const int N = 110, M = 1e5 + 10, mod = 998244353;
int n, min, max, ans = 1, sum = 1, pcnt;
int a[N], b[N][M], c[M][N], pri[M], vis[M], f[M], g[M];

inline int dec(int a, int b) { a -= b; return a < 0 ? a + mod : a; }
inline int inc(int a, int b) { a += b; return a >= mod ? a - mod : a; }
inline int mul(int a, int b) { return (ll)a * b - (ll)a * b / mod * mod; }

void init_pri(int limit) {
	for (int i = 2; i < limit; i++) {
		if (!vis[i]) pri[++pcnt] = i;
		for (int j = 1; j <= pcnt && i * pri[j] < limit; j++)
			{ vis[i * pri[j]] = 1; if (i % pri[j] == 0) break; }
	}
}

void split(int x, int *cnt) {
	static std::map<int, int> map;
	for (int i = 1; i <= pcnt; i++) while (x % pri[i] == 0) ++cnt[i], x /= pri[i];
	if (x != 1) ++cnt[map.find(x) == map.end() ? (pri[++pcnt] = x, map[x] = pcnt) : map[x]];
}

int solve(int *cnt) {
	for (register int i = min; i <= max; i++) f[i] = 0;
	min = max = 0, f[0] = 1;
	for (int i = 1; i <= n; i++) {
		for (int j = 0; j <= cnt[i]; j++)
			for (int k = 0; k <= j; k++)
				for (int i = min; i <= max; i++)
					g[i + (k << 1) - j] = inc(g[i + (k << 1) - j], f[i]);
		min -= cnt[i], max += cnt[i];
		for (register int i = min; i <= max; i++) f[i] = g[i], g[i] = 0;
	}
	return f[0];
}

void main() {
	read(n), init_pri(1e5);
	for (int i = 1; i <= n; i++) read(a[i]), split(a[i], b[i]);
	for (int i = 1; i <= n; i++) for (int j = 1; j <= pcnt; j++) c[j][i] = b[i][j];
	for (int i = 1; i <= pcnt; i++) ans = mul(ans, solve(c[i]));
	for (int i = 1; i <= pcnt; i++) for (int j = 1; j <= n; j++) sum = mul(sum, mul((mod + 1) >> 1, mul(c[i][j] + 2, c[i][j] + 1)));
	print(mul(inc(ans, sum), (mod + 1) >> 1), '\n');
}

} signed main() {
#ifdef MEMSET0_LOCAL_ENVIRONMENT
	freopen("1.in", "r", stdin);
#endif
	return ringo::main(), 0;
}
```
