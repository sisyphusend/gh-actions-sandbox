# AGENTS.md — GitHub Actions 学习项目协作规则

> 本文件是给 AI 编码助手（以及未来的自己）看的协作约定。任何 AI 工具在此目录工作时都应遵循。

## 这个项目是什么

`sisyphusend` 在系统学习 GitHub Actions（2026-09-10 开始，WSL2 + Arch 环境）。
学习方式：**AI 写 workflow 和笔记 → 用户在 GitHub 上实操验证 → 踩坑沉淀回笔记**。
总路线见 [README.md](README.md)，进度日志见 [00-journal/daily-log.md](00-journal/daily-log.md)。

## 角色分工

| 角色 | 职责 |
| ---- | ---- |
| **AI 助手** | 编写/修改 workflow 示例、写笔记模板、分析失败原因、给出验证命令；**验证前主动用 `gh` 帮用户查 run 结果** |
| **用户** | 在浏览器 / CLI 实际触发、观察日志、打勾验证清单；卡住了把报错截图或描述发给 AI |

## 硬性约定（AI 必须遵守）

1. **可运行的 workflow 必须放在 `.github/workflows/`**，子目录里的 yml 只是笔记示例（GitHub 不识别子目录）。示例文件两处各留一份，内容保持一致。
2. **一次只推进一个阶段**（Ln），每阶段：示例 yml + `notes.md`（格式：目标 / 关键 YAML / 验证清单 / 踩坑 / 官方链接 / 下一步）。
3. **每个 workflow 都要有 `workflow_dispatch`**，保证用户随时能在浏览器手动触发。
4. **run-name 带 actor 和事件名**（如 `run-name: ${{ github.actor }} L2 (${{ github.event_name }})`），方便在 run 列表一眼分辨。
5. **注释写"为什么"和坑**，尤其 cron 用 UTC、schedule 只在默认分支生效、PR 的 ref 是合并引用这类反直觉点。
6. **推送后必须验证**：AI 用 `gh run list` / `gh run view` 确认 run 真的 success 再汇报；失败先查 annotation，别让用户自己盯页面。
7. **提交信息格式**：`feat(L2): 中文描述` / `fix(L2): ...` / `docs(L2): ...`。
8. 测试仓库为 `sisyphusend/gh-actions-sandbox`（当前 public），远程操作一律通过已登录的 `gh` CLI。

## 常用 gh 命令速查

```bash
# 触发
gh workflow run <file.yml> [-f key=value]      # workflow_dispatch（可传参）
git push                                        # push 事件
gh pr create --base main --head <branch>        # pull_request 事件
gh release create v0.0.1 --notes "..."          # release 事件
# schedule 无命令可触发，只能等 cron（UTC 时间，会延迟）

# 观察
gh run list --limit 5                           # 最近 run（含触发事件）
gh run watch                                    # 实时盯最新 run
gh run view <run-id> --log                      # 看日志
gh run view <run-id>                            # 看失败 annotation
```

## 失败排查顺序（AI 诊断 SOP）

1. `gh run view <run-id>` 看 **annotation** —— 区分"job 没启动"和"step 执行失败"；
2. "not started / account locked" 类 → **账号级问题**（账单、权限），改 workflow 没用（教训见 L1：账单锁定排查了 3 次才定位）；
3. step 失败 → `--log-failed` 看具体报错，对照 workflow 语法；
4. 根因和修法**必须沉淀进对应阶段的 notes.md 踩坑区**。

## 当前进度（改完这里同步更新 README 的进度表）

- [x] L0 环境与心智模型
- [x] L1 第一个 workflow（含 workflow_dispatch）
- [ ] L2 事件触发器 ← **当前阶段**（骨架已建，待用户按 notes.md 验证清单逐项打勾）
- [ ] L3 Job / Step / Runner / Action
- [ ] L4 上下文、表达式、变量、Secret
- [ ] L5 制品、缓存、矩阵、并发
- [ ] L6 端到端 CI 流水线（Go）
- [ ] L7 进阶（自定义 Action / OIDC / 自托管 Runner）

## 协作示例（本规则的实际用法）

> 用户："接着学 L3" → AI 读上一阶段 notes.md 的"下一步行动"，写出 L3 示例 + 笔记，
> push 后用 `gh run list` 确认跑通，最后给用户一组"你来做"的验证步骤（浏览器点哪里、跑哪条命令、日志里看什么）。
