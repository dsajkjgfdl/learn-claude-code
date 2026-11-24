# s12 Task System

## 这章解决什么问题

让工作目标脱离对话上下文，作为 durable task 保存在磁盘上。

## 新增的核心状态

- `TaskManager`
- `.tasks/*.json` 任务记录
- `blockedBy / blocks` 依赖关系

## 新增的工具/接口

- `task_create`
- `task_update`
- `task_list`
- `task_get`

## 接到主循环哪里

- 任务操作作为普通工具进入同一条 loop
- 任务状态落盘，因此不会随着上下文压缩一起丢失

## 和前面章节的边界

- `task` 不是 `todo`
- `task` 也不是后台命令执行槽位

## 最小时序

1. 创建任务
2. 建立依赖关系
3. 更新状态
4. 前置任务完成后解锁后续任务

## 自己重写时最小要保留什么

- durable task record
- 依赖关系
- 任务读写工具

## 一句话记忆

`task` 是 durable 工作图，不是当前会话里的待办列表。
