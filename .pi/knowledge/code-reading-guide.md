# Buzz 代码阅读路线

## 目标

用最少的阅读量建立正确心智模型，再根据具体功能下钻。不要从 `buzz-relay/src/main.rs` 或大型 UI 文件逐行开始。

## 第一层：产品与强制约束

按顺序阅读：

1. `VISION.md`：产品整体方向；
2. 与目标 surface 相关的 `VISION_*.md`；
3. `AGENTS.md`：贡献规则、架构惯例、测试要求和常见陷阱；
4. `TESTING.md`：验证层次与真实 relay workflow；
5. `README.md`：用户视角和开发入口。

完成后应能回答：Buzz 解决什么问题、community 是什么、为什么优先 Nostr event、哪些测试会改变本地环境。

## 第二层：协议与核心类型

推荐顺序：

1. `crates/buzz-core/src/lib.rs`：core 能力目录；
2. `crates/buzz-core/src/kind.rs`：所有 event kind 的权威注册表；
3. `crates/buzz-core/src/event.rs`：stored event wrapper；
4. `crates/buzz-core/src/filter.rs`：NIP-01 filter matching；
5. `crates/buzz-core/src/verification.rs`：signature/id verification；
6. `crates/buzz-core/src/tenant.rs`：community identity；
7. `crates/buzz-core/src/nip10.rs`：thread marker。

完成后应能回答：event 如何分类、channel scope 在哪里、哪些数据是 client-signed、哪些 tenant 信息只能由 server 提供。

## 第三层：Relay 主路径

推荐顺序：

1. `crates/buzz-relay/src/lib.rs`：模块目录；
2. `crates/buzz-relay/src/router.rs`：HTTP/WS surface 与 host binding；
3. `crates/buzz-relay/src/connection.rs`：connection lifecycle；
4. `crates/buzz-relay/src/handlers/auth.rs`：NIP-42 与 membership gates；
5. `crates/buzz-relay/src/handlers/event.rs`：WS EVENT、ephemeral 和 fan-out；
6. `crates/buzz-relay/src/handlers/ingest.rs`：persistent ingest；
7. `crates/buzz-relay/src/handlers/req.rs`：subscription、history 和 search；
8. `crates/buzz-relay/src/subscription.rs`：fan-out indexes；
9. `crates/buzz-relay/src/state.rs`：AppState 与 caches。

阅读 `ingest.rs` 时按 gate 分段理解，不要把它当普通线性 CRUD：write fence → crypto → auth/scope → kind routing → channel/access → kind validation → transaction → side effect → dispatch。

## 第四层：Persistence 与 realtime

1. `crates/buzz-db/src/lib.rs`：store inventory 和 DB runtime boundary；
2. `buzz-db/src/event.rs`：event query/insert；
3. `buzz-db/src/channel.rs`：channel/member；
4. `buzz-db/src/thread.rs` 与 `reaction.rs`：原子派生状态；
5. `buzz-db/src/community.rs`：tenant lifecycle；
6. `buzz-db/src/workflow.rs`：workflow persistence；
7. `crates/buzz-pubsub/src/lib.rs`：Redis routing；
8. `crates/buzz-search/src/lib.rs`：PostgreSQL FTS；
9. `crates/buzz-audit/src/lib.rs`：hash-chain audit。

不要先深入 `buzz-db/src/lib.rs` 的 read-replica fence 细节；只有修改 routed reads 时才需要完整掌握。

## 第五层：Agent 系统

1. `VISION_AGENT.md`；
2. `crates/buzz-acp/README.md`；
3. `crates/buzz-acp/src/config.rs`；
4. `crates/buzz-acp/src/relay.rs`；
5. `crates/buzz-acp/src/queue.rs`；
6. `crates/buzz-acp/src/pool.rs`；
7. `crates/buzz-acp/src/prompt_framing.rs`；
8. `crates/buzz-agent/src/lib.rs`；
9. `crates/buzz-dev-mcp/src/lib.rs`；
10. `crates/buzz-cli/src/lib.rs` 与目标 command implementation。

完成后应能区分：relay、harness、Agent、MCP server 和 CLI 各自负责什么。

## 第六层：客户端

### Desktop

1. `desktop/src/main.tsx`：provider hierarchy；
2. `desktop/src/app/App.tsx`：machine/community boundary；
3. `desktop/src/app/router.tsx`：页面导航；
4. `desktop/src/shared/api/`：Tauri/relay API boundary；
5. `desktop/src/features/communities/useCommunityInit.ts`：community reset；
6. 再进入 `desktop/src/features/<target>/`。

### Mobile

1. `mobile/lib/main.dart`；
2. `mobile/lib/app.dart`；
3. `mobile/lib/shared/relay/`；
4. `mobile/lib/shared/theme/`；
5. `mobile/lib/features/<target>/`。

### Web

1. `web/src/main.tsx`；
2. `web/src/app/App.tsx`；
3. `web/src/features/repos/` 或 `web/src/features/invite/`。

## 按任务快速定位

| 想修改的内容 | 优先入口 |
|---|---|
| Event 定义 | `buzz-core/src/kind.rs` |
| Event 写入规则 | `buzz-relay/src/handlers/ingest.rs` |
| WebSocket live delivery | `event.rs`、`subscription.rs` |
| 历史查询/search | `req.rs`、`buzz-db/src/event.rs`、`buzz-search` |
| Channel/member | `buzz-db/src/channel.rs`、relay side effects/commands |
| Agent-facing operation | `buzz-cli` |
| Agent queue/session | `buzz-acp/src/queue.rs`、`pool.rs` |
| Workflow | `buzz-workflow` + `buzz-relay/src/workflow_sink.rs` |
| Desktop UI | `desktop/src/features/<feature>/` |
| Mobile UI | `mobile/lib/features/<feature>/` |
| Git hosting | `buzz-relay/src/api/git/`、`buzz-db/src/git_repo.rs` |
| Media | `buzz-media`、`buzz-relay/src/api/media.rs` |

## 阅读纪律

- 愿景描述目标，代码描述当前事实；两者冲突时同时记录。
- 先读 public module docs、types 和 tests，再读长函数实现。
- 涉及权限时同时检查 write、historical read、search 和 live fan-out，不能只看一条路径。
- 涉及 community state 时搜索所有 cache/reset/fan-out keys 是否 tenant-scoped。
- 每次形成稳定结论后更新 `.pi/knowledge/`，原始搜索输出只放 `.pi/local/`。
