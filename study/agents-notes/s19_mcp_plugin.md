# s19 MCP Plugin

## 这章解决什么问题

把外部进程暴露出来的工具，接回到同一条 agent 工具控制面里。

## 新增的核心状态

- `CapabilityPermissionGate`
- `MCPClient`
- `PluginLoader`
- `MCPToolRouter`

## 新增的工具/接口

- 不再只靠固定本地工具列表
- 外部 MCP tools 会被发现、前缀化、注册进统一 tool pool

## 接到主循环哪里

- 构建 tool pool 时合并本地工具和外部工具
- 调用外部工具时仍然先过权限门，再由 router 转给对应 MCP server
- 返回结果再被标准化成统一 `tool_result`

## 和前面章节的边界

- plugin manifest 不是 MCP server
- MCP server 不是某一个具体 tool
- 外部能力不会绕过本地权限控制

## 最小时序

1. 启动外部 server
2. 查询它有哪些 tools
3. 给这些 tools 加前缀并注册
4. 模型调用外部 tool
5. router 转发并规范化结果

## 自己重写时最小要保留什么

- server discovery
- 权限 gate
- tool routing
- 结果规范化

## 一句话记忆

MCP 真正重要的不是“接了外部能力”，而是“外部能力仍然走同一条控制面”。
