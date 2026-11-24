# Hermes Agent 与 `agents/s_full.py` 智能体实现方案对比报告

生成时间：2026-04-15  
对比对象：
- `D:\study\hermes-agent`
- `D:\study\learn-claude-code\agents\s_full.py`

## 1. 结论先行

这两个对象不是同一成熟度层级的“同类实现”。

- `agents/s_full.py` 更准确地说是“教学型总装参考”。仓库 README 明确把整个项目定义为 teaching repository，并建议把 `s_full.py` 放到最后阅读，用来把前面拆开的机制重新放回一张图里，而不是把它当成生产方案的等价物来读。
- `hermes-agent` 则是“产品型/平台型 agent runtime”。它的目标不是只演示主循环，而是把模型接入、工具注册、权限审批、会话持久化、上下文压缩、记忆、委派、网关、多平台入口都做成可运行系统。

如果只问一句话该怎么选：

- 想学习“一个 coding agent 的主干是怎么拼起来的”，`s_full.py` 更好。
- 想参考“一个长期运行、可扩展、可持久化、可多入口部署的 agent 平台怎么落地”，`hermes-agent` 明显更强。

## 2. 先把定位分清

### `s_full.py` 的定位

- 教学仓库 README 把项目定义为 “A teaching repository for implementers...”，并明确写了 “Read `s_full.py` last”。
- `s_full.py` 文件头把自己定义为 `Capstone Teaching Agent`，说明它本质上是“课程总装文件”。
- 因此，对 `s_full.py` 的评价应聚焦在：可读性、机制显式程度、最小正确实现、重构友好度。

### `hermes-agent` 的定位

- Hermes 文档把核心架构描述为 `AIAgent (run_agent.py)` + prompt builder + provider runtime + tool registry + session storage + gateway + plugins。
- 它覆盖 CLI、Gateway、ACP、Batch Runner、Cron、Plugins、Memory Provider、Context Engine、MCP 等多个入口和扩展面。
- 因此，对 Hermes 的评价应聚焦在：工程完备度、运行时治理、扩展能力、长期会话能力、产品化成本。

## 3. 总体对比

| 维度 | `agents/s_full.py` | `hermes-agent` | 判断 |
| --- | --- | --- | --- |
| 系统定位 | 教学型单文件总装 | 产品型多模块平台 | 不是同层级 |
| 代码组织 | 单文件集中 | 多入口、多目录、分层模块 | Hermes 更适合长期演进 |
| 主循环 | 直接 `messages.create()` + tool loop | `AIAgent.run_conversation()`，含 API mode、预算、fallback、回调 | Hermes 更完整 |
| 模型接入 | 基本只围绕 Anthropic client | `chat_completions` / `codex_responses` / `anthropic_messages` 三模式 | Hermes 更通用 |
| 工具系统 | `TOOLS` + `TOOL_HANDLERS` 手写映射 | `ToolRegistry` + toolset + availability check + dispatch + MCP/plugin discovery | Hermes 更可扩展 |
| 提示词系统 | 一个短 `SYSTEM` 字符串 | 分层 system prompt，区分缓存层和临时层 | Hermes 更稳定 |
| 上下文压缩 | 单次总结式 `auto_compact()` | 可插拔 context engine + tool-pair-safe compression | Hermes 更稳 |
| 记忆 | `s_full.py` 实际未实现持久记忆层 | 内置 memory + external memory provider manager | Hermes 更完整 |
| 会话持久化 | 本地文件碎片化持久化：`.tasks/.team/.transcripts` | SQLite + FTS5 + lineage + contention handling | Hermes 更适合长期会话 |
| 子智能体 | 同进程再开一个小 loop | 受限工具集、深度限制、并行 child、独立上下文 | Hermes 更安全、更可控 |
| 多智能体 | thread + inbox JSONL 原型 | 更偏“父 agent + 子 agent + gateway/session/runtime”体系 | `s_full.py` 更直观，Hermes 更真实 |
| 权限与安全 | `run_bash()` 仅字符串黑名单 | `approval.py`、危险模式、会话级审批、环境隔离 | Hermes 明显更强 |
| 扩展机制 | 几乎无插件边界 | tool/plugin/context-engine/memory-provider 多扩展点 | Hermes 更适合生态化 |
| 调试成本 | 低，容易全局阅读 | 高，行为分散在 prompt/config/hooks/registry/runtime | `s_full.py` 更容易读，Hermes 更难改 |

## 4. `agents/s_full.py` 的优点

### 4.1 可读性极高，主干机制暴露得很直接

优点最明显的一点，是你几乎可以在一个文件里看完整个最小 agent 主干：

- 基础文件与命令工具：`safe_path()`、`run_bash()`、`run_read()`、`run_write()`、`run_edit()`
- 规划：`TodoManager`
- 一次性子代理：`run_subagent()`
- 技能加载：`SkillLoader`
- 上下文压缩：`microcompact()`、`auto_compact()`
- 持久任务板：`TaskManager`
- 后台任务：`BackgroundManager`
- teammate/inbox：`MessageBus`、`TeammateManager`
- 真正主循环：`agent_loop()`

这对“理解 agent 如何从 while-loop 长出来”非常友好。

### 4.2 状态位置非常显式

`s_full.py` 的状态没有被隐藏在很多框架层里：

- Todo 在 `TodoManager.items`
- 任务在 `.tasks/task_*.json`
- 队友消息在 `.team/inbox/*.jsonl`
- 压缩转录在 `.transcripts`
- 大输出在 `.task_outputs/tool-results`

这种“状态落点一眼看见”的设计，非常适合学习和快速 debug。

### 4.3 原型速度快，改造成本低

因为没有复杂注册体系、配置层、数据库和插件装配层：

- 改一个工具 schema 很快
- 改主循环行为很快
- 增一个新状态结构很快
- 拿来做 PoC 或课程演示也很快

如果你的目标是“先跑起来，再决定哪些能力值得保留”，这类结构很合适。

### 4.4 多机制放在同一个认知平面里

单文件还有一个实际好处：不同机制之间的接线关系很容易看懂。

比如：

- 背景任务结果如何回流到消息历史
- inbox 如何在每轮 loop 前被消费
- `TodoWrite` 如何影响 reminder
- 手动压缩如何在当前 loop 末尾触发

这些在产品系统里通常分散到很多模块，初学者很难建立“控制平面”的整体感。

## 5. `agents/s_full.py` 的缺点

### 5.1 它是教学总装，不是完整产品 runtime

这是最重要的短板，也是最容易误判的地方。

文件头声称整合了 `s01-s18` 的核心机制，但在 `s_full.py` 里，下面这些名字只出现在头部映射中，并没有对应实现：

- `PermissionManager`
- `HookManager`
- `MemoryManager`
- `CronScheduler`
- `WorktreeManager`

也就是说：

- 对整个教学仓库来说，这些章节和实现是存在的
- 但对 `s_full.py` 这个“总装文件”本身来说，这些能力并没有真的并入

所以它更像“帮助理解全图的整合参考”，而不是“已经具备全部章节能力的最终 runtime”。

### 5.2 安全模型偏弱

`run_bash()` 使用 `subprocess.run(..., shell=True)`，只靠少量危险字符串黑名单拦截命令。这对教学足够，但对真实运行环境明显不够：

- 不具备完善的命令模式识别
- 没有审批回调
- 没有会话级授权状态
- 没有环境隔离后端
- 没有 destructive checkpoint

这意味着它不适合直接当成真实 coding agent 的执行面。

### 5.3 工具系统扩展性有限

`s_full.py` 的工具系统是典型的教学式写法：

- schema 列在 `TOOLS`
- 执行映射列在 `TOOL_HANDLERS`
- agent-level 行为在 loop 里手工分支

优点是直白，缺点是：

- 工具数量一多，维护成本快速上升
- 动态可用性检查没有独立抽象
- 插件工具和外部工具很难自然接进来
- 很难支撑多 toolset、多平台 preset、多后端选择

### 5.4 持久化能力只够原型，不够长期运行

`TaskManager`、`MessageBus`、`TeammateManager` 的持久化方式非常轻：

- 任务是单文件 JSON
- inbox 是 JSONL
- teammate 生命周期靠线程 + 配置文件

这在教学中很好懂，但在真实系统里会遇到：

- 并发一致性脆弱
- 恢复语义有限
- 查询能力弱
- 缺少跨会话搜索
- 难做平台隔离和 lineage

### 5.5 提示词与运行时治理过于简化

`SYSTEM` 只有很短的几行指令，虽然够演示 loop，但不够支撑复杂运行时约束：

- 没有分层 prompt assembly
- 没有 cached vs ephemeral prompt 的清晰边界
- 没有平台提示、环境提示、上下文文件优先级体系
- 没有 memory snapshot / user profile snapshot
- 没有针对模型族的差异化执行 guidance

结果是：能跑，但难稳。

### 5.6 子智能体和 teammate 更偏“概念演示”

`run_subagent()` 本质上仍是同进程内再次跑一个小 loop；`TeammateManager` 是 thread + inbox 文件 + auto-claim 轮询。

这很好地说明了“什么是新上下文”“什么是隔离任务”，但距离真实系统的多 agent 运行时还有距离：

- 没有真正的运行环境隔离
- 没有独立权限面
- 没有独立会话数据库
- 没有细粒度中断治理
- 没有严格的资源配额

## 6. Hermes Agent 的优点

### 6.1 架构面完整，是真正的产品型 agent runtime

Hermes 不是只把 loop 跑通，而是把一个 agent 平台需要的外围控制面也做了：

- `AIAgent` 负责主循环、API mode、重试、fallback、工具执行、压缩、持久化
- `prompt_builder.py` 负责 system prompt 分层组装
- `tools/registry.py` 负责工具注册和派发
- `delegate_tool.py` 负责子代理委派
- `hermes_state.py` 负责 SQLite 会话库和 FTS5 搜索
- `gateway/` 负责多平台消息入口
- `plugins/` 负责 memory provider / context engine 等扩展

这一层次明显更接近“真实可部署系统”。

### 6.2 模型接入与运行时兼容能力强

Hermes 的 `AIAgent` 不绑定单一 provider，而是内置三种 API mode：

- `chat_completions`
- `codex_responses`
- `anthropic_messages`

这直接带来两个优势：

- 不同模型/厂商切换时，不需要重写整套 agent loop
- 工具调用、消息转换、缓存、响应解析可以在统一内部消息格式上收敛

这类抽象在产品系统里非常关键。

### 6.3 Prompt 体系明显更成熟

Hermes 把 system prompt 当作“分层构件”，而不是一段大字符串。它会组合：

- `SOUL.md` 身份层
- tool-aware guidance
- memory/user profile snapshot
- skills index
- context files
- timestamp/session/provider/model
- platform hint

并且显式区分：

- 缓存稳定的 system prompt
- 仅在 API 调用时注入的临时层

这比 `s_full.py` 的简单 `SYSTEM` 更适合长会话和多平台运行。

### 6.4 工具系统是真正可扩展的

Hermes 的工具不是一大坨手工映射，而是：

- 每个 tool 模块自注册到 `ToolRegistry`
- `get_definitions()` 做 availability check
- `dispatch()` 做同步/异步桥接和错误包装
- `model_tools.py` 负责 schema 汇总、toolset 解析、MCP/plugin discovery

这带来的收益非常实际：

- 新工具接入边界清晰
- 可以按 toolset 控制暴露面
- 可以动态过滤不可用工具
- 可以自然接 MCP 和 plugin tools

### 6.5 委派方案更接近真实生产实现

Hermes 的 `delegate_task` 不只是“再开一个 loop”，而是有清晰治理规则：

- child 拿到 fresh conversation
- toolset 会被裁剪
- 一组危险工具默认禁止给 child
- 有 delegation depth limit
- 支持并行 child agents
- parent 只接收 child 总结，不吃掉中间过程上下文

这是更成熟的“上下文隔离 + 预算治理 + 能力收敛”的做法。

### 6.6 会话和历史能力远强于原型实现

`hermes_state.py` 使用 SQLite + FTS5，并支持：

- session metadata
- message history
- tool_calls/reasoning 持久化
- session lineage
- full-text search
- contention handling

这意味着 Hermes 天然适合：

- 长期会话
- 断点恢复
- 跨会话回忆
- 网关与 CLI 共用底层存储

### 6.7 安全和治理更完整

Hermes 明确把权限与安全当成运行时一等公民，而不是简单 if/else：

- `tools/approval.py` 负责危险命令检测和审批
- terminal 支持多环境后端
- destructive command 前可做 checkpoint
- subagent 会被裁剪危险工具
- context / URL / website access 也有独立安全检查

这类治理能力是从“能跑”迈向“能托管”的关键差异。

## 7. Hermes Agent 的缺点

### 7.1 复杂度和认知门槛很高

Hermes 最明显的问题不是能力弱，而是能力太多，导致认知成本高。

很多行为被分散到：

- `run_agent.py`
- `prompt_builder.py`
- `model_tools.py`
- `tools/*`
- `plugins/*`
- `gateway/*`
- `config.yaml`
- callbacks / hooks / runtime providers

因此你很难像读 `s_full.py` 一样，在一个地方把“这轮请求到底怎么走”全看完。

### 7.2 大文件热点明显

Hermes 文档自己也承认：

- `run_agent.py` 是一个约 9,200 行的核心类热点
- `cli.py` 也是数千行级别入口

这意味着：

- 修改风险集中
- 行为耦合点多
- 回归面大
- 阅读路径长

从长期维护角度看，这种热点文件需要很强的工程纪律。

### 7.3 行为来源分散，排障更难

一个行为可能同时受下面几层影响：

- system prompt 指导
- runtime branch
- registry dispatch
- plugin hook
- tool-specific check
- gateway/platform callback
- config override

所以 Hermes 的问题很多时候不是“不会做”，而是“出问题后需要花更久确定是谁在起作用”。

### 7.4 对二开者不够轻

如果你的目标只是做一个“本地 coding agent”，Hermes 可能会显得偏重：

- provider/runtime abstraction 太全
- gateway/platform 接线太多
- session/memory/plugin 面太广
- 配置表面积大

这类系统非常适合作为成熟参考，但不一定适合直接照搬。

## 8. 哪种方案适合什么场景

### 更适合从 `s_full.py` 起步的场景

- 你要自己从 0 到 1 手搓 agent
- 你最关心主循环、工具回路、最小状态机
- 你要做课程、演示、PoC、实验性原型
- 你准备先做对，再慢慢补安全和持久化
- 你希望重构时能一眼看到所有核心状态

### 更适合参考 Hermes 的场景

- 你要做长期运行的 agent 产品
- 你需要多模型/多 provider 接入
- 你需要会话持久化、搜索、恢复
- 你需要多入口：CLI、Gateway、IDE/ACP、Cron
- 你需要工具插件化、MCP、上下文引擎、记忆 provider
- 你需要更可靠的审批、隔离和并发执行治理

## 9. 如果你要吸收两者优点，建议怎么组合

最实际的路线不是二选一，而是“用 `s_full.py` 的清晰骨架 + 借 Hermes 的治理结构”：

### 可以保留 `s_full.py` 的东西

- 单轮 loop 的显式结构
- 明确可见的状态对象
- 简单直接的任务板思维
- 教学友好的工具回路
- 对子代理/团队协作的最小心智模型

### 应该向 Hermes 借的东西

- provider/api mode 抽象
- 分层 prompt builder
- 注册式工具系统
- availability check 与 toolset 过滤
- delegation guardrails
- 会话数据库与搜索
- 权限审批与危险命令检测
- 可插拔 context engine / memory provider

如果你打算把当前教学版推进成更像真实系统，我认为优先级应当是：

1. 先补权限审批和危险命令 gate  
2. 再把工具系统从“手工映射”升级成“注册式 registry”  
3. 再补会话数据库和搜索  
4. 再考虑插件化和多入口  

## 10. 关键证据位置

### `agents/s_full.py`

- 教学总装定位：`D:\study\learn-claude-code\agents\s_full.py:3`
- 头部映射中提到但文件内未落地的子系统名：`D:\study\learn-claude-code\agents\s_full.py:20` 到 `:31`
- 基础安全边界与 `run_bash()`：`D:\study\learn-claude-code\agents\s_full.py:130`、`:136`
- 子代理原型：`D:\study\learn-claude-code\agents\s_full.py:221`
- 压缩逻辑：`D:\study\learn-claude-code\agents\s_full.py:317`
- 任务板：`D:\study\learn-claude-code\agents\s_full.py:354`
- 后台任务：`D:\study\learn-claude-code\agents\s_full.py:420`
- inbox / teammate：`D:\study\learn-claude-code\agents\s_full.py:456`、`:491`
- 系统提示词：`D:\study\learn-claude-code\agents\s_full.py:645`
- 主循环：`D:\study\learn-claude-code\agents\s_full.py:746`

### 教学仓库 README

- teaching repository 定位：`D:\study\learn-claude-code\README.md:5`
- 明确建议最后再看 `s_full.py`：`D:\study\learn-claude-code\README.md:204`

### Hermes Agent

- 核心 `AIAgent`：`D:\study\hermes-agent\run_agent.py:492`
- 分层 system prompt：`D:\study\hermes-agent\run_agent.py:3057`
- 工具调用执行与并发：`D:\study\hermes-agent\run_agent.py:6702`、`:6725`、`:6799`
- 主对话入口：`D:\study\hermes-agent\run_agent.py:7543`
- 工具注册中心：`D:\study\hermes-agent\tools\registry.py:48`、`:59`、`:116`、`:149`
- 子代理治理：`D:\study\hermes-agent\tools\delegate_tool.py:32`、`:53`、`:90`、`:238`
- 技能与上下文文件组装：`D:\study\hermes-agent\agent\prompt_builder.py:563`、`:873`、`:986`
- 外部记忆管理器：`D:\study\hermes-agent\agent\memory_manager.py:72`、`:146`、`:234`、`:238`
- 上下文压缩器：`D:\study\hermes-agent\agent\context_compressor.py:60`、`:177`、`:666`
- 会话数据库与搜索：`D:\study\hermes-agent\hermes_state.py:115`、`:791`、`:886`、`:990`
- 架构文档总览：`D:\study\hermes-agent\website\docs\developer-guide\architecture.md`

## 11. 最后的判断

如果你现在的目标是“判断哪个实现方案更高级”，答案是 Hermes。  
如果你的目标是“判断哪个更适合作为自己重构和吸收的起点”，答案通常是 `s_full.py`。  
如果你的目标是“做一个真正可长期运行的 agent 系统”，最合理的路径是：先用 `s_full.py` 抓住主干，再按 Hermes 的方式逐步补齐控制面。

## 12. 智能体模块通用性分层图

这一节不再按仓库章节顺序划分，而是按“脱离具体业务后是否仍然成立”来划分。

判断原则只有两条：

1. 没有这个模块，智能体还能不能完成“接收任务 -> 决策 -> 调工具/执行 -> 返回结果”的最小闭环？  
2. 这个模块解决的是“智能体核心认知问题”，还是“规模化、异步化、渠道化、平台化”问题？

### 12.1 分层图

```text
+------------------------------------------------------------------+
| L1 通用核心层：几乎所有业务智能体都需要                           |
|------------------------------------------------------------------|
| 1. Agent Loop / Orchestrator                                     |
| 2. Prompt / Policy Assembly                                      |
| 3. Tool / Capability Abstraction                                 |
| 4. Context / Session State                                       |
| 5. Error Recovery / Retry Boundary                               |
| 6. Safety Gate  (当智能体有外部执行能力时即为通用必需)            |
+------------------------------------------------------------------+
                              |
                              v
+------------------------------------------------------------------+
| L2 常见增强层：高频有用，但不是所有场景都必需                     |
|------------------------------------------------------------------|
| 1. Todo / Planning                                               |
| 2. Session Persistence / Search                                  |
| 3. Clarification / Approval Workflow                             |
| 4. Checkpoint / Rollback                                         |
| 5. Background Execution                                          |
| 6. Hook / Extension Points                                       |
+------------------------------------------------------------------+
                              |
                              v
+------------------------------------------------------------------+
| L3 场景专项层：只在特定业务或产品形态下需要                       |
|------------------------------------------------------------------|
| 1. Long-term Memory / User Profile                               |
| 2. Skills / Progressive Disclosure                               |
| 3. Durable Task Board / Dependency Graph                         |
| 4. Cron / Scheduled Automation                                   |
| 5. Subagent / Agent Teams / Team Protocols / Autonomous Agents   |
| 6. Messaging Gateway / Multi-platform Delivery                   |
| 7. Worktree Isolation                                            |
| 8. MCP / Plugin Ecosystem                                        |
| 9. Voice / Vision / Browser / ACP / IDE Integration              |
+------------------------------------------------------------------+
```

### 12.2 模块归类表

| 模块 | 通用性层级 | 为什么 |
| --- | --- | --- |
| Agent Loop / Orchestrator | L1 通用核心 | 没有主循环就没有 agent，所有可执行智能体都需要 |
| Prompt / Policy Assembly | L1 通用核心 | 任何业务目标、角色、约束都必须进入模型 |
| Tool / Capability Abstraction | L1 通用核心 | 只要不是纯聊天，智能体就需要动作层 |
| Context / Session State | L1 通用核心 | 多步决策必须保留上下文，否则无法持续推进 |
| Error Recovery / Retry Boundary | L1 通用核心 | 模型失败、工具失败、超时、中断都必须收敛处理 |
| Safety Gate | L1 通用核心 | 只要有副作用操作，就需要统一安全边界 |
| Todo / Planning | L2 常见增强 | 复杂任务很有用，但单步场景可以不要 |
| Session Persistence / Search | L2 常见增强 | 长会话和跨会话很重要，单次执行 agent 可省略 |
| Clarification / Approval Workflow | L2 常见增强 | 面向真实用户和高风险任务很重要，封闭流水线可弱化 |
| Checkpoint / Rollback | L2 常见增强 | 对代码编辑和命令执行很有价值，但不是所有业务都需要 |
| Background Execution | L2 常见增强 | 长任务和异步任务有用，短任务不必做 |
| Hook / Extension Points | L2 常见增强 | 对长期演进和产品扩展有价值，但不是最小闭环必需 |
| Long-term Memory / User Profile | L3 场景专项 | 只有长期陪伴、个性化、跨会话学习时才真正需要 |
| Skills / Progressive Disclosure | L3 场景专项 | 适合大知识库和专家工作流，不是所有业务都需要 |
| Durable Task Board / Dependency Graph | L3 场景专项 | 适合项目管理、并行协作、复杂依赖任务 |
| Cron / Scheduled Automation | L3 场景专项 | 只有定时任务业务才需要 |
| Subagent / Agent Teams / Team Protocols | L3 场景专项 | 只有复杂任务需要分工协作时才值得引入 |
| Autonomous Agents | L3 场景专项 | 只有“主动找活”“自主巡检”类业务才需要 |
| Messaging Gateway / Multi-platform Delivery | L3 场景专项 | 这是交付渠道，不是认知内核 |
| Worktree Isolation | L3 场景专项 | 主要是 coding agent 的并行开发需求 |
| MCP / Plugin Ecosystem | L3 场景专项 | 只有要做开放能力平台时才值得抽象成一层 |
| Voice / Vision / Browser / ACP / IDE Integration | L3 场景专项 | 都属于模态或接入面增强，不是通用基础能力 |

### 12.3 映射到当前教学仓库章节

#### L1 通用核心

- `s01 Agent Loop`
- `s02 Tool Use`
- `s06 Context Compact`
- `s07 Permission System`
- `s10 System Prompt`
- `s11 Error Recovery`

#### L2 常见增强

- `s03 TodoWrite`
- `s08 Hook System`
- `s13 Background Tasks`

说明：`s09 Memory System` 在很多产品里很常见，但从“所有业务都需要”这个标准看，它更接近 L3。

#### L3 场景专项

- `s04 Subagent`
- `s05 Skill Loading`
- `s09 Memory System`
- `s12 Task System`
- `s14 Cron Scheduler`
- `s15 Agent Teams`
- `s16 Team Protocols`
- `s17 Autonomous Agents`
- `s18 Worktree Task Isolation`
- `s19 MCP Plugin`

### 12.4 一句话判断法

如果一个模块解决的是“智能体怎么思考、怎么行动、怎么保持闭环”，它大概率属于 L1。  
如果一个模块解决的是“智能体怎么更稳、更长、更方便维护”，它通常属于 L2。  
如果一个模块解决的是“智能体怎么协作、怎么跨渠道、怎么定时、怎么插件化、怎么多模态”，它通常属于 L3。
