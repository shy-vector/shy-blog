---
title: 'Multi-objective Deep Learning: Taxonomy and Survey of the State of the Art'
published: 2026-02-21
updated: 2026-02-21
description: '一篇多目标深度学习综述的阅读笔记'
image: ''
tags: ['ML', 'DL', 'MOO']
category: 'Deep Learning'
draft: false
---

# 多目标深度学习：技术分类和调查

[Multi-objective Deep Learning: Taxonomy and Survey of the State of the Art](https://arxiv.org/abs/2412.01566)

## 摘要

多目标优化的应用：

1. 多任务学习 (Multi-Task Learning)
2. 考虑稀疏性 (sparsity) 等次要目标
3. 多准则超参数调优 (multicriteria hyperparameter tuning)

成本挑战：
1. 参数数量庞大
2. 非线性程度高
3. 有一定随机性

总结内容：
1. 三种学习范式：监督、无监督、强化
2. 生成模型

## Ⅰ 引入

Pareto 最优：无法再在同一时刻改进所有目标

1. ML 中的多准则：用单一模型解决多任务
2. 次要目标：稀疏性、稳定性、可解释性
3. RL 也有多目标优化的需求

多目标优化和深度学习的计算开销都很大，两者的结合在算法效率和决策都存在挑战

Section Ⅱ 预备知识 A 深度学习 (1 监督，2 无监督，3 生成模型)，B 多目标优化 C 多目标机器学习的细节\
Section Ⅲ 提供了途径的分类\
Section Ⅳ 调查当前技术状况：A 监督，B 无监督，C 生成模型，D 神经网络架构，E 多目标深度学习的成功应用\
Section Ⅴ 强化学习\
Section Ⅵ 用深度学习反过来改进多目标优化

## Ⅱ 预备知识

### A 深度学习

主要介绍各种机器学习算法的共性．

#### 1 监督学习

用含参映射 $f_\theta$ 拟合未知映射
$$
\begin{aligned}
f: \mathbb R^n &\rightarrow \mathbb R^m \\
             x &\mapsto y
\end{aligned}
$$
其中 $\theta \in \mathbb R^q$，通常维度很高．

> $f_\theta$ 的构造可以选择使用多项式、Fourier 序列、Gaussian 过程．

而在深度神经网络中，$f_\theta$ 通过 $\ell$ 层构造
$$
\begin{aligned}
z^{(0)} &= x, \\
z^{(j)} &= \sigma^{(j)}(W^{(j)}z^{(j-1)} + b^{(j)}), \quad j = 1, \cdots \ell, \\
      y &= z^{(\ell)}
\end{aligned}
$$
网络的边权都是参数 $\theta = \{(W^{(j)}, b^{(j)})\}_{j=1}^{\ell}$．

> 上面是经典的 feed-forward 结构，除此之外还有：卷积 (convolutional) 神经网络 CNN，残差 (residual) 神经网络 ResNet，带反馈回路的循环 (recurrent) 神经网络 RNN．

现有含 $N$ 个样本的数据集 $\mathcal D = \{(x_i, y_i)\}_{i=1}^N$，拟合映射 $f$ 的问题就变成了高维的非线性优化问题：最小化经验损失函数 $L(\theta)$
$$
\theta^* = \arg\min_{\theta \in \mathbb R^q} L(\theta)
$$
$L(\theta)$ 的形式自定，如均方差损失 (MSE)
$$
L(\theta) = \dfrac{1}{N}\sum_{i=1}^{N}  \Vert y_i - f_\theta(x_i)  \Vert_2^2
$$

> 可以为了稀疏性添加正则项

获取 $\theta^*$ 的经典方法是基于梯度优化的反向传播，因此梯度 $\nabla L(\theta)$ 在训练中扮演重要角色．

> 随机梯度 (SGD) 、动量 (Momentum) 、Adam 优化器

#### 2 无监督学习、自监督学习

**无监督学习**

从无标签数据中提取有意义的表示．

1. 无监督学习直接识别数据的潜在结构
2. 自监督学习先利用数据生成伪标签，得到代理任务 (surrogate/pretext task) 后转化为监督学习

无监督学习通常包括聚类、降维、密度估计．

:::note[聚类]
将数据集 $\mathcal D = \{(x_i)\}_{i=1}^{N}$ 分成 $N_C$ 类．常见的优化目标是最小化簇内方差和最大化簇间间隔
$$
\min_{C_1, \cdots, C_K} \sum_{k=1}^{N_C}\sum_{x_i \in C_k}  \Vert x_i - \mu_k  \Vert^2
$$
其中 $\mu_k = \frac{1}{|C_k|} \sum_{x_i \in C_k} x_i$ 为簇心．解决这个优化目标的传统技术有 K-means 和 Gaussian 混合模型，而在数据维度非常高 (如图像) 的情况下，现代的神经聚类则是先将原始数据编码到低维隐空间后再传统聚类．
:::

:::note[降维]
在保持尽可能多信息的情况下，为高维数据找到低维表示．自编码器 (AutoEncoders，AE) 利用神经网络构建了两个映射，$f_\theta$ 将输入空间编码至隐空间，$g_\phi$ 将隐空间解码至输出空间
$$
\begin{aligned}
f_\theta: \mathbb R^n &\rightarrow \mathbb R^m \\
x &\mapsto z \\
g_\phi: \mathbb R^m &\rightarrow \mathbb R^n \\
z &\mapsto \tilde{x}
\end{aligned}
$$
最小化重构损失
$$
\min_{\theta, \phi} \sum_{i=1}^{N}  \Vert x_i - g_\phi(f_\theta(x_i))  \Vert_2^2
$$
便得到内在的 $\mathbb R^m$ 低维结构，它编码了数据集的信息．
:::

:::note[密度估计]
把输出视作概率分布，根据输入，估计每个数据点的概率密度，得到输出．常见方法有核密度估计 (Kernel Density Estimation，KDE，用若干基本核函数 (如高斯核函数) 来拟合目标分布函数) 、变分自编码器 (VAE，从隐空间得到是分布 (通常是多元 Gaussian 分布，含一系列均值和方差) 而不是单个向量，采样出向量后再解码) ．因为是用分布描述输出，常用于异常检测、数据生成．
:::

**自监督学习**

自监督预训练阶段
$$
\theta^* = \arg\min_{\theta} \mathbb E_{x \sim \mathcal D_{\text{unlabeled}}}[\mathcal L_{\text{pretext}}(f_\theta(g(x)), \tau(x))]
$$
迁移至下游任务
$$
\phi^* = \arg\min_{\phi}\mathbb E_{(x, y) \sim \mathcal D_{\text{labeled}}}[\mathcal L_{\text{down}}(h_\phi(f_{\theta^*}(x)), y)]
$$
用于视频帧预测、确定图像旋转角、重建数据掩码部分

:::note[对比学习]
$g$ 用于数据增强，伪标签生成函数 $\tau$ 并不是显式的（如旋转、掩码）：同一张图的两个增强视图为正样本，不同图为负样本
$$
\min_{\theta} \mathbb E_{x \sim \mathcal D} \mathbb E_{x^+ \sim \text{aug}(x) \atop \{x_i^-\}_{i=1}^{N} \sim \mathcal D } \left[ -\log \frac{\exp(\text{sim}(f_\theta(x^+), f_\theta(x)) / \tau)}{\exp(\text{sim}(f_\theta(x^+), f_\theta(x)) / \tau) + \sum_{i=1}^{N} \exp(\text{sim}(f_\theta(x_i^-), f_\theta(x)) / \tau)} \right]
$$
其中 $\text{aug}(x)$ 表示增强样本，$\text{sim}$ 为余弦相似．
:::

:::note[生成式模型]
$g$ 是破坏函数 (掩码、噪声、下采样) ，$f_\theta$ 是重建函数，$\mathcal L$ 是重建损失．
$$
\min_\theta \mathbb E_{x \sim \mathcal D}[\mathcal L(f_\theta(g(x)), x)]
$$
比如掩码自编码 (MAE)
$$
\mathcal L_\text{MAE} = \mathbb E_{x \sim \mathcal D} [ \Vert x_{\text{masked}} - g_\theta(f_\phi(x_{\text{visible}}))  \Vert_2^2]
$$
其中 $x_\text{visible} = x \odot M$，$x_{\text{masked}} = x \odot (1 - M)$，$M \in \{0, 1\}^{d}$．MAE 认为：还原 = 理解．
:::

#### 3 生成式模型

使用训练样本 $x \sim \mathcal X$ 学习未知分布 $\mathcal X$，这和自监督学习框架高度相关．为此构造映射 $g: \mathbb R^p \rightarrow \mathbb R^n$，使得 $g(z) \sim \mathcal X$，其中 $z \sim \mathcal Z$ 来自更简单的分布 $\mathcal Z$．

:::note[生成式对抗网络 (GAN)]

生成器 $g_\theta : \mathbb R^p \rightarrow \mathbb R^n$，判别器 $f_\phi : \mathbb R^n \rightarrow [0,1]$ 是二分类器．

判别器希望将 $x \sim \mathcal X$ 识别为正类
$$
\begin{aligned}
\mathcal L_D &= -\mathbb E_{x \sim \mathcal X} [1 \cdot \log f_\phi(x) + 0 \cdot \log (1 - f_\phi(x))] \\
&= -\mathbb E_{x \sim \mathcal X} [\log f_\phi(x)]
\end{aligned}
$$
生成器希望让 $g_\theta(z)$ 被识别为正类
$$
\begin{aligned}
\mathcal L_G &= -\mathbb E_{x \sim \mathcal X} [1 \cdot \log f_\phi(g_\theta(x)) + 0 \cdot \log (1 - f_\phi(g_\theta(x)))] \\
&= -\mathbb E_{x \sim \mathcal X} [\log f_\phi(g_\theta(x))]
\end{aligned}
$$
二者博弈
$$
\begin{aligned}
(\theta^*, \phi^*) &= \arg\min_{\theta, \phi} (\mathcal L_D + \mathcal L_G) \\
&= \arg\min_{\theta, \phi} (-\mathbb E_{x \sim \mathcal X} [\log f_\phi(x)] -\mathbb E_{x \sim \mathcal X} [\log f_\phi(g_\theta(x))]) \\
&= \arg\min_{\theta}\max_{\phi}(\mathbb E_{x \sim \mathcal X} [\log f_\phi(x)] + \mathbb E_{x \sim \mathcal X} [\log (1 - f_\phi(g_\theta(x)))]) \\
&= \arg\min_{\theta}\max_{\phi} \mathcal L_{\text{GAN}}
\end{aligned}
$$
:::

### B 多目标优化

#### 1 概念

考虑含 $K$ 个目标的 MOP 问题
$$
\min_{\theta \in \mathbb R^q} L(\theta) = \min_{\theta \in \mathbb R^q} \begin{bmatrix} L_1(\theta) \cdots L_K(\theta) \end{bmatrix}^\top
$$
一般不存在点 $\theta^*$ 使得所有目标 $L_k$ 都达最小值，故考虑 Pareto 集
$$
\mathcal{P} =
\left\{
\theta \in \mathbb{R}^q
~\middle|~
\nexists\,\hat{\theta}, ~~
\begin{array}{}
L_k(\hat{\theta}) \leq L_k(\theta) & \text{for~} k = 1,\cdots,K, \\
L_k(\hat{\theta}) < L_k(\theta) & \text{for at least one~}k
\end{array} ~
\right\}
$$
相应地，$\mathcal P$ 在目标空间 $\mathbb R^K$ 的像 $\mathcal P_\mathcal F = L(\mathcal P)$ 被称作 Pareto 前沿 (一种流形，而非离散点集)．在光滑假设下，$\dim \mathcal P_\mathcal F = K - 1$，比如双目标问题下，Pareto 前沿是曲线 (递减，可不凸) ．在深度学习中，$q$ 非常大，而 $K$ 通常比较小，因此一般对 Pareto 前沿进行可视化，而不是 Pareto 集．

> 特别地，如果存在不冲突的目标，则 Pareto 前沿的维数会减小．比如双目标问题中，两个目标不冲突，可同时达到最小值，此时 Pareto 前沿会从一条曲线坍缩成一个点．

点 $\hat\theta$ 对于所有目标至少都和点 $\theta$ 一样好，且对于至少一个目标严格比点 $\theta$ 更优，则称点 $\hat\theta$ 支配点 $\theta$，记作 $\hat\theta \prec \theta$．因此 $\mathcal P$ 包含了所有的非支配点．

#### 2 梯度以及最优性条件

称满足 KKT 条件 (一阶最优条件) 的点 $\theta^*$ 为 Pareto 临界点
$$
\exists\, \alpha^* \in \mathbb R_{\ge 0}^q, \: \sum_{k=1}^K \alpha_k^* = 1, \: \sum_{k=1}^K \alpha_k^* \nabla L_k(\theta^*) = 0
$$
记 Pareto 临界点集为 $\mathcal P_c$，则必有 $\mathcal P \subseteq \mathcal P_c$．

> 否则，必存在所有梯度的凸组合，使得往该组合行进时，所有目标变优．可想象从原点出发的三个梯度向量构成三棱锥，沿着终点落在底面的向量行进，可使三个目标均变优．

当目标函数 (比如 ReLU、L1 正则化) 不光滑 (不满足 Lipschitz 梯度连续性) ，仅满足 Lipschitz 连续性时，我们无法定义梯度 $\nabla L_k$ ．如果目标函数是凸函数，我们可以定义次梯度 (subdifferential)
$$
\partial L_k = \{g \in \mathbb R^q : \forall\, y \in \mathbb R^q, f(y) \ge f(x) + g^\top (x - y)\}
$$
其存在性由支撑超平面定理保证．此时我们有 KKT 条件的非平滑版本
$$
0 \in \text{conv} \bigcup_{k=1}^K \partial L_k(\theta^*)
$$

:::tip[一些次微分的例子]
$$
\partial\, \text{ReLU}(x) = \begin{cases} \{0\}, & x < 0, \\ [0, 1], & x = 0, \\ \{1\}, & x > 0. \end{cases}
$$
:::

:::important[Lipschitz 连续性]
$$
\exists\, L \in \mathbb R, \forall x, y \in \mathbb R^q,  \Vert f(x) - f(y)  \Vert_2 \le L  \Vert x - y  \Vert_2
$$
最小的 $L$ 就是 Lipschitz 常数 $k_0 = \sup_{x \ne y}  \Vert f(x) - f(y)  \Vert_2 /  \Vert x - y  \Vert_2$．

设 $y = x + \xi$，使用一阶 Taylor 展开
$$
f(x + \xi) \approx f(x) + \nabla f(x)^\top \xi
$$
便有
$$
\begin{aligned}  \Vert \nabla f(x)^\top \xi  \Vert_2 &\le  \Vert \nabla f(x)  \Vert_2 \cdot  \Vert \xi  \Vert_2 \\  \Vert \nabla f(x)^\top \xi  \Vert_2 &\le k_0  \Vert \xi  \Vert_2 \end{aligned}
$$
第二条不等式恒成立，且第一个等号可以成立，因此
$$
\Vert \nabla f(x)  \Vert_2 \le k_0
$$
得 $k_0 = \max  \Vert \nabla f(x)  \Vert_2$，此时 $f(y)$ 可以被 bound 住
$$
f(x) - k_0 \xi \le f(y) \le f(x) + k_0 \xi
$$
光滑（Lipschitz 梯度连续）
$$
\exists\, L \in \mathbb R, \forall x, y \in \mathbb R^q,  \Vert \nabla f(x) - \nabla f(y)  \Vert_2 \le L  \Vert x - y  \Vert_2
$$
最小的 $L$ 便是凸优化里面的 smoothness $k_1$．
:::

#### 3 方法

**a 计算单个 Pareto 最优解 (先决策后优化)**

**标量化**：构造映射 $L(\theta) \mapsto \hat L(\theta) \in \mathbb R$，变成单目标优化．如果是直接加权，就相当于用移动的超平面去截 Pareto 前沿，这只有在凸函数的情况下才能保证解的唯一性．

**$\epsilon$-constraint 方法**：钦定某个目标 $L_k$ 作单目标优化，其他目标变成约束
$$
\min_{\theta \in \mathbb R^q} L_k(\theta) \quad \text{s.t.} \quad \forall\, i \in \{1, \cdots, K\} \setminus k, \: L_i(\theta) \le \epsilon_i
$$

**偏好向量法**：给定参考点和偏好向量 $p, r \in \mathbb R^q$，从参考点出发，在目标空间中沿着偏好向量方向，向原点行进，找到路程最长的终点
$$
\max_{t \in \mathbb R, \theta \in \mathbb R^q } t \quad \text{s.t.} \quad p + tr - f(\theta) \in \mathbb R_+^K
$$
这种方法能很好应对非凸问题，但约束条件较难处理．

:::note[多梯度下降法 (MGDAs)]
关键是共同梯度 $d(\theta) \in \mathbb R^q$ 的选取
$$
d(\theta) = - \sum_{k=1}^K w_k \nabla L_k(\theta)
\quad \text{where} \quad w = \arg\min_{\hat w \in [0, 1]^K \atop \sum_{k=1}^K \hat w_k = 1} \left \Vert \sum_{k=1}^K \hat w_k \nabla L_k(\theta) \right\Vert_2^2
$$
不断更新 $\theta$ 至收敛 (或者触发给定的停止条件)
$$
\theta^{(i+1)} = \theta^{(i)} + \eta(\theta^{(i)}) \cdot d(\theta^{(i)})
$$
也可以使用 (Quasi-)Newton 法确定 $d(\theta)$．传统的 MGDAs 只能保证结果收敛到 $\mathcal P_c$ 中的某个点，无法得知这个点每个目标的重要程度，因此对于运行多次 MGDAs 得到的多个点，我们无法作出最终决定．
:::

**b 计算整个 Pareto 集 (先优化后决策)**

多次调整权重后分别运行标量法，或者运行多起点的 MGDAs (彼此孤立)，都无法很好地覆盖 $\mathcal P$ (用离散点集近似 $\mathcal P$)．

定义种群 $P = \{\theta^{(j)}\}_{j=1}^M$，其 performance 一般使用超体积指标衡量
$$
\text{HV}(P) = \lambda \left( \bigcup_{\theta \in P} [L(\theta), r] \right)
$$
其中 $\lambda (\cdot)$ 为 Lebesgue 测度，$r \in \mathbb R^q$ 为参考点 (通常选取目标空间的最大值点) ．

:::note[多目标进化算法 (MOEAs)]
1. 交叉 (crossover)：第 $i$ 代 $P(i)$ 中两个体 $\theta_{1,2}$ 交叉，得到新种群 $\widehat P(i)$；
2. 变异 (mutation)：$\widetilde P(i) = \mathcal M(\widehat P(i))$；
3. 选择 (selection)：适者生存 (按非支配关系 (non-dominance)、分散指标 (spread metric) 排序)，$P(i+1)$ 可来自 $P(i) \cup \widetilde P(i)$ (精英主义)．
:::

多任务学习是机器学习一个重要板块，哪怕在最基础的分类任务中，保持模型准确率和模型复杂度的平衡也是社区讨论的重点．之前的多任务学习文章 (Uncertainty; GradNorm; MGDA) 主要关注的是如何在有多个任务时找到**一个**能够权衡利弊的解，但由于 Pareto 的存在，一个独立解是难以满足**多种**偏好的．

最近的多任务学习文章 (Pareto MTL) 开始意识到多个 Pareto 解的重要性，但由于没有利用 Pareto 前沿的性质，该方法生成的每一个解都是从头训练的，并且无法形成连续的 Pareto 前沿．

:::important[延拓法]
我们试图解决的，就是利用 **Pareto 前沿的连续性质**，连续地**从一个 Pareto 解出发**找到其他的解．

考虑将 KKT 条件转化为零值问题
$$
H(\theta^*, \alpha^*) =
\begin{pmatrix}
\sum_{k=1}^K \alpha_k^* \nabla L_k(\theta^*) \\
\sum_{k=1}^K \alpha_k^* - 1
\end{pmatrix}
= 0
$$
如果有二阶连续可微的 $C^2$ 正则假设，那么隐函数定理保证
$$
\mathcal M = \{(\theta, \alpha) \in \mathbb R^{q+K} : H(\theta, \alpha) = 0\}
$$
是 $\dim(\mathcal M) = (q + K) - (q + 1) = K - 1$ 维的光滑流形．我们先用某种手段得到一个 Pareto 临界点 $(\theta_0, \alpha_0)$，然后求解此处的切空间，这需要 Hessian 矩阵
$$
\mathbf H = H'(\theta^*, \alpha^*) =
\begin{pmatrix}
\sum_{k=1}^K \alpha_k^* \nabla^2 L_k(\theta^*) & L_1(\theta^*) & \cdots & L_K(\theta^*) \\
0 \quad \cdots \quad 0 & 1 & \cdots & 1
\end{pmatrix}
\in \mathbb R^{(q+1) \times (q+K)}
$$
切空间就是 $\mathbf H$ 的核
$$
\text{ker}\, \mathbf H = \{v \in \mathbb R^{q+K} : \mathbf Hv = 0\}
$$
求出 $v$ 后，便可得到临界点的附近 (**延拓**的点集)
$$
(\theta, \alpha) = (\theta_0, \alpha_0) + tv, \quad t \in (-\epsilon, \epsilon)
$$
> 1. 存储代价 $O(q^2)$ 无法接受，通常不直接求解 $J$，而使用 Hessian 向量积 $\nabla \nabla^\top L_k(\theta^*) v = \nabla (v^\top \nabla L_k(\theta^*))$．
> 2. 另外，$C^2$ 正则性都被许多网络结构破坏掉了 (ReLU)，虽然有文献给出了仅满足 Lipschitz 连续性的目标的解决办法，但这会提高计算开销．
> 3. 延拓点一般会稍微脱离 Pareto 前沿，于是我们会有修正步骤，比如从延拓点出发，使用 MGDA 得到正确的临界点．
:::

**c 交互式方法**

决策者不是一次性给出所有偏好，而是在优化过程中根据当前解的情况，动态调整偏好，决策和优化不断交替．

比如我们观察当前 $\theta^{(t)}$ 和 $L(\theta^{(t)})$ 的情况，可能会希望改善 $L_1$，接受 $L_3$​ 变差，但 $L_2$​ 不能恶化．这反映在延拓法里，就是在切空间 $\ker \mathbf H$ 中选择方向导数满足
$$
\frac{\partial L_1}{\partial v} > 0, \quad \frac{\partial L_2}{\partial v} \ge 0, \quad \frac{\partial L_3}{\partial v} < 0
$$
的 $v$ 进行延拓．

### C 多目标机器学习

将多目标优化和机器学习结合不是新鲜事．尽管从数据中学习模型时会遇到多种性能指标，我们一般不将多目标优化应用在深度学习中，因为两者计算开销都非常大．针对深度学习，各种多目标优化算法的**痛点**如下：

1. 标量化虽然转化为单目标，但会新增约束条件而较难处理；
2. MGDA 收敛速度与单目标相当，但每轮迭代都需要计算步进方向；
3. MOEAs 实现了全局优化，而且不用计算梯度，但庞大的种群个体数会带来个体评估的开销，收敛速度也缓慢；
4. 延拓法收敛速度很快，但 Hessian 矩阵的计算开销很大，并且通常没有 $C^2$ 正则性．

深度学习能跟多目标优化的**结合点**：

1. 特征选择
2. 超参数调优
3. 结构搜索
4. 数据插补 (data imputation)
5. 多目标训练：支持向量机 (SVM)、决策树、Bayesian 分类器、径向基 (Radial Basis Function，RBF) 神经网络、聚类
6. 多目标聚类

## Ⅲ 多目标深度学习的分类

