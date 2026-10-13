---
layout: post

title: 精读02 - Internal Representations as Indicators of Hallucinations in Agent Tool Selection

date: 2026-09-28 21:49:23 +0900

categories: [可解释性]
tags: [最相关工作]
math: true
---

来源：Internal Representations as Indicators of Hallucinations in Agent Tool Selection

> 不等工具调用执行后再检查错误，而是在 LLM 生成工具调用的同一次前向传播中，利用内部隐藏状态提前判断该调用是否可能是幻觉。

数据表示如下：

$$
D=\{(q_i,c_i,f_i^*,a_i^*,y_i)\}_{i=1}^{N}
$$

- LLM根据(q_i, c_i)生成预测调用；

**标签自动生成：**

对于每个具有参考调用(f<sub>i</sub><sup>*</sup>, a<sub>i</sub><sup>\*</sup>)，

1. 从提示中移除参考工具调用；
2. 保留用户请求q_i和上下文c_i；
3. 让LLM自己预测工具调用；
4. 保存这次生成过程中**最后一层的隐藏状态**；
5. 将模型预测与参考调用比较；
6. 自动赋予正确或幻觉标签：调用或参数不正确为1幻觉，均正确为0。

**最后一层隐藏状态：**

$$
h_t^{(L)}\in\mathbb R^d
$$

- the final-layer hidden state at token position t，具体取3个：

  - 预测函数名第一个子词token的隐藏状态：$$t_{\mathrm{func}}$$
  
    - 参数区域：对全部参数token的位置集合对应的隐藏状态做平均池化；
  
  
    - 调用结束位置：可能汇总了整个调用信息
  
- 3个向量的组合：拼接+投影

$$
z_i=
\Pi\left(
h_{t_{\mathrm{func}}}^{(L)}
\;\Vert\;
\frac{1}{|T_{\mathrm{args}}|}
\sum_{t\in T_{\mathrm{args}}}h_t^{(L)}
\;\Vert\;
h_{t_{\mathrm{end}}}^{(L)}
\right)
\in\mathbb R^m
$$

- 产生幻觉概率：z<sub>i</sub>输入轻量前馈神经网络；给定分类阈值，交叉熵训练。

****

#### 实验与结果

| 模型         | 总体准确率 | 幻觉 Precision | 幻觉 Recall | 幻觉 F1 |
| ------------ | ---------- | -------------- | ----------- | ------- |
| GPT-OSS 20B  | **86%**    | **86%**        | 53%         | 0.66    |
| Llama 3.1 8B | 73%        | 71%            | 61%         | 0.66    |
| Qwen 7B      | 74%        | 34%            | **62%**     | 0.44    |

- GPT-OSS中，报错时比较可信，但会漏掉近一半的错误调用

| 聚合方式                     | 准确率    | AUC   |
| ---------------------------- | --------- | ----- |
| 最后一层全序列均值池化       | **74.6%** | 0.721 |
| 最后一个 token               | 74.4%     | 0.720 |
| 首 token＋末 token＋均值拼接 | 74.3%     | 0.719 |

- 复杂聚合没有明确优势，简单均值池化可能就是最值得采用的基线。

****
