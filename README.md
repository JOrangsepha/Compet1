# RETECO 2a · Drones 官方 BM25 基线复现

2026-11-25 · CPU · SemEval 2027 Task 1, Sub-track 2a

## 工作计划与进度

**当前进度（2026-11-25）：阶段 1 的基线复现已完成。** Drones 的 train/dev 共四组分数均与官方结果一致；通过本地格式检查，不代表已完成参赛报名或正式提交。阶段 2、3 尚未开始。

| 时间 | 阶段 | 任务与验收标准 | 状态 |
| --- | --- | --- | --- |
| 11/25–11/30 | 1 · 基线复现 | 在 Drones 上用 CPU 跑通「只用当前轮」和「拼接全部历史」两种官方 BM25。复核 nDCG@10、TREC 输出和重复运行的一致性。 | 已完成（11/25） |
| 12/01–12/20 | 缓冲期 | 整理运行记录、依赖和复现命令，不开新实验。 | 计划中 |
| 12/21–12/27 | 2 · 历史筛选小实验 | 在 3 个领域的公开 train 上比较「相关历史」「最近历史」和「全部历史」。保持 BM25 不变，对相关历史与最近历史使用相同的历史词元预算，区分轮次选择与查询变短的影响。记录失败案例，并在进入 dev 前锁定规则。 | 未开始 |
| 12/28–12/31 | 3 · 全域开发集验证 | 用锁定的规则在 2a 全部 11 个领域的公开 dev 上评估一次。记录各领域 nDCG@10、领域宏平均、复现命令和通过格式检查的示例。若未稳定超过最强对照，保留全历史 BM25。 | 未开始 |

参赛范围为 2a。以上日期是内部排期，正式预测提交以[官方赛程](https://datascienceuibk.github.io/RETECO/timeline.html)为准。

## 阶段 1 · 运行记录

使用官方公开的 Drones 领域数据（16,381 篇文档）。按官方切分跑 train（104 个有标签的对话轮次）和 dev（38 个），用同一个索引和 BM25 参数（k1=0.9，b=0.4），只改变查询是否拼接历史。未训练模型或调参，未使用 GPU、模型 API 或隐藏测试集。

| BM25 输入 | Train nDCG@10 | Dev nDCG@10 |
| --- | ---: | ---: |
| 当前轮 | 0.1400 | 0.1695 |
| 当前轮 + 全部对话历史 | 0.3162 | 0.2954 |

四项都与[官方 Drones 基线表](https://github.com/DataScienceUIBK/RETECO/blob/7ee8acfa711639290bed5ff5b5a5aad9480ba539/starter_kit/BASELINE_RESULTS.md#track-2--recor)四位小数一致。Dev 上拼接历史比只用当前轮高 **0.1259**；这是官方两种配置的差异，不是新增方法的收益。四个 TREC 排序文件都通过了官方格式检查（零错误、零警告），并检查了主题全覆盖、同主题文档去重和连续排名。官方轻量评分器从落盘的 TREC 文件独立重算，也得到表中分数。

## 复现

本仓库存放工作计划和运行记录。下面的脚本、日志和本地排序文件位于另行交付的复现包中。

Linux、Python 3.12 和 Java 21。首次运行会下载公开数据与依赖。只安装 CPU 基线实际使用的依赖；Pyserini 仅用于 Lucene analyzer，不安装其神经检索依赖。使用提供相同 `pytrec_eval` 导入接口的 `pytrec-eval-terrier` wheel，不修改官方基线代码。

在解压后的复现包目录中运行：

```bash
bash run_baseline.sh
```

脚本校验数据 SHA-256，使用新的输出目录以避免命中官方脚本缓存。首次实测基线计算（包括建索引和四组评估）约 8 秒，不含下载和配置环境。

- 官方代码：[`7ee8acf`](https://github.com/DataScienceUIBK/RETECO/tree/7ee8acfa711639290bed5ff5b5a5aad9480ba539/starter_kit)。
- 官方数据：[`c21e10b`](https://huggingface.co/datasets/DataScience-UIBK/RETECO-SemEval2027/tree/c21e10ba8aa60eaa201f60fc4604364469c5ec17/track2_recor/drones)。
- `baseline.log`、`validation.log`：原始运行和检查输出摘录。
- `runs/`：仅公开 train/dev 的四份本地排序结果和汇总指标。不是线上榜单成绩或正式提交。复现包不含原始数据、环境目录或凭据。

已在全新目录重新下载固定版本数据并从头安装依赖、运行脚本；两次运行的四个 TREC 文件和指标 JSON 逐字节一致。
