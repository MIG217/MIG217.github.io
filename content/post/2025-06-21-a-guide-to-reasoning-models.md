---
title: "How to Use Reasoning Models?"
author: ["Mingrui Guo"]
date: "2025-06-15"
categories: ["AI"]
tags: ["Reasoning Models"]
draft: false
ShowToc: true
TocOpen: false
math: true
---


<style>
pre code {
  white-space: pre;
  overflow-x: auto;
}
</style>

<p style="color: #1E90FF;">
  The following insights are drawn from the <em>Reasoning with o1</em> video course by <a href="https://learn.deeplearning.ai/courses/reasoning-with-o1/lesson/h8dkv/introduction" style="color: #1E90FF;">DeepLearning.ai</a>.
</p>
<br>


This article explores how to effectively prompt and utilize the new generation of reasoning models. Models released over the past year have demonstrated remarkable progress in reasoning and planning tasks. OpenAI has deeply optimized Chain of Thought (CoT) processing, using reinforcement learning to fine-tune models so they automatically integrate step-by-step reasoning into their response process. 

While current model performance is already impressive, the more significant long-term development is **reasoning-time scalability**. Reasoning model performance improves not only with increased training compute but also with the thinking time allocated during **inference (test-time or inference-time compute)**. This provides an entirely new dimension for scaling large model performance.

However, **reasoning models aren't suitable for every scenario**. This article will cover the types of tasks reasoning models excel at, and when you might need smaller, faster models, or even hybrid approaches. This article structure includes:

- Introduction to Reasoning Models
- Designing Prompts for Reasoning Models
- Using Reasoning Models for Planning
- LLMs as Judges
- Meta Prompting

## Introduction

Before reasoning model emerged, most AI models behaved like children-always blurting out the first thing that came to mind. The revolutionary breakthrough of reasoning models lies in learning a valuable skills: **think before you speak**. This enables them to achieve unprecedented performance levels in complex tasks including **mathematics, programming, science, stragegic planning, and logical reasoning**. 

### CoT: The Core Mechanism

The key advantage of reasoning models lies in their **native integration of chain-of-thought reasoning process** (@weiChainofThoughtPromptingElicits2023). Let's understand this through an example. When we present the model with a letter scrambling problem:

```
oyfjdnisdr rtqwainr acxz mynzbhhx -> Think step by step
Use the example above to decode:
oyekaijzdf aaptcg suaokybhai ouow aght mynznvaatzacdfoulxxz
```

Rather than providing an immediate answer, the model engages in the following thought process: [Cipher Decoding Process](https://chatgpt.com/share/684c1c86-b920-8001-920a-54cd3e81378b)

1. **Problem Understanding:** Analyze the given example to identify patterns
2. **Hypothesis Formation:** Consider whether it's an anagram or some form of cipher
3. **Hypothesis Testing:** Notice that the encrypted text is exactly twice the length of the orginal
4. **Iternative Refinement:** When the first hypothesis fails, use existing information to form new hypotheses
5. **Optimal Path Discovery:** Through continuous trial and error, ultimately find the correct solution

This process encompasses the key steps humans use when solving complex problems:

- Problem and solution space identification
- Hypothesis development and testing
- Approach adjustment and path selection


What makes reasoning model so speical is that you don't need complex prompts to guide them through deep thinking. This means reasoning models **requires less contextual prompting to produce high-quality results** for complex tasks, truly achieving the leap from "quick reaction" to "deep thinking".

Of course, this deep reasoning comes with trade-offs - when using reasoning models, you need to **balance reasoning quality against response speed**.


### Breakthrough and Performance Leap

The performance leap of reasoning model is primarily attributed to two key breakthroughs:

**1. Inference-time Compute**

Research has found that in the model's post-training phase, the more reinforcement learning conducted, the higher the model's accuracy. But more surprisingly, **allowing models to "think longer" during inference significanly improves result quality**. By giving models more thinking time, even with the same model parameters and training data, superior performance can be achieved.

{{< figure src="/images/02-reasoning_tokens.png" width="700px" class="align-center" title="Image source: [OpenAI](https://openai.com/index/learning-to-reason-with-llms/)" >}}

**2. Consensus Voting**

Another key breakthrough is teaching models to verify outputs through **consensus voting**. The mechanism works as follows:

1. Generate multiple different solutions for the same problem
2. Train the model select the most frequently occuring solution as the final answer

In Minerva's experiments, MATH benchmark accuracy improved from **33.6% to 50.3%** (@brownLargeLanguageMonkeys2024). Experiments showed that the consensus mechanism **stabilizes at around 100 samples**, meaning **significant performance improvements can be achieved without generating massive number of samples**.

{{< figure src="/images/Screenshot 2025-06-14 at 12.13.19 PM.png" width="700px" class="align-center" title="Comparing coverage (performance with an oracle verifier) to mainstream methods available for picking the correct answer (majority voting, reward model selection and reward model majority voting) as we increase the number of samples." >}}

These breakthroughs have yielded remarkable results: Taking GPT-4o and o1 models as examples:

- **Mathematical Olympiad-level abilities (AIME 2024):** GPT-4o achieved 13% accuracy, while o1 reached 83%
- **Coding abilities:** GPT-4o scored 11%, while o1 achieved 89%

{{< figure src="/images/headline-desktop.png" width="700px" class="align-center" title="o1 greatly improves over GPT-4o on challenging reasoning benchmarks." >}}

- **General Mathematics (MATH):** o1 achieved a massive 30% improvement over GPT-4o
- **College-level Knowledge (MMLU):** o1 improved across all categories, with college mathematics accuracy reaching an astounding 98.1%

{{< figure src="/images/breakdown.png" width="700px" class="align-center" title="o1 improves over GPT-4o on a wide range of benchmarks, including 54/57 MMLU subcategories. " >}}

### Emerging Capabilities of Reasoning Models

Beyond leaps in traditional benchmarks, reasoning models have demonstrated some execiting emerging capabilities.


**1. Abstract Reasoning**

When given 16 words and asked to find underlying categories and correctly classify them, GPT-4o's performance was somewhat random, identifying only two categories with errors. Meanwhile, o1 perfectly identified all four categories and correctly classified all 16 words. This capability is crucial for handing abstract problems beyond standardized tests.

{{< figure src="/images/Screenshot 2025-06-14 at 2.11.37 PM.png" width="700px" class="align-center" title="Abstract reasoning" >}}

**2. Generator-Verifier Gap**

For many problems (such as mathematics, programming, puzzles), verifying a good answer is much easier than generating one from scratch. Reasoning models excel at leveraging this principle. They can:

- Generate an initial solution
- Verify it and identify issues
- Iterate based on feedback, gradually approaching the perfect answer

When this **generator-verifier gap** exists, we can trade more computation at inference time for better performance.

**3. Potential Application Areas**

This powerful reasoning capability makes these models highly promising in the following domains:

- **Data Analysis:** Interpreting complex datasets, such as genomic sequencing results in biology
- **Scientific Computing:** Writing and debudding specialized code for computational fluid dynamics or astrophysics simulations
- **Experimental Design:** Proposing new experimental approaches in chemistry or explaining complex physics experimental results
- **Algorithm Development:** Assisting in creating or optimizing data analysis algorithms bioinformatics
- **Literature Synthesis:** Reasoning across multiple research papers to form coherent conclusions


## Effective Prompting

Here are four key prompting principles that have emerged for working with reasoning models. While these principles don't cover every scenario, they can help you explore and understand how this new generation of reasoning models differs from others.

### 1. Keep It Simple and Direct

When writing prompts, aim for **clarity and conciseness**. Direct instructions often produce the best results, while complex descriptions and excessive background information can actually interfere with the model's internal reasoning process.

### 2. No Need for Explicit Chain-of-Thought

Reasoning models no longer require manually adding "think step by step" or breaking down tasks steps in your prompts like previous models did. These models have been trained to natually provide effective explanations and reasoning processes in their responses. Therefore, starting with a simple, direct prompt is always the best approach.

Of course, for highly specialized tasks or complex contexts, you might still consider adding CoT-style step-by-step peompts. However, it's recommended to **start with simple prompts and adjust details based on output quality**.

For example: Suppose you need a function that outputs SMILES IDs for all molecules related to insulin.

**Good Prompt**

```yaml
"Generate a function that outputs the SMILES IDs for all the molecules involved in insulin."
```

**Bad Prompt**
```yaml
"Generate a function that outputs the SMILES IDs for all the molecules involved in insulin."
"Think through this step by step, and don't skip any steps:"
"- Identify all the molecules involve in insulin"
"- Make the function"
"- Loop through each molecule, outputting each into the function and returning a SMILES ID"
"Molecules:"
```

### 3. Use Structured Prompts

When your prompt content becomes complex, **use separators (like Markdown, XML tags, or quotes) to break it into different sections**. This structured format not only improves model accuracy but also makes troubleshooting much easier. 

Example: Customer service assistant scenario

```yaml
   "<instructions>You are a customer service assistant for AnyCorp, a provider"
   "of fine storage solutions. Your role is to follow your policy to answer the user's question. "
   "Be kind and respectful at all times.</instructions>\n"
   "<policy>**AnyCorp Customer Service Assistant Policy**\n\n"
   "1. **Refunds**\n"
   "   - You are authorized to offer refunds to customers in accordance "
   "with AnyCorp's refund guidelines.\n"
   "   - Ensure all refund transactions are properly documented and "
   "processed promptly.\n\n"
   "2. **Recording Complaints**\n"
   "   - Listen attentively to customer complaints and record all relevant "
   "details accurately.\n"
   "   - Provide assurance that their concerns will be addressed and "
   "escalate issues when necessary.\n\n"
   "3. **Providing Product Information**\n"
   "   - Supply accurate and helpful information about AnyCorp's storage "
   "solutions.\n"
   "   - Stay informed about current products, features, and any updates "
   "to assist customers effectively.\n\n"
   "4. **Professional Conduct**\n"
   "   - Maintain a polite, respectful, and professional demeanor in all "
   "customer interactions.\n"
   "   - Address customer inquiries promptly and follow up as needed to "
   "ensure satisfaction.\n\n"
   "5. **Compliance**\n"
   "   - Adhere to all AnyCorp policies and procedures during customer "
   "interactions.\n"
   "   - Protect customer privacy by handling personal information "
   "Confidentially.\n\n"
   "6. **Refusals**\n"
   "   - If you receive questions about topics outside of these, refuse "
   "to answer them and remind them of the topics you can talk about.</policy>\n"
```
We use `<instructions>` tags to clearly define the task role and behavioral guidelines. Then we provide structured policy content through `<policy>` tags, making it clear to the model what "policy" specifically refers to. This way, the model clearly understands its role, the rules it should follow, and the tasks it needs to complete, with clear boundaries and minimal confusion.

### 4. Show rather than Tell

Instead of explaining your requirements through lengthy text descriptions, **provide a relevant example** that allows the model to intuitively understand the task domain and output format you expect.

Here's and example: We still use `<prompt>` and `<policy>` to define roles and rules, but we add an `<example>` tag that directly provides a sample question-answer pair to help the model understand the expected response format and citation style. 

```python
"<prompt>You are a lawyer specializing in competition law, "
"assisting business owners with their questions.</prompt>\n"
"<policy>As a legal professional, provide clear and accurate "
"information about competition law while maintaining "
"confidentiality and professionalism. Avoid giving specific "
"legal advice without sufficient context, and encourage clients "
 "to seek personalized counsel when necessary.</policy>\n"
"""<example>
<question>
I'm considering collaborating with a competitor on a joint marketing campaign. Are there any antitrust issues I should be aware of?
</question>
<response>
Collaborating with a competitor on a joint marketing campaign can raise antitrust concerns under U.S. antitrust laws, particularly the Sherman Antitrust Act of 1890 (15 U.S.C. §§ 1–7). Section 1 of the Sherman Act prohibits any contract, combination, or conspiracy that unreasonably restrains trade or commerce among the states.


**Key Considerations:**


1. **Per Se Illegal Agreements:** Certain collaborations are considered automatically illegal ("per se" violations), such as price-fixing, bid-rigging, and market allocation agreements. For example, in *United States v. Topco Associates, Inc.*, 405 U.S. 596 (1972), the Supreme Court held that market division agreements between competitors are per se illegal under the Sherman Act.


2. **Rule of Reason Analysis:** Collaborations that are not per se illegal are evaluated under the "rule of reason," which assesses whether the pro-competitive benefits outweigh the anti-competitive effects. In *Broadcast Music, Inc. v. Columbia Broadcasting System, Inc.*, 441 U.S. 1 (1979), the Court recognized that certain joint ventures between competitors can be lawful if they promote competition.


3. **Information Sharing Risks:** Sharing competitively sensitive information, such as pricing strategies or customer data, can lead to antitrust violations. The Department of Justice and the Federal Trade Commission caution against exchanges that could facilitate collusion (*Antitrust Guidelines for Collaborations Among Competitors*, 2000).


**Recommendations:**


- **Define the Scope:** Clearly delineate the parameters of the collaboration to focus on the marketing campaign without involving competitive aspects like pricing or market division.
- **Implement Safeguards:** Establish protocols to prevent the exchange of sensitive information that is not essential to the marketing effort.
- **Legal Consultation:** Given the complexities of antitrust laws, consult with a legal professional to ensure the collaboration complies with all legal requirements.


**Conclusion:**


While joint marketing campaigns between competitors are not inherently illegal, they must be structured carefully to avoid antitrust pitfalls. Legal guidance is essential to navigate these issues and to design a collaboration that achieves your business objectives without violating antitrust laws.
</response>
</example>"""
```

In practice, you can use these principles as your default starting point and gradually adjust based on task complexity. If the model doesn't perform as expected, rather than adding more explanations, **prioritize adding or improving examples first**. This typically leads to more robust results. 




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



