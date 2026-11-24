# s16 Team Protocols

## 这章解决什么问题

让 team 通信从“自由文本”升级成“可跟踪、可配对的结构化握手协议”。

## 新增的核心状态

- `RequestStore`
- `request_id` 关联键
- shutdown / plan approval 两套协议状态

## 新增的工具/接口

- `shutdown_request`
- `shutdown_response`
- `plan_approval`
- 仍复用 `send_message` 和 inbox 体系

## 接到主循环哪里

- lead 和 teammate 都会在自己的 loop 中识别协议消息
- `RequestStore` 负责追踪请求从 pending 到 approved 或 rejected 的变化

## 和前面章节的边界

- 这章不是新增 team 成员本身
- 这章新增的是成员之间的结构化协作协议

## 最小时序

1. 一方发出带 `request_id` 的请求
2. 对方收到后处理并回应同一个 `request_id`
3. 系统更新请求状态
4. 发起方据此决定后续动作

## 自己重写时最小要保留什么

- request_id
- request store
- 协议 handler

## 一句话记忆

协议的关键不是“消息格式更复杂”，而是“请求和响应终于能被可靠关联”。
