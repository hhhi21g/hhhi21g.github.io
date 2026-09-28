---
layout: post

title: 精读03 - The Why Behind the Action

date: 2026-09-28 21:50:23 +0900

categories: [可解释性]
tags: [最相关工作]
---

来源：The Why Behind the Action: Unveiling Internal Drivers via Agentic Attribution

> 现有failure attribution本质上仅限于具有显示错误的场景，但忽略了：**agent的决策过程有问题、不可靠或不一致，但结果是正确或可接受的。**

**组件级归因：**

> where the main influence comes from，component表示一次action或observation

$$
\psi_i=\log p_{\pi_\theta}(a_T\mid\mathcal C_{\le i})
$$

$$
g_i=\psi_i-\psi_{i-1}
$$

$$
f_{\mathrm{comp}}(\mathcal C,a_T,\pi_\theta)
:=
\{g_i\}_{i=1}^{2T+1}
$$

- 给定当前组件的所有前缀C的条件下，在已有的模型的policy基础上，计算生成最终action的对数似然；
- 前后两个组件的对数似然做差值得到temporal gain，如果gain很大，则表明当前C使得最终action更有可能发生。

**句子级归因函数：**

> which precise sentences

$$
S(C_i)=\{s_{i,1},s_{i,2},\ldots,s_{i,N_i}\}
$$

$$
\hat{\mathcal C}_{\le i}
=
(C_1,\ldots,C_{i-1},S(\hat C_i))
$$

通过**消融**某个句子之后模型预测的变化来衡量因果影响：
$$
\operatorname{Drop}(s_{i,j})
=
\log p_{\pi_\theta}
(a_T\mid\hat{\mathcal C}_{\le i})
-
\log p_{\pi_\theta}
(a_T\mid\hat{\mathcal C}_{\le i}\setminus s_{i,j})
$$
只**保留**某个句子，看他是否能够独立支持目标行动：
$$
\operatorname{Hold}(s_{i,j})
=
\log p_{\pi_\theta}(a_T\mid s_{i,j})
-
\log p_{\pi_\theta}(a_T\mid\hat{\mathcal C}_{\le i})
$$

$$
f_{\mathrm{sent}}
\left(
\{C_1,\ldots,S(\hat C_i)\},
a_T,\pi_\theta
\right)
:=
\left\{
\operatorname{Drop}(s_{i,j})
+
\operatorname{Hold}(s_{i,j})
\right\}_{j=1}^{N_i}
$$

****

