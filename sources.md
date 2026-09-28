# Sources

长期维护的资料源索引。优先收集“别人怎么实际驾驭 agent”的资料，其次收集解释这些实践的研究论文。

## 实用工作流与官方经验

- [Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Claude Code power user tips](https://support.claude.com/en/articles/14554000-claude-code-power-user-tips)
- [Claude Code common developer use cases](https://support.claude.com/en/articles/14553517-claude-code-common-developer-use-cases)
- [OpenAI: Harness engineering](https://openai.com/index/harness-engineering/)
- [OpenAI: Codex App Server](https://openai.com/index/unlocking-the-codex-harness/)
- [OpenAI: Running Codex safely](https://openai.com/index/running-codex-safely/)
- [OpenAI: Scientific computing in the age of agentic AI](https://openai.com/index/scientific-computing-agentic-ai/)
- [OpenAI: Agents SDK evolution](https://openai.com/index/the-next-evolution-of-the-agents-sdk/)
- [OpenAI: Symphony orchestration](https://openai.com/index/open-source-codex-orchestration-symphony/)
- [OpenAI: GPT-5.6 harness efficiency](https://openai.com/index/gpt-5-6-frontier-intelligence-efficiency/)
- [OpenAI: Introducing the Agents API](https://openai.com/index/introducing-the-agents-api/)
- [OpenAI: Research acceleration](https://openai.com/index/research-acceleration-view-inside-openai/)
- [GitHub: Agent Plugins 1.0](https://github.blog/changelog/2026-08-12-agent-plugins-1-0-in-vs-code-copilot-cli-and-the-copilot-app/)
- [GitHub: comment-triggered Copilot automations](https://github.blog/changelog/2026-08-03-trigger-copilot-automations-with-comments/)
- [VS Code: The Coding Harness Behind GitHub Copilot](https://code.visualstudio.com/blogs/2026/05/15/agent-harnesses-github-copilot-vscode)
- [Anthropic: A harness for every task](https://claude.com/blog/a-harness-for-every-task-dynamic-workflows-in-claude-code)
- [AgentWay: Claude Code workflows](https://agentway.dev/en/claudecode/workflows)
- [Anthropic: The AI-Native SDLC playbook](https://claude.com/blog/the-ai-native-sdlc-playbook)
- [Anthropic: Claude Code guide for startups](https://claude.com/blog/claude-code-guide-for-startups)
- [Anthropic: Claude Code Auto mode](https://claude.com/blog/auto-mode-default-in-claude-code)
- [GitHub: content exclusions in Copilot app and CLI](https://github.blog/changelog/2026-09-02-content-exclusions-generally-available-in-copilot-app-and-cli/)
- [GitHub: Copilot code review approvals](https://github.blog/changelog/2026-09-01-copilot-code-review-can-now-approve-pull-requests/)
- [GitHub: Enterprise managed permissions for Copilot agent operations](https://github.blog/changelog/2026-09-09-enterprise-managed-permissions-for-github-copilot-agent-operations/)
- [GitHub: Enterprise-managed sandbox in Copilot for JetBrains](https://github.blog/changelog/2026-09-08-enterprise-managed-sandbox-in-copilot-for-jetbrains/)
- [GitHub: local sandboxing in the Copilot app](https://github.blog/changelog/2026-09-23-local-sandboxing-in-the-github-copilot-app/)
- [GitHub: OpenTelemetry in the Copilot app](https://github.blog/changelog/2026-09-22-opentelemetry-in-the-github-copilot-app/)
- [GitHub: agentic autofix uses Copilot Memory](https://github.blog/changelog/2026-09-25-agentic-autofix-now-uses-copilot-memory/)
- [Google Cloud: Agent Factory harness recap](https://cloud.google.com/blog/topics/developers-practitioners/agent-factory-recap-agent-harnesses-shifting-left-and-autonomous-coding)

## GitHub 趋势入口

- [GitHub Trending: daily](https://github.com/trending?since=daily)
- [GitHub Trending: weekly](https://github.com/trending?since=weekly)
- [GitHub Trending: Python weekly](https://github.com/trending/python?since=weekly)
- [GitHub Trending: TypeScript weekly](https://github.com/trending/typescript?since=weekly)
- [GitHub Trending: Jupyter Notebook weekly](https://github.com/trending/jupyter-notebook?since=weekly)

筛选时优先关注和 coding agents、AI workflow、agent harness、developer productivity、MCP、CLI automation、repo analysis、testing/verification 相关的项目。趋势项目不只记录热度，还要拆解它解决的痛点、采用的方法、可迁移的工作流启发。

近期已检查、适合按 release/重大 README 变化再跟进的趋势项目：

- [code-review-graph](https://github.com/tirth8205/code-review-graph)
- [OpenCodeReview](https://github.com/alibaba/open-code-review)
- [LoopX](https://github.com/huangruiteng/loopx)
- [TencentDB Agent Memory](https://github.com/TencentCloud/TencentDB-Agent-Memory)
- [Agent Skills](https://github.com/addyosmani/agent-skills)
- [Symphony](https://github.com/openai/symphony)
- [MemoHarness](https://github.com/HowieHwong/MemoHarness)
- [OpenAI Plugins](https://github.com/openai/plugins)
- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [Cloudflare Security Audit Skill](https://github.com/cloudflare/security-audit-skill)
- [Strands Harness SDK](https://github.com/strands-agents/harness-sdk)

采集时保持增量：如果某个趋势仓库、文章、官方文档或论文已经在 `weekly/` 中详细拆解过，后续周报默认跳过；只有出现重大更新、新 release、新案例或新的实践争议时才再次纳入，并标注新增点。

## 治理、安全和权限边界

- [TechRadar: governance controls for AI coding agents](https://www.techradar.com/pro/why-ai-coding-agents-keep-stalling-before-production-and-the-governance-controls-that-fix-it)

## 核心论文与技术报告

- [AI Harness Engineering: A Runtime Substrate for Foundation-Model Software Agents](https://arxiv.org/abs/2605.13357)
- [Agentic Harness Engineering: Observability-Driven Automatic Evolution of Coding-Agent Harnesses](https://arxiv.org/abs/2604.25850)
- [Code as Agent Harness](https://arxiv.org/abs/2605.18747)
- [Meta-Engineering Harnesses for AI-Native Software Production](https://arxiv.org/abs/2605.25665)
- [Harnessing Agentic Evolution](https://arxiv.org/abs/2605.13821)
- [MemoHarness: Agent Harnesses That Learn from Experience](https://arxiv.org/abs/2607.14159)
- [Harnessing Code Agents for Automatic Software Verification (Aria)](https://arxiv.org/abs/2607.06341)
- [Control the Harness, Control the Cost: Routing and Governing AI Coding Agents in the Enterprise](https://arxiv.org/abs/2609.28919)

## Benchmarks and evaluation

- [SWE-Bench Pro](https://arxiv.org/abs/2509.16941)
- [SWE-Bench](https://www.swebench.com/)
- [Terminal-Bench](https://www.tbench.ai/)

## 能力封装、工具治理与可运行参考

- [GitHub: MCP allowlists in enterprise managed settings](https://github.blog/changelog/2026-08-06-mcp-allowlists-in-enterprise-managed-settings/)
- [thecarbonlayer/carbon](https://github.com/thecarbonlayer/carbon)
- [SUNRNEHUI/agent-harness](https://github.com/SUNRNEHUI/agent-harness)

## 搜索关键词

- AI coding agent workflow
- Claude Code best practices
- Codex harness engineering
- agent harness practical workflow
- context engineering for coding agents
- AI agent verification workflow
- human in the loop coding agent
- dynamic workflows coding agent
- worktree agent workflow
- agent governance isolate scope approve
