---
layout: post

title: 相关工作02 - From Failed Trajectories to Reliable LLM Agents

date: 2026-09-28 21:28:23 +0900

categories: [可解释性]
tags: [相关工作]
---

来源：From Failed Trajectories to Reliable LLM Agents:  Diagnosing and Repairing Harness Flaws

> 失败信息散落在模型推理、工具调用、环境返回、状态变化和最终提交等多个步骤中；运行时失败难以对应到静态实现；修好一个问题可能引入新的回归。

**A：建模运行轨迹并对齐Harness实现**

> 它要解决两个问题：
>
> 1. 失败证据散落在多个运行步骤中，怎样把它们组织起来？
> 2. 即使知道某个运行步骤有问题，怎样找到对应的提示词、工具定义或控制器代码？

- TraceStep：把轨迹切分为可诊断的步骤

  ```text
  TraceStep:前三项是从轨迹中保留的基本字段，后三项是Trace Abstraction Agent推导标注
  ├── ID
  ├── Request message
  ├── Response message
  ├── Role
  ├── Execution status
  └── Artifact/state effect
  ```

- Data-flow Alignment

  > 当前TraceStep中的信息来自哪个早期TraceStep?他在传播过程中被复制、总结、改写还是遗漏了？

  ```
  给定当前步骤 St
        ↓
  从 St−1 开始向前搜索
        ↓
  比较早期步骤的 request/response
  与当前步骤的 request
        ↓
  判断是否存在显式或语义复用
        ↓
  记录 source span、target span 和 reuse relation
  ```

- Control-flow Alignment：为什么执行完前一步后，系统会进入后一步？

  ```text
  S5
  ↓ 在条件 C 下触发控制逻辑 L
  S6
  ```

​	为建立control-flow link，联合检查四类证据：当前TraceStep；当前步骤前后的时间邻域；已经构建的数	据流关系；相关harness实现。

- Implementation Anchors：把运行证据定位到Harness实现

  > 行为具体由哪段提示词、代码、配置或验证逻辑产生？应该在哪里修复？

  Agent联合分析：执行轨迹 + Harness artifacts，以建立锚点，例如：

  ```text
  轨迹中出现 complete_task
          ↓
  在 Harness 中搜索对应调用和控制逻辑
          ↓
  找到 execute→ task_completed
          ↓
  确认它控制 S5 → S6 的转换
  ```

​	如果一种行为横跨多个组件，Agent会沿已经建立的data-flow和control-flow links继续追踪。

- 责任映射层：Trace Abstraction Agent会进一步把每个TraceStep周围的证据映射到ETCLOVG七个harness层。一个TraceStep可能同时涉及多层。

**B. Harness Flaw诊断**

- Failure Attribution：单条轨迹的失败归因

  1. 确认失败表现是什么，检查最终步骤的A中的信息；
  2. 沿证据边向前回溯：沿data/control-flow，从final step反向回溯到早期，并给出一个较宽的候选范围；
  3. 判断每个候选步扮演了什么错误角色？首次形成、传播、没有检查要求等，选择最终的责任step；
  4. Layer Assignment：选择最终responsible steps，使用A中的责任映射层，将失败归入相应层。

- 从Diagnosis Records聚合为Flaw Record

  > 如果系统只修复单条轨迹，很容易出现过拟合，可能让当前样例通过，却不能解决更普遍的harness机制问题。

​	Diagnosis Agent会把具有相似根因的diagnosis records聚合起来，形成一个面向修复的flaw record.

```text
Diagnosis 1：
工具返回 success，但目标记录没有生成

Diagnosis 2：
工具返回 success，但目标文件没有写入

Diagnosis 3：
工具返回 success，但应用状态没有更新
                    ↓
共同根因
完成逻辑只检查执行状态，
不检查预期 artifact/state effect
                    ↓
Flaw Record：
completion accepted without artifact/state effect
```

**C. Scoped Repair**：repair agent

```text
Flaw Record
     ↓
映射到 Repair Operators
     ↓
确定 Primary / Auxiliary Operators
     ↓
生成 Repair Specification
     ↓
在规范约束下生成 Candidate Patch	
```

****