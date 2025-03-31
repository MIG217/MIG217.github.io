---
author:
- Mingrui Guo
bibliography: /Users/mig217/mig217/content/My Library.bib
categories:
- AI
csl: "https://www.zotero.org/styles/apa"
date: 2025-03-30
draft: false
ShowToc: true
tags:
- LLM Agents
- ZH
title: Memory, Reasoning, and Planning of Language Agents
TocOpen: false
---

Language Agents have emerged as one of the most exciting research
directions in AI over the past two years. **This article explores three
core components**: **long-term memory via HippoRAG, reasoning
capabilities with Grokked Transformers, and world modeling through
WebDreamer**---all essential for building effective language-driven AI
systems.

## Why Agents Again?

Russell & Norvig in "Artificial Intelligence: A Modern Approach" define
an agent as "**anything that can perceive its environment through
sensors and act upon that environment through actions.**"("Artificial
Intelligence: A Modern Approach, 4th US Ed.," n.d.)

\*\*\*\*\*\*\*image\*\*\*\*\*

Many people believe modern agents can be simply defined as **"LLM +
external environment."** This view suggests that language models
themselves have limited functionality with only text input-output
interfaces; once connected to an external environment, able to perceive
environmental information and influence the environment, they become
agents.

However, **this definition is oversimplified**. In reality, there are
two main competing views in the community:

- **LLM-first view: We make an LLM into an agent**
  - Implications: scaffold on top of LLMs, prompting focused, heavy on
    engineering
- **Agent-first view: We integrate LLMs into AI agents so they can use
  language for reasoning and communication**
  - Implications: All the same challenges faced by previous AI agents
    (e.g., **perception, reasoning, world models, planning**) still
    remain, but we need to re-examine them through the new lens of LLMs
    and tackle new ones (e.g., **synthetic data, self-reflection,
    internalized search**)

### Characteristics of Modern Language Agents

Contemporary AI agents, with integrated LLMs, can **use language as a
vehicle for reasoning and communication**

- **Instruction following, in-context learning, output customization**
- **Reasoning (for better acting): state inference, self-reflection,
  replanning, etc.**

Unlike traditional agents, reasoning in language agents is essentially a
new form of "action". In traditional AI agents, actions typically refer
to the external world (such as manipulating robots). But in language
agents, **reasoning occurs in the internal environment**, in the form of
"inner monologue." Its core process include

- **Reasoning by generating tokens is a new type of action**
  (vs. actions in external environments)
- Internal environment, where reasoning takes place in an inner
  monologue fashion
- Self-reflection is a 'meta' reasoning action (i.e., reasoning over the
  reasoning process), akin to metacognitive functions
- Reasoning is for better acting, by inferring environmental states,
  retrospection, etc.
- Percept and external action spaces are substantially expanded, thanks
  to using language for communication and multimodal perception

### Evolution of AI agents

To understand the uniqueness of language agents, we can compare the
evolution of AI agents:

(Zhou et al., 2024)

## Reference {#reference .unnumbered}

:::: {#refs .references .csl-bib-body .hanging-indent entry-spacing="0" line-spacing="2"}
::: {#ref-zhouSelfDiscoverLargeLanguage2024 .csl-entry}
Zhou, P., Pujara, J., Ren, X., Chen, X., Cheng, H.-T., Le, Q. V., Chi,
E. H., Zhou, D., Mishra, S., & Zheng, H. S. (2024). *Self-Discover:
Large Language Models Self-Compose Reasoning Structures*
(arXiv:2402.03620). arXiv. <https://doi.org/10.48550/arXiv.2402.03620>
:::
::::
