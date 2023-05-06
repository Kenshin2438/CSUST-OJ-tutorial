# 3651-这才是真的冰阔落

## Catalan数（拓展问题）

首先，使用银行卡的人可以任意放置。假设有$$k$$个人使用银行卡，那么问题就变成：

{% hint style="info" %}
$$n−k$$ 个人排队，他们只有面值为 $$50$$ 或 $$100$$ 的钱币，问最终剩余 $$50$$ 的张数在 $$[L,R]$$ 的合理方案的数目。
{% endhint %}

对于这个问题，假设有$$a$$张$$50$$和$$b$$张$$100$$的钱币$$(a \geq b)$$，则最终剩余的$$50$$为$$a-b$$张，则合法方案数可以等价为**不越过对角线，从**$$(0,0)$$**到**$$(a,b)$$**的格路问题**的解。

{% embed url="https://zhuanlan.zhihu.com/p/31050581" %}

具体过程参考上面给出的链接，此处不做赘述。结果为：

$$
{a+b \choose a} - {a+b \choose a+1}
$$

## Solution

又因为$$a+b=n-k,L \leq a-b \leq R$$，则$$a$$的范围是：

$$
\max(\lceil\frac{n-k+L}{2}\rceil, \lceil\frac{n-k}{2}\rceil) \to \min(n-k, \lfloor\frac{n-k+R}{2}\rfloor)
$$

对于固定的$$k$$，该问题的答案为：

$$
\begin{aligned} Ans &= \sum_{a=\lceil\frac{n-k+L}{2}\rceil}^{\min(n-k, \lfloor\frac{n-k+R}{2}\rfloor)}\Bigg[{n-k \choose a} - {n-k \choose a+1}\Bigg]\newline &= {n-k \choose \lceil\frac{n-k+L}{2}\rceil} - {n-k \choose \min(n-k, \lfloor\frac{n-k+R}{2}\rfloor)+1} \end{aligned}
$$

回到原问题，只要枚举$$k$$的取值同时计算就能得到答案。

$$
\begin{aligned} Ans &= \sum_{k=0}^{n-L}{n \choose k}\Bigg[{n-k \choose \lceil\frac{n-k+L}{2}\rceil} - {n-k \choose \min(n-k, \lfloor\frac{n-k+R}{2}\rfloor)+1}\Bigg] \end{aligned}
$$

现在剩下的问题就是取模了，同之前一样这次的模数依旧不是一个的素数。本题需要预处理**组合数**，4.12周练A题给的的方法在这里不太能过。

对于组合数$${n \choose m}=\frac{n!}{m! \times (n-m)!}$$，$$m!$$是否能直接求逆元呢？显然不行的，因为它之中可能包含有与$$p$$不互素的因子。

那么，我们考虑分离其中与$$p$$互素的部分，按照一般的方法直接求解 阶乘，逆元[^1]；不互素的部分（即包含模数$$p$$的素因子）在求组合数的时候再计算。

时间复杂度 $$\mathcal{O}(\sqrt{P} + n \log n \log P)$$

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
  int n, p, l, r; cin >> n >> p >> l >> r;

  vector<int> primeFactors;
  int phi;
  { // 分解p，得到其所有的素因子，并同时求出欧拉函数值（可选）
    int tmp = phi = p;
    for (int x = 2; x * x <= tmp; x++) {
      if (tmp % x == 0) {
        primeFactors.emplace_back(x);
        while (tmp % x == 0) tmp /= x;
        phi = phi / x * (x - 1);
      }
    }
    if (tmp != 1) {
      primeFactors.emplace_back(tmp);
      phi = phi / tmp * (tmp - 1);
    }
  }
  
  const auto qpow = [](ll x, ll e, ll mod) {
    ll res = 1;
    for (x %= mod; e > 0; e >>= 1, x = x * x % mod)
      if (e & 1) res = res * x % mod;
    return res;
  };

  vector<ll> fac(n + 1), inv(n + 1);
  fac[0] = inv[0] = fac[1] = inv[1] = 1LL;
  for (int i = 2; i <= n; i++) {
    int x = i;
    for (const int &pr : primeFactors)
      while (x % pr == 0) x /= pr;

    fac[i] = fac[i - 1] * x % p;
    // 使用费马小定理求逆元（注意使用条件，由于fac[i]与p互素，则逆元存在）
    inv[i] = qpow(fac[i], phi - 1, p);
  }

  const auto pot = [](int p, int n) -> ll {
    ll e = 0;
    while (n > 0) e += (n /= p);
    return e;
  };

  const auto binom = [&](int n, int m) -> ll {
    if (n < 0 || m < 0 || n < m) return 0LL;
    if (n == m) return 1LL;
    ll res = fac[n] * inv[m] % p * inv[n - m] % p;
    for (const int &pr : primeFactors) {
      ll e = pot(pr, n) - pot(pr, m) - pot(pr, n - m);
      res = res * qpow(pr, e, p) % p;
    }
    return res;
  };

  ll ans = 0;
  for (int k = 0; k <= n; k++) {
    int infA = (l + n - k + 1) / 2;
    int supA = min(n - k, (r + n - k) / 2);

    if (infA > supA) break;
    ll nCk = binom(n, k);
    ll Ak = binom(n - k, infA) - binom(n - k, supA + 1);
    ans = (ans + (Ak + p) % p * nCk % p) % p;
  }
  cout << ans << '\n';
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

[^1]: 并非真正意义上的阶乘和逆元，而是去除了与$$p$$的公因子，剩余数（或逆元）的累乘
