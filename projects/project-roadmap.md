# 个人工程项目路线

## P0: Harness Learning Knowledge Base

目标：维护这个仓库本身，让它成为持续学习和复盘的入口。

产出：

- 每周一篇实用经验周报
- 持续更新资料源和前沿地图
- 把个人实践抽象成模板

验收：

- README 能解释项目目标和目录
- 每篇周报有来源链接和至少一个可执行实践点

## P1: Task Contract Template

目标：设计一套个人使用的 AI 任务契约模板。

核心字段：

- 目标
- 背景
- 输入材料
- 输出格式
- 验收标准
- 禁止事项
- 需要人工确认的决策

适合先做成 Markdown 模板，之后再考虑 CLI 或小型网页工具。

## P2: Episode Package Logger

目标：把一次 AI 协作过程整理成可复盘的 episode package。

核心能力：

- 记录任务契约
- 记录关键命令和结果
- 记录失败归因
- 记录最终验证
- 生成复盘摘要

最小实现：一个本地脚本或 Markdown 模板即可。不要一开始就做复杂平台。

## P3: Personal Harness CLI

目标：做一个轻量 CLI，把任务契约、日志、验证清单和周报生成串起来。

建议命令：

- `harness new`：创建任务契约
- `harness log`：追加关键事件
- `harness verify`：记录验证结果
- `harness recap`：生成复盘卡

技术选择：Python 或 Node.js 均可。优先选你最愿意长期维护的栈。

## P4: Verification Companion

目标：围绕 AI 生成代码/文档建立独立审查流程。

核心思想：

- 实现者和验证者分离
- 验证者只看契约、diff、证据和测试结果
- 输出明确的通过/风险/阻塞结论

这是最接近前沿 harness 研究的个人项目方向。

