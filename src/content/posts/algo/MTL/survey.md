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

1. 基于 $\ell_{p,q}$ 正则化：
	1. $\ell_{2,1}$ 范数 (行稀疏) $\min_{\mathbf W,\mathbf b} L(\mathbf W, \mathbf b) + \lambda \Vert \mathbf W \Vert_{2,1}$
	2. 加权 $\ell_{2,1}$ 范数 $\min_{\mathbf W,\mathbf b, \{ \gamma_i \}} L(\mathbf W, \mathbf b) + \lambda \sum_{i=1}^d \gamma_i \Vert \mathbf w_i \Vert_2$
	3. 重叠组稀疏
	4. 平方根损失 (对异常值敏感) $\min_{\mathbf W,\mathbf b} \sqrt{L(\mathbf W, \mathbf b)} + \lambda \Vert \mathbf W \Vert_{2,1}$
	5. 安全筛选：优化前筛选出 $\mathbf W$ 的零行，对应着无用特征
	6. $\ell_{\infty, 1}$ 范数 $\min_{\mathbf W,\mathbf b} L(\mathbf W, \mathbf b) + \lambda \Vert \mathbf W \Vert_{\infty,1}$，可以用块坐标下降法
	7. Capped-$\ell_{p,1}$ ​正则化 $\min_{\mathbf W,\mathbf b} L(\mathbf W, \mathbf b) + \lambda \sum_{i=1}^d \min(\Vert \mathbf w_i \Vert_p, \theta)$，$p = 1, 2$。效果是对重要特征 (对应较大的 $\Vert \mathbf w_i \Vert_p$) 不过度惩罚，对无关特征则保持惩罚力度。
2. 基于参数分解：Multi-level Lasso 把特征选择分成全局稀疏和局部稀疏，$\min_{\boldsymbol \theta, \hat {\mathbf W}, \mathbf b} L(\mathbf W, \mathbf b) + \lambda_1 \Vert \boldsymbol \theta \Vert_1 + \lambda_2 \Vert \hat {\mathbf W} \Vert_1, \:\text{s.t.}\: w_{ji} = \theta_j \hat w_{ji}, \theta_j \ge 0$，推广形式 $\min_{\boldsymbol \theta, \hat {\mathbf W}, \mathbf b} L(\mathbf W, \mathbf b) + \lambda_1 \Vert \boldsymbol \theta \Vert_p^p + \lambda_2 \sum_{i=1}^m \Vert \hat {\mathbf w}^i \Vert_q^q$
3. 基于结构先验：
	1. 先验知识：单个特征用于哪些任务的关系满足树结构。所有任务作为叶节点建树，把正则项设计成 $f(\mathbf W) = \sum_{i=1}^d \sum_{v \in V} \lambda_v \Vert \mathbf w_{i,G_v} \Vert_2$，其中 $V$ 是树的所有节点，$G_v$ 是子树所有叶节点，$\mathbf w_{i, G_v}$ 是从 $\mathbf W$ 中选取的 $|G_v|$ 维子向量。这样就实现了层次化地特征选择，在 $\ell_{2,1}$ 的行稀疏基础上实现行内选择的稀疏。
	2. 先验知识：不同任务选用的特征互斥。使用列稀疏 $\min_{\mathbf W,\mathbf b} L(\mathbf W, \mathbf b) + \lambda \Vert \mathbf W \Vert_{1,2}^2$。
4. 基于 Bayesian 模型：正则来自先验。(待学习 Bayesian 统计学后再补充)
	1. 广义正态先验
	2. Horseshoe 先验

### 低秩途径

如果多个任务相关，那么参数矩阵 $\mathbf W \in \mathbb R^{d \times m}$ 应该低秩。可以将每个任务的参数分解为共享低秩子空间和任务特定部分：
$$
\mathbf w^i = \mathbf u^i + \mathbf \Theta^\top \mathbf v^i
$$
其中子空间 $\mathbf \Theta \in \mathbb R^{h \times d}$，$h < d$，此时优化问题为
$$
\begin{aligned}
\min_{\mathbf U, \mathbf V, \mathbf \Theta, \mathbf b} & \:\: L(\mathbf U + \mathbf \Theta^\top \mathbf V, \mathbf b) + \lambda \Vert \mathbf U \Vert_F^2 \\
\text{s.t.} & \:\: \mathbf \Theta \mathbf \Theta^\top = \mathbf I
\end{aligned}
$$
其中 $\Vert \cdot \Vert_F$ 是 Frobenius 范数，$\Vert \mathbf U \Vert_F = \sqrt{\sum_{i=1}^m\sum_{j=1}^n |u_{ij}|^2}$。

凸松弛、迹范数正则化、Capped 迹范数、深度模型的低秩。

### 任务聚类途径

如果多个任务相关，那么可以通过聚类来构建任务相关关系的等价类。

早期通过评估任务间迁移精度构建任务转移矩阵，然后最大化簇内相似度进行聚类。但该方法分两步，可能不是最优的。

Bayesian 方法、正则化、GO-MTL (软聚类与重叠簇)

### 任务关系学习途径

从数据自动学习任务之间的定量关系 ($m \times m$ 相似方阵、协方差矩阵等)

多任务高斯过程 (MTGP)、任务关系学习 (MTRL)、非对称关系、局部学习方法中的人物关系。

### 分解途径

将参数矩阵 $\mathbf W \in \mathbb R^{d \times m}$ 分解为多个分量 $\mathbf W = \sum_{k=1}^h \mathbf W_k$，每个分量受不同的正则化约束 (行列稀疏、低秩(迹范数))，可以简单使用双分量分解，可以多分量分解，可以基于树的全分量分解。

### 各种途径的比较

| 途径     | 核心思想      | 优点                       | 局限性                    |
| ------ | --------- | ------------------------ | ---------------------- |
| 特征学习   | 学习共享特征表示  | **可解释性好**，能处理异构特征；深度模型强大 | 对异常任务敏感，线性变换**表达能力有限** |
| 低秩     | 参数矩阵低秩    | **捕获全局结构**，凸松弛易优化        | **仅适用于线性模型**，扩展非线性困难   |
| 任务聚类   | 任务划分为簇    | 可识别任务群组，**结果直观**         | 多数**需预先指定簇数**，忽略簇间负相关  |
| 任务关系学习 | 学习任务间定量关系 | **可量化任务相似性**，增强可解释性      | 关系矩阵学习可能过拟合，**计算量大**   |
| 分解     | 参数分解为多分量  | **能建模复杂层次结构**，灵活性强       | 分量数需确定，**模型复杂度高**      |
### 基准数据集

| 数据集                 | 任务数 (m) | 总样本量    | 特征维度 (d) | 任务类型        | 特点           |
| ------------------- | ------- | ------- | -------- | ----------- | ------------ |
| **School**          | 139     | 15,362  | 28       | 回归          | 层次结构         |
| **SARCOS**          | 7       | 48,933  | 21       | 回归          | 大规模，共享物理知识   |
| **Computer Survey** | 180     | 36,000  | 13       | 回归          | 任务多，但每个任务数据少 |
| **Parkinson**       | 42      | 5,875   | 19       | 回归          | 纵向           |
| **Sentiment**       | 4       | 8,000   | 高维文本特征   | 分类 (二分类)    | NLP          |
| **MHC-I**           | 35      | 15,236  | 特征工程得来   | 分类 (二分类)    | 任务可聚类        |
| **Landmine**        | 29      | 14,820  | 9        | 分类 (二分类)    | 任务可聚类        |
| **Office-Caltech**  | 4       | 2,533   | 图像深度特征   | 分类 (多类，31类) | CV           |
| **Office-Home**     | 4       | ~15,500 | 图像深度特征   | 分类 (多类，65类) | CV，大规模       |
| **ImageCLEF**       | 4       | ~2,400  | 图像深度特征   | 分类 (多类，12类) | CV           |

任务的选取需满足自然相关性，比如同一领域的重复采样、同一个体的连续采样。

1. **School 数据集**用于预测 139 座学校 (任务) 的学生成绩。共 15,362 位学生 (样本)，每份样本含学校特征和个人特征。
2. **SARCOS 数据集**研究 SARCOS 仿真机器人 7 个关节 (任务) 所需扭矩的逆动力学问题。共 48,933 种运动状态 (样本)，每份样本含 7 个关节的位移、速度、加速度 (21 个特征)。7 个任务共享力学原理。
3. **Computer Survey 数据集**用于预测 180 位消费者 (任务) 对 PC 的评分。共 20 台 PC (样本)，每份样本含 13 个特征 (该 PC 的价格、CPU、RAM 等)。
4. **Parkinson 数据集**用于预测 42 位帕金森患者 (任务) 的症状评分。共 5,875 条纵向记录 (样本)，每份样本含 19 个生物医学特征描述。
5. **Sentiment 数据集**用于评判 4 个产品领域 (任务) 中评论的褒贬。共 2,000 条评论 (样本)，高维文本特征。
6. **MHC-I 数据集**用于评判 35 种 MHC-I 分子 (任务) 能否与给定多肽结合。共 15,236 种多肽 (样本)，特征工程。这 35 种分子按作用机理可分类，因此可任务聚类。
7. **Landmine 数据集**用于评判 29 个雷区 (任务) 雷达图像数据点是否为地雷。共 14,820 个数据点 (样本)，由 9 个特征描述。所有任务中，绿植沙漠各占一半，因此任务可聚类。
8. **Office-Caltech 数据集**用于 4 个领域 (Amazon、DSLR、Webcam、Caltech) (任务) 图像的多分类。共 2,533 张图片 (样本)，10 分类。
9. **Office-Home 数据集**用于 4 个领域 (Art、Clipart、Product、Real-World) (任务) 图像的多分类。约 15,500 张图片 (样本)，65 分类。
10. **ImageCLEF 数据集**用于 4 个图片来源 (任务) 的图像多分类。约 2,400 张图片 (样本)，12 分类。

总体而言，MTL 优于单任务学习；不同数据集适合不同方法，如含簇结构的数据集（School, MHC-I, Landmine）上聚类、关系学习、分解方法表现更好；图像数据集上深度模型占优。

### 正则化 MTL 方法的另一种分类

1. 学习特征协方差：对应 MTFL 途径，特征选择途径等
2. 学习任务关系：对应 MTRL 途径，任务聚类途径等

### MTL 的优化技术

1. 梯度下降及其变体：适用于光滑无约束问题；非光滑时可用次梯度；有约束时用投影梯度。深度 MTL 中常用随机梯度下降、梯度归一化 (GradNorm)、梯度手术等技巧平衡多任务学习。
2. 块坐标下降 (BCD)：交替优化参数块 (如任务参数和关系矩阵)，每步子问题较简单，广泛用于 MTL。
3. 近端方法 (Proximal Method)：处理非光滑正则项 (如 $\ell_{2,1}$​、迹范数)，通过近端算子迭代更新，加速收敛。

## MTL 与其他学习范式的结合

### 半监督学习与主动学习

- 半监督多任务学习：利用无标记数据挖掘几何结构。Liu 用随机游走和狄利克雷过程；Zhang and Yeung 用高斯过程。
- 多任务主动学习：选择对多个任务都有信息量的样本。Reichart 提出两种协议；Fang and Tao 基于置信区间。
- 半监督多任务主动学习：结合两者。

### 聚类

- 多任务 Bregman 聚类：使不同任务的聚类中心相近。
- 多任务核 k-means：学习核矩阵同时最小化任务间 MMD。
- 基于 MTFL 和 MTRL 的聚类：将标签视为待学习的簇指示。

### 强化学习

多任务强化学习 (MRL) 利用任务间共享知识加速学习。方法包括：

- 分层贝叶斯模型、狄利克雷过程；
- 稀疏正则化；
- 策略草图、注意力机制；
- 策略蒸馏和知识迁移；
- 分布式与在线 MRL。

### 多视图学习

多任务多视图学习：每个任务有多个视图。常见方法：

- 图正则化：不同任务在共享视图上预测一致；
- 非负矩阵分解：同时考虑视图内聚类、视图间一致性和任务间共享子空间；
- 深度交叉连接网络：融合多视图特征。

### 图模型

- 多任务贝叶斯网络结构学习：共享先验，启发式搜索；
- 联合学习多个高斯图模型：通过 $\ell_{\infty, 1}$ 正则迫使精度矩阵联合稀疏；
- 特征交互矩阵学习：用 $\ell_{2, 1}$ 或张量迹范数建模任务间特征交互。

## 展望

未来方向包括：

1. 异常任务处理：现有方法虽能减轻负影响，但缺乏系统理论和安全保证。
2. 深度多任务模型：当前深度模型多为硬共享，对异常任务脆弱，需要设计更鲁棒灵活的深度结构。
3. 非监督任务拓展：将五类方法推广到半监督、主动、强化、多视图等任务，以及逻辑、规划等 AI 领域。
