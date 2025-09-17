# 省流版

提出了PPG优于PAT，进行了大量的特征提取工作，三个新方法没见过fliter、wrapper、embedding

提出了分开建模，分为LFC与HFC分开进行建模，发现成效不错。

| 名称    | 全称                     | 含义                           |
| ------- | ------------------------ | ------------------------------ |
| **LFC** | Low Frequency Component  | 低频成分：变化缓慢的趋势性波动 |
| **HFC** | High Frequency Component | 高频成分：变化快速的瞬时波动   |

- **LFC建模专注拟合趋势** → 用**ANN**（非线性拟合能力强）
- **HFC建模专注拟合残差** → 用**RNN**（捕捉时序依赖）

# 1. 研究背景

## 1.1 存在的问题

传统设备依赖袖带式设备，无法连续监测

PAT(pulse arrival time)对于low frequency blood不敏感

PPG特征不统一

# 2. 创新点

## 2.1 频率分解建模策略

因为“lower frequency component of PAT was not well correlated to SBP”所以采用将提取到的特征分为LFC与HFC分别建模

## 2.2 首次大规模验证PPG形态特征优于PAT

## 2.3 采用外部验证数据集证明模型泛化能力强

# 3. 特征提取

| 特征类别 | 示例                         | 含义                                  |
| -------- | ---------------------------- | ------------------------------------- |
| PAT类    | PATV, PATP, PATMD, PATIT     | ECG到PPG不同特征点的时间差            |
| HR       | 心率                         | 心跳周期倒数                          |
| PIR类    | PIRP, PIRMD, PIRIT           | PPG峰值与谷值强度比，反映动脉直径变化 |
| CT / AS  | Crest Time / Ascending Slope | 波形上升时间与斜率                    |
| PPGK     | PPG特征值                    | 与血液黏度、外周阻力相关              |
| BW类     | SBW, DBW, BWR                | 分支宽度（systolic/diastolic）        |
| APG类    | AIa, AIb, AIR                | 二阶导数波形特征，反映动脉硬化        |
| PCA类    | PC1~PC4                      | 主成分，表征整体波形形状              |

## 3.1 特征选择

使用6种方法评估特征重要性：

- **Filter方法**：Pearson相关、互信息（MI）
- **Wrapper方法**：递归特征消除（RFE）
- **Embedded方法**：Ridge、Lasso、Random Forest

#### 结果：

- **Top 3 特征**：PPGK、PIRP、HR（优于PAT）
- **最终选取28个特征**用于建模（包含PAT、HR、PPG形态）

# 4. Frequency Component Separation

#### 📌 动机：

- 既往研究发现PAT对**低频血压变化不敏感**
- 提出将血压与特征分解为：
  - **LFC（低频成分）<0.006Hz**
  - **HFC（高频成分）≥0.006Hz**

#### 📌 方法：

- 使用Butterworth低通滤波器分离LFC/HFC
- 分别建模LFC与HFC，再合并输出

# 5. 模型构建于评估

| 模型    | 特点                                   | 最优任务        |
| ------- | -------------------------------------- | --------------- |
| **LR**  | 线性回归                               | 基线模型        |
| **RF**  | 随机森林（Extremely Randomized Trees） | 非线性拟合      |
| **ANN** | 单隐藏层（100节点，tanh）              | **LFC建模最优** |
| **RNN** | 双向LSTM（2层×40单元，250时间步）      | **HFC建模最优** |

# 6. 实验结果与分析

内部验证

| 指标     | SBP                | DBP           |
| -------- | ------------------ | ------------- |
| ME       | 0.05 mmHg          | -0.05 mmHg    |
| SDE      | **6.92 mmHg**      | **3.99 mmHg** |
| MAD      | 5.07 mmHg          | 2.86 mmHg     |
| BHS等级  | A                  | A             |
| IEEE等级 | B（SBP）/ A（DBP） | ✅             |

外部验证

| 指标   | SBP         | DBP         |
| ------ | ----------- | ----------- |
| ME     | -0.006 mmHg | -0.004 mmHg |
| SDE    | 7.04 mmHg   | 4.77 mmHg   |
| MAD    | 5.13 mmHg   | 2.81 mmHg   |
| 相关性 | 0.52 ± 0.40 | 0.53 ± 0.38 |

# 7. 结论

1. **PPG形态特征（如PPGK、PIRP）在血压估计中优于传统PAT指标**；
2. **频率分解建模策略（LFC+HFC）可显著提升估计精度**；
3. **非线性模型（ANN+RNN）比线性模型更适合复杂生理映射**；
4. **模型在内外部验证中均满足国际标准（AAMI、BHS、IEEE）**，具备实际应用潜力。