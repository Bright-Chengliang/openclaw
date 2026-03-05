# API 与 Plugin SDK 接口清单

`dist/plugin-sdk/` 是暴露给扩展开发者的核心 API 接口层。所有的内部功能均通过该 SDK 被安全导出。

## 核心 SDK 模块定义

通过 `dist/plugin-sdk/index.d.ts` 等文件暴露：

### 1. `runtime` (运行时环境)
- `RuntimeEnv` 类型: 提供了基础的运行时上下文，包含 `log`, `error`, `exit` 方法。
- `defaultRuntime` / `createNonExitingRuntime`: 用于初始化安全的执行环境。

### 2. `logger` & `logging` (结构化日志)
- 级别方法：`logInfo`, `logWarn`, `logSuccess`, `logError`, `logDebug`
- 通过 chalk 支持带有成功、警告、危险主题的日志打印。

### 3. `utils` (公共工具库)
包含大量高频使用的文件与数据处理方法：
- **文件系统**: `ensureDir`, `pathExists`
- **数据结构与解析**: `safeParseJson`, `isPlainObject`, `isRecord`
- **特定领域处理**:
  - WhatsApp JID 格式化与 E164 规范化。
  - UTF-16 安全的字符串截断 (Slicing)。
- **环境配置**: `CONFIG_DIR` 解析，主目录寻找等。
- **其他**: `sleep`, `clamp`, `escapeRegExp`。

### 4. `account-id` & `version`
- 处理系统内部通用的账号标识逻辑。
- 获取当前系统 `VERSION` 常量（源自 build-info）。

## WebSocket 通信 API 协议

开发者如果要编写新的 Client 或 Node 连接到 Gateway，需要遵循其 TypeBox 校验的协议：

- **连接端点**: `ws://<host>:18789`
- **Payload 格式**: 基于纯文本的 JSON。
- **协议分类**:
  1. `req` (Request): 请求调用特定方法。
     `{ "type": "req", "id": "<uuid>", "method": "...", "params": {...} }`
  2. `res` (Response): 响应结果。
     `{ "type": "res", "id": "<uuid>", "ok": true/false, "payload": {...} }`
  3. `event` (Server Push):
     `{ "type": "event", "event": "<event_name>", "payload": {...} }`

## CLI 指令 (系统入口点)

`openclaw` 命令提供了全面的控制接口：
- `openclaw onboard [--install-daemon]`: 引导式配置系统和启动 Daemon 服务。
- `openclaw doctor`: 系统健康度检查和环境诊断。
- `openclaw <扩展指令>`: 执行由插件提供的扩展 CLI 逻辑。

## 技能系统 (Agent Tools API)
技能无需编写代码，通过在 `SKILL.md` 中暴露 shell 命令或 HTTP 调用即可被 Agent 加载为 Tool。其抽象程度高于 Plugin SDK，专门服务于 LLM 的意图执行。
