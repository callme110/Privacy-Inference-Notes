---
title: "Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation"
authors:
  - "Yoshua Bengio"
  - "Nicholas Léonard"
  - "Aaron Courville"
publication:
  - "arXiv:1308.3432v1, 2013-08-15"
tags:
  - paper-note
  - cryptographic-neurons
  - stochastic-neurons
  - straight-through-estimator
  - reinforce
  - conditional-computation
  - gradient-estimation
  - sparse-gating
  - type/论文
description: "经典 stochastic neuron 梯度估计论文，系统比较 noisy rectifier、STS、REINFORCE 型无偏估计与 straight-through estimator，并把它们用于 conditional computation 稀疏门控。"
source: "https://arxiv.org/abs/1308.3432"
local_pdf: "./Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation.pdf"
zotero_collection: "Research on Cryptographic Neurons"
status: "初读"
created: "2026-06-23"
---

# Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation

原论文链接：[arXiv:1308.3432](https://arxiv.org/abs/1308.3432)

本地 PDF：[Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation.pdf](./Estimating%20or%20Propagating%20Gradients%20Through%20Stochastic%20Neurons%20for%20Conditional%20Computation.pdf)

上位地图：[[MOC - 计算机]] · [[Research on Cryptographic Neurons]]

相关主题：[[Stochastic Neuron]]、[[Straight-Through Estimator]]、[[REINFORCE]]、[[Conditional Computation]]、[[Sparse Gating]]、[[Gradient Estimator]]

### Abstract

这篇 2013 年论文讨论一个至今仍很重要的问题：如果神经元输出是随机的、二值的，或者包含 hard non-linearity，反向传播该如何穿过去？

普通 backprop 需要计算链式法则。如果一个神经元输出是硬阈值：

$$
h=\mathbf{1}\{a>0\},
$$

那么它几乎处处导数为 0，梯度无法流动。论文比较四类解决方案：

- noisy rectifier：在非线性中注入噪声，使某些区域仍有可用梯度；
- STS unit：Stochastic Times Smooth，用随机门控乘上平滑量；
- stochastic binary neuron 的 REINFORCE 型无偏估计；
- straight-through estimator：反向传播时假装硬阈值是恒等函数。

论文把这些估计器用于 conditional computation：用稀疏随机门控决定哪些专家单元需要被计算，从而减少每个样本访问的参数量。

### Knowledge

#### 1. 随机神经元的基本形式

论文把随机神经元写成：

$$
h_i=f(a_i,z_i),
$$

其中 \(a_i\) 是输入和参数的可微函数，\(z_i\) 是噪声源。若 \(f\) 对 \(a_i\) 有非零导数，普通 backprop 仍可用；若 \(f\) 是二值采样，导数路径断裂。

#### 2. Noisy rectifier

带噪声 rectifier 可写成：

$$
h_i=\max(0,a_i+z_i).
$$

如果 \(z_i\) 服从 logistic noise，论文给出：

$$
\Pr(h_i>0)=\sigma(a_i),
$$

$$
\mathbb{E}[h_i]=\log(1+\exp(a_i)).
$$

直观上，噪声让本来死掉的 hard region 偶尔被激活，从而有机会收到梯度信号。

#### 3. STS unit

STS（Stochastic Times Smooth）定义为：

$$
p_i=\sigma(a_i),
$$

$$
b_i\sim \mathrm{Bernoulli}(\sqrt{p_i}),
$$

$$
h_i=b_i\sqrt{p_i}.
$$

它满足：

$$
\mathbb{E}[h_i]=p_i.
$$

STS 的思想是：前向有稀疏随机性，期望上仍近似平滑函数，因而可用普通梯度训练。

#### 4. Stochastic binary neuron 的无偏估计

若

$$
h_i\sim \mathrm{Bernoulli}(\sigma(a_i)),
$$

且损失为 \(L\)，论文证明：

$$
\hat g_i=(h_i-\sigma(a_i))L
$$

是

$$
\frac{\partial \mathbb{E}[L]}{\partial a_i}
$$

的无偏估计。这是 Bernoulli 情况下的 REINFORCE / score-function estimator。

还可以加入 baseline 降低方差：

$$
\hat g_i=(h_i-\sigma(a_i))(L-\bar L_i),
$$

其中最小方差常数为：

$$
\bar L_i=\frac{\mathbb{E}[(h_i-\sigma(a_i))^2L]}{\mathbb{E}[(h_i-\sigma(a_i))^2]}.
$$

#### 5. Straight-through estimator

Straight-through estimator 的做法很粗暴：

$$
\frac{\partial L}{\partial a_i}\approx \frac{\partial L}{\partial h_i}.
$$

也就是前向使用 hard sample，反向假装 hard threshold 是 identity。它有偏，但实现极简单，并且在实验中表现很好。

### Method

论文不是只提出一个估计器，而是在“无偏但方差高”和“有偏但可用”之间画出一张地图：

| 方法 | 梯度性质 | 直观优点 | 主要代价 |
|---|---|---|---|
| REINFORCE 型估计 | 无偏 | 理论干净，不需要可微路径 | 方差高 |
| baseline/centered estimator | 无偏 | 降低方差 | 需要估计 baseline |
| noisy rectifier | 可 backprop | 梯度偶尔流动 | 可能有死单元 |
| STS | 期望平滑 | 稀疏与平滑折中 | 设计较特殊 |
| straight-through | 有偏 | 极简、效果强 | 理论不严格 |

### Experiments

实验使用 MNIST conditional computation 架构。gater path 产生稀疏门控 \(h_i\)，expert/main path 产生隐藏单元 \(H_i\)，最终使用：

$$
H_i h_i.
$$

目标是让门控大部分时间为 0，只计算少量专家。实验设置中，gater 输出平均 10% 非零。

结果要点：

- 所有测试方法都能训练。
- Straight-through 获得最好的验证/测试错误率之一。
- noisy rectifier 优于无噪声 rectifier baseline。
- 噪声不只是 regularizer，也可能帮助探索参数空间。
- 条件计算的预期计算节省可以实现，且没有明显性能损失。

### Insights

#### 1. 无偏不是唯一美德

REINFORCE 型估计无偏，但高方差。Straight-through 有偏，却常常更实用。这是深度学习里很常见的交换：优化过程关心的不只是统计正确性，还关心信号是否稳定、方向是否有用、实现是否简单。

#### 2. 随机门控让模型按样本选择计算路径

Conditional computation 的目标是：

$$
\text{每个样本只激活模型的一小部分参数}.
$$

这和现代 mixture-of-experts、动态路由、token pruning 有直接血缘关系。门控单元像铁路道岔：不是每列车都走所有轨道，而是根据输入选择少数线路。

#### 3. 噪声可以是探索机制

论文观察到 noisy baseline 甚至在训练目标上也可能优于无噪声版本。这说明噪声不只防过拟合，还可能帮助模型逃离坏的 hard decision 区域。

### Critical Reading

#### Strengths

- 清楚区分无偏估计、低方差估计、启发式估计。
- 给出了 stochastic binary neuron 的简洁无偏梯度证明。
- 解释了 straight-through estimator 的经验价值和理论风险。
- 把估计器放入 conditional computation 场景，而不是孤立讨论。

#### Limitations

- 实验规模较小，主要是 MNIST 架构，不能直接代表现代大模型。
- Straight-through 的多层深度情形没有严格保证。
- 条件计算节省依赖硬件/实现是否真的跳过未激活专家。
- baseline 估计和方差控制在复杂网络中可能比公式看起来更难。

### 用户可能“不知道自己不知道”的背景

#### 1. Pathwise gradient 与 score-function gradient

若随机变量可以写成可微重参数化：

$$
h=g(a,\epsilon),
$$

可以用 pathwise gradient。若是离散采样，常用 score-function：

$$
\nabla_\theta \mathbb{E}_{h\sim p_\theta}[L(h)]
=
\mathbb{E}[L(h)\nabla_\theta \log p_\theta(h)].
$$

REINFORCE 属于后者。

#### 2. Straight-through 为什么会流行

ST estimator 不严格，但在二值网络、量化、离散 latent variable、VQ-VAE、hard attention 中都反复出现。原因是它把“训练时需要梯度”和“前向时需要离散决策”这两个需求硬接起来。

#### 3. Conditional computation 和计算安全也有关联

动态门控会改变执行路径。对安全系统而言，执行路径可能成为 side channel；对效率系统而言，它是节省算力的来源。相同机制在不同语境下可能是优势，也可能是泄漏面。

### 可沉淀到 `03_Knowledge` 的原子概念

- [[Stochastic Neuron]]
- [[Straight-Through Estimator]]
- [[REINFORCE]]
- [[Score Function Estimator]]
- [[Conditional Computation]]
- [[Sparse Gating]]
- [[Noisy Rectifier]]

### Sources

- arXiv：https://arxiv.org/abs/1308.3432
- 本地 PDF：`Research on Cryptographic Neurons/Papers/Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation/Estimating or Propagating Gradients Through Stochastic Neurons for Conditional Computation.pdf`

## 标签

#status/进行中 #type/笔记 #type/论文
