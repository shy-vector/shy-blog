---
title: 【LADR】【01】Vector Space
published: 2025-02-04
updated: 2025-07-18
description: 'Linear Algebra Done Right 第一章'
image: ''
tags: [Math, Linear Algebra, LADR]
category: 'Linear Algebra'
draft: false
---

观察发现，诸如复数、空间向量、实值函数等数学对象的运算都有很多共同点 (交换群，数乘，分配律)，总结这些共同的特点就是 Linear Algebra Done Right 第一章的主要任务．这一章节主要引入了**向量空间** (线性空间) 这一概念，以统一这类数学对象运算的线性性．同时像集合子集那样，这一章节也引入了向量空间的**子空间**这一概念，子空间内元素的运算性质与所属向量空间的一致，这反映了向量空间整体与局部的相似性．

## $\mathbf R^n$ 和 $\mathbf C^n$

### $\mathbf F$

记号 $\mathbf F$ 一般代表 $\mathbf R$ 或 $\mathbf C$，字母 $\mathbf F$ 源于**域（field）**．称 $\mathbf F$ 中的元素为**标量（scalar）**．

### $\mathbf F^n$

#### 定义

$$
\mathbf F^n = \{(x_1, \cdots, x_n) : x_k \in \mathbf F,\, k = 1, \cdots, n\}
$$

这里 $\mathbf F$ 不一定代表 $\mathbf R$ 或 $\mathbf C$，任何集合均可．记加法恒等元
$$
0 = (0, \cdots, 0)
$$
依照上下文判断是 $\mathbf F$ 中的加法恒等元还是 $\mathbf F^n$ 中的加法恒等元．

#### 加法、加法交换律、加法逆元

$\mathbf F^n$ 中的加法定义为对应坐标相加：
$$
(x_1, \cdots, x_n) + (y_1, \cdots, y_n) = (x_1 + y_1, \cdots, x_n + y_n)
$$
$\mathbf F^n$ 中的加法满足交换律：
$$
\forall\, x, y \in \mathbf F^n,\: x + y = y + x
$$
对于 $\mathbf F^n$ 中的 $x$ 和 $y$，若 $x + y = 0$，则称 $x$，$y$ 互为加法逆元，记 $y = -x$．容易证明对于 $\mathbf F^n$ 中任意的 $x$，其加法逆元存在且唯一．

#### 标量乘法

若 $\lambda \in \mathbf F$，则 $\mathbf F^n$ 中的标量乘法定义为：
$$
\lambda(x_1, \cdots, x_n) = (\lambda x_1, \cdots, \lambda x_n)
$$

## 向量空间

### 集合上的加法和标量乘法

定义集合 $V$ 上的加法是一种映射 $f : (u, v) \rightarrow w$，其中 $u, v, w \in V$，记 $w = u + v$．

定义集合 $V$ 上的标量乘法是一种映射 $f : (\lambda, v) \rightarrow w$，其中 $\lambda \in \mathbf F$，$v, w \in V$，记 $w = \lambda v$．

这里加法和标量乘法都是封闭的（closed）．

### 向量空间

若集合 $V$ 定义了加法和 $\mathbf F$ 上的标量乘法，并且 $V$ 及其所有元素、加法、$\mathbf F$ 上的标量乘法满足：

1. ⭐ 集合中存在**加法恒等元**（identity）$\exists\,0 \in V, \forall\, v \in V, v + 0 = v$；
2. ⭐ 所有元素均存在**加法逆元**（inverse）$\forall\, v \in V, \exist\,w \in V, v + w = 0$；
3. **加法交换律**（commutativity）$\forall\, u, v \in V,\: u + v = v + u$；
4. **加法结合律**（associativity）$\forall\, u, v, w\in V, (u + v) + w = u + (v + w)$；
5. 域中存在**标量乘法恒等元** $\exists\,1\in \mathbf F, \forall\,v \in V, 1v=v$；
6. **标量乘法结合律** $\forall\, a, b \in \mathbf F, \forall\, v \in V, a(bv)=(ab)v$；
7. **标量乘法左分配律**（distributive properties）$\forall\,\lambda\in\mathbf F, \forall\, u, v\in V, \lambda(u + v) = \lambda u + \lambda v$；
8. **标量乘法右分配律** $\forall\,\lambda,\mu\in\mathbf F, \forall\, v\in V, (\lambda + \mu)v = \lambda v + \mu v$，

则称集合 $V$ 是 $\mathbf F$ 上的**向量空间**．向量空间中的元素被称为**向量**，或被称为**点**．

一般研究 $\mathbf R$ 上的向量空间，即**实向量空间**，也就是标量为实数的情况．

最小的向量空间是 $\{0\}$，其中 $0$ 是加法恒等元．**所有的向量空间必含加法恒等元**．

> 注意：空集 $\oslash$ 不是向量空间，因为 $\oslash$ 不满足向量空间定义的第一个条件，即不存在加法恒等元（注意存在量词放在命题句首）．

容易证明，**加法恒等元唯一**．

容易证明，**加法逆元唯一**．因此记号 $-v$ 和 $u - v$ 可以使用．

> 1. 标量 $0$ 与任意向量相乘必得加法恒等元，即 $0v = 0$；
>
> > 证明：（标量乘法的右分配律）
> > $$
> > 0v = (0 + 0)v = 0v + 0v
> > $$
> > 两边加上 $0v$ 的逆元得（加法逆元存在，加法结合律）
> > $$
> > 0v + (-0v) = 0v + (0v + (-0v))
> > $$
> > 即（加法逆元的定义）
> > $$
> > 0 = 0v
> > $$
>
> 2. 标量 $-1$ 与任意向量相乘必得该向量的加法逆元，即 $(-1)v = -v$；
> 3. 任意标量与加法恒等元相乘必得加法恒等元，即 $\lambda 0 = 0$．

### 值函数空间

#### 定义

$$
\mathbf F^S = \{f \mid f:S\rightarrow \mathbf F\}
$$

#### 值函数集上的加法

对于任意的 $f, g \in \mathbf F^S$，若函数 $h \in \mathbf F^S$ 满足
$$
\forall\, x \in S,\: h(x) = f(x) + g(x)
$$
则称 $h$ 是 $f$ 与 $g$ 的**和**，记作 $f + g$．

> 注意：等式左边是 $\mathbf F^S$ 上的加法，等式右边是 $\mathbf F$ 上的加法．

#### 值函数集上的标量乘法

对于任意的 $\lambda \in \mathbf F$ 与 $f \in \mathbf F^S$，若函数 $g \in \mathbf F^S$ 满足
$$
\forall\, x \in S,\: g(x) = \lambda f(x)
$$
则称 $g$ 是 $f$ 的**乘积**，记作 $\lambda f$．

#### 性质

容易证明，值函数集是向量空间．

## 向量空间的子空间

在向量空间中，相比对任意子集，我们对子空间更感兴趣．

### 子空间

#### 定义

设 $U$，$V$ 均为 $\mathbf F$ 上的向量空间，且

1. ⭐ $U \sub V$；
2. ⭐ $U$ 的加法、标量乘法与 $V$ 的加法、标量乘法相同（即向量空间的八个条件相同）；
3. $U$ 的加法恒等元与 $V$ 的加法恒等元相同．

则称 $U$ 是 $V$ 的**子空间**，最简单的子空间是 $\{0\}$．

> 注意：仅条件 2 成立并不能保证加法恒等元相同（只保证各自的加法恒等元存在．两个向量空间分别有各自的加法恒等元，比如向量空间 $U$ 围绕着 $0_u$ 存在（$\exists\, 0_u \in U$），向量空间 $V$ 围绕着 $0_v$ 存在（$\exists\, 0_v \in U$）），不过可以保证各自的乘法恒等元保持一致（因为都是 $\exists\,1 \in \mathbf F$）．
>
> 但事实上，条件 3 可由条件 1 和条件 2 共同导出：
>
> > 在 $U$ 中，
> > $$
> > u + 0_u = u
> > $$
> > 因为 $U \sub V$，所以 $u, 0_u \in V$，故 $0_u = 0_v$．

#### 判定方法

 集合 $U$ 是向量空间 $V$ 的子空间，当且仅当

1. $U \sub V$；
2. ⭐ 父集的加法恒等元在子集中，即 $0_v \in U$；
3. 子集的加法、标量乘法（映射）分别是父集的加法、标量乘法（映射）的子集；
4. ⭐ 在父集的加法下，子集是封闭的，即 $\forall\, u_1, u_2 \in U, u_1 + u_2 \in U$；
5. ⭐ 在父集的标量乘法下，子集也是封闭的，即 $\forall\, \lambda \in F, \forall\, u \in U, \lambda u \in U$；

> 必要性（$\Rightarrow$）是显然的，下面证明**充分性（$\Leftarrow$）**：$U \sub V$ 直接被证明，下面证明八个条件：
>
> 1. $U$ 的加法恒等元存在：对于任意的 $u \in U$，都有 $u \in V$ (条件 1)，因此在向量空间 $V$ 中，
>    $$
>    u + 0_v = u
>    $$
>    而 $0_v \in U$（条件 2），故 $0_u = 0_v$；
>
> 2. $U$ 中所有元素均存在加法逆元：对于任意的 $u \in U$，都有 $(-1)u \in U$（条件 5），即 $-u \in U$；
>
> 3. 剩余的条件均可由条件 3 配合条件 4，5 直接导出．

#### 一些子空间的例子

1. 极限为 0 的数列的线性组合，其极限仍为 0，即 $\{z : \lim z = 0\}$ 是 $\mathbf C^\infty$ 的子空间；
2. 连续函数的线性组合必连续，即 $\{f \in \mathbf R^D : f \text{ is continuous}\}$ 是 $\mathbf R^D$ 的子空间；
3. 可微函数的线性组合必可微，即 $\{f \in \mathbf R^D : f \text{ is differentiable}\}$ 是 $\mathbf R^D$ 的子空间；

### 子空间的和

子空间的**并**往往就不是子空间了，因此我们对子空间的和更感兴趣．

#### 定义

设 $V_1, \cdots, V_m$ 均为 $V$ 的子空间，定义它们的**和（sum）**
$$
V_1 + \cdots + V_m = \{v_1 + \cdots + v_m : v_i \in V_i,\, i = 1, \cdots, m\}
$$

#### 性质

1. ⭐ $V_1 + \cdots + V_m$ 是 $V$ 的子空间；

2. $V_1 + \cdots + V_m$ 是包含 $V_1, \cdots, V_m$ 的 $V$ 的最小子空间．

   也就是说，若子空间 $U$ 包含 $V_1, \cdots, V_m$，则必包含 $V_1 + \cdots + V_m$．

   也就是说，包含了 $V_1, \cdots, V_m$ 的 $U$ 是子空间的必要条件是：$U$ 包含 $V_1 + \cdots + V_m$．

> 性质 1 易证，性质 2 类似于「包含若干子集的最小子集正是这些子集的并」，因为「包含若干子集的子集也都包含这些子集的并」．
>
> 形象的解释就是：挖去 $V_1 + \cdots + V_m$ 中任意子集都会导出不封闭或者不包含 $V_i$（仅能形象解释，无法证明）．正式的**证明**是：对任意的 $i$，取
> $$
> \forall\, v_i =0_1+\cdots+v_i+\cdots+0_m \in V_1 + \cdots + V_m
> $$
> 故 $V_i \sub V_1 + \cdots + V_m$，即 $V_1 + \cdots + V_m$ 是包含 $V_1, \cdots, V_m$ 的 $V$ 的子空间．
>
> 设 $V$ 的任何一个包含 $V_i$ 的子空间 $U$，则 $U$ 必包含 $V_1 + \cdots + V_m$，不凭别的，就凭 $U$ 包含了所有 $V_i$（也就是说，所有 $v_i$ 也都在 $U$ 里），并且 $U$ 是封闭的（结合前面提到的「所有 $v_i$ 也都在 $U$ 里」，有限个 $v_i$ 的和（也就是 $V_1 + \cdots + V_m$ 的元素）封闭在 $U$ 内）．
>
> 形象地说，$U$ 由于自身的封闭性，不得不接受 $V_i$ 共创的所有“产物”，从而包含了 $V_1 + \cdots + V_m$．

3. $U + U = U$；

> 由 $U$ 的封闭性直接导出．
>

4. ⭐ **子空间和的交换律**：$U + W = W + U$；
5. ⭐ **子空间和的结合律**：$(V_1 + V_2) + V_3 = V_1 + (V_2 + V_3)$；

> 由向量空间的加法交换律和加法结合律直接导出．

6. **子空间和的加法恒等元**：零子空间 $\{(0_1, \cdots, 0_m)\} = \{(0, \cdots, 0)\}$；
7. 只有零子空间有加法逆元．

> 由性质 3 及反证法可推出．

#### 例子

| $U$                              | $W$                              | $U+W$                            |
| -------------------------------- | -------------------------------- | -------------------------------- |
| $\{(x, 0, 0)\in\mathbf F^3\}$    | $\{(0, y, 0)\in\mathbf F^3\}$    | $\{(x, y, 0)\in\mathbf F^3\}$    |
| $\{(x, x, y, y)\in\mathbf F^4\}$ | $\{(x, x, x, y)\in\mathbf F^4\}$ | $\{(x, x, y, z)\in\mathbf F^3\}$ |

### 子空间的直和

#### 定义

设 $V_1, \cdots, V_m$ 均为 $V$ 的子空间，如果 $V_1 + \cdots + V_m$ 中的每个元素都能用 $v_1 + \cdots + v_m$ 这个形式唯一地表示出来，则称 $V_1 + \cdots + V_m$ 为**直和（direct sum）**，记作 $V_1 \oplus \cdots \oplus V_m$．

在 3.2.3 中表格的第一行的例子正是直和，第二行的例子不是直和．

#### 性质

1. $V_1 + \cdots + V_m$ 是直和，当且仅当用 $v_1 + \cdots + v_m$ 表示 $0$ 的方式是唯一的；
2. ⭐ $U + W$ 是直和，当且仅当 $U \cap W = \{0\}$．

> 性质 1 不难证明，向量做差结合封闭性即可得证．
>
> 性质 2 类似于「子集的不相交并」，下面证明性质 2：
>
> **必要性（$\Rightarrow$）**：任取 $v \in U \cap W$，则 $v \in U$，于是我们根据 $v$ 找到了 $0$ 的一个表示是 $0 = v + (-v)$；又因为 $v \in W$，也就是说我们又找到了 0 的一个表示是 $0 = (-v) + v$，由于 $0$ 的表示唯一，
> $$
> \begin{cases}
> v = -v \\
> -v = v
> \end{cases}
> $$
> 解得 $v = 0$．也就是说从 $U \cap W$ 取出的元素都是 $0$，于是 $U \cap W = \{0\}$．
>
> **充分性（$\Leftarrow$）**：根据性质 1，只需证明 $0$ 的表示 $0 = u + w$ 是唯一的．
>
> 显然 $w = -u \in U$（$U$ 的封闭性），又因为 $w \in W$，因此 $w \in U \cap W$，故 $w = 0$，$u = -w = 0$，这说明在 $0$ 的表示中，$u$ 和 $w$ 只能为 $0$，是唯一的．
>
> 注意：性质 2 仅使用于两个子空间的情况，原因显然．
