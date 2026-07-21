# 🐙 Git & GitHub 完全入门教程

> **本教程采用"概念 → 命令 → 实操 → 常见错误"四步教学法，覆盖从零基础到团队协作的全部核心知识。**

---

## 📖 目录

1. [Git 核心概念](#1-git-核心概念)
2. [Git 基础操作](#2-git-基础操作)
3. [分支与合并](#3-分支与合并)
4. [GitHub 协作流程](#4-github-协作流程)
5. [GitHub Actions — CI/CD 自动化](#5-github-actions--cicd-自动化)
6. [高级技巧与灾难恢复](#6-高级技巧与灾难恢复)
7. [常见错误与修复](#7-常见错误与修复)
8. [命令速查表](#8-命令速查表)
9. [推荐学习资源](#9-推荐学习资源)

---

## 1. Git 核心概念

### 1.1 Git 是什么？

Git 是目前世界上最先进的**分布式版本控制系统**。它由 Linus Torvalds（Linux 内核的创建者）于 2005 年开发，用于管理 Linux 内核的源代码。

> **Git ≠ GitHub**：Git 是运行在你电脑上的版本控制工具，GitHub 是存放 Git 仓库的云端平台。

### 1.2 三种状态与三个工作区

这是理解 Git 最重要的思维模型（来源：Pro Git Book）：

| 状态 | 所在位置 | 含义 |
|------|---------|------|
| **Modified（已修改）** | Working Directory（工作目录） | 你改了文件，但还没告诉 Git |
| **Staged（已暂存）** | Staging Area / Index（暂存区） | 已标记这些修改，准备纳入下一次提交 |
| **Committed（已提交）** | .git Directory（本地仓库） | 数据已安全存入 Git 数据库 |

```
工作目录 ──git add──▶ 暂存区 ──git commit──▶ 本地仓库 (.git)
   ↑                                                    │
   └──────────────── git checkout ──────────────────────┘
```

> 💡 **关键认知**：`git commit` 记录的是**暂存区**的快照，而不是工作目录。如果在 `git add` 之后又修改了文件，必须再次 `git add`。

### 1.3 Git 的四个核心设计思想

1. **快照，而非差异** — 每次 commit 是整个项目的完整快照，未变的文件只存链接
2. **几乎所有操作都是本地的** — 查看历史、比较差异、提交全在本地完成，无需网络
3. **一切都有校验和（SHA-1 哈希）** — Git 能检测到任何数据损坏
4. **Git 几乎只增不删** — 一旦提交，数据很难丢失

---

## 2. Git 基础操作

### 2.1 安装与初始配置

```bash
# 检查是否安装
git --version

# 配置用户名和邮箱（必须！）
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# 推荐配置
git config --global init.defaultBranch main      # 默认分支名设为 main
git config --global core.editor "code --wait"    # VS Code 作为默认编辑器
```

### 2.2 创建仓库与第一次提交

**从零开始（git init）：**

```bash
mkdir my-project && cd my-project
git init
echo "# My Project" >> README.md
git add README.md
git commit -m "chore: initial commit"
```

**克隆已有仓库（git clone）：**

```bash
git clone https://github.com/user/repo.git
cd repo
# 已包含完整历史，可以直接开始工作
```

### 2.3 暂存与提交

```bash
git status                    # 查看当前状态（最常用的命令之一）
git add <file>                # 暂存指定文件
git add .                     # 暂存当前目录所有修改
git add -p                    # 交互式暂存（逐块选择，推荐！）

git diff                      # 工作目录 vs 暂存区（未暂存的改动）
git diff --staged             # 暂存区 vs 上一次提交（已暂存的改动）

git commit -m "feat: add user login"
git commit -m "fix: resolve null pointer in payment flow"
```

### 2.4 Commit Message 规范（Conventional Commits）

格式：`type(scope): description`

| Type | 用途 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: add OAuth2 login` |
| `fix` | 修复 Bug | `fix: prevent race condition` |
| `docs` | 文档变更 | `docs: update API reference` |
| `refactor` | 重构（不改功能） | `refactor: extract validation` |
| `test` | 测试相关 | `test: add auth unit tests` |
| `chore` | 构建/工具/依赖 | `chore: upgrade deps` |

### 2.5 查看历史

```bash
git log --oneline                    # 每条提交一行
git log --oneline --graph --all      # 彩色分支图，最推荐！
git log -p                           # 显示每次提交的具体 diff
git log --author="John"              # 只看某人的提交
git show <commit-hash>               # 查看某次提交的详细信息
git blame <file>                     # 看文件每一行是谁改的
```

---

## 3. 分支与合并

### 3.1 理解分支

> 🌿 **分支的本质**：一个指向某个 commit 的可移动指针。创建分支几乎不占空间（只存一个 40 字节的 SHA-1 哈希）。

```bash
git branch <name>                # 新建分支
git checkout -b <name>           # 新建并切换
git switch <name>                # 切换分支（Git 2.23+，推荐）

git branch                       # 本地分支列表
git branch -a                    # 所有分支（含远程）
git branch --merged              # 已合并的分支（可安全删除）
git branch -d <name>             # 安全删除分支
git branch -D <name>             # 强制删除
```

### 3.2 合并的三种方式

| 方式 | 命令 | 结果 | 适用场景 |
|------|------|------|---------|
| **Merge Commit** | `git merge <branch>` | 产生合并节点，保留完整历史 | 团队协作，需要审计记录 |
| **Squash & Merge** | `git merge --squash` | N 个 commit 压成 1 个 | 零碎 WIP commit |
| **Rebase & Merge** | `git rebase main` | 线性历史，干净整洁 | 个人分支整理 |

### 3.3 Rebase 完全指南

> ⚠️ **Rebase 黄金法则**：永远不要对已推送到公共仓库的提交执行 rebase！

```bash
# 标准 rebase
git checkout feature
git rebase main

# 交互式 rebase（整理提交历史的利器）
git rebase -i HEAD~5

# 可用操作：
# pick   — 保留该 commit
# reword — 保留但修改 commit message
# squash — 合并到上一个 commit（保留 message）
# fixup  — 合并到上一个 commit（丢弃 message）
# drop   — 删除该 commit

# 如果出问题
git rebase --abort         # 放弃 rebase
git rebase --continue      # 解决冲突后继续
```

### 3.4 解决合并冲突

冲突发生时，Git 会在文件中插入标记：

```
<<<<<<< HEAD          ← 当前分支的版本
你的修改
=======               ← 分隔线
别人的修改
>>>>>>> feature       ← 要合并进来的分支
```

```bash
# 手动编辑 → 删除标记 → git add → git commit

# 或用可视化工具
git mergetool

# 太复杂直接放弃
git merge --abort

# 已推送的错误 merge（不要 reset！）
git revert -m 1 <merge-commit-hash>
```

### 3.5 主流分支策略

| 策略 | 长期分支数 | 适合团队 | 核心特点 |
|------|-----------|---------|---------|
| **GitHub Flow** | 1 (main) | 小型团队 | 最简单：拉分支 → 改 → PR → 合并 |
| **GitFlow** | 2 (main+develop) | 大型团队 | 最严格，但已标记为"遗留策略" |
| **Trunk-Based** | 1 (trunk) | 成熟团队 | 分支存活 < 1 天，需要 feature flag |
| **GitLab Flow** | 环境分支 | 分阶段发布 | 只向前合并，不向后合并 |

> 💡 **选型建议**：从最简单的开始，只在确实需要时才增加复杂度。小团队不要用 GitFlow。

---

## 4. GitHub 协作流程

### 4.1 Pull Request（PR）— GitHub 协作的核心

PR 的本质是**"我的代码写完了，请审查后合并"**。它是代码审查、讨论、知识传递的核心场所。

**PR 完整生命周期：**

```
Issue #42 → 创建分支 → 写代码+Commit → Push → 创建 PR
→ CI 自动检查 → Code Review → 修改 → Approve → Merge → 删分支 🎉
```

**PR 页面的四个 Tab：**

| Tab | 内容 |
|-----|------|
| **Conversation** | PR 描述 + 评论、Review 意见、讨论 |
| **Commits** | 每次 push 带来的 commit 列表 |
| **Files Changed** | 修改的文件及 diff（绿色=新增，红色=删除） |
| **Checks** | CI/CD 自动化检查的运行结果 |

### 4.2 Code Review 最佳实践

**Code Review Pyramid（审查优先级从高到低）：**

1. **API 语义** — 对外接口设计是否正确、安全？
2. **实现语义** — 逻辑正确吗？边界条件处理了吗？
3. **文档** — 有必要注释和变更说明吗？
4. **测试** — 覆盖是否充分？
5. **代码风格** — 交给 Linter，不要人工纠结

> 🏆 **黄金法则**："珍惜审查者的时间" —— 自动化一切能自动化的检查（Lint、格式、测试）。代码审查的目的是知识传递，不是找 Bug。

### 4.3 给开源项目贡献代码（10 步流程）

1. **Fork** — 把原仓库复制到自己账号下
2. **Clone** — `git clone <你的 fork 地址>`
3. **设置 Upstream** — `git remote add upstream <原仓库地址>`
4. **创建分支** — `git checkout -b fix/typo`，不要在 main 上直接改
5. **写代码** — 保持修改范围尽可能小
6. **Commit** — 清晰的 commit message
7. **同步 Upstream** — `git pull upstream main`
8. **Push** — `git push origin <分支名>`
9. **创建 PR** — 标题清晰、描述详细、关联 Issue、附截图
10. **响应 Review** — 耐心沟通，尊重维护者决定

### 4.4 开源贡献沟通六原则

1. 提供上下文（你做了什么，为什么）
2. 先做好功课（查 README、已有 Issue、现有 PR）
3. 简洁明了
4. 保持公开沟通（不要私信维护者）
5. 耐心等待（维护者可能很忙）
6. 尊重社区决定

---

## 5. GitHub Actions — CI/CD 自动化

### 5.1 四个核心概念

| 概念 | 说明 |
|------|------|
| **Event** | 触发器：push、pull_request、schedule、workflow_dispatch |
| **Workflow** | `.github/workflows/xxx.yml` 中定义的自动化流程 |
| **Job** | 一个任务单元，多 Job 默认并行，可用 `needs` 串行 |
| **Runner** | 执行 Job 的虚拟机：ubuntu-latest、windows-latest、macos-latest |

### 5.2 完整 CI 示例（Node.js）

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    needs: lint                          # lint 通过后才跑测试
    runs-on: ubuntu-latest
    strategy:
      matrix:                             # 矩阵测试：多版本并行
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test

  build:
    needs: test                           # 测试通过后才构建
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run build
```

---

## 6. 高级技巧与灾难恢复

### 6.1 git stash — 暂存未完成的工作

> 📦 **场景**：正在开发功能 A，领导让你立刻修 Bug。不想提交半成品，又不能丢失进度。

```bash
git stash                              # 暂存所有修改
git stash push -m "WIP: login refactoring"
git stash list                         # 查看所有 stash
git stash pop                          # 恢复最近一个并删除
git stash apply stash@{1}              # 恢复指定的 stash
git stash branch new-branch            # 从 stash 创建新分支（最安全）
```

### 6.2 git cherry-pick — 精准移植单个提交

> 🍒 **场景**：develop 上有一个安全修复，需要应用到 release/v1.x、v2.x、v3.x，但不能整个合并 develop。

```bash
git cherry-pick <commit-hash>
git cherry-pick A B C                   # 移植多个
git cherry-pick --abort                 # 放弃
# ⚠️ 会产生新 hash，如果原 commit 后续通过 merge 进入目标分支会造成冲突
```

### 6.3 git reflog — 终极救命稻草

> 🆘 **git reflog 记录了 HEAD 和分支指针的所有移动历史**（默认保留 90 天）。只要 commit 曾经存在过，就能找回。

| 灾难场景 | 恢复方法 |
|---------|---------|
| 意外 `git reset --hard` | `git reflog` → `git reset --hard HEAD@{2}` |
| 误删分支 `git branch -D feat` | `git reflog` → `git branch feat <sha>` |
| Rebase 搞砸 | `git reset --hard ORIG_HEAD` |
| 误用 `git commit --amend` | `git reset --hard HEAD@{1}` |
| Detached HEAD 提交后切走 | `git reflog` → `git branch save-work <sha>` |

> ⚠️ **reflog 只能恢复曾 commit 过的内容**。从未 commit 的修改在 `git reset --hard` 后永久丢失。危险操作前先 stash 或提交到临时分支！

### 6.4 git bisect — 二分法定位 Bug

> 🔍 **场景**：用户报告 Bug，你确定三周前没问题，中间有 500 次提交。Bisect 在约 9 步内找出元凶。

```bash
git bisect start
git bisect bad HEAD            # 当前版本有问题
git bisect good v1.0.0         # 三周前没问题

# Git 自动 checkout 到中间 commit，你测试后告诉 Git：
git bisect good                # 这个版本没问题
git bisect bad                 # 这个版本有 bug

# 重复几次，精确找到引入 bug 的第一个 commit
git bisect reset               # 结束 bisect

# 全自动！
git bisect run npm test        # 用脚本自动判断
```

### 6.5 交互式 Rebase — 提交历史整容术

把 12 个"fix typo""WIP""oops"整理成 3 个清晰的逻辑提交：

```bash
git rebase -i HEAD~5

# 编辑器打开后：
# pick a1b2c3d feat: add login page
# fixup b2c3d4e fix typo        → 合并到上一个，丢弃 message
# fixup c3d4e5f WIP             → 合并到上一个，丢弃 message
# reword e5f6g7h feat: complete login  → 保留但改 message
#
# 结果：5 个 commit → 2 个干净的 commit！

# 自动化技巧
git commit --fixup=<sha>              # 创建 fixup commit
git rebase -i --autosquash main       # 自动排列 fixup
```

### 6.6 Git Hooks — 自动化门卫

| Hook | 触发时机 | 典型用途 |
|------|---------|---------|
| **pre-commit** | `git commit` 之前 | 跑 Lint、扫描密钥、拒绝大文件 |
| **commit-msg** | 输入 message 后 | 强制 Conventional Commits 格式 |
| **pre-push** | `git push` 之前 | 跑完整测试、阻止直接 push 到 main |

```bash
# 团队共享 hooks（放在版本控制中）
mkdir .githooks
# 编写 .githooks/pre-commit（可执行脚本）
git config core.hooksPath .githooks
```

### 6.7 git worktree — 同时工作在多个分支

```bash
# 在另一个目录创建关联工作树，共享同一个 .git
git worktree add -b hotfix ../repo-hotfix main
# 不丢失 IDE 状态，不 stash，不切换分支！
```

---

## 7. 常见错误与修复

### 7.1 绝对不要做的事

1. ❌ 不要对已推送的 commit 执行 rebase
2. ❌ 不要 `git push --force` 到共享分支（用 `--force-with-lease`）
3. ❌ 不要提交机密信息（密码、API Key）到仓库
4. ❌ 不要提交大文件（>100MB 二进制文件）——用 Git LFS
5. ❌ 不要让分支存活数周不合并
6. ❌ 不要在 GitHub 网页上直接编辑文件
7. ❌ 不要用 `git reset` 撤销已推送的 commit ——用 `git revert`

### 7.2 常见错误速查

| 错误 | 原因 | 修复 |
|------|------|------|
| `git add` 后又改了文件 | add 是快照 | 再次 `git add` |
| `git diff` 没有输出 | 全部已暂存 | `git diff --staged` |
| commit message 写错 | 手误 | `git commit --amend -m "..."` |
| 忘了加文件到上次 commit | 少 add 了 | `git add <file>` → `git commit --amend --no-edit` |
| push 被拒绝 | 远程比你新 | `git pull --rebase` → `git push` |
| 进入 Vim 不知道怎么退出 | 忘加 `-m` | `Esc` → `:wq` → `Enter` |
| Detached HEAD 警告 | checkout 了 commit | `git switch -` |

---

## 8. 命令速查表

### 基础操作
| 命令 | 用途 |
|------|------|
| `git init` | 初始化新仓库 |
| `git clone <url>` | 克隆远程仓库 |
| `git status` | 查看工作区状态 |
| `git diff` | 查看未暂存的改动 |
| `git diff --staged` | 查看已暂存的改动 |
| `git add <file>` / `git add -p` | 暂存文件 / 交互式暂存 |
| `git commit -m "..."` | 提交 |
| `git commit --amend` | 修改上一次提交 |

### 分支操作
| 命令 | 用途 |
|------|------|
| `git branch` / `git branch -a` | 本地 / 所有分支 |
| `git switch -c <name>` | 新建并切换分支 |
| `git merge <branch>` | 合并分支 |
| `git rebase <branch>` | 变基 |
| `git rebase -i HEAD~N` | 交互式变基 |
| `git branch -d <name>` | 删除分支 |

### 远程操作
| 命令 | 用途 |
|------|------|
| `git remote -v` | 查看远程仓库 |
| `git fetch` | 拉取更新（不合并） |
| `git pull --rebase` | 拉取 + 变基（推荐） |
| `git push origin <branch>` | 推送到远程 |
| `git push --force-with-lease` | 安全强制推送 |

### 撤销与恢复
| 命令 | 用途 |
|------|------|
| `git restore <file>` | 丢弃工作区修改 |
| `git restore --staged <file>` | 取消暂存 |
| `git reset --soft HEAD~1` | 撤销 commit（保留修改） |
| `git reset --hard HEAD~1` | 彻底撤销（⚠️丢弃修改） |
| `git revert <hash>` | 安全撤销（创建反向 commit） |
| `git reflog` | 灾难恢复（查看 HEAD 历史） |

### 高级操作
| 命令 | 用途 |
|------|------|
| `git stash` / `git stash pop` | 暂存 / 恢复工作区 |
| `git cherry-pick <hash>` | 移植单个 commit |
| `git bisect start` | 二分法定位 Bug |
| `git blame <file>` | 查看每行代码作者 |
| `git worktree add` | 创建关联工作树 |
| `git log --oneline --graph --all` | 彩色分支历史图 |

---

## 9. 推荐学习资源

按学习顺序排列：

1. 📘 **[Pro Git Book](https://git-scm.com/book/zh/v2)** — 最权威的 Git 教材，免费
2. 🎓 **[Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials)** — 图解清晰，强烈推荐
3. 🌐 **[The Odin Project — Git Basics](https://www.theodinproject.com/lessons/foundations-git-basics)** — 最适合初学者的互动教程
4. 🎮 **[Learn Git Branching](https://learngitbranching.js.org)** — 游戏化学习 Git 分支
5. 🏫 **[GitHub Skills](https://skills.github.com)** — GitHub 官方交互课程
6. 📖 **[opensource.guide](https://opensource.guide)** — 开源贡献入门必备

---

> 🎉 **恭喜！** 你已完成了从 Git 基础概念到 GitHub 协作工作流、从分支策略到灾难恢复的完整学习路径。掌握这些知识后，需要的就是在实际项目中不断练习。
>
> 本教程基于 **Pro Git、Atlassian Git Tutorials、GitHub Docs、freeCodeCamp、The Odin Project、opensource.guide** 等 29 篇权威教程编写。
>
> 📝 由 **Claude Code** 自动生成 · 最后更新：2026-07-21
