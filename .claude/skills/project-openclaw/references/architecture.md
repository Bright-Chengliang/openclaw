# OpenClaw 架构设计

OpenClaw 采用典型的网关/控制平面架构，核心是一个中心化的 Daemon 进程，负责协调外部消息平台与内部 AI Agent 运行时。

## 顶层系统架构

```text
Chat Apps (WhatsApp/Telegram/Slack/Discord/Signal/iMessage/WebChat等)
                    |
                    v
    +-------------------------------+
    |           Gateway             |
    |       (Control Plane)         |
    |    ws://127.0.0.1:18789       |
    +---------------+---------------+
                    |
        +-----------+-----------+
        |     |     |     |     |
     Pi Agent CLI  WebChat macOS iOS/Android
      (RPC)                App   Nodes
```

## 核心设计理念

1. **Single Control Plane**：网关负责统一处理所有的会话 (Sessions)、路由 (Routing) 和工具调用，屏蔽了底层的通道差异。
2. **插件化通道集成**：所有的 Chat App 平台都通过 `extensions/` 机制作为插件接入，Gateway 通过统一的接口生命周期管理它们。
3. **嵌入式 Agent 运行时**：整合了 `pi-mono` 生态系统的 Agent，但不强绑定会话逻辑。会话发现、工具连线由 OpenClaw 接管。
4. **客户端与节点模型 (Client/Node)**：
   - 客户端通过 WebSocket 连接 Gateway。
   - 移动端或桌面端可以作为 Node (节点) 接入，提供本地设备能力（如 `canvas.*`, `camera.*`, `screen.record`, `location.get` 等）。

## 进程与协议架构

### Gateway Daemon
- 唯一长驻进程，单节点部署。
- 监听 HTTP 和 WebSocket（默认端口 18789）。
- WebSocket 提供强类型的通信协议。

### WebSocket Wire Protocol
- **Transport**: 基于文本帧的 JSON 负载。
- **Handshake**: 必须以 `connect` 帧开始，包含设备绑定、Auth 签名 (v3) 等安全校验。
- **Request/Response**: `{type:"req", id, method, params}` -> `{type:"res", id, ok, payload|error}`
- **Server Push (Events)**: `{type:"event", event, payload, seq?, stateVersion?}` (如 `agent`, `chat`, `presence`, `health` 事件)
- **幂等性保障**：Side-effect 操作支持 Idempotency Keys 校验。

## 目录与模块分布 (基于 npm 包结构)

- `dist/entry.js`: 系统启动编排，负责加载 Config、CLI、Gateway Server、Extensions。
- `dist/plugin-sdk/`: 插件 SDK，分离了网关核心实现与插件开发接口。
- `extensions/`: 存放独立的通信通道和能力插件，被 Gateway 动态加载。
- `skills/`: 定义了通过统一 Schema 被 Agent 发现和调用的工具集合。
