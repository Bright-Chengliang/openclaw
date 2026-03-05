---
name: project-openclaw
description: |
  OpenClaw 项目知识库。包含项目架构、核心组件、业务流程和接口清单。
  多通道 AI 网关，支持 22+ 消息平台集成，可扩展插件系统。

  **自动触发场景**：
  - 在 openclaw 项目目录下工作时
  - 涉及 openclaw 相关的开发、调试、重构任务
  - 需要理解 openclaw 的架构或实现细节时
  - 处理消息通道、网关、Agent、插件、技能系统相关问题

  **关键词**: openclaw, gateway, agent, channel, plugin, extension, skill, session, daemon, model, whatsapp, telegram, discord, slack, signal, pi-agent
---

# OpenClaw 项目知识库

> 项目路径: `/home/bright/.nvm/versions/node/v24.13.0/lib/node_modules/openclaw`
> 技术栈: TypeScript + Node.js (ESM) + Express v5 + WebSocket + Commander
> 版本: v2026.2.26 | 构建: tsdown (Rolldown) | 包管理: pnpm 10.23

## 项目概述

OpenClaw 是一个自托管的个人 AI 助手网关。通过统一的 Gateway 进程桥接 22+ 消息平台（WhatsApp、Telegram、Discord、Slack、Signal、iMessage 等）到 AI Agent 运行时（基于 pi-mono 生态）。核心定位："The product is the assistant, the Gateway is just the control plane."

## 目录结构

| 目录 | 职责 |
|------|------|
| `openclaw.mjs` | CLI bin 入口，启用编译缓存后加载 `dist/entry.js` |
| `dist/` | 构建产物（tsdown 打包的 ESM 模块） |
| `dist/plugin-sdk/` | 插件开发 SDK（含 `.d.ts` 类型声明） |
| `dist/bundled/` | 内置 Hooks（boot-md, session-memory 等） |
| `dist/control-ui/` | Web 控制面板前端资源 |
| `dist/canvas-host/` | Canvas/A2UI 宿主资源 |
| `extensions/` | 40 个扩展插件（消息通道、认证、记忆、语音等） |
| `skills/` | 52+ 个内置技能（weather, github, coding-agent 等） |
| `docs/` | 产品文档（Mintlify 格式） |

## 核心架构

```
┌─────────────────────────────────────────────────┐
│              Gateway (单进程 Daemon)              │
│        Express + WebSocket @ :18789              │
├─────────┬──────────┬──────────┬─────────────────┤
│ Channel │  Agent   │  Plugin  │    Session      │
│ Manager │  Runner  │ Registry │    Manager      │
├─────────┴──────────┴──────────┴─────────────────┤
│            Model Selection + Auth                │
│       (Anthropic/OpenAI/Google/Ollama/...)       │
└─────────────────────────────────────────────────┘
     ↕           ↕           ↕           ↕
  WhatsApp    Telegram    Discord    Slack ...
  (Baileys)  (grammY)   (Carbon)   (Bolt)
```

## 扩展分类 (extensions/)

- **消息通道**: whatsapp, telegram, discord, slack, signal, bluebubbles, imessage, msteams, googlechat, matrix, line, irc, nostr, mattermost, nextcloud-talk, synology-chat, feishu, zalo, tlon, twitch
- **认证与基础设施**: copilot-proxy, google-antigravity-auth, google-gemini-cli-auth, minimax-portal-auth, qwen-portal-auth, device-pair, diagnostics-otel
- **能力扩展**: voice-call, talk-voice, phone-control, memory-core, memory-lancedb, llm-task, open-prose, lobster, thread-ownership

每个扩展通过 `package.json` 的 `openclaw.extensions` 字段声明入口，Gateway 启动时动态加载。

## 消息处理生命周期

1. **Inbound** — 消息到达 Extension → 标准化为内部格式
2. **Routing** — Allowlist/Pairing 校验 → Agent 路由 → Session Key 计算
3. **Session** — 查找/创建 Session → 排队（steer/followup/collect）
4. **Agent** — Pi Agent Runtime (RPC) → LLM 推理 → 工具调用
5. **Outbound** — 流式响应 → Chunking/Coalescing → 投递回源通道
6. **Persist** — 对话写入 `<SessionId>.jsonl` → 更新 `sessions.json`

## 关键依赖

| 依赖 | 用途 |
|------|------|
| `@whiskeysockets/baileys` | WhatsApp Web 协议 |
| `grammy` | Telegram Bot API |
| `@slack/bolt` | Slack 集成 |
| `@buape/carbon` | Discord 集成 |
| `@mariozechner/pi-*` | Agent 运行时 (pi-mono) |
| `@sinclair/typebox` | 协议 Schema 定义 |
| `playwright-core` | 浏览器控制 (CDP) |
| `sqlite-vec` | 向量存储 |

## 常用命令

| 操作 | 命令 |
|------|------|
| 开发模式 | `pnpm dev` / `pnpm gateway:dev` |
| 构建 | `pnpm build` |
| 测试 | `pnpm test` / `pnpm test:fast` |
| Lint | `pnpm check` |
| TUI | `pnpm tui` |
| 诊断 | `openclaw doctor` |

## 详细参考

- [架构设计](references/architecture.md) — Gateway 架构、Daemon 模型、WebSocket 协议
- [核心组件](references/components.md) — Extensions、Skills、Agent Runtime、Session 管理
- [业务流程](references/workflows.md) — 消息生命周期、Hook 与事件流
- [数据模型](references/data-models.md) — 会话存储、配置结构、协议帧格式
- [API 接口](references/api-surface.md) — Plugin SDK、WebSocket API、CLI 命令
- [开发指南](references/dev-guide.md) — 构建测试、运维管理、故障排查
