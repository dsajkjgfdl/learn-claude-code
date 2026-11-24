# s13 Background Tasks

## 这章解决什么问题

把慢命令放到后台跑，让主循环不用一直阻塞等待。

## 新增的核心状态

- `BackgroundManager`
- 后台任务表
- 完成通知队列
- `.runtime-tasks/*.json` 和 `.log`

## 新增的工具/接口

- `background_run`
- `check_background`

## 接到主循环哪里

- 模型调用前先 `drain_notifications()`
- 已完成的后台结果会以摘要形式重新注入对话

## 和前面章节的边界

- `background task` 是运行时执行槽位
- 它不是 `s12` 的 durable task board

## 最小时序

1. 模型调用 `background_run`
2. 系统立即返回 `task_id`
3. 后台线程真正执行命令
4. 结果写日志并推入通知队列
5. 下一轮再把摘要带回模型

## 自己重写时最小要保留什么

- 后台线程
- 任务状态表
- 通知回流机制

## 一句话记忆

并行的是等待和执行，不是主循环本身。
