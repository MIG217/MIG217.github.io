---
title: "A Guide to Reasoning Models"
author: ["Mingrui Guo"]
date: "2025-03-21"
categories: ["AI"]
tags: ["Reasoning Models"]
draft: true
ShowToc: true
TocOpen: false
math: true
---

推理模型在人工智能领域标志着一种重要的发展，显著提升了系统在问题求解与复杂推理任务中的能力。本文将讨论这些模型适用的场景、其使用方式，以及如何设计有效的提示以进一步优化其推理表现。

*This content is adapted from DeepLearning.AI's course "Reasoning with o1".*


## Introduction

o1 是 OpenAI 于 2024 年推出的新一代推理模型系列。其显著特点在于具备“先思考后作答”的能力，即在生成最终答案之前会进行系统性的推理过程。这一特性使得 o1 在处理复杂任务时展现出更高的准确性与一致性，显著提升了其在多步骤问题解决中的表现。

### o1的工作原理

**思维链推理：核心机制**

The thing that makes the o1 family so different is how they natively integrate chain of thought into how they solve problems.
o1系列模型的独特之处在于它们将思维链原生集成到问题解决过程中。其思维过程包括：

1. 识别问题和解决方案空间
2. 发展假设
3. 测试假设
4. 拒绝想法和回溯
5. 识别最有希望的路径



**强化学习：训练过程**

during the post-training process, the more reforcement learning we did, more accurate the model got. But what was possibly more surprising was the more we allowed it to think at inference time, we got an even sharper increase in pperformance. 

So they key breakthrough here was the ability to think for a longer time at inference time. And get better results. 

A breakthrough in reasoning: the ability to think for a longer time and get better results.

**Verfication in LLMs via Consensus / Majority Vote**

The other key breakthrough that led to the increase in performance with o1 was teaching it to verify outputs via consensus voting. 

- Generate a bunch of solutions and take the most common one
    - Can be thought of as similar to **sampling at low temperature**
- Mainerva [Lewkowycz et al.] goes from 33.6% to 50.3% on MATH due to consensus
- Consensus flatlines before 100 samples. You don’t need a huge amount of samples to realize the performance improvement here.

### 推理模型优势

**适用场景**


This article explores Reasoning models, which has demonstrate remarkable capabilities in reasoning and planning tasks. While powerful, reasoning model isn't always the optimal choice for every solution. I'll clarify when and how to effective use reasoning model.

- 数据分析：解释复杂数据集（如基因组测序结果）
- 数学问题求解：推导解决方案或证明
- 实验设计：提出化学实验设置或解释物理实验结果
- 科学编程：编写和调试专业代码
- 生物和化学推理：解决需要深度领域知识的问题
- 算法开发：帮助创建或优化数据分析工作流程
- 文献综合：跨多个研究论文进行推理

### Takeaway

- o1系列模型通过产生推理令牌来扩展推理时间的计算能力
- o1提供更高的智能，但需要权衡更高的延迟和成本
- 它在需要测试和学习方法的任务中表现出色
- 主要应用场景包括规划、编码和特定领域推理（如法律和STEM学科）

## Prompting

###  Principles

There are four principles for prompting the o1 models. 

1. **Simple & direct**: Write prompts that are straightforward and concise. Direct instructions yield the best results with the o1 models.
2. **No exolicit CoT required**: You can skip step-by-step ("Chain of Thought") reasoning prompts. The o1 models can infer and execute these itself without detailed breakdowns.
3. **Structure**: 
   1. Break complex prompts into sections using delimiters like markdown, XML tags, or quotes.
   2. This structured format enhances model accuracy - and simplifies your own troubleshooting.
4. **Show rather than tell**: Rather than using excessive explanation, give a contextual examples to give the model understanding of the broad domain of your task.

### Prompt Comparison: Good vs. Bad

| 🔍 Prompt Type | 💬 Example |
|---------------|------------|
| ❌ **Bad Prompt** | ```text  Tell me something about AI.  ``` |
| ✅ **Good Prompt** | ```text  Explain the difference between supervised and unsupervised learning with real-world examples.  ``` |


## Planning

One the great use cases where o1 models shine is creating a plan to solve a task. Given a set of tools to carry out the plan and constraints to set bounds around the task. 

This kind of use case would be very slow if we used o1 for every step. So what we'll do is generate a plan with o1-mini, and then execute each step with GPT-4o-mini. This trade-off of intelligence against lantency and cost is a practical one that we've seen developers use to great effect so far. 

推理模型最擅长的一个应用场景是制定解决任务的计划，在这个过程中，我们有一组工具来执行计划，并且有约束条件来限定任务的范围。

但是，如果在任务的每一个步骤都使用推理模型来执行，那整个过程就会变得非常慢。因此，一个更好的做法是用推理模型生成计划，然后用非推理模型执行每个具体步骤。

这种方法实际上是在智能性、延迟和成本之间做权衡。这种策略已经被开发者广泛使用，并且效果很好。

步骤：
- OpenAI o系列模型/ Deepseek R1 负责制定计划，因为它更适合高效地规划任务。
- ChatGPT 4o / Deepseek V3 负责执行计划的各个步骤，这样可以减少延迟并降低成本。
这个方法本质上是在性能和资源消耗之间取得平衡，是一种实际可行的优化策略。

### Plan Generation + Execution Architecture

Let's me tell you about the overall architecture of the task, so that we don't get lots in the details as we go through the code.

You begin with a Scenario. This comes from a customer where they're making an ask which requires multi-step logic to answer that scenario. That scenario will be given the o1-mini, who will have at its disposal some instructions tof how to build a plan, and then a number of tools which it can use to carry out the plan. This is great use case for o1, where we use its multi-step reasoning logic to build a durable plan before we then engage 4o-mini as w worker to carry out each step of the plan. 

Once the plan is completed, we'll have an answer which we provide back to our customer. 

Now let's dig into the code and see how this work in practice.



