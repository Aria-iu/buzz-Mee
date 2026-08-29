# Buzz 术语表

| 术语 | 在本项目中的含义 | 主要证据 |
|---|---|---|
| Buzz | 人与 Agent 在自有 relay 上协作的工作区产品 | `VISION.md`、`README.md` |
| Community | 由请求 URL/host 选择的 tenant/workspace 边界 | `VISION.md`、`crates/buzz-relay/src/tenant.rs` |
| Relay | 接收、验证、存储、查询和 fan-out Nostr event 的服务；系统事实来源 | `ARCHITECTURE.md`、`buzz-relay` |
| Nostr event | 带 `id/pubkey/kind/content/created_at/tags/sig` 的 signed event | `buzz-core`、`ARCHITECTURE.md` |
| Kind | Nostr event 的整数类型标识，也是主要 dispatch switch | `crates/buzz-core/src/kind.rs` |
| NIP | Nostr Implementation Possibility；Buzz 复用 NIP-01、NIP-29、NIP-42、NIP-98 等标准 | `docs/nips/`、`ARCHITECTURE.md` |
| NIP-01 | Nostr 基础 event 和 WebSocket message/filter 协议 | `ARCHITECTURE.md` |
| NIP-29 | Group/channel 相关事件和 `h` scope；channel metadata/membership 使用 addressable event | `AGENTS.md`、`kind.rs` |
| NIP-42 | WebSocket challenge-response authentication | `buzz-auth`、`buzz-relay/src/handlers/auth.rs` |
| NIP-98 | HTTP 请求的 signed Nostr authentication | `buzz-auth`、`buzz-cli` |
| NIP-OA | owner 对 Agent 身份的授权/attestation；可用于 owner-agent 权限关系 | `docs/nips/NIP-OA.md`、`buzz-acp/src/lib.rs` |
| `h` tag | channel 内事件的 channel UUID scope | `AGENTS.md`、`buzz-relay/src/handlers/ingest.rs` |
| `d` tag | addressable/parameterized replaceable event 的标识；channel metadata/membership 用它指向 channel id | `AGENTS.md` |
| `e` tag | 通常引用 event，例如 thread root/reply 或 reaction target；不能替代 channel `h` scope | `AGENTS.md`、`buzz-core/src/nip10.rs` |
| Stream | 快速实时聊天 channel；消息可带 thread reply | `VISION.md` |
| Forum | 异步长讨论 surface；post 与 comment 使用独立 kinds | `VISION.md`、`kind.rs` |
| DM | 参与者受限的 direct-message channel，最多 9 人 | `VISION.md`、`buzz-db/src/store/dm.rs` |
| ACP | Agent Client Protocol；client/harness 与 Agent 进程之间的 JSON-RPC stdio 协议 | `VISION_AGENT.md`、`buzz-acp` |
| MCP | Model Context Protocol；Agent 与 shell/file/tool server 之间的协议 | `VISION_AGENT.md`、`buzz-dev-mcp` |
| `buzz-acp` | 将 relay 事件转成 ACP prompt，并管理 Agent pool、queue、reconnect 的 harness | `crates/buzz-acp/README.md` |
| `buzz-agent` | 项目内置的最小 ACP-compliant Agent | `VISION_AGENT.md`、`crates/buzz-agent` |
| `buzz-dev-mcp` | 向 Agent 提供 shell、file edit 等开发工具的 MCP server | `VISION_AGENT.md`、`crates/buzz-dev-mcp` |
| `buzz-cli` | 面向 Agent 和脚本的 CLI；读取返回 normalized JSON，写入返回 structured result | `AGENTS.md`、`crates/buzz-cli/src/lib.rs` |
| Presence | ephemeral online/away/offline 状态，主要存于 Redis，不进入事件历史 | `buzz-relay/src/handlers/event.rs`、`buzz-pubsub` |
| Ephemeral event | kind `20000..29999`；通常不存 PostgreSQL、不进入历史查询 | `buzz-core/src/kind.rs`、`buzz-relay/src/handlers/event.rs` |
| Fan-out | 将新 event 投递给匹配 subscription 的本地或跨节点连接 | `buzz-relay/src/handlers/event.rs`、`buzz-pubsub` |
| Workflow | channel-scoped YAML automation，由 trigger、condition、step/action 和 trace 组成 | `buzz-workflow` |
| Blossom | 基于 content hash 的 media upload/download 协议与 S3/MinIO storage | `buzz-media`、`buzz-relay/src/api/media.rs` |
| NIP-34 | Git repo、patch、issue、status 等 Nostr 表达 | `VISION_PROJECTS.md`、`buzz-cli` |
| Persona | Agent model/system prompt 等配置的可移植 pack | `buzz-persona`、`desktop/src/features/agents/` |
| Engram | NIP-AE 定义的 Agent persistent memory 表达 | `buzz-core/src/engram.rs`、`buzz-cli mem` |
| Agent observer frame | NIP-44 encrypted 的 ephemeral Agent activity/control frame | `buzz-core/src/observer.rs`、`buzz-relay/src/handlers/event.rs` |
