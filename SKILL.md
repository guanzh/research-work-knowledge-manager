---
name: research-work-knowledge-manager
description: Use when any new research input (paper, data, code, notes, experiment, meeting, draft, web page, search result) enters the project. Evaluates impact on research goal, claims, evidence, gaps, and tasks. Maintains source registry, claim matrix, evidence matrix, gap register, and task backlog to continuously evolve the research project toward a defined output.
version: 2.5.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, knowledge-management, goal-driven, evidence-tracking, claim-matrix, gap-analysis, task-backlog]
    related_skills: [arxiv, deep-research, academic-paper, wildlife-manuscript-builder]
    aliases: [goal-driven-research-system]
---

# 目标驱动科研知识系统 (Goal-Driven Research Knowledge System)

> 原名 Research Work Knowledge Manager (RWKM)，v2.0 升级为围绕科研目标持续进化的知识系统。

## Overview

科研工作中，任何新资料——论文 PDF、数据集、代码、实验记录、组会纪要、论文草稿、网页、搜索结果、批注、合作者邮件——进入项目时，系统都执行同一组问题：

> 它从哪里来？它可信吗？它支持什么主张？它挑战什么？它补上什么缺口？它制造什么新问题？它能让论文哪一部分变好？它应该转化成什么任务？

**核心公式：**

```
research input + goal + claims + evidence + gaps + tasks + review
= evolving research project
```

**核心原则（v2.0 升级版）：**

> source_registry 负责登记每一个输入的身份、来源和可信度。
> claim_matrix 负责追踪每条论文主张有什么证据支持、有什么证据挑战。
> evidence_matrix 负责记录每个资料到底提供了什么可用的证据单元。
> gap_register 负责登记还缺什么。
> task_backlog 负责管理下一步要做什么。
> conflict_log 负责记录矛盾。
> decision_log 负责记录人的判断。
> wiki 负责整理可复用的知识页面。

**与 v1.x 的关系：** v1.x 的三层 ID（file_id / work_id / version_id）、版本追踪、冲突日志、决策日志、Wiki 知识页全部保留。v2.0 在此基础上新增了主张层（claim）、证据层（evidence）、缺口层（gap）和任务层（task），把系统从"管文件"升级为"管科研进化"。

## When to Use

**任何新资料进入时：**
- 论文 PDF / 数据集 / 代码 / 图表 / protocol / 草稿 → 走通用 8 步 pipeline
- 实验记录 / 组会纪要 / 导师反馈 / 合作者邮件 → 提取决策、反馈、待办
- 网页 / 搜索结果 / 外部报告 → 提取观点、来源、数据、风险
- 批注 / 读书笔记 → 提取个人判断、问题、想法

**查询和检查时：**
- 查询项目当前有效文件版本
- 检查论文草稿是否引用了过期数据或图表
- 评估一条 claim 的证据支持全景（literature + data + code + feedback）
- 生成科研变化周报（新资料 → 影响哪些 claim → 关闭哪些 gap → 生成哪些 task）
- 检测文件之间的版本冲突、数值冲突、来源冲突、证据冲突
- 投稿前完整一致性检查（source → claim → evidence → manuscript）

## 五表核心架构

> 完整 schema 和字段说明见 `references/five-core-tables.md`

v2.0 的核心数据结构。任何新资料进入后，最终都要写入这五张表：

| 表 | 文件 | 回答的问题 |
|----|------|-----------|
| **source_registry** | `registry/source_registry.jsonl` | 我有哪些资料？从哪里来？可信度如何？ |
| **claim_matrix** | `registry/claim_matrix.jsonl` | 我的论文主张是什么？有什么证据支持/挑战？ |
| **evidence_matrix** | `registry/evidence_matrix.jsonl` | 每个资料提供了什么可用的证据单元？ |
| **gap_register** | `registry/gap_register.jsonl` | 还缺什么？ |
| **task_backlog** | `registry/task_backlog.jsonl` | 下一步做什么？ |
| **submission_log** | `registry/submission_log.jsonl` | 这篇论文投到哪里了？审稿到哪一轮了？ |

进化逻辑：
```
new source → affects claims → provides evidence → closes/opens gaps → creates tasks → improves manuscript
```

### source_type（13 种输入类型）

| source_type | 典型格式 | 主要提取内容 |
|-------------|---------|-------------|
| `literature_pdf` | PDF | 方法、结论、引用价值、局限 |
| `web_page` | 网页/博客 | 观点、来源、时间、链接 |
| `dataset` | CSV/Excel/RData | 变量、样本量、版本、质量 |
| `code` | .R/.py/.Rmd | 方法流程、输入输出、依赖 |
| `experiment_log` | 实验/观察记录 | 条件、过程、结果、异常 |
| `meeting_note` | 组会/导师/访谈 | 决策、反馈、待办 |
| `manuscript_draft` | .docx/.md | 主张、结构、证据缺口 |
| `figure` | .png/.jpg/.pdf | 来源数据、生成方法、意图 |
| `search_result` | 搜索结果集 | 候选文献、线索、待筛选 |
| `annotation` | 批注/读书笔记 | 个人判断、问题、想法 |
| `email_or_message` | 邮件/聊天 | 决策、要求、修改建议 |
| `external_report` | 技术报告/政策 | 背景、数据、论证、风险 |
| `audio_transcript` | 访谈/会议转写 | 观点、证据、待确认 |

> `source_type` 是资料的**原始形态**，`content_type` 是资料的**科研用途分类**。两者独立，例：source_type=dataset, content_type=camera_trap_data

### content_type 扩展（生态统计专用）

以下为 `content_type` 的可能取值，用于更精确标注 `source_type = dataset` 或 `code` 的科研用途：

- `camera_trap_data` — 红外相机照片/视频及标注
- `acoustic_monitoring_data` — 被动声学录音及频谱
- `transect_survey_data` — 样线/样点调查记录
- `occupancy_model_results` — occupancy/detection 模型输出
- `multi_species_abundance_model` — 多物种丰度/群落模型结果
- `bayesian_hierarchical_model` — 贝叶斯分层模型（如 HMSC）输出
- `distance_sampling_results` — distance sampling 密度估计
- `species_distribution_model` — SDM/栖息地适宜性结果
- `genetic_data` — eDNA/metabarcoding/微卫星数据
- `remote_sensing_data` — 卫星/无人机遥感数据
- `patrol_data` — 巡护/威胁监测记录
- `behavioral_data` — 行为观察/活动节律数据

### 保留的 v1.x 表

以下从 v1.x 继承，继续使用：work_registry.jsonl、version_registry.jsonl、conflict_log.md、decision_log.md

### source_registry 关键字段

```json
{
  "file_id": "F20260608001",
  "source_type": "dataset",
  "provenance": {"origin": "field_collection", "collected_by": "team", "processing": "raw"},
  "linked_claims": ["C001"],
  "linked_gaps": ["G002"],
  "linked_tasks": ["T021"],
  "impact_on_goal": "high"
}
```

### claim_matrix 示例

```json
{
  "claim_id": "C001",
  "statement": "Method A improves measurement reliability under condition B.",
  "status": "weakly_supported",
  "supporting_sources": [{"file_id": "F...", "source_type": "dataset"}],
  "challenging_sources": [{"file_id": "F...", "source_type": "meeting_note"}],
  "evidence_strength": "medium"
}
```

### evidence_matrix 示例

```json
{
  "evidence_id": "E001",
  "source_file_id": "F...",
  "evidence_statement": "Dataset V002 shows Method A has lower variance than baseline.",
  "related_claim": "C001",
  "evidence_direction": "supports",
  "reliability": "needs_review"
}
```
### 第六张表：submission_log（投稿追踪）

| 表 | 文件 | 回答的问题 |
|----|------|-----------|
| **submission_log** | `registry/submission_log.jsonl` | 这篇论文投到哪里了？审稿到哪一轮了？ |

```json
{
  "submission_id": "S20260609001",
  "manuscript_work_id": "W_gibbon_acoustic_2025",
  "journal": "Biological Conservation",
  "status": "drafting|submitted|under_review|revised|accepted|rejected",
  "round": 1,
  "submitted_at": "2026-06-09",
  "decision": "major_revision|minor_revision|accept|reject|null",
  "decision_at": null,
  "review_files": ["F20260609021", "F20260609022"],
  "response_file": null,
  "linked_claims": ["C010", "C011", "C015"],
  "notes": ""
}
```

- `manuscript_work_id`：论文是一个长期 work 对象
- `linked_claims`：该论文中最核心的 3–5 条主张，方便从投稿记录直接跳回 claim_matrix
- Agent 规则：用户说"这篇稿子投了 X 期刊"时，自动创建 submission_log 条目；审稿意见返回时，review_files 登记审稿文件

### manuscript_draft 的投稿相关字段增强

对 `source_type = manuscript_draft` 记录增加可选字段：

```json
"manuscript_meta": {
  "intended_journal": "Biological Conservation",
  "stage": "draft|submitted|revised|accepted|rejected",
  "linked_submission_id": "S20260609001"
}
```

## 三层 ID 体系（从 v1.x 继承）

| ID 类型 | 含义 | 格式示例 | 说明 |
|---------|------|----------|------|
| `file_id` | 具体物理文件 | `F20260608001` | 每个文件唯一，不可变 |
| `work_id` | 长期变化的科研对象 | `W_alpha_protocol` | 跨越版本的逻辑实体 |
| `version_id` | work 的第 N 版 | `V003` | 递增，记录演进关系 |

查询时 agent 必须查 source_registry + work_registry + version_registry，不应凭记忆或文件名猜测。

## 通用 8 步 Pipeline（v2.0 核心流程）

任何新资料进入后，都走同一 pipeline：

```
ingest → classify → extract → evaluate → link → update → plan → review
```

| 步骤 | 做什么 | 产出 |
|------|--------|------|
| **ingest** | 登记资料，生成 file_id，计算 checksum，**先与已有 registry 条目比对 checksum**，记录 provenance，删除 inbox 原始文件 | source_registry 新条目 |
| **classify** | 判断 source_type（13 种之一）、content_type、所属 project | source_registry 字段填充 |
| **extract** | 提取主张、数据、方法、结论、限制、问题 | 临时 extracted_units |
| **evaluate** | 判断可信度、相关性、时效性、impact_on_goal | impact 评估 |
| **link** | 关联到 claim / gap / task / wiki / manuscript 章节 | linked_* 字段 |
| **update** | 更新 claim_matrix → evidence_matrix → gap_register → task_backlog → wiki → conflict_log | 5 表 + wiki 更新 |
| **plan** | 生成下一步任务 | task_backlog 新条目 |
| **review** | 标记需要人类判断的地方，输出 Evolution Report | decision_log + Evolution Report |

### 统一输出：Evolution Report

无论什么类型资料，pipeline 结束后输出统一报告：

```markdown
# Research Input Evolution Report

## New Input
file_id, source_type, title, path

## Provenance
origin, collected_by, collected_at, processing

## Relevance to Goal: high/medium/low

## Extracted Units
claims, evidence, methods, limitations, risks, tasks

## Impact
claims strengthened/weakened, gaps closed/opened, tasks created

## Manuscript Impact
sections affected, suggested changes

## Human Review Needed
- [ ] D008: ...
```

> 填充示例见 `assets/evolution-report-example.md`。Agent 生成报告时应参照此例的粒度和风格：每个字段给具体值而非占位符，Human Review Needed 用 Dxxx 编号。

## 不同 source_type 的专项处理

以下为 13 种 source_type 在 pipeline 的 extract 和 link 步骤中的专项规则。每种包含提取内容、关键处理规则、典型警惕项。

### dataset
- **提取**: 数据来源、版本、变量列表、样本量、缺失值、异常值、处理流程、对应 code file_id
- **处理**: 建立 provenance 链 generated_by → cleaned_by → analyzed_by → used_in_figure → used_in_claim
**警惕**：不建立 provenance 链的 dataset 无法追溯分析结果来源
- **重复 ingest（checksum 比对）**：ingest 前先计算新文件的 checksum，与 source_registry 中已有条目的 checksum 比对。若匹配，跳过 ingest 并告知用户文件已存在。不比对则可能在 workspace 重组后重复登记同一文件（仅路径不同）。

### code
- **提取**: 代码目的、输入/输出 file_id、依赖环境、对应数据版本、生成图表、支持结果
- **处理**: 代码不与数据孤立登记，建立 code↔data↔figure↔result 关系
- **警惕**: 登记 code 时只看文件名不读内容，遗漏依赖关系

#### R 生态专项：renv / targets 支持

当 `source_type = code` 且文件位于 R 项目目录中时：

1. 检测 `renv.lock`：
   - 若存在，计算 checksum 并写入该 code 文件的 provenance 中：
   ```json
   "provenance": {
     "origin": "local_repo",
     "env_lockfile": "F20260609099"
   }
   ```
   - 同时登记 renv.lock 本身为 source_type=code 的独立条目

2. 检测 `_targets.R` 或 `_drake.R`：
   - 读取文件，识别 `tar_target()` 或 `drake_plan()` 中定义的 target 名称
   - 每个 target 生成一个 work_id（如 `W_acoustic_model_target_occupancy`）
   - dataset / figure 的 provenance 中增加字段：
   ```json
   "generated_by_target": "W_acoustic_model_target_occupancy"
   ```
   - 目的：未来回看结果时，可追溯到 "哪条 pipeline 的哪个 target 在什么包环境下产出"

### meeting_note
- **提取**: 决定→decision_log, 质疑→claim_matrix.challenging, 待办→task_backlog
- **处理**: 会议记录同时影响 decision_log + claim_matrix + gap_register + task_backlog
- **警惕**: 只提取待办事项，忽略会议中提出的质疑（应写入 challenging_sources）

### manuscript_draft
- **提取**: 论文结构、核心论点、缺失引用、过强表述、证据不足段落
- **处理**: 进入后立即一致性检查：引用是否过期、与 wiki 是否一致
- **警惕**: 草稿进入后只 regist 不做检查，导致过期引用残留

#### RMarkdown / Quarto 一体化

当 `source_type = manuscript_draft` 且文件扩展名为 `.Rmd` 或 `.qmd` 时：

- 自动视为**复合 source**，同时在 source_registry 中登记两条：
  - 一条 `source_type = manuscript_draft`（追踪论文内容）
  - 一条 `source_type = code`（追踪运行该文档产生的 figures/tables）
- 两者共享同一个 `work_id`，通过 `linked_code` / `linked_manuscript` 字段互相关联
- code 条目中记录：knitr/Quarto 版本、使用的 R packages、生成的 figure/table file_id 列表

### search_result
- **提取**: 搜索问题/日期/关键词、来源链接（保存快照）、筛选标准、可信度
- **处理**: 网页易变，保存快照和访问日期
- **警惕**: 仅保存链接不保存快照，后续链接失效无法回溯

### literature_pdf
- **提取**: 研究问题、方法、核心结论、引用价值、局限
- **处理**: 关联到 claim 作为 supporting_sources 或 challenging_sources
- **警惕**: 只记 supporting 不记 challenging，破坏证据平衡

### web_page
- **提取**: 观点、来源可信度、发布时间、原始链接
- **处理**: 同 search_result，保存快照和访问日期
- **警惕**: 不判断来源可信度（个人博客 vs 机构报告）

### experiment_log
- **提取**: 条件、过程、结果、异常、设备 ID、操作者、参数设置、运行时间戳、输入数据 file_id、输出数据 file_id
- **处理**: 关联到对应 dataset 和 code，建立实验→数据→分析链条。**自动 provenance 追踪**：将实验提取的定量结果（效应量、p 值、置信区间、模型选择指标）写入 evidence_matrix，自动匹配到 claim_matrix 中的相关主张（匹配规则见 `references/experiment-provenance.md`）。实验异常同样登记为 evidence（direction='challenges' 或 reliability='needs_review'）。
- **provenance 必填字段**：
  ```json
  "experiment_provenance": {
    "method": "occupancy_model | distance_sampling | glm | hmsc | ...",
    "parameters": {"key": "value"},
    "input_data": ["F20260608001"],
    "output_data": ["F20260608002"],
    "runtime_env": "R 4.3.2 / unmarked 1.3.0",
    "run_timestamp": "2026-06-08T14:30:00"
  }
  ```
- **警惕**: 实验记录中的异常被忽略（可能是重要发现）；实验结果直接当作 claim 而跳过 evidence_matrix 登记

### figure
- **提取**: 来源数据 file_id、生成方法（代码/工具）、图表意图
- **处理**: provenance 中必须包含 generated_by_code 和 source_data
- **警惕**: 图表只登记不追溯来源数据，无法复现

### annotation
- **提取**: 个人判断、问题、想法、与 claim 的关联
- **处理**: 个人批注是"待验证"级别，不直接作为 evidence
- **警惕**: 批注直接当作证据使用

### email_or_message
- **提取**: 决策、要求、修改建议、截止日期
- **处理**: 决策→decision_log，要求→task_backlog，修改建议→claim 或 manuscript
- **警惕**: 非正式沟通中的建议直接改写 claim

### external_report
- **提取**: 背景、数据、论证、风险、作者/机构立场
- **处理**: 标注来源可信度和潜在 bias
- **警惕**: 技术报告当作 peer-reviewed 论文级别引用

### audio_transcript
- **提取**: 观点、证据、待确认事项、说话人角色
- **处理**: 转写内容属"待确认"级别，关键信息需与说话人核实
- **警惕**: 转写错误导致错误引用观点

## 日常使用任务

### 新资料进入
> "用 Goal-Driven Research System 处理这个新资料。"

Agent 执行完整 8 步 pipeline，输出 Evolution Report。

### 查询证据全景
> "C001 现在有什么证据支持？"

Agent 查 claim_matrix → 列出所有 supporting_sources 和 challenging_sources（含 source_type 标注）。

### 评估影响
> "这个新数据集对论文有什么影响？"

Agent 走 pipeline → 输出 Evolution Report 的 Impact 段。

### 一致性检查
> "投稿前检查。"

Agent 检查：source_registry 版本链完整 → claim_matrix 每条有证据 → evidence_matrix 的 reliability → manuscript_draft 引用均为 active → conflict_log 清空 → decision_log 清空。

### 生成周报
> "本周科研变化。"

Agent 列出：新增 source → 影响的 claim → 关闭的 gap → 新开的 gap → 创建的 task → pending decision。

### 投稿前一致性检查

> 「对 `W_xxx` 这篇稿子做投稿前一致性检查。」

Agent 步骤：

1. 从 work_registry 找到 W_xxx 对应的 active manuscript_draft file_id
2. 从 claim_matrix 拉出所有 status != 'unsupported'、且被该稿子引用的 claim
3. 逐条检查：
   - 每个 claim 是否至少有一个 supporting evidence（evidence_direction = 'supports'）
   - 是否存在 evidence 标记 reliability='needs_review' 但已在正文中被引用为事实
   - figure/table 的 source_file_id 是否均为 active version（version_registry 中 status='active'）
   - 引用文献是否在 source_registry 中均有登记（无 ghost citations）
4. 输出 `outputs/pre_submission_check_W_xxx_YYYYMMDD.md`：

```markdown
# 投稿前一致性检查 — W_example

## Claims Without Supporting Evidence
- C012: "..." — 无 supporting evidence

## Unreliable Evidence Used as Fact
- E008 (reliability=needs_review) 被引用于 Discussion ¶3

## Stale Figures/Tables
- F20250501015 (Figure 2): version V002 非 active，当前 active 为 V004

## Ghost Citations
- Smith et al. 2023: 在正文引用但未在 source_registry 登记

## Overall: ❌ 不通过 — N issues need resolution before submission
```

## 目录结构（v2.0）

```
research_workspace/
├── inbox/              # 新资料暂存
├── sources/            # 原始资料正式存储
├── registry/           # 六张表 + 三个日志
│   ├── source_registry.jsonl
│   ├── work_registry.jsonl
│   ├── version_registry.jsonl
│   ├── claim_matrix.jsonl
│   ├── evidence_matrix.jsonl
│   ├── gap_register.jsonl
│   ├── task_backlog.jsonl
│   ├── submission_log.jsonl
│   ├── conflict_log.md
│   ├── decision_log.md
│   └── impact_log.md
├── wiki/               # 知识页面
├── outputs/            # 生成物（Evolution Reports）
├── releases/           # 冻结版本
└── archive/            # 历史文件
```

## 最小可行启动 (MVP)

> **核心原则：如果目标文件夹已有文件，自动扫描并 ingest，不能忽略。**

最少先建立：

```
registry/source_registry.jsonl     # 空数组 []
registry/work_registry.jsonl       # 空数组 []  — 三层 ID 体系必需
registry/version_registry.jsonl    # 空数组 []  — 三层 ID 体系必需
registry/claim_matrix.jsonl        # 空数组 []
registry/evidence_matrix.jsonl     # 空数组 []
registry/gap_register.jsonl        # 空数组 []
registry/task_backlog.jsonl        # 空数组 []
registry/submission_log.jsonl       # 空数组 []
registry/conflict_log.md           # "# Conflict Log"
registry/decision_log.md           # "# Decision Log"
registry/impact_log.md             # "# Impact Log"
wiki/index.md                      # Wiki 索引
```

**建立目录后，立即执行：**
0. **扫描目标文件夹**：用 `os.walk()` 列出目标文件夹中所有已有文件（排除新创建的 registry/ 和 wiki/ 目录自身）。
   - 如果 **无已有文件** → 正常启动，等待新资料进入 inbox。
   - 如果 **有 1–9 个已有文件** → 走下面第 0a 步"简单 ingest"。
   - 如果 **有 10+ 个已有文件或包含完整手稿** → 提示用户走下方"成熟项目批量初始化"流程（更高效）。
0a. **简单 ingest（1–9 个文件）**：逐个文件走通用 8 步 pipeline 的 ingest + classify 两步——生成 file_id，填入 source_registry，判断 source_type，记录 provenance，将文件移到 sources/。不提取 claim/evidence/gap（文件量少，且用户没有明确要求深入分析时，先 regist 即可，后续走正常 pipeline 逐步提取）。
0b. **写入 impact_log**：记录初始化时自动 ingested 的几个文件。

> 更多渐进采用路径见 `assets/adoption-self-assessment.md`

### 成熟项目批量初始化（文件夹已有文件时的通用流程）

> **适用范围：目标文件夹已有任何文件（不限于"成熟项目"），且用户希望一次性完成全部 ingest。**

当目标文件夹已有文件时（无论文件数量多少），不走逐个 source 的 8 步 pipeline——因为逐个 ingest 效率低、输出碎片化。应走**批量初始化流程**，一次性完成所有文件的登记和处理：

1. **创建目录骨架**：同 MVP，建立 inbox/sources/registry/wiki/outputs/releases/archive
2. **批量扫描全部文件**：用 `os.walk()` 列出所有文件，按 source_type 归类（manuscript_draft / dataset / code / figure / annotation / meeting_note 等）
3. **确定 version 链**：从文件名（v3→v4→v5）和目录结构推断手稿和分析流程的版本演进，创建 work_registry 和 version_registry
4. **提取全部 claims**：如果包含手稿，读取当前 active 手稿全文，从摘要、引言研究问题、结果、讨论和结论中提取论文级主张（通常 6–12 条），一次性写入 claim_matrix。每条 claim 标注 evidence_strength（根据是否有定量数据支撑判断 strong/medium/low）。如果无手稿，跳过此步。
5. **提取全部 evidence**：从 analysis_summary.json（或等价汇总文件）、evidence_chain.md 和手稿结果表中提取证据单元，一次性写入 evidence_matrix。标注 reliability（verified = 有定量数字可检验 / needs_review = 仅手稿论证）。如果无上述文件，跳过此步。
6. **提取全部 gaps**：交叉阅读手稿的"待补充"段落、evidence_chain 中的审稿风险预判和数据契约，按 manuscript_completeness / method_description / data_completeness / analysis_gap / decision_pending 分类，一次性写入 gap_register
7. **创建 tasks**：从 gaps 反向生成 task（每个 high-priority gap 生成 1–2 个 task），一次性写入 task_backlog
8. **写入 decision_log**：需要作者判断的事项（目标期刊、数据脱敏方案、异常值处理等）写入 decision_log
9. **更新 impact_log**：记录初始化操作
10. **输出 Evolution Report**：一次性生成综合报告，包含所有注册表统计、claim/evidence/gap 全景、优先级排序的下一步行动

**初始化后自动 ingest 的后续处理**：已 ingest 的文件应移入 `sources/`（或按 workspace 重组流程处理），确保 `inbox/` 干净。

**关键区别 vs 8 步 pipeline**：
| 维度 | 单源 8 步 pipeline | 成熟项目批量初始化 |
|------|-------------------|-------------------|
| 输入 | 1 个新文件 | 10–50+ 个已有文件 |
| claim 提取 | 从新资料提取 1–3 条影响 | 从手稿全文提取全部 claim (6–12) |
| 版本链 | 不涉及 | 必须从文件名推断并创建 |
| evidence 提取 | 单源证据 | 从分析汇总+证据链批量提取 |
| 执行策略 | 串行逐 source | 并行一次性填充所有 5 表 |
| 输出 | 每次 1 份简短报告 | 1 份综合全景报告 |

**警惕**：
- 不要逐个文件走 8 步 pipeline（会有 30+ 轮输出，信息碎片化）
- 不要跳过 work_registry/version_registry（成熟项目一定有版本演进历史）
- claim 提取不要只从摘要抄——要通读引言（研究问题）、结果（定量结论）和讨论（主张边界）全段
- gaps 提取不要只看手稿"待补充"——evidence_chain 的审稿风险和数据契约是同等重要的缺口来源

## Workspace 重组流程（已存在混乱工作区的整理）

> 具体输出模板见 `references/workspace-reorganization-example.md`
> v1.0→v2.0 升级的 Python 执行模式见 `references/v2-upgrade-execution-patterns.md`

1. 扫描全景 → 2. 识别分析包 → 3. 请求作者确认当前版本 → 4. 建立目录骨架 → 5. 移动原始数据到 sources/ → 6. 归档旧包 → 7. 设当前包 → 8. 登记所有文件到 source_registry → 9. 建 wiki 和 claim_matrix → 10. 清理残留
10. （若从 v1.0 升级到 v2.0）追加步骤：创建 inbox/outputs/releases → 升级 file_registry 为 source_registry → 创建 claim/evidence/gap/task 四表 → 回填 linked_* → 更新 wiki 引用 → 升级 impact_log

## Agent 执行约束

1. **先查 registry 再回答**：涉及文件状态、版本、关系的查询，必须读 registry，不能凭记忆或文件名猜测
2. **只读来源库**：`sources/` 中的原始文件默认只读，不直接改写
3. **不自行决定**：遇到版本取舍、数据矛盾等需要专业判断的事项，写入 decision_log 等待用户决定
4. **每次操作追加 impact_log**：记录时间、来源、影响维度（claims/gaps/tasks）
5. **Wiki 必须引用 file_id 和 claim_id**：不写「某个文件」「之前的数据」等模糊表述
6. **Windows 非 ASCII 路径**：含中文路径时用 `execute_code` + Python 完成文件操作。获取文件名一律用 `os.listdir()` 而非手写字符串（中文引号 `""` 与 ASCII `"` 肉眼不可区分），ingest 成功后用 `os.remove()` 删除 inbox 中的原始文件。
7. **Memory 写入策略**：完成完整 pipeline 后写入 Memory，但仅写入项目级惯例（workspace 路径映射、敏感数据标记习惯、常用 journal target），不写入具体 file_id / claim_id / task_id 列表，避免 Memory 与 registry 双重来源冲突。

### 人类决策 vs Agent 决策

| 场景 | 由 Agent 决定 | 写入 decision_log 等待人 |
|------|:---:|:---:|
| 解析 source_type | ✅ | |
| 判断数据质量低 | | ✅（记录 Dxxx） |
| 合并两个版本的分析脚本 | | ✅ |
| 小幅重命名 wiki 条目 | ✅（但写 impact_log） | |
| 确定 claim 的 evidence_strength | | ✅ |
| 生成 file_id | ✅ | |
| 决定 supersede 旧版本 | | ✅ |
| 提取 meeting_note 中的待办 | ✅ | |
| 选择 journal target | | ✅ |
| 关闭 gap（认定缺口已解决） | | ✅ |
| 分类 content_type | ✅ | |
| 判断 source 可信度 | | ✅（记录评估依据） |

## Hermes Integration

### Memory：写入什么，不写什么

每次完成一次完整 pipeline 后，向 Hermes Memory 写入项目级惯例，不写入具体数据。

**写入 Memory（惯例/偏好）**：
- workspace 路径 → project 映射
- 用户偏好的敏感数据标记习惯
- 常用 journal target
- 特定 work_id 的主要 data source 类型（如 "W_gibbon_acoustic_2025 的主要 data source 是 acoustic + camera trap"）

**不写入 Memory（避免与 registry 双重来源冲突）**：
- 具体 file_id 列表
- claim_id 列表
- task_id 列表
- version 链细节

后续 session 自动加载已写入的惯例，无需用户重复说明。

### Cron：每周自动 Lint + 周报

配置 Cron job（Hermes cronjob 工具），每周自动生成 workspace 健康报告：

**调度**: 每周一 09:00
**skill**: `goal-driven-research-system`
**enabled_toolsets**: [terminal, file]

#### 检查规则

1. **孤立资料 (Orphan Sources)**
   - 扫描 source_registry 中 linked_claims=[] AND linked_gaps=[] AND linked_tasks=[] 的条目
   - 输出 file_id + source_type + title

2. **悬置主张 (>90d)**
   - 扫描 claim_matrix 中 status='unsupported' 且 last_updated 距今 > 90 天的条目
   - 输出 claim_id + statement + last_updated

3. **陈旧缺口 (>180d)**
   - 扫描 gap_register 中 linked_tasks=[] 且 created_at 距今 > 180 天的条目
   - 输出 gap_id + description + created_at

4. **待决策 (>30d)**
   - 扫描 decision_log 中 status='pending' 的条目，检查 created_at
   - 输出 decision_id + description + created_at

5. **断链版本**
   - 扫描 version_registry，检查每个 work_id 的版本链是否连续
   - 输出 version 断链的 work_id

#### 周报输出格式

输出：`outputs/weekly_health_report_YYYYMMDD.md`

```markdown
# Weekly Health Report – YYYY-MM-DD

## 1. Orphan Sources
- F20260601023 (meeting_note): "项目组会议 6/1" — 未链接到任何 claim/gap/task

## 2. Aging Unsupported Claims (>90d)
- C023: "Acoustic model outperforms human observers" — last updated: 2025-12-01

## 3. Aging Gaps Without Tasks (>180d)
- G010: "缺乏干季声学检测率对比数据" — created: 2025-11-15

## 4. Pending Decisions (>30d)
- D008: "是否采用 occupancy 模型作为主分析" — created: 2026-04-20

## 5. Broken Version Chains
- W_acoustic_model_2025: V003 references V001 as parent (V002 missing)

## 6. Suggested Next Actions
- [ ] 审查 C023 是否仍然重要，或降级并归档
- [ ] 为 G010 创建 fieldwork 任务
- [ ] 修复 W_acoustic_model_2025 版本链
```

### Curator：skill 自身版本演化

RWKM skill 使用 curator 追踪版本变更：
- 每次对五表 schema 或 pipeline 步骤做重大修改时，更新 version 字段
- 在 registry/impact_log.md 写入一条 "skill_structure" 记录
- curator 可追踪高频触发的修正类型和几乎不用的功能分支

### Session Search：回溯历史决策

需要了解历史决策时，用 `session_search` 而非重读整个 decision_log。
搜索关键词：决策 ID (Dxxx)、claim ID、work_id。

## Common Pitfalls

1. **只登记不评估影响**：登记了 source 但没有走 extract→evaluate→link 步骤，变成单纯的文件清单
2. **claim 和 evidence 混在一起**：claim 是论文级别主张，evidence 是资料级别证据单元——一个 claim 可以有多个 evidence
3. **忽略 challenging_sources**：只记录支持的证据，不记录挑战的证据
4. **gap 和 task 不分**：gap 是"缺什么"（名词），task 是"做什么"（动词）
5. **靠文件名判断版本**：必须查 registry 中的 version_id 和 status
6. **不记录冲突**：发现不一致时私下修正而不写入 conflict_log
7. **替用户做决定**：需要专业判断的事项写入 decision_log，不自行裁决
8. **Terminal 中文路径失败**：用 `execute_code` + Python `os`/`shutil`/`hashlib`
   - **特别注意中文引号**：文件名含 `""`（Unicode 全角引号）时，手写字符串常错用 ASCII `"` 导致 `FileNotFoundError`。解决：一律用 `os.listdir()` 获取精确文件名再操作，不要凭肉眼写出路径字符串。
9. **Workspace 重组后分析脚本路径断裂**：重组将文件从项目根目录（如 `鸣叫记录表(1)(1)/`）移到 `sources/` 后，已有分析脚本（`.R`/`.py`/`.Rmd`）中硬编码的 `DATA_DIR` 路径会静默失效，指向不存在的位置或旧数据。重组后必须检查并更新分析脚本中的 `DATA_DIR` 路径，验证路径存在后再运行。

## Verification Checklist

- [ ] source_registry.jsonl 每行合法 JSON，file_id 唯一
- [ ] claim_matrix 每条 claim 至少有 1 个 supporting 或 challenging source
- [ ] evidence_matrix 每条 evidence 指向有效 claim_id
- [ ] gap_register 中 closed gap 有关闭原因和日期
- [ ] task_backlog 中 pending task 有明确 assignee 和 priority
- [ ] 所有 active source 的 superseded_by 为 null
- [ ] version_registry 版本链连续无断裂
- [ ] conflict_log 每个冲突有唯一 ID 和状态
- [ ] decision_log 每个待决策有唯一 ID，不混入已自行处理的内容
- [ ] wiki 页面引用均使用 file_id 或 claim_id
- [ ] 每次新资料进入后 impact_log 有对应记录

> **升级计划**: v2.5 升级方案见 `references/v2.5-upgrade-plan.md`