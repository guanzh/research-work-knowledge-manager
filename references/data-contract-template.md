# Data Contract Template

> 从 wildlife-manuscript-builder Gate 1.5 吸收。在正式使用数据撰写论文前，冻结此合同。

## 何时使用

当 registry 中存在属于同一项目的多个数据文件，且即将用于撰写论文、报告或进行统计分析时，agent 应引导用户冻结一份 data contract。

## 合同内容

### 1. Raw Observed Facts（原始观测事实）

列出所有直接来自原始数据文件的事实，标注来源 file_id：

| # | Fact | Source file_id | Confidence |
|---|------|---------------|------------|
| 1 | 2023年3-8月布设红外相机 40 台 | F20260601001 | confirmed |
| 2 | 共记录独立事件 1,247 次 | F20260601001 | confirmed |

### 2. Derived Analysis Facts（衍生分析事实）

列出经处理、建模后产生的事实：

| # | Fact | Derived from | Method | Confidence |
|---|------|-------------|--------|------------|
| 1 | 站点级占域率 0.34 (0.22-0.48) | F20260601001 | Occupancy model, RPresence | model_output |
| 2 | 森林覆盖度对占域有正向效应 (β=1.2) | F20260601001, F20260603001 | GLMM, lme4 | model_output |

### 3. Variable Provenance and Units（变量来源与单位）

| Variable | Unit | Source file_id | Column/Field | Notes |
|----------|------|---------------|-------------|-------|
| site_id | categorical | F20260601001 | `station` | 40 levels |
| detection | 0/1 | F20260601001 | `detected` | per survey occasion |
| forest_cover | proportion (0-1) | F20260603001 | `forest_pct` | 30m Landsat derived |

### 4. Count Reconciliation Table（计数核对表）

确保不同文件、不同分析阶段的数据量一致：

| Item | Count | Source | Date checked |
|------|-------|--------|-------------|
| Total survey occasions | 480 | F20260601001 | 2026-06-08 |
| Sites with ≥1 detection | 28 | F20260601001 | 2026-06-08 |
| Sites in model (after excluding missing covariates) | 38 | F20260603001 | 2026-06-08 |

### 5. Manuscript-Safe Facts（可用于论文的事实）

经核实可在论文中使用的最终事实集：

| # | Fact | Strength | Caveat |
|---|------|---------|--------|
| 1 | 森林覆盖度越高的站点长臂猿占域率越高 | model-supported | 相关非因果；样本期仅6个月 |
| 2 | 占域率在距村庄>2km处显著上升 | model-supported | 村庄距离为欧氏距离，未考虑路径 |

### 6. Unresolved Facts（未解决事实）

不能在论文中使用的事实及原因：

| # | Fact | Why unresolved | Action needed |
|---|------|---------------|---------------|
| 1 | 2022年与2023年占域率变化趋势 | 2022年数据仅有15个站点，与2023年不重叠 | 决定：仅报告2023年，或将2022年作为补充材料 |
| 2 | 声学监测与红外相机占域率对比 | 两种方法的时间窗口不一致 | 作者确认：是否做方法对比分析 |

## 与 Registry 的关系

- Data contract 中的每个来源文件必须在 `file_registry.jsonl` 中有对应记录
- 合同冻结后，若任何来源文件的 `status` 变为 `superseded`，agent 必须标记此合同为 stale，并在 `conflict_log.md` 中记录
- 合同文件本身也应分配 file_id 并登记到 registry
