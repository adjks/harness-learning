# 网络实践提炼：如何 Harness 驾驭 Agent

## 目标

观察官方团队、工程团队和高阶用户如何使用 coding agents，把经验抽象成个人可复用的 harness 模式。

## 实践模式 1：Explore -> Plan -> Implement -> Verify

来源：

- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Claude Code common developer use cases](https://support.claude.com/en/articles/14553517-claude-code-common-developer-use-cases)
- [AgentWay Claude Code workflows](https://agentway.dev/en/claudecode/workflows)

外部实践：复杂任务不要让 agent 直接改代码。先让它探索代码库和约束，再产出计划，最后切换到执行模式，并用测试或检查命令验证。

可迁移做法：

- 多文件改动、陌生代码、架构选择，一律先进入“只读探索”。
- 要求 agent 输出“会改哪些文件、为什么、如何验证”。
- 执行阶段让 agent 按计划逐步做，不要把探索、设计、实现混在同一个模糊请求里。

## 实践模式 2：Verification is the harness

来源：

- [Claude Code power user tips](https://support.claude.com/en/articles/14554000-claude-code-power-user-tips)
- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)

外部实践：高质量 agent 工作流的核心不是更会写 prompt，而是给 agent 一个可以自检的反馈回路，例如测试、lint、浏览器、模拟器、端到端检查或可执行断言。

可迁移做法：

- 每次任务开始前写一句“完成标准是什么”。
- 代码任务提供测试命令；文档任务提供结构和链接检查；前端任务提供浏览器截图或交互检查。
- 让 agent 报告“我运行了什么验证、结果是什么、还有什么没验证”。

## 实践模式 3：Humans steer, agents execute

来源：

- [OpenAI: Harness engineering](https://openai.com/index/harness-engineering/)

外部实践：人类从“亲自写每一行代码”转向“设计环境、指定意图、建立反馈回路”。agent 负责执行，人负责方向、约束、验收和系统设计。

可迁移做法：

- 把 prompt 写成任务契约，而不是一句愿望。
- 重点审查 agent 的计划、边界和验证证据，而不是盯着每一行生成过程。
- 对重复任务沉淀模板，让下一次 agent 进入更好的工作环境。

## 实践模式 4：Isolate, scope, approve

来源：

- [TechRadar: governance controls for AI coding agents](https://www.techradar.com/pro/why-ai-coding-agents-keep-stalling-before-production-and-the-governance-controls-that-fix-it)
- [VS Code: The Coding Harness Behind GitHub Copilot](https://code.visualstudio.com/blogs/2026/05/15/agent-harnesses-github-copilot-vscode)

外部实践：agent 能力越强，越需要隔离环境、权限范围和人工批准。真正的 harness 不只是工具调用，还包括上下文组装、工具暴露、agent loop、权限边界和审计。

可迁移做法：

- 高风险任务使用独立分支或 worktree。
- 删除、部署、付费 API、外部写入等动作必须人工确认。
- 每次任务结束保留 diff、命令、验证和人工干预记录。

## 实践模式 5：Workflows as reusable harness

来源：

- [Anthropic: A harness for every task](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code)

外部实践：把常见任务封装成可重复运行的 workflow，让程序负责循环、分支、状态和子任务编排，模型只在需要判断和生成的地方介入。

可迁移做法：

- 把“PR 审查”“资料周报”“前端验收”“文档事实核查”做成固定流程。
- 对重复流程设置预算、停止条件和输出格式。
- 把好用的流程纳入仓库，而不是留在聊天记录里。

## 当前启发

推断：个人使用 agent 的关键，不是立刻搭一个复杂多代理平台，而是先建立四件小东西：任务契约、验证命令、隔离分支、复盘记录。它们合起来就是一个轻量个人 harness。
