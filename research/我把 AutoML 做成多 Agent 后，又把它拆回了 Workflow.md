过去一段时间，我一直在尝试让大模型像算法工程师一样工作：读数据、做特征、调模型、看错例，再根据实验结果决定下一步。目标听起来很自然，真正做起来却比“让模型写代码”麻烦得多。

我先后搭了两版系统。第一版像一个虚拟算法团队，每项工作都由专门的 Agent 负责；第二版借鉴 AutoResearch，把重点改成围绕实验结果持续迭代的 Workflow。前者太自由，后者又被我管得太死。两次尝试都没有得到理想答案，却让我想清楚了一件事：AutoML 的难点不是该用几个 Agent，而是如何划分研究判断与工程约束。

这篇文章记录的就是这段演进过程，以及我准备怎么做下一版。

## 我想自动化的不是训练，而是研究过程

真实的机器学习项目很少是“选个模型，调用一次 `fit()`”。假设一个分类模型的 Recall 不够高，原因可能是标签噪声、类别不平衡、特征表达不足、参数不合适，也可能只是分类阈值有问题。算法工程师的大部分时间，都花在提出假设、设计实验和逐个排除这些可能性上。

而且很多约束不会在项目开始时一次性暴露。离线数据与线上输入可能格式不同；训练集里很好用的字段，部署时未必拿得到；一个离线涨点明显的特征，也可能因为使用了未来信息而完全不能上线。

所以我真正想自动化的是下面这个循环：

```text
观察 → 分析 → 假设 → 实验 → 验证 → 修正认知 → 再观察
```

理想状态下，人只需要给出目标和业务边界，例如 `Recall >= 0.90`。系统读取数据与实验历史，判断当前瓶颈，提出假设，修改代码并运行实验，再决定保留、回滚或换方向。

问题也出在这里。这个循环里混着两类性质完全不同的工作。

一类是开放式判断：当前更像数据问题还是特征问题？下一轮应该先看错例还是调参数？一个方向值得试一个候选，还是批量试五个？这些问题很难提前写成完整的 `if/else`，正适合交给 LLM。

另一类是确定事实：训练进程是否成功、文件是否存在、指标文件是否属于当前实验、特征是否依赖线上不可用字段。这些事情有唯一答案，本来就不该让 LLM 猜。

我前两版系统的问题，本质上都是没有把这两类工作分清。

### Workflow 不是一张流程图

最开始，我把 Workflow 理解成一条流水线：

```text
数据分析 → 特征工程 → 模型调参 → 错例分析 → 代码检查
```

后来才发现，这只是 Pipeline。Agent Workflow 还要回答几个更麻烦的问题：当前目标和状态是什么，下一步允许执行哪些动作，谁来做决定，成功或失败后分别去哪里，以及整个循环什么时候应该停。

传统 Pipeline 关心“下一步是什么”，Agent Workflow 关心“根据刚刚发生的事，下一步应该是什么”。多出来的这半句话，恰好是系统里最难的部分。

### 为什么不让一个 Agent 从头跑到尾

理论上，给 Claude Code 一段长 Prompt，让它不断分析数据、改代码和训练模型，似乎最省事。我也试过这种思路。短任务里它确实简单，任务一长，上下文就开始变成负担。

训练日志、Shell 输出、报错堆栈、旧指标、已经否定的特征和重复读取的文件会持续进入同一段上下文。信息越来越多，有效信息密度却越来越低。某个特征明明在三轮前已经失败，相关讨论仍可能影响后续判断；一次报错有价值的可能只有最后五行，却占了大量上下文；旧实验被多次提起后，Agent 甚至会混淆“历史最好”和“当前结果”。

这也让人很难接手。面对一条跑了很久的对话，我往往无法迅速回答：现在进行到哪一步，这一轮到底改了什么，训练是否真的成功，当前 Best 是哪个，下一轮又准备做什么。

所以长任务需要拆分，但拆分的目的不是追求更多 Agent，而是隔离低价值过程信息。数据分析可以在独立上下文里读几百行统计结果，返回给 Leader 的应该只有压缩后的发现，例如：

```json
{
  "findings": [
    "feature_x missing rate = 23%",
    "severe class ratio = 14%",
    "training format differs from online inference format"
  ],
  "recommendation": "fix data compatibility before feature search"
}
```

主上下文只保留会改变后续决策的证据。完整日志和中间产物留在实验目录，需要排查时再读。

## 第一版：组一支多 Agent 算法小队

我最初的设计很直观。既然算法项目有不同分工，就为每个岗位配置一个 Agent：

```text
Human
  ↓
主 Agent
  ├─ 数据分析 Agent
  ├─ 特征工程 Agent
  ├─ 特征机理解析 Agent
  ├─ 模型调参 Agent
  ├─ 错例分析 Agent
  └─ 代码质检 Agent
```

主 Agent 负责理解目标和派发任务，其他 Agent 各做一段。一轮优化可能先由数据分析 Agent 检查分布，再让特征 Agent 改代码，代码质检 Agent 做检查，模型 Agent 训练；效果不理想时，错例分析 Agent 接手，然后把结论传回主 Agent。

这个设计不是一无是处。每个角色的 Prompt 更聚焦，上下文也有一定隔离。错例分析 Agent 不必反复阅读全部代码，代码质检 Agent 也不用知道每次特征讨论的细节。哪个环节表现不好，还可以单独调整对应 Prompt。

我甚至加过一个“小队指令增量进化 Agent”。它观察日常交互，把我反复强调的要求沉淀回角色指令，例如：每轮不要新增太多特征；先确认路径和字段真实存在；新增特征要说明来源与业务含义；错例分析不能停留在现象，要继续判断是数据、特征还是模型问题。

这套机制确实让角色行为稳定了一些，也让我看到 Prompt 适合承载什么：工作习惯、分析框架和输出要求。可一旦要求变成硬约束，Prompt 就靠不住了。

它还有一个实际价值：人工经验不再每次从头传递。Feature Agent 最初的指令可能只有“根据实验结果设计新特征”，经过几轮纠偏后，会逐渐加入每轮控制数量、说明来源字段、解释预期作用、禁止依赖线上不可用信息等要求。对于长期项目，这类角色规范很有用。

但它改善的是行为倾向，而不是系统事实。“先确认路径存在”写十遍，也不如一次 `Test-Path`；“不要使用禁用字段”再醒目，也不如在特征生成后跑血缘检查。这是第一版留下的一个重要判断：经验可以写进 Prompt，底线必须写进程序。

### Agent 会把局部任务越做越大

特征 Agent 的目标是提升指标。只要指标还没达标，它就有理由继续加均值、差分、比值、聚合和组合特征。每次新增看起来都有道理，几轮之后，原本五到十个特征能解决的问题可能膨胀到二十多个。

这不是某个 Agent 不够聪明，而是系统没有搜索预算。每个角色都在努力完成自己的局部目标，却没人对整体复杂度、可维护性和收益负责。

### 自然语言约束很容易被绕过去

我曾明确要求“不要使用绝对位置相关特征”。Agent 的确没有直接加入绝对位置，却构造了位置差值、索引统计量和变换后的衍生变量。形式上遵守了 Prompt，业务含义上仍然越界。

从那以后，我不再把 Prompt 当成安全边界。它可以告诉 Agent 应该怎么做，但“不能做什么”必须落到字段白名单、特征血缘和自动校验上。

### 多 Agent 不会自动互相纠错

我原本以为多个 Agent 可以彼此检查，实际情况往往相反：一个 Agent 声称已经在某个路径写入代码，后续 Agent 如果没有访问文件系统验证，就会把这句话当成事实。不存在的路径、不存在的字段、根据旧日志推断出的“新结果”，都可能沿协作链继续传播。

多个 Agent 共享未经验证的自然语言，并不比单个 Agent 更接近真相。真正的 Grounding 只能来自环境：文件要读过，字段要查过，命令要跑过，结果要与当前实验对应。

### 协作成本很快超过收益

机器学习实验有很强的前后依赖。特征代码写完才能检查，训练结束才能做错例分析，诊断有结果后才知道下一轮做什么。角色虽然很多，流程大部分仍是串行的。

于是我承担了多 Agent 的上下文加载、信息转述和调用延迟，却没有得到多少并行收益。更尴尬的是，代码格式、路径检查、指标计算和大部分 Leakage Check 本来就不需要 LLM。

第一版最大的误区，是按“团队里有哪些岗位”设计系统，而不是按“任务和状态如何流动”设计系统。我开始意识到，真正缺少的不是另一个 Agent，而是一个能管理实验过程的 Workflow。

用两个词概括，第一版是 Role-driven，而机器学习研发更需要 State-driven。前者先问“谁负责”，后者先问“现在知道了什么，下一步最值得验证什么”。角色并非没有用，只是不应该成为系统骨架。

## 第二版：围绕实验搭一个 Loop

第二版里，我减少了常驻角色，把主要决策集中到 Research Leader，并把系统中心从“谁来做”改成“一次实验怎样被提出、执行和评估”。

```text
Human Goal
    ↓
Research Leader
    ↓
Feature / Model / Diagnostic Specialist
    ↓
Experiment Executor
    ↓
Evaluator
    ↓
Keep / Rollback
    ↓
Memory
    ↓
Next Loop
```

### 一轮实验内部发生了什么

第二版的 Loop 被拆成七个步骤：

```text
Observe → Hypothesis → Action → Experiment → Evaluate → Memory → Next Round
```

Observe 读取当前目标、Best Metrics、代码版本、历史实验和已知问题。它们共同构成 Experiment State。这里的“读取”必须来自结构化记录，而不是让 Leader 凭聊天历史回忆。

Hypothesis 是最需要 LLM 的一步。它不能只是“继续优化特征”，而要包含观察、可能原因、准备验证的修改和预期影响。例如：

> False Negative 主要集中在整体功率偏低、通道间差异却不明显的样本。现有特征可能更关注通道间不均匀性，缺少整体功率信息，因此下一轮测试全局统计特征。

Action 再把假设路由给合适的 Specialist。问题指向特征，就调用 Feature Agent；指向模型容量或参数，就调用 Model Agent；暂时无法归因，则先让 Diagnostic Agent 补证据。

Experiment Executor 理论上应该很“笨”。它只执行命令，记录 exit code、stdout 和 stderr，保存模型、指标和本轮 metadata。它不负责解释实验，也不能把“代码运行过”写成“实验成功”。

Evaluate 比较 Candidate 与当前 Best。如果目标单一，可以确定性比较 F1；同时关注 Recall 和 Precision 时，则按预先声明的业务规则判断。最后，Memory 记录本轮假设、修改、状态、指标变化和 Keep/Rollback 决定。失败实验同样要保存，因为“这个方向已经试过且为什么失败”也是证据。

每轮实验只回答三个问题：为什么做、改了什么、结果有没有变好。Research Leader 读取当前最优指标与历史记录，提出一个可验证的 Hypothesis；Specialist 将它落实为代码修改；Executor 运行训练；Evaluator 比较 Candidate 与当前 Best，决定 Keep 或 Rollback。无论成功失败，本轮结果都会进入 Memory。

### 第二版具体改了什么

第一处调整是减少常驻角色。数据分析、特征解释和代码检查不再因为“是一个独立职责”就必须对应一个 Agent，只保留 Feature、Model 和 Diagnostic 这类确实需要开放式判断的 Specialist。训练、指标读取、结果落盘等工作尽量下沉给普通程序。

第二处调整是把长任务切成单轮实验。第一版里，Agent 接到“把指标做到 0.9”后容易在内部不断展开，外面的人很难知道它具体试了什么。第二版要求每轮留下完整记录：

```text
Round 7
Hypothesis：整体功率信息表达不足
Action：新增 mean / min / max 特征
Status：success
F1：0.86 → 0.88
Decision：Keep
```

第三处是 Keep/Rollback。所有 Candidate 都从当前 Best 分叉，验证后再决定是否替换：

```text
Best
 ↓
Candidate
 ↓
Evaluate
 ├─ Better → Keep → New Best
 └─ Worse  → Rollback
```

这样一次失败不会直接污染主代码，也更容易回答最终提升究竟来自哪次修改。它看起来只是版本管理，实际对 Agent 很重要：如果没有清晰基线，模型会把多轮修改揉在一起解释，最后连它自己都说不清哪个假设有效。

第四处是把 Memory 分成成功与失败两类。Success Memory 保存有效特征、参数、指标变化和对应版本；Failure Memory 保存被否定的假设、运行错误、违反约束的候选和失败原因。后者并不是“负面日志”，而是下一轮搜索空间的一部分。没有它，Agent 很容易换一种说法重复同一个实验。

最后，我把完整 Code Safety Check 移到了若干轮探索之后。第一版每轮都让质检 Agent 做代码质量、泄漏、复现性和安全检查，严谨是严谨，Loop 也慢得几乎失去意义。第二版只在实验中保留最低限度的运行与数据安全 Gate，最终候选确定后再做完整 Review。

这个调整后来也需要修正。像代码风格这样的检查可以后置，但训练状态、Artifact 来源、数据泄漏和线上字段约束不能等到最后。哪些检查能后置、哪些必须卡在每轮入口，本身就是 Workflow 设计的一部分。

例如：

```text
Hypothesis：类别不平衡导致重度召回不足
Action：调整 class weight
Result：Recall 上升，但 Precision 明显下降
Decision：Rollback
```

或者：

```text
Hypothesis：现有特征缺少整体功率信息
Action：增加整体统计特征
Result：Recall 与 F1 同时上升
Decision：Keep
```

这一版带来的改善很直接。实验有了清楚的开始与结束；失败修改不会直接污染当前最佳版本；Success Memory 和 Failure Memory 可以减少重复试错；完整的代码质量检查也被移到若干轮探索之后，不再拖慢每次实验。

它终于有点像自动研究系统了。然后，我遇到了一个更隐蔽也更危险的问题。

### 训练失败了，Workflow 还在继续

某一轮训练命令实际执行失败，没有生成新的 `metrics.json`。但 Workflow 没有中断，Evaluator 继续读取固定路径，碰巧读到了上一轮遗留的指标，最后还把本轮标记成 `completed`。

表面上三轮实验都跑完了，事实上后三轮建立在一次失败和一份旧文件上。一次小错误污染了整个实验状态。

正确的链路应该是：

```text
Train
  ↓
Exit Code Check
  ↓
Artifact Validation
  ↓
Evaluate
```

任何一步失败，依赖它的节点都不允许执行。Workflow 不能只描述成功以后去哪，还要明确失败时哪些路径必须立刻断开。

### “文件存在”不等于“结果属于这一轮”

当多个实验共用 `metrics.json`、`error_analysis.json` 和 `status.json` 时，`os.path.exists()` 几乎没有证明力。每个实验都应该有独立目录和唯一身份：

```text
runs/
├── exp_0001/
│   ├── hypothesis.json
│   ├── config.json
│   ├── stdout.log
│   ├── stderr.log
│   ├── metrics.json
│   └── status.json
└── exp_0002/
    └── ...
```

Artifact 还要记录 `experiment_id`、代码版本、数据版本、开始与结束时间。Evaluator 读取结果前，至少应确认：

```text
artifact.experiment_id == current_experiment_id
AND status == success
AND artifact validation passed
```

实验文件不是普通输出，而是 Workflow State 的一部分。来源不可信，后面的判断就没有意义。

### 协作仍然藏在聊天记录里

Leader 为什么调用 Diagnostic？传过去哪些数据？Diagnostic 看到了什么证据，又为什么判断是特征问题？第二版虽然减少了角色，Agent 之间的通信仍然是一团自然语言，很难定位判断究竟在哪一层出错。

我更希望 Specialist 返回结构化结果：

```json
{
  "task_id": "diagnostic_007",
  "evidence": [
    "FN mainly occurs in low-power, low-variance samples",
    "17 suspicious samples are far from the class centroid"
  ],
  "root_causes": [
    {"type": "feature", "confidence": 0.70},
    {"type": "label_quality", "confidence": 0.30}
  ],
  "recommended_actions": [
    "test global power statistics",
    "inspect 17 suspicious labels"
  ],
  "unknowns": []
}
```

Leader 不需要看到 500 条预测记录、几十张中间表和完整日志，只需要保留影响决策的证据。这样既节省上下文，也能追查每个结论从哪里来。

我希望这种 Message Contract 不只用于 Diagnostic。Leader 下发任务时也要明确 `task_id`、输入 Artifact、约束、预算和期望输出；Specialist 返回结果时绑定同一个 `task_id` 与 `experiment_id`。协作链会从“Agent A 说一段话，Agent B 自己理解”变成：

```text
Agent A → Structured Result → State Store → Agent B
```

如果方向跑偏，就能区分是 Leader 的问题定义错了、输入不完整、Specialist 判断错误，还是回传时丢了信息。

### Loop 有了，Re-planning 还没有

为了让第二版稳定，我不断增加规则：指标提升就 Keep，低于阈值就 Tune，连续几轮不涨就 Diagnostic，一轮只测试一个 Hypothesis。这些规则单独看都合理，叠在一起却把系统变成了 `Static Workflow + LLM Nodes`。

真正的算法研究不会严格按照“特征一次、调参一次、诊断一次”推进。假如第一轮发现训练数据与线上输入格式不一致，原定的 Feature Search 就应该停止，后面的任务改成确认线上字段、重建合法特征空间，再重新开始实验。

Loop 只是重复，Re-planning 才是根据新证据改变路线。第二版有了前者，还没有真正做到后者。

## 两次尝试之后，我重新划了一条边界

第一版的问题是 Agent 太自由，第二版的问题是 Workflow 太僵硬。我现在更认可的原则是：研究决策可以动态，执行边界必须确定。

|交给 LLM 的判断|交给程序的事实与约束|
|---|---|
|当前瓶颈是什么|文件和字段是否存在|
|下一轮研究什么|训练是否成功|
|优先处理数据、特征还是模型|Artifact 是否属于当前 Run|
|一个方向投入多少实验预算|特征是否违反线上约束|
|是否应该改变研究路线|是否发生数据泄漏|
|如何综合证据解释错例|当前状态是否允许继续|

我把它记成两组词：

```text
LLM：What / Why / Next
Workflow：Can / Must / Is
```

LLM 负责在信息不完整时做研究判断，程序负责提供事实并守住边界。Workflow 的作用不是替 Agent 思考，而是让它在可信的环境里思考。

## 下一版：State-driven Dynamic Workflow

如果重新搭一版，我不会再提前规定 Round 1 做 Feature、Round 2 Tune、Round 3 Diagnostic。Leader 每次只根据当前 State 规划一小段任务，执行后读取新证据，再决定下一步：

```text
Plan → Execute → Observe → Re-plan
```

State 也不该只有一个 F1。它至少应包含当前 Best、实验历史、成功与失败记录、业务约束、Feature Contract、数据与代码版本、当前诊断和仍未回答的问题。

Leader 管理的不是固定步骤，而是一条随证据变化的 Task Queue。例如开始时，系统可能判断瓶颈在特征表达：

```json
{
  "goal": "Recall >= 0.90",
  "current_focus": "feature_representation",
  "tasks": [
    {"task": "analyze_false_negative", "priority": 1},
    {"task": "generate_feature_candidates", "priority": 2, "budget": 4}
  ]
}
```

Diagnostic 如果发现明显的 Label Noise，Leader 可以直接删除后面的特征任务，把焦点改为 `data_quality`。Agent 有权修改 Research Layer 的后续路线，但不能绕过训练检查、数据约束和状态机。

### 实验预算也应动态分配

第二版为了清晰，固定每轮只测一个 Hypothesis，但不同阶段本来就需要不同的探索力度。证据很弱时，可以先试一两个候选；某个方向连续有效时，可以分配更多预算，批量测试几个机理不同的方案；发现严重的数据问题，则暂停模型实验。

具体数字不重要，关键是 Budget 本身应该由当前证据决定，而不是写死在流程里。

### 失败要分层处理

工程执行应该 Fail-fast，研究流程则要允许 Recover。训练命令失败后，本次 Candidate 必须立即停止，Evaluator 不能继续；但 Leader 可以根据错误选择修一次、放弃方案或重新规划，不必让整个 Workflow 一起退出。

我会把实验状态做成明确的状态机：

```text
pending
   ↓
running ─────────→ failed
   ↓
execution_success
   ↓
artifact_validated
   ↓
evaluated
   ├─ rejected
   └─ kept
```

这里没有 `failed → evaluated` 这条路。

不同失败也要进入不同状态。训练返回非零 exit code，本轮记为 `failed` 并保存 stderr；指标文件缺失，则是 `artifact_invalid`；候选特征依赖线上不可用字段，应在训练前标为 `candidate_rejected`；检测到 Leakage 的实验不能进入 Leaderboard。

这几种失败都只终止当前 Branch。Leader 仍可根据错误决定修一次、放弃候选或换方向。否则 Fail-fast 很容易被做成“一出错整个系统退出”，研究过程反而失去恢复能力。

### Experiment Store 是下一版的地基

如果只能优先补一个模块，我会先做 Experiment Store，而不是增加 Agent。每个实验要有唯一身份，也要能追溯它从哪个版本分叉：

```text
Baseline
   ├─ exp_001 Feature A
   │     └─ exp_004 Feature A + B
   ├─ exp_002 Feature C
   └─ exp_003 Tune X
```

一条实验记录至少包含：

```json
{
  "experiment_id": "exp_0007",
  "parent_id": "exp_0005",
  "status": "success",
  "code_version": "xxx",
  "data_version": "xxx",
  "started_at": "...",
  "finished_at": "...",
  "hypothesis": "...",
  "action": "...",
  "metrics": {}
}
```

这样 Success/Failure Memory 不再是一段脱离来源的总结。任何结论都能回答“来自哪次实验、基于哪版代码和数据”，Rollback 与人工 Review 也才有可靠依据。

### 硬约束下沉到 Hook 和 Tool

“请记住检查数据泄漏”不应该继续占 Prompt。修改特征后自动跑血缘与禁用字段检查；训练前验证数据划分和配置；训练结束后检查返回码与 Artifact；最终交付前再做完整的 Leakage、复现性和代码质量检查。

如果基于 Claude Code 实现，我会把检查挂在对应生命周期上：

```text
修改 Feature Code 后
→ Feature Lineage Check + Forbidden Feature Check

执行训练前
→ Data Split Check + Config Validation

训练命令结束后
→ Exit Code Check + Artifact Validation

最终输出前
→ Leakage Check + Reproducibility + Code Quality
```

能由脚本给出确定答案的事情，就别再花一次模型调用。

### Agent 数量继续收缩

下一版我只会保留三个主要角色：

- **Research Leader**：读取 State，诊断瓶颈，提出假设，分配预算并负责 Re-planning。
- **ML Experiment Agent**：调用数据分析、特征工程、调参和运行实验等 Skills，完成具体修改与实验。
- **Diagnostic Subagent**：在独立上下文里读取大量预测结果、日志与样本，最后只返回结构化诊断。

Code Safety 不再是常驻 Agent，而是 Rules、Scripts、Hooks 与必要时的最终语义 Review。只有任务需要独立上下文、会产生大量中间信息或确实需要专业判断时，我才会新建 Subagent。

整个流程最终会接近这样：

```text
Human Goal + Constraints
          ↓
Research Leader 读取 Experiment State
          ↓
Diagnose + Dynamic Plan
          ↓
选择 Action + Budget
          ↓
Experiment Agent / Diagnostic Subagent
          ↓
Deterministic Execution
          ↓
Success + Artifact Gate
          ↓
Evaluation + State Update
          ↓
Research Leader Re-plan
          ↓
Next Loop / Finish
```

这套设计不是让 Agent 想做什么就做什么，而是允许它动态选择研究路线，同时把每次执行限制在可观察、可追溯、可回滚的环境里。

## 自动化到底有没有更快

第一版在局部任务上省了时间，但多 Agent 的串行协作拖慢了整体吞吐。第二版单轮更快，却发生过“系统显示完成三轮，实际三次训练都失败”的假效率。

第一版最容易制造一种错觉：每个 Agent 都在工作，所以整个系统应该很高效。实际上，数据分析 Agent 的节省可能被后续五次上下文加载和信息转述全部吃掉。尤其当任务无法并行时，局部能力提升并不等于端到端吞吐提升。

第二版的问题更麻烦。控制台可以连续显示 `Round 1 completed`、`Round 2 completed`、`Round 3 completed`，看起来速度飞快；如果三轮都读取了旧指标，它们对研究没有任何贡献，反而增加了人工排错成本。这种自动化甚至比手动实验更慢，因为人还要先拆穿它“已经完成”的假象。

所以我现在不太关心每小时跑了多少轮，更关心每小时产生了多少条可信、可复现、能帮助下一步决策的新证据。衡量一套 AutoML Workflow，至少要看这几个指标：

- **有效实验率**：成功运行、产物完整且能够进入比较的实验占比。
- **人工干预次数**：需要人纠正路径、旧结果、特征越界和错误诊断的频率。
- **重复实验率**：系统是否反复尝试已经被 Failure Memory 排除的方向。
- **Time-to-Useful-Result**：从给定任务到发现有效特征、标签问题或更优参数，需要多久。
- **人工 Review 成本**：最终结果是否仍需人重新检查数据泄漏、线上可用性和结果来源。

这里面我最看重 Time-to-Useful-Result。它不要求每次尝试都涨点：找到一批疑似错标样本、排除一个看似合理的特征方向、确认线上数据缺少某个字段，都算有用结果。好的研究系统不是永远成功，而是每次失败都能缩小问题空间。

自动化不是把“做实验”换成“检查 Agent”。只有开放式判断被放大、确定性错误被系统拦住，它才真正提高研发效率。

## 最后

回头看，这三种设计各自在回答不同的问题：

```text
Role-driven Multi-Agent：谁来做？
Loop-driven Workflow：按什么流程做？
State-driven Dynamic Workflow：基于现有证据，下一步最值得做什么？
```

我一开始想要的是一个能自动跑更多实验的系统。踩完这些坑以后，我更希望它像一个靠谱的算法研究助手：会根据结果修正判断，也不会因为路径幻觉、训练失败或旧文件污染，把整个研究方向带偏。

一句话总结：

> 让 Agent 保留研究的自由度，让 Workflow 保证实验的可信度。

## 参考资料

### Agent、Workflow 与 Claude Code

1. **Anthropic, Building Effective AI Agents**：Workflow、Agent 及常见 Agentic Workflow 模式。
2. **Anthropic, Effective Context Engineering for AI Agents**：长任务中的上下文管理与信息压缩。
3. **Claude Code Docs — Subagents**：角色、工具权限与独立上下文隔离。
4. **Claude Code Docs — Skills**：将可复用的工作方式封装为能力，而不是独立 Agent。
5. **Claude Code Docs — Hooks**：在生命周期节点执行确定性检查与阻断。
6. **Claude Agent SDK**：程序化使用 agent loop、工具与上下文管理能力。
7. **Anthropic, How We Built Our Multi-Agent Research System**：多 Agent 研究系统的规划、协作与可靠性问题。

### AutoML 与 Machine Learning Agent

8. **Huang et al., MLAgentBench, ICML 2024**：将机器学习实验本身作为 Agent 任务进行评估。
9. **Chan et al., MLE-bench, ICLR 2025**：从完整机器学习工程角度评估 Agent。
10. **Jiang et al., AIDE**：在候选代码空间中通过执行反馈持续探索。
11. **Hong et al., Data Interpreter**：数据科学任务中的动态规划与工具使用。
