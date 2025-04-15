---
title: "Memory, Reasoning, and Planning of Language Agents"
author: ["Mingrui Guo"]
date: "2025-03-30"
categories: ["AI"]
tags: ["LLM Agents", "RAG", "World Models", "Reasoning", "EN"]
draft: false
ShowToc: true
TocOpen: false
---

Language Agents have emerged as one of the most exciting research directions in AI over the past two years. This article explores three core components: **long-term memory via HippoRAG, reasoning capabilities with Grokked Transformers, and world modeling through WebDreamer**.

## Why Agents Again?

Russell & Norvig in “Artificial Intelligence: A Modern Approach” define an agent as “**anything that can perceive its environment through sensors and act upon that environment through actions.**”[@ArtificialIntelligenceModern]

{{< figure src="/images/20250401agent.png" title="Fig.1:Agent-Environment Interaction Framework" width="500px" class="align-center" >}}

Many people believe modern agents can be simply defined as **“LLM + external environment.”** This view suggests that language models themselves have limited functionality with only text input-output interfaces; once connected to an external environment, able to perceive environmental information and influence the environment, they become agents.

{{< figure src="/images/20250401modernagent.png" title="Fig.2:‘Modern’ agent = LLM + external environment?" width="700px" class="align-center" >}}

However, **this definition is oversimplified**. In reality, there are two main competing views in the community:

- **LLM-first view: We make an LLM into an agent**
  - Implications: scaffold on top of LLMs, prompting focused, heavy on engineering

- **Agent-first view: We integrate LLMs into AI agents so they can use language for reasoning and communication**
  - Implications: All the same challenges faced by previous AI agents (e.g., perception, reasoning, world models, planning) still remain, but we need to **re-examine them through the new lens of LLMs** and tackle new ones (e.g., **synthetic data, self-reflection, internalized search**)

### Characteristics of Modern Language Agents

Contemporary AI agents, with integrated LLMs, can **use language as a vehicle for reasoning and communication**

- **Instruction following, in-context learning, output customization**
- **Reasoning (for better acting): state inference, self-reflection, replanning, etc.**

Unlike traditional agents, reasoning in language agents is essentially a new form of “action”. In traditional AI agents, actions typically refer to the external world (such as manipulating robots). But in language agents, **reasoning occurs in the internal environment**, in the form of “inner monologue.” Its core process include

{{< figure src="/images/20250401reasoningagent.png" title="Fig.3:Inner Monologue and Reasoning in Language Agents" width="500px" class="align-center" >}}

- **Reasoning by generating tokens is a new type of action** (vs. actions in external environments)
- **Internal environment**, where reasoning takes place in an inner monologue fashion
- **Self-reflection** is a ‘meta’ reasoning action (i.e., reasoning over the reasoning process), akin to metacognitive functions
- **Reasoning is for better acting**, by inferring environmental states, retrospection, etc.
- **Percept and external action spaces** are substantially expanded, thanks to using language for communication and multimodal perception

### Evolution of AI agents

To understand the uniqueness of language agents, we can compare the evolution of AI agents:

| Feature        | Logical Agent | Neural Agent | Language Agent |
|---------------|--------------|--------------|---------------|
| **Expressiveness** | Low {{< rawhtml >}}<br>{{< /rawhtml >}} Bounded by the logical language | Medium {{< rawhtml >}}<br>{{< /rawhtml >}} Anything a (small-ish) NN can encode | High {{< rawhtml >}}<br>{{< /rawhtml >}} Almost anything, especially verbalizable parts of the world |
| **Reasoning** | Logical inferences {{< rawhtml >}}<br>{{< /rawhtml >}} Sound, explicit, rigid | Parametric inferences {{< rawhtml >}}<br>{{< /rawhtml >}} Stochastic, implicit, rigid | Language-based inferences {{< rawhtml >}}<br>{{< /rawhtml >}} Fuzzy, semi-explicit, flexible |
| **Adaptivity** | Low {{< rawhtml >}}<br>{{< /rawhtml >}} Bounded by knowledge curation | Medium {{< rawhtml >}}<br>{{< /rawhtml >}} Data-driven but sample inefficient | High {{< rawhtml >}}<br>{{< /rawhtml >}} Strong prior from LLMs + language use |

Early AI agents could only capture limited aspects of human intelligence, such as symbolic reasoning or unimodal perception.

Language agents show significant improvements over traditional logical agents and neural agents in expressiveness, reasoning flexibility, and adaptivity. Their language-driven reasoning abilities enable them to better handle uncertainties in complex environments and formulate more reasonable action strategies. 

### A Conceptual Framework for Language Agents

The capabilities of language agents can be divided into three different levels (as shown in the figure), core-competencies similar to human cognitive processes, form lower-level perception, memory, embodiment, to upper-level planning, reasoning, and world models. They simultaneously span issues of safety, evaluation, synthetic data, and efficiency.
 
{{< figure src="/images/20250401conceptual.png" title="Fig.4:Capability Hierarchy and Challenges of Language Agents" width="600px" class="align-center" >}}

That’s the introduction. This article will further explore three main aspects of language agents:

1. On long-term memory: HippoRAG
2. On reasoning: Grokked Transformers
3. On world models and planning: WebDreamer

## HippoRAG: Neurobiologically-Inspired Long-Term Memory for LLMs

Humans and animals continuously learn by gaining and strengthening knowledge. Nobel Prize winner Eric Kandel highlighted memory’s vital role, saying, “Memory is everything. Without it, we are nothing.” [@marksSearchMemoryEmergence2006] Memory relies on synaptic plasticity, where brain connections grow stronger to support learning. Sleep even helps solidify memories for the long term. 

Ideally, AI, especially large language models (LLMs), should learn and build knowledge over time too. But **LLMs struggle with this, often suffering from catastrophic forgetting, where they lose past knowledge—a major limitation**.

### Non-Parametric Memory

Researchers use non-parametric memory to help large language models (LLMs) learn continuously by storing new knowledge externally, as seen in Retrieval-Augmented Generation (RAG). This lets LLMs dynamically pull in outside information, acting as long-term memory. According to studies [@xieAdaptiveChameleonStubborn2024a], **LLMs adapt well to external data, even when it contradicts their own knowledge**.

{{< figure src="/images/20250401example.png" title="Fig.5: LLMs can effectively incorporate external evidence, even when it conflicts with their parametric memory, provided the evidence is coherent and persuasive" width="600px" class="align-center" >}}

Despite these benefits, current RAG implementations have limitations. Traditional RAG systems rely on vector embeddings for retrieval, which often struggle to capture complex associations.

### Long-term Memory in Humans

The hippocampal indexing theory [@teylerHippocampalMemoryIndexing1986] provides insights into how human memory achieves efficient recall. It suggests that:

- **Neocortex stores raw sensory data** (e.g., auditory and visual information).
- **Hippocampus acts as an index**, linking disparate memory fragments into a structured retrieval system.
- **Parahippocampal regions** facilitate connections between stored experiences, aiding in memory retrieval.

{{< figure src="/images/20250401hippo.webp" title="Fig.6: Hippocampus creates index for the memories to be stored in different part of neocortex [@sExploringHippoRAGNeurobiologically2024]" width="500px" class="align-center" >}}

Indexing procedure enables two fundamental faculties of human memory: 

- **Pattern separation**: process for differentiating memories (neocortex and parahippocampus) 
- **Pattern completion**: process for recovering complete memories from relevant associations (mostly hippocampus, specifically CA3)

### HippoRAG: Bringing Human-Like Memory to LLMs

HippoRAG [@gutierrezHippoRAGNeurobiologicallyInspired2025a] simulates this memory mechanism by building a similar structured index for RAG systems. Its workflow is divided into two phases:

**Offline Indexing Phase:**

- **Concept Extraction:** Uses an LLM to extract triplets (concepts, noun phrases, and their relationships) from text
- **Knowledge Graph Construction:** Builds a schema-less knowledge graph using the extracted concepts and relationships as nodes and edges
- **Dense Encoding:** Employs dense retrievers to consolidate similar or synonymous concepts

**Online Query Phase:**

- **Concept Identification:** Identifies key concepts from the query (such as "Stanford" and "Alzheimer's")
- **Similar Node Retrieval:** Finds nodes in the index similar to query concepts to serve as seed nodes
- **Graph Search:** Employs the Personalized PageRank algorithm to search the graph
Reranking: Reranks original passages based on concept weights

{{< figure src="/images/20250401HippoRAG.png" title="Fig.7: Detailed HippoRAG Methodology." width="700px" class="align-center" >}}

**The Personalized PageRank algorithm is a critical component of HippoRAG**. It performs a random walk starting from seed nodes, dispersing probability mass to neighboring nodes. Nodes close to seed nodes or at the intersection of multiple seed nodes naturally receive higher weights.

### Performance

HippoRAG delivers significant performance improvements across multiple benchmark datasets, particularly in **multi-hop QA tasks** and **iterative retrieval scenarios**.

**Multi-Hop QA Performance**

- **2WikiMultiHopQA**: Achieves an 11% improvement in R@2 and 20% in R@5, leveraging its entity-centric design for superior retrieval.
- **MuSiQue**: Shows a 3% improvement, demonstrating robustness across datasets.

{{< figure src="/images/20250402singlesetp.png" title="Fig.8: Single-step retrieval performance." width="500px" class="align-center" >}}

**Integration with Existing Methods**

HippoRAG complements existing iterative retrieval approaches:

- When integrated with IRCoT, R@5 performance improves further, highlighting the synergistic benefits of structured retrieval with multi-step reasoning.

{{< figure src="/images/20250402multistep.png" title="Fig.9: Multi-step retrieval performance." width="500px" class="align-center" >}}

### Memory in LLMs: Key Insights

- **Memory is fundamental to human learning**. Our sophisticated memory mechanisms enable **pattern recognition, association creation, and dynamic recall of relevant memories** beyond surface-level similarities.

- While LLMs struggle with long-term memory through parametric continual learning, **non-parametric memory (e.g., RAG) offers a promising solution**.

- Recent developments in RAG focus on **adding more structure to embeddings** (e.g., HippoRAG, GraphRAG) to enhance:
  - **Sensemaking**: the ability to interpret larger, more complex, or uncertain contexts.
  - **Associativity**: the capacity to draw multi-hop connections between disparate pieces of info.

Despite these advances, we are still far from developing a truly sophisticated memory system. Key challenges, such as handling episodic memory and spatiotemporal reasoning, remain unsolved. 

As we refine memory systems, **the next crucial step is to explore reasoning**, which builds upon memory to enable more advanced cognitive abilities.


## Grokking of Implicit Relations in Transformers

In the current landscape of LLM research, explicit reasoning methods such as Chain of Thought (CoT) have garnered significant attention. However, **implicit reasoning**—a more **fundamental capability—is essential for understanding the true nature of these models**. Let's explore the implicit reasoning mechanisms within the Transformer architecture.

### Implicit Reasoning in LMs

Implicit reasoning refers to a model's ability to **generate correct outputs without explicitly showing its reasoning steps**. This fundamental capability shapes how language models process and utilize information.

**Key Aspects of Implicit Reasoning:**

- Models **learn to predict next tokens without explicit reasoning chains** during pre-training
- Shapes how language models **develop structured knowledge representations**
- Recent insights into emergent reasoning capabilities:
  - Base models develop fundamental reasoning constructs during pre-training
  - Reinforcement learning optimizes selection of existing reasoning patterns

**Current Challenges:**

Research has identified several limitations in language models' implicit reasoning abilities:

- Compositional Reasoning
  - Models excel primarily at **single-step** reasoning (Yang et al. 2024)
  - The gap in compositional ability persists even as models scale up (Press et al. 2023)
- Comparative Analysis
  - Even advanced models like GPT-4 face difficulties with implicit attribute comparisons, despite having access to the relevant information (Zhu et al. 2023)


### Grokked Transformers are Implicit Reasoners

These limitations have fueled a narrative that autoregressive LLMs cannot truly reason. However, a recent paper (@wangGrokkedTransformersAre2024) challenges this view, suggesting **Transformers possess untapped reasoning potential worthy of deeper investigation**.

**Research questions**

This investigation explores two key questions:

1. Can Transformers learn to reason implicitly?
2. What factors control the acquisition of implicit reasoning?

**Experimental Design**

- **Model implementation:**   
  The study uses a standard GPT-2 style Transformer (8 layers, 768 hidden dimensions, 12 attention heads) with conventional AdamW optimization (learning rate 1e-1, batch size 512, weight decay 0.1, 2000 warm-up steps). 
- **Compositional Reasoning Framework:**   
  For testing implicit reasoning, the authors created synthetic knowledge graphs with \( |E| \) entities and 200 relation types, split into **ID** and **OOD** atomic facts. The key mechanism is two-hop composition: 
  $$
  (h, r₁, b) ∧ (b, r₂, t) ⇒ (h, r₁∘r₂, t)
  $$
  
  Example: from "Barack has-wife Michelle" and "Michelle born-in 1964," infer "Barack has-wife∘born-in 1964."

- **Inductive Learning Assessment:**  
  The study examines how models learn deduction reules from examples without explicit instruction, using two test scenarios:  
  1. **ID Generalization:** Novel combinations of familiar atomic facts used in other compositions.
  2. **ODD/Systematic Generalization:** Facts seen individually but never used in compositions-success here indicates true reasoning rather than memorization.

**Key Takeaways**

- **Takeaway #1: Transformers Learn to Reason Through 'Grokking'**

  Initially, models quickly reach 100% training accuracy (overfitting) while test accuracy remains low. However, after continuing training for approximately 20 times more steps beyond overfitting, test accuracy suddenly jumps to 100%. 

  This establishes a clear connection between grokking and the emergence of reasoning capabilities in transformers—**reasoning abilities aren't learned immediately but emerge after extended training periods.**

{{< figure src="/images/Screenshot 2025-04-15 at 9.51.55 AM.png" title="Fig.10: transformers can learn to reason implicitly, but this skill is only robustly acquired through grokking" width="700px" class="align-center" >}}

- **Takeaway #2: Generalization Varies Across Reasoning Types**

  - With compositional reasoning, models achieved perfect performance on in-distribution (ID) test examples but failed to generalize to out-of-distribution (OOD) scenarios. 
  - For comparative reasoning, however, models eventually reached 100% accuracy on both ID and OOD test sets. 

  This indicates that **the type of logical structure being learned significantly impacts how well the acquired reasoning generalizes.**

- **Takeaway #3: Data Distribution Matters More Than Data Size**

  While previous research suggested that grokking requires a critical threshold of data size, this study challenges that assumption. The researchers found that **data distribution—specifically the ratio between inferred facts and atomic facts (φ)—is far more important than total data quantity**. 

  When keeping this ratio fixed and increasing data size, generalization speed remained consistent. But when maintaining data size while increasing the φ ratio from 3.6 to 18, generalization speed increased dramatically. 
  
{{< figure src="/images/Screenshot 2025-04-15 at 9.57.49 AM.png" title="Fig.11: The speed of grokking on the in-distribution (ID) test performance (a) correlates with the ratio between inferred and atomic facts, and (b) is not influenced by the size of training data." width="700px" class="align-center" >}}

### Analyzing the changes during grokking




## world models and planning


## Reference

