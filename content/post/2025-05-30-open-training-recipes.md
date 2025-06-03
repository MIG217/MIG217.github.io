---
title: " Open Training Recipes for Reasoning in Language Models"
author: ["Mingrui Guo"]
date: "2025-05-30"
categories: ["AI"]
tags: ["LLM", "Post Training", "Pre-training", "EN"]
draft: true
ShowToc: true
TocOpen: false
---

In today's rapidly evolving AI landscape, the remarkable progress we've witnessed is largely attributed to open scientific research and fully open models. However, as time progresses, more and more research and development work is becoming increasingly closed off.

We still need to delve deeper into how language models work, improve their capabilities, and make them safer, more efficient, and more reliable. Simultaneously, we need to extend language models' abilities beyond text into domains like healthcare, science, and even complex decision-making processes. Most importantly, we must bring these models into real-world applications, ensuring they are **deployable, interpretable, and effectively mitigate biases and risks**.

To achieve these goals, we need:

- **Fully open language models** (including data, code, and training details)
- **Transparent research processes** (facilitating review and understanding)
- **Reproducible results** (driving robust scientific progress)
- **Accessible ecosystems** (supporting broader researcher participation)

At the University of Washington (UW) and AI2, this commitment to openness is foundational. Through two major initiatives—**OLMo** (Open Language Models) and **Tulu** (an open post-training framework)—they are building a fully open language model ecosystem that spans pretraining, post-training, intermediate training stages, and agent development.

In this post, I’ll introduce three of their key efforts to improve reasoning in language models, each representing a distinct but complementary stage of the development pipeline:

- **Pretraining** (e.g., OLMo, Dolma)
- **Post-training** (e.g., Tulu, OpenInstruct)
- **Test-time inference** (e.g., S1, Self-RAG, OpenScholar)

## Overview

### OLMo 2

OLMo 2(@olmo2OLMo22025) is available in two sizes: **7B and 13B parameters**. Despite being trained on significantly fewer tokens, both versions achieve **performance on par with Llama 3 and Qwen 2.5 across standard open-source evaluation benchmarks**.

{{< figure src="/images/2024-11-25-olmo-2-blog-post-4-page-5.png" width="700px" class="align-center" >}}

On the “compute vs. model quality” curve, **OLMo 2 falls on the so-called Pareto optimal** frontier—demonstrating that with careful data curation and training strategies, it is possible to achieve competitive results without relying on massive computational resources.

{{< figure src="/images/olmo2-2.png" width="700px" class="align-center" >}}

### Tulu 3

Tulu 3(@lambertTulu3Pushing2025) is AI2’s latest instruction-tuned model, built on top of Llama 3-405B. Through multi-stage instruction tuning, safety alignment, and tool-augmented reasoning, **Tulu 3 now surpasses DeepSeek V3 and comes close to GPT-4o on reasoning-intensive tasks.**

{{< figure src="/images/image-23.png" width="700px" class="align-center" >}}

In the following sections, I will walk through this post-training pipeline step by step and explain how each component contributes to the final performance gains.

## Post Training: Tulu

我们从后训练讲起，因为现代大语言模型中的大部分“推理能力”，都是在这一阶段发展和强化的。构建现代大语言模型（LLM）通常包括两个主要阶段：

- 预训练（Pretraining）：模型通过大规模（主要来源于互联网）的数据学习下一个 token 的预测。这一阶段产出的基础模型具备一定的通用能力，但尚未安全，也缺乏指令遵循和强推理能力。
- 后训练（Post-training）：对基础模型进行精调，使其能够理解人类意图、使用工具、进行推理，并遵守安全性与合规性要求。

{{< figure src="https://www.datocms-assets.com/64837/1732046543-tulu-3-post-training-light-2.png?dpr=2&fit=max&fm=webp&h=810&w=1550" width="700px" class="align-center" title="The base pre-trained LMs are neither safe nor robust for public use and interactions, thus require post-training.(@Tulu3Opens)" >}}

Post-training serves multiple important functions:
- Alignment with Human Preferences: Collects data on which responses humans prefer
- Tool Use and Agent Capabilities: Enables searching, code execution, and other tool utilization
- Reasoning Enhancement: Teaches models to solve complex problems (e.g., arithmetic word problems)

### Tulu: Open Instruction Tuning Recipe

Tulu 是一套开放、可复现、具备领先性能的后训练方法。后训练流程包含3个关键步骤，这三步之间会根据模型反馈反复调整、迭代。

{{< figure src="https://www.datocms-assets.com/64837/1732167081-tulu3_figure1_v2-001-1.png?dpr=1.5&fit=max&fm=webp&h=810&w=1550" width="700px" class="align-center" title="An overview of the Tülu 3 recipe" >}}

1. **Instruction Tuning（指令微调）**：通过大量“指令 + 回答”的数据（可由人工标注或模型合成生成）对基础模型进行微调，使其更擅长执行指令任务。
2. **Preference Tuning（偏好微调）**：采集人类偏好反馈（例如“你更喜欢哪个回答？”），训练模型生成更符合人类期望的答案。Tulu 3 系统性比较了 DPO（Direct Preference Optimization）与 PPO（Proximal Policy Optimization）两种方法。
3. **Reinforcement Learning with Verifiable Reward（带可验证奖励的强化学习）**：在 RLHF 基础上进一步创新，通过引入可控、可验证的奖励信号对模型进行精调，增强其在复杂情境下的稳健性。

成功的模型适配从以下四个步骤开始：
1. 为每类目标能力（如数学、编程、安全性等）设立明确评估标准；
2. 针对这些能力设计具代表性的任务提示（Prompt）；
3. 确保数据合法合规，避免版权问题；
4. 对数据进行去污染，确保评估集与训练集无重叠。

在实际训练中，Tulu 3 结合这些步骤，针对不同任务能力构建了如下数据集组合：

{{< figure src="/images/Screenshot 2025-05-25 at 7.05.04 PM.png" width="700px" class="align-center"  title="Summary of prompt dataset">}}

#### Step 1: Supervised Finetuning

Tulu 训练流程的第一步是 Supervised Finetuning（SFT），旨在让预训练语言模型具备基本的任务执行能力。SFT 的方式是使用大量 “Prompt + Completion”（指令 + 回答）样本对模型进行微调，使其学会如何响应人类输入。

为了构建高质量的指令数据，Tulu 团队提出了一套可并行执行的“双轨数据构建策略”：
- 数据整理（Data Curation）：围绕核心任务（如对话、编程、推理等）设计并收集高质量样本；
- 数据混合（Data Mixing）：将人工标注数据与大模型生成数据进行组合，覆盖广泛能力，提升泛化效果。

在模型评估中，团队系统性测试了多个能力维度（对话、知识、推理、代码、多语言、安全性）的表现。为进一步提升训练效果，他们采用以下策略优化数据配置：
- 精选在特定任务上表现优异的数据集；
- 混合人工与合成数据，确保多样性与规模；
- 按任务能力维度调整数据比例，实现训练均衡。

{{< figure src="/images/Screenshot 2025-05-25 at 7.29.45 PM.png" width="700px" class="align-center"  title="Comparison of different instruction tuning datasets, showing that different instruction-tuning datasets can excel in different aspects, and mixtures perform best on average.">}}


**推理能力的数据挑战与解决方案**

相比输出单一答案的任务，推理类问题（如数学、逻辑等）往往更复杂，要求模型具备多步思维能力。研究表明，Chain-of-Thought（思维链）数据在这类任务中极为有效。但高质量 CoT 数据往往需要专家逐步标注，成本高昂、效率低，难以规模化采集，且风格单一，缺乏多样性。

为解决上述难题，Tulu 团队根据论文 Scaling Synthetic Data Creation with 1,000,000,000 Personas(@geScalingSyntheticData2025) 提出“Persona-Driven 数据生成”方法：
- 针对特定技能（如数学、代码）设计具象角色（如化学家、儿童、程序员）；
- 模型基于角色设定生成任务与解题过程，提升内容多样性与可扩展性;

{{< figure src="/images/Screenshot 2025-05-25 at 7.42.13 PM.png" width="700px" class="align-center"  title="Personas can work with a wide range of data synthesis prompts (e.g., “create a math problem”) to guide an LLM to synthesize data with corresponding perspectives.">}}

Tulu 团队设计了约 25 万个 persona（人物设定），并引导模型生成三类核心任务数据，并结合 GPT-4o 与 Claude Sonnet 补全逐步解答，构成完整的 Chain-of-Thought 样本。
- 数学问题（Math Problems）
  - ~150K 高难数学题
  - ~50K 小学数学题
- 编程问题（Python Coding）
  - ~35K Python 编程任务
- 精确指令遵循（Precise Instruction Following, IF）
  - ~30K 指令执行样本

实验结果表明：
- 添加 persona 数据后，模型在数学任务上明显提升，尤其在复杂问题上效果更好。GSM8K 这类简单题提升相对有限。

{{< figure src="/images/Screenshot 2025-05-25 at 7.52.18 PM.png" width="700px" class="align-center"  title="Impact of Persona-Driven Math Data">}}

为进一步提升质量，Tulu 引入 GPT-4 自一致性投票机制，保留最优解路径，剔除近 40% 噪声样本。最终仅保留 60% 数据，也能取得更高准确率。

{{< figure src="/images/Screenshot 2025-05-25 at 7.53.26 PM.png" width="700px" class="align-center"  title="Less data, Same or Better Performance">}}

Other approaches to generate COT data
1. Manual Human Annotation (e.g., GSM8K dataset): Annotators write step by step solutions
  - High-quality reasoning traces
  - Limited scale (only 7K)
  - Lack of diversity in reasoning styles
2. Program-Aided Language Models (PAL): Convert math problems into Python code execution traces
  - Guarantee correctness through execution
  - Less natural language reasoning, less intuitive
  - Limited to problems that can be coded
3. Self-generated COT (self-ask): using LLMs to generate their reasoning paths
  - Scalable to many problems
  - Quality highly dependent on base model

**Capability-driven Data Mixing**

1. Data mixing for SFT
  - Training on real user interactions with strong models is helpful almost across the board.
  - Safety training is largely orthogonal to the other skills.
  - Persona-based data synthesis is very useful for targeting new skills.

{{< figure src="/images/Screenshot 2025-05-25 at 8.01.44 PM.png" width="700px" class="align-center"  title="Performance during our SFT ablations, showing the effect of removing safety, WildChat, Persona, and Math data in isolation.">}}

2. SFT performance potential

SFT mixtures show strong performance, achieving a higher average score than other comparable mixes. All models, including Tülu 2 SFT, were trained on either Llama 3.0 or 3.1. Our final Tülu 3 70B model was used to help format this table.

{{< figure src="/images/Screenshot 2025-05-25 at 8.05.04 PM.png" width="700px" class="align-center"  title="Summary of the performance of our Tülu 3 SFT models against comparable baselines.">}}

#### Step 2: Preference tuning













