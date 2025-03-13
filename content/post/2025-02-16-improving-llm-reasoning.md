---
title: "如何优化大语言模型（LLM）的推理能力？"
author: ["Mingrui Guo"]
date: "2025-02-16"
categories: ["AI"]
tags: ["LLM", "Agents"]
draft: false
ShowToc: true
TocOpen: false
---

2024年，大语言模型在推理能力方面取得了显著突破。以O系列模型为例，在ARC-AGI评估任务中展现了令人瞩目的性能【1】：

- **O3模型达到了87.5%的准确率**，尽管每个任务的计算成本较高（超过$1,000）
- 相比之下，**未采用特殊推理技术的传统LLMs准确率通常低于25%**

{{< figure src="/images/o-series-performance.jpg" title="Fig.1: O-Series Performance" width="700px" class="align-center" >}}

如何通过有效的Prompting 来激发大语言模型的深层次推理能力，一直是研究者和开发者关注的核心问题。以下是几种主要的触发方法：

- **少量示例CoT提示（Few-shot CoT）**：通过提供少量推理示例，引导模型学习推理模式并应用到新问题中。
- **指令型提示（Instruction prompting）**：明确指导模型逐步思考问题，避免直接跳至答案。
- **指令微调（Instruction tuning）**：针对多步思考的推理任务对模型进行微调，提升其在类似任务中生成连贯思维链的能力。
- **强化学习（Reinforcement learning）**：利用强化学习技术训练模型，使其能够生成更完整、更准确的推理链。

**本文重点：**

我们将深入探讨Inference-time techniques，特别关注如何通过扩展token预算来提升LLM的推理能力。主要包括三个维度：

- 基本提示词技巧：**使用更多的token预算来生成单一的解决方案**。
- 从多个候选中进行搜索和选择，增加推理的**宽度**。
- 模型迭代自我改进，增加推理的**深度，最终到达最优解**。

## 使用更多的Token生成单一解决方案 

优化提示词能显著提升模型在各类任务中的表现。本节将介绍一些提示词工程技术，帮助我们更好地完成复杂任务。
下图对比了Standard Prompting和CoT Prompting两种方法【2】【3】：

- **Standard Prompting**：仅给出最终答案，没有推理过程，容易导致错误结果。
- **CoT Prompting**：展示完整的推理过程，让**模型清晰地说明从问题到答案的推导步骤**。

{{< figure src="/images/CotvsSd.png" title="Fig.2: Chain-of-thought prompting enables large language models to tackle complex arithmetic commonsense, and symbolic reasoning tasks. Chain-of-thought reasoning processes are highlighted." width="700px" class="align-center" >}}

### Zero-shot CoT Prompting：通过指令引导生成思维链推理

0-shot CoT 是一种通过简短指令引导LLM进行推理的方法，无需依赖任何示例。在这种方法中，模型不需要看到具体的示范或训练数据，仅**通过一个简单的指令（如"Let's think step by step."）（fig.3）即可开始推理**【4】。

{{< figure src="/images/0-shot-cot.png" title="Fig.3: Example inputs and outputs of GPT-3" width="700px" class="align-center" >}}

**优缺点：**

- **0-shot CoT的表现显著优于普通的0-shot方法**，特别是在数学和符号推理等较难的任务中（fig.4）
- 相比Few-shot CoT，0-shot CoT更加便捷，因为无需手动标注示例。但从性能来看，**0-shot CoT的表现仍然不如Few-shot CoT**（fig.5）

{{< figure src="/images/accuractcomparison.jpg" caption="fig.4" >}}
{{< figure src="/images/gsm8k.jpg" caption="fig.5" >}}



## 参考文献：
1. “OpenAI O3 Breakthrough High Score on ARC-AGI-Pub.” ARC Prize. Accessed February 15, 2025.
2. Wei, Jason, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, et al. "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models." Preprint, arXiv, January 10, 2023.
3. Nye, Maxwell, Anders Johan Andreassen, Guy Gur-Ari, Henryk Michalewski, Jacob Austin, et al. "Show Your Work: Scratchpads for Intermediate Computation with Language Models." Preprint, arXiv, November 30, 2021.
4. Kojima, Takeshi, Shixiang Shane Gu, Machel Reid, Yutaka Matsuo, and Yusuke Iwasawa. "Large Language Models are Zero-Shot Reasoners." Preprint, arXiv, January 29, 2023.
5. Yasunaga, Michihiro, Xinyun Chen, Yujia Li, Panupong Pasupat, Jure Leskovec, et al. "Large Language Models as Analogical Reasoners." Preprint, arXiv, March 9, 2024.
6. Yang, Chengrun, Xuezhi Wang, Yifeng Lu, Hanxiao Liu, Quoc V. Le, Denny Zhou, and Xinyun Chen. "Large Language Models as Optimizers." Preprint, arXiv, April 15, 2024.
7. Zhou, Denny, Nathanael Schärli, Le Hou, Jason Wei, Nathan Scales, et al. "Least-to-Most Prompting Enables Complex Reasoning in Large Language Models." Preprint, arXiv, April 16, 2023.
8. Zhou, Pei, Jay Pujara, Xiang Ren, Xinyun Chen, Heng-Tze Cheng, et al. "Self-Discover: Large Language Models Self-Compose Reasoning Structures." Preprint, arXiv, February 6, 2024.
9. Wang, Xuezhi, Jason Wei, Dale Schuurmans, Quoc Le, Ed Chi, and others. "Self-Consistency Improves Chain of Thought Reasoning in Language Models." Preprint, arXiv, March 7, 2023.
10. Li, Yujia, David Choi, Junyoung Chung, Nate Kushman, Julian Schrittwieser, and others. "Competition-Level Code Generation with AlphaCode." Science, Decemeber 9, 2022.
11. Chen, Xinyun, Renat Aksitov, Uri Alon, Jie Ren, Kefan Xiao, and others. "Universal Self-Consistency for Large Language Model Generation." Preprint, arXiv, November 29, 2023.
12. Yao, Shunyu, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, and Karthik Narasimhan. "Tree of Thoughts: Deliberate Problem Solving with Large Language Models." Preprint, arXiv, December 3, 2023.
13. Shinn, Noah, Federico Cassa, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. "Reflexion: Language Agents with Verbal Reinforcement Learning." Preprint, arXiv, October 10, 2023.
14. Madaan, Aman, Niket Tandon, Prakhar Gupta, Skyler Hallinan, Luyu Gao, and others. "Self-Refine: Iterative Refinement with Self-Feedback." Preprint, arXiv, May 25, 2023.

15. Huang, Jie, Xinyun Chen, Swaroop Mishra, Huaixiu Steven Zheng, Adams Wei Yu, Xinying Song, and Denny Zhou. "Large Language Models Cannot Self-Correct Reasoning Yet." Preprint, arXiv, March 14, 2024.

16. Du, Yilun, Shuang Li, Antonio Torralba, Joshua B. Tenenbaum, and Igor Mordatch. "Improving Factuality and Reasoning in Language Models through Multiagent Debate." Preprint, arXiv, May 23, 2023.

17. Sutton, Richard. The Bitter Lesson.