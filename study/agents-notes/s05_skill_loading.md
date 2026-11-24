# s05 Skill Loading

## 这章解决什么问题

让模型廉价地知道“有哪些专长可用”，再按需加载完整内容。

## 新增的核心状态

- `SkillManifest`
- `SkillDocument`
- `SkillRegistry`

## 新增的工具/接口

- `load_skill`

## 接到主循环哪里

- system prompt 先放便宜的 skill catalog
- 需要细节时再通过 `load_skill` 读取完整 skill body

## 和前面章节的边界

- 这不是把所有技能全文塞进 prompt
- 也不是把知识全靠模型记忆

## 最小时序

1. 模型先看到技能目录
2. 判断当前任务是否需要某个 skill
3. 调用 `load_skill`
4. 把加载到的内容用于后续行动

## 自己重写时最小要保留什么

- skill 元数据
- skill 正文存储
- 按需加载接口

## 一句话记忆

这章教的是“两层技能模型”：先发现，再按需加载。
