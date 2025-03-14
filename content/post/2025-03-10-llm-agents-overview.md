---
title: "LLM Agents: Brief History and Overview"
author: ["Mingrui Guo"]
date: "2025-03-10"
categories: ["AI"]
tags: ["LLM Agents", "EN"]
draft: false
ShowToc: true
TocOpen: false
---

## Introduction

To understand LLM agents, we need to break the term into two foundational components: **Large Language Models (LLMs)** and **Agents**. While LLMs have gained widespread recognition, the concept of "agent" in this context requires deeper exploration. 

### What is an Agent?

In artificial intelligence, an agent is **an "intelligent" system that perceives and interacts with an "environment" to achieve specific goals**. The classification of agents varies based on their operational environment:

- **Physical environments**: robots, autonomous vehicles, etc.
- **Digital environments**: DQN for Atari games, Siri, AlphaGo
- **Humans as environments**: Chatbots

Agents typically follow a **perception-reasoning-action cycle**, where they:

1. Observe their environment  
2. Process information and make decisions  
3. Take actions that affect the environment

{{< figure src="/images/agents.png" title="Fig.1: Agent-Environment Interaction" width="400px" class="align-center" >}}

### What's an LLM Agent?

An LLM agent **integrates the powerful language capabilities of LLMs with the goal-oriented, interactive nature of agents**. These systems represent a significant evolution in AI, with capabilities **ranging from basic conversational skills to complex reasoning and planning**.

LLM agents can be classified into three progressive levels of sophistication:

**Level 1: Text Agent**
- Basic agents that process and respond to text input
- Examples: ELIZE, LSTM-DQN

**Level 2: LLM Agent**
- Advanced agents that leverage LLMs for direct action generation
- Examples: SayCan, Language Planner

**Level 3: Reasoning Agent**
- Use LLM to reason to act
- Examples: ReAct, AutoGPT

{{< figure src="/images/3levelagents.png" title="Fig.2: Three Levels of LLM Agents: From Text to Reasoning" width="500px" class="align-center" >}}

## Pre-LLM Language Agents

### ELIZA (1966): The Pioneer of Text Agents

The development of text-based agents dates back to the early days of AI. ELIZA, created in 1966, **marked a significant milestone as one of the first chatbots**.

It's simple yet effective **rule-based approach involved pattern matching and response templates to simulate human conversation**. While users found ELIZA remarkably engaging, the system had inherent limitions:

- Limited to **specific domains** and use cases
- Require a extensive **manual rule** creation
- Unable to handle **complex interactions** or understanding

{{< figure src="/images/ELIZA_conversation.png" title="Fig.3: ELIZA: The First Chatbot" width="500px" class="align-center" >}}

Despite these constraints, **ELIZA established the conceptual foundation for future conversational agents and demonstrated the potential of natural language interfaces**.

### LSTM-DQN (2015): Reinforcement Learning for Text Agents

Prior to the emergence of LLMs, **RL was a dominant approach for developing text-based agents**. This methodology **treated text as both the observation and action space**, similar to how traditional RL handles pixels and keyboard inputs in video games. The core idea was that optimizing for reward signals would natually lead emergence of language intelligence[1].

{{< figure src="/images/RL4textagent.png" title="Fig.4: LSTM-DQN for Reinforcement Learning in Text-Based Environments" width="700px" class="align-center" >}}


However, this approach faced several significant limitions:

- Domain-specific applications
- Dependence on explicit scalar reward signals
- Requires extensive training
  
**These early approaches highlighted both the promise and challenges of creating intelligent text-based agents, setting the stage for the transformative impact that large language models would later bring to this field.**

## The Emergence of Large Language Models

LLMs have revolutionized text agents through **next-token prediction on massive text corpora. During inference, they solve diverse new tasks through prompting alone**[2]. This emergent generality creates exciting possibilities for building more capable agents.

### A Brief History of LLM Agents

The rise of LLM agents began with models like GPT-3 in 2020. Initially, researchers explored their potential across diverse tasks, which broadly fell into two categories:

- **Reasoning tasks**: such as symbolic question answering and logical inference
- **Acting tasks**: including interactive applications like games and robotics

{{< figure src="/images/historyofllmagents.png" title="Fig.5: Evolution of LLM Agents: From Reasoning and Acting to ReAct" width="700px" class="align-center" >}}

Over time, reasoning and acting converged, giving rise to **reasoning agents—models** that **combine structured thinking with goal-driven actions**. This led to two key research directions:

1. **Applications**: Web interaction, software engineering, scientific discovery, and more
2. **Methods**: Memory systems, planning, multi-agent collaboration, and adaptive learning

### Enhancing LLMs with External Knowledge and Computation

While LLMs excel at many tasks, some require more than just next-token prediction—they **demand reasoning, external knowledge, or computation**. To address these limitations, researchers have developed various techniques.

**(1) Code-Augmented Computation**

For tasks involving calculations or formal reasoning, LLMs can **generate code instead of directly predicting an answer**. The generated code is then executed to produce the final result[3].

Example: Prime factorization, Fibonacci sequences 

**(2) Retrieval-Augmented Generation (RAG) for Knowledge**

For knowledge-intensive queries, LLMs can **retrieve relevant information from external corpora before generating a response**[4]. This is typically using:

- **Extra corpora** 
- **A retriever** (e.g., BM25, DPR, etc.) 

{{< figure src="/images/retrievalbasednlp.png" title="Fig.6: An illustration comparing (a) black-box language models and (b) retrieval-oriented NLP models, the paradigm this post advocates for" width="700px" class="align-center" >}}

**Limitation: RAG depends on the availability of a relevant corpus**. If the needed information is missing (e.g., "Who is the latest Prime Minister?"), retrieval alone is insufficient.

**(3) Tool-Use for Dynamic Information**

When static corpora fall short, LLMs can invoke **external tools in real time**. This is achieved by introducing special tokens that trigger API calls[5][6]. Common tools include: 

- Search engine, calculator, etc.
- Task-specific models (translation)
- APIs

**This approach significantly expands capabilities but introduces new challenges in tool selection and interaction management.**

{{< figure src="/images/TALM.png" title="Fig.7: Examples of TALM Text-to-Text Interface in Different Tasks" width="700px" class="align-center" >}}

### What if both knowledge and reasoning are needed?

Many tasks require both **reasoning and external knowledge**, pushing researchers to develop hybrid approaches. For example, one can interleave retrieval with chain-of-thought reasoning[7] or generate follow-up queries to refine responses[8].



However, early solutions were **fragmented**. Even within a single task like QA, different benchmarks posed distinct challenges, leading to a proliferation of task-specific techniques.






## References

[1] Narasimhan, Karthik, Tejas Kulkarni, and Regina Barzilay. “Language Understanding for Text-Based Games Using Deep Reinforcement Learning.” arXiv preprint, September 11, 2015.

[2] Brown, Tom B., Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared Kaplan, and others. “Language Models Are Few-Shot Learners.” arXiv preprint, July 22, 2020.

[3] Chen, Wenhu, Xueguang Ma, Xinyi Wang, and William W. Cohen. “Program of Thoughts Prompting: Disentangling Computation from Reasoning for Numerical Reasoning Tasks.” arXiv. November 22, 2022.

[4] SAIL Blog. "Building Scalable, Explainable, and Adaptive NLP Models with Retrieval." October 5, 2021.

[5] Parisi, Aaron, Yao Zhao, and Noah Fiedel. "TALM: Tool Augmented Language Models." arXiv, May 24, 2022.

[6] Schick, Timo, Jane Dwivedi-Yu, Roberto Dessì, Roberta Raileanu, Maria Lomeli, and others. "Toolformer: Language Models Can Teach Themselves to Use Tools." arXiv, February 9, 2023.

[7] Trivedi, Harsh, Niranjan Balasubramanian, Tushar Khot, and Ashish Sabharwal. "Interleaving Retrieval with Chain-of-Thought Reasoning for Knowledge-Intensive Multi-Step Questions." arXiv, June 23, 2023.

[8] Press, Ofir, Muru Zhang, Sewon Min, Ludwig Schmidt, Noah A. Smith, and Mike Lewis. "Measuring and Narrowing the Compositionality Gap in Language Models." arXiv, October 17, 2023.

[9] Yao, Shunyu, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. "ReAct: Synergizing Reasoning and Acting in Language Models." arXiv, March 10, 2023.

[10] Shinn, Noah, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. "Reflexion: Language Agents with Verbal Reinforcement Learning." arXiv, October 10, 2023.

