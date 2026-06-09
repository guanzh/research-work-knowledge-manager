# Research Input Evolution Report — 示例

## New Input
- **file_id**: F20260609001
- **source_type**: dataset
- **title**: 2026 年 5 月高黎贡山长臂猿声学监测数据集
- **path**: sources/acoustic/202605_gaoligong_gibbon/

## Provenance
- **origin**: field_collection
- **collected_by**: 野外团队
- **collected_at**: 2026-05-01 ~ 2026-05-31
- **processing**: raw（未经人工标注）

## Relevance to Goal: high
该数据集直接服务于论文核心问题："雨季夜间声学检测率是否显著高于干季"

## Extracted Units
- **claims**: 新模型在雨季夜间检测率更高（待验证）
- **evidence**: 31 天连续录音、12 个监测点、约 1,200 小时音频
- **methods**: 被动声学监测（PAM）、自动识别模型 v3.2
- **limitations**: 仅覆盖雨季，缺乏干季对比数据；自动识别模型未在该区域校准
- **risks**: 模型 false positive 可能高估检测率
- **tasks**: 人工标注 10% 样本以校准模型、安排干季补充录音

## Impact
- **claims strengthened**: C005（自动识别模型可用于长臂猿监测）— 新增 supporting evidence
- **gaps closed**: 无
- **gaps opened**: G010（缺乏干季声学检测率对比数据）、G011（自动识别模型区域校准未完成）
- **tasks created**: T025（人工标注样本）、T026（安排 12 月干季补充录音）

## Manuscript Impact
- **sections affected**: Methods（增加声学监测设备描述）、Results（增加雨季检测率结果）
- **suggested changes**: 在 Discussion 中明确"当前结果仅限于雨季"，并说明干季数据需求

## Human Review Needed
- [ ] D010: 是否将自动识别模型 v3.2 作为主分析模型，还是先做人工标注校准？
- [ ] D011: 干季补充录音：2026 年 12 月 vs 2027 年 1 月？
