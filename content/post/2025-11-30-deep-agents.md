---
title: "Deep Agents From LangGraph"
author: ["Mingrui Guo"]
date: "2025-11-30"
categories: ["AI"]
tags: ["LLM Agents"]
draft: false
ShowToc: true
TocOpen: false
math: true
---

在过去的一年里，AI Agent 的演进出现了两个非常重要的趋势：

1. **智能体正在变得更通用（Generalist）**：可以承担越来越多类型的任务；
2. **智能体的任务时长变得更长（Long-horizon）**：能够连续执行几十甚至上百个步骤的复杂任务。

根据 METR 的基准测试，AI 能自动完成的人类任务等效时长大约 每 7 个月翻倍。这意味着智能体从“短对话助手”，发展为“能够连续运行数百甚至上千步的自主系统”。

{{< figure src="static/images/length-of-tasks-log.png" width="700px" class="align-center" >}}

与此同时，通用智能体数量激增，如 Manus 和 Claude Code 等系统正在承担远不止“写代码”或“回答问题”的任务。它们能够**组织研究流程、规划任务、调用大量工具，并产出复杂成果**。但随着任务时长与任务复杂度提升，工程上的挑战也随之而来：

- **Manus**：典型任务需要调用约 50 次工具
- **Anthropic Production Agents**：实际生产系统常常会进行 数百轮对话与推理

这些长周期、多步骤、工具密集型的系统，被称为 **Agentic System**。尽管不同项目的具体实现差别较大，但它们普遍遵循以下四个核心原则：

1. **使用 Planning 保证任务方向正确**

在超长任务中，如果缺乏规划，模型非常容易偏航。因此 Agentic System 普遍采用明确的任务计划来做“航向控制”。

常见做法包括：

- **Manus**：使用 `todo.md` 保存任务清单，并在执行过程中不断读写更新；
- **Claude Code**：要求用户先批准计划，再执行工具与操作；
- **Gemini Deep Research**：在执行前强制生成计划、并请求用户确认；
- **Anthropic Multi-Agent Researcher**：将 research plan 写入文件系统，在关键流程中重新读取，确保最终报告“遵循原计划”；

2. **使用 Filesystem 进行 Offload Context**

随着工具调用次数、搜索结果、观察内容不断增多，把所有内容保存在消息历史里会快速耗尽上下文窗口。

因此，可以采用“外部化记忆”机制：

- 将庞大中间结果（如搜索原始结果）保存到磁盘
- 在上下文中只放简短摘要，节省 Tokens
- 在需要时再从文件读取完整内容
- 长期记忆可以独立保存，不随对话窗口消失

Example:

- **Anthropic Multi-Agent Researcher**：将研究计划写到文件里 → 调研完成后再读回来 → 保证报告结构与原计划一致；
- **Manus**：将 `todo.md` 持久化，多次更新、反复读取

文件系统就是 Agentic System 的 外部化记忆（Externalized Memory）。


3. **使用Sub-agents 隔离上下文与任务**

Agentic System 另一个关键能力是：任务拆分与子智能体协同。原因很简单：每个子智能体都有独立的上下文窗口。

- 主智能体无需承载所有推理上下文
- 子智能体可以专注于独立的子任务
- 可以并行化任务（如资料收集、数据比对）

Example:

- **Claude Code**：代码分析和修复任务由不同子 agent 完成
- **Anthropic Multi-Agent Researcher**：子 agent 专门负责检索、整理、验证
- **Manus**：子 agent 负责某些并行可拆解的子任务
- **Open Deep Research**：子智能体只负责“收集信息”，最终报告由主 agent 统一撰写，避免结构冲突

Cognition / Walden Yan 提到过“多智能体隐式决策冲突的问题”，如果多个子智能体分别撰写部分内容，那么最终合并会出现**风格不统一、推理冲突、结构不一致**。

因此子智能体应该只做可并行、可聚合、无冲突的任务，例如：信息收集、文献爬取、数据提取、独立判断或评分，**不建议独立撰写报告章节（容易冲突）**。

4. **使用精心设计的 Prompt Engineering**

Agentic System 的 System Prompt 通常非常巨大、精细、迭代多次。

Example: 

- **Claude Code**几十次迭代
- **Manus**长而复杂
- **Open-Deep-Research**结构明确、角色分工清晰

尽管 Agentic System 在架构上看似只是：“LLM + 工具调用 + 循环”，但背后依赖的 Prompt 复杂度非常高，用于确保：

- 工具调用顺序正确
- 输出格式一致
- 避免幻觉
- 长周期行为稳定
- Sub-agent 的工作可被主 agent 正确整合

因此 Prompt 工程是 Agentic System 的“隐形关键工程”。

**Summery: Agentic System 的方法论矩阵**


| Agent                | Filesystem           | Planning         | Sub-Agents             | Prompting           |
| -------------------- | -------------------- | ---------------- | ---------------------- | ------------------- |
| Manus                | ✔ 使用文件系统             | ✔ todo.md & 重读计划 | ✔ 多个并行子 agent          | ✔ 大量 prompt 迭代      |
| Anthropic Researcher | ✔ 保存计划与资料            | ✔ 写计划并重复读取       | ✔ delegated researcher | –                   |
| Open Deep Research   | ✔ 使用 LangGraph State | ✔ think tool     | ✔ 信息收集子 agent          | ✔ 自定义 System Prompt |
| Claude Code          | ✔ 使用文件系统存代码与补丁       | ✔ plan mode      | ✔ 任务拆分                 | ✔ 强大的 prompt 体系     |

这些系统尽管背景不同，却共享了深度智能体的“四大原则”：

- Planning：保持方向、避免偏航
- FileSystem：外部化记忆、节省上下文窗口
- Sub-Agent：上下文隔离、任务解耦
- Prompt Engineering：确保整个系统稳定运行

## Planning

随着智能体承担的任务越来越复杂，例如多文件代码修改、多轮研究任务、产品设计等，仅依靠“下一步该做什么”的即时推理已经完全不够。Agentic System 都引入了 Planning 工具：

- **Claude Code**: 使用 `TodoWrite` 工具生成带审批的任务计划，允许用户确认再执行。
- **Manus**: 自动生成并持续更新 `todo.md` 文件，贯穿整个任务流程。

**智能体需要一个结构化、可追踪、可更新的任务计划（TODO 列表），并将其保存到“状态”（State）中。**

### 构建 State

Planning 需要在State中维护以下信息:

- `messages`: 对话历史

- `todos`: 任务规划和进度追踪

- `files`: 上下文信息存储

因此，我们扩展默认的 `AgentState`，创建 `DeepAgentState`：

1. **定义 TODO 数据结构** 

```py
from typing import Literal
from typing_extensions import TypedDict

class Todo(TypedDict):
    """结构化的任务项,用于追踪复杂工作流的进度
    
    属性:
        content: 简短、具体的任务描述
        status: 当前状态 - pending(待处理)、in_progress(进行中)或 completed(已完成)
    """
    content: str
    status: Literal["pending", "in_progress", "completed"]
```

2. **文件系统Reducer**

为了支持增量式的文件更新,我们需要一个 reducer function:

```Py
def file_reducer(left, right):
    """合并两个文件字典,右侧优先
    
    作为 files 字段的归约函数,允许对虚拟文件系统进行增量更新
    
    参数:
        left: 左侧字典(现有文件)
        right: 右侧字典(新增/更新的文件)
    
    返回:
        合并后的字典,右侧值覆盖左侧值
    """
    if left is None:
        return right
    elif right is None:
        return left
    else:
        return {**left, **right}
```

3. **扩展Agent的State**

通过继承 `AgentState`，我们保留 `messages`，同时加入 `todos` 和 `files`。这样，整个 Agent 就在一个统一的状态树中管理：对话内容、任务计划、虚拟文件。这极大增强了 agent 的能力与可控性。

```Py
from typing import Annotated, NotRequired
from langgraph.prebuilt.chat_agent_executor import AgentState

class DeepAgentState(AgentState):
    """扩展的代理状态,包含任务追踪和虚拟文件系统
    
    继承自 LangGraph 的 AgentState 并添加:
    - todos: Todo 项列表,用于任务规划和进度追踪
    - files: 虚拟文件系统,存储为文件名到内容的字典映射
    """
    todos: NotRequired[list[Todo]]
    files: Annotated[NotRequired[dict[str, str]], file_reducer]
```

### Planning 工具设计

为了让智能体能够管理自己的任务计划，我们需要设计两个基本工具：

- `write_todos`：写入或更新 TODO 列表

- `read_todos`：从 state 中读取当前 TODO，辅助决策

1. **`write_todos`**

这个工具会将 LLM 生成的 Todo 列表直接写入 state：

```Py
@tool(description=WRITE_TODOS_DESCRIPTION, parse_docstring=True)
def write_todos(todos, tool_call_id):
    return Command(
        update={
            "todos": todos,
            "messages": [
                ToolMessage(f"Updated todo list to {todos}", tool_call_id=tool_call_id)
            ],
        }
    )
```

工具返回 `Command`
  - 更新了 `state.todos`
  - 写入了一条 `ToolMessage`

因此，执行 `write_todos` 后，智能体的状态自动包含新的任务计划。

2. **`read_todos`**

```Py
@tool(parse_docstring=True)
def read_todos(state, tool_call_id):
    todos = state.get("todos", [])
    ...
```
与 `write_todos` 不同，`read_todos` 返回的是字符串，不是 `Command`。

但 `create_react_agent` 会自动将字符串包装成 `ToolMessage` 并更新 `messages` 字段。


### 价值与优势

1. TODO 是智能体的“显式认知步骤（Explicit Cognition）”: 帮助模型在长任务中拆解、记忆、执行。

2. TODO 是智能体的“规划工具”, 支持如下能力：
- 多步骤执行
- 子任务跟踪
- 状态回溯
- 推理清晰化
- 避免遗忘中间步骤

3. TODO 是智能体的“可控接口”: 开发者或用户可以“审阅”智能体的计划，从而让智能体更可控、更稳定。

## FileSystem

在长时间运行的代理任务中，代理可能需要执行数十次工具调用。在这个过程中，重要的上下文信息可能会丢失或被遗忘。通过将关键信息保存到文件中，我们可以：

- 持久化重要信息
- 在多次工具调用后重新获取上下文
- 更好地引导代理完成复杂任务

### 虚拟文件系统设计

我们将定义三个工具，并为它们提供清晰的描述，以便 LLM 能够理解和使用它们：

`ls`：列出虚拟文件系统（即字典的 所有键）中的所有文件。

`read_file`：根据指定的文件路径读取文件内容。

`write_file`：将指定的内容写入指定的文件路径。

### 核心实现

1. **列出文件：`ls`**

- 使用 Injected State 访问图状态
- 从状态中获取 files 字典并返回所有键（文件路径）的列表

```Py
@tool(description=LS_DESCRIPTION)
def ls(state: Annotated[DeepAgentState, InjectedState]) -> list[str]:
    return list(state.get("files", {}).keys())
```

2. **读取文件：`read_file`**

这个工具支持：
- 根据路径读取内容
- 根据 `offset` 和 `limit` 分页读取
- 自动编号（方便调试）
- 长行截断

```Py
@tool(description=READ_FILE_DESCRIPTION, parse_docstring=True)
def read_file(file_path, state, offset=0, limit=2000) -> str:
    ...
```
读取流程：查看文件是否存在 -> 将内容按行切分 -> 按 `offset`、`limit` 截取 -> 返回带行号的结果。

效果：
```vbnet
     1   The MCP (Model Context Protocol) is...
     2   It allows systems to...
```

3. **写入文件：`write_file`**

写文件需要更新 `state`，因此使用 `Command`：

```Py
@tool(description=WRITE_FILE_DESCRIPTION, parse_docstring=True)
def write_file(file_path, content, state, tool_call_id) -> Command:
    files = state.get("files", {})
    files[file_path] = content
    return Command(
        update={
            "files": files,
            "messages": [
                ToolMessage(f"Updated file {file_path}", tool_call_id=tool_call_id)
            ],
        }
    )
```

实现更新 `state.files`，写入 `ToolMessage` 到 `state.messages`

### Summary

虚拟文件系统是构建深度代理的重要基础设施。虽然在简单示例中可能显得不必要,但在处理复杂的多步骤任务时，它能够显著提升代理的可靠性和性能。

✓ 防止 LLM 遗忘重要信息

✓ 随时加载任意步骤的上下文

✓ 支持多文件代码生成

✓ 支持复杂研究任务（多次搜索、多次整合）

✓ 支持“草稿—修改—再草稿”的循环

✓ 支持在不同 agent 之间共享状态

这套文件操作工具（ls、read_file、write_file）将是深度代理抽象的核心功能之一。通过理解其底层实现原理——基于 LangGraph 状态的模拟文件系统——你可以根据自己的需求进行定制和扩展。




