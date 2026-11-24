# s02 Tool Use

## 这章解决什么问题

让同一条主循环能触达更多真实能力，而不用重写循环本身。

## 新增的核心状态

- 这一章几乎没有新增复杂状态
- 关键新增是 `TOOL_HANDLERS`、`safe_path()` 和 `normalize_messages()`

## 新增的工具/接口

- `bash`
- `read_file`
- `write_file`
- `edit_file`

## 接到主循环哪里

- 模型调用前先做 `normalize_messages()`
- 模型返回 `tool_use` 后，用 dispatch map 找到对应 handler

## 和前面章节的边界

- 主循环没有变
- 变化的是“能调用什么工具”和“工具如何分发”

## 最小时序

1. 用户提出需求
2. 模型选择具体工具
3. dispatch map 执行对应 handler
4. 结果继续以 `tool_result` 回流

## 自己重写时最小要保留什么

- 一个工具注册表
- 一个安全的工作目录边界
- 一次统一的消息规范化

## 一句话记忆

这章的关键不是“多了几个工具”，而是“主循环不变，能力靠分发表扩张”。
