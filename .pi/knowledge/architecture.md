# Buzz 系统架构

## 总体结构

```text
Desktop / Mobile / Web / CLI / Agent
                │
       Nostr WebSocket + narrow HTTP
                │
          buzz-relay (Axum)
     ┌──────────┼───────────┐
     │          │           │
 PostgreSQL   Redis      S3/MinIO
 event/data   realtime    media/git objects
 + FTS        fan-out
```

依据：`README.md`、`ARCHITECTURE.md`、`crates/buzz-relay/src/router.rs`。

## 关键架构原则

### 1. Host 先绑定 Community

WebSocket upgrade 之前，`buzz-relay` 从 `Host` 解析 `TenantContext`；未知 host 返回 generic rejection。后续 DB、Redis、workflow、search 和 fan-out 均需携带 community scope。

代码证据：`crates/buzz-relay/src/router.rs` 中 `nip11_or_ws_handler()`；tenant 类型位于 `crates/buzz-core/src/tenant.rs`。

### 2. Relay 负责跨子系统协调

`buzz-core` 提供无 I/O 的类型、kind、filter 和 verification。`buzz-db`、`buzz-auth`、`buzz-pubsub`、`buzz-search`、`buzz-audit`、`buzz-workflow` 各自负责一个基础能力；`buzz-relay` 组合它们并维持处理顺序和安全边界。

证据：`ARCHITECTURE.md`、`crates/buzz-relay/src/lib.rs`、根 `Cargo.toml`。

### 3. Event-first，而非 endpoint-first

常规产品行为优先表示为 signed Nostr event。HTTP surface 保持窄：`/events`、`/query`、`/count`、workflow webhook、media、git、invite/operator/admin 与 health 等确需 HTTP 的路径。

当前路由证据：`crates/buzz-relay/src/router.rs`。设计规则：`AGENTS.md`、`CONTRIBUTING.md`。

### 4. Access control 不是客户端过滤

Channel membership 与 community scope 在 relay 端执行。订阅成功不代表永久拥有投递权限；live fan-out 的发送 chokepoint 会重新检查 tenant、private-channel membership、author-only 和 owner-only 规则，以防 stale subscription 泄漏。

代码证据：`crates/buzz-relay/src/handlers/event.rs` 中 `filter_fanout_by_access()`。

### 5. Persistent 与 Ephemeral 路径分离

- Persistent event：验证并写入 PostgreSQL，再异步 Redis publish、本地 fan-out、workflow trigger；audit enqueue 保留 bounded backpressure。
- Ephemeral event：验证后走 Redis 与 live fan-out，不进入 PostgreSQL 历史；presence 还会更新 Redis 状态。

代码证据：`crates/buzz-relay/src/handlers/event.rs` 中 `handle_event()`、`handle_ephemeral_event()`、`dispatch_persistent_event()`。

## 客户端架构

### Desktop

React app 通过 Tauri native backend 访问 relay 和本地系统能力。`desktop/src/app/App.tsx` 建立 machine onboarding、community init、community-scoped `QueryClient` 与 key-based remount boundary。Community 切换不 reload 页面，而是重建 community-scoped React subtree，并要求 module-level cache 在 `resetCommunityState()` 中显式清理。

证据：`AGENTS.md` Desktop App 章节、`desktop/src/app/App.tsx`、`desktop/src/features/communities/useCommunityInit.ts`。

### Mobile

Flutter app 使用 Riverpod + Hooks。`mobile/lib/main.dart` 预加载 preferences，再用 `ProviderScope` 注入状态。功能按 `mobile/lib/features/` 划分，并通过 `shared/` 使用 relay、theme、storage 等共同能力。

证据：`AGENTS.md` Mobile App 章节、`mobile/pubspec.yaml`、`mobile/lib/main.dart`。

### Web

Web 是独立 React/Vite bundle，当前重点是 repo browser 与 invite landing；relay 可在配置开启时作为 SPA fallback 托管它。

证据：`web/src/main.tsx`、`web/src/features/repos/`、`web/src/features/invite/`、`buzz-relay/src/router.rs`。

## Agent 架构

```text
Buzz Relay
   │ WebSocket / NIP-42
   ▼
buzz-acp harness
   │ ACP JSON-RPC over stdio
   ▼
Goose / Codex adapter / Claude adapter / buzz-agent
   │ MCP JSON-RPC over stdio
   ▼
buzz-dev-mcp 或其他 MCP server
```

`buzz-acp` 负责 relay reconnect、channel discovery、author gate、per-channel queue、Agent pool 和 session/prompt；Agent 使用 `buzz-cli` 与 relay 交互。`buzz-agent` 与 `buzz-dev-mcp` 通过协议组合，不相互 import。

证据：`VISION_AGENT.md`、`crates/buzz-acp/README.md`、`crates/buzz-acp/src/lib.rs`。

## 数据与基础设施

- PostgreSQL：events、community、channel/member、workflow、moderation、git metadata、audit 等；search 使用 generated `tsvector` + GIN。`buzz-db/src/lib.rs` 现在是兼容 facade，内部按 `runtime/`（pool、migration、replica routing）与 `store/`（domain SQL）分层。
- Redis：跨 relay node event fan-out、presence、NIP-98 replay seen-set、rate limit、cache invalidation 和 connection control。事件 topic 使用 `buzz:{community}:channel:{id}` / `buzz:{community}:global`，subscriber 按本地需求动态执行精确 `SUBSCRIBE`；typing 作为 ephemeral event 走相同 fan-out，而不是当前独立 ZSET store。
- S3/MinIO：media 和相关 object storage。
- Migrations：`migrations/`，relay 可按配置自动执行；开发流程通常通过 `just setup`/`just migrate`。

证据：`buzz-db/src/{runtime,store}/`、`buzz-pubsub/src/{lib,topic,subscriber}.rs`、`buzz-search`、`buzz-media`、`TESTING.md`。

## 文档与当前代码差异

当前复核基线为 `00e61eaf`。根 `ARCHITECTURE.md` 的部分摘要仍提及旧 `search_index_tx` worker、通配 `PSUBSCRIBE` 和旧 Pub/Sub职责；当前代码已使用 PostgreSQL generated FTS、community-scoped动态精确 `SUBSCRIBE`，并在 `buzz-pubsub` 提供 Redis rate limiter。因此后续判断以当前代码为准，并把 `ARCHITECTURE.md` 作为高价值但可能滞后的说明文档。
