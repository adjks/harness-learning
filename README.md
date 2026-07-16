# harness-learning

AI Agent Harness 工程学习库：跟踪前沿研究、沉淀个人 AI 工作流经验，并把想法转成可实现的工程项目路线。

## 这个仓库关注什么

这里的 Harness 不是单纯的测试脚手架，也不是 Harness.io 公司平台，而是 **AI agent harness**：支撑 AI 编程代理工作的运行时工程系统。它决定代理如何观察项目、选择上下文、调用工具、记录状态、接受权限约束、验证结果、归因失败，并把一次任务变成可复盘的证据包。

核心问题是：

- 如何让 AI 编程代理不只是“生成补丁”，而是产生可验证、可审计、可维护的工程变更？
- 如何把上下文、工具、记忆、权限、日志、测试和人工干预组织成稳定的运行时？
- 如何让个人 AI 工作流从聊天式使用，进化成可复用、可复盘、可持续改进的工程系统？

## 目录

- [research/](research/)：前沿论文、技术报告、benchmark 和架构范式
- [weekly/](weekly/)：每周自动生成的实用经验周报
- [projects/](projects/)：适合个人实现的工程项目路线
- [workflows/](workflows/)：个人 AI 工作流模板、复盘方法和实践记录
- [sources.md](sources.md)：长期维护的资料源索引

## 学习路线

1. **定义边界**：区分 AI agent harness、传统 test harness、CI/CD 平台和多代理框架。
2. **读核心论文**：从 runtime substrate、trace-based evaluation、self-evolving harness、contract-driven verification 四条线入手。
3. **拆模块**：任务契约、上下文选择、工具权限、项目记忆、执行状态、观测日志、失败归因、验证闭环。
4. **做个人工作流**：把每次使用 AI 的过程沉淀为任务契约、证据包、验证记录和复盘卡。
5. **做工程项目**：从轻量日志规范开始，逐步实现个人 harness 原型。

## 当前索引

- 前沿地图：[research/frontier-map.md](research/frontier-map.md)
- 项目路线：[projects/project-roadmap.md](projects/project-roadmap.md)
- 个人工作流：[workflows/personal-ai-workflow.md](workflows/personal-ai-workflow.md)
- 第一篇周报：[weekly/2026-07-17.md](weekly/2026-07-17.md)

## 周更机制

计划每周一 09:00（Asia/Shanghai）运行一次 Codex 自动化任务：

- 收集 AI Agent Harness、coding agents、SWE benchmarks、agent observability、verification harness 相关资料
- 生成 `weekly/YYYY-MM-DD.md`
- 更新本 README 的周报索引
- 如有新增内容，提交并推送到 GitHub

周报默认会包含来源链接；不确定结论会标注为“推断”或“待验证”。

