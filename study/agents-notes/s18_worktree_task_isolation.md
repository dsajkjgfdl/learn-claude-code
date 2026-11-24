# s18 Worktree Task Isolation

## 这章解决什么问题

让多个任务可以并行执行，但目录彼此隔离，不互相踩文件。

## 新增的核心状态

- `EventBus`
- `TaskManager`
- `WorktreeManager`
- `.worktrees/index.json`
- task 和 worktree 的绑定关系

## 新增的工具/接口

- `task_bind_worktree`
- `worktree_create`
- `worktree_list`
- `worktree_enter`
- `worktree_status`
- `worktree_run`
- `worktree_closeout`
- `worktree_remove`
- `worktree_keep`
- `worktree_events`

## 接到主循环哪里

- 任务板负责目标和归属
- worktree 工具负责进入某条执行车道并在对应目录里运行命令
- event bus 让工作树生命周期可观察

## 和前面章节的边界

- task 是控制面对象
- worktree 是执行面对象
- 两者相关，但不是同一个东西

## 最小时序

1. 创建任务
2. 给任务绑定 worktree
3. 进入该目录车道工作
4. 在该 worktree 中执行命令
5. 完成后 keep 或 remove

## 自己重写时最小要保留什么

- worktree 注册表
- task 到 lane 的绑定
- 进入、运行、收尾三段生命周期

## 一句话记忆

任务管目标，worktree 管执行车道；不要把两者混成一个状态对象。
