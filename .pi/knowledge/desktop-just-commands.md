# Desktop `just` 命令速查

> 目的：区分“仅前端”“前端 + Tauri”以及“前端 + Tauri + Relay”命令。事实依据为仓库根 `Justfile`、`desktop/package.json`、`desktop/src-tauri/tauri.conf.json` 和 Windows CI/Release workflow。

## 一眼选择

| 开发目标 | 命令 | React/Vite | 真实 `src-tauri` | Relay | 说明 |
|---|---|---:|---:|---:|---|
| 只开发前端 UI | `just desktop-dev` | 是 | 否 | 否（可用 Mock） | 只启动 Vite；浏览器打开终端打印的端口，并在 URL 使用 `?e2e=mock&resetDevState=1` |
| 前端 + 真实 Tauri，不启动本地 Relay | `just desktop-standalone` | 是 | 是 | 不启动 | 适合真实 IPC、Keyring、窗口、Native WebSocket 等；可在应用中连接远程 Relay |
| 前端 + Tauri + 本地 Relay | `just dev` | 是 | 是 | 是 | 还会启动/检查 Docker、PostgreSQL、Redis、migrations 和本地 `buzz-relay` |
| 前端 + Tauri + 内部 staging Relay | `just staging` | 是 | 是 | 远程 staging | 使用仓库写死的内部 staging 地址，通常需要内部网络/权限 |
| 前端 + Tauri + production Relay | `just production` | 是 | 是 | 远程 production | 面向真实远程环境，不应用于随意测试未完成改动 |

## 仅前端命令

这些命令不编译或运行 `desktop/src-tauri`。

| 命令 | 实际作用 | 是否启动界面 | 主要产物/结果 |
|---|---|---:|---|
| `just desktop-install` | 在 pnpm workspace 根执行 `pnpm install` | 否 | 安装/更新 `node_modules` |
| `just desktop-install-ci` | 执行 `pnpm install --frozen-lockfile` | 否 | 严格按 `pnpm-lock.yaml` 安装依赖 |
| `just desktop-typecheck` | `cd desktop && pnpm typecheck` | 否 | 执行 `tsc --noEmit` 类型检查 |
| `just desktop-check` | `cd desktop && pnpm check` | 否 | Biome 与 Desktop 项目规则检查 |
| `just desktop-fix` | `pnpm exec biome check --write .` | 否 | 自动修改可修复的格式/Lint问题 |
| `just desktop-test` | `cd desktop && pnpm test` | 否 | Node 原生 Test Runner 单元测试 |
| `just desktop-build` | `cd desktop && pnpm build` | 否 | `tsc && vite build`，生成 `desktop/dist/` |
| `just desktop-dev` | 启动 Vite开发服务器 | 是（浏览器） | HMR前端开发环境；不含真实Tauri |
| `just desktop-e2e-smoke` | 构建E2E前端并运行Playwright Smoke | 自动浏览器 | 使用Mock Bridge，不需要真实Tauri或Relay |
| `just desktop-screenshot ...` | 构建E2E前端并调用截图助手 | 自动浏览器 | PNG截图；使用Mock Bridge |

### `desktop-dev` 的运行边界

```text
浏览器
  → Vite
  → React
  → E2E Mock Bridge（URL启用mock时）
```

它不会执行：

```text
Cargo
src-tauri
Tauri IPC
本地 buzz-relay
Docker
```

Mock URL示例（端口以命令输出为准）：

```text
http://127.0.0.1:1420/?e2e=mock&resetDevState=1
http://127.0.0.1:1420/?e2e=mock&resetDevState=1#/settings
```

## 带 Tauri 的运行和构建命令

| 命令 | React/Vite | Tauri Rust | Relay/基础设施 | 用途 |
|---|---:|---:|---|---|
| `just desktop-standalone` | 运行 | 编译并运行 | 不启动本地Relay、DB或Docker | 真实Desktop开发；启动后选择或添加Community |
| `just dev` | 运行 | 编译并运行 | 启动本地Relay、Docker services、migrations | 完整本地集成开发 |
| `just staging` | 运行 | 编译并运行 | 连接内部staging Relay | staging联调 |
| `just production` | 运行 | 编译并运行 | 连接production Relay | production行为验证，风险较高 |
| `just desktop-release-build target=<triple>` | 生产构建 | Release构建 | 不启动Relay | 打包完整Tauri应用；默认Target是macOS ARM，不是Windows |

### `desktop-standalone` 的运行边界

```text
Tauri Windows/macOS/Linux应用
├── React（由Vite提供开发资源）
└── 真实src-tauri Rust
      └── 可连接用户选择的远程Relay
```

它与 `desktop-dev` 的关键区别：

```text
desktop-dev        = 浏览器 + React + Mock
desktop-standalone = 原生Tauri窗口 + React + 真实Rust
```

## Tauri Rust检查命令（不启动应用窗口）

| 命令 | 作用 | 是否运行React/Tauri窗口 |
|---|---|---:|
| `just desktop-tauri-fmt` | 格式化 `desktop/src-tauri` Rust代码 | 否 |
| `just desktop-tauri-fmt-check` | 检查Tauri Rust格式 | 否 |
| `just desktop-tauri-check` | `cargo check` Tauri crate | 否 |
| `just desktop-tauri-clippy` | 对Tauri crate运行Clippy | 否 |
| `just desktop-tauri-test` | 运行Tauri Rust测试 | 否 |
| `just desktop-ci` | 前端检查/测试/构建 + Tauri检查/测试 | 否 |

## Windows原生开发注意事项

当前仓库正式Windows CI/Release使用：

```text
Windows Runner
+ Git Bash
+ Windows原生Node/pnpm
+ Rustup/MSVC
+ Windows SDK
```

Hermit Stable没有Windows原生Binary。当前部分本地Recipe还存在Windows缺口：

1. `bootstrap`、`dev`、`desktop-standalone`等Recipe会把仓库 `bin/` 放到 `PATH` 前面，从而触发Hermit入口。
2. `scripts/instance-env.sh`生成的Tauri `beforeDevCommand`以 `exec ./node_modules/.bin/vite`开头；Tauri在Windows使用 `cmd.exe`执行该命令，而`exec`是POSIX Shell内建命令，因此会报`'exec' is not recognized`并终止启动。
3. `scripts/instance-env.sh`的worktree分支图标逻辑会调用仅适用于macOS的`swift`；Windows无`swift`时会打印警告。该警告本身不是当前致命错误。
4. `desktop-standalone`复制Sidecar的本地逻辑没有像`scripts/bundle-sidecars.sh`一样明确处理Windows `.exe`后缀。
5. `scripts/instance-env.sh`调用 `python3`，Windows环境需要确保该命令存在。

2026-06-23实测：Windows原生Git Bash中，上游基线的`just desktop-standalone`完成Sidecar Dev构建后，在Tauri `BeforeDevCommand`阶段因上述`exec`不兼容失败；此前的`swift: command not found`是非致命警告。当前个人工作树已把Vite启动命令改为跨平台的`pnpm exec vite`并在Windows跳过Swift图标生成；契约测试已通过，完整Tauri窗口重跑仍以实际启动结果为准。

因此在当前源码基线上：

| 场景 | Windows状态 |
|---|---|
| `desktop-install*`、`desktop-typecheck`、`desktop-check`、`desktop-test`、`desktop-build` | 可使用Windows原生 `pnpm`/`just` |
| `desktop-dev` | 通常可用；要求Git Bash和 `python3`；失败时可直接 `cd desktop && pnpm dev` |
| `desktop-standalone`、`dev` | 产品支持Windows，但当前本地Recipe已确认不能原样启动；需移除/替换POSIX `exec`并处理其余Windows差异，或按Windows workflow手动执行 |
| Windows正式安装包 | 参考 `.github/workflows/windows-canary.yml`、`.github/workflows/release.yml` 和 `scripts/bundle-sidecars.sh` |

## 常用组合

### 快速前端开发

```bash
just desktop-install-ci
just desktop-typecheck
just desktop-dev
```

### 前端提交前检查

```bash
just desktop-typecheck
just desktop-check
just desktop-test
just desktop-build
```

### 非Windows平台的真实Tauri、无本地Relay开发

```bash
just desktop-standalone
```

### 完整本地集成开发

```bash
just dev
```

## 证据入口

- 根 `Justfile`：所有 `just` Recipe 的当前实现。
- `desktop/package.json`：`pnpm` Script与前端依赖。
- `desktop/src-tauri/tauri.conf.json`：Tauri如何调用Vite及读取 `desktop/dist`。
- `desktop/src/testing/e2eBridge.ts`：浏览器Mock Bridge。
- `.github/workflows/ci.yml`：Windows Rust检查与 `.exe` Sidecar占位规则。
- `.github/workflows/windows-canary.yml`：Windows Canary安装包构建。
- `.github/workflows/release.yml`：正式Windows Release构建。
- `scripts/bundle-sidecars.sh`：跨平台Sidecar打包及Windows `.exe`处理。
