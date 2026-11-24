# s10 System Prompt

## 这章解决什么问题

把 system prompt 从一大块硬编码字符串，改成有边界的组装流水线。

## 新增的核心状态

- `SystemPromptBuilder`
- prompt section 列表
- stable / dynamic 的边界意识

## 新增的工具/接口

- 没有新增面向用户的工具
- 真正新增的是 prompt assembly pipeline

## 接到主循环哪里

- 每次模型调用前先构建 system prompt
- tool、skill、memory、`CLAUDE.md`、dynamic context 被分段组装
- 更动态的提醒以独立内容注入，而不是硬塞进稳定 prompt

## 和前面章节的边界

- `memory` 是输入素材之一，不等于整个 prompt
- system prompt 不是“越长越好”的说明书

## 最小时序

1. 收集稳定信息
2. 收集较动态的信息
3. 分段拼出 system prompt
4. 再发起模型调用

## 自己重写时最小要保留什么

- 一个 builder
- 清晰的 section 切分
- 稳定区和动态区的边界

## 一句话记忆

prompt 不是一整坨字符串，而是一条有边界的输入装配流水线。
