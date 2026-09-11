# AGENTS.md — GitHub Actions 学习项目

- 本目录用于 `sisyphusend` 系统学习 GitHub Actions：AI 写 workflow 与笔记，用户在 GitHub 实操验证；AI 推送后必须用 `gh run list`/`gh run view` 确认成功再汇报。
- 可运行的 workflow 放 `.github/workflows/`（子目录 yml 仅为笔记示例），每个都带 `workflow_dispatch`；一次只推进一个阶段（Ln），配套 notes.md（目标/验证清单/踩坑/链接）。
- 测试仓库 `sisyphusend/gh-actions-sandbox`，远程操作用已登录的 `gh` CLI；提交信息格式 `feat(L2): 中文描述`。
- 失败排查：先 `gh run view` 看 annotation 区分"没启动 vs step 失败"，账号级问题（账单/权限）不要改 workflow；踩坑沉淀进 notes.md。
- 进度以 README.md 进度表为准（当前：L2 事件触发器进行中）。
