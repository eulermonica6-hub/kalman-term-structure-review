# Kalman Filters for Interest-Rate Term-Structure Estimation

## 论文展示

**Review on the applications of Kalman filter class approaches to term structure estimation of interest rates**

- Authors: Shu Wu, Lina Zhang, Rende Li
- Venue: *International Review of Economics and Finance*, 109 (2026), 105440
- DOI: [10.1016/j.iref.2026.105440](https://doi.org/10.1016/j.iref.2026.105440)

这是一篇方法综述，不是单一数据集上的新模型论文。本仓库重点说明其综述工程、状态空间统一视角、滤波器适用性比较和文中汇总的评测设计。

## 一句话概括

论文以期限结构的状态空间表示为主线，系统梳理 KF、EKF、IEKF、UKF 和 SRUKF 在利率期限结构估计中的适用条件、参数调优、计算复杂度与稳定性，并将线性/非线性模型和数据来源放进同一比较框架。

## 工程特色

### 1. 可追溯的系统检索与筛选

- 检索 Web of Science、Scopus、PubMed、IEEE Xplore 和 Google Scholar。
- 覆盖 1990–2025 年，关键词包括 Kalman filter、term structure、Vasicek、CIR、EKF、UKF、SRUKF 等。
- 纳入标准要求研究将 Kalman 类方法用于利率期限结构估计，或讨论相关算法理论与改进。
- 2,345 篇初始记录经标题/摘要筛选、全文审查后，最终纳入 101 篇。

| 阶段 | 文献数 |
|---|---:|
| Initial retrieval | 2,345 |
| Title/abstract screening | 345 |
| Full-text review | 120 |
| Final inclusion | 101 |

### 2. 用统一状态空间语言连接不同利率模型

连续时间利率模型被离散为状态方程，债券价格/收益率被写为观测方程：

```text
xₜ₊₁ = M(xₜ, Φ, h) + process noise
Yₜ(τ) = (1/τ) log A(Φ,τ) + (1/τ) B(Φ,τ)xₜ + observation noise
```

该统一表达可以同时讨论：

- Vasicek 等线性高斯模型与经典 KF；
- CIR、非线性观测方程等场景与 EKF/UKF；
- 需要更高数值稳定性的 SRUKF；
- QML、参数估计、隐状态估计和期限结构预测。

### 3. 把“调参”作为工程问题单独处理

综述不只罗列算法，还集中讨论：

- 初始状态 `x̂₀` 与初始协方差 `P₀`；
- 过程噪声和观测噪声协方差 `Q/R`；
- EKF 的 Jacobian 线性化误差；
- UKF 的 sigma-point 参数；
- SRUKF 的 Cholesky 因子传播与数值稳定性。

### 4. 同时关注统计性能与运行成本

比较维度包括 RMSE、参数收敛、跨独立运行的稳定性、Wilcoxon signed-rank 检验和 CPU 时间，而不是只展示单条拟合曲线。

## 实验与证据设计

### 综述型实验结构

论文的证据由三层构成：

1. **文献综合**：按照纳入研究归纳模型、滤波器、数据源与适用条件；
2. **方法对比**：汇总既有研究中 EKF、UKF、SRUKF 的仿真和经验比较；
3. **稳定性证据**：重点讨论 1,000 次独立运行的 RMSE 分布、参数收敛和运行时间。

因此，表格中的部分数值是所引用原始研究的结果汇总，不能全部理解为本论文重新采集数据后独立复现的实验。

### 代表性比较设置

- 利率模型：Vasicek、CIR，以及多因子 affine term-structure models。
- 观测：零息债券收益率/价格、swap rate、LIBOR 等，覆盖多个期限。
- 算法：KF、EKF、UKF、SRUKF；部分研究还涉及 IEKF 与 KF-QML。
- 指标：变量/参数 RMSE、标准差、收敛速度、显著性检验和 CPU 时间。
- Monte Carlo：代表性比较使用 1,000 次独立运行；图示示例使用 500 个周度时间步（约 10 年）。

### 适用性结论

| 场景 | 更合适的工具 | 工程理由 |
|---|---|---|
| 线性、高斯状态空间 | KF | 计算简单且具有最优线性估计性质 |
| 弱非线性、可获得 Jacobian | EKF | 通过局部线性化实现，但依赖导数近似 |
| 非线性状态/观测关系 | UKF | 用 sigma points 传播均值和协方差，避免显式 Jacobian |
| 强非线性或数值稳定性要求高 | SRUKF | 传播协方差平方根，降低数值不稳定风险 |
| 噪声相关或模型扩展 | Augmented KF / 更丰富状态空间 | 将相关噪声或宏观因子纳入状态表示 |

### 代表性数值证据

论文汇总的比较表显示，在强非线性的 CIR 模型中，UKF 相比 EKF 的 RMSE 优势具有统计显著性（Wilcoxon 检验 `p < 0.001`）；在弱非线性的 Vasicek 模型中差异不显著（`p = 0.073`）。在 SRUKF 与 UKF 的比较中，SRUKF 的 RMSE 波动更小，体现出平方根传播带来的稳定性。

一个来自已发表研究的三因子 affine 模型汇总示例中，SRUKF 在 10 年期限上的 RMSE 低于 EKF：

| 模型 | 期限 | EKF RMSE | SRUKF RMSE | EKF CPU (s) | SRUKF CPU (s) |
|---|---:|---:|---:|---:|---:|
| AFM(N) | 10 years | 29.60 | 15.96 | 0.15 | 0.22 |
| AFO(3) | 10 years | 37.73 | 0.58 | 0.16 | 0.23 |
| AFG(3) | 10 years | 28.00 | 9.82 | 0.17 | 0.24 |

该表也展示了典型权衡：SRUKF 通常以略高 CPU 成本换取更好的非线性估计稳定性。

## 数据来源与复现边界

综述记录的研究使用过 Fama–Bliss、上海证券交易所、SHIBOR、Datastream、LIBOR、Bloomberg、ECB/Eurostat、印度 CCIL 等数据源，覆盖美国、中国、欧洲和印度市场。部分数据属于商业数据库或受机构许可约束，因此本展示仓库不分发原始数据。

由于本文是综述，复现应区分：

- **可复核内容**：检索范围、纳入标准、状态空间公式、算法适用性和已发表结果的出处；
- **依赖原研究内容**：具体 Monte Carlo 参数、样本清洗、数据许可和表格数值；
- **不应过度外推**：UKF/SRUKF 的优势取决于非线性程度、噪声设定、初值、期限范围和数值实现。

## 引用

```text
Wu, S., Zhang, L., & Li, R. (2026). Review on the applications of Kalman
filter class approaches to term structure estimation of interest rates.
International Review of Economics and Finance, 109, 105440.
https://doi.org/10.1016/j.iref.2026.105440
```
