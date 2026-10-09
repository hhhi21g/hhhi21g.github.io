---
layout: post

title: Agent与轨迹调试基础01 - Reflexion

date: 2026-10-09 13:10:23 +0900

categories: [可解释性]
tags: [Agent与轨迹调试基础]
math: true
---

来源：Reflexion: Language Agents with Verbal Reinforcement Learning

不更新模型权重，让智能体通过自然语言总结失败原因，从而在后续尝试中表现得更好。

#### Reflexion

三个功能角色 + 一个记忆系统

<img src=../../../assets/img/可解释/image-20261009154323785.png width="400">

##### 1. Actor($$M_a$$)

根据当前环境状态选择动作：

$$
a_i \sim \pi_\theta(a_i\mid s_i)
$$

$$
\theta=\{M_a,\mathrm{mem}\}
$$

执行动作后，从环境获得观察$$o_i$$，连续执行后形成一条轨迹：$$
\tau_t=[a_0,o_0,\ldots,a_i,o_i]
$$

> 可以采用不同的推理方式：CoT / ReAct

##### 2. Evaluator($$M_e$$)

Actor完成一次尝试后，Evaluator对完整轨迹进行评分：$$
r_t=M_e(\tau_t)
$$

Evaluator Model取决于具体任务：如，推理任务，用 exact match 判断答案是否正确等。

##### 3. Self-reflection($$M_sr$$)

生成文字自我反思，并提供未来可用的反馈：
$$
sr_t=M_{sr}(\tau_t,r_t,\mathrm{mem})
$$

- 输入当前轨迹、reward信号、mem

##### 4. Memory

- 短期记忆：当前任务的轨迹；
- 长期记忆：从过去尝试中提炼出的反思文本，$$sr_t$$

****

#### 实验与结果

**ALFWorld：序列决策**

文字形式的交互环境，任务包括寻找物体、移动物体以及使用一个物体处理另一个物体。

Actor使用ReAct。相对基线取得22个百分点的提升。

反思主要解决：1. 发现长轨迹前面某个关键动作选错了；2. 记住已经搜索过的位置，避免无效的重复探索。

**HotPotQA：语言推理**

要求模型结合多份文档回答问题。

单纯的CoT或ReAct没有表现出稳定的逐轮学习；加入Reflexion后，智能体可以利用前几轮失败经验改进。且进一步进行下面3组实验，最后一种效果最好。说明提升不只是看到了上一轮的做法，而与“把错误提炼成指导性经验”有关。

```text
CoT
CoT + 保存上一轮轨迹
CoT + 保存轨迹 + 语言反思
```

**程序生成**

在HumanEval、MBPP、LeetcodeHardGym上测试Python和Rust代码生成。

Reflexion并非在所有设置上都更好，MBPP Python上表现为下降。(如果评价信号或自我反思不可靠，Reflexion不保证带来提升)

****
