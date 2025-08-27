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

> 01_baseline.py

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

### 获得共现矩阵(仅根据click)

> 02_single-co-visitation.py

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

### 获得共现矩阵(三种事件)

> 03_multi-co-visitation.py

基于02进行修改

**1. 建立三个嵌套字典以保存共现商品**

```python
co_visitation_clicks = defaultdict(lambda: defaultdict(int))
co_visitation_cart = defaultdict(lambda: defaultdict(int))
co_visitation_order = defaultdict(lambda: defaultdict(int))
```

**2. 进行热门统计，以在候选共现商品不足时进行填充**

```python
# 热门统计
global_popular = Counter()
```

```python
with open(train_path, 'r', encoding='utf-8') as f:
    for line in tqdm(f, total=total_lines, desc='构建共现矩阵'):
        session = json.loads(line)
        events = session['events']

        # 更新全局热门
        for e in events:
            global_popular[e['aid']] += 1

        aids = [event['aid'] for event in session['events'] if event['type'] == 'clicks']
        if len(aids) < 2:  # 点击数太少则跳过
            continue
```

**3. 只考虑相邻事件，建立：click共现，click->cart，click->order三个共现**

```python
        for i in range(len(aids) - 1):
            a, t1 = events[i]['aid'], events[i]['type']
            b, t2 = events[i + 1]['aid'], events[i + 1]['type']
            if a == b:
                continue

            # clicks共现
            if t1 == 'clicks' and t2 == 'clicks':
                co_visitation_clicks[a][b] += 1
                co_visitation_clicks[b][a] += 1

            # click -> cart
            if t1 == 'clicks' and t2 == 'carts':
                co_visitation_order[a][b] += 1

            # click -> order
            if t1 == 'clicks' and t2 == 'orders':
                co_visitation_order[a][b] += 1
```

**4. 每个字典保留top 50**，保留共现矩阵

```python
# 每个商品只保留 top 50 共现商品
def trim_topk(matrix, k=50):
    for a in matrix:
        matrix[a] = dict(sorted(matrix[a].items(), key=lambda x: -x[1])[:k])
    return matrix

co_visitation_clicks = trim_topk(co_visitation_clicks)
co_visitation_cart = trim_topk(co_visitation_cart)
co_visitation_order = trim_topk(co_visitation_order)
```

```python
with open(co_vis_pkl, 'wb') as f:
    pickle.dump({
        "clicks": dict(co_visitation_clicks),
        "cart": dict(co_visitation_cart),
        "order": dict(co_visitation_order)
    }, f)
```

**5. 进行预测：将三个事件赋予不同权重，若不足20以热门top50填充**

```python
# 热门top50
global_popular_top = [aid for aid, _ in global_popular.most_common(50)]

# 权重
weights = {"clicks": 1.0, "cart": 1.5, "order": 2.0}
```

不同共现矩阵中的候选商品赋予不同得分：

```python
        candidates = defaultdict(int)
        for aid in recent_clicks:
            for b, score in co_visitation_clicks.get(aid,{}).items():
                candidates[b] += weights["clicks"] * score
            for b,score in co_visitation_cart.get(aid,{}).items():
                candidates[b] += weights["cart"] * score
            for b,score in co_visitation_order.get(aid,{}).items():
                candidates[b] += weights["order"] * score
```

热门商品补位：

```python
        # 热门商品补位
        if len(top_items) < 20:
            need = 20 - len(top_items)
            for aid in global_popular_top:
                if str(aid) not in top_items:
                    top_items.append(str(aid))
                if len(top
```

**5. 得分：0.19178**

**6. 修改选取候选物品的得分：最近的20次点击，越近权重越高，越远越低**

```python
        for idx, aid in enumerate(clicked[-20:]):
            weight = (idx + 1) / 20
            for b, score in co_visitation_clicks.get(aid, {}).items():
                candidates[b] += weight * weights["clicks"] * score
            for b, score in co_visitation_cart.get(aid, {}).items():
                candidates[b] += weight * weights["cart"] * score
            for b, score in co_visitation_order.get(aid, {}).items():
                candidates[b] += weight * weights["order"] * score
```

**7. 得分: 0.19413**

****

### 获得共现矩阵(三种事件 + 时间戳)

> 03_2_multi-co-visitation-ts.py

**1. 按时间戳排序，避免乱序**

```python
events = sorted(events, key=lambda x: x['ts'])
```

**2. 两个事件时间间隔越大，权重越低；反之则权重越大**

```python
 # 遍历相邻点击
        for i in range(len(clicks) - 1):
            a, t1, ts1 = clicks[i]['aid'], clicks[i]['type'], clicks[i]['ts']
            b, t2, ts2 = clicks[i + 1]['aid'], clicks[i + 1]['type'], clicks[i + 1]['ts']
            if a == b:
                continue

            # 时间差（秒）
            dt = max(1, ts2 - ts1)
            # 时间权重：越近权重越高（可以调公式）
            w = 1.0 / (1 + (dt / 600))  # 600秒(10分钟)作缩放

            # clicks共现
            co_visitation_clicks[a][b] += w
            co_visitation_clicks[b][a] += w

        # 处理 click -> cart / order（保持原逻辑，但带时间权重）
        for i in range(len(events) - 1):
            a, t1, ts1 = events[i]['aid'], events[i]['type'], events[i]['ts']
            b, t2, ts2 = events[i + 1]['aid'], events[i + 1]['type'], events[i + 1]['ts']
            if a == b:
                continue

            dt = max(1, ts2 - ts1)
            w = 1.0 / (1 + (dt / 600))

            if t1 == 'clicks' and t2 == 'carts':
                co_visitation_cart[a][b] += w
            if t1 == 'clicks' and t2 == 'orders':
                co_visitation_order[a][b] += w
```

****

### 生成候选商品集

> 04_train-candidates.py

**1. 分批写入候选商品集，避免MemoryError**

```python
batch_size = 100000  # 每10万条写一次，可根据内存情况调节
candidate_rows = []
part_id = 0
total_batches = (total_lines // batch_size) + 1

with open(test_path, 'r', encoding='utf-8') as f, tqdm(f, total=total_test, desc='生成候选集') as session_pbar, tqdm(
        total=total_batches, desc='写入批次') as batch_pbar:
    for i, line in enumerate(session_pbar):
        session = json.loads(line)
        sid = session['session']

        clicked = [event['aid'] for event in session['events'] if event['type'] == 'clicks']

        candidates = defaultdict(int)
        for idx, aid in enumerate(clicked[-20:]):
            weight = (idx + 1) / 20
            for b, score in co_visitation_clicks.get(aid, {}).items():
                candidates[b] += weight * weights["clicks"] * score
            for b, score in co_visitation_cart.get(aid, {}).items():
                candidates[b] += weight * weights["cart"] * score
            for b, score in co_visitation_order.get(aid, {}).items():
                candidates[b] += weight * weights["order"] * score

        # 取top50作为候选
        top_items = sorted(candidates.items(), key=lambda x: -x[1])[:50]

        for aid, score in top_items:
            candidate_rows.append({
                "session": sid,
                "candidate": aid,
                "score": score,
                "source": "co_vis"
            })

        # 热门补位
        for aid in global_popular_top:
            if aid not in candidates:
                candidate_rows.append({
                    "session": sid,
                    "candidate": aid,
                    "score": 0,
                    "source": "popular"
                })

        # 分批写入
        if (i + 1) % batch_size == 0:
            df = pd.DataFrame(candidate_rows)
            df.to_parquet(f"dataset/candidates_part_{i // batch_size}.parquet", index=False)
            candidate_rows = []  # 清空缓存
            part_id += 1
            batch_pbar.update(1)

# 最后一批写入
if candidate_rows:
    df = pd.DataFrame(candidate_rows)
    df.to_parquet(f"dataset/candidates_part_last.parquet", index=False)
    part_id += 1
    batch_pbar.update(1)

print("候选集已分批保存到 dataset/")
```

**2. 候选文件整合**

```python
files = sorted(glob.glob("dataset/candidates_part_*.parquet"))

writer = None

with tqdm(total=len(files), desc="合并候选集文件") as merge_pbar:
    for f in files:
        df = pd.read_parquet(f)
        table = pa.Table.from_pandas(df)
        if writer is None:
            writer = pq.ParquetWriter("dataset/candidates.parquet", table.schema)
        writer.write_table(table)
        merge_pbar.update(1)

if writer:
    writer.close()

print("候选集已成功合并保存到 dataset/candidates.parquet")
```

**同理，可获得test-candidates**

> 04_2_test-candidates.py

****

### 提取session标签，保存为结构化parquet文件

> get_train_labels.py

<p>
    <img src="https://hhhi21g.github.io/assets/img/SR/com01/c3.png" alt="alt text"  />
</p>

****

### 获得训练特征train features

> 05_get-train-features.py

**各个特征计算方法: **

```python
for row in tqdm(candidates_df.itertuples(), total=len(candidates_df), desc="Processing candidates"):
    session = row.session
    cand = row.candidate
    base_score = row.score

    true_items = session_dict.get(session, [])
    if not true_items:
        continue

    # 时间加权特征
    co_scores = []
    co_scores_weighted = []
    session_len = len(true_items)
    max_ts = max([ts for _, ts in true_items]) if session_len > 0 else 0

    for aid, ts in true_items:
        score = co_vis.get(aid, {}).get(cand, 0)
        co_scores.append(score)
        # 时间衰减权重
        alpha = 0.001
        weight = np.exp(-alpha * (max_ts - ts))
        co_scores_weighted.append(score * weight)

    co_max = np.max(co_scores) if co_scores else 0
    co_mean = np.mean(co_scores) if co_scores else 0
    co_sum = np.sum(co_scores) if co_scores else 0

    co_max_weighted = np.max(co_scores_weighted) if co_scores_weighted else 0
    co_mean_weighted = np.mean(co_scores_weighted) if co_scores_weighted else 0
    co_sum_weighted = np.sum(co_scores_weighted) if co_scores_weighted else 0
```

**结构如下：**

```python
buffer_feat.append({
    "session": session,
    "candidate": cand,  # 候选商品
    "base_score": base_score,
    "co_max": co_max,  # 所有历史商品与候选商品最大共现分数
    "co_mean": co_mean,  # 平均共现分数
    "co_sum": co_sum,    # 总共现分数
    "co_max_weighted": co_max_weighted,  # 时间衰减函数，计算weight
    "co_mean_weighted": co_mean_weighted,
    "co_sum_weighted": co_sum_weighted,
    "session_len": session_len
})
```

**同理，可获得test features**

> 05_2_get-test-features.py

****

### 使用LightGBM快速测试

**1. 将labels逐步返回，分批次提供给训练**

```python
def labels_generator(labels_path):
    parquet_file = pq.ParquetFile(labels_path)
    for i in range(parquet_file.num_row_groups):
        batch = parquet_file.read_row_group(i).to_pandas()
        session_dict = {}
        for _, row in batch.iterrows():
            session = row['session']
            session_dict[session] = {
                'clicks': set(row.get('clicks', [])),
                'carts': set(row.get('carts', [])),
                'orders': set(row.get('orders', []))
            }
        yield session_dict  # 逐步返回，一小批一小批的提供给训练过程
        del batch
        gc.collect()
```

features同理

**2. 使用LightGBM进行训练3个二分类模型**

```python
def train_lgb_incremental(features_path, labels_path, target_cols):
    labels_gen = labels_generator(labels_path)
    models = {target: None for target in target_cols}

    parquet_file = pq.ParquetFile(features_path)
    total_batches = parquet_file.num_row_groups

    for batch_idx, batch_df in enumerate(tqdm(feature_batch_generator(features_path, labels_gen),
                                              total=total_batches,
                                              desc="Processing Batches")):
        feature_cols = [c for c in batch_df.columns if c not in ['session', 'candidate',
                                                                 'clicks_label', 'carts_label', 'orders_label']]
        for target_col in tqdm(target_cols, desc=f"Training Models on Batch {batch_idx + 1}", leave=False):
            X = batch_df[feature_cols]
            y = batch_df[target_col]
            lgb_train = lgb.Dataset(X, label=y)

            params = {
                'objective': 'binary',
                'metric': 'auc',
                'boosting_type': 'gbdt',
                'learning_rate': 0.05,
                'num_leaves': 64,
                'max_depth': -1,
                'min_data_in_leaf': 50,
                'feature_fraction': 0.8,
                'bagging_fraction': 0.8,
                'bagging_freq': 5,
                'verbosity': -1
            }

            if models[target_col] is None:
                models[target_col] = lgb.train(params, lgb_train, num_boost_round=100)
            else:
                models[target_col] = lgb.train(params, lgb_train, num_boost_round=100, init_model=models[target_col])

            del X, y, lgb_train
            gc.collect()

        del batch_df
        gc.collect()

    # 保存模型
    for target_col, model in models.items():
        model.save_model(f"lgbm_{target_col}.txt")

    return models
```

**3. 使用3个模型进行测试**

> 06_2_test.py

```python
def predict_submission(features_path, models, topk=20):
    parquet_file = pq.ParquetFile(features_path)
    # print(f"[INFO] Total row groups in parquet: {parquet_file.num_row_groups}")

    results = []

    for rg in tqdm(range(parquet_file.num_row_groups), desc="Predicting"):
        batch = parquet_file.read_row_group(rg).to_pandas()
        # print(f"[DEBUG] Row group {rg}: shape = {batch.shape}")

        batch_results = []  # 每个 batch 的结果先放临时列表

        for target in ["clicks", "carts", "orders"]:
            model = models[target]
            model_feature_names = model.feature_name()

            # 补齐缺失特征
            for f in model_feature_names:
                if f not in batch.columns:
                    batch[f] = 0

            X = batch[model_feature_names]
            batch[f"{target}_pred"] = model.predict(X)

            # 分组取 TopK
            preds = (
                batch.groupby("session")
                .apply(lambda x: x.nlargest(topk, f"{target}_pred")["candidate"].astype(str).tolist())
            )
            preds = preds.reset_index().rename(columns={0: "labels"})
            preds["labels"] = preds["labels"].apply(lambda x: " ".join(x))
            preds["session_type"] = preds["session"].astype(str) + f"_{target}"

            batch_results.append(preds[["session_type", "labels"]])

        # 合并并去重，避免 batch 内部重复
        batch_results = pd.concat(batch_results, ignore_index=True)
        batch_results = batch_results.drop_duplicates(subset=["session_type"], keep="first")

        results.append(batch_results)

    # 拼接所有 batch
    submission = pd.concat(results, ignore_index=True)

    # 再次全局去重，保证唯一
    submission = submission.drop_duplicates(subset=["session_type"], keep="first")

    return submission
```

**4. 得分：0.21721**

****
