---
title: "Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial?"
authors:
  - "Akira Ito"
  - "Takayuki Miura"
  - "Yosuke Todo"
publication:
  - "arXiv:2510.06692v2, 2026-03-27"
tags:
  - paper-note
  - cryptographic-neurons
  - model-extraction
  - hard-label-attack
  - relu-network
  - cryptanalysis
  - persistent-neuron
  - cross-layer-extraction
  - type/论文
description: "分析 hard-label cryptanalytic model extraction 的多项式时间 claim：persistent neurons 可能使交点收集随深度指数变难，作者提出 cross-layer extraction 作为补救。"
source: "https://arxiv.org/abs/2510.06692"
local_pdf: "./Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial.pdf"
zotero_collection: "Research on Cryptographic Neurons"
status: "初读"
created: "2026-06-23"
---

# Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial?

原论文链接：[arXiv:2510.06692](https://arxiv.org/abs/2510.06692)

本地 PDF：[Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial.pdf](./Is%20the%20Hard-Label%20Cryptanalytic%20Model%20Extraction%20Really%20Polynomial.pdf)

上位地图：[[MOC - 计算机]] · [[Research on Cryptographic Neurons]]

相关主题：[[Model Extraction]]、[[Hard-Label Attack]]、[[ReLU Network]]、[[Cryptanalytic Model Extraction]]、[[Persistent Neuron]]、[[Cross-Layer Extraction]]

### Abstract

这篇论文回应一条很强的结论：只给 hard-label 输出，也就是只看到模型最后分类标签，能否在多项式时间内抽取 ReLU 网络参数？

前序工作把 ReLU network extraction 类比为密码分析：ReLU 类似 S-box，线性层类似 affine/linear layer，模型参数类似 secret key。Carlini 等工作声称，在 hard-label setting 下也能多项式时间抽取模型。

本文的主张更谨慎：这个多项式时间结论依赖一个隐含假设，即攻击者能用多项式查询为每个目标神经元收集足够多 activation boundary intersection points。但随着深度增加，一些神经元可能几乎总是 active 或 inactive。作者称这些为 persistent/dead neurons。若某个 persistent neuron 很少切换状态，观察到它的边界可能需要指数级查询。

一句话概括：

> Hard-label extraction 的难点不只是“看不到 logits”，而是深层 ReLU 的部分边界可能几乎不可达。

### Knowledge

#### 1. ReLU 网络和 block cipher 的类比

ReLU MLP 可以写成：

$$
f^{(\ell)}(x)=\sigma(W^{(\ell)}f^{(\ell-1)}(x)+b^{(\ell)}),
$$

其中 \(\sigma(t)=\max(0,t)\)。它和 block cipher 的类比是：

- linear/affine layer 对应 \(W,b\)；
- nonlinear layer 对应 ReLU；
- 参数对应 secret material；
- oracle access 对应只能查询输入输出。

但差别也很关键：DNN 是 piecewise linear。给定某个 activation pattern，局部就是 affine map；block cipher 的 S-box 则是离散非线性，不存在这种连续几何边界。

#### 2. hard-label extraction 的核心障碍

logit setting 可以看输出分数，hard-label setting 只能看：

$$
\arg\max_j f_j(x).
$$

攻击者需要通过决策边界和 activation boundary 的几何关系，反推出内部 ReLU 边界。前序算法依赖收集 intersection points，也就是若干边界相交的局部几何证据。

#### 3. Dead neuron 与 persistent neuron

给定输入分布 \(p_x\)，若某神经元激活概率极低：

$$
\Pr[\text{active}]\le \epsilon,
$$

称为 dead neuron。若非激活概率极低：

$$
\Pr[\text{inactive}]\le \epsilon,
$$

称为 persistent neuron。

如果攻击预算是 \(N\)，而 \(\epsilon<1/N\)，攻击者几乎看不到状态切换。此时不是算法实现不够努力，而是查询分布下目标事件太稀有。

### Main Results

#### 1. 原多项式结论的隐含条件会失效

作者在 MNIST/Fashion-MNIST 上训练 20-layer MLP，观察每层神经元的最小 activation/inactivation probability。结果显示，深层存在概率随深度指数变小的 dead/persistent neurons。

若一个 persistent neuron 的切换概率为 \(\epsilon\)，看到一次切换的查询量量级约为：

$$
O(1/\epsilon).
$$

若

$$
\epsilon \approx e^{-cL},
$$

则查询量随深度 \(L\) 指数增长。

#### 2. 忽略 persistent neuron 会传播误差

dead neuron 被忽略时输出近似为 0，影响有限；persistent neuron 一直 active，它的输出持续流入后续层。若攻击者把它当成 dead neuron，就等于删除了一个非零坐标，后续层的 signature estimation 会产生不可消除误差。

作者从几何上分析这个误差，并在实验中观察到误差随宽度 \(d\) 大约按：

$$
O(1/d)\ \text{到}\ O(1/(d\sqrt d))
$$

下降，但攻击者无法控制 victim model 的宽度，因此不能靠这个修复。

#### 3. Cross-layer extraction

Cross-layer extraction 的想法是：既然 persistent neuron 自己的边界看不到，就从更深层的边界中恢复它造成的线性组合。

若 \(P\) 是 persistent neurons 的索引集合，下一层某神经元边界会包含项：

$$
\sum_{j\in P} w^{(\ell+1)}_{k,j}w^{(\ell)}_j.
$$

作者构造一个 cross-layer vector，把可恢复的非 persistent 坐标和 persistent neuron 的线性组合放到一起，通过多组深层 boundary points 恢复其 span。

重要限制：cross-layer extraction 通常恢复的是 persistent weights 的 span 或线性组合，而不是每个 persistent neuron 的单独参数。

### Experiments

实验包括：

- 训练 20-layer MLP，hidden width 256，数据集 MNIST 和 Fashion-MNIST。
- 统计各层 minimum activation/inactivation probability。
- 收集 \(10^6\) intersection points，观察 switching probability 和 discovered intersection fraction 的关系。
- 在第 9 层设一个 persistent neuron 缺失，尝试恢复第 10 层神经元 signature，分析 \(1-\cos\theta\) 误差。

关键观察：

- 深层神经元确实出现极小切换概率。
- discovered intersection points 与 switching probability 强相关。
- 持久神经元导致的估计误差不会仅靠增加 intersection points 完全消失。

### Insights

#### 1. 多项式攻击常常藏着分布假设

一个算法的步骤数可能是多项式，但它需要的“有用样本”是否能以多项式查询拿到，是另一回事。这里的多项式 claim 卡在 sampling event 的概率上。

#### 2. hard-label 比 logit 少的不只是数值精度

logits 给攻击者一个连续观察面；hard-label 把这个面压成离散区域。攻击者只能通过边界搜索恢复几何结构，因此对边界可达性高度敏感。

#### 3. Cross-layer extraction 是“从影子恢复看不见的物体”

persistent neuron 的自身边界不可见，但它在更深层输出中留下线性组合痕迹。Cross-layer extraction 像从多个投影中恢复一个隐藏方向：能恢复 span，但不一定恢复每个原始向量。

### Critical Reading

#### Strengths

- 精准挑战了 hard-label extraction 的关键隐含假设。
- 把“几乎不切换的神经元”从经验现象提升到攻击复杂度问题。
- 给出 cross-layer extraction 作为建设性补救，而不是只做否定。
- 同时包含理论误差分析和 MNIST/FMNIST 实验。

#### Limitations

- 实验主要是 MLP 和视觉小数据集，距离现代大模型仍远。
- Cross-layer extraction 的完整复杂度、鲁棒性和工程可行性还需要更多验证。
- 攻击仍依赖较强的架构知识和查询能力。
- 论文重点是 ReLU MLP；其他激活函数、归一化层、残差结构可能改变边界几何。

### 用户可能“不知道自己不知道”的背景

#### 1. Activation boundary 不是 decision boundary

ReLU 的 activation boundary 是某个内部神经元从 0 切到正值的超平面；decision boundary 是最终类别 argmax 改变的边界。Hard-label attacker 直接看到的是后者，只能间接推断前者。

#### 2. 查询复杂度受罕见事件支配

若某事件概率是 \(\epsilon\)，期望看到一次需要 \(1/\epsilon\) 次采样。深层神经元的切换概率若指数小，多项式步骤的算法也会因采样失败而退化为指数查询。

#### 3. “恢复等价模型”和“恢复原模型”不同

对于 persistent neurons，cross-layer extraction 可能恢复功能等价的 span，而不是原始参数逐项对应。这对攻击目标很关键：若目标是复制预测行为，等价模型可能足够；若目标是盗取精确参数，则不一定足够。

### 可沉淀到 `03_Knowledge` 的原子概念

- [[Hard-Label Attack]]
- [[Model Extraction]]
- [[Activation Boundary]]
- [[Decision Boundary]]
- [[Persistent Neuron]]
- [[Dead Neuron]]
- [[Cross-Layer Extraction]]
- [[Query Complexity]]

### Sources

- arXiv：https://arxiv.org/abs/2510.06692
- 本地 PDF：`Research on Cryptographic Neurons/Papers/Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial/Is the Hard-Label Cryptanalytic Model Extraction Really Polynomial.pdf`

## 标签

#status/进行中 #type/笔记 #type/论文
