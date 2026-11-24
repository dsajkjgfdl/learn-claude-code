# s07 Permission System

## 这章解决什么问题

在“模型想做什么”和“系统真的去执行什么”之间插入明确的控制门。

## 新增的核心状态

- `BashSecurityValidator`
- `PermissionManager`
- 权限模式：`default / plan / auto`

## 新增的工具/接口

- 对用户来说工具列表几乎没变
- 真正新增的是每次工具执行前的 permission pipeline

## 接到主循环哪里

- 每个工具调用在真正执行前，都会经过 deny -> mode -> allow -> ask user

## 和前面章节的边界

- 权限不是布尔开关
- 权限也不是 hook

## 最小时序

1. 模型发出工具调用
2. 系统先跑权限判断
3. 允许则执行
4. 不允许则阻断或请求确认

## 自己重写时最小要保留什么

- deny 规则
- mode 判断
- allow 规则
- 最后的用户确认分支

## 一句话记忆

安全不是一个 `if allowed`，而是一条分层的执行管道。
