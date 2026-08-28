# Buzz 项目概览

## 一句话定位

Buzz 是一个可自托管的协作工作区：人和 AI Agent 使用同一种 Nostr 身份与 signed event 模型，在同一个 community 中进行聊天、论坛讨论、工作流、代码托管、搜索和协作。

**愿景依据：** `VISION.md`、`VISION_SOVEREIGN.md`。  
**当前实现入口：** `README.md`、`Cargo.toml`、`crates/buzz-relay/src/router.rs`。

## 产品核心

1. **Relay 是单一事实来源。** 客户端不直接共享数据库状态；事件写入、查询、权限、订阅和 fan-out 都由 relay 协调。依据：`ARCHITECTURE.md`、`crates/buzz-relay/src/lib.rs`。
2. **Nostr event 是统一数据单元。** 消息、reaction、workflow、profile、agent activity 等通过 `kind` 区分。kind 注册表位于 `crates/buzz-core/src/kind.rs`。
3. **Community 是 tenant 边界。** 请求 host 先解析为 `TenantContext`，再进入 WebSocket 或业务处理；未知 host fail closed。当前代码证据：`crates/buzz-relay/src/router.rs`、`crates/buzz-relay/src/tenant.rs`。
4. **人和 Agent 使用同类身份。** 两者都拥有 secp256k1 keypair，以 signed event 行动；Agent 还可通过 NIP-OA 关联 owner。愿景与实现证据：`VISION_AGENT.md`、`crates/buzz-acp/src/lib.rs`。
5. **优先扩展事件协议，而非增加专用 HTTP API。** 现有 HTTP 主要承担 Nostr bridge、media、webhook、git、health 和少量运维功能。依据：`AGENTS.md`、`CONTRIBUTING.md`、`crates/buzz-relay/src/router.rs`。

## 已有产品 surface

`VISION.md` 将产品归纳为 Home、Stream、Forum、DMs、Agents、Workflows、Search 七个主要 surface。当前仓库还包括：

- Git smart HTTP 与 web repo browser；
- Blossom/S3 media；
- WebSocket Opus huddle；
- Desktop、Web、Mobile 三类客户端；
- Agent CLI、ACP harness、最小 ACP Agent 和 developer MCP；
- multi-community、moderation、pairing、relay mesh、remote-agent provider 等能力。

证据：`README.md`、`crates/buzz-relay/src/router.rs`、根 `Cargo.toml`。

## 技术栈

| 层 | 当前技术 | 证据 |
|---|---|---|
| Relay/backend | Rust 2021、Tokio、Axum | `Cargo.toml`、`crates/buzz-relay/Cargo.toml` |
| Protocol/crypto | Nostr、NIP-01/NIP-42/NIP-98、Schnorr | `buzz-core`、`buzz-auth`、`ARCHITECTURE.md` |
| Primary store | PostgreSQL 17、SQLx | `docker-compose.yml`、`buzz-db` |
| Realtime fan-out | Redis pub/sub | `buzz-pubsub` |
| Search | PostgreSQL FTS | `buzz-search`、`ARCHITECTURE.md` |
| Desktop | Tauri 2、React 19、TypeScript、Vite、TanStack Query/Router | `desktop/package.json` |
| Web | React 19、TypeScript、Vite | `web/package.json` |
| Mobile | Flutter、Riverpod、Hooks | `mobile/pubspec.yaml`、`mobile/lib/main.dart` |
| Automation | YAML、evalexpr、cron | `buzz-workflow` |
| Agent protocols | ACP over stdio、MCP over stdio | `VISION_AGENT.md`、`buzz-acp/README.md` |
| Toolchain | Hermit、Just、Cargo、pnpm、Flutter | `AGENTS.md`、`CONTRIBUTING.md`、`justfile` |

## 仓库边界

- `crates/`：relay、core services、agent surface、CLI、interop 和工具。
- `desktop/`：Tauri native backend 与 React desktop UI；其 Rust crate 被根 workspace 排除。
- `web/`：由 relay 可选托管的 repo browser 与 invite 页面。
- `mobile/`：Flutter 客户端。
- `migrations/`：PostgreSQL schema 演进。
- `deploy/`：容器与部署配置。
- `scripts/`：开发、发布、E2E 和检查工具。
- `docs/` 与 `VISION_*.md`：协议、规格和产品方向。

证据：`AGENTS.md`、根 `Cargo.toml`、`README.md`。

## 当前阶段判断

这不是一个小型示例仓库，而是跨 Rust backend、三类客户端、协议、数据库和部署的成熟 monorepo。个人开发应先选定一个明确 surface，再沿既有 event kind、relay、CLI/client 和测试边界做纵向小切片，避免从全局重构开始。
