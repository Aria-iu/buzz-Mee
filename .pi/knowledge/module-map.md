# Buzz 模块地图

根 `Cargo.toml` 当前包含 29 个 Rust workspace member（含 example），并明确排除 `desktop/src-tauri`。下表以各 crate `Cargo.toml` description、`src/lib.rs` 和项目文档为依据。

## Core 与 Relay

| 模块 | 职责 |
|---|---|
| `buzz-core` | 无 I/O 核心：kind registry、event wrapper、filter、signature verification、tenant、thread、pairing、engram 等共享类型 |
| `buzz-relay` | Axum relay server；WebSocket/NIP-01、HTTP bridge、tenant binding、ingest、fan-out、media、git、workflow、huddle、operator/admin 路由 |
| `buzz-db` | PostgreSQL data access；event、community、channel/member、DM、workflow、moderation、git、push、deletion、replica routing |
| `buzz-auth` | NIP-42、NIP-98、scope 与 authorization |
| `buzz-pubsub` | Redis pub/sub、presence、typing 和跨节点 fan-out |
| `buzz-search` | community-scoped PostgreSQL full-text search |
| `buzz-audit` | per-community hash-chain audit log |
| `buzz-deletion` | 整个 community 的 durable deletion engine |
| `buzz-datastore-tracing` | datastore tracing policy macros，避免不受控敏感观测字段 |
| `buzz-conformance` | multi-tenant runtime trace schema 与独立 replay checker |

## Agent 与自动化

| 模块 | 职责 |
|---|---|
| `buzz-acp` | Relay event → ACP Agent 的 harness；queue、pool、reconnect、author gate、observer telemetry |
| `buzz-agent` | 项目内置最小 ACP Agent；LLM loop 与 MCP client |
| `buzz-dev-mcp` | Developer MCP server；shell、file edit 等工具 |
| `sprig` | 将 ACP harness、Agent 和 developer MCP 打包为 all-in-one binary |
| `buzz-cli` | Agent-first CLI；messages、channels、agents、projects、repos、workflow、media、memory 等操作 |
| `buzz-sdk` | typed Nostr event builders 与共享 client-side protocol helpers |
| `buzz-persona` | `.persona.md` persona pack parser/loader |
| `buzz-workflow` | YAML workflow schema、condition、sequential executor、scheduler、action sink interface |
| `buzz-backend-kubernetes` | remote Agent 的 Kubernetes provider binary |

## Media、Voice、Push 与 Mesh

| 模块 | 职责 |
|---|---|
| `buzz-media` | Blossom/S3 media storage、validation、thumbnail |
| `buzz-voice` | 可复用本地 voice primitives；relay 的 live huddle 转发仍位于 `buzz-relay/src/audio/` |
| `buzz-push-gateway` | Mobile 使用的 blind、capability-gated NIP-PL push gateway |
| `buzz-relay-mesh` | inter-relay QUIC mesh transport、membership 与 fenced wire contract |

## Git、Pairing 与工具

| 模块 | 职责 |
|---|---|
| `git-sign-nostr` | 使用 Nostr secp256k1 key 对 Git commit/tag 签名 |
| `git-credential-nostr` | 为 Buzz Git smart HTTP 生成 NIP-98 auth header |
| `buzz-pair-relay` | NIP-AB 设备配对的 ephemeral sidecar relay |
| `buzz-pairing-cli` | NIP-AB pairing interop CLI |
| `buzz-ws-client` | 共享 NIP-42 WebSocket client |
| `buzz-admin` | operator CLI：membership、keys、reconcile 等管理操作 |
| `buzz-test-client` | relay integration/E2E client 与 ignored live tests |

## 非 Rust workspace surface

| 目录 | 职责与入口 |
|---|---|
| `desktop/` | Tauri 2 + React desktop；React 入口 `desktop/src/main.tsx`，app boundary `desktop/src/app/App.tsx`，native crate `desktop/src-tauri/` |
| `web/` | React repo browser/invite；入口 `web/src/main.tsx` |
| `mobile/` | Flutter mobile；入口 `mobile/lib/main.dart`，feature 在 `mobile/lib/features/` |
| `admin-web/` | 部署级管理 web surface |
| `migrations/` | PostgreSQL schema migration |
| `deploy/` | Compose、Helm/Kubernetes 等部署描述 |
| `scripts/` | CI、release、screenshots、E2E、worktree 和 maintenance 工具 |
| `benchmarks/` | Agent orchestration 等 benchmark/testbed |
| `docs/` | NIP、formal spec、remote-agent 和产品/工程文档 |

## 重要依赖方向

1. `buzz-core` 应保持 zero-I/O，并被其他服务 crate 依赖。
2. Service crate 不应自行从客户端 event tag 推导 tenant；由 relay 传入 server-resolved scope。
3. 跨 service 的业务编排集中在 `buzz-relay`，而不是让 DB/search/pubsub 相互调用。
4. Agent-facing 新操作优先进入 `buzz-cli`；`buzz-dev-mcp` 保持开发工具职责。
5. Desktop 和 Mobile 通过各自 shared layer 与 relay 通信，不应把 feature module 之间形成任意耦合。

证据：`ARCHITECTURE.md`、`AGENTS.md`、各 crate `Cargo.toml` 与 `src/lib.rs`。
