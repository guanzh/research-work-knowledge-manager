# Source Type Handlers

为每种 source_type 定义 agent 在 pipeline 的 extract 和 evaluate 步骤中的专项处理规则。

## dataset
extract: 数据来源、版本、变量列表、样本量、缺失值、异常值、处理流程、对应代码 file_id
关键: 建立 provenance 链（generated_by → cleaned_by → analyzed_by → used_in_figure → used_in_claim）

## code
extract: 代码目的、输入/输出 file_id、依赖环境、对应数据版本、生成图表、支持结果
关键: 代码不应只登记为文件，应与数据、图表、结果建立关系

## meeting_note
extract: 决定→decision_log, 质疑→claim_matrix.challenging, 待办→task_backlog
关键: 会议记录同时影响 decision_log + claim_matrix + gap_register + task_backlog

## manuscript_draft
extract: 论文结构、核心论点、缺失引用、过强表述、证据不足段落
关键: 进入后立即一致性检查（引用是否过期、与 wiki 是否一致）

## search_result
extract: 搜索问题/日期/关键词、来源链接（保存快照）、筛选标准、可信度
关键: 网页易变，保存快照和访问日期

## literature_pdf
extract: 研究问题、方法、核心结论、引用价值、局限
link: supporting_sources 或 challenging_sources

## web_page
extract: 观点、来源可信度、发布时间、原始链接
注意: 可信度评估比学术文献更严格

## experiment_log
extract: 实验条件、过程、结果、异常、与 protocol 偏差

## figure
extract: 来源数据 file_id、生成代码 file_id、表达意图、版本

## annotation
extract: 个人判断、问题、想法
注意: 不进入 evidence_matrix（主观判断），但可触发 gap 或 task

## email_or_message
extract: 发送者、核心决策、修改建议、敏感信息处理

## external_report
extract: 背景、数据来源、论证、风险、可借鉴点

## audio_transcript
extract: 观点、证据引用、待确认信息

## 通用 evaluate 维度
对所有 source_type 评估：可信度、相关性、时效性、完整性、风险
→ 影响 impact_on_goal: high/medium/low