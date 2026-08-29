# Buzz 环境、构建与测试

> 本文只整理仓库现有命令。本轮调研没有启动服务、构建项目、运行 migration 或重置数据。

## 工具链

建议每个 shell 先执行：

```bash
. ./bin/activate-hermit
```

Hermit 将仓库 `bin/` 放到 `PATH` 前方，确保使用 pinned Rust、Node、pnpm、Flutter、Dart 和 lefthook。最低版本信息见 `CONTRIBUTING.md`，实际开发优先相信 Hermit pins。

## 首次本地设置

```bash
cp .env.example .env   # 或使用 just bootstrap 自动创建
just setup
```

`just setup` 会 bootstrap 工具、启动 Docker services 并执行 migrations。它不是纯读取命令，可能创建 `.env`、下载工具、启动容器并改变本地 DB。

依据：`README.md`、`CONTRIBUTING.md`、`TESTING.md`、`justfile`。

## 常用运行命令

| 命令 | 作用 | 风险/副作用 |
|---|---|---|
| `just relay` | 启动 debug relay | 自动确保 services/migrations，绑定默认端口 |
| `just relay-release` | 启动 release relay | 构建并使用 release binary |
| `just dev` | Relay + Tauri Desktop | 构建 sidecars、启动 services、打开应用 |
| `just desktop-dev` | Vite desktop frontend | web-only，不代表真实 Tauri workflow |
| `just web` | Web repo browser | 启动 Vite server |
| `just mobile-dev` | Flutter app | 使用 worktree-specific debug identity；可能启动 simulator |
| `just down` | 停止 dev Docker services | 保留数据 |
| `just reset` | 重建开发环境 | **破坏性：清除共享 dev/desktop 数据** |

默认 relay 使用 `:3000`，health `:8080`，metrics `:9102`；PostgreSQL `:5432`，Redis `:6379`，MinIO `:9000/9001`。如果 Buzz Desktop 或另一 relay 已运行，端口和数据可能冲突。

## 测试层次

### 1. 静态检查与格式

```bash
just check
```

组合 Rust fmt/clippy、Desktop、Tauri、Web、Mobile、安全检查与 file-size gate。自动修复使用：

```bash
just fix-all
```

`fix-all` 会修改文件，只应在明确进入开发或格式修复任务后运行，不能把它当作只读检查。

### 2. Unit tests

```bash
just test-unit
```

不需要 Postgres/Redis。适合 core types、filter、auth、workflow schema/executor 和客户端纯逻辑。

可按 surface 缩小：

```bash
cargo test -p buzz-core
cargo test -p buzz-cli
cd desktop && pnpm test
cd mobile && flutter test
```

注意：根 Cargo workspace 排除了 `desktop/src-tauri`，根 `cargo test` 不会运行 Tauri tests：

```bash
cargo test --manifest-path desktop/src-tauri/Cargo.toml
```

### 3. Integration tests

```bash
just test
```

需要 PostgreSQL 和 Redis；recipe 会确保服务可用。修改 `buzz-relay`、`buzz-db` 或 `buzz-auth` 时应运行。

### 4. Relay E2E

```bash
cargo test -p buzz-test-client -- --ignored
```

这些测试要求先启动真实 relay。测试集中在 `crates/buzz-test-client/tests/`，覆盖 WebSocket、media、Nostr interop 等。

### 5. Desktop E2E

```bash
cd desktop
pnpm test:e2e:smoke
pnpm test:e2e:integration
```

必须使用 `pnpm build:e2e`，不能用普通 `pnpm build` 代替 mock bridge build。`reuseExistingServer` 可能导致 port 4173 服务旧 bundle；代码改动后需确认不是 stale server。

### 6. Mobile

```bash
cd mobile
dart format --output=none --set-exit-if-changed .
flutter analyze
flutter test
```

根目录等价命令：`just mobile-check`、`just mobile-test`。用户可见改动应尽量在真实 simulator/device 中验证 workflow。

### 7. Full CI Gate

```bash
just ci
```

这是 PR 前总门禁，包含 repository-wide checks/tests 及 Desktop/Web build。耗时和依赖最多，不应替代针对性快速验证。

## Live Relay + CLI 验证

根 `TESTING.md` 给出完整顺序：bootstrap/setup → release build → 单独启动 relay → health check → `buzz-admin generate-key` → `buzz channels create` → `buzz messages send/get/thread`。

关键注意：

- CLI 使用 `BUZZ_PRIVATE_KEY` 和 `BUZZ_RELAY_URL`；先清除旧 `BUZZ_AUTH_TAG` 等 inherited env，避免误连 staging 或签名失败。
- `BUZZ_RELAY_URL` 对 CLI 可为 HTTP；`buzz-acp` WebSocket 路径使用 `ws://`。
- 测试 Agent identity 必须不同于发送者 identity，且必须加入目标 channel。
- `crates/buzz-cli/TESTING.md` 是完整 command-by-command live runbook。

## UI 截图验证

Desktop 不能在普通浏览器中直接等价运行。项目提供：

```bash
just desktop-screenshot --name <name> [options]
```

Playwright spec 截图必须调用 `waitForAnimations(page)`。PR screenshot 不得上传 relay media 或第三方图床，应使用 `scripts/post-screenshots.sh`。

依据：`AGENTS.md` Testing / PR Screenshots。

## Git 与提交

项目要求：

```bash
git commit -s
```

DCO sign-off 是上游 PR 门禁；commit message 使用 Conventional Commits。当前 `.pi` 调研规则更严格：未经项目所有者明确要求，不执行任何 commit 或 push。

## 当前环境未验证项

截至当前知识基线，尚未重新验证：

- Hermit 工具是否已下载；
- Docker services 是否运行；
- 当前 `.env` 是否存在或有效；
- ports 是否被占用；
- unit/integration/E2E 当前是否通过；
- Desktop/Mobile 是否能在本机启动。

这些属于进入开发阶段后的环境基线任务，不能把文档中的 expected result 当作本机实测结果。
