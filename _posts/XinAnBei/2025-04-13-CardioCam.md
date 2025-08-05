---
layout: post

title: 信安杯心跳认证方法_CardioCam

date: 2025-04-13 16:31:23 +0900

categories: [信安杯]
---

### CardioCam: Leveraging Camera on Mobile Devices to Verify Users While Their Heart is Pumping

论文链接：https://dl.acm.org/doi/10.1145/3307334.3326093

#### PCA(主成分分析)： 机器学习中无监督学习的一种降维算法

将高维数据映射到低维空间，通过数据投影的线性转化(正交变换)，用较少的数据集维数，**保留数据集中对方差贡献最大的特征**，也就是保留数据的主要特征。

具体来说：

- PCA会计算原始数据的**协方差矩阵**，然后找到协方差矩阵的特征向量和特征值；

  **特征向量**：新空间的基向量；

  **特征值：**描述了每个特征向量的重要程度；

- 选择保留**最大的k个特征值对应的特征向量**；

- 将原始数据投影到这k个特征向量所张成的低维空间中。

------

#### 8.1  Feature Transformation grounded on PCA(基于PCA的特征变换)

心脏波形(Cardiac waves)不同时期会出现小规模的变化，因此本文提出一种特征变换方案，用于构建可靠的用户特征，并在PCA基础上进行用户验证；

- PCA将心脏特征**转换为低维空间中的一组正交主成分**，其中**前几个**最具代表性并对信号干扰稳健；

- 原始成分通过**对生物特征矩阵**应用**奇异值分解(SVD)**,
  - 生物特征矩阵包含：n次心动周期观察的心脏特征；
  
  - 将原始成分表示为：**W = { w<sub>1</sub>, w<sub>2</sub>, ... , w<sub>p</sub>}**, w<sub>j</sub>表示n*1的原始成分向量；
  
    > SVD讲解：https://bainingchao.github.io/2018/10/11/%E4%B8%80%E6%AD%A5%E6%AD%A5%E<br/>6%95%99%E4%BD%A0%E8%BD%BB%E6%9D%BE%E5%AD%A6%E5%A5%87%E5%BC%82%E5%<br/>80%BC%E5%88%86%E8%A7%A3SVD%E9%99%8D%E7%BB%B4%E7%AE%97%E6%B3%95/
  
- 选择具有**最大归一化方差**的前k个主成分，称为**心脏摘要(cardiac abstracts)**;

  - 所有的心脏周期都具有相似的前几个主成分，其描述心脏波形的形态轮廓，而其余主成分则能更好地区分不同的个体；

  - 因此，**舍弃前两个主成分**，从第三个主成分开始进行主成分选择过程，公式如下：
  
    <p>
        <img src="https://hhhi21g.github.io/assets/img/xinan/x0.png" alt="alt text" style="zoom:67%;" />
    </p>
    
    
    
    - k: 选定的主成分数量；*τ* = 0.9是预定义的阈值，通过经验确定；		

------

#### 8.2  Profile Matching

通过**测量新捕获的心脏摘要**与用户档案中的心脏摘要的相似性来判断用户身份是否合法。

- CardioCam使用**一组心脏摘要向量F = {f<sub>1</sub>,...,f<sub>70</sub>},** 该向量是从合法用户配置文件中的70个心脏周期派生的
  - 每个心脏周期通过**将心脏特征向量与8.1中的W相乘得到心脏摘要向量**

- 每个**新捕获的**需要验证的心脏波将**基于PCA进行特征变换**，以**获得一个新的心脏摘要向量s**

- 计算每个s和F之间的**平均欧几里得距离**：

  <p>
      <img src="https://hhhi21g.github.io/assets/img/xinan/x1.png" alt="alt text" style="zoom:67%;" />
  </p>

- 使用**阈值*η***，Dist(s)<= η, 则通过认证；

  - 为获得优化的阈值，**需要一些合法样本和伪造者的对抗样本**来检查和评分一组预定义的阈值；

  - 使用**Youden's J统计量进行评分**，通过递归计算不同阈值η 对应的J统计量，选择使得J统计量最大化的阈值η<sub>u</sub>

    <p>
        <img src="https://hhhi21g.github.io/assets/img/xinan/x2.png" alt="alt text" style="zoom:67%;" />
    </p>

****
