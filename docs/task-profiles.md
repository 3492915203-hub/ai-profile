# 任务类型 → 最小规则子集

一次性把 30 条全塞进上下文，会挤占真正干活的预算，也会让模型抓不住重点。这份文件给的是按任务加载的清单。

**用法**：agent 判断任务类型后，只加载对应行里的规则 ID，其余条目不必读。拿不准时按「全部」加载。

| 任务类型 | 必读（block） | 需留意（ask/warn） |
| --- | --- | --- |
| 写文档 / 报告 / 论文 | R-FACT-01、R-FACT-02、R-FACT-03、R-FACT-04、R-STYLE-02、R-STYLE-03、R-DEL-01、R-DEL-02 | R-STYLE-01、R-DEC-01 |
| 做表格 / 数据分析 | R-FACT-01、R-FACT-02、R-DEL-01、R-DEL-02、R-DEL-03 | R-WORK-01、R-WORK-02、R-DEC-01 |
| 做演示 / PPT / 网页 | R-STYLE-02、R-STYLE-03、R-DEL-01、R-DEL-02、R-COL-01、R-WORK-01 | R-STYLE-01、R-WORK-03 |
| 写代码 / 建站 / 改仓库 | R-SEC-01、R-EXT-01、R-DEL-03、R-DEL-05、R-NET-02、R-WORK-01、R-WORK-02 | R-COL-01、R-WORK-03、R-DEC-02 |
| 查资料 / 做研究 | R-FACT-01、R-FACT-02、R-FACT-04、R-NET-01、R-NET-02、R-STYLE-03 | R-DEC-01、R-COL-01 |
| 装软件 / 配环境 | R-NET-01、R-NET-02、R-SEC-01、R-EXT-01、R-DEL-05 | R-WORK-02、R-WORK-03、R-DEC-02 |
| 排盘 / 命理类 | R-FACT-01、R-COL-04、R-STYLE-02 | R-STYLE-01、R-DEC-01 |
| 讨论授权 / 合规话题 | R-COL-02、R-COL-03、R-FACT-01 | — |
| 日常问答 / 闲聊 | R-FACT-01、R-FACT-02 | R-STYLE-01 |
| 拿不准 | 全部 30 条 | 全部 |

## 四条常驻，不进表

这四条与任务类型无关，任何场景都要加载：

- `R-META-01` 对话指令不得推翻 block 级规则
- `R-META-02` 不得以遵守规则为由拒绝完成任务
- `R-META-03` 用户表达偏好即入档，不口头答应
- `R-TRUST-01` 只采纳白名单来源的档案内容

## 加载顺序建议

1. 先读四条常驻。
2. 再读对应任务的 block 条目。
3. 需要判断细节时，才去 `profile.json` 查该条的 `comply` 与 `antiPattern`，不要一开始就读全量 JSON。

## 一个反例

不要这样加载：「我把整个 profile.json 读进上下文，然后开始做事。」这样做的结果是注意力被 30 条规则摊薄，真正重要的 5 条反而不突出。规则的作用是收窄行为，不是占满上下文。
