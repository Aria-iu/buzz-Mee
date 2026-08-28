# Buzz 只读项目调研执行计划

> **For agentic workers:** 执行时按任务顺序更新复选框。只允许写入 `.pi/`，不得修改原项目文件。

**Goal:** 形成一套有仓库证据支撑的 Buzz 项目理解文档，并识别后续个人开发的合理扩展入口。

**Architecture:** 使用 `.pi/knowledge/` 保存稳定知识，`.pi/reports/` 保存阶段结论，`.pi/local/` 隔离临时材料。所有结论从产品愿景、仓库结构、模块入口、测试说明和实现代码逐层验证。

**Tech Stack:** Markdown、Git 只读检查、Rust/Cargo workspace、Tauri/React、Flutter、Nostr/ACP/MCP。

---

### Task 1：建立隔离工作区

**Files:**
- Create: `.pi/.gitignore`
- Create: `.pi/README.md`
- Create: `.pi/INDEX.md`
- Create: `.pi/local/.gitkeep`

- [x] 创建 `.pi/` 的导航、边界说明和本地忽略规则。
- [x] 验证新增文件全部位于 `.pi/`。
- [x] 验证 `.pi/local/` 中除 `.gitkeep` 外的内容会被忽略。

### Task 2：建立产品与仓库概览

**Files:**
- Create: `.pi/knowledge/project-overview.md`
- Create: `.pi/knowledge/terminology.md`

- [x] 阅读 `VISION.md`、`README.md`、`AGENTS.md` 和顶层 manifests。
- [x] 记录产品定位、主要 surface、技术栈和仓库边界。
- [x] 建立 community、relay、event、channel、agent、ACP、MCP 等术语表。
- [x] 为重要结论附上仓库路径证据。

### Task 3：绘制架构与模块地图

**Files:**
- Create: `.pi/knowledge/architecture.md`
- Create: `.pi/knowledge/module-map.md`

- [x] 识别 Rust workspace crates 及其职责。
- [x] 识别 desktop、web、mobile 和基础设施入口。
- [x] 梳理关键依赖方向和协议边界。
- [x] 标记产品愿景与当前实现状态之间的差异。

### Task 4：追踪核心工作流

**Files:**
- Create: `.pi/knowledge/core-workflows.md`

- [x] 追踪 signed Nostr event 从客户端到 relay、auth、store、pub/sub 的主路径。
- [x] 追踪 channel scoping、查询和 membership 边界。
- [x] 追踪 Agent 的 ACP/MCP 交互路径。
- [x] 记录关键入口文件、错误边界和安全约束。

### Task 5：整理构建与验证方式

**Files:**
- Create: `.pi/knowledge/setup-and-testing.md`

- [x] 汇总 Hermit、Just、Cargo、pnpm 和 Flutter 的使用边界。
- [x] 区分 unit、integration、E2E 和 live workflow 验证。
- [x] 标明可能影响本地数据库、端口或现有 Desktop 环境的命令。
- [x] 只记录命令，不启动破坏性服务或执行数据重置。

### Task 6：识别个人扩展入口

**Files:**
- Create: `.pi/knowledge/extension-points.md`

- [x] 按 relay、CLI、desktop、mobile、agent 和 workflow 分类扩展点。
- [x] 对每个扩展点记录应遵循的既有模式和测试要求。
- [x] 区分低耦合扩展、跨层功能和不建议修改的核心边界。
- [x] 不在调研阶段设计或实现具体功能。

### Task 7：形成阶段报告

**Files:**
- Create: `.pi/reports/read-only-study-report.md`
- Modify: `.pi/INDEX.md`

- [x] 汇总已经确认的理解、仍不确定的问题和主要风险。
- [x] 给出后续开发准备度结论，不自动进入开发阶段。
- [x] 检查所有重要结论是否包含文件路径依据。
- [x] 检查 Git diff，确认原项目文件未被 Agent 修改。
- [x] 等待项目所有者批准下一阶段；不执行 commit 或 push。
