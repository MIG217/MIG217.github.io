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

Language Agents have emerged as one of the most exciting research directions in AI over the past two years. This article explores three core components: **long-term memory via HippoRAG, reasoning capabilities with Grokked Transformers, and world modeling through WebDreamer**—all essential for building effective language-driven AI systems.

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


## Grokking of Implicit Relations in Transformers



## Reference
