# s14 Cron Scheduler

## 这章解决什么问题

让 agent 不只会“现在做事”，还会“在未来某个时间点触发工作”。

## 新增的核心状态

- `CronLock`
- `CronScheduler`
- session-only 或 durable 的 schedule 存储
- recurring 和 one-shot 两种触发模式

## 新增的工具/接口

- `cron_create`
- `cron_delete`
- `cron_list`

## 接到主循环哪里

- 主循环旁边有调度检查
- 命中时间的 schedule 会把 prompt 注入回对话，再交给同一个 loop 处理

## 和前面章节的边界

- `cron` 决定“什么时候开始”
- `background` 决定“开始以后怎么跑”

## 最小时序

1. 创建 schedule
2. 调度器周期性检查当前时间
3. 命中时生成触发消息
4. one-shot 删除，recurring 保留等待下次触发

## 自己重写时最小要保留什么

- cron 匹配逻辑
- schedule 存储
- 触发后的注入路径

## 一句话记忆

调度器解决的是“未来何时开始”，不是“开始以后如何执行”。
