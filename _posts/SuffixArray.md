---
title: SuffixArray
date: 2021-02-08 10:06:43
tags: ACM
---
```
#include<iostream>
#include<stack>
#include<iomanip>
#include<algorithm>
#include<string>
#include<string.h>
#include<cmath>
#include<set>
#include<vector>
#include<queue>
#include<map>
using namespace std;
const int N = 1e6 + 10;
using ll = long long;
const int INF = 0x3f3f3f3f;
using pii = pair<int, int>;
using vi = vector<int>;
#define rep(i, a, b) for (int i = (a); i < (b); i++)
#define mp make_pair 
#define pb push_back
#define clr(x, v) memset(x, v, sizeof (x))
#define all(x) x.begin(), x.end()
#define fi first
#define se second
const int MOD = 1e9 + 7;

struct SuffixArray {
    static const int MAXN = 3e5 + 10;
    string s;
    int n; 
    int rank[MAXN]; // position -> rank
    int sa[MAXN]; // rank -> position
    int lcp[MAXN]; // longest common prefix for [i]th rank and [i-1]th rank
    int tmp[MAXN], buc[MAXN];
    
    SuffixArray(string s) : s(s){
        n = s.size();
        int m = 256;
        for (int i = 0; i < m; i++) buc[i] = 0;
        for (int i = 0; i < n; i++) buc[rank[i] = s[i]]++;
        for (int i = 1; i < m; i++) buc[i] += buc[i - 1];
        for (int i = n - 1; i >= 0; i--) sa[--buc[rank[i]]] = i;
        for (int k = 1; k <= n; k <<= 1) {
            int p = 0;
            for (int i = n - k; i < n; i++) tmp[p++] = i;
            for (int i = 0; i < n; i++) if (sa[i] >= k) tmp[p++] = sa[i] - k;
            for (int i = 0; i < m; i++) buc[i] = 0;
            for (int i = 0; i < n; i++) buc[rank[tmp[i]]]++;
            for (int i = 1; i < m; i++) buc[i] += buc[i - 1];
            for (int i = n - 1; i >= 0; i--) sa[--buc[rank[tmp[i]]]] = tmp[i];
            swap(rank, tmp);
            p = 1;
            rank[sa[0]] = 0;
            for (int i = 1; i < n; i++) {
                rank[sa[i]] = (tmp[sa[i - 1]] == tmp[sa[i]] && sa[i - 1] + k < n && sa[i] + k < n && tmp[sa[i - 1] + k] == tmp[sa[i] + k]) ? p - 1 : p++; 
            }
            m = p;
            if (p >= n) break;
        }

        int pre = 0;
        for (int i = 0; i < n; i++) {
            if (pre) pre--;
            if (rank[i] == 0) continue;
            int j = sa[rank[i] - 1];
            while (s[i + pre] == s[j + pre]) pre++;
            lcp[rank[i]] = pre;
        }
    }
};
ll getDistinctSubstring(string s) {
    SuffixArray sa(s);
    int n = s.size();
    ll res = 0;
    for (int i = 0; i < n; i++) {
        res += n - sa.sa[i] - sa.lcp[i];
    }
    return res;
}
int main() {
    // string s = "aabaaaab";
    // SuffixArray sa(s);
    // for (int i = 0; i < (int)s.size(); i++) {
    //     cout << s.substr(sa.sa[i]) << " " << sa.lcp[i] << endl;
    // }
    return 0;
}


```
