# s04 Subagent

## 这章解决什么问题

在不污染父上下文的前提下，把探索或子任务隔离出去。

## 新增的核心状态

- `AgentTemplate`
- 子 agent 的 `messages=[]` 新上下文

## 新增的工具/接口

- `task`：启动一个 fresh-context 子 agent

## 接到主循环哪里

- 父 agent 在工具分发时调用 `task`
- 子 agent 在同一进程里跑自己的 loop
- 最后只把摘要结果带回父 agent

## 和前面章节的边界

- `subagent` 不是长期成员
- 它的重点是上下文隔离，不是并行团队协作

## 最小时序

1. 父 agent 发现需要隔离探索
2. 调用 `task`
3. 子 agent 用全新 `messages` 工作
4. 子 agent 返回摘要
5. 父 agent 继续自己的上下文

## 自己重写时最小要保留什么

- fresh `messages=[]`
- 独立工具执行回路
- 只返回摘要，不回放完整对话

## 一句话记忆

`subagent` 的本质是上下文隔离，不是“另开一个更大的 agent”。
