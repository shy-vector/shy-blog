---
title: 【LADR】【03】Linear Maps
published: 2025-07-22
updated: 2025-07-22
description: 'Linear Algebra Done Right 第三章'
image: ''
tags: [Math, Linear Algebra, LADR]
category: 'Linear Algebra'
draft: false
---

# 线性映射

## 线性映射也构成向量空间

### 习题 3A

#### 第 11 题

设 $V$ 是有限维的，$T \in \mathcal L(V)$。证明
$$
\exists\, \lambda \in \mathbf F, T = \lambda I
$$
当且仅当
$$
\forall\, S \in \mathcal L(V), \: S\,T = TS
$$

> 必要性 ($\Rightarrow$) 易证，充分性 ($\Leftarrow$) 如下。
> 
> **思路**：要证 $Tv = \lambda v$，为了能利用上 $ST = TS$，我们取 $S \in \mathcal L(V)$，且满足 $v = Su$，此时
> $$
> Tv = T(Su) = S(Tu) \overset{?}{=} \lambda v
> $$
> 虽然 $v = Su$ 和线性映射这两个条件会让 $S$ 的存在性无法保证，但可以看到，我们还希望让 $S$ 映射到 $v$ 的方向上，于是我们进一步取 $S_v \in \mathcal L(V)$，且满足 $S_v u = \varphi(u)v$，其中 $\varphi : V \rightarrow \mathbf F$。
> 
> 为了保证 $S_v$ 的线性性，此时 $\varphi$ 必须为线性映射，但我们又想让它在某个输入下输出 $1$，以便我们作替换
> $$
> Tv = T(\varphi(?)v) = T(S_v (?)) = S_v(T(?)) = \varphi(T(?)) v
> $$
> *想让线性映射输出特定，输入只能是基向量*。因为 $V$ 是有限维的 ($\dim V = 0$ 的情形显然，不做讨论)，设 $V$ 的一组基为
> $$
> \{v_1, v_2, \cdots, v_m\}
> $$
> 设法让 $\varphi(v_1) = 1$，其他的输出任意。所有输出确定后，根据**线性映射引理**，这个线性映射 $\varphi$ 存在，且被唯一确定，因此将 $?$ 替换成 $v_1$ 即可完成证明。

#### 第 13 题

设 $V$ 是有限维的，$U$ 是 $V$ 的子空间，$S \in \mathcal L(U, W)$，证明：
$$
\exists\, T \in \mathcal L(V, W),\: \forall\, u \in U,\: Tu = Su
$$

> $T$ 在 $U$ 上的映射和 $S$ 保持一致即可，人为构造在 $V - U$ 上的映射，使整体保持线性性较为困难。
> 
> **证明**：不妨从整体出发，直接在 $V$ 上找满足局部条件的映射，这样容易保证线性性。为了满足局部条件，我们选取从 $U$ 基 (线性无关组) 拓展出来的 $V$ 基 ($\dim V = 0$ 的情形显然，不做讨论)
> $$
> \{u_1, \cdots, u_{m}, v_{m+1}, \cdots, v_n\}
> $$
> 我们令
> $$
> \begin{aligned}
> Tu_i &= Su_i, & i &= 1, \cdots, m \\
> Tv_j &\in W, & j &= m + 1, \cdots, n
> \end{aligned}
> $$
> 根据**线性映射引理**，$T \in \mathcal L(V, W)$ 存在且唯一，且 $\forall\, u \in U$，
> $$
> \begin{aligned}
> Tu &= c_1Tu_1 + \cdots + c_mTu_m \\
> &= c_1Su_1 + \cdots + c_mSu_m = Su
> \end{aligned}
> $$
> 原命题得证。

#### 第 14 题

设 $V$ 是有限维的，$W$ 是无限维的，$\dim V > 0$，证明：$\mathcal L(V, W)$ 是无限维的。

> **证明**：只需证能构造出无穷序列 $\{T_i\}$，使得 $\forall\, n \in \mathbf N_+$，
> $$
> T_1, \cdots, T_n
> $$
> 线性无关，即
> $$
> c_1T_1 + \cdots + c_nT_n = 0
> $$
> 只有平凡解，也就是 $\forall\, v \in V$，
> $$
> (c_1T_1 + \cdots + c_nT_n)v = 0
> $$
> 只有平凡解。设 $V$ 的一组基为
> $$
> \{v_1, \cdots, v_m\}
> $$
> 由于 $W$ 是无限维的，因此存在无穷序列 $\{w_i\}$，使得 $\forall\, k \in \mathbf N_+$，
> $$
> w_1, \cdots, w_k
> $$
> 线性无关。
> 
> > 为了强迫映射线性无关，我们取 $k = n$，令
> > $$
> > T_iv_j = w_i, \quad i = 1, \cdots, n,\quad j = 1, \cdots, m
> > $$
> > 根据**线性映射引理**，这些 $T_i$ 均存在且唯一。对于 $\forall\, v \in V$，
> > $$
> > v = d_1v_1 + \cdots + d_mv_m
> > $$
> > 此时
> > $$
> > \begin{aligned}
> > & (c_1T_1 + \cdots + c_nT_n)v \\
> > ={}& c_1T_1(d_1v_1 + \cdots + d_mv_m) + {}\\
> > &\cdots + c_nT_n(d_1v_1 + \cdots + d_mv_m) \\
> > ={}& (d_1 + \cdots + d_m)(c_1w_1 + \cdots + c_nw_n)
> > \end{aligned}
> > $$
> > 我们发现 $d_1 + \cdots + d_m$ 可能为 $0$。
> 
> 该怎么办？这个问题其实可以规避掉。因为基的任意性，我们在验证 $\forall\, v \in V$，
> $$
> c_1T_1v + \cdots + c_nT_nv = 0
> $$
> 平凡解情况的时候，其实可以用 $v_1$ 去检验，从而减少不必要的任意性。
> > 我们改变映射的定义：
> > $$
> > T_iv_1 = w_i, \quad i = 1, \cdots, n
> > $$
> > $T_iv_j$ 任意，$j = 2, \cdots, m$，此时
> > $$
> > (c_1T_1 + \cdots + c_nT_n)v_1 = c_1w_1 + \cdots + c_nw_n = 0
> > $$
> > 显然只有平凡解。
>
> 原命题得证。

## 零空间与值域

### 习题 3B

#### 第 9 题

设 $T \in \mathcal L(V, W)$ 是单射，$v_1, \cdots, v_n$ 在 $V$ 中线性无关，证明：
$$
Tv_1, \cdots, Tv_n
$$
在 $W$ 中线性无关。

> 在不浪费维度的情况下，单射保持线性无关性。
> 
> **证明**：$T \in \mathcal L(V, W)$ 是单射，故 $\text{null}\, T = \{0\}$，考虑若方程
> $$
> c_1Tv_1 + \cdots + c_nTv_n = 0
> $$
> 成立，则 $c_1v_1 + \cdots + c_nv_n \in \text{null}\, T$，又因为 $v_1, \cdots, v_n$ 在 $V$ 中线性无关，因此
> $$
> c_1v_1 + \cdots + c_nv_n = 0
> $$
> 仅有平凡解，原方程也仅有平凡解，原命题得证。

#### 第 11 题

设 $V$ 是有限维的，$T \in \mathcal L(V, W)$，证明：存在 $V$ 的一个子空间 $U$，使得
$$
U \cap \text{null}\, T = \{0\} \quad \text{and} \quad \text{range}\, T = \{Tu : u \in U\}
$$

> 也就是说，除了 $0$，核 (零空间) 对值域没有贡献。
>
> **证明**：$\text{null}\, T$ 是 $V$ 的子空间，故进行直和分解
> $$
> V = U \oplus \text{null}\, T
> $$
> 此时 $U \cap \text{null}\, T = \{0\}$，且 $U$ 基与 $\text{null}\, T$ 基共同组成 $V$ 基
> $$
> \{u_1, \cdots, u_m, w_1, \cdots, w_k\}
> $$
> 我们将证明此 $U$ 即为所求子空间。任取 $Tv \in \text{range}\, T$，
> $$
> \begin{aligned}
> Tv &= T(c_1u_1 + \cdots + c_mu_m + d_1w_1 + \cdots + d_kw_k) \\
> &= T(c_1u_1 + \cdots + c_mu_m) = Tu, \quad u \in U
> \end{aligned}
> $$
> 故 $\text{range}\, T \subset \{Tu : u \in U\}$，易证 $\text{range}\, T \supset \{Tu : u \in U\}$，得证。

#### 第 18 题

设 $V$ 和 $W$ 都是有限维的，$U$ 是 $V$ 的子空间，证明：
$$
\exists\, T \in \mathcal L(V, W), \: \text{null}\, T = U
$$
当且仅当
$$
\dim U \ge \dim V - \dim W
$$

> 必要性 ($\Rightarrow$) 显然，下面证明充分性 ($\Leftarrow$)。
> 
> **证明**：考虑直和分解 $V = U \oplus X$，取 $V$ 基
> $$
> \{u_1, \cdots, u_m, \cdots, x_1, \cdots, x_k\}
> $$
> 取 $W$ 基
> $$
> \{w_1, \cdots, w_n\}
> $$
> 根据题意，可以保证直和补维数不大于陪域维数
> $$
> k = \dim V - \dim U \le \dim W = n
> $$
> 构造线性映射，可以尝试从基入手：
> 
> > 根据**线性映射引理**，取 $T \in \mathcal L(V, W)$，满足
> > $$
> > Tu_i = 0, \quad Tx_j = w_j
> > $$
> > 易证 $\text{null}\, T \supset U$。任取 $Tv = 0$，直和分解 $v  = u + x$，于是
> > $$
> > Tx = T(v - u) = 0
> > $$
> > 得 $x \in \text{null}\, T$，但 $T$ 不是单射，无法证明 $x = 0$。
> 
> 怎么办？引理无法保证线性映射的单射与否，但我们先前已经有了更进一步的结论：**习题 3B.16** 告诉我们，只要定义域的维数不大于陪域的维数，我们还有能力让找到的线性映射成为单射。
>
> > 由于 $\dim X \le \dim W$，取单射 $S: X \rightarrow W$，根据**习题 3A.13**，将 $S$ 进一步拓展成 $T: V \rightarrow W$，使其除了满足
> > $$
> > \forall\, x \in X, \: Tx = Sx
> > $$
> > 还满足
> > $$
> > \forall\, u + x \in V,\:T(u + x) = Sx
> > $$
> > 这通过令 $Tu_i = 0$ 即可实现。
>
> 现在 $T$ 在 $X$ 上的限制 $S$ 是单射，也就是说
> $$
> T(u + x) = 0 \Leftrightarrow Sx = 0 \Leftrightarrow x = 0 \Leftrightarrow u + x \in U
> $$
> 这说明 $\text{null}\, T = U$，原命题得证。