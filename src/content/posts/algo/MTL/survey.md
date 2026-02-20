---
title: 'A Survey on Multi-Task Learning'
published: 2026-02-21
updated: 2026-02-21
description: '一篇多任务学习综述的阅读笔记'
image: ''
tags: ['ML', 'DL', 'MOO', 'MTL']
category: 'Deep Learning'
draft: false
---

# 多任务学习

[A Survey on Multi-Task Learning](https://ieeexplore.ieee.org/document/9392366/)

多任务学习 (MTL)：充分利用多个相关任务的有用信息，增强所有任务的泛化性能．

1. 本文从算法建模、应用、理论分析三个角度分析 MTL；
2. MTL 的定义、五个类别及其特点 (特征学习途径、低秩途径、任务聚类途径、任务关系学习途径、分解途径)；
3. 与其他学习范式的结合 (半监督学习、主动学习、无监督学习、强化学习、多视图学习、图模型)；
4. 任务数、数据维数很高时，在线、并行、分布式的 MTL 模型，以及降维、特征哈希，都有计算和存储的优势；
5. 现实世界的应用；
6. MTL 的理论分析、未来方向．

## Introduction

人类可以同时学习多个任务，并且某个任务的学习会对其他任务的学习有帮助 (学网球和壁球)．MTL 也将从某个任务学习到的知识运用到其他任务的学习，增强所有任务的泛化性能．

起初 MTL 的动机是减轻数据的稀缺问题：多个任务有着各自比较少的标记数据．为了增强数据，最初的 MTL 把所有标记数据聚集起来，都运用到所有任务的学习．大数据时代来临后，相比单任务模型，MTL 在 CV、NLP 领域有着更好的性能．

MTL 和迁移学习 (transfer learning) 有区别：在迁移学习中，目标任务的性能提升有源任务的功劳，但总目标是提升目标任务的性能，而不是源任务的性能，知识被迁移；而在 MTL 中，每个任务被平等对待，知识被共享．MTL 和持续学习 (continual learning) 有区别：持续学习的任务一个接一个地被学习，而 MTL 的任务同时被学习．MTL 和多标签学习 (multi-label learning)、多任务回归 (multi-output regression) 有一定联系：把每个可能的标签视作一个任务，是 MTL 的特殊情形，但不能等同．MTL 和多视图学习 (multi-view learning) 不同：多视图学习中的每个数据都有多个视图，这些视图仅为训练单个任务．

## MTL Models

给定 $m$ 个任务 $\{\mathcal T_i\}_{i=1}^m$，存在任务互相关联的子集．MTL 的目标就是：利用一些任务所包含的知识，同时学习所有任务，以改进每个任务的学习．

记任务 $\mathcal T_i$ 的训练集 $\mathcal D_i = \{\mathbf x_j^i, y_j^i\}_{j=1}^{n_i}$，其中特征 $\mathbf x_j^i \in \mathbb R^{d_i}$．

如果每个任务的特征空间相同 ($\forall\, i \ne j, \: d_i = d_j$)，则称该 MTL 为 homogeneous-feature MTL，否则称为 heterogeneous-feature MTL．

MTL 可同时含 (无/半) 监督学习、强化学习、多视图学习、图模型的不同类型的多任务，这时是 heterogeneous MTL，反之相同类型的多任务则是 homogeneous MTL．

> 若无特殊说明，本文的 MTL 默认是 homogeneous-feature 的，且为 homogeneous MTL．

1. when to share：是否用多个单任务模型处理多任务？交叉验证计算开销大、数据量要求大，此时宜用 MTL．
2. what/how to share：三种形式．
   - **共享特征**：将输入特征先选择/映射到隐空间里，分别学习隐空间到各任务输出空间映射．
   - **共享样本**：维护所有样本对各任务的权重，加权到各个损失函数上．
   - **共享参数**：预先定义任务的相关性结构 (如对参数空间施加低秩约束后在低维子空间中调参、对任务聚类)、自动学习任务之间的相似性、将参数分解为受正则约束的共享部分和任务特定部分．

目前对 MTL 的研究主要集中在共享特征和共享参数上，针对共享样本的研究较少．

### 特征学习途径

我们假设不同任务之间存在共享的特征表示．

#### 特征变换途径

**线性变换**

:::note[多任务特征学习 (MTFL)]
用正交变换 (旋转、反射，长度角度均不变) 把原始特征映射进共享特征空间 (**硬共享**)．这样一来，多任务的输入空间不再是原来的 $m$ 个特征空间 $\mathbb R_i^d$，转变成 $1$ 个共享特征空间 $\mathbb R^d$．
$$
\begin{aligned}
\min_{\mathbf A, \mathbf U, \mathbf b} & \:\: \sum_{i=1}^{m} \dfrac{1}{n_i} \sum_{j=1}^{n_i} \mathcal L((\mathbf a^i)^\top \mathbf U^\top \mathbf x_j^i + b_i, y_j^i) + \lambda \Vert \mathbf A \Vert_{2,1}^2 \\
\text{s.t.} & \:\: \mathbf U \mathbf U^\top = \mathbf I
\end{aligned}
$$
其中 $\mathbf U^\top$ 对 $m$ 个任务共享，$\mathbf a^i$ 和 $b_i$ 为 $\mathcal T_i$ 私有．
$$
\begin{aligned}
&\mathbb R_i^d \rightarrow \mathbb R^d \rightarrow \mathbb R_i \\
&\mathbf x_j^i \mapsto \mathbf U^\top \mathbf x_j^i \mapsto (\mathbf a^i)^\top \mathbf U^\top \mathbf x_j^i + b_i
\end{aligned}
$$
我们不会让所有任务盲目共享所有特征，而是**仅共享某些特征的子集**，以避免无关特征为训练带来的干扰，实现共享特征的选择．这样的**特征选择**该如何实现呢？「选择」步骤是在共享层后发生的，因此我们可以对 $\mathbf A$ 下手，对其进行约束．如何约束？注意到整个过程是优化过程，也就是最小化损失函数，于是我们可以给损失函数添加正则项，利用「最小化过程」为 $\mathbf A$ 的约束提供源动力．

$\ell_{2,1}$ 正则化
$$
\begin{aligned}
&\mathbf{A} = [\mathbf{a}^1, \mathbf{a}^2, ..., \mathbf{a}^m] \in \mathbb{R}^{d \times m} \\
&\Vert \mathbf A \Vert_{2,1}^2 = (\Vert \mathbf A_{1,:} \Vert_2 + \Vert \mathbf A_{2,:} \Vert_2 + \cdots + \Vert \mathbf A_{d,:} \Vert_2)^2
\end{aligned}
$$
优化过程中，外层的 $\ell_1$ 正则化会倾向于让每行的 $\ell_2$ 范数贴近坐标轴，会尽量让 $\mathbf A$ 的每一行极化：要么使其贴近 $\mathbf 0$，要么不动．这意味着该行对应的特征被所有任务弃用/选择，这样就实现了特征选择．

这里用正则化约束实现无关特征的弃用，有个重要前提：**无关特征被映射到共享特征空间后依旧无关**，否则我们只能弃用共享特征空间的无关特征，而无法关联到原始特征空间．这个前提由 $\mathbf U$ 的正交性保证．

可以证明，上面的优化问题有个等价形式：
$$
\begin{aligned}
\min_{\mathbf W, \mathbf D, \mathbf b} & \:\: L(\mathbf W, \mathbf b) + \lambda \,\text{tr}(\mathbf W^\top \mathbf D^{-1} \mathbf W) \\
\text{s.t.} & \:\: \mathbf D \succeq \mathbf 0, \:\text{tr}(\mathbf D) \le 1
\end{aligned}
$$
其中 $L(\mathbf W, \mathbf b) = \sum_{i=1}^m \frac{1}{n_i} \sum_{j=1}^{n_i} \mathcal L((\mathbf w^i)^\top \mathbf x_j^i + b_i)$，$\mathbf w^i = \mathbf U \mathbf a^i$ 是 $\mathcal T_i$ 的参数．正则项实际上是所有 $\mathbf w^i$ 的马氏距离的平方和，鼓励 $\mathbf W$ 低秩，这与所有任务共享一个低维特征子空间是一致的．当 $\mathbf W$ 固定时，$\mathbf D$ 有解析解
$$
\mathbf D = \frac{\sqrt{\mathbf W^\top \mathbf W}}{\text{tr}\sqrt{\mathbf W^\top \mathbf W}}
$$
:::

与 MTFL 类似，**多任务稀疏编码 (Multi-task Sparse Coding)** 则通过字典 $\mathbf U \in \mathbb R^{d \times D}$ 将特征空间升维至 $D > d$，然后用稀疏的系数向量 $\mathbf a^i$ 进行特征选择
$$
\begin{aligned}
\min_{\mathbf A, \mathbf D, \mathbf b} & \:\: L(\mathbf {UA}, \mathbf b) \\
\text{s.t.} & \:\: \Vert \mathbf a^i \Vert_1 \le \lambda, \: \Vert \mathbf u^j \Vert_2 \le 1, \: \forall\, (i, j) \in [m] \times [D]
\end{aligned}
$$
相比 MTFL，稀疏编码中的每个任务都从这个共享字典中独立地挑选原子 (字典的列)，而不是用行稀疏性鼓励在顶点处取得．

**非线性变换**

我们往往用**大量**共享层来学习共同特征表示，而且共享层不局限于全连接，还可以是**卷积层、池化层**等 (CV 里的共享卷积、NLP 里的共享 BERT 编码器)．

:::tip[对抗学习共同特征]
使用三个网络进行极大极小博弈
$$
\min_{\theta_f, \theta_c} \max_{\theta_d} \sum_{i=1}^m \frac{1}{n_i} \sum_{j=1}^{n_i} \Big( \mathcal L_{\text{task}}(N_{\theta_c}(N_{\theta_f}(\mathbf x_j^i)), y_j^i) - \mathcal L_{\text{ce}} (N_{\theta_d}(N_{\theta_f}(\mathbf x_j^i)), d_j^i) \Big)
$$
其中特征网络 $N_{\theta_{f}}$ 负责学习共同特征，领域网络 $N_{\theta_{d}}$ 负责从共同特征中辨别出这个数据出自哪个任务 (体现在让交叉熵损失 $\mathcal L_{\text{ce}}$ 最小，让整个表达式最大)．这时 $N_{\theta_{f}}$ 会尽可能最小化任务损失，同时要阻止 $N_{\theta_{d}}$ 辨别成功．换言之，$N_{\theta_{f}}$ 学到了共同特征．
:::

:::tip[交叉连接网络 (Cross-stitch Network)]
我们还可以不强制所有任务共享完全相同的底层表示，而是用同一个网络来学习共同特征．

具体地，取每个任务的第 $i$ 层 $\mathbf x_i$，每层取第 $j$ 个节点 $x_{i,j}$，最后得到一个 $m$ 维向量 $(x_{i,j}^1, \cdots, x_{i,j}^m)$．我们考虑在这个时候学习共同特征
$$
\begin{pmatrix}
\tilde x_{i,j}^1 \\
\vdots \\
\tilde x_{i,j}^m
\end{pmatrix}
=
\mathbf A_i
\begin{pmatrix}
x_{i,j}^1 \\
\vdots \\
x_{i,j}^m
\end{pmatrix}
$$
$\mathbf A_i$ 越倾向于对角矩阵，共同特征的学习程度就越低．$\mathbf A_i$ 可以通过反向传播更新．
:::

#### 特征选择途径


### 低秩途径


### 任务聚类途径


### 任务关系学习途径


### 分解途径


