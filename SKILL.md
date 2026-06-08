---
name: goal-driven-research-system
description: Use when any new research input (paper, data, code, notes, experiment, meeting, draft, web page, search result) enters the project. Evaluates impact on research goal, claims, evidence, gaps, and tasks. Maintains source registry, claim matrix, evidence matrix, gap register, and task backlog to continuously evolve the research project toward a defined output. Formerly research-work-knowledge-manager — the old name is kept as an alias.
version: 2.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, knowledge-management, goal-driven, evidence-tracking, claim-matrix, gap-analysis, task-backlog]
    related_skills: [arxiv, deep-research, academic-paper, wildlife-manuscript-builder]
    aliases: [research-work-knowledge-manager]
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
| **ingest** | 登记资料，生成 file_id，计算 checksum，记录 provenance | source_registry 新条目 |
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

## 不同 source_type 的专项处理

> 详细规则见 `references/source-type-handlers.md`

| source_type | 关键处理 |
|-------------|---------|
| **dataset** | 建立 provenance 链：generated_by → cleaned_by → analyzed_by → used_in_figure → used_in_claim |
| **code** | 不与数据孤立登记，建立 code↔data↔figure↔result 关系 |
| **meeting_note** | 提取决策→decision_log，质疑→影响 claim，待办→task_backlog |
| **manuscript_draft** | 进入后立即一致性检查：引用是否过期、与 wiki 是否一致 |
| **search_result** | 保存快照、访问日期、原始链接（网页易变） |
| **literature_pdf** | 提取方法/结论/局限 → 关联到 claim 作为 supporting/challenging |

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
│   ├── conflict_log.md
│   ├── decision_log.md
│   └── impact_log.md
├── wiki/               # 知识页面
├── outputs/            # 生成物（Evolution Reports）
├── releases/           # 冻结版本
└── archive/            # 历史文件
```

## 最小可行启动 (MVP)

最少先建立：

```
registry/source_registry.jsonl     # 空数组 []
registry/claim_matrix.jsonl        # 空数组 []
registry/evidence_matrix.jsonl     # 空数组 []
registry/gap_register.jsonl        # 空数组 []
registry/task_backlog.jsonl        # 空数组 []
registry/conflict_log.md           # "# Conflict Log"
registry/decision_log.md           # "# Decision Log"
registry/impact_log.md             # "# Impact Log"
wiki/index.md                      # Wiki 索引
```

> 更多渐进采用路径见 `assets/adoption-self-assessment.md`

## Workspace 重组流程（已存在混乱工作区的整理）

> 具体输出模板见 `references/workspace-reorganization-example.md`

1. 扫描全景 → 2. 识别分析包 → 3. 请求作者确认当前版本 → 4. 建立目录骨架 → 5. 移动原始数据到 sources/ → 6. 归档旧包 → 7. 设当前包 → 8. 登记所有文件到 source_registry → 9. 建 wiki 和 claim_matrix → 10. 清理残留

## Agent 执行约束

1. **先查 registry 再回答**：涉及文件状态、版本、关系的查询，必须读 registry，不能凭记忆或文件名猜测
2. **只读来源库**：`sources/` 中的原始文件默认只读，不直接改写
3. **不自行决定**：遇到版本取舍、数据矛盾等需要专业判断的事项，写入 decision_log 等待用户决定
4. **每次操作追加 impact_log**：记录时间、来源、影响维度（claims/gaps/tasks）
5. **Wiki 必须引用 file_id 和 claim_id**：不写「某个文件」「之前的数据」等模糊表述
6. **Windows 非 ASCII 路径**：含中文路径时用 `execute_code` + Python 完成文件操作

## Hermes Integration

> 详细说明见 `references/hermes-integration.md`

- **Memory**: 首次在某 workspace 执行时，将项目惯例写入 memory
- **Cron**: 每周自动 lint（检查 orphan files / broken chains / stale refs / aging decisions）
- **Curator**: skill 自身版本用 curator 追踪
- **Session Search**: 回溯历史决策时用 `session_search` 而非重读整个 decision_log

## Common Pitfalls

1. **只登记不评估影响**：登记了 source 但没有走 extract→evaluate→link 步骤，变成单纯的文件清单
2. **claim 和 evidence 混在一起**：claim 是论文级别主张，evidence 是资料级别证据单元——一个 claim 可以有多个 evidence
3. **忽略 challenging_sources**：只记录支持的证据，不记录挑战的证据
4. **gap 和 task 不分**：gap 是"缺什么"（名词），task 是"做什么"（动词）
5. **靠文件名判断版本**：必须查 registry 中的 version_id 和 status
6. **不记录冲突**：发现不一致时私下修正而不写入 conflict_log
7. **替用户做决定**：需要专业判断的事项写入 decision_log，不自行裁决
8. **Terminal 中文路径失败**：用 `execute_code` + Python `os`/`shutil`/`hashlib`

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