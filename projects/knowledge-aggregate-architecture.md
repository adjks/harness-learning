# Knowledge Aggregate Architecture

这份图用于构思一个个人或团队可用的知识聚合体：知识门户负责统一入口，WeKnora / LLMWiki 负责知识沉淀和检索，Pi Agent CLI 负责人、Agent 与知识之间的任务桥接。

## Architecture Map

```mermaid
%%{init: {"theme": "base", "themeVariables": {
  "fontFamily": "Inter, Microsoft YaHei, sans-serif",
  "primaryBorderColor": "#334155",
  "lineColor": "#64748b",
  "clusterBkg": "#f8fafc",
  "clusterBorder": "#cbd5e1"
}}}%%

flowchart LR
    U(["人 / Agent 使用者"]):::actor

    subgraph Portal["知识门户：统一入口与导航"]
        P["门户首页<br/>导航 / 搜索 / 深链入口"]:::portal
        LIB["图书馆<br/>电子书 / PDF / EPUB"]:::source
        DOC["内部知识文档<br/>制度 / 项目 / 经验"]:::source
        SYS["内部系统托管<br/>业务系统 / Dashboard / 工具入口"]:::source
    end

    subgraph Knowledge["知识库：WeKnora + LLMWiki"]
        INGEST["知识采集与整理<br/>上传 / 同步 / 清洗 / 分块"]:::pipeline
        KB["WeKnora Knowledge Base<br/>RAG / Agent / MCP / API"]:::knowledge
        WIKI["LLMWiki 编译<br/>Markdown Wiki / 知识图谱"]:::knowledge
        RAG["混合检索<br/>Vector / BM25 / GraphRAG"]:::knowledge
        MCP["MCP / REST API<br/>检索 / 上传 / 管理"]:::api
    end

    subgraph Agent["Agent CLI：Pi Agent 桥梁"]
        PI["Pi Agent CLI<br/>人机交互 / Agent Loop / Tool Calling"]:::agent
        TASK["任务执行<br/>查询 / 摘要 / 写作 / 自动化"]:::agent
    end

    U --> P
    U --> PI

    P --> LIB
    P --> DOC
    P --> SYS

    LIB --> INGEST
    DOC --> INGEST
    SYS -. 系统上下文 / 链接索引 .-> INGEST

    INGEST --> KB
    KB --> WIKI
    KB --> RAG
    KB --> MCP

    PI --> MCP
    PI --> TASK
    TASK --> KB
    TASK --> P

    P -. Wiki 深链 .-> WIKI
    P -. 问答搜索入口 .-> RAG
    PI -. 调用内部系统上下文 .-> SYS

    classDef actor fill:#fff7ed,stroke:#f97316,stroke-width:2px,color:#7c2d12;
    classDef portal fill:#fffbeb,stroke:#d97706,stroke-width:2px,color:#78350f;
    classDef source fill:#eff6ff,stroke:#2563eb,stroke-width:1.5px,color:#1e3a8a;
    classDef pipeline fill:#ecfdf5,stroke:#059669,stroke-width:1.5px,color:#064e3b;
    classDef knowledge fill:#f0fdf4,stroke:#16a34a,stroke-width:2px,color:#14532d;
    classDef api fill:#eef2ff,stroke:#4f46e5,stroke-width:1.5px,color:#312e81;
    classDef agent fill:#faf5ff,stroke:#9333ea,stroke-width:2px,color:#581c87;
```

## Component Roles

- **知识门户**：面向人提供统一导航、搜索入口和深链入口，把电子书、内部文档、内部系统托管入口组织成可浏览的知识地图。
- **WeKnora / LLMWiki 知识库**：把原始文档和系统上下文沉淀为可检索、可问答、可编译的知识资产，核心能力包括 RAG、混合检索、Wiki 编译、知识图谱、MCP 和 REST API。
- **Pi Agent CLI**：作为人和知识之间的执行桥梁，让用户用命令行或 Agent 工作流触发查询、摘要、写作、自动化和内部系统上下文调用。

## Relationship Notes

- 实线表示核心信息流：知识从门户收集到知识库，Agent 通过 MCP / API 使用知识库能力，再把任务结果回写到门户或知识库。
- 虚线表示辅助连接：门户可跳转到 Wiki 或问答入口，Pi Agent 可引用内部系统上下文，内部系统也可以被整理成链接索引。
- 这张图先服务构思和讨论，不包含部署拓扑、数据库选型、鉴权模型或完整 API 合约。

## Open Questions

- 权限模型：门户、知识库和 Agent CLI 是否共享同一套身份与权限？
- 同步机制：电子书、内部文档和系统链接是手动导入、定时同步，还是由 Agent 辅助整理？
- MCP / API 边界：Pi Agent 主要调用 WeKnora 的 MCP 工具，还是走 REST API / SDK？
- 门户深链：门户里的每个知识入口是否需要直接跳到 Wiki 页面、RAG 问答或原始系统？
- CLI 工作流：Pi Agent 更偏“临时问答助手”，还是要沉淀为可复用的项目级命令和技能包？
