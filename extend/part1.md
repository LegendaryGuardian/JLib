# Part 1

## 凸包

```cpp
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
    int find(int x) {
        while (f[x] != x) {
            x = f[x];
        }
        return x;
    }
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
    int time() {
        return his.size();
    }
    void revert(int tm) {
        while (his.size() > tm) {
            auto [x, y] = his.back();
            his.pop_back();
            f[y] = y;
            siz[x] -= siz[y];
        }
    }
};
```

## 可持久化线段树

```cpp
struct Node {
    Node *l = nullptr;
    Node *r = nullptr;
    int cnt = 0;
};
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
int kth(Node *tl, Node *tr, int l, int r, int k) {
    if (r - l == 1) {
        return l;
    }
    int m = (l + r) / 2;
    int cnt = (tr && tr->l ? tr->l->cnt : 0) - (tl && tl->l ? tl->l->cnt : 0);
    if (k < cnt) {
        return kth(tl ? tl->l : tl, tr ? tr->l : tr, l, m, k);
    } else {
        return kth(tl ? tl->r : tl, tr ? tr->r : tr, m, r, k - cnt);
    }
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