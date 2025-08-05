---
layout: post

title: Competition01_OTTO-Multi-Objective Recommender System

date: 2025-07-01 15:45:23 +0900

categories: [推荐系统]
tags: [竞赛]
---


竞赛地址：https://www.kaggle.com/competitions/otto-recommender-system/data?select=train.jsonl

作为第一个推荐系统方面的kaggle竞赛，这里将从零开始，详细记录每一步的进行。

### 竞赛介绍

OTTO是德国最大的在线商店，任务为：根据用户的一次访问(session)中行为序列(点击、加购、购买)，预测用户未来可能对哪些商品感兴趣(最多20个)。对每个session预测三类行为：

- clicks: 可能会点击的商品；
- carts: 可能会加购的商品；
- orders: 可能会下单的商品。

****

### 数据集

**训练数据：**

train.jsonl - 包含完整的会话数据

- session - 唯一的会话ID
- events - 会话中事件的时间顺序
  - aid - 商品ID
  - ts - 事件的时间戳，毫秒级
  - type - 事件类型，clicks/carts/orders

```python
# 查看前三行数据
with open("dataSet\\train.jsonl","r",encoding="utf-8") as f:
    for i, line in enumerate(f):
        data = json.loads(line)
        print(data)
        if i >= 2:
            break
```

<p>
    <img src="https://hhhi21g.github.io/assets/img/SR/com01/c1.png" alt="alt text" />
</p>

**测试数据：**

test.jsonl - 包含待预测的测试会话数据，不包含最终行为

需要继续预测用户买了什么，继续浏览了什么

- type - 通常不包含orders, 需要预测

  > 其余同train

<p>
    <img src="https://hhhi21g.github.io/assets/img/SR/com01/c2.png" alt="alt text"  />
</p>

**得分：**

根据Recall@20评估预测结果，即*命中数/实际目标数*

> 最多可以预测20个，在以召回率为评估的情况下，提交每行预测刚好20个是必要且推荐的做法。

对三个召回值进行加权平均：

score = 0.10 · R<sub>clicks</sub> + 0.30 · R<sub>carts</sub> + 0.60 · R<sub>orders</sub>

****

### baseline

纯计数，**统计出train.jsonl的event中出现次数最多的20个商品**，作为预测时每个session的商品，即给每个用户不加个性化的推荐热门商品。轮流将事件置为clicks, carts, orders，构建submission.csv。

**返回出现次数最多的前20个商品**：

```python
top_clicks = Counter()

# 计算总行数
with open('dataSet\\train.jsonl', 'r', encoding='utf-8') as f:
    total_lines = sum(1 for _ in f)

with open('dataSet\\train.jsonl', 'r', encoding='utf-8') as f:
    for line in tqdm(f,total=total_lines,desc="统计热门商品"):
        session = json.loads(line)
        for event in session['events']:
            top_clicks[event['aid']] += 1

# 返回出现次数最多的前20个商品,返回的是一个列表(商品ID,出现次数)
top_items = [str(aid) for aid, _ in top_clicks.most_common(20)]
```

**轮流给商品设置事件，生成submission.csv:**

```python
submission_rows = []

# 获取 test 文件总行数
with open('dataSet\\test.jsonl', 'r', encoding='utf-8') as f:
    total_test = sum(1 for _ in f)

with open('dataSet\\test.jsonl', 'r', encoding='utf-8') as f:
    for line in tqdm(f,total = total_test,desc="构建submission.csv"):
        session = json.loads(line)
        sid = session['session']
        for t in ['clicks', 'carts', 'orders']:
            submission_rows.append({
                'session_type': f"{sid}_{t}",
                'labels': ' '.join(top_items)
            })

submission = pd.DataFrame(submission_rows)
submission.to_csv('output\\submission.csv', index=False)
```

**得分：0.00664(private)**

****

### 获得共现矩阵

**1. 以嵌套字典保存共现商品**

```python
# 嵌套字典: 字典里的值还是一个字典
# 外层字典: co_visitation[a], 商品a
# 内层字典: co_visitation[a][b], 表示商品a和商品b的共现次数
# defaultdict: 自动初始化
co_visitation = defaultdict(lambda: defaultdict(int))
```

**2. 构建商品共现字典**，为加快计算，只考虑相邻点击。

事实上，如果两个商品出现在同一个session ,但他们相隔很远，就可能关系不大；并且如果将所有商品两两共现，就会出现很多毫无关联的商品对，导致共现矩阵庞大且稀疏；限定相邻点击，大幅减少计算量。因此，相邻点击可视为一种精简而精准的共现建模方法。

```python
# 构建商品共现字典
with open(train_path, 'r', encoding='utf-8') as f:
    for line in tqdm(f, total=total_lines, desc='构建共现矩阵'):
        session = json.loads(line)
        aids = [event['aid'] for event in session['events'] if event['type'] == 'clicks']
        if len(aids) < 2:  # 点击数太少则跳过
            continue

        # 只考虑相邻点击
        for i in range(len(aids) - 1):
            a, b = aids[i], aids[i + 1]
            if a != b:
                co_visitation[b][a] += 1
                co_visitation[a][b] += 1
```

**3. 每个商品只保留top 50共现商品，并将结果保留**

```python
# 每个商品只保留 top 50 共现商品
for a in co_visitation:
    co_visitation[a] = dict(sorted(co_visitation[a].items(), key=lambda x: -x[1])[:50])

# 保存共现矩阵(以二进制写入模式打开文件,适用于pickle序列化)
with open(co_vis_pkl,'wb') as f:
    pickle.dump(dict(co_visitation), f)

print(f'共现矩阵已保存到 {co_vis_pkl}')
```

**4. 预测**

- 针对测试session, 仅考虑最近的5个点击事件，召回相关商品作为候选：捕捉用户的最新意图，限制候选集合大小；

- 获得累积加分(即共现次数)最多的20个商品，作为最终预测结果。

  > 这里暂时将三个事件的20个交互商品都预测为相同的。

```python
with open(test_path, 'r', encoding='utf-8') as f:
    for line in tqdm(f, total=total_test, desc='为测试集生成推荐'):
        session = json.loads(line)
        sid = session['session']

        # 获得最近的点击事件
        clicked = [event['aid'] for event in session['events'] if event['type'] == 'clicks']
        recent_clicks = clicked[-5:]

        # 推荐商品池
        candidates = defaultdict(int)
        for aid in recent_clicks:
            related = co_visitation.get(aid, {})
            for b, score in related.items():
                candidates[b] += score  # 累加得分

        top_items = [str(aid) for aid, _ in sorted(candidates.items(), key=lambda x: -x[1])[:20]]

        for t in ['clicks', 'carts', 'orders']:
            submission_rows.append({
                'session_type': f"{sid}_{t}",
                'labels': ' '.join(top_items)
            })

submission = pd.DataFrame(submission_rows)
submission.to_csv('output\\submission.csv', index=False)
```

**5. 得分: 0.18326**

****
