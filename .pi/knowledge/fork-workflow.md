# 本 Fork 的 Git 工作规则

## Remote 策略

当前 remotes：

```text
origin   = https://github.com/Aria-iu/buzz-Mee.git
upstream = https://github.com/block/buzz.git（仅 fetch，push URL 为 DISABLED）
```

项目所有者要求：

- 只向 `origin` 推送；
- `upstream` 仅用于获取原作者更新，绝不向其推送；
- 未明确要求时不执行 commit 或 push；
- 当项目所有者明确提醒“更新/同步本地项目和 fork”时，该次提醒即授权执行本文件“同步约定”中的 fetch、合并和向 `origin` push。

## 开发分支策略

`main` 保持与 `upstream/main` 一致；个人修改位于 `feature/personal-changes`，后续具体功能也应使用独立分支，不直接在 `main` 上实现：

```text
main
└── feature/<short-name>
```

分支命名可按改动类型使用：

- `feature/<name>`：个人功能；
- `fix/<name>`：bug fix；
- `docs/<name>`：文档；
- `experiment/<name>`：明确可丢弃的试验。

Agent 不会为未批准的功能自行创建分支；只有在项目所有者批准具体功能开发后执行。

## Commit 与 Push

当项目所有者明确要求提交时：

```bash
git commit -s
```

原因：仓库 DCO gate 要求 `Signed-off-by`。本机已配置 SSH commit signing 与 `commit.gpgsign=true`，因此提交同时具有 GitHub `Verified` 签名。Commit message 遵守 Conventional Commits，例如：

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

当项目所有者提醒“更新/同步本地项目和 fork”时，按以下约定执行：

1. 检查当前分支、工作区、remotes 和上下游差异；
2. 若有未提交修改，先用 `git stash push -u` 安全保存；
3. `git fetch --no-tags upstream main` 获取原作者更新；
4. 将本地 `main` 以 `--ff-only` 更新到 `upstream/main`；若不能快进则停止并报告，不覆盖历史；
5. 将更新后的 `main` push 到 `origin/main`；
6. 返回提醒前的个人功能分支，将 `main` 合入该分支；需要 merge commit 时添加 sign-off，并由已配置的 SSH key 签名；
7. 若发生冲突，停止并报告，不猜测解决、不强推；
8. 无冲突时将个人功能分支 push 到 `origin`；
9. 恢复 stash，最后核对本地与远端提交、工作区状态。

同步默认使用普通 merge，避免改写已共享功能分支历史；只有项目所有者明确授权时才使用 `rebase` 或 `--force-with-lease`。同步动作与功能实现提交分开，便于审计和回退。
