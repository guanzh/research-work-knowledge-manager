# Five Core Tables

v2.0 的核心数据结构。任何新资料进入后，最终都要写入这五张表。

## 1. source_registry（来源注册表）

继承 v1.x 的 file_registry.jsonl，扩展字段如下。

```json
{
  "file_id": "F20260608001",
  "work_id": "W_alpha_protocol",
  "version_id": "V003",
  "title": "Alpha dataset V003",
  "path": "sources/datasets/F20260608001_alpha_v003.csv",
  "file_type": "csv",
  "source_type": "dataset",
  "content_type": "camera_trap_data",
  "project": "alpha_project",
  "status": "active",
  "checksum": "sha256_value",
  "summary": "Updated dataset with outlier rule applied.",
  "sensitivity": "internal",
  "sensitivity_note": "Contains exact survey site coordinates — mask before public use.",
  "provenance": {
    "origin": "field_collection",
    "collected_by": "team_alpha",
    "collected_at": "2026-03",
    "processing": "raw"
  },
  "linked_claims": ["C001", "C003"],
  "linked_gaps": ["G002"],
  "linked_tasks": ["T021", "T022"],
  "impact_on_goal": "high",
  "supersedes": ["F20260528004"],
  "superseded_by": null,
  "ingested_at": "2026-06-08T10:30:00Z"
}
```

### 新增字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `source_type` | string | 13 种之一（见下方） |
| `provenance` | object | 来源链：origin, collected_by, collected_at, processing |
| `linked_claims` | string[] | 关联的 claim_id 列表 |
| `linked_gaps` | string[] | 关联的 gap_id 列表 |
| `linked_tasks` | string[] | 关联的 task_id 列表 |
| `impact_on_goal` | string | high / medium / low — 对当前科研目标的影响程度 |

### source_type 枚举（v2.0 扩展为 13 种）

| source_type | 典型格式 | 主要提取内容 |
|-------------|---------|-------------|
| `literature_pdf` | PDF | 方法、结论、引用价值、局限 |
| `web_page` | 网页/博客 | 观点、来源、时间、链接 |
| `dataset` | CSV/Excel/RData/Rds | 变量、样本量、版本、质量问题 |
| `code` | .R/.py/.Rmd/.ipynb | 方法流程、输入输出、依赖 |
| `experiment_log` | 实验/观察记录 | 条件、过程、结果、异常 |
| `meeting_note` | 组会/导师/访谈笔记 | 决策、反馈、待办 |
| `manuscript_draft` | .docx/.md | 主张、结构、证据缺口 |
| `figure` | .png/.jpg/.pdf | 来源数据、生成方法、表达意图 |
| `search_result` | 搜索结果集合 | 候选文献、线索、待筛选 |
| `annotation` | 批注/读书笔记 | 个人判断、问题、想法 |
| `email_or_message` | 邮件/聊天记录 | 决策、要求、修改建议 |
| `external_report` | 技术报告/政策文件 | 背景、数据、论证、风险 |
| `audio_transcript` | 访谈/会议转写 | 观点、证据、待确认 |

### source_type vs content_type

- `source_type`：资料的**原始形态**（是什么格式/来源进来的）
- `content_type`：资料的**科研用途分类**（camera_trap_data / acoustic_data / protocol 等）

例：一个 .csv 文件，source_type = `dataset`，content_type = `camera_trap_data`

---

## 2. claim_matrix（主张矩阵）

追踪每条论文主张的证据支持全景——不仅来自文献，还来自数据、代码、实验、讨论反馈。

```json
{
  "claim_id": "C001",
  "statement": "Method A improves measurement reliability under condition B.",
  "status": "weakly_supported",
  "manuscript_section": "Results",
  "supporting_sources": [
    {"file_id": "F20260608001", "source_type": "literature_pdf", "note": "prior study supports similar mechanism"},
    {"file_id": "F20260609002", "source_type": "dataset", "note": "preliminary result shows lower variance"},
    {"file_id": "F20260610003", "source_type": "experiment_log", "note": "repeated trial supports the effect"}
  ],
  "challenging_sources": [
    {"file_id": "F20260611004", "source_type": "web_page", "note": "alternative explanation suggested"},
    {"file_id": "F20260612005", "source_type": "meeting_note", "note": "advisor questioned sample size"}
  ],
  "evidence_strength": "medium",
  "weakness": "Dataset size is still small.",
  "next_task": "T021",
  "updated": "2026-06-08"
}
```

### claim status 枚举

| status | 含义 |
|--------|------|
| `strongly_supported` | 多源证据一致支持，无严重挑战 |
| `weakly_supported` | 有部分支持但证据不足或存在挑战 |
| `contested` | 支持和挑战相当，待解决 |
| `speculative` | 仅有推理，无实证 |
| `dropped` | 已放弃 |

---

## 3. evidence_matrix（证据矩阵）

记录"每个资料到底提供了什么可用的证据单元"。

```json
{
  "evidence_id": "E001",
  "source_file_id": "F20260609002",
  "source_type": "dataset",
  "evidence_statement": "Dataset V002 shows that Method A has lower variance than baseline under condition B.",
  "related_claim": "C001",
  "evidence_direction": "supports",
  "evidence_strength": "medium",
  "reliability": "needs_review",
  "limitations": "Sample size small (n=118). Outlier rule not finalized.",
  "manuscript_use": "Potentially useful for Results section.",
  "next_action": "Check whether analysis code V004 reproduces this result.",
  "created_at": "2026-06-08T10:30:00Z"
}
```

### 边界：claim vs evidence

- **claim**：论文级别的论点（"Method A is better"）
- **evidence**：资料级别的证据单元（"dataset X shows variance ratio=0.73"）

一个 claim 可以有多个 evidence（来自不同 source），一个 evidence 只支持一个 claim。如果同一证据同时影响多个 claim，拆成多条 evidence 记录。

---

## 4. gap_register（缺口登记表）

追踪"还缺什么"。

```json
{
  "gap_id": "G002",
  "description": "Missing control group for comparison under condition C.",
  "type": "data_gap",
  "affects_claims": ["C001"],
  "severity": "blocking",
  "potential_sources": ["collaborator_lab_data"],
  "status": "open",
  "created_at": "2026-06-08"
}
```

### gap type 枚举

| type | 含义 |
|------|------|
| `data_gap` | 缺少数据 |
| `method_gap` | 缺少分析方法 |
| `literature_gap` | 缺少文献支撑 |
| `evidence_gap` | 有 claim 但无证据 |
| `analysis_gap` | 有待完成的分析 |

---

## 5. task_backlog（任务积压表）

追踪"下一步做什么"。

```json
{
  "task_id": "T021",
  "description": "Validate missing value handling in dataset V002.",
  "triggered_by": "F20260609002",
  "affects": ["claim_C001", "evidence_E001"],
  "priority": "high",
  "status": "pending",
  "assigned_to": "author",
  "created_at": "2026-06-08"
}
```

### task priority

| priority | 含义 |
|----------|------|
| `blocking` | 不完成就无法继续 |
| `high` | 本周应完成 |
| `medium` | 本月应完成 |
| `low` | 有空再做 |

---

## 五表之间的关系

```
source_registry ──(linked_claims)──► claim_matrix
       │                                    │
       │(linked_gaps)                       │(supporting/challenging)
       ▼                                    ▼
  gap_register ◄──────────────────  evidence_matrix
       │                                    │
       │                                    │(next_action)
       ▼                                    ▼
  task_backlog ◄────────────────────────────┘

进化逻辑:
  new source
    → affects claims (support or challenge)
    → provides evidence (evidence_matrix)
    → closes or opens gaps (gap_register)
    → creates tasks (task_backlog)
    → improves manuscript
```

## 何时创建/更新各表

| 表 | 创建时机 | 更新时机 |
|----|---------|---------|
| source_registry | 每次新资料进入（ingest 步骤） | 替代关系变化时 |
| claim_matrix | 论文规划阶段首次建立 | 每次新资料支持或挑战已有主张时 |
| evidence_matrix | 每次 extract 步骤发现可复用证据单元时 | 证据可靠性变化时 |
| gap_register | 论文规划阶段首次建立 | 缺口被关闭或新缺口被发现时 |
| task_backlog | 每次 plan 步骤生成新任务时 | 任务完成或取消时 |
