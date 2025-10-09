---
title: "Introduction to training LLMs for AI agents"
author: ["Mingrui Guo"]
date: "2025-10-02"
categories: ["AI"]
tags: ["LLM Agents"]
draft: flase
ShowToc: true
TocOpen: false
math: true
---

大家可能都已经对 LLM 很熟悉了。大概在两三年前，ChatGPT、Claude、Llama、DeepSeek 等模型相继出现，可以说是彻底改变了世界。但在使用这些强大工具的同时,一个核心问题值得探讨：**这些模型到底是如何训练的？**

本文将从宏观视角梳理 LLM 的训练流程，重点关注训练 AI Agents 所需的关键技术路径,而非底层实现细节。

## LLM Training Pipeline

LLM 的训练是一项复杂的系统工程,通常可以划分为三个核心阶段: **预训练(Pre-training)、经典后训练(Classic Post-training/RLHF) 和 推理强化学习(RL for Reasoning)**。在实际应用中,我们还会结合 提示工程(Prompting) 和 微调(Fine-tuning) 来进一步激发模型潜力。

### General LLM Training Pipeline

从整体上看，大语言模型（LLM）的训练分为三个阶段，每个阶段的目标、规模和挑战各不相同：

1. **Pre-training**：在大规模文本上学习预测下一个词，建立通用知识基础；这是规模最大、成本最高的一步，瓶颈在于高质量数据和算力资源。
2. **Classic Post-training / RLHF**：通过人类反馈强化学习，使模型输出更符合用户偏好；相比预训练，所需数据和成本大幅降低，但高度依赖优质反馈和有效评测体系。
3. **RL for Reasoning**：让模型在回答前进行推理，提升解决数学、编程等客观问题的能力；规模和成本介于前两者之间，难点在于设计合适的 RL 环境并防止模型“自我黑客”。

![](../../docs/images/general-llm-training-pipeline.png)

**三个阶段对比**

| 阶段 | 核心目标 | 数据规模 | 训练时间 | 成本级别 | 主要瓶颈 |
| --- | --- | --- | --- | --- | --- | 
| 预训练（Pre-training）| 学习预测下一个词，构建知识底座 | ~10 万亿 tokens | 数月 | 千万美元级 | 高质量数据、算力资源 |
| 经典后训练 / RLHF（Classic Post-training / RLHF）| 让模型符合用户偏好 | ~10 万个问题 | 几天 | 数万–十万美元 | 人类反馈数据、评测体系 |
| 推理型 RL（RL for Reasoning）| 提升推理和思考能力 | 百万级问题 | 数周 | 百万美元级 | RL 环境设计、防止自我黑客 |



### 成功的关键要素

了解了训练的宏观阶段后，我们再深入一层，看看在每个阶段中，哪些要素是决定模型成败的关键。

1. **模型架构 (Architecture)**: 模型的骨架，目前 Transformer 仍是绝对主流，MoE (Mixture of Experts) 因其高效扩展性而备受关注。
2. **训练算法与损失函数 (Algorithm & Loss)**: 模型优化的导航系统，决定了模型如何从数据中学习。
3. **数据与强化学习环境 (Data & RL Environment)**: 模型学习的“教材”。尤其在对齐和能力提升阶段，高质量的数据和精心设计的 RL 环境至关重要。
4. **评测 (Evaluation)**: 衡量训练效果的标尺，指导着整个优化迭代的方向。
5. **系统与基础设施 (Systems & Infrastructure)**: 支撑训练的动力引擎，决定了你能否高效、稳定地将模型规模化。

一个有趣的转变是，在 2023 年之前，学术界和业界的焦点更多集中在**架构和算法**的创新上。而如今，随着技术路线逐渐收敛，共识已经形成：**真正拉开模型效果差距的，是数据、评测和系统这三大支柱**。

LLM 训练已从单纯的算法竞赛，演变为一项复杂的系统工程：

- **数据决定上限**：数据的质量和多样性，直接决定了模型可能达到的最终高度。
- **评测决定方向**：科学的评测体系，是迭代优化、做出正确技术决策的指南针。
- **系统决定规模**：强大的基础设施，决定了你能否训练更大、更强的模型。

### 提示工程与领域微调

当我们获得一个训练好的基础模型后，如何高效地将其应用于特定场景？**提示工程 (Prompting)** 和 **领域微调 (Fine-tuning)** 是两种最核心的技术。

**提示（Prompting）**

我们可以将称之为 **“提问的艺术”**，即通过精心设计的指令 (Prompt) 来引导模型精准地执行任务。其核心特点是**轻量、低成本且由评测驱动**。对于大多数开发者而言，这是将**模型与业务结合的最高效途径**。

**微调（Fine-tuning）**

微调的核心思想是利用特定领域的数据，对基础模型进行“专项强化训练”，使其成为该领域的“专家”。例如，医疗公司可利用专业医疗数据对模型进行微调，显著提升其在医疗报告理解上的准确性。相比提示工程，微调需要额外的**数据和算力投入**，但**能让模型更深入地内化领域知识，效果上限更高**。

![](../../docs/images/llm-specializing-pipeline.png)


## Pretraining 

接下来，我们将深入探讨预训练阶段。这部分将围绕三个核心展开：**方法 (Method)**、**数据 (Data)** 和**算力 (Computation)**。

### Method

预训练的**目标是让模型学习世界知识**，但实现这一目标的任务却很简单：**预测下一个词 (Next Token Prediction)**。

这个过程与手机输入法预测下一个词的原理类似。然而，当模型在超过 10 万亿 Token 的互联网级数据上完成这个任务后，推理、归纳甚至“思考”的能力便会“涌现”出来。这一核心思想自 GPT-2 以来，便成为业界主流。

#### 简单语言模型：N-gram

要理解现代 LLM，可以回顾其最朴素的前身——**N-gram 模型**。这是一种纯粹的统计方法，其核心思想是：**一个词的出现概率，只与它前面的 N-1 个词相关**。

- **预测方式**：通过在语料库中统计 N-gram 片段的出现频率来计算概率。
- **致命缺陷**：
  - 存储灾难：需要存储海量片段，无法扩展到互联网数据。
  - 无法泛化：无法处理未在语料库中见过的文本。

![](https://cdn.botpenguin.com/assets/website/Screenshot_2023_09_20_at_4_36_17_PM_9bace7cd14.webp)

为了解决这些问题，研究者们转向了神经网络。**神经网络语言模型本质上可以看作是 N-gram 模型的一种高效、可泛化的“参数化近似”**。它不再死记硬背所有统计频次，而是学习文本中蕴含的模式，并将这些模式压缩到模型的参数中。

#### 自回归 (Autoregressive) 神经网络模型

现代 LLM 普遍采用**自回归 (AR)** 模式进行训练，其工作流程可分解为以下几步：

1. **文本向量化 (Word Embedding)：**
   - **分词 (Tokenization)**：将文本 "She likely prefers" 切分为 Token 序列 ["She", "likely", "prefers"]。
   - **嵌入 (Embedding)**：将每个 Token 映射为一个高维向量。这组向量成为神经网络的数字输入。

2. **上下文融合 (Neural Network Processing)：**
   这组向量被送入 Transformer 网络。通过自注意力等机制，模型对输入信息进行复杂的加权融合，生成一个蕴含了全部上下文信息的新向量。

3. **概率预测 (Probability Prediction)：**
   - 这个上下文向量通过一个线性层，被转换为一个与词汇表等大的高维向量（称为 logits）。Logits 向量中的每个值代表对应词成为下一个词的可能性得分。
   - 使用 Softmax 函数将 logits 转换为一个总和为 1 的概率分布。例如，"dogs" 的概率为 0.7，"cats" 为 0.2 等。

4. **训练与推理：**
   - **训练时**：模型将预测概率（如 "dogs" 的 0.7）与真实答案（概率 1.0）进行比较，通过**交叉熵损失 (Cross-Entropy Loss)** 计算差距，并据此反向传播更新模型参数。
   - **推理时**：模型根据生成的概率分布进行采样 (Sampling)，通常选择概率最高的词，并将其拼接回输入，循环往复，生成完整的回答。

![](https://lena-voita.github.io/resources/lectures/lang_models/neural/nn_lm_idea_linear-min.png)

### Data 

预训练数据是大模型的"养料"，其规模之大令人咂舌。当前主流模型通常需要消耗 超过 10 万亿 tokens 的数据：

- **Llama 4**：大约在 20–40 万亿 tokens
- **DeepSeek V3**：大约 15 万亿 tokens
- **Llama 3**：大约 15 万亿 tokens

换算一下，这相当于 超过 200 亿个独立网页 的内容量。然而**规模不等于质量**。从充斥着广告和低质信息的互联网中提炼出“精华”，是预训练成败的关键。

**数据处理流程**

1. **数据抓取与文本提取**：利用 `Common Crawl` 等开源项目，从互联网上下载海量的原始网页快照，再从中精准提取正文文本。这是计算成本最高的环节之一。

2. **多层过滤与去重**：
   - **内容过滤**：移除有害、不适宜 (NSFW) 及隐私内容。
   - **启发式过滤**：通过长度、字符等简单规则，快速筛掉明显的低质量文本。
   - **数据去重**：去除高度雷同的内容，避免模型学习无效模式。

3. **基于模型的质量筛选**：训练一个分类器，让它学习**高质量文本（如维基百科）**的特征，然后用它去为海量网页打分，筛选出最“像”高质量内容的文本。

4. **数据混合 (Data Mix)**：将来自网页、代码、书籍、论文等不同来源的干净数据，按特定比例混合。这个**数据配方直接塑造了模型的能力倾向**，例如，**增加代码数据的权重，可以显著提升模型的编程能力**。


**预训练后期的策略**

- **中期训练 (Mid-training)**：在预训练后期，使用一个规模更小但**质量极高的“黄金数据集”（如书籍、论文）进行第二阶段训练**，以巩固和深化核心知识。
- **长上下文持续预训练 (Continual Pre-training for Longer Context)**：为节约成本，通常在训练前期使用较短上下文（如 4k），在最后阶段才逐步增加到目标长度（如 128k）。

由于涉及核心竞争力和潜在的版权风险，**顶级的数据处理流程和数据配方，是各家机构真正的护城河**。


### Computation

在预训练中，**计算量 (Compute)** 投入几乎是决定模型性能的唯一关键因素。这个规律对所有类型的数据和模型都普遍适用。

计算量主要由两个维度决定：
- **模型规模 (Parameters)**：模型包含的参数数量。
- **数据量 (Training Tokens)**：用于训练的文本总量。

幸运的是，模型性能与计算量之间的关系并非杂乱无章，而是遵循着一种可预测的规律——**缩放法则 (Scaling Laws)**。这意味着我们可以**在小规模实验中观察模型表现，然后依据缩放曲线，精准推断出在投入百倍、千倍计算量后模型的性能会达到何种水平**。

![](https://i.imgur.com/7lhHT8n.png)

#### Scaling Laws: Tuning

缩放法则彻底改变了超参数的调优方式，将一个极其昂贵的过程变得高效。

- **传统做法**：直接启动多个不同超参的大模型并行试错，成本高昂。
- **现代做法**：在多个小模型上快速实验，找到超参数与模型规模之间的缩放关系，再将最优配置外推 (Extrapolate) 到目标大模型上。

缩放法则让我们可以从小模型的廉价试错中，找到适用于大模型的规律，极大提升了研发效率。

#### Scaling laws for development

在选择模型架构时，我们真正关心的不应是它在当前算力下的表现，而是当计算资源扩大10倍、100倍后，哪个架构会变得更强(@kaplanScalingLawsNeural2020)。关键在于比较两个指标：

- **初始性能 (Constant)**：在同等算力下，谁的性能更好。
- **缩放速率 (Scaling Rate)**：每增加单位算力，谁的性能提升得更快。

**Example: Transformer vs. LSTM** 
- **问题**: Transformer 架构还是 LSTM 架构？   
- **方法**: 我们可以在较小的规模下，分别训练这两种模型，并绘制它们的性能曲线
- **观察 (见图)**:
![](../../static/images/Screenshot%202025-10-04%20at%2012.49.10 PM.png)
    - **Transformer 的线始终在 LSTM 的线下方**: 在任何计算规模下，Transformer 的表现都更好。这被称为有更好的“常数”。
    - **Transformer 的线的斜率更陡**: 每增加一点计算资源，Transformer 性能的提升幅度比 LSTM 更大。这被称为有更好的“缩放速率”。

**应永远选择缩放速率更优的架构**。 一个架构即使当前表现稍逊，但只要其潜力巨大，就更值得投资。

#### Scaling laws: eg Chinchilla

在计算预算固定的前提下，资源应该更偏向于扩大模型规模还是增加数据量？DeepMind 的 Chinchilla 论文通过缩放法则给出了答案(@hoffmannTrainingComputeOptimalLarge2022)。

Chinchilla 发现：为了最高效地利用算力，模型参数量与数据量之间存在一个“黄金比例”——**每 1 个模型参数，大约需要 20 个 Token 的数据来训练**。

![](../../static/images/Screenshot%202025-10-04%20at%2012.57.17 PM.png )

**局限性：**
- **被忽略的成本**：Chinchilla 法则只优化了训练成本，完全没有考虑模型部署后的**推理成本**。
- **现实权衡**：对于商业服务而言，推理成本是决定成败的关键。因此，更明智的决策可能是：**宁愿在训练阶段投入更多资源（例如，用更多数据训练一个相对较小的模型），以换取未来长期、低廉的推理成本**。

### Summary: 训练一个前沿模型的代价

这里以一个假设的405B参数的 Llama 3 模型为例，估算了其训练所需的各项成本（基于2023-2024年数据）:

- **规模**：405B 参数，15.6T Tokens 数据。
- **硬件**：16,000 张 NVIDIA H100 GPU。
- **时间**：约 70 天。
- **金钱成本**: 综合硬件租用和人力等成本，为 **5200万美元**（范围可能在 5000 万到 8000 万美元之间）。
- **环境成本**: 仅训练这一个模型所产生的碳排放，约等于 **2000 次从纽约到伦敦的往返机票**。
- **未来趋势**: 每一代新模型的训练算力消耗都将大约是上一代的 **10 倍**。

大模型的预训练阶段，本质上是一场**围绕方法、数据和算力的系统性工程**。它以一个极其简单的任务为起点，通过海量高质量数据和庞大计算资源的投入，最终实现了智能的涌现。

## Posttraining

预训练的目标是**预测下一个词**。这使得模型精通语言模式，却不理解“遵循指令”或“帮助用户”这类概念。

例如，如果你对一个原始的预训练模型说：“给6岁的孩子解释一下登月”，它很可能不会回答，而是续写一个相似的问题，比如：“给6岁的孩子解释一下万有引力”。

因此，我们需要一个额外的“后训练”阶段，来校准模型的行为，使其真正变得有用、可控。

### 后训练的两个主要阶段

**1. Alignment / Instruction Following**

这是让模型变得有用的第一步，也是 **2022年 ChatGPT 取得成功的核心**。

- **目标**：让模型学会理解并遵循用户指令，知道什么样的回答是“好”的回答。
- **任务**：主要通过监督微调 (SFT) 和基于人类反馈的强化学习 (RLHF)。
- **数据**：与预训练相比，数据量小得多，大约在 5千到 50万个高质量问答对之间。

这个阶段好比教一个满腹经纶的学者“如何与人有效沟通”，只需少量高质量范例，即可让它学会互动的范式。

**2. Reasoning**

这是在“对齐”之后更高级的阶段，是 **2024年以来的新趋势**（以 o1 等模型为代表）。

- **目标**：不仅要回答得“好”，更要通过深度思考，确保答案的**正确性**，尤其是在数学、编程等有客观标准的任务上。
- **方法**：主要通过带有验证器的强化学习 (RL with Verifiers)。

它引入了**Test-time Compute** 的概念。传统模型性能在训练结束后就已固定；而具备推理能力的模型，可以在**回答问题时投入更多计算资源来“思考”**，从而得到更准确的答案。

### Supervised Finetuning

**SFT (Supervised Fine-tuning)** 是对齐模型的第一步，其核心是**Behavior Cloning**。

它的原理与预训练一致——预测下一个词，但关键区别在于，它不再使用海量的互联网数据，而是在一个**高质量的、我们期望的“问题-答案”数据集**上进行训练，强制模型只学习和模仿“标准答案”的说话方式。

**SFT 可以做什么？**

SFT 可以将一个原始的知识库模型，转变为一个能干的助手，教会它：

1. **Instruction following**：听懂并执行命令。
2. **Desired format or style**：例如，使用表情符号、要点列表，或保持某种特定的语气。
3. **Tool use**：教会模型调用计算器、搜索引擎等外部API来完成复杂任务。
4. **Early reasoning**：通过学习带有推理链的范例，教会模型在回答前先“思考”。

理论上，**任何你能提供“优质输入-输出”配对的任务，都可以通过 SFT 来学习。**

**SFT 的数据从何而来？**

1. **Ask Humans**

最直接的方式。雇佣人类专家为各种问题编写高质量答案。这是 **GPT-3 进化到初代 ChatGPT 的关键一步**，优点是质量最高，缺点是成本高昂且速度缓慢。

2. **Synthetic Data**

利用一个更强大的“教师模型”来自动生成海量问答数据。**斯坦福的 Alpaca 模型(@duboisAlpacaFarmSimulationFramework2024a)**是这一方法的早期成功典范，它开启了使用合成数据来复刻闭源模型能力的浪潮。

3. **Generate & Verify**

当训练最强模型，没有更强的“老师”可以请教时，你需要一个能判断对错的“裁判”。其步骤是：
- 让模型针对一个问题生成多个候选答案。
- 使用**Verifier**——例如代码测试程序、数学检查器——来自动筛选。
- 只保留通过验证的优质答案作为训练数据。

**DeepSeek R1** 正是通过这种“头脑风暴 + 裁判筛选”的模式，训练出了顶级的推理能力


**SFT 需要多少数据？**

SFT 所需的数据量远小于预训练：

- **简单任务 (学习风格)**：约1万条数据通常足够。LIMA 论文(<@zhouLIMALessMore2023>)发现，数千个高质量范例就能教会模型期望的风格。
- **复杂任务 (推理、工具使用)**：需要更多数据，例如 **DeepSeek R1** 使用了约**80万条**样本。

预训练已让模型学到了几乎所有知识。SFT 的作用更像是**激活和规范**，它告诉模型在何种情境下，应该唤醒哪种它已具备的能力，使其行为模式符合特定用户的偏好。

### Reinforcement Learning

SFT 本质是“行为克隆”，但这存在三大缺陷：

1. **受限于范例质量**：模型的上限无法超越提供标准答案的人类或教师模型。但人类**作为“裁判”的能力，远高于作为“创作者”的能力**。
2. **可能教会模型幻觉**：当模型模仿一个它自己无法验证真实性的答案时（例如引用一篇它没读过的文献），它学会的不是事实，而是“编造看起来煞有介事的假信息”的行为模式。
3. **成本高昂**：编写大量完美的“标准答案”费时费钱。

**RL 的核心思想：最大化期望行为**

为了解决上述问题，RL 提出**与其克隆好的行为，不如去强化好的行为**。我们不再提供唯一标准答案，而是让模型自己探索多种答案，然后通过一个 **Reward Signal** 告诉它哪些更好，从而引导它产出更高分的答案。

**Reward Signal 从何而来？**

1. **Rule-based Rewards**: 对于有明确对错标准的任务（如代码、数学），直接用规则来打分。代码能通过单元测试就给高分，反之则给低分。
2. **RLHF - RL from Human Feedback**: 这是成就 **ChatGPT 的关键技术**。让模型生成多个答案，然后让人类选择“更喜欢哪一个”，用这些偏好数据训练一个 **Reward Model**，让这个AI裁判去指导主模型。
3. **LLM as a Judge**: 直接用一个最强的 LLM 来给答案打分，作为奖励信号。

### DeepSeek R1 的强化学习流程

DeepSeek R1 采用了一种精细化的“分而治之”策略，对不同任务“对症下药”，如图所示(@alammarIllustratedDeepSeekR12025)：

- **对于推理等客观任务**：使用基于规则的验证器（如单元测试）作为“硬核裁判”，奖励信号准确可靠。
- **对于写作等主观任务**：使用模拟人类偏好的奖励模型作为“AI裁判”，判断回答的有用性和安全性。

![](https://substackcdn.com/image/fetch/$s_!HRLo!,w_1456,c_limit,f_webp,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0e5f9acf-b4ca-4ec4-9731-4845c8fc5515_902x394.png)

这种结合了**客观规则和主观模型**的混合式强化学习，使得模型在保证逻辑严谨的同时，也提升了在开放性对话中的表现。

### GRPO 算法

GRPO 由 DeepSeek R1 推广(@shaoDeepSeekMathPushingLimits2024)，是目前开源社区最常用的 RL 算法。其流程简单直接：

1. **生成**：针对一个问题，生成多个不同答案。
2. **打分**：用奖励模型或验证器为每个答案打分。
3. **学习**：更新模型参数，鼓励模型多产出高分答案的行为，抑制低分行为。

![](../../static/images/Screenshot%202025-10-04%20at%205.13.14 PM.png)

在实际操作中，通常会加入 **KL散度约束 (KL Divergence)**，它像一根“缰绳”，防止模型为了追求高分而“走火入魔”，确保其生成的内容依然自然、流畅。

### Infra is Key

在大规模 RL 中，**算法本身并非最难，底层的软硬件基础设施才是真正的挑战**，尤其是在需要模型与环境进行多步交互的智能体 (Agent) 任务中，生成样本（采样）的过程极其消耗计算资源。

**Kimi 团队的解决方案**展示了顶级基础设施的形态(@teamKimiK2Open2025)：

1. **智能调度**：暂停耗时过长的采样任务，先用已有数据更新模型，再用新模型继续被暂停的任务，避免流程卡顿。
2. **大规模并行**：当一个任务等待外部API时，GPU会立即切换到其他任务，最大化利用率。
3. **架构优化**：将训练、推理等引擎部署在同一物理节点，最大限度减少通信延迟。

![](../../static/images/Screenshot%202025-10-04%20at%205.16.11 PM.png)

## Evaluation 

评估是机器学习与人工智能中最关键的环节之一。它的重要性体现在三个方面：

1. **量化进展**：帮助识别模型改进方向，衡量性能变化，指导超参数选择。
2. **模型选择**：用于比较不同模型，以确定最适合特定应用场景的方案。
3. **生产可用性判断**：即使模型在评测中表现最佳，也需要通过评估确定其是否达到实际应用要求。

评估主要分为两类：**Closed-ended Evaluation** 和 **Open-ended Evaluation**。

### Closed-ended Evaluation

**核心思想**：将评测问题转化为**有少数几个固定答案的格式（如多选题）**，从而**可以轻松地自动化验证**。

- **典型例子**：MMLU (Massive Multitask Language Understanding) 是一个广泛使用的基准测试，它包含大量类似大学考试的多项选择题，覆盖了从数学到历史的众多学科(@hendrycksMeasuringMassiveMultitask2021)。
- **主要挑战**：
  - **Prompt Sensitivity**: 对同一个问题，不同的提问方式可能会导致模型给出截然不同的答案。
  - **Data Contamination**: 评测数据常出现在公开语料中，模型可能在预训练阶段就已经“见过”了考题，导致评测分数虚高。

### Open-ended Evaluation

对于像 ChatGPT 这样以对话和生成为核心的模型，封闭式评测远远不够。开放式评测旨在评估它们在真实、无固定答案场景下的表现。

**主要挑战**：

- **应用场景多样**：模型需要处理从聊天、编程到内容摘要等各种任务。
- **答案开放性强**：模型的回答通常很长，且没有唯一的“标准答案”，因此无法通过简单的文本匹配来判断对错。

因此，研究者提出了 **Preference Comparison** 的评估思路：

1. **人工评估：ChatBot Arena**

**Chatbot Arena** (@chiangChatbotArenaOpen2024) 采用了双盲人类（double-blind human evaluation）评估方式：**用户在不知道模型身份的情况下，与随机两款聊天机器人交互并投票选择更优者**。

- **优点**： 结果客观、可信度高。
- **缺点**： 成本高昂且速度缓慢，难以进行大规模、高频率的迭代测试。

2. **LLM 自动评估：AlpacaEval**

为了降低人工成本，研究者提出用 **LLM 充当评审员**。AlpacaEval（@duboisLengthControlledAlpacaEvalSimple2025） 是早期代表方法，核心流程如下：

- 针对同一个指令，分别获取**基准模型**和**待评测模型**的回答。
- 将这两个回答提交给一个强大的**裁判LLM**，让它判断哪个更好。
- 通过大量比较，计算出待评测模型的胜率 (Win Rate)。

**优点**
- 与 Chatbot Arena 的结果高度一致（Spearman 相关系数 0.98）。
- 成本极低（约 3 分钟、10 美元以内即可完成全流程）。

**缺点**
- 存在**Spurious Correlation**风险，即评审模型的偏好可能影响评测结果。

## Systems 

当我们讨论如何提升模型性能时，一个共识是“扩展性决定上限 (scaling is what matters)”。这意味着投入更多的计算资源通常能带来更好的结果。但现实是，**我们所有人都受限于计算资源**。

既然计算是瓶颈，为什么不直接购买更多的 GPU 呢？原因有三：

1. **GPU昂贵且稀缺**: 顶级 GPU 不仅价格高昂，而且供应紧张。即使有预算，也未必能买到。
2. **物理限制**: 大规模 GPU 集群的通信开销巨大。GPU 之间的数据传输速度可能成为新的瓶颈，拖慢整体训练速度。
3. **效率问题**: 必须确保每一块 GPU 的计算潜力都被充分压榨，否则只是徒增成本。

因此，与其盲目堆砌硬件，不如**通过系统级的优化来高效地分配和利用现有资源**。

### GPU 的基础概念

要优化训练过程，首先需要了解 GPU 的核心特性：

1. **大规模并行处理**：与 CPU 核心少而强不同，GPU 拥有成千上万个核心。它采用“单指令多数据”（SIMD）模式，在大量线程上对不同数据执行相同的指令，专为**高吞吐量**而生。

![](https://cvw.cac.cornell.edu/gpu-architecture/gpu-characteristics/ArchCPUGPUcores.png)

2. **矩阵乘法优化**：GPU 最初为图形处理涉及，而图形处理的本质是密集的矩阵运算。因此，GPU 内置了专门用于加速矩阵乘法的硬件单元（如 Tensor Cores），这类运算的速度通常是其他浮点操作的 10 倍以上。

![](https://pbs.twimg.com/media/FcEzLBDakAEnVeZ?format=jpg&name=medium)

3. **计算不再是瓶颈**：GPU 的浮点计算能力 (FLOPs) 增速远超其内存带宽和通信速度(@ivanovDataMovementAll2021)。这意味着，如今的瓶颈不再是“算得慢”，而是“喂不饱”。**保持计算单元持续有数据可算是系统优化的核心挑战**。如下图所示，计算性能的提升曲线远比内存带宽陡峭。

![](https://pbs.twimg.com/media/F_5F9haa8AAEv-T?format=jpg&name=large)

4. **内存层次结构**：GPU内部的内存是分层级的。离计算核心越近的内存（如L1 Cache、Shared Memory）速度越快，但容量越小；离得越远（如Global Memory DRAM）容量越大，但速度越慢。高效的算法需要精心设计，以最大化地利用高速缓存，减少对慢速全局内存的访问。

![](https://cvw.cac.cornell.edu/gpu-architecture/gpu-memory/GPUMemLevels.png)

**关键指标：MFU**

模型 FLOP 利用率 (Model Flop Utilization, MFU) 是衡量系统效率的关键指标：

$$\text{MFU} = \frac{\text{GPU 的理论峰值吞吐量 (Theoretical Best)}}{\text{观测到的模型吞吐量 (Observed Throughput)}}$$

MFU 的值反映了你的代码在多大程度上压榨出了 GPU 的理论性能。一个 MFU 为 1 的程序意味着计算单元在任何时候都处于忙碌状态。
在实际应用中，**MFU 能达到 50% 就已经是非常出色的表现**，许多大公司也需要投入大量精力才能将这个数字从 15%-20% 提升到 50%。

### 系统优化技术

了解了 GPU 的特性后，我们可以探讨一些具体的优化方法。

1. **Low-Precision Training**

**核心思想**：使用更少的bits（如`bf16`）来表示数字，从而减少内存占用和加速数据传输。

由于深度学习训练过程（尤其是随机梯度下降）本身充满了噪声，因此大多数运算并不需要32为浮点数（`fp32`）的高精度。将矩阵乘法等主要计算从 `fp32` 切换到 `bf16` 可以带来显著的性能提升。

**For Training: Automatic Mixed Precision (AMP)**

- **Weights**: 以 `fp32` 格式存储主权重，以保持精度。
- **Computation**: 在进行前向和反向传播时，将权重和激活值转换为 `bf16` 进行矩阵运算，以获得速度提升和内存节省。
- **Gradients**: 以 `bf16` 格式存储，进一步节省内存。
- **Update**: 将 `bf` 梯度转换回 `fp32`，用于更新主权重。

2. **Operator Fusion**

**核心思想**：将多个连续的计算操作合并成一个单一的计算内核（Kernel），以减少对全局内存的读写次数。

在PyTorch等框架中，每一行独立的计算（如 `y = x.cos()`）都可能触发一次全局内存读写。  
Operator Fusion 可以将 `y = x.cos()` 和 `z = y.sin()` 这样的多个操作合并，实现**一次读取、多次计算、一次写回**，从而大幅减少内存访问开销。

![](https://quadric.io/wp-content/uploads/2023/09/Figure4-memory-1024x603.png)

`torch.compile` 就是PyTorch中实现 Operator Fusion 的强大工具。

3. **Tiling**

**核心思想**：通过巧妙地组织计算顺序，最大化地复用已加载到高速缓存中的数据。

以矩阵乘法为例，传统的计算方式需要频繁地从全局内存中读取整个行和列。而Tiling则将大矩阵切分成小块（Tile）。计算时，先将几个小块加载到高速的共享内存中，完成所有与这些小块相关的计算后，再加载下一批。  
这样，加载到高速缓存中的每个数据点都被多次使用，**极大地减少了对慢速全局内存的访问次数**。

![](https://nichijou.co/static/8a6ac4a5ad16d10fb7057c2042b31ad6/e1ec8/block.jpg)

**Example: Flash Attention**

FlashAttention 是注意力机制优化算法(@daoFlashAttentionFastMemoryEfficient2022)，它结合了上述技术：

- **Operator Fusion**: 将注意力的多个计算步骤（矩阵乘法、缩放、掩码、Softmax）融合成一个内核。
- **Tiling**: 在计算注意力分数时使用Tiling策略，避免实例化巨大的注意力矩阵。
- **Recomputation**: 在反向传播时，不存储中间结果（如注意力矩阵），而是重新计算它们。因为重计算比从全局内存中读取这些巨大的中间结果要快得多。

![](../../static/images/Screenshot%202025-10-08%20at%2012.50.52 PM.png)

通过这些系统级的优化，FlashAttention 在没有改变任何模型逻辑的情况下，实现了高达 **1.7 倍**的端到端训练加速。

![](../../static/images/Screenshot%202025-10-08%20at%2012.53.42 PM.png)

### 多GPU并行策略

当模型规模大到单个 GPU 无法容纳时，我们就必须采用并行计算策略。

当训练一个 P 参数的模型，通常需要约 **16P GB** 的内存：

- **Model Weights**: 4P GB
- **Gradients**: 4P GB
- **Optimizer States**: 8P GB (Adam需存储均值和方差)

这意味着，训练一个7B参数的模型大约需要 $16 \times 7B \approx 112GB$ 的显存，这远远超出了单张 GPU 的容量。

1. **Data Parallelism**

**核心思想**：将模型和优化器完整地复制到每张 GPU 上，然后将训练数据分片，每张 GPU 处理一部分数据，每一步计算完成后，聚合所有的 GPU 的梯度，然后同步更新各自的模型副本。

- **优点**：简单直接，可以有效利用多张 GPU 加速训练。
- **缺点**：**完全没有节省内存**。如果模型本身放不下一张卡，数据并行也无能为力。

为了解决显存问题，**ZeRO**(@rajbhandariZeROMemoryOptimizations2020)等技术应运而生。它是一种增强的数据并行，通过**将优化器状态、梯度甚至模型参数分片到不同的 GPU 上**，从而极大地降低了单张 GPU 的显存压力。

![](../../static/images/Screenshot%202025-10-08%20at%201.17.14 PM.png)

2. **Model Parallelism**

**核心思想**：将模型本身切分到不同的 GPU 上。这适用于数据并行无法解决的超大模型场景。

- **Pipeline Parallelism**: 按模型的Layer进行分层。例如，GPU 0 负责第 1-10 层，GPU 1 负责 11-20 层，数据像流水线一样依次流过各个 GPU (@huangGPipeEfficientTraining2019)。

![](../../static/images/Screenshot%202025-10-09%20at%207.48.24 AM.png)

- **Tensor Pipeline**: 在单层内部进行切分。例如，将一个巨大的权重矩阵切分成几块，分别放到不同的 GPU 上进行计算，最后将结果聚合(@shoeybiMegatronLMTrainingMultiBillion2020)。

![](../../static/images/Screenshot%202025-10-09%20at%207.50.41 AM.png)

3. **Architectural Sparsity**

**核心思想**： 并非每个输入数据都需要经过模型的所有参数。通过激活模型的一部分参数来处理输入，可以有效降低计算量。

**混合专家模型（Mixture of Experts, MoE）**是该思想的典型代表。**MoE 模型包含多个专家子网络和一个路由器。对于每个输入，路由器会选择性地激活一个或几个专家来处理它** (@fedusReviewSparseExpert2022)。

![](../../static/images/Screenshot%202025-10-09%20at%207.50.41 AM.png)

**优点**：可以在保持计算量（FLOPs）不变的情况下，大幅增加模型总参数量。由于每次只有部分专家被激活，因此非常适合并行化，可以将不同的专家部署在不同的 GPU 上。

## Summary

本次讨论虽然涵盖了许多核心内容，但AI领域的发展日新月异，还有很多重要话题我们未能深入：

- 在技术实现层面，我们没有探讨像**MoE（混合专家模型）和SSM（状态空间模型）这样的前沿架构；也未涉及模型解码策略与推理优化、ChatGPT等产品的UI/工具设计**，以及至关重要的**多模态技术**。

- 整个领域还面临着更宏大和深刻的挑战：**如何有效防止技术滥用、如何突破上下文窗口的限制、如何应对高质量数据枯竭的“数据墙”危机，以及如何解决数据收集的合法性问题**。

这些悬而未决的议题，共同构成了AI下一阶段的发展蓝图和核心挑战。

## Reference

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-alammarIllustratedDeepSeekR12025" class="csl-entry">

Alammar, Jay. 2025. “The Illustrated DeepSeek-R1.”
https://newsletter.languagemodels.co/p/the-illustrated-deepseek-r1.

</div>

<div id="ref-chiangChatbotArenaOpen2024" class="csl-entry">

Chiang, Wei-Lin, Lianmin Zheng, Ying Sheng, Anastasios Nikolas
Angelopoulos, Tianle Li, Dacheng Li, Hao Zhang, et al. 2024. “Chatbot
Arena: An Open Platform for Evaluating LLMs by Human Preference.” arXiv.
<https://doi.org/10.48550/arXiv.2403.04132>.

</div>

<div id="ref-daoFlashAttentionFastMemoryEfficient2022"
class="csl-entry">

Dao, Tri, Daniel Y. Fu, Stefano Ermon, Atri Rudra, and Christopher Ré.
2022. “FlashAttention: Fast and Memory-Efficient Exact Attention with
IO-Awareness.” arXiv. <https://doi.org/10.48550/arXiv.2205.14135>.

</div>

<div id="ref-duboisLengthControlledAlpacaEvalSimple2025"
class="csl-entry">

Dubois, Yann, Balázs Galambosi, Percy Liang, and Tatsunori B. Hashimoto.
2025. “Length-Controlled AlpacaEval: A Simple Way to Debias Automatic
Evaluators.” arXiv. <https://doi.org/10.48550/arXiv.2404.04475>.

</div>

<div id="ref-duboisAlpacaFarmSimulationFramework2024a"
class="csl-entry">

Dubois, Yann, Xuechen Li, Rohan Taori, Tianyi Zhang, Ishaan Gulrajani,
Jimmy Ba, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto.
2024. “AlpacaFarm: A Simulation Framework for Methods That Learn from
Human Feedback.” arXiv. <https://doi.org/10.48550/arXiv.2305.14387>.

</div>

<div id="ref-fedusReviewSparseExpert2022" class="csl-entry">

Fedus, William, Jeff Dean, and Barret Zoph. 2022. “A Review of Sparse
Expert Models in Deep Learning.” arXiv.
<https://doi.org/10.48550/arXiv.2209.01667>.

</div>

<div id="ref-hendrycksMeasuringMassiveMultitask2021" class="csl-entry">

Hendrycks, Dan, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika,
Dawn Song, and Jacob Steinhardt. 2021. “Measuring Massive Multitask
Language Understanding.” arXiv.
<https://doi.org/10.48550/arXiv.2009.03300>.

</div>

<div id="ref-hoffmannTrainingComputeOptimalLarge2022" class="csl-entry">

Hoffmann, Jordan, Sebastian Borgeaud, Arthur Mensch, Elena Buchatskaya,
Trevor Cai, Eliza Rutherford, Diego de Las Casas, et al. 2022. “Training
Compute-Optimal Large Language Models.” arXiv.
<https://doi.org/10.48550/arXiv.2203.15556>.

</div>

<div id="ref-huangGPipeEfficientTraining2019" class="csl-entry">

Huang, Yanping, Youlong Cheng, Ankur Bapna, Orhan Firat, Mia Xu Chen,
Dehao Chen, HyoukJoong Lee, et al. 2019. “GPipe: Efficient Training of
Giant Neural Networks Using Pipeline Parallelism.” arXiv.
<https://doi.org/10.48550/arXiv.1811.06965>.

</div>

<div id="ref-ivanovDataMovementAll2021" class="csl-entry">

Ivanov, Andrei, Nikoli Dryden, Tal Ben-Nun, Shigang Li, and Torsten
Hoefler. 2021. “Data Movement Is All You Need: A Case Study on
Optimizing Transformers.” arXiv.
<https://doi.org/10.48550/arXiv.2007.00072>.

</div>

<div id="ref-kaplanScalingLawsNeural2020" class="csl-entry">

Kaplan, Jared, Sam McCandlish, Tom Henighan, Tom B. Brown, Benjamin
Chess, Rewon Child, Scott Gray, Alec Radford, Jeffrey Wu, and Dario
Amodei. 2020. “Scaling Laws for Neural Language Models.” arXiv.
<https://doi.org/10.48550/arXiv.2001.08361>.

</div>

<div id="ref-rajbhandariZeROMemoryOptimizations2020" class="csl-entry">

Rajbhandari, Samyam, Jeff Rasley, Olatunji Ruwase, and Yuxiong He. 2020.
“ZeRO: Memory Optimizations Toward Training Trillion Parameter Models.”
arXiv. <https://doi.org/10.48550/arXiv.1910.02054>.

</div>

<div id="ref-shaoDeepSeekMathPushingLimits2024" class="csl-entry">

Shao, Zhihong, Peiyi Wang, Qihao Zhu, Runxin Xu, Junxiao Song, Xiao Bi,
Haowei Zhang, et al. 2024. “DeepSeekMath: Pushing the Limits of
Mathematical Reasoning in Open Language Models.” arXiv.
<https://doi.org/10.48550/arXiv.2402.03300>.

</div>

<div id="ref-shoeybiMegatronLMTrainingMultiBillion2020"
class="csl-entry">

Shoeybi, Mohammad, Mostofa Patwary, Raul Puri, Patrick LeGresley, Jared
Casper, and Bryan Catanzaro. 2020. “Megatron-LM: Training Multi-Billion
Parameter Language Models Using Model Parallelism.” arXiv.
<https://doi.org/10.48550/arXiv.1909.08053>.

</div>

<div id="ref-teamKimiK2Open2025" class="csl-entry">

Team, Kimi, Yifan Bai, Yiping Bao, Guanduo Chen, Jiahao Chen, Ningxin
Chen, Ruijue Chen, et al. 2025. “Kimi K2: Open Agentic Intelligence.”
arXiv. <https://doi.org/10.48550/arXiv.2507.20534>.

</div>

<div id="ref-zhouLIMALessMore2023" class="csl-entry">

Zhou, Chunting, Pengfei Liu, Puxin Xu, Srini Iyer, Jiao Sun, Yuning Mao,
Xuezhe Ma, et al. 2023. “LIMA: Less Is More for Alignment.” arXiv.
<https://doi.org/10.48550/arXiv.2305.11206>.

</div>

</div>
