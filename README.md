# RETECO 2a · Drones 官方 BM25 基线复现

2026-11-25 · CPU · SemEval 2027 Task 1, Sub-track 2a

使用官方公开的 Drones 领域数据（16,381 篇文档）。按官方切分跑 train（104 个有标签的对话轮次）和 dev（38 个），用同一个索引和 BM25 参数（k1=0.9，b=0.4），只改变查询是否拼接历史。未训练模型或调参，未使用 GPU、模型 API 或隐藏测试集。

| BM25 输入 | Train nDCG@10 | Dev nDCG@10 |
| --- | ---: | ---: |
| 当前轮 | 0.1400 | 0.1695 |
| 当前轮 + 全部对话历史 | 0.3162 | 0.2954 |

四项都与[官方 Drones 基线表](https://github.com/DataScienceUIBK/RETECO/blob/7ee8acfa711639290bed5ff5b5a5aad9480ba539/starter_kit/BASELINE_RESULTS.md#track-2--recor)四位小数一致。Dev 上拼接历史比只用当前轮高 **0.1259**；这是官方两种配置的差异，不是新增方法的收益。四个 TREC 排序文件都通过了官方格式检查（零错误、零警告），并检查了主题全覆盖、同主题文档去重和连续排名。官方轻量评分器从落盘的 TREC 文件独立重算，也得到表中分数。

## 复现

本仓库仅存放运行记录。下面的脚本、日志和本地排序文件位于另行交付的复现包中。

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

下一步（尚未执行）：保持索引与评分不变，以“全部历史”和“最近几轮”为对照，测试相关轮次筛选，再到独立开发集验证。

已在全新目录重新下载固定版本数据并从头安装依赖、运行脚本；两次运行的四个 TREC 文件和指标 JSON 逐字节一致。
