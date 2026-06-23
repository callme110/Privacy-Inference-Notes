---
title: "ALICE: An Interpretable Neural Architecture for Generalization in Substitution Ciphers"
authors:
  - "Jeff Shen"
  - "Lindsay M. Smith"
publication:
  - "arXiv:2509.07282v2"
  - "Preprint, last revised 2025-09-25"
tags:
  - paper-note
  - cryptographic-neurons
  - transformer
  - substitution-cipher
  - cryptogram-decipherment
  - permutation-learning
  - interpretability
  - generalization
  - type/论文
description: "以替换密码解密为神经网络组合泛化与可解释性测试床，解释 ALICE 如何用 encoder-only Transformer、symbol-wise pooling 和 Gumbel-Sinkhorn bijective head 学习隐藏置换。"
source: "https://arxiv.org/abs/2509.07282"
project: "https://jshen.net/alice/"
code: "https://github.com/al-jshen/alice"
local_pdf: "./ALICE - An Interpretable Neural Architecture for Generalization in Substitution Ciphers.pdf"
zotero_collection: "Research on Cryptographic Neurons"
status: "初读"
created: "2026-06-23"
---

# ALICE: An Interpretable Neural Architecture for Generalization in Substitution Ciphers

原论文链接：[arXiv:2509.07282](https://arxiv.org/abs/2509.07282)

本地 PDF：[ALICE - An Interpretable Neural Architecture for Generalization in Substitution Ciphers.pdf](./ALICE%20-%20An%20Interpretable%20Neural%20Architecture%20for%20Generalization%20in%20Substitution%20Ciphers.pdf)

上位地图：[[MOC - 计算机]] · [[Research on Cryptographic Neurons]]

相关主题：[[Transformer]]、[[替换密码]]、[[置换学习]]、[[Gumbel-Sinkhorn]]、[[组合泛化]]、[[可解释性]]、[[符号错误率]]

### Abstract

这篇论文把古典 cryptogram solving 变成一个神经网络研究问题。任务不是让模型“懂加密协议”，而是让模型在不知道替换表的情况下，从一段 monoalphabetic substitution cipher 的密文中恢复明文。表面上是娱乐报纸里的字谜，深层上是一个隐藏置换的反演问题。

论文提出 **ALICE**，一个 encoder-only Transformer。和 decoder-only 模型逐字符生成不同，ALICE 一次读完整段密文，一次输出整段明文。它的两个关键结构是：

- **symbol-wise token pooling**：同一个密文字母在同一条输入中必须对应同一个明文字母，因此模型在输出前把相同 symbol 的 hidden states 聚合。
- **bijective decoding head**：用 Gumbel-Sinkhorn 学习近似 permutation matrix，使输出天然接近一对一替换表。

论文最有价值的结论不是“Transformer 会解替换密码”，而是：只要任务族有共享结构，模型不必枚举 \(26!\) 个 cipher，就可能学习一种可迁移的反演策略。作者报告 ALICE 在约 1500 个训练 cipher 后可以泛化到未见 cipher，而这个训练覆盖只占全部 cipher 空间的

$$
\frac{1500}{26!} \approx 3.7 \times 10^{-24}.
$$

### Knowledge

#### 1. 替换密码是隐藏置换的反演

设字母表为 \(\Sigma\)，大小为 \(V\)。单表替换密码使用一个双射：

$$
f:\Sigma \rightarrow \Sigma,\qquad f \in S_V.
$$

若明文为

$$
x=(x_1,\ldots,x_L),
$$

密文为

$$
c_i=f(x_i),
$$

则解密目标是估计

$$
\hat{x}_i \approx f^{-1}(c_i).
$$

这个问题像一张地图的地名被整体替换：路网结构没有变，名字全换了。攻击者不是逐个猜字母，而是从共现、词形、频率和句法痕迹中恢复那张隐藏映射表。

#### 2. Bijective constraint 是任务本体

替换密码的关键约束是双射。若两个不同密文字母都被解成同一个明文字母，输出即使局部像英语，也已经违反了密码结构：

$$
f(a)=f(b)\Rightarrow a=b.
$$

普通 token-wise softmax 像让每个格子独立填答案，容易产生 many-to-one。bijective head 像数独规则：每行每列只能占一次。这个差别不是装饰，而是把任务有效输出空间从任意分类压到 permutation polytope 附近。

#### 3. Gumbel-Sinkhorn 是可微的软排列

硬 permutation matrix 不可微。ALICE 训练时用 Sinkhorn normalization 先得到 doubly stochastic matrix：

$$
S^0(X)=\exp(X),
$$

$$
S^\ell(X)=T_c(T_r(S^{\ell-1}(X))),
$$

其中 \(T_r,T_c\) 分别归一化行和列。doubly stochastic matrix 的每行、每列和为 1，可以看成“尚未拍板的软置换”。

推理时再求硬分配：

$$
P^\*=\arg\max_{P\in P_N}\langle X,P\rangle_F.
$$

训练阶段保持可微，推理阶段恢复严格一对一，这是 Gumbel-Sinkhorn 在结构预测任务中的核心价值。

### Method

#### 1. ALICE-BASE

ALICE-BASE 使用 encoder-only Transformer 主干，包含 full attention、RMSNorm、SwiGLU、RoPE 和 symbol-wise pooling。它不做自回归搜索，因此解密速度主要是一次前向传播成本，而不是逐字符 beam search。

#### 2. ALICE-BIJECTIVE

ALICE-BIJECTIVE 在主干后加入可解释的 permutation head。这个 head 不只是提升输出合法性，还让模型内部显式产生一个接近 cipher key 的对象。相比 attention map，这种解释变量更接近任务语义：attention 只是信息路由痕迹，permutation matrix 才是替换密码里的核心隐藏结构。

### Experiments

论文主要在 QUOTES500K 上实验。训练时每条 quote 动态采样 substitution cipher，测试时评估未见 cipher。评价指标是 Symbol Error Rate：

$$
\mathrm{SER}=\frac{\#\text{incorrect characters}}{\#\text{evaluated characters}}.
$$

关键结果：

- 短序列 \((<128)\) 上 ALICE-BASE 的 SER 约为 \(1.09\%\)。
- 长序列 \((>128)\) 上 SER 约为 \(0.06\%\)。
- 训练 cipher 数量从 1000 到 1500 之间出现明显泛化拐点。
- early exit 和 probing 显示，模型早层更像在使用 letter frequency，后层逐步形成 word-level structure。

### Insights

#### 1. 组合空间巨大不等于样本复杂度巨大

\(26!\) 很大，但所有替换表共享同一种结构：它们都是字母表上的置换。模型不是学习 \(26!\) 张表，而是学习“如何从语言统计中恢复置换”。这提醒读者区分 hypothesis space size、task family structure 和 sample complexity。

#### 2. 可解释性最好落在任务变量上

可解释性不只是画热力图。ALICE 的更好做法是让模型内部显式存在一个可解释对象：cipher mapping。对于隐私推断、实体链接、图匹配等任务，类似思路也成立：如果真实问题有隐藏结构，就尽量让模型输出或中间变量对齐这个结构。

#### 3. Cryptogram 是神经推理的半透明测试床

它比 parity、automata 更接近自然语言，又比开放式 NLP 更容易形式化。它有明确输入、输出、隐藏变量和硬约束，因此适合研究神经网络到底是在记忆样本，还是学到了可迁移算法。

### Critical Reading

#### Strengths

- 任务形式化清楚，隐藏变量 \(f\)、输出 \(\hat{x}\)、约束 bijection 都明确。
- 结构设计和任务约束贴合，尤其是 symbol-wise pooling 与 bijective head。
- 不只报告最终准确率，还分析 early exit、probe 和可解释映射。
- 泛化实验区分了训练文本数量和训练 cipher 多样性。

#### Limitations

- 这是 preprint，结论仍需要更多复现与同行评审。
- 任务限定在 monoalphabetic substitution cipher，不能外推到现代密码协议安全性。
- “未见 cipher 泛化”仍在同一语言分布、同一 cipher family 和同一 alphabet 条件下。
- bijective head 改善可解释性，但短序列 SER 略差于 ALICE-BASE，说明硬结构约束可能增加优化难度。

### 用户可能“不知道自己不知道”的背景

#### 1. Unicity distance

短密文不是模型不够聪明，而可能是信息论上不够确定。古典密码分析中的 unicity distance 可粗略写成：

$$
n_0 \approx \frac{H(K)}{D},
$$

其中 \(H(K)\) 是密钥熵，\(D\) 是语言冗余度。密文太短时，多组 key/plaintext 都可能合理。

#### 2. Doubly stochastic matrix 与 permutation matrix

Birkhoff-von Neumann theorem 说明 doubly stochastic matrices 的极点是 permutation matrices。Sinkhorn 的作用可以理解为把模型输出推到“排列多面体”附近，再在推理时选一个硬极点。

#### 3. 和隐私推断的桥接

Cryptogram 与 privacy inference 都可写成：

$$
\text{observable traces}\rightarrow \text{latent structure}.
$$

区别在于，cryptogram 的 latent structure 是合法解密目标；privacy inference 中的 latent structure 往往是攻击者不应获得的信息。

### 可沉淀到 `03_Knowledge` 的原子概念

- [[替换密码]]
- [[Permutation Matrix]]
- [[Gumbel-Sinkhorn]]
- [[Doubly Stochastic Matrix]]
- [[Symbol Error Rate]]
- [[Unicity Distance]]
- [[组合泛化]]
- [[结构化输出约束]]

### Sources

- arXiv：https://arxiv.org/abs/2509.07282
- Project page：https://jshen.net/alice/
- GitHub：https://github.com/al-jshen/alice
- 本地 PDF：`Research on Cryptographic Neurons/Papers/ALICE - An Interpretable Neural Architecture for Generalization in Substitution Ciphers/ALICE - An Interpretable Neural Architecture for Generalization in Substitution Ciphers.pdf`

## 标签

#status/进行中 #type/笔记 #type/论文
