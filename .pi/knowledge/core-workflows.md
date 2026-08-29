# Buzz 核心工作流

## 1. WebSocket 连接与认证

```text
HTTP GET /
  → Host 解析 TenantContext
  → WebSocket upgrade
  → community lifecycle gate
  → connection semaphore
  → NIP-42 challenge
  → AUTH event verification
  → ban / allowlist / relay membership gate
  → Authenticated(AuthContext)
  → recv/send/heartbeat loops
  → cleanup subscriptions、presence、connection registry
```

关键约束：

- tenant 在读取任何 frame 前绑定，客户端不能通过 event tag 改写；
- 认证必须在 5 秒内完成；
- send data 与 control frame 使用独立 bounded channel，慢客户端会被关闭；
- heartbeat 每 30 秒执行，连续 3 次 missed pong 后断开；
- NIP-OA owner mapping 只在 cryptographic verification 和 materialization 成功后进入 auth context。

代码证据：`crates/buzz-relay/src/router.rs`、`connection.rs`、`handlers/auth.rs`。

## 2. Persistent Event 写入

WebSocket `EVENT` 和 HTTP `POST /events` 最终共享 `handlers/ingest.rs::ingest_event()`，即“两种 transport，一条 ingest pipeline”。

主路径概括：

1. 检查 community durable write fence。
2. 拒绝 AUTH、relay-only 和不适用于该 transport 的 kind。
3. 在 blocking task 中验证 event id 与 Schnorr signature。
4. 检查 timestamp drift、content size、authenticated pubkey match。
5. 用 allowlist 映射 kind → required `Scope`；未知 kind 拒绝。
6. command、feedback、report、moderation 等特殊 kind 进入专用 handler。
7. 解析 channel scope：一般来自 `h` tag；reaction/deletion 可从 target event 推导；global-only kind 强制 `channel_id = None`。
8. 执行 token channel restriction、channel membership/open visibility、archive 和 kind-specific validation。
9. 对 thread reply 解析 NIP-10 root/reply ancestry，并准备 transaction metadata。
10. 按 kind 执行普通 insert、replaceable upsert、parameterized replaceable upsert，或 reaction 的 atomic transaction。
11. 新插入事件执行必要 side effect；reply 同 transaction 更新 thread counter，并 best-effort 发 live summary。
12. enqueue audit，然后 schedule post-commit Redis publish、本地 guarded fan-out 和 workflow trigger。
13. 返回 NIP-01 `OK` 或 HTTP structured response。

代码证据：`crates/buzz-relay/src/handlers/ingest.rs`、`handlers/event.rs`、`handlers/side_effects.rs`。

### 重要语义

- Duplicate 普通 event 视为幂等 accepted，并返回 `duplicate:` message。
- Parameterized replaceable event 有 stale-write/conflict 语义，CLI 对应 exit code 5。
- Event 已 durable accepted 后，部分 side effect 失败不会回滚 event，但会记录 error；因此 caller 的 `OK` 不等于所有异步下游均完成。
- Search 不再有独立 index worker；PostgreSQL generated FTS column 随 event row 建立。

## 3. Ephemeral Event 写入

kind `20000..29999` 在 WebSocket handler 中单独处理：

1. Auth/pubkey/scope/write-fence 检查；
2. signature verification；
3. presence kind 更新/清除 Redis presence；
4. channel-scoped ephemeral event 检查 membership；
5. 标记 local event id，用于 Redis round-trip 去重；
6. Redis publish；
7. 通过 guarded local fan-out 发送；
8. 不写 PostgreSQL、不参与历史 REQ。

Agent observer frame 是特殊 ephemeral path：content 必须 NIP-44 encrypted，并验证 owner-agent route、freshness 与 per-agent telemetry rate limit。

代码证据：`crates/buzz-relay/src/handlers/event.rs`。

## 4. REQ、历史读取与 Live Subscription

```text
REQ(filters)
  → auth + MessagesRead scope
  → subscription/channel-value limits
  → community-scoped accessible channel resolution
  → token restriction + uncached membership confirmation
  → sensitive-kind filter gates
  → search: one-shot PostgreSQL FTS + EOSE
  或
  → register community-scoped subscription
  → 每个 filter 单独 DB query（NIP-01 OR semantics）
  → result-level access filter + dedup
  → EVENT... + EOSE
  → 之后接收 live fan-out
```

重要边界：

- 一个 REQ 最多 1024 active subscriptions；显式 `#h` value 总量有上限；
- multi-filter 采用每 filter 一次 query，再按 event id dedup，避免错误合并 limit/time window；
- search subscription 是 one-shot，不注册 live fan-out；
- private、p-gated、author-only、shared-gated 和 owner-only event 同时有 filter-level 与 result-level防护；
- live fan-out 再次检查 receiver tenant 和当前 membership，subscription registry 命中本身不是授权证明。

代码证据：`crates/buzz-relay/src/handlers/req.rs`、`handlers/event.rs`、`subscription.rs`。

## 5. Channel Scope 与 Thread

- Channel 内消息使用 `h` tag，不使用 `e` tag 表示 channel。
- kind 39000/39001/39002 是 addressable channel 描述/管理事件，channel id 在 `d` tag。
- Reaction 的 channel 从最后一个有效 `e` target event 推导。
- Thread 使用 NIP-10 `root`/`reply` marker；relay 校验 parent 与 root 都属于同一 channel，并限制深度。
- 新 reply 必须同步更新 root materialized counters。

证据：`AGENTS.md`、`buzz-core/src/nip10.rs`、`buzz-relay/src/handlers/ingest.rs`、`buzz-db/src/store/thread.rs`。

## 6. Workflow 执行

```text
stored channel event
  → post-commit dispatch
  → WorkflowEngine::on_event(community_id, event)
  → load enabled channel workflows（10s cache）
  → trigger/type/filter match
  → 重新验证 workflow owner 当前 channel authority
  → create run
  → sequential executor + ActionSink
  → update trace/status
```

Schedule trigger 每 60 秒扫描，通过 durable claim 防止多 pod 对同一 schedule instant 重复执行。Workflow execution kinds、command kinds、gift wrap 和 relay-signed workflow message 被排除，防止 loop。

当前限制：approval token/resume 尚未端到端接通，命中 approval gate 的 run 明确标记 Failed。证据：`crates/buzz-workflow/src/lib.rs`。

## 7. Agent 处理 @mention

```text
buzz-acp 启动
  → 解析 Agent identity 与 owner
  → 建立 Agent subprocess pool
  → NIP-42 连接 relay
  → discovery accessible channels
  → subscribe mention/member notifications
  → author gate（DM 更严格）
  → per-channel queue + dedup/batch
  → ACP session/prompt
  → Agent 通过 MCP/CLI 执行操作
  → signed event 回到 relay
  → crash/reconnect/session recovery
```

同一 channel 最多一个 prompt in flight；多个 Agent subprocess 可以并发处理不同 channel。默认 `respond-to=owner-only`，没有可验证 owner 时 fail closed。

证据：`crates/buzz-acp/README.md`、`crates/buzz-acp/src/lib.rs`。

## 错误与安全边界

- Client validation/auth 错误返回 sanitized 原因，内部 DB/system detail 不直接暴露。
- 多处 DB authorization lookup 失败采用 fail closed。
- 所有 runtime crate 禁止 `unsafe`，生产路径不应新增 `unwrap/expect`。
- HTTP 与 WebSocket ingest 共享核心处理，减少 transport 行为漂移。
- Community isolation 还通过 conformance trace + independent replay checker 进行运行期验证。

证据：`AGENTS.md`、`CONTRIBUTING.md`、`buzz-relay/src/handlers/ingest.rs`、`buzz-conformance`。
