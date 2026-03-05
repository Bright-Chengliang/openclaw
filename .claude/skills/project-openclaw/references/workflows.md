# OpenClaw 业务工作流

本节描述消息在 OpenClaw 系统中的完整生命周期。

## 消息处理生命周期 (Inbound to Outbound)

1. **接收 (Inbound)**
   - 消息通过特定平台的 Webhook 或 WebSocket 到达对应的 Extension (如 Telegram 机器人收到消息)。
   - Extension 将平台特定的消息格式转换为 OpenClaw 内部标准格式，并投递给 Gateway。

2. **路由与权限校验 (Routing & Auth)**
   - Gateway 解析消息来源，通过 Allowlist 检查和设备配对验证 (Device Pairing)。
   - 解析目标 Agent 路由：决定是分配给默认 `main` 代理，还是特定的功能代理。
   - 计算 Session Key：基于通道类型、群组/私聊属性、用户 ID 映射到特定的 Session 作用域。

3. **会话管理 (Session Binding)**
   - Gateway 在内存和本地文件系统中查找或创建关联的 Session。
   - 如果对应 Agent 当前正忙 (Busy)，消息可能进入 Queue 队列。支持不同的排队模式：`steer` (插入中断), `followup` (跟随), `collect` (聚合)。

4. **Agent 推理 (Agent Processing)**
   - 将组装好的上下文（历史记录 + 新消息）传递给底层 Pi Agent Runtime (以 RPC 模式)。
   - 触发 LLM 模型推理。如果启用了多个模型，按照 Primary -> Fallback 链路执行。

5. **工具与技能调用 (Tool Execution - 可选)**
   - 如果 LLM 决定调用工具，Agent 运行时暂停文本生成。
   - 调用注册在 Gateway 上的工具 (如 `exec` 执行命令, 访问浏览器, 执行 Skills/Bash 脚本, 或下发设备节点控制指令 `camera.take_photo`)。
   - 工具结果返回给 Agent 继续推理。

6. **响应组装与流式下发 (Outbound Streaming)**
   - Agent 开始流式输出回复文本。
   - Gateway 负责接收数据块，根据不同通道的限制（如长度、发送频率）执行 Chunking 和 Coalescing（聚合节流）。
   - 将格式化后的内容派发回源 Extension。

7. **投递与持久化 (Delivery & Persistence)**
   - Extension 将消息发回对应的 Chat App。
   - (同时) Gateway 将整个对话回合（Prompt, Completion, Tool calls）持久化写入 `<SessionId>.jsonl` 文件。
   - 更新 `sessions.json` 的元数据（最后活跃时间等）。

## 扩展：Hook 与事件流
- 在流程的不同阶段，Gateway 会发出系统事件 (如 `agent`, `chat`, `presence`)，WebSocket Client 或 UI 控制面板可监听这些事件以实现状态同步与监控。
- 内置 Hooks (如 `session-memory`) 会在特定的生命周期事件上被触发。
