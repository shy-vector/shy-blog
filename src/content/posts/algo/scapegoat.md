---
title: 替罪羊树 (ScapeGoat-Tree)
published: 2025-07-29
updated: 2025-07-29
description: '相比有旋 Treap 通过旋转维护平衡，替罪羊树通过暴力重构维护平衡，通过平衡因子的大小决定重构时机，常数相比 FHQ-Treap 较小．'
image: ''
tags: [Algorithm, Treap, ScapeGoat]
category: 'Algorithm'
draft: false
---

作为一种平衡树，替罪羊树维护平衡的方式并不是旋转，而是对失衡子树中序遍历，然后直接二分重构，常数相比 FHQ-Treap 较小．

## 平衡因子

由于重构代价较大，我们放宽平衡的标准，以实现 $O(\log n)$ 的均摊复杂度．

考虑以 $p$ 为根节点的子树 $T_p$，节点数为 $|T_p|$，$p$ 的左右儿子分别为 $l(p)$，$r(p)$．我们令
$$
\alpha_p = \frac{1}{|T_p|} \max(|T_{l(p)}|, |T_{r(p)}|)
$$
这个因子越大，说明这棵子树失衡程度越大．我们设定一个失衡标准 $\alpha \in (0.5, 1)$，当 $\alpha_p \ge \alpha$ 时，我们认为这棵子树需要重构．正常情况下，所有子树将满足 $\alpha$-重量平衡
$$
\alpha_p < \alpha
$$
容易得到，若子树满足 $\alpha$-重量平衡，则树高
$$
h(p) \le |\log_{1/\alpha}|T_p||
$$
这也被称 $\alpha$-高度平衡，保证了访问复杂度为 $O(\log n)$．

> 实际上，高度对于访问是最直接的因素，不过如果用 $\alpha$-高度平衡来判断是否失衡，其浮点数的运算会带来较大的常数，所以一般使用 $\alpha$-重量平衡来判断失衡情况．
> 
> 另外，一次插入操作可能会带来自下而上多棵所属子树的失衡，显然取最高的祖先进行子树的重构是最优的．不过其实取路径上任何一个节点作为「替罪羊」重构子树就行，这能让子树高度至少减 $1$．

通常取 $\alpha \in [0.7, 0.8]$．若取 $\alpha = 0.5$，则重构过于频繁；若取 $\alpha = 1$，则完全不重构，退化成普通 BST，树可能退化成链．

## 惰性删除

如果计数仅剩 $1$ 的值即将被删除，我们可以选择不改变树结构，仅让计数减为 $0$ 成为空节点，但过多的空节点会影响访问效率．

我们设定一个容忍度，当非空节点数占比小于这个容忍度时，我们认为这棵树也需要重构，重构时将删除所有空节点．

因为是常数优化，这个容忍度取值要求不高，通常可以直接挪用 $\alpha$ 作为这个容忍度．

> 有趣的是，由于子树因删除操作导致的重构会删除空节点，失衡祖先可能恢复平衡．因此有时也可以选取所有失衡节点作为「替罪羊」进行重构，这样低的「替罪羊」可能会为上面的「替罪羊」替罪，奇妙地减少总重构开销．

## 均摊复杂度的粗略分析

以下为了简化分析，不区分节点数和非空节点数，并且每一次插入操作后，我们选取最高失衡节点作为「替罪羊」，这样不用考虑多次重构的问题．

> 当然任何路径上的节点都可以作为「替罪羊」，这种情况通过选取适当的势能函数 (所有左右子树大小差之和)，通过摊还分析，可以证明与下面相同的均摊复杂度．

考虑某一次插入操作，导致新节点所属最高子树 $T_p$ 失衡，重构此树的代价是 $O(|T_p|)$．但别忘了，我们是怎么让这棵子树失衡的？从上一次重构后到这次重构前，我们往这棵子树添加了多少节点？

在这一次重构前，子树 $T_p$ 不满足 $\alpha$-重量平衡
$$
\max(|T_{l(p)}|, |T_{r(p)}|) > \alpha |T_p|
$$
同时
$$
|T_p| = |T_{l(p)}| + |T_{r(p)}| + 1
$$
得到
$$
\left||T_{l(p)}| - |T_{r(p)}|\right| > (2\alpha - 1)|T_p|
$$
在上一次重构后，如果我们想让子树 $T_p$ 最快失衡，则至少添加 $(2\alpha - 1)|T_p|$ 个节点．也就是总共 $O(|T_p| \log |T_p|)$ 的时间复杂度，均摊到每次插入操作的复杂度为
$$
\frac{O(|T_p| \log |T_p|) + O(|T_p|)}{O(|T_p|)} = O(\log |T_p|)
$$

删除操作同理．

> 真的很不严谨啦...

## 参考实现

```cpp
#include <algorithm>
#include <cstdint>
#include <iostream>

template <class T, int N, int ALPHA>
struct ScapeGoat {
  int tot = 0, rt = 0, l[N + 1], r[N + 1];
  T val[N + 1]; int frq[N + 1];
  int siz[N + 1], dsiz[N + 1], cnt[N + 1];

  struct {
    int siz;
    int arr[N + 1];
    int& operator[](int i) { return arr[i]; }
    void clr()             { siz = 0; }
    void push(int x)       { arr[siz++] = x; }
  } arr;

  int node(T x) {
    tot += 1;
    val[tot] = x, frq[tot] = 1;
    l[tot] = r[tot] = 0;
    siz[tot] = dsiz[tot] = cnt[tot] = 1;
    return tot;
  }

  void pull(int p) {
    siz[p] = siz[l[p]] + siz[r[p]] + 1;
    dsiz[p] = dsiz[l[p]] + dsiz[r[p]] + (frq[p] > 0);
    cnt[p] = cnt[l[p]] + cnt[r[p]] + frq[p];
  }

  bool check(int p) {
    return (
      std::max(siz[l[p]], siz[r[p]]) * 100 >= ALPHA * siz[p] ||
      dsiz[p] * 100 <= ALPHA * siz[p]
    );
  }

  void flatten(int p) {
    if (p == 0) return;
    flatten(l[p]);
    if (frq[p] > 0) arr.push(p);
    flatten(r[p]);
  }

  void build(int &p, int L, int R) {
    if (R < L) return (void)(p = 0);
    int M = (L + R) / 2;
    p = arr[M];
    build(l[p], L, M - 1);
    build(r[p], M + 1, R);
    pull(p);
  }

  void rebuild(int &p) {
    arr.clr();
    flatten(p);
    build(p, 0, arr.siz - 1);
  }

  void insert(int &p, T x, bool enable = true) {
    bool                                balance = !check(p);
    if       (p == 0)                   return (void)(p = node(x));
    if       (x < val[p])               insert(l[p], x, enable && balance);
    else if  (x > val[p])               insert(r[p], x, enable && balance);
    else                                frq[p] += 1;
                                        pull(p);
    if       (enable && !balance)       rebuild(p);
  }

  // lazy remove, x must exists
  void remove(int &p, T x, bool enable = true) {
    bool                                balance = !check(p);
    if       (x < val[p])               remove(l[p], x, enable && balance);
    else if  (x > val[p])               remove(r[p], x, enable && balance);
    else                                frq[p] = std::max(frq[p] - 1, 0);
                                        pull(p);
    if       (enable && !balance)       rebuild(p);
  }

  // return [0, cnt[p]-1], nums of < x
  int rank(int p, T x) {
    if       (p == 0)                   return 0;
    if       (x < val[p])               return rank(l[p], x);
    else if  (x > val[p])               return cnt[l[p]] + frq[p] + rank(r[p], x);
    else                                return cnt[l[p]];
  }

  // k in [1, cnt[p]]
  int kth(int p, int k) {
    if       (p == 0)                   return -1;
    if       (k <= cnt[l[p]])           return kth(l[p], k);
    else if  (k > cnt[l[p]] + frq[p])   return kth(r[p], k - cnt[l[p]] - frq[p]);
    else                                return p;
  }

  int     prev      (int p, T x)      { return kth(p, rank(p, x)); }
  int     next      (int p, T x)      { return kth(p, rank(p, x + 1) + 1); }

  void    clear     ()                { tot = rt = 0; }
  void    insert    (T x)             { insert(rt, x); }
  void    remove    (T x)             { remove(rt, x); }
  int     rank      (T x)             { return rank(rt, x); }
  int     kth       (int k)           { return kth(rt, k); }
  int     prev      (T x)             { return prev(rt, x); }
  int     next      (T x)             { return next(rt, x); }
};

using i64 = int64_t;
ScapeGoat<i64, 100'000, 75> arr{};

int main() {
  std::ios::sync_with_stdio(false);
  std::cin.tie(nullptr);
  int n;
  std::cin >> n;
  while (n--) {
    int op;
    i64 x;
    std::cin >> op >> x;
    if (op == 1) {
      arr.insert(x);
    } else if (op == 2) {
      arr.remove(x);
    } else if (op == 3) {
      std::cout << arr.rank(x) + 1 << '\n';
    } else if (op == 4) {
      std::cout << arr.val[arr.kth(x)] << '\n';
    } else if (op == 5) {
      std::cout << arr.val[arr.prev(x)] << '\n';
    } else if (op == 6) {
      std::cout << arr.val[arr.next(x)] << '\n';
    }
  }
}
```