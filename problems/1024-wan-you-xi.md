# 1024-玩游戏

## 坑点

注意本题要求是要**完全覆盖**，且**游戏时间为**$$[Li,Ri)$$**左闭右开**。

## Solution

对于区间$$[l,r]$$按左端点排好序，再依次枚举。

**当前枚举到的区间必须选择的条件下（即确定左边界）**，由于左端点的有序性，此时只需要考虑区间右边界的位置。显然这个位置为**包括当前区间右边界在内的第**$$k$$**小的右边界（满足被至少**$$k$$**个区间完全覆盖的同时区间最长）**。

这里考虑使用**优先队列**去维护最大的$$k$$个右边界值，超过$$k$$个元素则直接 `pop()` 队顶元素。

## Code

```cpp
#include <bits/stdc++.h>
using namespace std;

#ifdef LOCAL
#include "debug.hpp"
#else
#define debug(...) 42
#endif

using ll = long long;
#define all(a) begin(a), end(a)
#define sz(x) (int)((x).size())

void solve() {
  for (int n, k; cin >> n >> k;) {
    vector<pair<int, int>> p(n);
    for (int i = 0; i < n; i++) {
      cin >> p[i].first >> p[i].second;
    }
    sort(all(p));
    priority_queue<int, vector<int>, greater<int>> pq;
    int ans = 0;
    for (int i = 0; i < n; i++) {
      int l, r; tie(l, r) = p[i];
      pq.emplace(r);
      while (sz(pq) > k) pq.pop();
      if (sz(pq) == k) {
        ans = max(ans, min(pq.top(), r) - l);
      }
    }
    cout << ans << '\n';
  }
}

int main() {
  cin.tie(nullptr)->sync_with_stdio(false);
  cout << fixed << setprecision(15);

  int T = 1;
  // cin >> T;
  while (T--) solve();

  return 0;
}
```
