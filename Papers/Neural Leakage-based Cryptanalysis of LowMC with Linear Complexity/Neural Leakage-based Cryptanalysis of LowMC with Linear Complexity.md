---
title: "Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity"
authors:
  - "Kwangjo Kim"
publication:
  - "Manuscript / local PDF"
tags:
  - paper-note
  - cryptographic-neurons
  - lowmc
  - neural-leakage
  - activation-boundary
  - mpc-in-the-head
  - picnic
  - key-recovery
  - type/论文
description: "分析一篇将神经网络 piecewise-linear activation-boundary leakage 引入 LowMC/PICNIC 密钥恢复的论文；核心是用扰动 probing 与多数投票恢复 round key，再利用线性 key schedule 恢复 master key。"
source: "local Zotero PDF; public web search returned no indexed primary page on 2026-06-23"
local_pdf: "./Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity.pdf"
zotero_collection: "Research on Cryptographic Neurons"
zotero_title_note: "Zotero 条目题名曾显示为 Improved Cryptanalysis of UOV and Rainbow，但 PDF 正文真实标题为 Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity。"
status: "初读"
created: "2026-06-23"
---

# Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity

本地 PDF：[Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity.pdf](./Neural%20Leakage-based%20Cryptanalysis%20of%20LowMC%20with%20Linear%20Complexity.pdf)

上位地图：[[MOC - 计算机]] · [[Research on Cryptographic Neurons]]

相关主题：[[LowMC]]、[[PICNIC]]、[[MPC-in-the-Head]]、[[Neural Leakage]]、[[Activation Boundary]]、[[Key Recovery]]、[[Majority Voting]]

> 元数据注意：Zotero 条目标题与 PDF 正文不一致。该笔记按用户确认与 PDF 正文真实标题处理，不按 `Improved Cryptanalysis of UOV and Rainbow` 处理。

### Abstract

这篇论文的核心问题是：如果一个密码原语或其相关执行过程被神经网络式、piecewise-linear 的表示近似，那么 ReLU activation boundary 是否会泄露传统布尔电路模型中看不到的信息？

作者把这个问题放到 LowMC 与 MPC-in-the-Head 签名语境下讨论。LowMC 是低乘法复杂度 block cipher，被 PICNIC 等基于对称原语的后量子签名方案使用。论文声称，通过 perturbation-based probing 建模 neural leakage，可以把 round-key recovery 化为每个 bit 独立的二元假设检验，再用 majority voting 恢复第一轮 round key；由于 LowMC 的 key schedule 是线性的，恢复第一轮 round key 后可以通过线性代数恢复 master key。

一句话概括：

> 这篇论文不是传统意义上只看 plaintext-ciphertext pair 的 LowMC 黑盒攻击，而是研究一种“神经/近似实现泄漏模型”下的密钥恢复路径。

### Knowledge

#### 1. LowMC 与 PICNIC 的关系

PICNIC 使用 MPC-in-the-Head 证明知识：

$$
C=\mathrm{LowMC}_k(P),
$$

其中公开密钥含有明密文对 \((P,C)\)，秘密是 key \(k\)。MPC-in-the-Head 把“知道 \(k\)”转成一个零知识证明，再通过 Fiat-Shamir 变换得到签名。

LowMC 的轮密钥由公开矩阵线性生成：

$$
rk_i=K_i\cdot k,\qquad K_i\in \mathbb{F}_2^{n\times n}.
$$

每轮可概括为：

$$
x^{(i)}=L_iS(x^{(i-1)})\oplus rk_i\oplus c_i.
$$

这里 \(S\) 是非线性 S-box 层，\(L_i\) 是线性层，\(c_i\) 是轮常量。

#### 2. Neural leakage 的直观图像

传统密码分析通常假设实现语义是精确布尔电路。神经网络近似或 piecewise-linear realization 则可能引入几何边界：当输入扰动跨过 ReLU 或阈值边界时，输出模式发生可探测变化。

这类似在一堵墙后听机器运转：密码算法理论上只允许看到最终输出，但近似实现的“接缝”可能把内部状态的节奏泄露出来。

#### 3. Round-key bit recovery 变成二元假设检验

论文把每个 round-key bit 看成一个二元假设：

$$
H_1: rk_1[i]=1 \Rightarrow \mathbb{E}[y]=+\mu,
$$

$$
H_0: rk_1[i]=0 \Rightarrow \mathbb{E}[y]=-\mu.
$$

给定多次 noisy oracle 输出 \(y_t\)，用多数投票估计：

$$
\hat b_i=
\begin{cases}
1, & \sum_{t=1}^{T}y_t\ge 0,\\
0, & \text{otherwise}.
\end{cases}
$$

这可以看成 sign test / Neyman-Pearson detector 的简化版本。每个 bit 独立估计，复杂度因此随 key length 线性增长。

### Method

#### 1. 攻击流程

论文给出的高层流程是：

1. 生成或获得 LowMC key schedule 矩阵 \(K_1\)。
2. 通过 neural leakage oracle 对每个 bit 做多次 probing。
3. 对每个 bit 用 majority voting 估计 \(\widehat{rk}_1[i]\)。
4. 得到 \(\widehat{rk}_1\) 后，若 \(K_1\) 可逆，则求：

$$
\hat{k}=K_1^{-1}\widehat{rk}_1.
$$

5. 用 Hamming distance、match rate 或 confidence 评估恢复质量。

#### 2. 线性复杂度来自哪里

线性复杂度不是凭空出现的，而来自两个前提：

- 每个 bit 的 leakage test 可以独立执行；
- key schedule 是线性的，且相关矩阵在实验设置中可逆或满秩。

若 key schedule rank-deficient，攻击者得到的是解空间而不是唯一 key。也就是说，论文的“linear complexity”是条件性结论，不是对任意 LowMC/PICNIC 实现的无条件破坏。

### Experiments

PDF 正文声称，在 Grain-SSG 生成 full-rank key schedule 的条件下，可以从恢复的 \(rk_1\) 进一步恢复 LowMC-128、LowMC-192 和 LowMC-256 的 master key，并报告 100% recovery。

这个实验结果的核心证据链是：

$$
\text{activation-boundary leakage}
\rightarrow
\text{round-key bit hypothesis tests}
\rightarrow
\widehat{rk}_1
\rightarrow
K_1^{-1}\widehat{rk}_1
\rightarrow
\hat{k}.
$$

### Insights

#### 1. 这是一种实现语义攻击，而不是纯算法攻击

如果 LowMC 以理想布尔电路运行，论文的 leakage oracle 并不存在。攻击成立的前提是存在某种 neural/hybrid/approximate implementation，使 activation boundary 可以被 probing。读这篇论文时不能把它误解为“LowMC 标准算法被线性时间破坏”。

#### 2. 线性 key schedule 是把局部泄漏放大的桥

round key 与 master key 的关系是线性映射：

$$
rk_1=K_1k.
$$

因此一旦 \(rk_1\) 被恢复，攻击不需要再解非线性密码结构，而是解一个线性系统。这里的结构很像电路中的“放大器”：真正泄漏的是局部 round key bit，但线性 key schedule 把局部信息连接回全局 secret。

#### 3. Majority voting 是统计决策，不是神秘神经推理

论文表面上用了 neural leakage，但核心恢复器非常朴素：重复观测、估计符号、投票判 bit。这个拆解很重要，因为它让读者能区分“泄漏模型的强假设”和“恢复算法的简单性”。

### Critical Reading

#### Strengths

- 把神经近似实现和密码实现安全联系起来，提出了一个值得关注的攻击面。
- 将 round-key recovery 归约到可解释的 binary hypothesis testing。
- 明确利用 LowMC key schedule 的线性结构，证据链短而清楚。

#### Limitations

- 论文依赖 neural leakage oracle；这不是标准黑盒 LowMC 安全模型。
- PDF 未在公开搜索中找到稳定索引页面，元数据和 Zotero 条目存在错配，后续需要核验版本来源。
- 4 页正文较短，实验细节、噪声模型、oracle 构造、参数选择和复现实验描述不足。
- “100% recovery”需要结合泄漏幅度 \(\mu\)、试验次数 \(T\)、噪声分布和 key schedule rank 才能判断强度。

### 用户可能“不知道自己不知道”的背景

#### 1. 黑盒攻击、侧信道攻击、实现语义攻击不是一回事

- **黑盒密码分析**：攻击者只看到规定输入输出。
- **侧信道攻击**：攻击者还看到时间、功耗、电磁等物理泄漏。
- **神经/近似实现泄漏**：攻击者利用实现函数的几何边界或近似误差。

这篇论文更接近第三类，不应直接与传统 LowMC algebraic attack 横向比较。

#### 2. Majority voting 的错误率取决于信噪比

若单次观测正确概率为 \(p>1/2\)，多数投票错误率随 \(T\) 增大下降；若 \(p\) 接近 \(1/2\)，需要的样本数会急剧增加。可粗略理解为 Hoeffding 型界：

$$
\Pr[\text{vote error}] \le \exp(-2T(p-1/2)^2).
$$

因此攻击复杂度中的“线性”通常还隐藏了一个关于信噪比的样本因子。

#### 3. Rank matters

若

$$
rk_1=K_1k
$$

中的 \(K_1\) 不可逆，恢复 \(rk_1\) 不等于唯一恢复 \(k\)。攻击者得到的是 affine solution space。这个细节决定了实验设置和真实参数之间能否直接迁移。

### 可沉淀到 `03_Knowledge` 的原子概念

- [[LowMC]]
- [[PICNIC]]
- [[MPC-in-the-Head]]
- [[Neural Leakage]]
- [[Activation Boundary]]
- [[Majority Voting]]
- [[Key Schedule]]
- [[Neyman-Pearson Detector]]

### Sources

- 本地 PDF：`Research on Cryptographic Neurons/Papers/Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity/Neural Leakage-based Cryptanalysis of LowMC with Linear Complexity.pdf`
- Web search check：2026-06-23 使用在线搜索查询真实标题，未发现稳定公开索引页面。

## 标签

#status/进行中 #type/笔记 #type/论文
