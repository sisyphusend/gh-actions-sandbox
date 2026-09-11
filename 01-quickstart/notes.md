# L1 — 第一个 workflow

## 1. 本阶段一句话目标

能在自己仓库里跑通官方 quickstart 的 YAML，理解 `name / run-name / on / jobs / runs-on / steps` 的作用。

## 2. 关键 YAML 片段

> ⚠️ **路径约定**：GitHub Actions **只识别**仓库根 `.github/workflows/*.yml`。
> **实测：`.github/workflows/<subdir>/<file>.yml` 子目录布局在这个 GitHub 环境下不会被识别**（`actions/workflows` API 返回 `total_count: 0`）。
> 真实运行的是 [`.github/workflows/01-quickstart-hello.yml`](../../.github/workflows/01-quickstart-hello.yml)（**文件名前缀**分组，不用子目录）。
> 本笔记里的 YAML 片段只是展示 + 字段解释，**内容应与 `.github/workflows/` 下文件保持同步**。

完整 YAML（真实文件内容）：

```yaml
name: L1 Quick Start
run-name: ${{ github.actor }} 在学习 L1 🚀

on: [push]

jobs:
  explore:
    runs-on: ubuntu-latest
    steps:
      - run: echo "🎉 触发事件是 ${{ github.event_name }}"
      - run: echo "🐧 runner 操作系统是 ${{ runner.os }}"
      - run: echo "🔎 当前分支 ${{ github.ref }}"
      - name: 检出代码
        uses: actions/checkout@v4
      - run: ls ${{ github.workspace }}
      - run: echo "🍏 本 job 状态：${{ job.status }}"
```

下面是核心结构 + 每个字段的 1 行解释：

```yaml
name: L1 Quick Start                # workflow 的名字（仓库 Actions 列表显示）
run-name: ${{ github.actor }} ...   # 每次运行的展示名（默认 = name，可加上下文动态化）
on: [push]                          # 触发事件：哪些事件会启动这个 workflow
jobs:                               # 工作流由若干 job 组成
  explore:                          # job 的 id（自定义），同一 workflow 内唯一
    runs-on: ubuntu-latest          # Runner：哪台机器跑（GitHub 托管 runner 镜像名）
    steps:                          # job 内若干 step，按顺序执行
      - run: echo "..."             # step 类型 1：直接跑 shell
        # - uses: actions/checkout@v4  # step 类型 2：调用现成的 Action
```

### 字段速记表

| 字段 | 作用 | 必填 | 我的踩坑 |
| ---- | ---- | ---- | -------- |
| `name` | workflow 的稳定名 | 否 | — |
| `run-name` | 每次运行的展示名 | 否 | 默认 `name`，加 `${{ github.actor }}` 可动态化 |
| `on` | 触发事件 | 否 | 不写 = 永不自动触发（只能 workflow_dispatch） |
| `jobs.<id>` | 一个具体 job | 是 | id 必须唯一，不能有空格 |
| `runs-on` | Runner 类型 | 是 | 三选一：ubuntu-latest / windows-latest / macos-latest |
| `jobs.<id>.steps[]` | 步骤数组 | 是 | 每个 step 要么 `run:` 要么 `uses:` |

### 上下文速记（L1 用到的 4 个）

| 上下文 | 含义 | L1 里用在哪 |
| ------ | ---- | ----------- |
| `github.event_name` | 触发事件的名称 | 第 1 个 echo |
| `runner.os` | Runner 的操作系统 | 第 2 个 echo |
| `github.ref` | 触发时的 ref（如 `refs/heads/main`） | 第 3 个 echo |
| `github.workspace` | Runner 上检出代码后的工作目录 | `ls` 那个 step |
| `github.actor` | 触发本次运行的用户名 | `run-name` 里 |
| `job.status` | 当前 job 最终状态（success/failure/cancelled） | 最后一个 echo |

## 3. 实战操作

### 步骤 1：本地准备好 workflow 文件
```bash
# 已经在 01-quickstart/hello.yml 里写好了
cat 01-quickstart/hello.yml
```

### 步骤 2：在 learn-github-action/ 里初始化 git 并 push 到 sandbox
```bash
cd learn-github-action
git init
git add .
git commit -m "L1: first workflow + L0 notes"
git branch -M main
git remote add origin https://github.com/sisyphusend/gh-actions-sandbox.git
git push -u origin main
# ⚠️ 注意：必须把 hello.yml 放在 .github/workflows/ 下，GitHub 才认！

### 步骤 3：等 GitHub Actions 页面跑通
- 访问 https://github.com/sisyphusend/gh-actions-sandbox/actions
- 看到一次绿色 ✅ 的运行记录
- 记下运行 URL

## 4. 踩坑记录

### 踩坑 1：workflow 路径必须在 `.github/workflows/`

**症状**：第一次 push 后 `gh run list` / REST API 都返回 `total_count: 0`，GitHub 完全没识别 workflow。

**原因**：我把 `hello.yml` 放在了 `01-quickstart/hello.yml`。GitHub Actions **只在仓库根的 `.github/workflows/` 目录下查找** workflow 文件。

**修复**：把 `hello.yml` 移到 `.github/workflows/` 下。

**教训**：
- 所有真实运行的 workflow 都放在 `.github/workflows/` 下。
- `01-quickstart/`、`02-events/` 等目录里只放 notes.md（笔记），YAML 内容以 notes.md 中的 inline 片段为准，与 `.github/workflows/` 下的真实文件保持同步。

### 踩坑 1b：`.github/workflows/<subdir>/` 子目录**不被识别**

**症状**：移到 `.github/workflows/01-quickstart/hello.yml`（带子目录）后，`actions/workflows` API 仍然返回 `total_count: 0`。但同一个 commit 加一个不带子目录的 `.github/workflows/01-quickstart-hello.yml`（文件名带前缀）立刻被识别为 `total_count: 1`。

**原因**：实测 GitHub Actions 在这个环境下**不递归扫描 `.github/workflows/` 的子目录**（与某些第三方文档说的"支持子目录"不一致；可能是 GHES/特定 plan 才支持，或 indexing 行为有变化）。**对个人 repo 而言，最稳妥的布局是「文件名前缀分组」**。

**修复**：
- 删掉 `.github/workflows/01-quickstart/` 子目录
- 把 hello.yml 写进 `.github/workflows/01-quickstart-hello.yml`（扁平文件名前缀）
- 后续所有阶段沿用 `.github/workflows/<stage>-<name>.yml` 命名

**最终布局约定**：
```
.github/workflows/
├── 01-quickstart-hello.yml          # L1
├── 02-events-on-push.yml            # L2
├── 02-events-on-pr.yml              # L2
├── 02-events-manual-dispatch.yml    # L2
├── 02-events-nightly.yml            # L2
├── 02-events-on-issue.yml           # L2
├── 03-jobs-and-steps-multi-job.yml  # L3
└── ...
```

### 后续踩坑（跑通后填）

- Actions 运行 URL：
- run-name 实际显示：
- 6 个 echo 输出截图：

### 踩坑 2：gh token 没有 `workflow` scope 导致 push 被拒

**症状**：
```
! [remote rejected] main -> main (refusing to allow an OAuth App to create or update workflow `.github/workflows/01-quickstart/hello.yml` without `workflow` scope)
error: failed to push some refs to 'https://github.com/sisyphusend/gh-actions-sandbox.git'
```

**原因**：
2022 年起，GitHub 强制要求 push workflow 文件时 OAuth/PAT 必须有 `workflow` scope（防止恶意 workflow 被注入用户仓库）。
当前 `gh auth status` 显示 token scopes 只有 `gist, read:org, repo`，没有 `workflow`，所以 push 被 remote 拒绝。

**修复**（待执行）：
重新登录加上 workflow scope：
```bash
gh auth login --scopes "gist,read:org,repo,workflow"
# 或交互式：
gh auth login  # 然后在提示中勾选 workflow
```

或者到 GitHub 网页 Settings → Developer settings → Personal access tokens（classic 或 fine-grained）手动创建一个含 workflow 权限的 token。

**教训**：
- 任何要 push workflow 文件到 GitHub 的账号，必须有 `workflow` scope（或 fine-grained PAT 的 Actions: Read and write 权限）。
- `gh auth login` 默认 scope 不包含 workflow（出于最小权限原则），第一次 push workflow 时会被打回。

## 5. 官方链接

- 快速入门：https://docs.github.com/zh/actions/get-started/quickstart
- 工作流语法（先看 Workflow 文件结构 + jobs 两节）：https://docs.github.com/zh/actions/reference/workflows-and-actions/workflow-syntax

## 6. 下一步行动

L1 跑通后 → **L2** 事件触发器（push / pull_request / workflow_dispatch / schedule / issues 5 种事件各一个示例）。

---

## 通过标准 checklist

- [ ] push 触发 workflow 跑通，状态绿色 ✅
- [ ] `run-name` 显示自己的 GitHub 用户名（`sisyphusend`）
- [ ] 6 个 echo step 都能在日志里看到对应输出
- [ ] `actions/checkout@v4` 那一步成功检出了代码（`ls` 输出非空或显示 .github 等）
- [ ] Actions 运行 URL 已记录到本文件第 4 节