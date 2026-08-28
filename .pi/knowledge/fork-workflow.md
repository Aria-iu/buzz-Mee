# 本 Fork 的 Git 工作规则

## Remote 策略

当前唯一 remote：

```text
origin = https://github.com/Aria-iu/buzz-Mee.git
```

项目所有者要求：

- 只向 `origin` 推送；
- 不向原作者仓库推送；
- 未明确要求时不添加 `upstream`；
- 未明确要求时不执行 commit 或 push。

## 开发分支策略

当前 checkout 位于 `main`。开始功能开发时，不直接在 `main` 上实现；使用独立分支：

```text
main
└── feature/<short-name>
```

分支命名可按改动类型使用：

- `feature/<name>`：个人功能；
- `fix/<name>`：bug fix；
- `docs/<name>`：文档；
- `experiment/<name>`：明确可丢弃的试验。

Agent 不会自行创建分支；只有在项目所有者批准具体功能开发后执行。

## Commit 与 Push

当项目所有者明确要求提交时：

```bash
git commit -s
```

原因：仓库 DCO gate 要求 `Signed-off-by`。Commit message 遵守 Conventional Commits，例如：

```text
feat(cli): add ...
fix(relay): reject ...
docs(pi): document ...
```

Push 目标始终写明 `origin`：

```bash
git push -u origin <branch>
```

禁止在没有明确授权时：

- `git push`；
- force push；
- 修改 remote；
- 删除 branch/tag；
- rebase 已共享分支；
- commit Agent 生成内容。

## Agent 记录与代码的关系

- 调研、计划、决策、测试摘要：`.pi/`；
- 原始日志和临时输出：`.pi/local/`，Git ignored；
- 实际功能代码：Buzz 既有模块；
- `.pi/` 不能成为放置产品代码的旁路目录。

## 同步原作者更新

当前不配置、不执行。若未来项目所有者明确要求同步，再单独设计只读 upstream fetch 流程；同步动作与个人功能 commit 分开，避免混合历史。
