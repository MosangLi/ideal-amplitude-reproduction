# ideal-amplitude-reproduction
Reproduction of the ideal amplitude factor from Kaiyuan Securities, including factor construction, backtesting, validation, and documentation.
# Ideal Amplitude Factor Reproduction

> 复现开源证券《振幅因子的隐藏结构——市场微观结构研究系列（7）》中的理想振幅因子，并完成因子构建、有效性检验、稳健性测试与标准化入库。

## Project Status

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Market](https://img.shields.io/badge/market-A--Share-red)
![Language](https://img.shields.io/badge/language-Python-blue)
![License](https://img.shields.io/badge/license-not%20specified-lightgrey)

当前进度：**研报拆解与复现口径确认**

## Overview

传统振幅因子在 A 股市场中具有一定的负向选股能力，但稳定性有限。研报按照股票所处的价格状态，将传统振幅拆分为：

- 高价振幅因子 `V_high`
- 低价振幅因子 `V_low`

在横截面上分别标准化后，构造理想振幅因子：

```text
IdealAmplitude = ZScore(V_high) - ZScore(V_low)
```

本项目将检验该因子的预测方向、IC、RankIC、分组收益和参数稳健性，并将最终实现整理为可重复运行的标准因子代码。

## Research Hypothesis

核心假设：

> 同一只股票在相对高价阶段产生的振幅，比低价阶段产生的振幅具有更强的负向选股能力。

因此，理想振幅因子理论上是一个**负向因子**：

```text
因子值越低 → 预期未来收益越高
因子值越高 → 预期未来收益越低
```

## Factor Construction

在调仓日 `T`，对每只股票执行以下步骤。

### 1. Calculate Daily Amplitude

回看最近 `N` 个交易日，每日振幅定义为：

```text
Amplitude_t = High_t / Low_t - 1
```

### 2. Split by Price State

将最近 `N` 个交易日按照收盘价从低到高排序：

```text
Low-price observations  = lowest λ × N days
High-price observations = highest λ × N days
```

### 3. Calculate Conditional Amplitude

```text
V_low  = mean(Amplitude_t | low-price observations)
V_high = mean(Amplitude_t | high-price observations)
```

### 4. Cross-sectional Standardization

在同一调仓日的股票横截面上，分别对 `V_high` 和 `V_low` 进行去极值和标准化：

```text
Z_high = ZScore(Winsorize(V_high))
Z_low  = ZScore(Winsorize(V_low))
```

### 5. Construct Ideal Amplitude

```text
IdealAmplitude = Z_high - Z_low
```

## Baseline Configuration

| Parameter | Baseline |
|---|---:|
| Market | China A-share |
| Lookback window `N` | 20 trading days |
| Price field | Close |
| Split ratio `λ` | 25% |
| Rebalance frequency | Monthly |
| Portfolio groups | 5 |
| Factor direction | Negative |
| Forward return | Next-period return |
| Transaction costs | To be configured |

> 部分数据清洗和交易执行口径未在研报中完全明确，项目会在 [`docs/assumptions.md`](docs/assumptions.md) 中记录所有复现假设。

## Validation Framework

因子有效性将按照以下顺序进行检验：

1. 数据完整性与因子覆盖率
2. 因子分布与异常值检查
3. IC 与 RankIC 分析
4. 五分组收益与单调性
5. 多空组合净值
6. 换手率与交易成本
7. 行业、市值中性化
8. 参数及样本外稳健性测试

## Target Metrics

以下数值来自原研报，仅作为复现参照，并非本项目当前结果：

| Metric | Reported | Reproduced |
|---|---:|---:|
| IC Mean | -0.067 | TBD |
| ICIR | -2.97 | TBD |
| Monthly Win Rate | 84.2% | TBD |
| Long-short Annual Return | 23.3% | TBD |

由于数据源、复权方式、股票池过滤和交易规则可能不同，复现工作的主要判断标准是：

- 因子方向一致
- IC 数量级合理
- 分组收益具有预期单调性
- 主要结论在不同参数下保持稳定

## Repository Structure

```text
ideal-amplitude-reproduction/
├── README.md
├── configs/
│   └── baseline.yaml
├── data/
│   └── README.md
├── docs/
│   ├── assumptions.md
│   ├── research_notes.md
│   └── reproduction_report.md
├── notebooks/
│   ├── 01_manual_validation.ipynb
│   ├── 02_factor_construction.ipynb
│   ├── 03_factor_evaluation.ipynb
│   └── 04_robustness_tests.ipynb
├── src/
│   ├── data/
│   ├── factors/
│   │   └── ideal_amplitude.py
│   ├── evaluation/
│   └── utils/
├── tests/
│   └── test_ideal_amplitude.py
├── outputs/
│   ├── figures/
│   └── tables/
├── requirements.txt
└── .gitignore
```

## Reproduction Roadmap

- [x] 登记研报信息
- [x] 提取核心假设
- [x] 确定初始因子公式
- [ ] 核对完整复现口径
- [ ] 完成小样本手算验证
- [ ] 准备并清洗行情数据
- [ ] 实现因子计算模块
- [ ] 完成 IC 与分组测试
- [ ] 复现多空组合
- [ ] 执行稳健性测试
- [ ] 分析与研报的结果差异
- [ ] 输出复现报告
- [ ] 标准化因子并入库

## Data Policy

本仓库不提交以下内容：

- 原始或付费行情数据
- 公司内部数据及文档
- Slack 消息或内部任务信息
- 账号、密码、Token 与 API Key
- 实盘交易配置
- 未获得再分发授权的研报 PDF

数据目录仅保留字段说明和获取方式。敏感配置通过环境变量或本地配置文件管理。

## Reproducibility

最终版本应满足：

```bash
pip install -r requirements.txt
python -m src.factors.ideal_amplitude
```

具体命令将在实现完成后更新。所有实验参数、数据版本和随机种子都会被记录，以保证结果可追踪、可重复。

## Reference

开源证券，《振幅因子的隐藏结构——市场微观结构研究系列（7）》，2020-05-16。

- [BigQuant 文档页面](https://bigquant.com/wiki/doc/w5WH1P01Bl)

## Disclaimer

本项目用于研究学习与研报复现，不构成任何投资建议。历史回测结果不代表未来表现。项目中的研究结论可能受到数据质量、交易规则和复现假设影响。
