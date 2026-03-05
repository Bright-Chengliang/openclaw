# 开发与运维指南

## 目录与文件路径

- **配置目录**: `~/.openclaw/`
  - `openclaw.json` (JSON5 格式的主配置文件)
  - `agents/` (Agent 配置与会话存储目录)
  - `skills/` (本地管理的自定义技能目录)
- **运行时端口**: 默认 HTTP / WebSocket 端口 `18789`

## 系统运维管理

### 状态查询
可以使用特殊的 Chat 命令与机器人直接交互以获取状态：
- `/status`：查看当前网关及组件健康状态
- `/usage`：查询模型 Token 使用量
- `/restart`：重启 Agent 服务
- `/activation`：检查并设置当前活动的模块

### 会话清理与维护
- `/new` 或 `/reset`：开启新会话，清理上下文
- `/compact`：触发会话记忆的压缩（由内置钩子 `session-memory` 接管历史总结）

## 本地开发与调试

项目使用 `pnpm` 作为包管理器。

### 构建项目
```bash
# 执行完整构建 (包含 TypeScript 编译、API 生成和前端 UI Bundle)
pnpm build

# 预打包（常用于发布前验证）
pnpm prepack
```

### 运行与测试
```bash
# 以开发模式启动 Gateway
pnpm gateway:dev

# 并行运行所有测试 (Unit, E2E, Live, Docker)
pnpm test:all

# 运行基础单元测试 (Fast)
pnpm test:fast

# 代码风格与质量检查
pnpm check
pnpm lint
```

### 协议与类型同步
如果修改了 Gateway 核心协议定义：
```bash
# 重新生成 JSON Schema 与 Swift 等跨语言协议定义
pnpm protocol:gen
```

## 故障排查

1. **会话状态异常**：
   - 检查 `~/.openclaw/agents/<agentId>/sessions/sessions.json`。
   - 验证日志文件以确认消息是否成功写入 `.jsonl`。

2. **通信通道断连**：
   - 检查对应 Extension (如 Baileys / grammy) 的认证状态。
   - 确保 `openclaw.json` 中该通道被正确启用并具有合法的 Token/Credentials。

3. **系统启动失败**：
   - 运行 `openclaw doctor` 获取诊断信息，它会自动检查依赖、网络、端口冲突等基础环境。
