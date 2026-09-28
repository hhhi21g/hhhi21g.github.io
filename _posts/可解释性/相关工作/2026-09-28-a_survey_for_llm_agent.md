---
layout: post

title: 相关工作01 - A Survey for LLM Agent Trajectory Analysis

date: 2026-09-28 21:25:23 +0900

categories: [可解释性]
tags: [相关工作]
---

来源：A Survey for LLM Agent Trajectory Analysis:  From Failure Attribution to Enhancement

- 传统结构化log难处理自然语言、推理型轨迹；
- 外部观察难以捕捉内部推理；
- code-level fix与system-level repair不匹配；
- 稳定性保障转向能力优化

```text
两大核心挑战
├─ 失败归因：定位哪个 Agent、哪一步以及为什么出错
└─ 系统增强与优化：利用诊断信息修复和改进系统

三项支撑基础
├─ 失败分类体系
├─ 轨迹监控与分析工具
└─ 数据集与基准
```

**Failure Taxonomy**

- failure出现在任务管道的哪里？where

  任务自然顺序：规划错误、任务执行问题、错误的相应生成

- agent的哪些核心能力失败了？what

  memory, reflection, planning, action, system-level

- who is responsible?  who

  规范问题、agent间不一致、任务验证

- 在给定的环境上下文，failure为什么出现？why

  exploration failure, exploitation failure, resource exhaustion

**Failure Attribution**

> 谁错了、哪一步错、为什么错？

- 基于模式分析的归因：从大量成功和失败轨迹中寻找反复出现的统计规律；
- 基于LLM推理的归因：将轨迹交给大模型，让模型像人工调试者一样进行语义分析；
- 基于模型微调的归因：训练专门负责归因的tracer model；
- 基于动态运行时的归因：根据失败轨迹提出根因假设，修改被怀疑的消息等，重新执行，检查。

**System Enhancement and Optimizition**

> 如何利用执行轨迹中的成功经验、失败信息和过程信号，真正提高智能体系统的功能、效率与适应能力？

```text
① 改系统外部结构
   环境、工作流、智能体配置(重新设计团队如何组织)
            ↓
② 改智能体内部能力
   提示词、策略、模型、技能(提升成员自身能力)
            ↓
③ 在运行过程中监督
   压缩上下文、检测异常、实时干预(在工作时实时提醒和纠偏)
```

- 结构与工作流优化：环境层 / 工作流层
- Agent内部优化：Prompt、Policy/instruction、Model、Skill-level
- 运行时与监督优化：Input-space(减少上下文噪声)、Behavior-space(注入外部转向信号)

**Trajectory Monitoring and Analysis Tools**

> 如何把冗长、异构的智能体轨迹转化为开发者能够检查、分析和操作的信息？

```text
4.4.1 系统级监控与被动诊断
       看见并分析发生了什么
                 ↓
4.4.2 交互分析与主动调试
       允许开发者探索和修改轨迹
                 ↓
4.4.3 实践型可观测性平台
       将追踪、评测和调试工程化
```

- 系统级监控与被动诊断工具：外部非侵入、自动化诊断、人的可视化；
- 交互分析和主动调试工具：允许开发者探索和干预轨迹；
- Practitioner-oriented Observability Platform

**Datasets and Benchmarks**

****