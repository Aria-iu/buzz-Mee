# Buzz 个人开发扩展入口

> 本文只识别扩展边界，不选择或设计具体功能。

## 选择原则

优先选择：

1. 用户价值明确、能在一个 surface 内闭环；
2. 复用现有 event kind 或 generic `/events`、`/query`、`/count`；
3. 不改变 tenant、auth、membership 与 signed-event 核心不变量；
4. 可以用 unit test + 一个真实 workflow 验证；
5. 与 upstream 同步时冲突面小。

## 低耦合扩展

### 1. `buzz-cli` 新的组合型命令

适合把现有 event/query 能力包装为更好用的 Agent operation，例如组合读取、格式化或批处理。入口通常是：

- `crates/buzz-cli/src/lib.rs`：Clap command surface；
- `crates/buzz-cli/src/commands/`：命令实现；
- `crates/buzz-cli/src/client.rs`：relay interaction；
- `crates/buzz-cli/TESTING.md`：live runbook。

约束：stdout 保持 normalized JSON，stderr structured error，exit code 保持既有 contract；agent-facing operation 应优先进入 CLI，而不是塞进 `buzz-dev-mcp`。

### 2. Desktop 单一 feature 内的 UI/UX

`desktop/src/features/<feature>/` 已按产品 feature 组织，适合增加局部 presentation、filter、preference 或 workflow UI。共享能力放 `desktop/src/shared/`。

约束：

- readable text 使用 rem-based named Tailwind token；
- community-scoped module singleton 必须加入 `resetCommunityState()`；
- 遵守 1000 lines/file gate；
- React Query object 本身 reference 不稳定，memo/perf 需要 stable dependency；
- UI PR 需要 E2E/runtime evidence 与截图。

证据：`AGENTS.md` Desktop App、`desktop/src/features/`。

### 3. Mobile 单一 feature

适合在 `mobile/lib/features/<feature>/` 增加已有 relay capability 的 mobile surface。

约束：

- 使用 `HookConsumerWidget`/`ConsumerWidget`，不用 `StatefulWidget`；
- feature 不直接 import 其他 feature，只依赖 `shared/`；
- 使用 Riverpod、Grid/Radii/theme extensions；
- 优先 widget test，并在真实 simulator/device 验证用户 workflow。

证据：`AGENTS.md` Mobile App、`mobile/lib/features/`。

### 4. Web repo browser 或 invite presentation

`web/src/features/repos/` 与 `web/src/features/invite/` 边界相对清晰，适合不改变 relay protocol 的展示增强。

约束：Web bundle 由 relay fallback 托管，route 和 content negotiation 不能破坏 NIP-11、WebSocket、git smart HTTP 与 invite landing。

## 中等耦合扩展

### 1. 使用现有 Kind 的新客户端能力

如果 relay 已存储并查询所需 event，主要工作是 SDK/CLI/client/UI。需要确认：

- kind 已在 `buzz-core/src/kind.rs`；
- ingest scope/validation 已支持；
- read filters 带明确 `kinds` 与正确 `h`/`d`/`e` tags；
- Desktop 与 Mobile constants 不发生漂移。

### 2. Workflow 新 action 或 trigger

入口：

- `buzz-workflow/src/schema.rs`：YAML schema；
- `buzz-workflow/src/executor.rs`：执行与 condition/template；
- `buzz-workflow/src/action_sink.rs`：effect interface；
- `buzz-relay/src/workflow_sink.rs`：relay-side implementation；
- Desktop workflow editor 与 CLI commands。

这是跨 crate 但边界明确的扩展。要特别检查 SSRF、timeout、owner current authority、loop prevention、trace 和 multi-pod scheduling。

### 3. Agent harness 行为

`buzz-acp` 可扩展 queue policy、prompt framing、observer rendering source、provider discovery 或 session lifecycle，但其安全面较大：owner/sibling author gate、DM fail-closed、bounded queue、turn timeout、reconnect dedup 都是已有不变量。

相关入口：`crates/buzz-acp/src/{config,filter,queue,pool,relay,prompt_framing}.rs`。

## 高耦合纵向功能

### 新 Event Kind

只有现有标准或 custom kind 无法表达功能时才增加。典型路径：

1. `buzz-core/src/kind.rs` 定义并注册；
2. shared payload/event builder；
3. `buzz-relay/src/handlers/ingest.rs` scope、global/channel、validation；
4. 必要的 `buzz-db` migration/store；
5. side effect、search/privacy、audit、pubsub/workflow；
6. `buzz-cli` agent-facing command；
7. Desktop/Mobile/Web consumers；
8. unit、integration、E2E 与真实 workflow。

这类功能很容易跨 5–8 个层，必须先写独立设计和 claim-based test plan。

### 新 HTTP Endpoint

默认不建议。先尝试 signed Nostr event、`POST /events`、`POST /query` 或 `POST /count`。只有 media、webhook、git、metadata、health 等真正 HTTP-native 场景才扩展 router，并保证 host-derived community 在 auth/data lookup 之前绑定。

## 不建议作为第一次个人改动的区域

1. `TenantContext`、host binding、community deletion 与 conformance boundary；
2. NIP-42/NIP-98/NIP-OA authentication；
3. private-channel subscription/fan-out access chokepoint；
4. event replaceability、thread atomic counters、reaction atomic insert；
5. DB read-replica fence 与 transaction/session invariants；
6. audit hash chain；
7. git receive-pack policy 与 protected refs；
8. remote-agent key handoff/provider conformance。

这些区域不是不可修改，而是需要 security review、formal/runtime evidence 或真实多节点验证，不适合作为熟悉仓库的第一步。

## Fork 开发建议

- 保留原 upstream remote，个人功能使用独立短生命周期 branch；
- 一个 branch/PR 只做一个纵向切片；
- 上游同步与个人功能开发分开处理，避免把 merge noise 混入功能 commit；
- 所有 Agent 调研、计划、决策和验证摘要继续只放 `.pi/`；
- 功能代码仍进入正确的原模块，不能用 `.pi/` 或自建 `my-code/` 绕过项目架构；
- 未经项目所有者明确要求，不创建 commit。

## 推荐的首次开发范围

首次功能最好符合以下形状：复用现有 kind + 修改一个客户端/CLI surface + 添加局部测试 + 可执行一个 end-to-end 用户流程。等熟悉 ingest、tenant 和 membership 后，再考虑新增 kind 或 relay-side state。
