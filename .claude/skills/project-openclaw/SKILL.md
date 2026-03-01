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

# OpenClaw — 项目知识库

> 📍 项目路径: `/home/bright/.nvm/versions/node/v24.13.0/lib/node_modules/openclaw`
> 🔧 技术栈: TypeScript + Node.js + Express + WebSocket + Commander
> 📅 索引生成时间: 2026-02-28

## 项目概述

OpenClaw 是一个自托管的个人 AI 助手网关系统。它通过统一的 Gateway 进程桥接 22+ 消息平台（WhatsApp、Telegram、Discord、Slack、Signal、iMessage 等）到 AI 编码代理（基于 `@mariozechner/pi-coding-agent`）。用户在自己的设备上运行，通过已有的聊天应用与 AI 交互。支持语音通话、Canvas 渲染、多模型切换、定时任务、浏览器控制等高级功能。

## 技术栈

| 层面 | 技术 | 版本 |
|------|------|------|
| 语言 | TypeScript / ESM | TS 5.9+ |
| 运行时 | Node.js | ≥22.12.0 |
| 构建 | tsdown (rolldown) | 0.20+ |
| HTTP 框架 | Express | 5.2+ |
| 实时通信 | ws (WebSocket) | 8.19+ |
| CLI 框架 | Commander | 14.0+ |
| AI 核心 | @mariozechner/pi-coding-agent | 0.55.0 |
| Schema 验证 | @sinclair/typebox + AJV + Zod | - |
| 包管理 | pnpm | 10.23.0 |
| 测试 | Vitest | 4.0+ |
| Lint/Format | oxlint + oxfmt | - |

## 目录结构概览

| 目录 | 职责 |
|------|------|
| `openclaw.mjs` | CLI 入口脚本，启用编译缓存后加载 `dist/entry.js` |
| `dist/` | 构建产物，376+ 个模块（tsdown 打包） |
| `extensions/` | 39 个扩展插件（消息通道、认证、记忆、语音等） |
| `skills/` | 55+ 个内置技能（weather、github、coding-agent 等） |
| `docs/` | 产品文档（Mintlify 格式） |
| `assets/` | 静态资源（头像占位符、Chrome 扩展、DMG 背景） |

## 核心架构

OpenClaw 采用**单进程网关 + 插件扩展**架构：

```
┌─────────────────────────────────────────────────┐
│                  Gateway (单进程)                 │
│  Express + WebSocket @ 127.0.0.1:18789          │
├─────────┬──────────┬──────────┬─────────────────┤
│ Channel │  Agent   │  Plugin  │    Session      │
│ Manager │  Runner  │ Registry │    Manager      │
├─────────┴──────────┴──────────┴─────────────────┤
│              Model Selection + Auth              │
│         (Anthropic/OpenAI/Google/Ollama/...)     │
└─────────────────────────────────────────────────┘
     ↕           ↕           ↕           ↕
  WhatsApp    Telegram    Discord    Slack ...
  (Baileys)  (grammY)   (Carbon)   (Bolt)
```

- Gateway 是单一长驻进程，拥有所有消息通道连接
- WebSocket 协议承载 RPC 请求/响应 + 服务端推送事件
- 插件系统提供 10+ 注册方法和 24 个生命周期钩子
- 技能系统通过 Markdown + YAML frontmatter 注入 Agent 系统提示词

## 核心组件速查

| 组件 | 路径 | 职责 | 详情 |
|------|------|------|------|
| CLI 入口 | `→ openclaw.mjs` → `dist/entry.js` | 命令行解析与分发 | [components.md#cli](references/components.md#cli-入口) |
| Gateway 服务器 | `→ dist/server*.js` | HTTP/WS 网关服务 | [components.md#gateway](references/components.md#gateway-服务器) |
| Agent Runner | `→ dist/runner-*.js` + `dist/reply-*.js` | AI 代理执行引擎 | [components.md#agent-runner](references/components.md#agent-runner) |
| Channel Manager | `→ dist/channel-*.js` + `dist/plugins-*.js` | 消息通道管理 | [components.md#channel](references/components.md#channel-manager) |
| Plugin Registry | `→ dist/plugin-registry-*.js` | 插件注册与生命周期 | [components.md#plugin](references/components.md#plugin-registry) |
| Session Manager | `→ dist/sessions-*.js` | 会话状态与锁管理 | [components.md#session](references/components.md#session-manager) |
| Model Selection | `→ dist/model-selection-*.js` | 多模型选择与故障转移 | [components.md#model](references/components.md#model-selection) |
| Config System | `→ dist/config-*.js` | 配置加载与验证 | [components.md#config](references/components.md#config-system) |
| Skills System | `→ dist/skills-*.js` | 技能加载与注入 | [components.md#skills](references/components.md#skills-system) |
| Daemon Manager | `→ dist/daemon-*.js` | 系统服务安装与管理 | [components.md#daemon](references/components.md#daemon-manager) |

## 关键入口点

| 入口 | 路径 | 说明 |
|------|------|------|
| CLI bin | `→ openclaw.mjs` | npm bin 入口，加载 dist/entry.js |
| 库导出 | `→ dist/index.js` | 主模块导出 |
| Plugin SDK | `→ dist/plugin-sdk/index.js` | 插件开发 SDK |
| 命令注册 | `→ dist/register.*.js` | 各子命令懒加载注册 |

## 主要工作流程

1. **消息处理流程** — 从聊天应用收到消息 → 通道适配器标准化 → 会话路由 → Agent 执行 → 回复投递
2. **Gateway 启动流程** — 加载配置 → 注册插件 → 启动通道连接 → 监听 WS/HTTP
3. **Onboarding 流程** — 认证选择 → 网关配置 → 工作区设置 → 通道配置 → 技能安装 → 守护进程安装
4. **Agent 执行流程** — 会话锁定 → 模型选择 → 系统提示构建 → LLM 调用 → 工具执行 → 流式响应
5. **插件注册流程** — 发现 manifest → 加载入口 → 调用 register(api) → 注册通道/工具/钩子

详见 → [references/workflows.md](references/workflows.md)

## 开发常用命令

| 操作 | 命令 |
|------|------|
| 安装 | `npm install -g openclaw@latest` |
| 初始化 | `openclaw onboard --install-daemon` |
| 启动网关 | `openclaw gateway --port 18789 --verbose` |
| 发送消息 | `openclaw message send --to +1234567890 --message "Hello"` |
| Agent 对话 | `openclaw agent --message "Ship checklist" --thinking high` |
| TUI 界面 | `openclaw tui` |
| 健康检查 | `openclaw status --deep` |
| 开发模式 | `pnpm dev` / `pnpm gateway:dev` |
| 构建 | `pnpm build` |
| 测试 | `pnpm test` |
| Lint | `pnpm lint` |

## 详细参考

- 架构详解 → [references/architecture.md](references/architecture.md)
- 组件清单 → [references/components.md](references/components.md)
- 业务流程 → [references/workflows.md](references/workflows.md)
- 数据模型 → [references/data-models.md](references/data-models.md)
- API接口 → [references/api-surface.md](references/api-surface.md)
- 开发指南 → [references/dev-guide.md](references/dev-guide.md)
