---
name: research-work-knowledge-manager
description: Use when importing/managing research files, checking version consistency, generating research summaries, or answering questions about research material status. Maintains file registries, version tracking, conflict detection, decision logging, and a wiki knowledge base for research projects. Not a specific software — a protocol for LLM agents (Codex, Hermes, Claude Code) to manage research knowledge systematically.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [research, knowledge-management, version-tracking, conflict-detection, wiki]
    related_skills: [arxiv, deep-research, academic-paper]
---

# 科研资料知识管家 (Research Work Knowledge Manager)

## Overview

科研工作中持续产生论文、数据、代码、图表、草稿、protocol、会议纪要等文件。时间一长，会出现：不知道哪个是最新版、旧数据被新草稿引用、同一结论在不同文件中说法不一致、图表来自旧代码但正文用了新方法、文件夹里版本混乱、LLM 回答问题时混用新旧文件。

本 skill 定义一套协议，让 LLM agent 在每次处理科研文件时必须记录**身份、版本、来源、冲突和当前结论**。

**核心原则：**

> 原始文件负责保存证据。
> registry 负责记录身份和版本。
> wiki 负责整理知识。
> conflict log 负责记录矛盾。
> decision log 负责记录人的判断。

## When to Use

- 导入新科研文件（论文、数据、protocol、草稿等）
- 查询项目的当前有效文件版本
- 检查论文草稿是否引用了过期数据或图表
- 生成科研变化周报/总结
- 检测文件之间的版本冲突、数值冲突、来源冲突
- 投稿前一致性检查
- 建立或维护科研 workspace 的 registry 和 wiki

**不要用于：** 单纯的文件重命名、不涉及版本关系的独立文件存储、非科研类文档管理。

## 目录结构

```
research_workspace/
├── inbox/              # 新文件暂存，等待导入
├── sources/            # 原始资料正式存储
│   ├── papers/
│   ├── datasets/
│   ├── code/
│   ├── drafts/
│   ├── figures/
│   └── experiments/
├── registry/           # 核心：登记表和日志
│   ├── file_registry.jsonl
│   ├── work_registry.jsonl
│   ├── version_registry.jsonl
│   ├── decision_log.md
│   ├── conflict_log.md
│   └── audit_log.md
├── wiki/               # 整理后的知识页面
├── outputs/            # 生成物（报告、周报、投稿检查）
├── releases/           # 冻结版本（投稿前、汇报前快照）
└── archive/            # 重复文件、废弃版本、历史文件
```

## 三层 ID 体系（最重要）

| ID 类型 | 含义 | 格式示例 | 说明 |
|---------|------|----------|------|
| `file_id` | 具体物理文件 | `F20260608001` | 每个文件唯一，不可变 |
| `work_id` | 长期变化的科研对象 | `W_alpha_protocol` | 跨越版本的逻辑实体 |
| `version_id` | work 的第 N 版 | `V003` | 递增，记录演进关系 |

**示例：**

```
F20260528004  Alpha protocol V002
F20260608001  Alpha protocol V003
```

- 两个**不同文件** → file_id 不同
- 属于**同一个 protocol** → work_id 相同
- **不同版本** → version_id 不同

**查询时 agent 必须查 registry，不应凭记忆或文件名猜测。**

## Registry 结构

### file_registry.jsonl

每行一个文件记录：

```json
{
  "file_id": "F20260608001",
  "work_id": "W_alpha_protocol",
  "version_id": "V003",
  "title": "Alpha protocol revision",
  "path": "sources/experiments/F20260608001_alpha_protocol.pdf",
  "file_type": "pdf",
  "content_type": "protocol",
  "project": "alpha_project",
  "status": "active",
  "checksum": "sha256_value",
  "summary": "Revised exclusion criteria and measurement schedule.",
  "supersedes": ["F20260528004"],
  "superseded_by": null,
  "ingested_at": "2026-06-08T10:30:00Z"
}
```

字段说明：
- `status`: `active`（当前有效）| `superseded`（被替代）| `archived`（归档）| `draft`（未定稿）
- `content_type`: `paper` | `dataset` | `protocol` | `code` | `figure` | `draft` | `notes` | `feedback` | `other`
- `supersedes` / `superseded_by`: 版本替代链

### work_registry.jsonl

每个科研对象一条记录：

```json
{
  "work_id": "W_alpha_protocol",
  "title": "Alpha Project Protocol",
  "project": "alpha_project",
  "current_version": "V003",
  "current_file_id": "F20260608001",
  "created_at": "2026-05-28",
  "updated_at": "2026-06-08"
}
```

### version_registry.jsonl

每次版本变更记录：

```json
{
  "work_id": "W_alpha_protocol",
  "version_id": "V003",
  "file_id": "F20260608001",
  "previous_version": "V002",
  "previous_file_id": "F20260528004",
  "change_summary": "Changed exclusion criteria: removed patients with condition X. Updated measurement schedule from weekly to biweekly.",
  "created_at": "2026-06-08T10:30:00Z"
}
```

### conflict_log.md

记录科研资料之间的不一致：

- **版本冲突**：protocol V002 纳入某类样本，V003 排除该类样本
- **数值冲突**：数据表写样本量 120，论文草稿写 118
- **来源冲突**：图 2 来自旧代码，但正文方法描述的是新代码
- **引用过期**：草稿引用的数据已被新版本替代

格式：每个冲突分配 `K` 前缀 ID，记录涉及文件、冲突描述、状态。

### decision_log.md

记录需要人判断的事项（agent 不能自行决定）：

- 两个 protocol 哪个是正式版本？
- 异常值是否排除？
- 新旧结果矛盾时论文采用哪个？
- 文件看起来像新版本但文件名不清楚，是否替代旧版本？

格式：每个决策分配 `D` 前缀 ID，记录上下文、涉及文件、待选项、状态（`pending` | `resolved`）。

## 新文件导入流程（13 步）

当新文件放入 `inbox/` 后，agent 必须按以下流程执行：

1. **扫描** inbox 中的所有新文件
2. **计算** 每个文件的 SHA-256 checksum，判断是否与已有文件重复
3. **分配** `file_id`（格式：`F` + `YYYYMMDD` + 三位序号）
4. **判断** 所属项目（从文件名、内容、或用户指令推断）
5. **判断** `content_type`（paper / dataset / protocol / code / figure / draft / notes / feedback / other）
6. **判断关系**：全新文件 / 重复文件 / 相关文件 / 某文件的新版本
7. **若是新版本**：分配新 `version_id`，在 version_registry.jsonl 中登记
8. **若替代旧版本**：更新 file_registry 中旧文件的 `superseded_by` 和 `status`，新文件记录 `supersedes`
9. **移动** 文件从 inbox 到 `sources/` 的对应子目录，更新 `path`
10. **更新 wiki** 页面（项目页、数据集页、protocol 页等）
11. **检查冲突**：新文件与已有文件之间是否存在不一致
12. **记录决策**：将 agent 无法自行判断的事项写入 decision_log.md
13. **输出导入总结**：列出新文件、新版本、替代关系、发现的冲突、待决策事项

## Wiki 知识页结构

Wiki 是 agent 整理后的「科研项目说明书」。每个页面应包含：

```markdown
# [项目/论文/实验名称]

## Status
active | draft | archived

## Current Active Files
| file_id | title | version | content_type |
|---------|-------|---------|--------------|
| F20260608001 | Alpha protocol | V003 | protocol |
| F20260607003 | Alpha dataset | V002 | dataset |

## Current Result / Key Findings
[当前结论，引用来源 file_id]

## Open Questions
- [待解决问题]

## Known Conflicts
- K20260608001: [冲突简述]

## Change History
- 2026-06-08: Protocol V003 替代 V002，排除条件变更
```

**Wiki 页面类型：** 项目页、论文页、实验页、数据集页、结论页、protocol 页。

## 日常使用任务

### 导入新文件
> 用 Research Work Knowledge Manager 导入 inbox 里的新文件。

Agent 应执行完整 13 步导入流程。

### 查询当前版本
> 查一下 alpha 项目当前有效 protocol 是哪一版。

Agent 必须查 `work_registry.jsonl` 和 `file_registry.jsonl`，不应凭记忆回答。

### 一致性检查
> 检查 manuscript draft 有没有引用过期的数据或图表。

Agent 应检查：
- 草稿中引用的所有 file_id
- 这些 file_id 的 `status` 是否为 `active`
- 图表来源 file_id 与正文方法描述是否一致
- 数据 file_id 对应版本与 protocol 版本是否匹配
- 所有不一致写入 conflict_log.md

### 生成周报
> 生成本周科研变化总结。

Agent 应列出：
- 新导入文件
- 新版本及替代关系
- 被替代/归档文件
- 新发现冲突
- 待决策事项
- 更新过的 wiki 页面
- 下一步建议

### 投稿前检查
> 投稿前做一次完整一致性检查。

Agent 应：
1. 确认所有活跃文件的版本链完整
2. 确认草稿中所有引用指向活跃版本
3. 确认图表、数据、代码、protocol 版本互相匹配
4. 列出所有未解决的冲突和待决策事项
5. 生成检查报告

## 最小可行启动 (MVP)

一开始不需要全部建成。最少先建立：

```
registry/file_registry.jsonl      # 空数组 []
registry/work_registry.jsonl      # 空数组 []
registry/version_registry.jsonl   # 空数组 []
registry/decision_log.md          # 空文件，标题 "# Decision Log"
registry/conflict_log.md          # 空文件，标题 "# Conflict Log"
wiki/index.md                     # Wiki 索引页
```

然后让 agent 实现最基本流程：
- 新文件进入 inbox → 分配 file_id → 判断是否新版本 → 更新 registry → 更新 wiki → 发现冲突记录 → 不确定就让用户决定

这已能解决 80% 的科研文件混乱问题。

## Workspace 重组流程（已存在混乱工作区的整理）

> 具体输出模板见 `references/workspace-reorganization-example.md`

当面对一个已有大量文件的混乱工作区（而非从零开始），按以下流程重组：

1. **扫描全景**：用 `search_files` 和 `os.walk` 枚举所有文件，统计按目录分布、按扩展名分布、同名文件跨目录重复情况
2. **识别分析包**：科研工作区常见「同一批数据被分析多次，每次生成一个完整包」的模式。识别哪些目录是独立分析包、哪些是原始数据
3. **请求作者确认当前版本**：将候选包列表提交给作者，确认哪个是当前有效版本（写入 decision_log 为 D001）。其余可归档
4. **建立 MVP 目录骨架**：创建 `sources/` `registry/` `wiki/` `archive/` `current/`，初始化空 registry 文件
5. **移动原始数据到 sources/**：将散落的 .xlsx / .pdf 等原始文件移入 sources/ 子目录（data/reference/raw_workbook）
6. **归档旧包**：将非当前的分析包整体移入 `archive/`
7. **设当前包**：将选定的有效包移入 `current/`（或直接留在原地）
8. **登记所有文件**：对 sources/ 和 current/ 中的核心文件计算 checksum、分配 file_id、写入 file_registry.jsonl 和 work_registry.jsonl
9. **建 wiki**：从 current/ 包中的 README、data_contract、result_cards 等文件提取信息，创建 wiki/index.md、wiki/data.md、wiki/analysis.md、wiki/manuscript.md
10. **清理残留**：删除 .pytest_cache、空目录等无科研价值的残留

## Agent 执行约束

1. **先查 registry 再回答**：涉及文件状态、版本、关系的查询，必须读 registry，不能凭记忆或文件名猜测
2. **只读来源库**：`sources/` 中的原始文件默认只读，不直接改写
3. **不自行决定**：遇到版本取舍、数据矛盾等需要专业判断的事项，写入 decision_log 等待用户决定，不假装已知答案
4. **每次操作追加 audit_log.md**：记录时间、操作类型、涉及文件
5. **Wiki 必须引用 file_id**：所有结论和引用必须指向具体的 file_id，不写「某个文件」「之前的数据」等模糊表述
6. **Windows 非 ASCII 路径**: 当工作区路径含中文等非 ASCII 字符时，`terminal` 工具可能拒绝执行（`Blocked: workdir contains disallowed character`）。此时所有文件操作（move、copy、sha256、listdir、walk）改用 `execute_code` 配合 Python `os`/`shutil`/`hashlib` 模块完成，不要尝试 terminal 中的 `mv`/`find`/`cp`

## Common Pitfalls

1. **靠文件名判断版本**：文件名不可靠，必须查 registry 中的 version_id 和 status
2. **忽略替代链**：只看 `status: active`，不检查 `supersedes` / `superseded_by` 链的完整性
3. **不记录冲突**：发现不一致时私下修正而不写入 conflict_log，导致问题被掩盖到投稿前
4. **替用户做决定**：把需要专业判断的事项自行裁决而不写入 decision_log
5. **Wiki 不更新**：导入新文件后只更新 registry，忘记同步更新 wiki 页面
6. **不计算 checksum**：导入时跳过 checksum，导致同一文件被重复分配 file_id
7. **混淆 file_id 和 work_id**：把同一文件的不同副本当成不同 work，或把不同 work 合并到同一 work_id
8. **Terminal 中文路径失败**：在 Windows 上，路径含中文时 `terminal` 工具会拒绝执行。不要尝试 `mv`/`find`/`cp`，直接用 `execute_code` + Python `os`/`shutil`/`hashlib` 完成所有文件操作

## Verification Checklist

- [ ] `file_registry.jsonl` 每行是合法 JSON，file_id 唯一
- [ ] `work_registry.jsonl` 中 `current_version` 和 `current_file_id` 与 `file_registry` 一致
- [ ] 所有 `active` 文件的 `superseded_by` 为 null
- [ ] 所有 `superseded` 文件的 `superseded_by` 指向有效 file_id
- [ ] `version_registry.jsonl` 版本链连续无断裂
- [ ] conflict_log 中每个冲突有唯一 ID，状态明确
- [ ] decision_log 中每个待决策事项有唯一 ID，不混入 agent 已自行处理的内容
- [ ] wiki 页面引用均使用 file_id，不模糊表述
- [ ] 导入新文件后 audit_log 有对应记录
