---
title: "Do Efficient Transformers Really Save Computation?"
authors:
  - "Kai Yang"
  - "Jan Ackermann"
  - "Zhenyu He"
  - "Guhao Feng"
  - "Bohang Zhang"
  - "Yunzhen Feng"
  - "Qiwei Ye"
  - "Di He"
  - "Liwei Wang"
publication:
  - "ICML 2024"
  - "arXiv:2402.13934v2, 2024-11-09"
tags:
  - paper-note
  - cryptographic-neurons
  - efficient-transformer
  - sparse-transformer
  - linear-transformer
  - chain-of-thought
  - dynamic-programming
  - computational-complexity
  - type/论文
description: "从 DP/CoT 推理任务角度分析 Sparse Transformer 与 Linear Transformer 是否真的节省计算；结论是通用 DP 下隐藏维度随问题规模增长，效率优势可能消失，但 locality 强的任务仍可能受益。"
source: "https://arxiv.org/abs/2402.13934"
local_pdf: "./Do Efficient Transformers Really Save Computation.pdf"
zotero_collection: "Research on Cryptographic Neurons"
status: "初读"
created: "2026-06-23"
---

# Do Efficient Transformers Really Save Computation?

原论文链接：[arXiv:2402.13934](https://arxiv.org/abs/2402.13934)

本地 PDF：[Do Efficient Transformers Really Save Computation.pdf](./Do%20Efficient%20Transformers%20Really%20Save%20Computation.pdf)

上位地图：[[MOC - 计算机]] · [[Research on Cryptographic Neurons]]

相关主题：[[Transformer]]、[[Sparse Transformer]]、[[Linear Transformer]]、[[Chain-of-Thought]]、[[Dynamic Programming]]、[[注意力复杂度]]

### Abstract

这篇论文问了一个很实际的问题：把标准 self-attention 换成 sparse attention 或 linear attention 后，模型是否真的省了计算？

直觉上，标准 Transformer 的 attention 对序列长度 \(L\) 是二次复杂度，efficient Transformer 把 attention 做稀疏化或核化，似乎应该更便宜。但作者指出，在推理任务里不能只看 attention pattern 的单步复杂度，还要看模型为了保持表达能力是否需要更大的 hidden dimension。

论文把 Chain-of-Thought 推理形式化为 dynamic programming（DP）过程。核心结论是：

- 对一般 DP 推理任务，Sparse Transformer 和 Linear Transformer 虽然有表达能力，但 hidden dimension 必须随问题规模增长，导致总复杂度接近标准 Transformer。
- 对具有 locality 的任务，例如 arithmetic evaluation 或 edit distance 的局部变体，efficient Transformer 仍可能真正节省计算。

一句话概括：

> Efficient attention 省下来的计算，可能会被更大的模型宽度重新吃掉；只有任务依赖结构足够局部时，省计算才比较稳。

### Knowledge

#### 1. attention 复杂度不能单独看

标准自注意力的序列复杂度是：

$$
O(L^2).
$$

Sparse Transformer 通过只看部分历史 token 降低 attention 连接数；Linear Transformer 通过核技巧把注意力写成可累积形式，使 attention 计算近似线性。

但总计算不是 attention mask 一项，而是：

$$
\text{total cost}\approx \text{attention cost}(L,D)+\text{FFN cost}(L,D),
$$

其中 \(D\) 是 hidden dimension。如果为了补偿信息瓶颈必须把 \(D\) 放大，理论节省可能消失。

#### 2. DP 是 CoT 推理的抽象模型

作者沿用一个 DP 形式化：推理过程产生一串中间状态，每个状态依赖若干历史状态：

$$
dp(i)=f(i,dp(h_1(i)),\ldots,dp(h_K(i)),s_{g_1(i)},\ldots).
$$

这很像 CoT：模型不是一步给答案，而是逐步写出子问题结果。

#### 3. log-precision 是关键假设

论文假设 Transformer 内部数值只有 \(O(\log L)\) bit 精度。这让每个 neuron 的信息容量有限。若模型的可访问 token 太少或压缩状态太小，就会出现信息瓶颈。

直观类比：一个工厂把传送带变窄了，如果仍要传同样多的零件，就必须增加箱子大小；箱子大小就是 hidden dimension。

### Theoretical Results

#### 1. 通用 DP：表达能力存在，但效率优势可能消失

论文先给出正结果：对符合条件的 DP 问题，Sparse/Linear Transformer 可以构造出能求解的模型，但需要

$$
D=O(\sqrt{L}).
$$

随后给出下界：对 regular DP 问题，如果输出长度 \(L=\Theta(n)\)，则 Sparse Transformer 与 Linear Transformer 需要

$$
D=\widetilde{\Omega}(\sqrt{L}).
$$

由于 hidden dimension 增大，整体复杂度回到接近：

$$
\widetilde{\Theta}(L^2).
$$

这就是论文标题中的悖论：efficient attention 本身更便宜，但为了保存足够信息，模型宽度必须增加。

#### 2. locality 让 efficient Transformer 真正有机会

如果 DP 满足 \(m\)-locality，即每个状态只依赖最近 \(m\) 个状态：

$$
dp(i)=f(dp(i-1),\ldots,dp(i-m),\ldots),
$$

Sparse Transformer 可以用 block size \(B=\Theta(m)\) 和常数 hidden dimension 求解，复杂度约为：

$$
\widetilde{O}(mL).
$$

当 \(m\ll L\) 时，这才真正小于 \(L^2\)。

### Experiments

论文在三个 DP/CoT 任务上验证理论：

- **Arithmetic**：算术表达式求值，局部性强。
- **Edit Distance (ED)**：编辑距离，有一定局部结构。
- **Longest Increasing Subsequence (LIS)**：最长递增子序列，局部性弱，更接近一般 DP。

实验观察：

- 标准 Transformer 在固定 embedding dimension 下可跨多个问题规模保持较好表现。
- Sparse/Linear Transformer 通常需要随着问题规模增大而增加 embedding dimension。
- locality 越弱，efficient Transformer 对 dimension 的需求越明显。
- Arithmetic 任务中，标准 Transformer 在达到 90% accuracy 的最小 GFLOPs 上仍低于 sparse/linear 变体；这提示理论上的 attention 节省不必然转成端到端节省。

### Insights

#### 1. “结构稀疏”必须匹配“依赖稀疏”

Sparse attention 最适合依赖本来就局部的任务。如果任务的真实依赖是全局的，硬把 attention 稀疏化，只是把全局信息压进更宽的 hidden state。

这像把高速公路缩成小路：如果城市交通本来是短距离通勤，小路足够；如果每天都要跨城运输，最终只能用更大的卡车补偿。

#### 2. 计算节省有守恒味道

论文的隐含 insight 是：推理任务需要传递的信息量不会因为换 attention 形式自动消失。它要么通过更多 token 连接传递，要么通过更宽 hidden states 存储。

#### 3. 对长上下文模型设计的启发

“efficient Transformer 是否好”不能脱离任务依赖图来问。更具体的问题应该是：

- 任务依赖是否局部？
- 是否存在可压缩的状态摘要？
- 模型能否在固定宽度下保留未来步骤需要的信息？

### Critical Reading

#### Strengths

- 把 efficient Transformer 的讨论从经验 benchmark 推到理论机制。
- 使用 DP/CoT 框架连接了推理任务和模型复杂度。
- 同时给上界和下界，避免只做存在性证明。
- 实验覆盖 arithmetic、ED、LIS，能体现 locality 差异。

#### Limitations

- 分析集中在 Sparse Transformer 和 Linear Transformer，不能直接外推到所有高效序列模型。
- log-precision 与固定层数/头数是假设，和真实大模型工程系统仍有距离。
- DP/CoT 是重要抽象，但不能覆盖所有自然语言推理。
- Linear Transformer 在 locality 场景下的 tight bound 仍有开放问题。

### 用户可能“不知道自己不知道”的背景

#### 1. 渐近复杂度和实际 wall-clock 不是一回事

\(O(L)\)、\(O(L\sqrt L)\)、\(O(L^2)\) 是理论趋势，不等于 GPU 上实际运行时间。kernel fusion、memory bandwidth、batch size、cache locality 都会改变实际速度。

#### 2. Hidden dimension 是信息通道宽度

在有限精度下，一个 hidden vector 能携带的信息大约随 \(D\log L\) 增长。若模型只能访问少量历史 token，却要决定后面全部 DP 状态，\(D\) 就必须变大。

#### 3. Locality 不是“输入相邻”，而是“依赖相邻”

一个任务输入看起来是长序列，不代表它有长程依赖；反之，一个局部窗口里的 token 也可能编码全局状态。论文里的 locality 指 DP 依赖图上的局部性。

### 可沉淀到 `03_Knowledge` 的原子概念

- [[Sparse Transformer]]
- [[Linear Transformer]]
- [[Dynamic Programming]]
- [[Chain-of-Thought]]
- [[Locality Assumption]]
- [[Log-Precision Transformer]]
- [[信息瓶颈]]

### Sources

- arXiv：https://arxiv.org/abs/2402.13934
- 本地 PDF：`Research on Cryptographic Neurons/Papers/Do Efficient Transformers Really Save Computation/Do Efficient Transformers Really Save Computation.pdf`

## 标签

#status/进行中 #type/笔记 #type/论文
