---
name: research-work-knowledge-manager
description: Manage a personal research knowledge system around research directions, questions, evidence, and judgment changes. Use when triaging or reading papers, turning papers/data/observations/meetings into durable research knowledge, synthesizing sources around a question, updating confidence and boundaries, deciding what evidence or literature to seek next, or auditing whether a knowledge base improves research decisions. Do not use merely to store files, track generic tasks, manage submissions, or produce standalone summaries.
metadata:
  skill_type: judgment
---

# Research Work Knowledge Manager

## 目标

把个人知识库维护成研究判断系统，而不是资料仓库。

围绕以下循环工作：

```text
研究方向 → 研究问题 → 当前判断 → 新证据
→ 判断更新 → 未知与边界 → 下一项证据需求
```

优先提高研究者定义问题、判断证据、识别边界、面对反证和更新认识的能力。不要把登记完整、笔记数量或目录整齐当作成功。

## 核心契约

1. 先读取工作区中的 `AGENTS.md`、索引、frontmatter 规范和来源写入边界。已有本地规则优先于本 skill 的通用结构。
2. 先确认当前研究方向、活跃问题和已有判断。信息不足时提出一个暂定版本并明确标记，不把推测写成用户结论。
3. 先判断新输入的位置和决策价值，再决定处理深度。不要平均处理所有材料。
4. 区分来源事实、作者解释、Agent 推断和研究者判断。凡进入核心论证链的内容，都回到原始来源核验。
5. 至少检验一个真正可能威胁当前判断的替代解释、反例或边界条件。
6. 把稳定更新写入研究问题页或综合页。单篇笔记只作为来源层与稳定知识层之间的过渡物。
7. 从未解决的不确定性生成下一项证据需求，同时说明何时应停止继续读文献。
8. 低风险、可验证的来源信息可以直接更新；证据权重、置信变化、问题重定义等高价值判断先生成更新提案，等待研究者确认后再写入稳定知识。

## 选择工作模式

根据请求选择最小模式，不要求每次运行完整流程。

| 模式 | 适用请求 | 必读参考 |
|---|---|---|
| `orient` | 建立研究方向、问题地图或开题认知框架 | [knowledge-model.md](references/knowledge-model.md) |
| `triage` | 判断一篇新论文或材料是否值得读、读到什么程度 | [paper-reading.md](references/paper-reading.md) |
| `read` | 重建论文论证、选择性细读、核验核心证据 | [paper-reading.md](references/paper-reading.md)、[evidence-judgment.md](references/evidence-judgment.md) |
| `update` | 把新材料或阅读结果加入个人知识库 | [knowledge-model.md](references/knowledge-model.md)、[workspace-adaptation.md](references/workspace-adaptation.md) |
| `synthesize` | 围绕共同问题综合多篇论文或多种材料 | [synthesis-and-selection.md](references/synthesis-and-selection.md)、[evidence-judgment.md](references/evidence-judgment.md) |
| `select` | 根据知识空白决定下一篇读什么或筛选候选文献 | [synthesis-and-selection.md](references/synthesis-and-selection.md) |
| `audit` | 检查知识库是否改善判断、是否产生确认偏误或无效积累 | [knowledge-model.md](references/knowledge-model.md)、[synthesis-and-selection.md](references/synthesis-and-selection.md)、[workspace-adaptation.md](references/workspace-adaptation.md) |

完整读取所选参考文件，不要只读取片段。不要加载与当前模式无关的参考。

## 通用判断流程

### 1. 明确问题

先回答：

- 这份输入与哪个研究问题有关？
- 研究者目前怎样理解该问题？
- 当前最关键的不确定性是什么？

如果无法关联现有问题，判断它是在暴露新的重要维度，还是暂时无关。不要为了入库而勉强建立关系。

### 2. 判断可能影响

检查它是否可能改变以下任一内容：

- 问题定义；
- 概念或测量；
- 机制解释；
- 证据边界；
- 研究设计；
- 核心引用责任；
- 当前置信程度。

如果均不能改变，只保留来源位置或建议不处理。

### 3. 选择处理深度

使用 `L0–L4` 阅读深度。新近性、高引用和标题相似不能单独决定精读。详见 [paper-reading.md](references/paper-reading.md)。

### 4. 分析证据

按“材料或观测 → 分析结果 → 作者解释 → 进一步外推”拆开论证。判断证据支持到哪一层、依赖哪些条件、有哪些替代解释。详见 [evidence-judgment.md](references/evidence-judgment.md)。

### 5. 比较并挑战

把新输入与现有判断、相邻文献和最强反证比较。不要只问它支持什么，还要问它挑战、限定或重新定义了什么。

### 6. 形成更新提案

至少说明：

- 原有判断；
- 新材料改变或未改变了什么；
- 改变的证据依据；
- 当前置信程度及边界；
- 仍未解决的问题；
- 什么证据会再次改变判断。

不要伪造精确置信数字。只有研究者已经使用数值置信度时才沿用。

### 7. 决定是否写入

区分候选、来源笔记、更新提案和稳定知识。遵守工作区规则，优先更新既有问题页，避免为同一问题创建重复页面。详见 [workspace-adaptation.md](references/workspace-adaptation.md)。

### 8. 生成下一项证据需求

说明下一步缺少的是事实、机制、边界、反证还是方法验证，以及什么研究设计或材料能够减少该不确定性。若继续阅读的边际收益已低，明确建议转向综合、研究设计、数据检验或写作。

## 输出约定

按任务提供最小充分输出。

### 新材料分诊

输出材料位置、与当前问题的关系、可能改变的判断、建议阅读深度、优先核验部分，以及“现在读／稍后读／不读”的明确建议。

### 单篇阅读

输出研究问题、核心主张、论证链、证据层级、关键假设、最强反对意见、适用边界和认知更新。不要以章节摘要代替论证重建。

### 跨材料综合

输出共同问题、材料之间的关系、证据权重差异、当前最合理判断、不确定性和下一项证据需求。不要生成作者与年份队列。

### 知识库更新

先展示拟更新页面、拟修改判断、来源依据和仍需研究者决定的内容。获得确认后再写入高价值判断。

### 下一篇选择

先定义所需证据特征，再筛选候选；同时加入反方向材料和停止阅读标准。不要让最新、热门或推荐算法代替研究问题。

## 边界

- 不默认移动、删除、重命名或发布来源文件。
- 不把 AI 摘要当作可引用证据。
- 不把个人批注、会议意见或 Agent 推断冒充经验事实。
- 不因一篇论文而静默重写长期研究方向。
- 不把每个知识缺口自动转换成任务。
- 不维护投稿状态、通用任务列表或完整文件版本链。
- 不为低价值材料制造完整笔记。
- 不把“我认为”添加到通用总结前冒充个人判断变化。

## 完成标准

以以下问题验收工作：

- 研究问题是否比处理前更精确？
- 证据能够支持到哪一层是否更清楚？
- 当前判断为什么成立、如何可能被推翻是否可见？
- 最强反证是否得到公平处理？
- 下一项值得寻找的证据是否更具体？
- 研究者下次能否独立复现这次判断过程？
