# s15 Agent Teams

## 这章解决什么问题

把一次性的 subagent 发展成持续存在、可收发消息的 teammate。

## 新增的核心状态

- `MessageBus`
- `TeammateManager`
- `.team/config.json`
- `.team/inbox/*.jsonl` 邮箱文件

## 新增的工具/接口

- `spawn_teammate`
- `list_teammates`
- `send_message`
- `read_inbox`
- `broadcast`

## 接到主循环哪里

- lead 负责生成和管理 teammates
- 每个 teammate 在独立线程里跑自己的 loop
- 通信通过 append-only inbox 文件完成

## 和前面章节的边界

- `teammate` 不是 `subagent` 放大版
- `subagent` 是一次性执行，`teammate` 会持续存在、空闲、再继续工作

## 最小时序

1. lead 生成 teammate
2. teammate 获得自己的 inbox 和线程
3. lead 发送消息到 inbox
4. teammate 读信、工作、回信
5. 工作完可回到 idle

## 自己重写时最小要保留什么

- 持久命名成员
- 文件邮箱
- 独立 agent loop

## 一句话记忆

从这章开始，系统里出现了“长期 actor”，而不只是“一次性工具调用”。
