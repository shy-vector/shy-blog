---
title: 背包问题
published: 2024-11-26
updated: 2025-07-19
description: '本文讲解 0-1 背包、完全背包．'
image: ''
tags: [Algorithm, DP, Knapsack]
category: 'Algorithm'
draft: false
---

## 0-1背包

### 问题

> 有$n$个物品和一个容量为$W$的背包，每个物品的重量为$w_i$，价值为$v_i$．求物品的最大总价值，其中物品总重不超过背包容量．

**0-1背包问题**是所有背包问题的鼻祖．

每个物品只有「在背包里」和「不在背包里」两种状态，对应二进制中的 $1$ 和 $0$，因而得名．

### 思路

背包问题的经典解法是**动态规划**．「执古之道，以御今之有」便是动态规划的核心思想，其基本步骤如下：

1. **明确问题（我在干啥）** 用容量为$W$的背包去装编号为 $1, 2, \cdots, n$ 的物品，求最大总价值．

2. **定义状态（找出问题的参变量）** 上面的问题我们记作 $\text{Sol}(n, W)$．$\text{Sol}$ 意为 $\text{Solution}$，即解决方案；

3. **拆解子问题（当前状态可以从哪些状态得来）** 这里我们先以**全局视角**看待问题．我们关注编号为 $n$ 的物品：在整个问题的最优解里，这个物品在不在背包里？

   - 在背包里．我们把这个物品放进背包里，背包价值已达 $v_n$，那背包还剩多少空间呢？

     还剩 $W-w_n$．这个策略产生了一个**子问题**：

     「用容量为 $W-w_n$ 的背包去装编号为 $1, 2, \cdots, n-1$ 的物品，求最大总价值」，即 $\text{Sol}(n-1, W-w_n)$．

     整个问题的解决方案便产生了：$\text{Sol}(n, W) = v_n + \text{Sol}(n-1, W-w_n)$，$w_n \le W$．

   - 不在背包里．编号为$n$的物品被忽视，这个策略也产生了一个**子问题**：

     「用容量为 $W$ 的背包去装编号为 $1, 2, \cdots, n-1$ 的物品，求最大总价值」，即 $\text{Sol}(n-1, W)$．

     整个问题的解决方案便产生了：$\text{Sol}(n, W) = \text{Sol}(n-1, W)$．

4. **状态转移（参考已解决子问题的答案，提出当前未解决问题的方案）** 两种策略分析完毕后，为了解决整个问题，我们该采取哪种策略呢？由于「在背包里」和「不在背包里」这两种情况是互补的，已经涵盖整个问题，故两者取最优即可 （**最优子结构**），即
   $$
   \text{Sol}(n, W) = \max(v_n + \text{Sol}(n-1, W-w_n),\, \text{Sol}(n-1, W))
   $$
   注意，我们**策略的选择可能并不自由**，因为某些情况下我们没得选：在这里当前物品可能太重装不下，只能采取「不在背包里」这个策略：
   $$
   \text{Sol}(n, W) = \text{Sol}(n-1, W)
   $$
   我们发现接下来的问题和原来的问题**形式上**是一样的，只是问题的**规模**变小了．到这里我们有很自然的实现方式：**递归**．任意小规模问题的方案与上式形式一样，我们考虑小规模问题「用容量为 $c$ 的背包去装编号为 $1, 2, \cdots, i$ 的物品」，则
   $$
   \text{Sol}(i, c) = \max(v_n + \text{Sol}(i-1, c-w_n),\, \text{Sol}(i-1, c))
   $$
   我们发现在递归过程中，问题的规模始终是单调减小的（**无后效性**），因此我们在关注物品 $1, 2, \cdots, i$ 的时候，「对物品 $i$ 是否放进背包中的决策」不会受「背包中物品 $i+1, i+2, \cdots, n$ 的存在与否」的影响．

   在递归过程中，一个问题可能同时是多个大问题的子问题（**子问题重叠**），因此我们需要用数组 $\text{dp}[i][c]$ 存储子问题的答案以供后续大问题的查阅，并干脆直接使用**递推**实现状态的转移：
   $$
   \text{dp}[i][c] = \max(v[i] + \text{dp}[i-1][c-w[i]],\, \text{dp}[i-1][c])
   $$

5. **解决基本小问题** 由于问题规模始终单调减小，递归终将产生基本小问题．在这里，基本小问题应该是 $\text{Sol}(1, 0)$，$\text{Sol}(1, 1)$，$\cdots$，$\text{Sol}(1, W)$ 了吧，不过我们还可以更进一步：$\text{Sol}(0, 0)$，$\text{Sol}(0, 1)$，$\cdots$，$\text{Sol}(0, W)$，这些问题的答案显然都是$0$．

至此，我们只需要从头到尾遍历一遍 $\text{dp}$ 数组便可以得到整个问题的答案：$\text{dp}[n][W]$．

### 代码

```cpp
#include <iostream>
using namespace std;
constexpr int MAXN = 10000;
constexpr int MAXW = 100;

int n, W;
int w[MAXN], v[MAXN];
int dp[MAXN+1][MAXW+1];

int main() {
  cin >> n >> W;
  for (int i = 1; i <= n; i++) {
    cin >> w[i] >> v[i];
  }
  for (int i = 1; i <= n; i++) { // 问题的规模逐渐变大
    for (int c = 0; c <= W; c++) {
      if (w[i] <= c) { // 装得下，我们有两种策略
        dp[i][c] = max(v[i] + dp[i-1][c-w[i]], dp[i-1][c]);
      } else { // 装不进去，我们没得选
        dp[i][c] = dp[i-1][c];
      }
    }
  }
  cout << dp[n][W];
}
```

时间复杂度：$O(nW)$，空间复杂度：$O(nW)$．

### 优化

$O(nW)$ 的空间复杂度并不优秀．我们使用了二维数组 $\text{dp}$ 存储状态，但是我们会发现在状态转移到 $i$ 的过程中，只使用到了 $i-1$ 的状态，$i-2, i-3, \cdots, 2,1$ 的状态对当前及以后的决策毫无帮助，无疑浪费了存储空间．无独有偶，$w[i]$ 和 $v[i]$ 自始至终只被使用过一次用于状态转移．

很自然的想法便是边读 $w, v$ 边转移 $\text{dp}$，并只存储两层的状态 $\text{dp}[2][W]$，每层转移新老交替，循环利用．但我们可以更进一步：只使用一层存储状态 $\text{dp}[W]$，新老状态共用，这便是**滚动数组**：
$$
\text{dp}[c] = \max(v + \text{dp}[c-w],\, \text{dp}[c])
$$
注意，此时遍历范围也可以获得简化：$c \in [w, W]$，因为一旦 $c < w$，即背包容量不足，此时子问题 $\text{dp}[i][c]$ 完全等价于 $\text{dp}[i-1][c]$．而在滚动数组的实现中，经过此次遍历后，$\text{dp}[c]$ 的含义自动从 $\text{dp}[i-1][c]$ 转化成 $\text{dp}[i][c]$，无需任何更新操作，故无需遍历 $c < w$．

如果沿用之前的思路（背包容量 $c$ 从 $w$ 到 $W$ 正向遍历），我们很快会发现问题：$\text{dp}[c-w]$ 已经不代表 $\text{dp}[i-1][c-w[i]]$ 了，而是早已被更新为 $\text{dp}[i][c-w[i]]$，因为状态的转移是按照背包容量从小到大的顺序进行的．**此时物品$i$已经被考虑了**，而不仅仅考虑物品 $1, 2, \cdots, i-1$，这就导致后面的大背包容量问题没法使用小规模问题「仅考虑物品 $1, 2, \cdots, i-1$」的答案 $\text{dp}[i-1][\cdots]$．该怎么办？

很简单，只需**将遍历顺序倒转过来**即可（背包容量 $c$ 从 $W$ 到 $w$ 遍历）：这保证了大背包容量问题能够使用小规模问题「仅考虑物品 $1, 2, \cdots, i-1$」的答案并优先得到解答．
$$
\text{dp}[c] = \max(v + \text{dp}[c-w],\, \text{dp}[c]), \quad c \:\: \text{from}\:W \:\text{to}\:w
$$

### 优化代码

```cpp
#include <iostream>
using namespace std;
constexpr int MAXN = 10000;
constexpr int MAXW = 100;

int n, W, w, v;
int dp[MAXW+1];

int main() {
  cin >> n >> W;
  for (int i = 1; i <= n; i++) {
  cin >> w >> v;
    for (int c = W; c >= w; c--) { // 倒过来遍历，保证子问题答案没有先被更新
      dp[c] = max(v + dp[c-w], dp[c]);
    }
  }
  cout << dp[W];
}
```

时间复杂度：$O(nW)$，空间复杂度：$O(W)$．

## 完全背包

### 问题

> 有$n$**种**物品和一个容量为$W$的背包，每种物品的数量无限，重量为$w_i$，价值为$v_i$．求物品的最大总价值，物品总重不超过背包容量．

与0-1背包问题类似，但每种物品可以取无限次．

### 思路

最朴素的思路就是在0-1背包的基础上，添加待选策略：在背包里第 $i$ 种物品有 $2$ 个、$3$ 个 ······ $k$ 个，则状态转移方程应为
$$
\text{dp}[i][c] = \max\limits_{k=0}^{\lfloor c/w[i] \rfloor}\{k \cdot v[i] + \text{dp}[i-1][c-k \cdot w[i]]\}
$$
滚动数组优化得
$$
\text{dp}[c] = \max\limits_{k=\lfloor c/w \rfloor}^{0}\{k \cdot v + \text{dp}[c-k \cdot w]\}, \quad c \:\: \text{from}\:W \:\text{to}\:0
$$
这样做确实可行，但时间复杂度已经来到 $O(nW^2)$，难登大雅之堂，有什么办法吗？

我们考虑这样一种情形：我们在考虑放多少个第 $i$ 种物品时($\text{dp}[i][c]$)，如果能放，不妨先只放 $1$ 个，其子问题变为：在背包容量减少 $w[i]$ 的情况下，「用容量为 $c-w[i]$ 的背包去装编号为 $1, 2, \cdots, i$ 的物品，求最大总价值」；($\text{dp}[i][c-w[i]]$)

如果不能放，或者不放，其子问题变为：在背包容量不变的情况下，「用容量为$c$的背包去装编号为 $1, 2, \cdots, i - 1$ 的物品，求最大总价值」．($\text{dp}[i-1][c]$)

据此我们写出状态转移方程
$$
\text{dp}[i][c] = \max(v[i] + \text{dp}[i][c-w[i]],\, \text{dp}[i-1][c])
$$
能否使用滚动数组优化？能，但要注意 $\text{dp}[i][c-w[i]]$，其与0-1背包不同：子问题中物品种数的考虑规模不变，但背包容量规模减小了．我们不能反向遍历数组，反而是正向遍历数组：因为反向遍历时，我们只能获得 $\text{dp}[i-1]$ 子问题的答案，不能获得**物品种数的考虑规模相同**的子问题 $\text{dp}[i]$ 的答案．
$$
\text{dp}[c] = \max(v + \text{dp}[c-w],\, \text{dp}[c]), \quad c \:\: \text{from}\:w \:\text{to}\:W
$$

### 代码

```cpp
#include <iostream>
using namespace std;
constexpr int MAXN = 10000;
constexpr int MAXW = 100;

int n, W, w, v;
int dp[MAXW+1];

int main() {
  cin >> n >> W;
  for (int i = 1; i <= n; i++) {
    cin >> w >> v;
    for (int c = w; c <= W; c++) { // 正向遍历
      dp[c] = max(v + dp[c-w], dp[c]);
    }
  }
  cout << dp[W];
}
```

时间复杂度：$O(nW)$，空间复杂度：$O(W)$．
