---
layout: post

title: Agent与轨迹调试基础02 - ToolSandBox

date: 2026-09-29 02:29:23 +0900

categories: [可解释性]
tags: [Agent与轨迹调试基础]
math: true
---

来源：TOOLSANDBOX: A Stateful, Conversational, Interactive Evaluation  Benchmark for LLM Tool Use Capabilities

#### 背景

**Stateful：世界会发生变化**

> 现有方法使用静态环境，或没有研究状态的影响

工具不是孤立的函数，而是与某个World State相联系。

- 改变状态：例如打开网络；
- 依赖状态：例如网络关闭时不能搜索附近餐馆。

这种依赖通常没有写进用户请求，所以智能体必须根据尝试和工具报错自行形成计划。

**Conversational：信息可能要通过对话获得**

> 现有方法单轮，或off-policy

现实请求不一定是完整、明确的。合格的智能体应该追问，测评环境必须允许对话根据智能体的行动继续发展。

**Interactive：执行过程中会出现意外**

真实执行可能出现：模型调用了错误工具、参数格式错误、模型发现错误后重新规划等。因此不能仅比较最终答案，也不能要求智能体严格复现一条预定路径。

****

- Execution Context：世界状态
- Tools：python函数
- Message Bus：消息调度机制，传递内容，也决定下一步轮到谁行动
- 测评标准：Milestones(达到了应该实现的关键状态)，Minefields(触发了不应发生的行为)

测试从User向Agent发送消息开始；Agent决定下一步行动(向用户提问 or 请求执行工具)；执行环境真正运行工具；多轮交互继续；用户结束会话(调用特殊工具end_conversation)。

#### Stateful

有状态工具：能够执行检查/依赖/修改世界状态至少一种操作的工具，占ToolSandbox工具箱的44%，工具之间形成隐式依赖。

| 世界状态         | 依赖它的操作                                         |
| ---------------- | ---------------------------------------------------- |
| Cellular service | `send_message` 等通信工具要求其开启                  |
| Wi-Fi            | `search_stock` 等网络服务要求其开启                  |
| Location service | 获取或使用当前位置的工具要求其开启                   |
| Low battery mode | 开启蜂窝网络、Wi-Fi 和定位服务时，要求低电量模式关闭 |

#### Conversational

> 如何用一个LLM模拟用户，与被测智能体进行可信的在线多轮对话？

使用GPT-4o驱动的用户模拟器，支持on-policy conversational roll-out。

用户模拟器代表一个希望通过Agent完成任务的人，他会：

- 提出初始要求；
- 回答Agent的澄清问题；
- 判断任务是否已经/无法完成；
- 必要时终止对话(唯一能使用的工具是end_conversation)

> 相关用户模拟研究，把完整的用户目标写入模拟器的prompt，会导致**两类问题:**
>
> - Hallucination：模拟器可能不知道某些问题的正确答案，却自行编造；
> - Instruction-following error：模拟器可能被被测Agent带偏。

**两个添加的组件：**

1. Knowledge Boundary

   明确告诉模拟器，知道/不知道哪些信息

2. Demonstration

   给模拟器提供少量示例对话，即few-shot demonstrations

#### Interactive

> 解决测评难题：当智能体可以自由对话、调用工具、犯错并纠正，而且一个任务存在多条正确路径时，怎样自动且可解释的评分？

**Milestone**

完成用户目标所必需的事件。使用DAG(有向无环图)：既能规定必要顺序，又能保留不受约束的操作顺序。

```text
# 例
M1 ─┐
    ├→ M3
M2 ─┘

允许：M1 → M2 → M3 → M4
允许：M2 → M1 → M3 → M4
不允许：M3 → M1 → M2 → M4
```

每个Milestone配有一个相似度函数，用来计算某个turn与该Milestone的匹配程度。不同Milestone可以使用不同的匹配方法：工具调用可使用AST结构匹配、电话号码等字段可以精确匹配等。

系统需要把Milestones映射到实际轨迹中的turns，系统在所有合法映射中，寻找平均相似度最高的一种，得到正向milestone分数 $$score_{M^{+}}$$。

> **直观理解：**能否从这条自由生成的轨迹中，找出一组满足必要时序关系的关键事件？这些事件与预期Milestones匹配的有多好？

**Minefield**

绝不能发生的事件。主要用于信息不足或任务不可完成的场景，也可形成一个DAG，并得到其匹配的分数$$score_{M^{-}}$$

**最终得分：**
$$
\mathrm{score}
=
\mathrm{score}_{M^+}
\times
\mathbf{1}\!\left(\mathrm{score}_{M^-}=0\right)
$$

- 未触发Minefield：保留Milestone得分；否则整条轨迹得分归零。

****

#### 测试与结果

测试场景分为：

- 初始世界状态；
- 初始消息；
- 可供Agent使用的工具；
- 由Milestones和Minefields组成的评测标准

1032个测试场景、34个工具、11个工具领域

场景类别：

- Single / Multiple Tool Call
- Single / Multiple User Turn：用户第一条消息是否包含全部必要信息
- State Dependency：要求某个工具执行前，世界必须处于待定状态；而该状态可以通过其他工具秀嘎
- Canonicalization：把用户的自然语言表达转换成API所要求的标准形式
- Insufficient Information：模型是否知道当前信息不足以完成任务
- Tool Augmentation：工具鲁棒性测试

****