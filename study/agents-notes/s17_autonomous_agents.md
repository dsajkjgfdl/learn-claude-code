# s17 Autonomous Agents

## 这章解决什么问题

让 teammate 在没有人手动点名时，也能自己找工作、认领任务并继续推进。

## 新增的核心状态

- 带 idle 轮询能力的 `TeammateManager`
- `RequestStore`
- 任务认领状态
- 压缩后的 identity re-injection

## 新增的工具/接口

- `idle`
- `claim_task`
- 同时继承 team 和 protocol 的既有工具

## 接到主循环哪里

- 正常工作结束后不一定退出，而是进入 idle phase
- idle 时会轮询 inbox 和 task board
- 如果 context 被压缩，会重新注入身份块再继续工作

## 和前面章节的边界

- 自主性不是“任意乱跑”
- 它仍然依赖任务板、邮箱和协议边界来保持可控

## 最小时序

1. teammate 完成当前显式工作
2. 调用 `idle`
3. 轮询新消息或未认领任务
4. 发现任务后 `claim_task`
5. 恢复到工作态

## 自己重写时最小要保留什么

- idle 轮询
- 安全的任务认领
- identity re-injection

## 一句话记忆

自主 agent 的关键不是“自己想做什么都行”，而是“空闲时也能按规则找活干”。
