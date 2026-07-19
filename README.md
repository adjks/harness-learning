# harness-learning

AI Agent Harness 实用工作流学习库：持续收集网络上个人开发者、工程团队和工具厂商如何驾驭 coding agents 的实践，再提炼成可借鉴的工作流模式。

## 这个仓库关注什么

这里的 Harness 不是单纯的测试脚手架，也不是 Harness.io 公司平台，而是 **AI agent harness**：包裹在模型外侧的工程运行时和工作流系统。它决定 agent 如何获得上下文、调用工具、执行计划、接受权限约束、验证结果、记录证据，并在失败后恢复。

本仓库的重点不是记录我自己的使用流水账，而是观察网络上的实用经验：

- 官方团队如何建议使用 Claude Code、Codex、Copilot coding agent 等工具？
- 高阶用户如何设计 plan mode、verification、worktree、subagent、workflow、memory、automation？
- 哪些实践可以迁移到个人 AI 工作流里，让 agent 更可靠、更可控、更可复盘？

## 目录

- [research/](research/)：前沿论文、技术报告、benchmark 和架构范式
- [weekly/](weekly/)：每周自动生成的网络实践周报
- [projects/](projects/)：适合个人实现的 harness 工程项目路线
- [workflows/](workflows/)：从外部实践提炼出的个人工作流模式
- [sources.md](sources.md)：长期维护的资料源索引

## 学习路线

1. **先看别人怎么用**：优先收集真实工具文档、官方经验、团队实践、社区工作流。
2. **抽象 harness 模式**：把实践归类为上下文、计划、验证、权限、隔离、记忆、复盘、自动化。
3. **转成个人可用清单**：每条经验都要回答“我下次使用 agent 时能怎么改？”
4. **再看前沿研究**：用论文解释为什么这些模式有效，而不是只停留在技巧层。
5. **做小项目验证**：把高频实践沉淀成模板、脚本或轻量 CLI。

## 当前索引

- 前沿地图：[research/frontier-map.md](research/frontier-map.md)
- 实践模式：[workflows/external-practice-patterns.md](workflows/external-practice-patterns.md)
- 项目路线：[projects/project-roadmap.md](projects/project-roadmap.md)
- 第一篇周报：[weekly/2026-07-17.md](weekly/2026-07-17.md)

## 周更机制

每周一 09:00（Asia/Shanghai）运行一次 Codex 自动化任务：

- 收集网络上关于 AI Agent Harness、coding agents、个人 AI 工作流、团队 agent 使用规范的实用资料
- 优先看官方文档、工程博客、真实团队实践、社区高质量经验贴，以及 GitHub 每日/每周趋势中的相关项目
- 生成 `weekly/YYYY-MM-DD.md`
- 更新本 README 和 `weekly/README.md` 的周报索引
- 如有新增内容，提交并推送到 GitHub

周报默认包含来源链接；不确定结论标注为“推断”或“待验证”。每篇周报至少提炼一个“可迁移到个人工作流的 harness 实践”。
