---
title: acm 模板
date: 2020-11-29 20:15:42
tags: ACM
---

```
#include<bits/stdc++.h>
using namespace std;
#define _debug(x) { cerr << "\033[94m" << __func__ << " " << #x << ": " << (x) << "\n\033[0m"; }
#define rep(i, a, b) for (int i = (a); i <= (b); i++)
#define pre(i, b, a) for (int i = (b); i >= (a); i--)
#define clr(x, v) memset(x, v, sizeof (x))
#define all(x) x.begin(), x.end()
#define sz(x) (int) x.size()
#define pb push_back
#define mkp make_pair
#define fi first
#define se second
template<class T> inline void amin(T &x, const T &y) { if (y<x) x=y; }
template<class T> inline void amax(T &x, const T &y) { if (x<y) x=y; }
using ll = signed long long;
using pii = pair<int, int>;
using vi = vector<int>;
using vvi = vector<vi>;
const int inf = 0x3f3f3f3f;
const int MOD = 1e9 + 7;
const int maxn = 700;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    int _;
    for(scanf("%d",&_);_;_--) {
        solve();
    }
    return 0;
}
```
