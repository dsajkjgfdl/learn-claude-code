# s09 Memory System

## 这章解决什么问题

让一些跨会话仍然有价值的信息活下来，但不把 memory 变成垃圾堆。

## 新增的核心状态

- `MemoryManager`
- `.memory/*.md` 单条记忆文件
- `MEMORY.md` 索引
- `DreamConsolidator` 作为后续整理入口

## 新增的工具/接口

- `save_memory`

## 接到主循环哪里

- 会话开始时加载 `.memory/`
- `save_memory` 写盘并重建索引
- 每轮 `build_system_prompt()` 时把 memory section 注入 prompt

## 和前面章节的边界

- `memory` 不是代码结构缓存
- `memory` 不是当前任务状态
- `memory` 也不是上下文压缩

## 最小时序

1. 用户说出长期偏好或重要非显然事实
2. 模型调用 `save_memory`
3. 记忆落到 `.memory/`
4. 下次会话或下一轮重新加载进 prompt

## 自己重写时最小要保留什么

- 一条记忆一个文件
- 一个索引文件
- 一个注入 prompt 的入口

## 一句话记忆

memory 只保存“以后还值得记住、但当前仓库不容易直接重新看出来”的信息。
