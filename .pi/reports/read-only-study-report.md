# Buzz 只读调研阶段报告

## 结论

第一阶段调研已形成可用于后续讨论的项目基线。Buzz 是一个以 host-scoped community、signed Nostr event 和 relay-centered orchestration 为核心的多端协作平台，而不是普通聊天 UI。个人开发最安全的起点是复用现有 kind，在单个 CLI/Desktop/Mobile surface 中完成小型纵向功能，暂不触碰 tenant/auth/fan-out 等安全核心。

## 已确认内容

1. **产品边界：** Community 是 workspace/tenant，relay 是事实来源，人和 Agent 使用同类 cryptographic identity。证据：`VISION.md`、`README.md`。
2. **后端边界：** 根 Rust workspace 由 focused crates 组成，`buzz-relay` 负责跨 auth、DB、pubsub、search、audit、workflow 的协调。证据：根 `Cargo.toml`、`ARCHITECTURE.md`、各 crate manifests。
3. **协议边界：** Nostr event kind 是主要 dispatch switch；常规新能力应优先用 event 表达。证据：`buzz-core/src/kind.rs`、`AGENTS.md`。
4. **租户边界：** WebSocket 在 upgrade 前根据 Host 绑定 `TenantContext`，未知 host fail closed。证据：`buzz-relay/src/router.rs`。
5. **权限边界：** Auth、scope、community、channel membership 和 result-level visibility 都由 relay 强制执行；subscription match 不是授权证明。证据：`handlers/auth.rs`、`ingest.rs`、`req.rs`、`event.rs`。
6. **Agent 边界：** `buzz-acp` 是 relay-to-ACP harness，`buzz-agent` 是 Agent，`buzz-dev-mcp` 是 tool server，三者通过协议组合。证据：`VISION_AGENT.md`、`buzz-acp/README.md`。
7. **客户端边界：** Desktop 使用 Tauri/React，Mobile 使用 Flutter/Riverpod/Hooks，Web 当前主要承担 repo browser/invite。证据：各 package manifests 与入口文件。
8. **验证边界：** 项目区分 unit、integration、ignored relay E2E、Desktop E2E、Mobile tests 和 live workflow。证据：`TESTING.md`、`CONTRIBUTING.md`、`justfile`。

## 关键风险

### 高风险核心

- `TenantContext` 与 multi-community isolation；
- NIP-42/NIP-98/NIP-OA authentication；
- private channel membership 与 guarded fan-out；
- event replaceability、thread counter 和 reaction transaction；
- audit hash chain、community deletion、DB replica fence；
- git policy 和 remote-agent key handoff。

这些区域需要专门设计、安全审查和 integration/runtime evidence，不应成为第一次个人改动。

### 文档漂移

`ARCHITECTURE.md` 部分描述仍提到旧 `search_index_tx`，当前 `handlers/event.rs` 已明确 PostgreSQL generated FTS column 取代独立 indexing worker。后续调研应以代码为当前事实，以 vision 文档为目标方向。

### Fork 同步

当前 Git context：

- branch：`main`，跟踪 `origin/main`；
- `origin`：`https://github.com/Aria-iu/buzz-Mee.git`；
- 未配置名为 `upstream` 的 remote。

进入开发阶段前，建议配置原项目 upstream remote，并从 `main` 创建个人 feature branch；但本轮未修改 Git remote 或 branch。

## 尚未验证

- 本机 Hermit/Docker/Node/Flutter 环境是否完整；
- `.env`、PostgreSQL、Redis、MinIO 和默认端口当前状态；
- 当前 HEAD 的 `just test-unit`、`just test`、`just ci` 结果；
- Desktop/Mobile 的真实运行状态；
- 任一具体个人功能的产品目标和验收标准。

因此，本报告只能证明“项目结构与约束已完成文档级理解”，不能声称当前 checkout 构建或测试通过。

## `.pi` 隔离验证

执行 `git status --porcelain=v1 --untracked-files=all` 后，Agent 新增内容全部位于 `.pi/`，没有发现 Agent 修改原项目文件。`.pi/local/ignore-probe.log` 经 `git check-ignore -v` 确认由 `.pi/.gitignore` 的 `local/*` 规则忽略。

原始验证输出保存在本地忽略目录：

- `.pi/local/git-status.txt`
- `.pi/local/git-context.txt`
- `.pi/local/git-ignore-check.txt`

## 开发准备度

**状态：可以开始需求探索，但尚未批准代码开发。**

进入开发前应依次完成：

1. 由项目所有者描述希望加入的第一个功能或要解决的问题；
2. 将功能对齐相关 `VISION_*.md` 与 package-local testing guidance；
3. 编写独立设计与 implementation plan；
4. 建立非 `main` feature branch；
5. 先运行最小环境/测试 baseline；
6. 再按 TDD 实现，并记录验证结果到 `.pi/reports/`。

## 本阶段产物

- `.pi/knowledge/project-overview.md`
- `.pi/knowledge/terminology.md`
- `.pi/knowledge/architecture.md`
- `.pi/knowledge/module-map.md`
- `.pi/knowledge/core-workflows.md`
- `.pi/knowledge/setup-and-testing.md`
- `.pi/knowledge/extension-points.md`
- `.pi/knowledge/code-reading-guide.md`
- `.pi/knowledge/feature-surface-map.md`
- `.pi/knowledge/fork-workflow.md`

本阶段未执行 commit、push、build、test、migration、service startup 或 data reset。
