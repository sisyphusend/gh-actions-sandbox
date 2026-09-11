# L2 — 事件触发器

## 目标

搞清楚 Actions 的"何时做"：事件（event）如何决定 workflow 是否运行、跑在哪个 ref 上、能拿到什么上下文。学完能回答：

1. push 到 feature 分支会不会触发只写了 `branches: [main]` 的 workflow？
2. `schedule` 为什么不准时？在哪个分支才生效？
3. `pull_request` 触发时 checkout 出来的是什么代码？

## 关键 YAML

见同目录两个示例（已同步到 `.github/workflows/`）：

- `01-events-overview.yml`：5 种触发器一屏看全（push / pull_request / workflow_dispatch / schedule / release）
- `02-workflow-dispatch-inputs.yml`：手动触发 + 传参（表单 / `gh workflow run -f`）

## 验证清单（学完打勾）

- [ ] push 到 main → 触发 overview，日志 `event_name=push`
- [ ] 浏览器手动触发 → `event_name=workflow_dispatch`
- [ ] 开一个 PR → 触发，`ref=refs/pull/N/merge`（注意不是分支名！）
- [ ] 给 overview 传 inputs 的那个 workflow：`gh workflow run 02-workflow-dispatch-inputs.yml -f env=prod -f dry_run=false` → 看日志回显
- [ ] 等 schedule 触发一次（UTC 3:17 ≈ 北京 11:17，可能延迟几十分钟）→ `actor=github-actions[bot]`
- [ ] 用 `act --list` 或 `gh workflow view` 确认触发器写对了（本地静态检查）

## 踩坑记录

- **schedule 只看默认分支**：cron 写在哪个文件里无所谓，GitHub 只按默认分支上的那份文件调度。
- **schedule 不准时**：整点是高峰，故意错开（如 `17 3 * * *`）；延迟几分钟到几十分钟属正常；仓库 60 天无活动会被自动停用。
- **pull_request 的 ref 是合并引用**：`refs/pull/<N>/merge` = "你的分支合并进 base 后的临时 commit"，不是你的 HEAD。想测"合并后代码"用 PR 触发是对的；想在自己的分支上跑用 `pull_request_target` 要格外小心权限（L7 再展开）。
- **tags-ignore 与 branches 过滤是"且"的关系**：`branches` 和 `tags` 同时写时两条都要满足。
- **（沿用 L1 的教训）**：job 秒失败且报 "not started" → 先查账号级问题（账单/权限），别急着改 workflow。

## 官方链接

- 触发 workflow 的事件：https://docs.github.com/zh/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows
- workflow_dispatch 传参：https://docs.github.com/zh/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#workflow_dispatch
- schedule（cron 语法）：https://docs.github.com/zh/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows#schedule

## 下一步行动

进入 L3：jobs / steps / runner —— `needs` 依赖、`if` 条件、timeout、strategy 初窥。
