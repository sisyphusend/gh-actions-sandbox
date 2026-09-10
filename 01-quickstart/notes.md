# L1 — 第一个 workflow

## 1. 本阶段一句话目标

能在自己仓库里跑通官方 quickstart 的 YAML，理解 `name / run-name / on / jobs / runs-on / steps` 的作用。

## 2. 关键 YAML 片段

> ⚠️ **路径约定**：GitHub Actions **只识别**仓库根 `.github/workflows/*.yml`（可下分子目录）。
> 真实运行的是 [`.github/workflows/01-quickstart/hello.yml`](../../.github/workflows/01-quickstart/hello.yml)。
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

**原因**：我把 `hello.yml` 放在了 `01-quickstart/hello.yml`。GitHub Actions **只在仓库根的 `.github/workflows/` 目录下查找** workflow 文件（可以下分子目录）。

**修复**：把 `hello.yml` 移到 `.github/workflows/01-quickstart/hello.yml`（按阶段分子目录，便于多阶段共存）。

**教训**：
- 所有真实运行的 workflow 都放在 `.github/workflows/` 下，按阶段分子目录（`.github/workflows/01-quickstart/`、`.github/workflows/02-events/` ...）。
- `01-quickstart/`、`02-events/` 等目录里只放 notes.md（笔记），YAML 内容以 notes.md 中的 inline 片段为准，与 `.github/workflows/` 下的真实文件保持同步。
- 后续每个阶段都会沿用这个布局约定。

### 后续踩坑（跑通后填）

- Actions 运行 URL：
- run-name 实际显示：
- 6 个 echo 输出截图：

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