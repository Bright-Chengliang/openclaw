# 数据模型与存储结构

## 配置文件

### 主配置 (`~/.openclaw/openclaw.json`)
- 格式: JSON5
- 内容: 网关端口、通道启用/禁用、Agent 默认配置、模型选择、认证凭据引用等

### Agent 配置
- 路径: `~/.openclaw/agents/<agentId>/`
- Workspace Bootstrap 文件: `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`
- 这些文件构成 Agent 的系统提示词上下文

## 会话存储

### 会话索引 (`sessions.json`)
- 路径: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- 记录所有活跃会话的元数据（Session Key、最后活跃时间、通道来源等）

### 对话转录 (`<SessionId>.jsonl`)
- 路径: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`
- 格式: Append-only JSONL
- 每行记录一个对话事件（用户消息、Agent 回复、工具调用及结果）

### 会话作用域 (Session Scoping)
- `main`: 默认共享会话（所有私聊共用）
- `per-peer`: 每个用户独立会话
- `per-channel-peer`: 每个通道+用户组合独立会话
- `per-account-channel-peer`: 最细粒度隔离

## WebSocket 协议帧格式

### Request
```json
{ "type": "req", "id": "<uuid>", "method": "...", "params": {...} }
```

### Response
```json
{ "type": "res", "id": "<uuid>", "ok": true, "payload": {...} }
```
```json
{ "type": "res", "id": "<uuid>", "ok": false, "error": {...} }
```

### Server Push Event
```json
{ "type": "event", "event": "<name>", "payload": {...}, "seq": 42, "stateVersion": "..." }
```

事件类型包括: `agent`, `chat`, `presence`, `health`, `heartbeat`, `cron`

### 认证握手
- 首帧必须为 `connect` 类型
- 支持 v3 签名协议（含 platform + deviceFamily 绑定）
- Side-effect 操作支持 Idempotency Key

## Extension Manifest

每个扩展的 `package.json` 包含:
```json
{
  "name": "@openclaw/<channel-name>",
  "openclaw": {
    "extensions": ["./index.ts"]
  }
}
```

## Skill Manifest (`SKILL.md`)

YAML Frontmatter 定义:
- `name`: 技能标识
- `description`: 触发描述
- `emoji`: 显示图标
- `requirements.bins`: 依赖的系统命令
- `requirements.config`: 需要的配置项

Markdown 主体提供 Agent 可执行的指令和上下文。

## 构建产物元信息 (`dist/build-info.json`)

记录构建时的版本号、Git commit hash、构建时间戳，供运行时版本检测使用。
