# GitHub Actions 学习笔记

> 本机背景：WSL2 + Arch。2026-09-10 开始系统学习 GitHub Actions。
> 学习资料：https://docs.github.com/zh/actions （GitHub 官方中文文档）
> 学习方式：**分阶段逐步推进**，每个阶段都会产生真实可运行的 workflow，并 push 到 GitHub 验证 Actions 跑通。

## GitHub Actions 是什么（一句话）

GitHub Actions 是 GitHub 内置的 **CI/CD 与自动化平台**：把"何时做"（事件触发）+ "做什么"（job / step / Action）+ "在哪做"（Runner）写成一个 YAML 文件放进 `.github/workflows/`，GitHub 就会在指定事件发生时自动执行它，并把日志、产物、状态徽章全部串起来。

## 当前进度

| 阶段 | 主题 | 状态 |
| ---- | ---- | ---- |
| **L0** | 环境与心智能模型 | ✅ 进行中 |
| L1 | 第一个 workflow | ☐ |
| L2 | 事件触发器 | ☐ |
| L3 | Job / Step / Runner / Action | ☐ |
| L4 | 上下文、表达式、变量、Secret | ☐ |
| L5 | 制品、缓存、矩阵、并发 | ☐ |
| L6 | 端到端 CI 流水线（Go） | ☐ |
| L7 | 进阶（自定义 Action / OIDC / 自托管 Runner 等） | ☐ |

## 目录结构

```
learn-github-action/
├── 00-journal/                 # 学习日志、踩坑、阶段验收
├── 01-quickstart/              # L1：第一个 workflow
├── 02-events/                  # L2：5 种触发器
├── 03-jobs-and-steps/          # L3：jobs / steps / runners
├── 04-context-and-expressions/ # L4：上下文 / 表达式 / Secret
├── 05-artifacts-matrix-cache/  # L5：制品 / 矩阵 / 缓存 / 并发
├── 06-ci-pipeline/             # L6：Go 项目端到端 CI
├── 07-advanced/                # L7：按需展开的进阶
└── 99-snippets/                # 可复用 YAML 片段库
```

## 测试仓库

| 仓库 | 可见性 | 用途 |
| ---- | ------ | ---- |
| `gh-actions-sandbox` | 私有 | L1–L5 零散练习 |
| `gh-actions-demo` | 公开 | L6 端到端 CI + README 状态徽章 |

## 学习约定

- 每个阶段一个独立子目录，目录里有 `notes.md` + 若干 `.yml` 示例。
- 笔记统一格式：**目标 / 关键 YAML / 踩坑 / 官方链接 / 下一步行动**。
- 每周在 `00-journal/daily-log.md` 写一段"本周学到的 3 件事 + 还不懂的 2 个点"。
- 跨阶段总避坑清单见 [`00-journal/daily-log.md`](00-journal/daily-log.md)。

## 参考资料

- 官方中文文档：https://docs.github.com/zh/actions
- 官方英文文档：https://docs.github.com/en/actions
- GitHub Actions 仓库与社区：https://github.com/actions
- Marketplace：https://github.com/marketplace?type=actions