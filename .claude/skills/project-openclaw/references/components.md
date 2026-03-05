# 核心组件与扩展机制

OpenClaw 拥有高度模块化的组件设计，以下是支撑系统运转的核心模块。

## 1. 扩展机制 (Extensions / Channels)

Extensions 位于 `extensions/` 目录下，作为插件在系统启动时被加载。
每个扩展是一个 npm scope 包 (如 `@openclaw/whatsapp`)，在 `package.json` 中通过 `openclaw.extensions` 字段声明入口。

**主要分类：**
- **Messaging Channels (消息通道)**:
  - `whatsapp` (基于 `@whiskeysockets/baileys`)
  - `telegram` (基于 `grammy`)
  - `slack` (基于 `@slack/bolt`)
  - `discord` (基于 `@buape/carbon` & `discord.js`)
  - 其他如 `signal`, `imessage`, `msteams`, `googlechat`, `matrix`, `line`, `wechat/zalo` 等。
- **Auth & Infrastructure (基础设施)**:
  - `copilot-proxy`, `google-antigravity-auth`, `minimax-portal-auth`
  - `device-pair`, `diagnostics-otel` (诊断追踪)
- **Capabilities (能力扩展)**:
  - `voice-call`, `phone-control`
  - `memory-core`, `memory-lancedb` (长期记忆与向量检索)
  - `llm-task`, `open-prose`

## 2. 技能系统 (Skills)

Skills 位于 `skills/` 目录，定义了 Agent 可使用的功能。支持通过 ClawHub 自动发现。
- **加载机制**: 按优先级加载，Workspace 本地 > Managed (`~/.openclaw/skills`) > Bundled (系统自带)。
- **Manifest (`SKILL.md`)**:
  - YAML Frontmatter: 定义 `name`, `description`, `emoji`, `requirements`。
  - Markdown 主体: 提供使用指南、执行的 Bash 命令或提示词模板。
- **Bundled Hooks**: 位于 `dist/bundled/`，如 `session-memory` (在新建/重置会话时自动触发保存上下文)，`boot-md` 等。

## 3. Agent 运行时组件

基于 `pi-mono` 生态封装的 Agent Runtime (`@mariozechner/pi-*` 系列包)。
- **Workspace 管理**: 核心配置依赖 `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`。
- **Model Selection**: 支持主模型与回退 (Fallback)，以及根据 Agent 配置进行重载。
- **多模型/多 Agent 路由**:
  - 每个 Agent 拥有独立的 Session 隔离上下文。

## 4. Session 管理器

Gateway 作为会话数据的 Single Source of Truth。
- **数据结构**: 存储在 `~/.openclaw/agents/<agentId>/sessions/`。
- **持久化**:
  - 索引文件: `sessions.json`
  - 对话转录 (Transcript): `<SessionId>.jsonl` (Append-only)
- **会话作用域**:
  - `main`: 默认共享空间
  - `per-peer` / `per-channel-peer`: 基于群组/用户的隔离空间
- **特性**: 支持 Cross-channel 身份链接与会话自动清理 (Pruning)。
