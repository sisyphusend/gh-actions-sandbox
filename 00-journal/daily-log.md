# 学习日志 & 阶段复盘

> 这里记录每天的踩坑、疑问、阶段验收、心得。结构按"阶段 / 日期"组织。

---

## 跨阶段总避坑清单（持续维护）

> 每学到一个新坑就追加一行。这部分是跨 L0–L7 通用的安全/性能 checklist。

| # | 坑 | 正确做法 | 何时学到的 |
| - | -- | -------- | ---------- |
| 1 | Secrets 写在 YAML 里 | 必须经仓库/环境设置注入 | — |
| 2 | PR 上用 `pull_request_target` + checkout 不可信代码 | fork 仓库不要这么做，会被脚本注入 | — |
| 3 | Action 引用 `@main` / `@master` | 固定到 major tag（如 `@v4`）或 commit SHA | — |
| 4 | `run:` 里直接拼 `${{ github.event.issue.title }}` 等用户输入 | 用 `env:` 转义：```yaml - run: echo "$T"   env: {T: ${{ github.event.issue.title }}}``` | — |
| 5 | `GITHUB_TOKEN` 给了多余权限 | 显式声明 `permissions: contents: read` 等最小权限 | — |
| 6 | commit message 写敏感信息 | Actions 日志长期保留，注意 | — |
| 7 | Runner 上跑不可信仓库代码 | 容易失陷，自托管 Runner 特别注意 | — |

---

## 阶段进度

| 阶段 | 开始日期 | 完成日期 | 关键产出 | 通过验收 |
| ---- | -------- | -------- | -------- | -------- |
| L0   | 2026-09-10 | —       | README + daily-log + 2 个测试仓库 | ☐ |
| L1   |          |          |          | ☐ |
| L2   |          |          |          | ☐ |
| L3   |          |          |          | ☐ |
| L4   |          |          |          | ☐ |
| L5   |          |          |          | ☐ |
| L6   |          |          |          | ☐ |
| L7   |          |          |          | ☐ |

---

## L0 — 环境与心智能模型（2026-09-10 开始）

### 学习目标
理解 GitHub Actions 在 DevOps 中的定位，建立基本名词表，准备好动手环境。

### 必读章节
- https://docs.github.com/zh/actions/get-started/understand-github-actions
- https://docs.github.com/zh/actions/get-started/actions-vs-apps

### 7 个核心名词（一句话自我解释）

| 名词 | 我的解释 |
| ---- | -------- |
| **Workflow** | （待 L0 完成时填实） |
| **Event** | （待 L0 完成时填实） |
| **Job** | （待 L0 完成时填实） |
| **Step** | （待 L0 完成时填实） |
| **Action** | （待 L0 完成时填实） |
| **Runner** | （待 L0 完成时填实） |
| **Artifact** | （待 L0 完成时填实） |

### Actions vs Apps 差异表（待 L0 完成时填实）

| 维度 | GitHub Actions | GitHub Apps |
| ---- | -------------- | ----------- |
| 触发方式 | webhook event | API 调用 |
| 持久性 | 临时（每次运行） | 长期安装 |
| 权限 | repo-scoped + workflow 内 token | installation token，可读写 |
| 适用场景 | CI/CD、自动化脚本 | 集成外部服务（如 Slack Bot） |

### 个人环境
- GitHub 账号：`sisyphusend`（token scopes: `gist`, `read:org`, `repo`）
- GitHub 计划类型：（待用户填）
- 本机：`gh` CLI 已登录 ✅（v2.100.0）
- 测试仓库已创建 ✅
  - `gh-actions-sandbox`（私有）：https://github.com/sisyphusend/gh-actions-sandbox
  - `gh-actions-demo`（公开）：https://github.com/sisyphusend/gh-actions-demo

> 创建过程小记：`gh repo create --public` 第一次报 `TLS handshake timeout`（graphql 端点）— 当时是网络抖动，重试无果后改用 REST API 兜底成功。
> **结论：这不是 WSL 下 `gh repo create` 的常态问题。后续仍优先用 `gh repo create`，仅当连续 2–3 次重试都失败时才走 REST 兜底。**

### 验收 checklist
- [x] 能在 `gh auth status` 中看到已登录账号
- [x] 两个测试仓库已创建（`gh repo view` 已验证）
- [ ] 7 个名词用自己的话说清楚了（**待用户读必读章节后填实**）
- [ ] Actions vs Apps 差异表填实（**待用户读必读章节后填实**）

---

## 后续阶段的日志模板（每完成一阶段在此追加一个 `## Ln — ...` 小节）

> 每个 Ln 小节建议包含：
> 1. 开始日期 / 完成日期
> 2. 必读章节
> 3. 关键 YAML 片段
> 4. 踩坑记录
> 5. 官方链接
> 6. 下一步行动