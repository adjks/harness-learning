# AI Agent Harness 工程前沿地图

## 一句话定义

AI Agent Harness 是包裹在基础模型之外的工程运行时。它负责把“模型能力”转化为“可执行、可验证、可审计、可持续改进的软件工程能力”。

## 与相邻概念的区别

- **传统 test harness**：重点是驱动被测对象、注入输入、收集测试结果。
- **AI agent framework**：重点常在工具调用、记忆、规划、多代理编排。
- **AI agent harness**：更强调工程闭环，包括任务规格、上下文、工具权限、状态、观测、验证、失败归因和人工干预记录。
- **CI/CD 平台**：重点是交付流水线；agent harness 可以调用 CI/CD，但不等同于 CI/CD。

## 四条前沿主线

### 1. Runtime substrate

代表资料：[AI Harness Engineering: A Runtime Substrate for Foundation-Model Software Agents](https://arxiv.org/abs/2605.13357)

关键观点：软件工程能力来自 model-harness-environment 系统，而不只来自模型本身。该方向把 harness 形式化为一组运行时职责，包括任务规格、上下文选择、工具接入、项目记忆、任务状态、观测、失败归因、验证、权限、熵审计和干预记录。

对个人工作流的启发：每次让 AI 做事前，先写清任务契约；每次完成后，保留验证证据和失败归因，而不是只保存最终答案。

### 2. Trace-based evaluation

代表资料同上，以及 SWE-Bench/SWE-Bench Pro 等软件工程代理 benchmark。

关键观点：只看最终 patch 是否通过测试是不够的。更好的评估对象是完整 episode：输入、上下文、动作、工具调用、失败、修复、验证和人工干预。

对个人工作流的启发：把一次 AI 协作变成“证据包”，至少记录目标、关键决策、运行命令、失败点、最终验证。

### 3. Self-evolving harness

代表资料：[Agentic Harness Engineering](https://arxiv.org/abs/2604.25850)

关键观点：harness 本身也可以被观测、修改和评估。前沿方向不只是调 prompt，而是让工具层、中间件、长期记忆和运行策略都成为可版本化、可回滚、可归因的工程组件。

对个人工作流的启发：把自己的 AI 工作流模板、检查清单和复盘规则当作代码一样迭代。每次修改规则前写下预期效果，之后用实际结果验证。

### 4. Contract-driven adversarial verification

代表资料：[Meta-Engineering Harnesses for AI-Native Software Production](https://arxiv.org/abs/2605.25665)

关键观点：AI-native 软件生产需要把需求转成明确契约，再用独立或对抗式验证检查实现是否满足契约。验证边界和契约完整性本身也是系统质量的一部分。

对个人工作流的启发：让一个 AI 负责实现，另一个独立视角负责审查需求契约、测试证据和遗漏场景。

## 可落地模块

- **Task contract**：目标、输入、输出、验收标准、禁止事项。
- **Context selector**：哪些文件、资料、历史记录应该进入上下文。
- **Tool policy**：哪些工具可用，哪些动作需要确认。
- **Project memory**：长期约定、架构决策、常见失败和修复经验。
- **Trace log**：关键动作、命令、结果、错误、人工干预。
- **Verifier**：测试、静态检查、人工检查清单、引用来源检查。
- **Failure taxonomy**：需求不清、上下文缺失、工具失败、验证不足、权限阻塞、模型误判。

## 当前判断

推断：未来一段时间，AI 编程代理的竞争不会只发生在模型层，也会发生在 harness 层。谁能更好地管理上下文、工具、权限、验证和经验复用，谁就更容易把模型能力转化成稳定生产力。

