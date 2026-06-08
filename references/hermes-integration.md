# Hermes Integration

## Memory: 记住项目惯例
首次在某 workspace 执行时，将项目惯例写入 memory：
- workspace 路径 → project 映射
- 用户偏好的敏感数据标记习惯
- 常用 journal target
- 后续 session 自动加载，无需重复说明

## Cron: 自动维护
建议每周 lint：检查 orphan files、broken version chains、stale references、aging decisions
→ 输出 Lint Report 到 outputs/

## Curator: 追踪 skill 版本
RWKM skill 的迭代用 curator 管理：bump version、追踪使用频率、旧版归档备份

## Session Search: 回溯历史决策
需要了解历史决策时，用 session_search 而非重读整个 decision_log