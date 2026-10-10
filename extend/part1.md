# Part 1

## 凸包

```cpp
// import Geometry.cpp

auto getHull(std::vector<P> p) {
    std::sort(p.begin(), p.end(),
        [&](auto a, auto b) {
            return a.x < b.x || (a.x == b.x && a.y < b.y);
        });
    std::vector<P> hi, lo;
    for (auto p : p) {
        while (hi.size() > 1 && cross(hi.back() - hi[hi.size() - 2], p - hi.back()) >= 0) {
            hi.pop_back();
        }
        while (!hi.empty() && hi.back().x == p.x) {
            hi.pop_back();
        }
        hi.push_back(p);
        while (lo.size() > 1 && cross(lo.back() - lo[lo.size() - 2], p - lo.back()) <= 0) {
            lo.pop_back();
        }
        if (lo.empty() || lo.back().x < p.x) {
            lo.push_back(p);
        }
    }
    return std::make_pair(hi, lo);
}
```

## 旋转卡壳

```cpp
// import Geometry.cpp

int ans = 0;
for (int i = 0, j = 1; i < n; i++) {
    j = std::max(j, i + 1);
    while (j < i + n && cross(p[(i + 1) % n] - p[i], p[(j + 1) % n] - p[j]) > 0) {
        j++;
    }
    ans = std::max({ans, square(p[i] - p[j % n]), square(p[(i + 1) % n] - p[j % n])});
}
```

## 闵可夫斯基和

使用函数 `minkowski` 计算两个凸包的闵可夫斯基和，输入和输出的凸包均满足首尾重合，即 `a.front() == a.back()`。

```cpp
// import Geometry.cpp

std::vector<P> minkowski(const std::vector<P> &a, const std::vector<P> &b) {
    int i = 0, j = 0;
    std::vector<P> c;
    c.reserve(a.size() + b.size() - 1);
    c.push_back(a[0] + b[0]);
    while (i + 1 < a.size() || j + 1 < b.size()) {
        if (i + 1 == a.size()) {
            j++;
        } else if (j + 1 == b.size()) {
            i++;
        } else if (cross(a[i + 1] - a[i], b[j + 1] - b[j]) > 0) {
            i++;
        } else if (cross(a[i + 1] - a[i], b[j + 1] - b[j]) < 0) {
            j++;
        } else {
            i++;
            j++;
        }
        c.push_back(a[i] + b[j]);
    }
    return c;
}
```

## 可撤销并查集

```cpp
struct DSU {
    std::vector<int> f, siz;
    std::vector<std::array<int, 2>> his;

    DSU() {};
    DSU(int n) {
        init(n);
    }
    init(int n) {
        f.resize(n);
        std::iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
        his.clear();
    }
};
```

```cpp
int find(int x) {
    while (f[x] != x) {
        x = f[x];
    }
    return x;
}
```

```cpp
bool merge(int x, int y) {
    x = find(x);
    y = find(y);
    if (x == y) {
        return false;
    }
    if (siz[x] < siz[y]) {
        std::swap(x, y);
    }
    his.push_back({x, y});
    siz[x] += siz[y];
    f[y] = x;
    return true;
}
```

```cpp
int time() {
    return his.size();
}
```

```cpp
void revert(int tm) {
    while (his.size() > tm) {
        auto [x, y] = his.back();
        his.pop_back();
        f[y] = y;
        siz[x] -= siz[y];
    }
}
```

## 可持久化线段树

```cpp
struct Node {
    Node *l = nullptr;
    Node *r = nullptr;
    int cnt = 0;
};
```
```cpp
Node *add(Node *t, int l, int r, int p) {
    Node *x = new Node;
    if (t) {
        *x = *t;
    }
    x->cnt += 1;
    if (r - l == 1) {
        return x;
    }
    int m = (l + r) / 2;
    if (p < m) {
        x->l = add(x->l, l, m, p);
    } else {
        x->r = add(x->r, m, r, p);
    }
    return x;
}
```

```cpp
std::vector<Node *> root(n + 1);
for (int i = 0; i < n; i++) {
    root[i + 1] = add(root[i], 0, m, a[i]);
}
```

## 双指针

```cpp
for (int l = 0, r = 1; r <= n; r++) {
    if (r == n || a[l] != a[r]) {
        l = r;
    }
}
```

## 双模数 NTT

```cpp
// import Polynomial.cpp

Poly<998244353> f(n);
Poly<1004535809> g(n);
auto C1 = CInv<1004535809, 998244353>;
auto C2 = CInv<998244353, 1004535809>;

std::vector<i64> ans(n);
for (int i = 0; i < n; i++) {
    ans[i] = 1LL * int(f[i] * C1) * 1004535809 + 1LL * int(g[i] * C2) * 998244353;
    ans[i] %= 1LL * 998244353 * 1004535809;
}
```

## 扩展欧几里得

```cpp
i64 exgcd(i64 a, i64 b, i64 &x, i64 &y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    }
    i64 g = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return g;
}
```

```cpp
// ax + b = 0 (mod m)
std::pair<i64, i64> sol(i64 a, i64 b, i64 m) {
    assert(m > 0);
    b *= -1;
    i64 x, y;
    i64 g = exgcd(a, m, x, y);
    if (g < 0) {
        g *= -1;
        x *= -1;
        y *= -1;
    }
    if (b % g) {
        return {-1, -1};
    }
    x = x * (b / g) % (m / g);
    if (x < 0) {
        x += m / g;
    }
    return {x, m / g};
}
```

## 多项式快速插值

```cpp
// import Polynomial.cpp

std::vector<Z> x(n), y(n);
for (int i = 0; i < n; i++) {
    std::cin >> x[i] >> y[i];
}

std::vector<Poly<P>> den(4 * n);
[&](this auto &&self, int p, int l, int r) {
    if (r - l == 1) {
        den[p] = {-x[l], 1};
        return;
    }
    int m = (l + r) / 2;
    self(2 * p, l, m);
    self(2 * p + 1, m, r);
    den[p] = den[2 * p] * den[2 * p + 1];
} (1, 0, n);

auto coef = den[1].deriv().eval(x);
for (int i = 0; i < n; i++) {
    coef[i] = 1 / coef[i];
}

auto f = [&](this auto &&self, int p, int l, int r) -> Poly<P> {
    if (r - l == 1) {
        return {coef[l] * y[l]};
    }
    int m = (l + r) / 2;
    auto L = self(2 * p, l, m);
    auto R = self(2 * p + 1, m, r);
    return L * den[2 * p + 1] + R * den[2 * p];
} (1, 0, n);
```

## 珂朵莉树

```cpp
std::map<int, i64> f;
for (int i = 0; i < n; i++) {
    f[i] = a[i];
}
f[n] = -1;
```

```cpp
auto split = [&](int i) {
    auto it = std::prev(f.upper_bound(i));
    if (it->first != i) {
        f[i] = it->second;
    }
};
```

```cpp
for (auto it = f.find(l); it->first != r; it++) {
    it->second += x;
}
```

```cpp
for (auto it = f.find(l); it->first != r; it = f.erase(it))
    ;
f[l] = x;
```

## 取整函数

使用时需保证 $m$ 为正整数。

```cpp
i64 ceilDiv(i64 n, i64 m) {
    if (n >= 0) {
        return (n + m - 1) / m;
    } else {
        return n / m;
    }
}
```

```cpp
i64 floorDiv(i64 n, i64 m) {
    if (n >= 0) {
        return n / m;
    } else {
        return (n - m + 1) / m;
    }
}
```

## Treap

```cpp
std::mt19937_64 rng(std::chrono::steady_clock::now().time_since_epoch().count());

struct Node {
    u64 w = rng();
    int v = 0;
    Node *l = nullptr;
    Node *r = nullptr;
}
```

```cpp
std::pair<Node *, Node *> split(Node *t, int v) {
    if (!t) {
        return {t, t};
    }
    push(t);
    if (t->v < v) {
        auto [l, r] = split(t->l, v);
        t->l = r;
        pull(t);
        return {l, t};
    } else {
        auto [l, r] = split(t->r, v);
        t->r = l;
        pull(t);
        return {t, r};
    }
}
```

```cpp
Node *merge(Node *a, Node *b) {
    if (!a) {
        return b;
    }
    if (!b) {
        return a;
    }
    if (a->w < b->w) {
        push(a);
        a->r = merge(a->r, b);
        pull(a);
        return a;
    } else {
        push(b);
        b->r = merge(a, b->l);
        pull(b);
        return b;
    }
}
```

## 线性基

```cpp
struct Basis {
    std::array<int, K> a {};
    std::array<int, K> t {};
    
    Basis() {
        t.fill(-1);
    }
};
```

```cpp
void add(int x, int y = 1E9) {
    for (int i = K - 1; i >= 0; i--) {
        if (x >> i & 1) {
            if (y > t[i]) {
                std::swap(a[i], x);
                std::swap(t[i], y);
            }
            x ^= a[i];
        }
    }
}
```

```cpp
int queryMax(int y = 0) {
    int x = 0;
    for (int i = K - 1; i >= 0; i--) {
        if ((~x >> i & 1) && t[i] >= y) {
            x ^= a[i];
        }
    }
    return x;
}
```

```cpp
bool contains(int x, int y = 0) {
    for (int i = K - 1; i >= 0; i--) {
        if ((x >> i & 1) && t[i] >= y) {
            x ^= a[i];
        }
    }
    return x == 0;
}
```

```cpp
int size(int y = 0) {
    int cnt = 0;
    for (int i = 0; i < K; i++) {
        cnt += a[i] && t[i] >= y;
    }
    return cnt;
}
```

```cpp
int kth(int k, int y = 0) {
    int cnt = size(y);
    int x = 0;
    for (int i = K - 1; i >= 0; i--) {
        if (a[i] && t[i] >= y) {
            cnt--;
            if ((x >> i & 1) != (k >> cnt & 1)) {
                x ^= a[i];
            }
        }
    }
    return x;
}
```

```cpp
int rank(int x, int y = 0) {
    int k = 0;
    for (int i = K - 1; i >= 0; i--) {
        if (a[i] && t[i] >= y) {
            k = k * 2 + (x >> i & 1);
        }
    }
    return k;
}
```