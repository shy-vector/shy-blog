---
title: 初等数论
published: 2026-01-29
updated: 2026-01-29
description: '本文是我自学数论的笔记，有些尾巴 (二次互反律、原根、离散对数、抽象代数补充) 还没收好，但暂时不想继续写了．'
image: ''
tags: ['Math', 'Number Theory']
category: 'Number Theory'
draft: false
---

# 初等数论

:::caution[前言]
本文是我自学数论的笔记，有些尾巴 (二次互反律、原根、离散对数、抽象代数补充) 还没收好，但暂时不想继续写了．

之前以为仅靠理解，就能简单运用起数论，来解决算法竞赛涉及的问题，笔记就没做．后来太久没做题，知识基本忘光了，就觉得比较浪费而可惜．加之数论研究的是“**数的结构**”，仅仅出于美的欣赏，我也应当在人生的某个阶段学习初等数论．

于是重新学习初等数论 (约等于从零开始学)，并附上自己的感想和笔记．

我原先想做类似速查的笔记服务于算法竞赛，但后来不知不觉就给自己加了很多戏，所以读者会觉得我的笔记前半段的内容偏工具向，后半段的内容形式偏自由．我没把精力放在编写偏竞赛向的知识、查 OI Wiki、贴代码实现，而是学习并欣赏抽象理论．

本来打算把精力主要放在学习别的东西上，但不知为何数论学上头了，笔记写了很久．本来个人水平就有限，却像是编教材一样要求自己 (也许我更适合当数学老师吧)．这下又耗费我寒假的两个星期，又开始焦虑自己学的东西没啥用了，又责怪自己学习不自律了，哎．
:::

:::warning[这笔记怎么掺了点代数？]
在写这篇笔记前，我只有一点点离散数学课上有关**群论**的知识，学到后面发现**环论**和数论有着紧密的联系．抽象代数理论让数论的相关证明得到简化，且更贴近本质．所以这篇笔记后半部分会掺进一些我临时学习的、比较简陋的代数理论，望读者见谅．
:::

:::warning[为什么这个结论你没给证明？]
可能原因是：
1. 结论过于显然．
2. 证明思路比较直接．
3. 证明过程比较繁琐，缺少美感．
4. 存在运用抽象理论的简洁证明，但个人的水平有限．
5. 我比较懒，看心情．
:::

:::warning[$\mathbb N$ 是什么？]
在数论领域中，我们通常规定**正整数集** $\mathbb N = \{1, 2, \cdots\}$，**非负整数集** $\mathbb N_0 = \{0, 1, 2, \cdots\}$．
:::

## 整除

:::note[带余除法]
设 $a, b \in \mathbb Z$，$b \ne 0$，则存在唯一的 $q, r \in \mathbb Z$，满足 $a = qb + r$，$0 \le r < |b|$．
:::

:::tip[证明]
考虑 $S = \{ a - qb : q \in \mathbb Z \} \cap \mathbb N_0$，显然 $S \neq \empty$ (**Archimedes 公理**)，由于 $(\mathbb N_0, \le)$ 是**良序的**，总可取 $r_0 = \min S$，设 $r_0 = a - q_0 b$．我们将证明 $r_0 < |b|$，且这样的 $r_0$ 是唯一的．(反证法) 假设 $r_0 \ge |b|$，则
$$
0 \le r_0 - |b| = a - q_0 b - |b| \in S
$$
这与 $r_0 = \min S$ 矛盾，故 $r_0 < |b|$，存在性得证．再设
$$
a = q_1b + r_1 = q_2 b + r_2
$$
其中 $q_1, r_1, q_2, r_2 \in \mathbb Z$，且 $r_1, r_2 \in [0, |b|)$，两式相减得
$$
(q_1 - q_2)b = r_1 - r_2 \in (-|b|, |b|)
$$
显然只有 $q_1 = q_2$，唯一性得证．

> 本质上，带余除法就是**从数轴上的点 $a$ 出发，以 $b$ 为步进，移动至某个长度为 $|b|$ 半开半闭的目标区间内**．
> 1. 因为步长和区间长一致，所以可达，区间内存在落点；
> 2. 因为区间半开半闭，区间内必然有且仅有一个落点．
:::

:::important[推论]
1. (**零**) 若 $a = 0$，则 $q = r = 0$．
2. (**符号**) 在数轴上，从点 $a$ 出发，到达 $[0, |b|)$ 内．当 $b > 0$ 时，$q > 0$ 表示左移，$q < 0$ 表示右移．
   | $a$ | $b$ | $q$ |
   | --- | --- | --- |
   | $+$ | $+$ | $\ge 0$ |
   | $+$ | $-$ | $\le 0$ |
   | $-$ | $+$ | $-$ |
   | $-$ | $-$ | $+$ |
3. (**折半**) 若 $a \ge b > 0$，则 $r < a / 2$，即 $r < \min(|b|, a / 2)$．
:::

:::tip[推论 3 的证明]
容易得知 $q > 0$，即 $q \ge 1$．(反证法) 假设 $r \ge a / 2$，则
$$
0 < r < b \le qb = a - r \le a / 2
$$
显然矛盾．

> 直观上讲，剃平后的 $a - r$ 是 $b$ 的倍数，故 $b \le a - r$，但又要 $b > r$，因此临界情况是折半．
:::

:::warning[C++]
在 C/C++ 中，`a / b` 如何取整，取决于编译器的具体实现．\
但从 [C99](https://en.cppreference.com/w/c/language/operator_arithmetic) 和 [C++11](https://en.cppreference.com/w/cpp/language/operator_arithmetic) 起，标准规定：**商向零取整**，即直接舍弃小数部分．\
`a % b` 被定义为 `a - a / b`，因此取模结果的符号取决于 `/` 如何取整．以下断言保证为真：
```c
assert(5 % 3 == 2);
assert(5 % -3 == 2);
assert(-5 % 3 == -2);
assert(-5 % -3 == -2);
```
形式化地，商向零取整的带余除法 $a = qb + r$ 中，$r$ 的目标区间不是 $[0, |b|)$，而是
$$
r \in
\begin{cases}
[0, |b|),   & a \ge 0, \\
(-|b|, 0],  & a < 0.
\end{cases}
$$
:::

:::note[整除]
在带余除法 $a = qb + r$ 中，若 $r = 0$，称 $b$ **整除** $a$，$b$ 是 $a$ 的**因数**，$a$ 是 $b$ 的**倍数**，记作 $b \mid a$．
:::

:::important[推论]
1. (**零**) $n \mid 0$ 对 $n \ne 0$ 恒成立，即 **$0$ 的因数是所有非零整数**．
2. (**幺**) $n \mid 1$ 当且仅当 $n = \pm 1$，即 **$1$ 的因数只有 $\pm 1$**．
3. (**凡因**) $\pm 1 \mid n$ 和 $\pm n \mid n$ 都恒成立，即**平凡因数**．
4. (**正负**) $\pm a \mid \pm b$，$\pm a \mid \mp b$ 互相等价，即**正负不影响因数组成**．
5. (**有界**) 若 $a \mid b$ 且 $b \ne 0$，则 $|a| \le |b|$，即**因数向 $0$ 靠拢**．
6. (**自反**) 若 $a \mid b$ 且 $b \mid a$，则 $|a| = |b|$．
7. (**传递**) 若 $a \mid b$ 且 $b \mid c$，则 $a \mid c$．
8. (**消去**) 若 $k \ne 0$，则 $ka \mid kb$ 等价于 $a \mid b$．
9. (**分解**) 若 $ab \mid m$，则 $a \mid m$ 且 $b \mid m$．
10. (**组合**) 若 $e \mid a$ 且 $e \mid b$，则 $e \mid (ka + lb)$，其中 $k, l \in \mathbb Z$．
:::

:::tip[推论 5 的证明]
设 $b = ad \ne 0$，则 $d \ne 0$，即 $|d| \ge 1$，于是
$$
|b| = |ad| = |a| |d| \ge |a|
$$
:::

## GCD 和 LCM

:::note[gcd 和 lcm]
今有 $a, b \in \mathbb Z$，$a^2 + b^2 > 0$．记 $D$ 为 $a$ 和 $b$ 的正公因数集，$M$ 为 $a$ 和 $b$ 的正公倍数集．\
我们称 $a$ 和 $b$ 的**最大 (正) 公因数**为 $\gcd(a, b) = \max D$，**最小 (正) 公倍数** $\text{lcm}(a, b) = \min M$．
1. 由于 $1 \in D$，故 $D \ne \empty$，又因为 $d \le |a|$ 且 $d \le |b|$，因此 $D$ 是有限集，$\gcd(a, b)$ 必然存在；
2. 由于 $|ab| \in M$，故 $M \ne \empty$，又因为 $M \sub \mathbb N$，且 $(\mathbb N, \le)$ 是良序的，因此 $\text{lcm}(a, b)$ 必然存在．
:::

:::important[gcd 的推论]
1. (**零**) $\gcd(0, n) = n$，其中 $n \ne 0$．
2. (**幺**) $\gcd(1, n) = 1$．
3. (**对称**) $\gcd(a, b) = \gcd(b, a)$．
4. (**正负**) $\gcd(a, b) = \gcd(|a|, |b|)$．
5. (**线性**) 若 $k \in \mathbb Z$，$k \ne 0$，则 $\gcd(ka, kb) = |k|\gcd(a, b)$．
6. (**幂性**) 若 $k \in \mathbb Z$，则 $\gcd(a^k, b^k) = \gcd(a, b)^k$．
7. (**最近公共祖先**) $e \mid \gcd(a, b)$，当且仅当 $e \mid a$ 且 $e \mid b$．
:::

:::tip[推论 5 的证明]
这里会使用到 Bézout 定理及其推论，详见下文．

一种证明方法是观察“张成空间”．
$$
\gcd(ka, kb) \mathbb Z = \text{span}(ka, kb) = k \,\text{span}(a, b) = k \gcd(a, b) \mathbb Z
$$
因此 $|\gcd(ka, kb)| = |k \gcd(a, b)|$，即 $\gcd(ka, kb) = |k| \gcd(a, b)$．

另一种证明方法是相互整除．
1. 先证 $k \gcd(a, b) \mid \gcd(ka, kb)$：由 Bézout 推论，只需证 $k\gcd(a, b) \mid ka$，消去 $k$ 后显然成立．
2. 再证 $\gcd(ka, kb) \mid k \gcd(a, b)$：欲将其转化为 $\gcd(ka, kb) / k \mid \gcd(a, b)$，需 $k \mid \gcd(ka, kb)$，而这是显然的，因为 $k \mid ka$，随后要证 $\gcd(ka, kb) / k \mid a$，两边同乘以 $k$ 显然成立．
因此 $|\gcd(ka, kb)| = |k \gcd(a, b)|$，即 $\gcd(ka, kb) = |k| \gcd(a, b)$．
:::

:::tip[推论 6 的证明]
这里会使用到互素相关的结论，详见下文．\
记 $d = \gcd(a, b)$，则 $\dfrac{a}{d} \perp \dfrac{b}{d}$，因此 $\Big(\dfrac{a}{d}\Big)^k \perp \Big(\dfrac{b}{d}\Big)^k$，
$$
\gcd(a^k, b^k) = d^k \gcd\left(\Big(\dfrac{a}{d}\Big)^k, \Big(\dfrac{b}{d}\Big)^k\right) = d^k = \gcd(a, b)^k
$$
:::

:::tip[推论 7 的证明]
($\Rightarrow$) $e \mid \gcd(a, b) \mid a$，$e \mid \gcd(a, b) \mid b$．\
($\Leftarrow$) 由 Bézout 定理，设 $\gcd(a, b) = ka + lb$，$k,l \in \mathbb Z$．由 $e \mid a$ 且 $e \mid b$ 得 $e \mid (ka + lb) = \gcd(a, b)$．
:::

:::note[互素]
若 $\gcd(a, b) = 1$，则称 $a$ 与 $b$ **互素**，记作 $a \perp b$．
:::

:::important[推论]
1. (**零**) 与 $0$ 互素的整数只有 $\pm 1$．
2. (**幺**) $\pm 1$ 与所有整数互素．
3. (**互素化**) 记 $d = \gcd(a, b)$，则 $\dfrac{a}{d} \perp \dfrac{b}{d}$．
4. (**排除异己**) 若 $a \perp b$，则 $a \mid bc$ 蕴含 $a \mid c$．
5. (**排除异己**) 若 $a \perp b$，则 $\gcd(a, bc) = \gcd(a, c)$．
6. (**Euclid 互素定理**) $a \perp m$ 且 $b \perp m$，当且仅当 $ab \perp m$．
7. (**互素组对**) 若 $\forall\, i, j$ 有 $a_i \perp b_j$，则 $\prod_i a_i \perp \prod_j b_j$．
8. (**互素乘积**) 若 $a \perp b$，则 $a \mid m$ 且 $b \mid m$ 蕴含 $ab \mid m$．
9. (**一般乘积**) 记 $d = \gcd(a, b)$，则 $a \mid m$ 且 $b \mid m$ 蕴含 $\left. \dfrac{ab}{d} \,\middle|\, m \right.$，即 $\text{lcm}(a, b) \mid m$．
10. (**互素空间的交**) 若 $a \perp b$，则 $a \mathbb Z \cap b \mathbb Z = ab \mathbb Z$．
11. (**一般空间的交**) 记 $d = \gcd(a, b)$，则 $a \mathbb Z \cap b \mathbb Z = \dfrac{ab}{d} \, \mathbb Z = \text{lcm}(a, b) \,\mathbb Z$．
:::

:::tip[推论 3 的证明]
记 $d = \gcd(a, b)$，则 $d \mid a$ 且 $d \mid b$，于是
$$
d = \gcd(a, b) = d \gcd\Big(\dfrac{a}{d}, \dfrac{b}{d}\Big)
$$
可得 $\gcd\Big(\dfrac{a}{d}, \dfrac{b}{d}\Big) = 1$，因此 $\dfrac{a}{d} \perp \dfrac{b}{d}$．
:::

:::tip[推论 4 的证明]
$\gcd(a, b) = 1$，由 Bézout 定理，设
$$
ka + lb = 1
$$
由 $a \mid bc$，设 $ad = bc$．欲证 $a \mid c$，为消去 $b$，上式两边同乘以 $c$，代入得
$$
kac + lbc = kac + lad = a(kc + ld) = c
$$
因此 $a \mid c$．
:::

:::tip[推论 5 的证明]
设 $d = \gcd(a, bc)$，$e = \gcd(a, c)$．\
由 $a \perp b$ 得 $d \perp b$，又 $d \mid bc$，故 $d \mid c$，结合 $d \mid a$ 可得 $d \mid e$．\
由 $e \mid a$ 且 $e \mid c \mid bc$ 得 $e \mid d$．\
故 $|d| = |e|$，即 $d = e$．
:::

:::tip[推论 6 的证明]
下面使用 Bézout 定理进行证明：

($\Rightarrow$) 存在 $k_1, l_1, k_2, l_2 \in \mathbb Z$，使得
$$
\begin{aligned}
k_1a + l_1m &= 1 \\
k_2b + l_2m &= 1
\end{aligned}
$$
两式相乘，整理得
$$
(k_1k_2)ab + (k_1l_2a + k_2l_1b + l_1l_2m)m = 1
$$
于是 $\gcd(ab, m) = 1$，因此 $ab \perp m$．

($\Leftarrow$) 存在 $k, l \in \mathbb Z$，使得
$$
kab + lm = 1
$$
我们有
$$
\begin{aligned}
(kb)a + lm &= 1 \\
(ka)b + lm &= 1 
\end{aligned}
$$
于是 $\gcd(a, m) = 1$ 且 $\gcd(b, m) = 1$，因此 $a \perp m$ 且 $b \perp m$．
> 从算术基本定理的角度来看，这个定理是显然的．
:::

:::tip[推论 8 的证明]
由于 $a \mid m$，$b \mid m$，不妨设 $m = ak$，则 $b \mid ak$，又因为 $b \perp a$，所以 $b \mid k$，从而 $ab \mid ak = m$．
:::

:::tip[推论 9 的证明]
记 $d = \gcd(a, b)$，由于 $a \mid m$ 且 $b \mid m$，不妨设 $m = ak$．\
则 $b \mid ak$，$\left. \dfrac{b}{d} \,\middle|\, \dfrac{a}{d}k \right.$，而 $\dfrac{b}{d} \perp \dfrac{a}{d}$，故 $\left. \dfrac{b}{d} \,\middle|\, k \right.$，因此 $\left. \dfrac{ab}{d} \,\middle|\, ak \right. = m$．
:::

:::tip[推论 10 的证明]
记 $M_1 = a \mathbb Z \cap b \mathbb Z$，$M_2 = ab \mathbb Z$．\
任取 $m \in M_1$，则 $a \mid m$ 且 $b \mid m$，又因 $a \perp b$，故 $ab \mid m$，即 $m \in M_2$，于是 $M_1 \sub M_2$．\
任取 $m \in M_2$，则 $ab \mid m$，得 $a \mid m$ 且 $b \mid m$，即 $m \in M_1$，于是 $M_2 \sub M_1$．\
故 $M_1 = M_2$．
:::

:::tip[推论 11 的证明]
记 $d = \gcd(a, b)$，$M_1 = a \mathbb Z \cap b \mathbb Z$，$M_2 = \dfrac{ab}{d} \, \mathbb Z$．\
任取 $m \in M_1$，则 $a \mid m$ 且 $b \mid m$，于是 $\left. \dfrac{ab}{d} \,\middle|\, m \in M_2 \right.$，因此 $M_1 \sub M_2$．\
任取 $m \in M_2$，则 $\left. \dfrac{ab}{d} \,\middle|\, m \right.$，得 $a \mid m$ 且 $b \mid m$，于是 $m \in M_1$，因此 $M_2 \sub M_1$．\
综上，$M_1 = M_2$．
:::

:::important[lcm 的推论]
1. 若 $\gcd(a, b) = 1$，则 $\text{lcm}(a, b) = |ab|$．
2. $\gcd(a, b) \,\text{lcm}(a, b) = |ab|$．
:::

:::tip[推论 1 的证明]
任取 $a$ 和 $b$ 的 (正) 公倍数 $m$，则 $a \mid m$ 且 $b \mid m$，又因 $\gcd(a, b) = 1$，于是 $ab \mid m$．得 $|ab| \le m$，而 $|ab|$ 也是 $a$ 和 $b$ 的 (正) 公倍数，因此等号可以成立，$\text{lcm}(a, b) = |ab|$．

更直接的证法是：因为 $a \perp b$，所以公倍数集
$$
a \mathbb Z \cap b \mathbb Z = ab \mathbb Z
$$
于是 $\text{lcm}(a, b) = |ab|$．
:::

:::tip[推论 2 的证明]
可以通过相互整除证明，下面使用更直接的证明方法：
$$
a \mathbb Z \cap b \mathbb Z = \dfrac{ab}{\gcd(a, b)} \mathbb Z
$$
于是 $\text{lcm}(a, b) = \left|\dfrac{ab}{\gcd(a, b)}\right|$，即 $\gcd(a, b) \,\text{lcm}(a, b) = |ab|$．
:::

## Bézout 定理

:::note[辗转相除法]
设 $a, b \in \mathbb Z$，且 $a^2 + b^2 > 0$，带余除法 $a = qb + r$，只要 $|a - qb| < |b|$，算法
$$
\gcd(a, b) = \gcd(b, a - qb) = \cdots
$$
将终止于 $\gcd(d, 0)$，此时 $|d|$ 即为 $\gcd(a, b)$．
:::

:::tip[正确性证明]
记 $D_{ab}$ 为 $a$ 和 $b$ 的公因数集，$D_{br}$ 为 $b$ 和 $r$ 的公因数集，我们将证明 $D_{ab} = D_{br}$．\
任取 $d \in D_{ab}$，则 $d \mid (a - qb) = r$，故 $d \in D_{br}$，即 $D_{ab} \sub D_{br}$．\
又任取 $d \in D_{br}$，则 $d \mid (qb + r) = a$，故 $d \in D_{ab}$，即 $D_{br} \sub D_{ab}$．\
因此 $D_{ab} = D_{br}$，从而 $\gcd(a, b) = \gcd(b, r)$，即 $\gcd(a, b) = \gcd(b, a - qb)$．

> 容易归纳证明 (固定步数后，寻找最小的一组 $a$ 和 $b$ 作为最差情况)，当 $a$ 和 $b$ 为 Fibonacci 数列邻项时，算法达到最差计算复杂度 $O(\log \min (a, b))$．由于每次带余除法都将余数折半，因此该算法十分高效．
:::

:::warning[向零取整]
我们看到，辗转相除法的正确性仅依赖于 $a$ 和 $r$ 互相被线性组合表示，不过可终止性依赖于 $|a - qb| < |b|$，因此可以让带余除法的商向零取整，即 C/C++ 的整数除法．
:::

```cpp
void gcd(i64 a, i64 b, i64 &d) {
    assert(a != 0 || b != 0);
    i64 *cur, *pre;
    for (cur = &a, pre = &b; *pre; std::swap(cur, pre)) {
        if (std::abs(*cur) >= std::abs(*pre)) {
            i64 q = (*cur) / (*pre);
            (*cur) -= q * (*pre);
        }
    }
    d = *cur;
    if (d < 0) d = -d;
}
```

:::note[Bézout 定理]
总存在 $k,l \in \mathbb Z$，使得 $\gcd(a, b) = ka + lb$，即**最大公因数总能用线性组合表示**．
:::

:::tip[证明]
考虑辗转相除法
$$
\begin{aligned}
a &= q_1 b + r_1 \\
&\hspace{0.525em}\vdots \\
r_{N-2} &= q_N r_{N-1} + r_N \\
r_{N-1} &= q_{N+1} r_{N}
\end{aligned}
$$
$r_1$ 和 $r_2$ 都可以表示成 $a$ 和 $b$ 的线性组合，归纳得知 $r_N$ 也可以，$\gcd(a, b) = |r_N|$ 也可以．
:::

:::important[推论]
记 $\text{span}(a, b) = \{ka + lb : k,l \in \mathbb Z\}$，则 $\text{span}(a, b) = \gcd(a, b) \mathbb Z$．
:::

:::tip[证明]
记 $S_{ab} = \text{span}(a, b)$，$d = \gcd(a, b)$．由 Bézout 定理知 $d \in S_{ab}$，容易得到 $d \mathbb Z \sub S_{ab}$．另一方面，任取 $ka + lb \in S_{ab}$，有 $ka + lb = (ka / d + lb / d) d \in d \mathbb Z$，于是 $S_{ab} \sub d \mathbb Z$．因此 $S_{ab} = d \mathbb Z$．

> 可以说 **$\gcd$ 是“张成空间”的基**，它告诉了我们有关该线性组合的全部信息．只要我们得到了 $\gcd$，就能有序地获取所有线性组合．我们也可以通过观察“张成空间”来反推 $\gcd$ 的一些性质．
:::

:::tip[循环群的子群]
循环群 $G = \langle g \rangle$ 的子群 $H$ 必为循环群．设 $I = \{k \in \mathbb Z : g^k \in H\}$，则 $H = \{g^k \in G : k \in I\}$．
1. 若 $H$ 是平凡群，显然也是循环群；
2. 否则取 $d = \gcd\,I$，由 Bézout 定理可知，$d$ 可被 $I$ 中所有数的线性组合表示．\
   因为 $H$ 的封闭性，所以 $d \in I$．我们将证明 $H = \langle g^d \rangle$：一方面 $\forall\, k \in I$，$d \mid k$，故 $I \sub d \mathbb Z$；\
   另一方面 $H$ 是群，容易得知 $I$ 是 $\mathbb Z$ 的加法子群，又因为 $d \in I$，故 $d \mathbb Z \sub I$．\
   所以 $I = d \mathbb Z$，$H = \{g^{dk} \in G : {dk} \in d \mathbb Z\} = \{(g^d)^k \in G : k \in \mathbb Z\} = \langle g^d \rangle$． 
:::

:::note[扩展 Euclid 算法]
设 $a, b \in \mathbb Z$，且 $a^2 + b^2 > 0$，欲寻找 $\gcd(a, b)$ 其中一种线性组合表示．考虑从
$$
\begin{cases}
1 \cdot a + 0 \cdot b = a \\
0 \cdot a + 1 \cdot b = b
\end{cases}
$$
开始，对其增广矩阵
$$
\begin{pmatrix}
1 & 0 & a \\
0 & 1 & b
\end{pmatrix}
$$
进行有限次初等行变换，最终得到行
$$
\begin{pmatrix}
k & l & \gcd(a, b)
\end{pmatrix}
$$
对应的方程正是
$$
ka + lb = \gcd(a, b)
$$
而这只需让每次初等行变换对应于增广列的辗转相除法的每一步即可：
$$
\cdots
\rightarrow
\begin{pmatrix}
s & t & g \\
m & n & h
\end{pmatrix}
\rightarrow
\begin{pmatrix}
m & n & h \\
s-qm & t-qn & g-qh
\end{pmatrix}
\rightarrow
\cdots
\rightarrow
\begin{pmatrix}
k & l & \gcd(a, b) \\
K & L & 0
\end{pmatrix}
$$
:::

```cpp
void exgcd(i64 a, i64 b, i64 &x, i64 &y, i64 &d) {
    assert(a != 0 || b != 0);
    i64 M[2][3] = {{1, 0, a}, {0, 1, b}};
    i64 *cur, *pre;
    for (cur = M[0], pre = M[1]; pre[2]; std::swap(cur, pre)) {
        if (std::abs(cur[2]) >= std::abs(pre[2])) {
            i64 q = cur[2] / pre[2];
            cur[0] -= q * pre[0];
            cur[1] -= q * pre[1];
            cur[2] -= q * pre[2];
        }
    }
    x = cur[0], y = cur[1], d = cur[2];
    if (d < 0) x = -x, y = -y, d = -d;
}
```

:::note[线性不定方程的特解]
线性不定方程 $ax + by = m$ 有解，当且仅当 $\gcd(a, b) \mid m$．\
此时设 $m = e \gcd(a, b)$，取 $\gcd(a, b) = ka + lb$，则
$$
m = e(ka + lb) = a \cdot ek + b \cdot el
$$
得到特解 $(x_0, y_0) = (ek, el)$．
:::

:::note[线性不定方程的通解]
若线性不定方程 $ax + by = m$ 有特解 $(x_0, y_0)$，则通解 ($k \in \mathbb Z$) 为
$$
\begin{cases}
x = x_0 + k \cdot \dfrac{b}{\gcd(a, b)} \\[1em]
y = y_0 - k \cdot \dfrac{a}{\gcd(a, b)}
\end{cases}
$$
:::

:::tip[通解的证明]
已有 $ax_0 + by_0 = m$，再设通解 $(x, y)$ 满足  ，则
$$
a(x - x_0) = b(y_0 - y)
$$
对 $a$ 和 $b$ 互素化，得
$$
\dfrac{a}{\gcd(a, b)}(x - x_0) = \dfrac{b}{\gcd(a, b)}(y_0 - y)
$$
于是
$$
\left. \dfrac{a}{\gcd(a, b)} \,\middle|\, \dfrac{b}{\gcd(a, b)}(y_0 - y) \right.
$$
又因为 $\dfrac{a}{\gcd(a, b)} \perp \dfrac{b}{\gcd(a, b)}$，故
$$
\left. \dfrac{a}{\gcd(a, b)} \,\middle|\, (y_0 - y) \right.
$$
可设 $\dfrac{a}{\gcd(a, b)} \cdot k = y_0 - y$，$k \in \mathbb Z$，代回 $a(x - x_0) = b(y_0 - y)$ 得证．
:::

## 素数

:::note[素数]
设 $n \in \mathbb N$ 且 $n > 1$，若 $n$ 因数只有 $1$ 和 $n$，则称 $n$ 是**素数**，否则是**合数**．
:::

:::warning[负素数？]
在整环 $(R, +, \cdot)$ 中，若元素 $p \in R$ 满足

1. 非零元：$p \ne 0$；
2. 非单位：$\nexists\, q$，$pq = 1$，即 $p$ 不是可逆元；
3. 素性：$\forall a, b \in R$，若 $p \mid ab$ (即 $\exists\, k \in R$，$ab = pk$)，则 $p \mid a$ 或 $p \mid b$．

则称 $p$ 为**素元**．

由于正负不影响整除关系，我们可以在 $\mathbb Z$ 上定义负素数：**$p$ 和 $-p$ 总是同为素数或同为合数**．但这会让各种描述变得拗口，因此若无特殊说明，以下内容的论域均为 $\mathbb N$：**素数一般指正素数，因数一般指正因数**．
:::

:::important[推论]
1. $\mathbb N$ 被分为三类：$1$，素数，合数．
2. 合数的最小非平凡因数是素数．
3. 素数有无穷多个．
4. (**素性**，**Euclid 整除定理**) 设 $a, b \in \mathbb Z$，$p$ 是素数，若 $p \mid ab$，则 $p \mid a$ 或 $p \mid b$．
:::

:::tip[推论 2 的证明]
设 $N$ 是合数，其最小非平凡因数 $p$ 满足 $1 < p < N$ 且 $p \mid N$．\
假设 $p$ 是合数，任取其非平凡因数 $q$ 满足 $1 < q < p$ 且 $q \mid p$．\
于是 $q \mid N$，这与 $p$ 是 $N$ 的最小非平凡因数矛盾．\
因此 $p$ 是素数．
:::

:::tip[推论 3 的证明]
(反证法) 假设仅有的 $k$ 个素数为 $\{p_k\}_{i=1}^k$．\
令 $N = \prod_{i=1}^k p_i + 1$，显然 $1 < p_i < N$，$i = 1, \cdots, k$，因此 $N$ 是合数．\
取 $N$ 的最小非平凡因数 $p$，则 $p \mid N$ 且 $p$ 是素数．设 $p = p_i$，那么 $p \mid \prod_{i=1}^k p_i$．\
故 $p \mid (N - \prod_{i=1}^k p_i) = 1$，即 $p = 1$，这与 $p$ 是素数矛盾．\
因此素数有无穷多个．
:::

:::tip[推论 4 的证明]
若 $p \mid a$ 则证明完毕，于是设 $p \nmid a$，考虑是否有 $p \perp a$ 从而有 $p \mid b$．\
由于素数 $p$ 只有因数 $1$ 和 $p$，所以 $\gcd(p, a)$ 只可能是 $1$ 或 $p$，而 $p \nmid a$，所以 $\gcd(p, a) = 1$．\
因为 $p \mid ab$，且 $p \perp a$，所以 $p \mid b$．

> **素性意味着不可分解，是素数的本质**，等价于算数基本定理中分解的唯一性．
:::

:::note[算术基本定理]
对于所有的 $n \in \mathbb N$ 且 $n > 1$，都存在唯一的素因数分解
$$
n = \prod_{i=1}^k p_i^{\alpha_i}
$$
其中 $p_i$ 是素数且互异，$\alpha_i \in \mathbb N$，$i = 1, \cdots, k$．
:::

:::tip[证明]
若 $n$ 是素数，则分解完成．若 $n$ 是合数，取其最小非平凡因数 $1 < p < n$，则 $p$ 是素数．\
$p$ 的唯一性是显然的，因此下面对 $n$ 的分解也是唯一的．
$$
n = p \cdot \dfrac{n}{p}
$$
对 $\dfrac{n}{p}$ 重复上述分解操作．由于 $\dfrac{n}{p} < n$，分解将终止，便得到 $n$ 唯一的素因数分解．

> 虽然有更严谨的归纳证明，但比较麻烦，个人更喜欢简洁的程序证明．
:::

## 同余

:::note[同余]
若 $m \mid (a - b)$，则称 $a$ 与 $b$ 模 $m$ **同余**，记作
$$
a \equiv b \pmod m
$$
容易证明，这等价于在带余除法 $a = q_1m + r_1$ 和 $b = q_2m + r_2$ 中，$r_1 = r_2$．
:::

:::warning[负模数？]
由于正负不影响整除关系，$a \equiv b \pmod m$ 等价于 $a \equiv b \pmod{(-m)}$．

因此若无特殊说明，**模数 $m$ 总是正整数**．
:::

:::important[推论]
1. 同余是**等价关系**：
   - 自反性：$a \equiv a \pmod m$；
   - 对称性：若 $a \equiv b \pmod m$，则 $b \equiv a \pmod m$；
   - 传递性：若 $a \equiv b \pmod m$，$b \equiv c \pmod m$，则 $a \equiv c \pmod m$．
2. **线性运算**：若 $a \equiv b \pmod m$，$c \equiv d \pmod m$，则
   - 同余式的和：$a + c \equiv b + d \pmod m$；
   - 同余式的积：$ac \equiv bd \pmod m$．
3. **一般缩放**：若 $k \in \mathbb Z$，且 $k \ne 0$，则 $ka \equiv kb \pmod{km}$ 等价于 $a \equiv b \pmod m$．
4. **模数因数**：若 $a \equiv b \pmod {km}$，则 $a \equiv b \pmod m$．
5. **互素消去**：若 $ka \equiv kb \pmod m$，且 $k \perp m$，则 $a \equiv b \pmod m$．
6. **一般消去**：若 $ka \equiv kb \pmod m$，则 $a \equiv b \pmod{\dfrac{m}{\gcd(k, m)}}$．
:::

:::note[乘法逆元]
设 $a, x, m \in \mathbb Z$，$m \ne 0$，考查同余方程
$$
ax \equiv 1 \pmod m
$$
若 $a \perp m$，则方程在模 $m$ 意义下有唯一解，称该解为 $a$ 的**乘法逆元**，记作 $a^{-1}$．否则方程无解．
:::

:::note[线性同余方程]
设 $a, b, x, m \in \mathbb Z$，$m \ne 0$，考查同余方程
$$
ax \equiv b \pmod m
$$
若 $\gcd(a, m) \mid b$，则方程在模 $m$ 意义下有 $\gcd(a, m)$ 个不同的解：
$$
x \equiv x_0 + k \cdot \dfrac{m}{\gcd(a, m)} \pmod m
$$
否则方程无解．
:::

:::tip[循环群的生成元]
设 $G$ 是循环群，$|G| = m$，$G$ 的生成元个数
$$
\begin{aligned}
\#\{g^a \in G : \langle g^a \rangle = G\}
&= \#\{g^a \in G : \forall\, 0 \le b < m, \exists\, x \in \mathbb Z, (g^a)^x = g^b\} \\
&= \#\{0 \le a < m : \forall\, 0 \le b < m, \exists\, x \in \mathbb Z, ax \equiv b \pmod m\} \\
&= \#\{0 \le a < m : \forall\, 0 \le b < m, \gcd(a, m) \mid b\} \\
&= \#\{0 \le a < m : \gcd(a, m) \mid \gcd(0, 1, \cdots, m - 1)\} \\
&= \#\{0 \le a < m : \gcd(a, m) \mid 1\} \\
&= \#\{0 \le a < m : \gcd(a, m) = 1\} \\
&= \varphi(m)
\end{aligned}
$$
其中 $\varphi(m)$ 是 Euler 函数，后面我们将正式定义它．\
可见 $g^k$ 是循环群 $G$ 的生成元，当且仅当 $k \perp |G|$．
:::

:::note[同余类]
设 $m \in \mathbb N$，$r \in \mathbb Z$，定义 $r$ 的**模 $m$ 同余类**
$$
[r]_m = \{x : x \equiv r \pmod m\}
$$
:::

:::important[推论]
根据同余的定义，容易得到
$$
[r]_m = r + m \mathbb Z
$$
模 $m$ 同余关系是等价关系，因此模 $m$ 同余类就是模 $m$ 同余关系的**等价类**：
1. $[a]_m = [b]_m$ 当且仅当 $a \equiv b \pmod m$．
2. 要么 $[a]_m = [b]_m$，要么 $[a]_m \cap [b]_m = \empty$．
:::

:::tip[$\mathbb Z / m \mathbb Z$]
容易证明，$\mathbb Z$ 关于模 $m$ 同余关系的**商集**为
$$
\mathbb Z / m \mathbb Z = \{[r]_m : 0 \le r < m\}
$$
我们注意到，任取 $x \in [r]_m$ 和 $y \in [s]_m$，都有
$$
\begin{aligned}
x + y &\in [r + s]_m \\
xy &\in [rs]_m
\end{aligned}
$$
这提示我们可以在 $\mathbb Z / m \mathbb Z$ 上定义
$$
\begin{aligned}
[a]_m + [b]_m &= [a + b]_m \\
[a]_m \cdot [b]_m &= [ab]_m
\end{aligned}
$$
容易证明它们是良定义的 (不依赖于具体的代表元)，此时 $(\mathbb Z / m \mathbb Z, +, \cdot)$ 构成了代数，简记成 $\mathbb Z / m \mathbb Z$．
:::

## 环

:::note[环]
今有

1. $R \ne \empty$，
2. $+: R \times R \rightarrow R$，
3. $\cdot: R \times R \rightarrow R$，

满足

1. $(R, +)$ 构成交换群，其幺元作为环的**零元** $0$，
2. $(R, \cdot)$ 构成半群，
3. 分配律：$\forall\, a, b, c \in R$，$a \cdot (b + c) = a \cdot b + a \cdot c$ 且 $(a + b) \cdot c = a \cdot c + b \cdot c$，

则称 $(R, +, \cdot)$ 是**环**．
:::

:::important[形形色色的环]
1. **幺环**：$(R, \cdot)$ 是幺半群，其幺元作为环的**幺元** $1$．
2. **交换环**：$(R, \cdot)$ 是交换群．
3. **零环**：$(\{0\}, +, \cdot)$，其中 $0$ 是加法幺元．
   - 零环是含幺交换环．
   - $|R| = 1$，当且仅当 $R$ 是零环．
   - $0 = 1$，当且仅当 $R$ 是零环．
   - $0$ 不可逆，除非 $R$ 是零环．
4. **零因子**：在非零环中，对于 $a \in R$ 且 $a \ne 0$，如果 $\exists\, b \in R$ 且 $b \ne 0$，满足
   $$
   a \cdot b = 0 \quad (b \cdot a = 0)
   $$
   则称 $a$ 是左 (右) 零因子，左右零因子统称为零因子．
5. **可逆元/单位**：在幺环中，对于 $a \in R$，若 $\exists\, b \in R$，满足
   $$
   b \cdot a = 1 \quad (a \cdot b = 1)
   $$
   则称 $a$ 是左 (右) 可逆元/单位，并称 $b$ 是 $a$ 的左 (右) 逆．
6. **消去律**：环有左 (右) 消去律，当且仅当环中不存在左 (右) 零因子．
   > 设 $ac = bc$，$c \ne 0$，那么 $(a - b)c = 0$，而且 $c$ 不是右零因子，因此 $a = b$．
7. **一个元素不能同时是左 (右) 零因子和左 (右) 可逆元**．
   > (反证法) 假设 $a \ne 0$ 是左零因子，$a^{-1}$ 是 $a$ 的左逆，则存在 $b \ne 0$，导出矛盾：
   > $$
   > b = a^{-1}ab = a^{-1}0 = 0
   > $$
8. **整环**：非零，含幺，可交换，不存在零因子．
   - 由于元素**不一定可逆**，不能使用除法；
   - 但零因子不存在，可以使用**消去律**．
9. **除环**：非零，含幺，所有非零元素都可逆．
10. **域**：非零，含幺，可交换，所有非零元素都可逆．
11. **乘法群/单位群**：在幺环 $(R, +, \cdot)$ 中，设 $R^{\times}$ 为 $R$ 中全体可逆元的集合，易知
    $$
    (R^{\times}, \cdot)
    $$
    构成群，称作幺环 $(R, +, \cdot)$ 的乘法群/单位群，记作 $(R, +, \cdot)^{\times}$．
:::

## $\mathbb Z / m \mathbb Z$

:::note[$\mathbb Z / m \mathbb Z$ 的结构]
1. $\mathbb Z / m \mathbb Z$ 是零环，当且仅当 $m = 1$．
2. 当 $m > 1$ 时，$\mathbb Z / m \mathbb Z$ 是交换幺环，幺元是 $[1]_m$．
3. $[r]_m$ 在 $\mathbb Z / m \mathbb Z$ 可逆，当且仅当 $r \perp m$，此时 $([r]_m)^{-1} = [r^{-1}]_m$．
4. 若 $m$ 是合数，设其非平凡因数是 $d$，则所有的 $[d]_m$ 都是零因子，其他非零元素 $[r]_m$ 均可逆．
5. 若 $m$ 是素数，则不存在零因子，且非零元素均可逆，此时 $\mathbb Z / m \mathbb Z$ 是域．
:::

:::important[$\mathbb Z / m \mathbb Z$ 的分解]
如同算术基本定理，将任何大于 $1$ 的正整数 $N$ 分解成素数之幂积
$$
N = p_1^{\alpha_1} p_2^{\alpha_2} \cdots p_k^{\alpha_k}
$$
形成
$$
N \mapsto (\alpha_1, \alpha_2, \cdots, \alpha_k)
$$
类似坐标分解的双射．我们也可以将大模数环分解成模数两两互素的小模数环之直积
$$
\mathbb Z / (m_1 m_2 \cdots m_k) \mathbb Z \cong \mathbb Z / m_1 \mathbb Z \times \mathbb Z / m_2 \mathbb Z \times \cdots \times \mathbb Z / m_k \mathbb Z
$$
形成
$$
[r]_{m_1 m_2 \cdots m_k} \mapsto ([r]_{m_1}, [r]_{m_2}, \cdots, [r]_{m_k})
$$
类似坐标分解的双射以及运算的保持．这意味着**一个模数系统可被分解成多个独立的素数幂模数系统**．

| 3\4 |  0 |  1 |  2 |  3 |
| --- | -- | -- | -- | -- |
|  0  |  0 |  9 |  6 |  3 |
|  1  |  4 |  1 | 10 |  7 |
|  2  |  8 |  5 |  2 | 11 |

这样构造的映射是自然的，且不难证明是同态映射，但能构成双射吗？因为同余的运算性质，
$$
[r]_{m_1 m_2 \cdots m_k} \sub [r]_{m_1}, [r]_{m_2}, \cdots, [r]_{m_k}
$$
每个小环好似一个坐标分量．对于所有的坐标，它们能否唯一确定一个大环？
:::

:::note[中国剩余定理 (Chinese Remainder Theorem, CRT)]
设 $m_1, m_2, \cdots, m_k \in \mathbb N$ 两两互素，那么对于任意的 $a_1, a_2, \cdots, a_k \in \mathbb Z$，同余方程组
$$
\begin{cases}
x \equiv a_1 \pmod{m_1} \\
x \equiv a_2 \pmod{m_2} \\
\hspace{1.1em} \vdots \\
x \equiv a_k \pmod{m_k}
\end{cases}
$$
在模 $M = m_1 m_2 \cdots m_k$ 意义下有**唯一解 $x_0$**，即该同余方程组完全等价于同余方程
$$
x \equiv x_0 \pmod{M}
$$
> 也就是说，所有坐标 $([a_1]_{m_1}, [a_2]_{m_2}, \cdots, [a_k]_{m_k})$ 都能被还原出唯一的 $[x_0]_M$．
:::

:::tip[证明]
假如我们能找到正交基底 $v_1, v_2, \cdots, v_k$，满足
$$
\begin{cases}
v_1 \equiv 1 \pmod{m_1} \\
v_1 \equiv 0 \pmod{m_2} \\
\hspace{1.4em} \vdots \\
v_1 \equiv 0 \pmod{m_k}
\end{cases}
\:
\begin{cases}
v_2 \equiv 0 \pmod{m_1} \\
v_2 \equiv 1 \pmod{m_2} \\
\hspace{1.4em} \vdots \\
v_2 \equiv 0 \pmod{m_k}
\end{cases}
\:
\cdots
\:
\begin{cases}
v_k \equiv 0 \pmod{m_1} \\
v_k \equiv 0 \pmod{m_2} \\
\hspace{1.4em} \vdots \\
v_k \equiv 1 \pmod{m_k}
\end{cases}
$$
便有
$$
\begin{cases}
a_1 v_1 \equiv a_1 \pmod{m_1} \\
a_1 v_1 \equiv 0 \pmod{m_2} \\
\hspace{2.35em} \vdots \\
a_1 v_1 \equiv 0 \pmod{m_k}
\end{cases}
\:
\begin{cases}
a_2 v_2 \equiv 0 \pmod{m_1} \\
a_2 v_2 \equiv a_2 \pmod{m_2} \\
\hspace{2.35em} \vdots \\
a_2 v_2 \equiv 0 \pmod{m_k}
\end{cases}
\:
\cdots
\:
\begin{cases}
a_k v_k \equiv 0 \pmod{m_1} \\
a_k v_k \equiv 0 \pmod{m_2} \\
\hspace{2.35em} \vdots \\
a_k v_k \equiv a_k \pmod{m_k}
\end{cases}
$$
此时
$$
\begin{cases}
a_1 v_1 + a_2 v_2 + \cdots + a_k v_k \equiv a_1 \pmod{m_1} \\
a_1 v_1 + a_2 v_2 + \cdots + a_k v_k \equiv a_2 \pmod{m_2} \\
\hspace{10.9em} \vdots \\
a_1 v_1 + a_2 v_2 + \cdots + a_k v_k \equiv a_k \pmod{m_k}
\end{cases}
$$
得方程组的特解
$$
x_0 = a_1 v_1 + a_2 v_2 + \cdots + a_k v_k
$$
因为通解需要同时满足 $k$ 条同余方程，每条方程解的间隔不一，所以通解的间隔应该为 $\text{lcm}(m_1, m_2, \cdots, m_k)$，即 $M = m_1 m_2 \cdots m_k$，故方程组的解 $x_0$ 在模 $M$ 意义下是唯一的．
> 唯一性的证明也可以通过设另一解来证明二者在模 $M$ 意义下相等．

现在的问题是如何找到这样的基底？
$$
\begin{cases}
v_i \equiv 0 \pmod{m_1} \\
\hspace{1.4em} \vdots \\
v_i \equiv 1 \pmod{m_i} \\
\hspace{1.4em} \vdots \\
v_i \equiv 0 \pmod{m_k}
\end{cases}
$$
首先肯定有 $\left.\dfrac{M}{m_i} \,\middle|\, v_i\right.$，然后我们发现 $\dfrac{M}{m_i} \perp m_i$，因此可取
$$
v_i = \Big(\dfrac{M}{m_i}\Big)_{m_i}^{-1} \cdot \dfrac{M}{m_i}
$$
如此便得正交基底，此时
$$
x_0 =
a_1 \cdot \Big(\dfrac{M}{m_1}\Big)_{m_1}^{-1} \cdot \dfrac{M}{m_1} +
a_2 \cdot \Big(\dfrac{M}{m_2}\Big)_{m_2}^{-1} \cdot \dfrac{M}{m_2} +
\cdots +
a_k \cdot \Big(\dfrac{M}{m_k}\Big)_{m_k}^{-1} \cdot \dfrac{M}{m_k}
$$
便得到模 $M$ 意义下的唯一解．
:::

## $(\mathbb Z / m \mathbb Z)^\times$

:::note[$(\mathbb Z / m \mathbb Z)^\times$]
1. $(\mathbb Z / m \mathbb Z, \cdot)$ 仅仅是交换幺半群，因为元素不一定都可逆，比如 $[0]_m$，$[a]_m$ ($a$ 与 $m$ 不互素)．
2. **模 $m$ 乘法群** $(\mathbb Z / m \mathbb Z)^\times$ 是交换群，所有元素均可逆．
3. $(\mathbb Z / p \mathbb Z)^\times$ 是循环群，即 $(\mathbb Z / p \mathbb Z)^\times \cong C_{p-1}$，$p$ 是素数．
:::

:::important[$(\mathbb Z / m \mathbb Z)^\times$ 的形式]
根据模 $m$ 意义下可逆的条件，我们有
$$
(\mathbb Z / m \mathbb Z)^\times = (\{[r]_m : r \perp m\}, \cdot)
$$
由于 $(\mathbb Z / m \mathbb Z)^\times$ 是群，运算的封闭性可以导出 Euclid 互素定理：$a \perp m$ 且 $b \perp m$，当且仅当 $ab \perp m$．
:::

:::important[$(\mathbb Z / m \mathbb Z)^\times$ 的分解]
根据 Euclid 互素定理，$r \perp M$ 当且仅当 $r \perp m_i$，$i \in \{1, \cdots, k\}$．\
也就是说，$[r]_M \in (\mathbb Z / M \mathbb Z)^\times$ 当且仅当 $[r]_{m_i} \in (\mathbb Z / m_i \mathbb Z)^\times$，$i \in \{1, \cdots, k\}$．于是
$$
\mathbb Z / M \mathbb Z \cong \mathbb Z / m_1 \mathbb Z \times \mathbb Z / m_2 \mathbb Z \times \cdots \times \mathbb Z / m_k \mathbb Z
$$
将保持至
$$
(\mathbb Z / M \mathbb Z)^{\times} \cong (\mathbb Z / m_1 \mathbb Z)^{\times} \times (\mathbb Z / m_2 \mathbb Z)^{\times} \times \cdots \times (\mathbb Z / m_k \mathbb Z)^{\times}
$$

| 4\9 |  1 |  2 |  4 |  5 |  7 |  8 |
| --- | -- | -- | -- | -- | -- | -- |
|  1  |  1 | 29 | 13 |  5 | 25 | 17 |
|  3  | 19 | 11 | 31 | 23 |  7 | 35 |
:::

:::note[$(\mathbb Z / m \mathbb Z)^\times$ 的阶数]
对于 $n \in \mathbb N$，定义 **Euler 函数**
$$
\begin{aligned}
\varphi(n) &= \left|(\mathbb Z / n \mathbb Z)^\times\right| \\
           &= \#\{0 \le k < n : \gcd(k, n) = 1\}
\end{aligned}
$$
:::

:::important[推论]
1. $\varphi(1) = 1$，此时 $(\mathbb Z / m \mathbb Z)^\times$ 是平凡群，仅含 $[0]_m$．
2. 当 $p$ 是素数时，$\varphi(p) = p - 1$．
3. 当 $p$ 是素数时，$\varphi(p^k) = p^k - p^{k-1} = p^k \Big(1 - \dfrac{1}{p}\Big)$．
4. 当 $m \perp n$ 时，$\varphi(mn) = \varphi(m) \varphi(n)$．
5. $\varphi(n) = n \displaystyle\prod_{p \mid n} \Big(1 - \dfrac{1}{p}\Big)$，$n > 1$，$p$ 是素数．
6. $\displaystyle\sum_{d \mid n} \varphi(d) = n$，$n \in \mathbb N$．
:::

:::tip[推论 2 的证明]
注意到 $\gcd(0, p) = p \ne 1$，$\gcd(k, p) = 1$，$k \in \{1, \cdots, p - 1\}$．
:::

:::tip[推论 3 的证明]
由 Euclid 互素定理，$\gcd(a, p^k) = 1$ 当且仅当 $\gcd(a, p) = 1$，于是
$$
\begin{aligned}
\varphi(p^k) &= \#\{0 \le a < p^k : \gcd(a, p^k) = 1\} \\
             &= \#\{0 \le a < p^k : \gcd(a, p) = 1\} \\
             &= p^k - \#\{0 \le a < p^k : \gcd(a, p) > 1\} \\
             &= p^k - \#\{0 \le a < p^k : \gcd(a, p) = p\} \\
             &= p^k - \#\{0 \le a < p^k : p \mid a\} \\
             &= p^k - \#\{0p, 1p, \cdots, p^{k-2}p\} \\
             &= p^k - p^{k-1}
\end{aligned}
$$
:::

:::tip[推论 4 的证明]
$m \perp n$，因此对 $(\mathbb Z / mn \mathbb Z)^{\times}$ 进行分解
$$
\begin{aligned}
(\mathbb Z / mn \mathbb Z)^{\times} &\cong (\mathbb Z / m \mathbb Z)^{\times} \times (\mathbb Z / n \mathbb Z)^{\times} \\ 
\left|(\mathbb Z / mn \mathbb Z)^{\times}\right| &= \left|(\mathbb Z / m \mathbb Z)^{\times}\right| \cdot \left|(\mathbb Z / n \mathbb Z)^{\times}\right| \\
\varphi(mn) &= \varphi(m) \varphi(n)
\end{aligned}
$$
:::

:::tip[推论 5 的证明]
$n > 1$，由算术基本定理
$$
\varphi(n) = \varphi\Big(\prod_{i=1}^{k} p_i^{\alpha_i}\Big)
           = \prod_{i=1}^k \varphi(p_i^{\alpha_i})
           = \prod_{i=1}^k p_i^{\alpha_i} \Big(1 - \dfrac{1}{p_i}\Big)
           = n \prod_{p \mid n} \Big(1 - \dfrac{1}{p}\Big)
$$
$p$ 是素数．
:::

:::tip[推论 6 的证明]
先回顾循环群 $G$ 的性质：
1. 对于所有的 $d \mid |G|$，总存在唯一的 $d$ 阶子群；
2. 循环群的子群仍是循环群；
3. 恰有 $\phi(|G|)$ 个生成元．

今按阶数给 $\mathbb Z / n \mathbb Z$ 里 $n$ 个数分类：关注阶数为 $d$ 的数，它们都对应了各自阶为 $d$ 的生成子群．\
但 $\mathbb Z / n \mathbb Z$ 加法群是循环群，其阶为 $d$ 的子群 $H$ 有且仅有一个．\
因此所有阶数为 $d$ 的数都成为了循环群 $H$ 的生成元，共有 $\varphi(d)$ 个．\
另外，根据有限群的 Lagrange 定理，$d \mid n$．
$$
\begin{aligned}
n &= |\mathbb Z / n \mathbb Z| = \sum_{d \mid n} \#\{0 \le k < n : \text{ord}_n(k) = d\} \\
  &= \sum_{d \mid n} \#\{ h : \langle h \rangle = H, H \le \mathbb Z / n \mathbb Z, |H| = d \} = \sum_{d \mid n} \varphi(d)
\end{aligned}
$$
:::

:::note[$(\mathbb Z / m \mathbb Z)^\times$ 的幂结构]
任取 $[a]_m \in (\mathbb Z / m \mathbb Z)^{\times}$，其中 $m > 1$，$0 \le a < m$，$\gcd(a, m) = 1$．\
研究 $[a]_m$ 的阶数 $\text{ord}_m(a)$：考虑到非平凡群 $(\mathbb Z / m \mathbb Z)^{\times}$ 的幺元是 $[1]_m$，便有
$$
a^{\text{ord}_m(a)} \equiv 1 \pmod m
$$
于是我们可以对任何幂 $[a]_m^n$ 进行带余除法 ($0 \le r < \text{ord}_m(a)$)
$$
a^n = a^{q \cdot \text{ord}_m(a) + r} = {\big(a^{\text{ord}_m(a)}\big)}^{q} \cdot a^r \equiv a^r \pmod m
$$
:::

:::important[结论]
1. $a^n \equiv 1 \pmod m$，当且仅当 $\text{ord}_m(a) \mid n$，其中 $m > 1$，$0 \le a < m$，$\gcd(a, m) = 1$．
2. (**Euler 定理**) $a^{\varphi(m)} \equiv 1 \pmod m$，其中 $m > 1$，$a \in \mathbb Z$，$\gcd(a, m) = 1$．
3. (**Fermat 小定理**) $a^{p-1} \equiv 1 \pmod p$，其中 $p$ 是素数，$p > 1$，$a \in \mathbb Z$，$p \nmid a$．
> 感受 Euler 定理：在 $(\mathbb Z / 10 \mathbb Z)^{\times} = \{[1]_m, [3]_m, [7]_m, [9]_m\}$ 中，
> $$
> 1^4 \equiv 3^4 \equiv 7^4 \equiv 9^4 \equiv 1 \pmod{10}
> $$
:::

:::tip[Euler 定理的证明]
设 $0 \le a < m$，$(\mathbb Z / m \mathbb Z)^{\times}$ 的阶数是 $\varphi(m)$，生成子群 $\langle [a]_m \rangle$ 阶数等于 $\text{ord}_m(a)$．\
根据有限群的 **Lagrange 定理**，我们有 $\text{ord}_m(a) \mid \varphi(m)$，于是得到
$$
a^{\varphi(m)} \equiv 1 \pmod m
$$
> 可不必 $a \in (\mathbb Z / m \mathbb Z)^{\times}$ ($0 \le a < m$)：不妨设 $b \in \mathbb Z$ 满足 $b \equiv a \pmod m$，那么
> $$
> b^{\varphi(m)} \equiv a^{\varphi(m)} \equiv 1 \pmod m
> $$
:::

:::note[原根]
设 $m > 1$，$0 \le g < m$，$\gcd(g, m) = 1$，若 $\text{ord}_m(g) = \varphi(m)$，则称 $g$ 是模 $m$ 的**原根**．
:::

:::important[性质]
1. 模 $m$ 的原根存在，当且仅当 $(\mathbb Z / m \mathbb Z)^\times$ 是循环群，此时 $[g]_m$ 是循环群 $(\mathbb Z / m \mathbb Z)^\times$ 的生成元．
:::

## $(\mathbb Z / p \mathbb Z)^\times$

:::note[$(\mathbb Z / p \mathbb Z)^\times$ 里的逆元配对]
设 $p$ 是素数．不难证明
$$
\begin{aligned}
f: (\mathbb Z / p \mathbb Z)^\times &\rightarrow (\mathbb Z / p \mathbb Z)^\times \\
                                  a &\mapsto a^{-1}
\end{aligned}
$$
是双射．此时分两种情况：
1. $a^{-1} \not\equiv a \pmod p$；
2. $a^{-1} \equiv a \pmod p$，即 $a^2 \equiv 1 \pmod p$，当且仅当 $a \equiv \pm 1 \pmod p$．

于是我们得知，在 $(\mathbb Z / p \mathbb Z)^\times$ 里，仅有 $[1]_p$ 和 $[p - 1]_p$ 的逆元是其自身，其他数两两配对，互为逆元．
:::

:::important[Wilson 定理]
设素数 $p > 2$，则
$$
(p - 1)! \equiv -1 \pmod p
$$
:::

:::tip[证明]
由于 $(\mathbb Z / p \mathbb Z)^\times$ 里逆元的配对
$$
\begin{aligned}
(p - 1)! &= 1 \cdot \underbrace{2 \cdot \cdots \cdot (p - 2)}_{\text{pairs}} \cdot (p - 1) \\
         &\equiv 1 \cdot (p - 1) \\
         &\equiv -1 \pmod p
\end{aligned}
$$
:::

:::note[二次同余方程]
前面我们已经完全掌握了线性同余方程解的情况．

设素数 $p > 2$，$a, b, c \in \mathbb Z$，$a \perp p$ (即 $p \nmid a$)，欲解二次同余方程
$$
ax^2 + bx + c \equiv 0 \pmod p
$$
尝试配方：为了避免求解 $a$ 的平方根，两边同时乘以 $4a$，前后两式的等价性由 $4a \perp p$ 保证：
$$
(2ax + b)^2 \equiv (b^2 - 4ac) \pmod p
$$
记 $y = 2ax + b$，$\Delta = b^2 - 4ac$，则
$$
y^2 \equiv \Delta \pmod p
$$
这意味着我们必须去研究这个方程解的存在性：怎样的 $\Delta$，才能作为模 $p$ 意义下的平方数？
:::

:::tip[观察 $\mathbf{QR}$]
$\mathbb Z_{13}^*$ 的平方数如下：

|  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 | 11 | 12 |
| -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- | -- |
|  1 |  4 |  9 |  3 | 12 | 10 | 10 | 12 |  3 |  9 |  4 |  1 |

只有 $\{1, 3, 4, 9, 10, 12\}$ 这些数才能作为平方数在模 $13$ 意义下被开根．
:::

:::note[二次剩余]
设素数 $p > 2$，$a \in \mathbb Z$，$p \nmid a$，若
$$
\exists\, x \in \mathbb Z, \quad a \equiv x^2 \pmod p
$$
则称 $a$ 是模 $p$ 的**二次剩余($\mathbf{QR}$)**，否则是**二次非剩余($\mathbf{QNR}$)**．记
$$
\begin{aligned}
\mathbb{QR}_p  &= \{[a]_p : \exists\, x \in \mathbb Z, \: a \equiv x^2 \pmod p\} \\
\mathbb{QNR}_p &= (\mathbb Z / p \mathbb Z)^\times \setminus \mathbb{QR}_p
\end{aligned}
$$
:::

:::tip[初探 $\mathbb{QR}_p$]
1. $1$ 总是 $\mathbf{QR}$．
2. $\mathbb{QR}_p$ 在 $(\mathbb Z / p \mathbb Z)^\times$ 上对称分布．
   > 设素数 $p > 2$，$p \nmid a$，因为 $(p - b)^2 \equiv b^2 \pmod p$ 总成立，所以方程
   > $$
   > x^2 \equiv a \pmod p
   > $$
   > **或无解**，或至少有两个不同余解．下面将说明：**若有解，则仅有两解**．
   > > 任取两解 $x_1^2 \equiv x_2^2 \equiv a \pmod p$ 得
   > > $$
   > > x_1^2 - x_2^2 = (x_1 + x_2)(x_1 - x_2) \equiv 0 \pmod p
   > > $$
   > > 这意味着任取两解，形式都仅有两种：$x_1 \equiv \pm x_2 \pmod p$．\
   > > 因此所有解都仅有该两种形式，方程有且仅有两个不同余解．
3. 设素数 $p > 2$，则 $\#\mathbb{QR}_p = \#\mathbb{QNR}_p = (p - 1) / 2$，其中
   $$
   \mathbb{QR}_p = \{[1^2]_p, [2^2]_p, \cdots, [((p - 1) / 2)^2]_p\}
   $$
:::

:::note[Legendre 符号]
容易验证 $\mathbf{QR}$ 和 $\mathbf{QNR}$ 之间的运算呈现出类似 $\{-1, 1\}$ 模 $p$ 乘法群的形式
$$
\begin{aligned}
\mathbf{QR} \cdot \mathbf{QR}   &= \mathbf{QR} \\
\mathbf{QR} \cdot \mathbf{QNR}  &= \mathbf{QNR} \\
\mathbf{QNR} \cdot \mathbf{QR}  &= \mathbf{QNR} \\
\mathbf{QNR} \cdot \mathbf{QNR} &= \mathbf{QR} \\
\end{aligned}
$$
于是我们定义 **Legendre 符号** (素数 $p > 2$)
$$
\Big(\dfrac{a}{p}\Big) =
\begin{cases}
-1, & a \:\text{是模}\: p \:\text{的}\: \mathbf{QNR} \\
 0, & p \mid a \\
 1, & a \:\text{是模}\: p \:\text{的}\: \mathbf{QR} \\
\end{cases}
$$
容易验证
$$
\begin{aligned}
\psi: (\mathbb Z / p \mathbb Z)^\times &\rightarrow \{-1, 1\} \\
                                 [a]_p &\mapsto \Big(\dfrac{a}{p}\Big)
\end{aligned}
$$
是满同态，于是我们可以用代数的方法研究二次剩余．
:::

:::important[满同态保持的积性]
素数 $p > 2$，$a,b \in \mathbb Z$，$p \nmid a, b$，我们有
$$
\Big(\dfrac{a}{p}\Big)\Big(\dfrac{b}{p}\Big) = \Big(\dfrac{ab}{p}\Big)
$$
特别地，根据定义，$(\frac{a^2}{p}) = 1$．
:::

:::tip[再探 $\mathbb{QR}_p$]
1. $\mathbb{QR}_p$ 在乘法上构成 $(\mathbb Z / p \mathbb Z)^\times$ 的子群．
2. 容易验证
   $$
   \begin{aligned}
   \varphi : (\mathbb Z / p \mathbb Z)^\times &\rightarrow \mathbb{QR}_p \\
                                        [a]_p &\mapsto [a^2]_p
   \end{aligned}
   $$
   是群同态，观察得知 $\ker \varphi = \{1, p - 1\}$，根据群的**同态基本定理 (第一同构定理)**，
   $$
   \begin{aligned}
   (\mathbb Z / p \mathbb Z)^\times \ge \mathbb{QR}_p &\cong (\mathbb Z / p \mathbb Z)^\times / \ker \varphi \\
   |\mathbb{QR}_p| &= \dfrac{|(\mathbb Z / p \mathbb Z)^\times|}{|\ker \varphi|} = \dfrac{p - 1}{2}
   \end{aligned}
   $$
:::

:::note[Euler 准则]
设素数 $p > 2$，$a \in \mathbb Z$，$p \nmid a$．

判断 $a$ 是否为 $\mathbf{QR}$，相当于观察 $(\frac{a}{p})$ 的值，但它无法被直接计算，所以需要我们去构造有关 $a$ 和 $p$ 的式子去得到正确的 $(\frac{a}{p})$．注意到值域是 $\{\pm 1\}$，我们联想到 $1$ 的平方根，所以先去找到模 $p$ 等于 $1$、有关 $a$ 和 $p$ 的式子，这又让我们联想到 Fermat 小定理
$$
a^{p-1} \equiv 1 \pmod p
$$
注意到 $p - 1$ 是偶数，不难得到平方根
$$
a^{(p-1)/2} \equiv \pm 1 \pmod p
$$
可将其作为 $(\frac{a}{p})$ 的计算式
$$
\Big(\frac{a}{p}\Big) \equiv a^{(p-1)/2} \pmod p
$$

> 我们人为构造了
> $$
> \begin{aligned}
> \phi: (\mathbb Z / p \mathbb Z)^\times &\rightarrow (\mathbb Z / p \mathbb Z)^\times \\
>                                  [a]_p &\mapsto [a^{(p-1)/2}]_p
> \end{aligned}
> $$
> Euler 准则就是 $\ker \phi = \ker \psi$．
:::

:::tip[证明]
当 $(\frac{a}{p}) = 1$ 时，即 $\exists\, x \in \mathbb Z, x^2 \equiv a \pmod p$，此时
$$
a^{(p-1)/2} \equiv (x^2)^{(p-1)/2} = x^{p-1} \pmod p
$$
是否有 $p \nmid x$？结论是肯定的：如若 $p \mid x$，由 $x^2 \equiv a \pmod p$ 得到 $p \mid (x + a)(x - a)$，又由 $p$ 的素性，必有 $p \mid (x + a)$ 或 $p \mid (x - a)$，两者皆会导出 $p \mid a$ 的矛盾．因此由 Fermat 小定理可得
$$
a^{(p-1)/2} \equiv 1 \pmod p
$$
当 $(\frac{a}{p}) = -1$ 时，即 $\nexists\, x \in \mathbb Z, x^2 \equiv a \pmod p$，任取 $c \in \mathbb Z_p^*$，考查线性同余方程
$$
cx \equiv a \pmod p
$$
由于 $c \perp p$，方程在模 $p$ 意义下有唯一解，且必有 $x \not\equiv c \pmod p$，否则与 $a$ 是 $\mathbf{QNR}$ 矛盾．因此在 $\mathbb Z_p^*$ 内，乘积与 $a$ 模 $p$ 同余的数两两配对，一共有 $(p - 1) / 2$ 条同余式．因此
$$
(p - 1)! \equiv a^{(p-1)/2} \equiv -1 \pmod p
$$
:::

:::important[小试牛刀]
素数 $p > 2$，探究
$$
x^2 \equiv -1 \pmod p
$$
先看何时有解，我们运用 Euler 准则
$$
\Big(\dfrac{-1}{p}\Big) \equiv (-1)^{(p-1)/2} \pmod p
$$
对 $(p-1)/2 \in \mathbb Z$ 分类讨论：

1. 当 $(p-1)/2 \equiv 0 \pmod 2$，即 $p \equiv 1 \pmod 4$ 时，$(\frac{-1}{p}) = 1$，方程有解．
2. 当 $(p-1)/2 \equiv 1 \pmod 2$，即 $p \equiv 3 \pmod 4$ 时，$(\frac{-1}{p}) = -1$，方程无解．

若方程有解，试求之．容易联想到 Wilson 定理
$$
(p - 1)! \equiv -1 \pmod p
$$
考虑将左式凑成平方的形式，而且我们知道 $\mathbf{QR}$ 对称分布的特性
$$
\begin{aligned}
(p - 1)! &\equiv \Big(\dfrac{p-1}{2}\Big)! (-1)^{(p-1)/2} \Big(\dfrac{p-1}{2}\Big)! \\
         &\equiv \Big(\Big(\dfrac{p-1}{2}\Big)!\Big)^2 \\
         &\equiv -1 \pmod p
\end{aligned}
$$
因此
$$
x \equiv \pm \Big(\dfrac{p-1}{2}\Big)! \pmod p
$$
:::