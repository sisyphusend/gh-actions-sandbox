# L3 — Job / Step / Runner / Action

## 目标

搞清楚"在哪做、按什么顺序做、失败了怎么办"：

1. job = 调度和计费的最小单位，每个 job 分到独立 runner（独立虚拟机）；
2. job 之间怎么串行（needs）、并行（默认）、传值（outputs）、跳过（if）；
3. step 之间怎么传值（GITHUB_OUTPUT）、怎么控制失败行为。

## 关键 YAML

同目录两个示例（已同步 `.github/workflows/`）：

- `01-jobs-orchestration.yml`：并行 → needs 串行 → outputs 跨 job 传值 → if 条件部署
- `02-step-controls.yml`：timeout-minutes、continue-on-error、failure()/always()

## 必须吃透的 4 个点

1. **needs 依赖图**：没有 needs 的 job 并行开跑；`needs: [a, b]` 表示 a、b 都成功才跑。Actions 页面的 SVG 图就是这张依赖图。
2. **跨 job 传值**：job A 在 step 上 `id: x` + `echo "k=v" >> "$GITHUB_OUTPUT"` → A 的 `outputs: {k: ${{ jobs.A.steps.x.outputs.k }}}` → 下游用 `needs.A.outputs.k`。注意只能传字符串，对象要用 JSON 字符串自己解。
3. **失败传播链**：step 失败 → job 失败 → needs 它的 job 全部跳过（显示为橙色 skipped，不是红色）。`if: failure()` 挂在哪个 job/step，就在"上游失败"时执行它——这是告警和清理的标准写法。
4. **runner 是一次性虚拟机**：每个 job 一台全新 ubuntu-latest，job 结束即销毁。所以"上次运行留下的文件"不存在——这就是 L5 要学 cache/artifact 的原因，先在这里埋个种子。

## 验证清单

- [ ] 看 [run 列表](https://github.com/sisyphusend/gh-actions-sandbox/actions) 里 L3 两条 workflow 的依赖图：lint/test 并行，build 汇聚，deploy/dryrun 二选一
- [ ] 在 main 上 push 触发：deploy 执行、deploy-dryrun 被跳过（灰色）
- [ ] 开 PR 触发：反过来，deploy-dryrun 执行
- [ ] `02-step-controls` 的日志：确认"会失败的 step"红了但 job 是绿的，且 4 个 step 都执行了
- [ ] 对比实验：删掉 `continue-on-error: true` 推一次，看后面 step 是否全部 skipped
- [ ] 用 `gh run view <run-id> --json jobs --jq '.jobs[] | {name, conclusion}'` 在 CLI 里看每个 job 的结论（success / skipped）

## 踩坑记录

- **（2026-09-11 实战踩中）job 的 outputs 里误用 `jobs` 上下文自引用**：写成 `version: ${{ jobs.build.steps.meta.outputs.version }}` → push 后 run 0 秒失败，workflow 名直接变成文件路径、没有任何日志和 annotation。正确写法是用 `steps` 上下文：`${{ steps.meta.outputs.version }}`。**识别技巧：run name 显示为路径 + 0s = workflow 级校验失败，区别于"job 没启动"（账号问题）和"step 失败"（执行问题），这是失败排查 SOP 的第三类。**
- **outputs 只能传字符串**：想传结构化数据，`echo "json=$(... | jq -c .)" >> "$GITHUB_OUTPUT"`，下游 `fromJSON()` 解。
- **GITHUB_OUTPUT vs GITHUB_ENV**：前者是 step/job 作用域（本 job 的后续 step + 下游 job），后者是环境变量作用域（本 job 后续 step）。混用是新手最常见的"变量读不到"原因，L4 展开讲 env 与 context 的区别。
- **skipped ≠ failed**：`needs` 的上游被 if 跳过时下游也会跳；想让"上游跳过也照跑"，给下游写 `if: always()` 或 `if: needs.x.result == 'skipped'`（注意这时 result 检查比 always 更精确）。
- **timeout-minutes 默认 360 分钟**：私有仓库会烧钱，公共仓库占资源；写死一个 5~15 是好习惯。

## 官方链接

- jobs 语法与依赖：https://docs.github.com/zh/actions/writing-workflows/workflow-syntax-for-github-actions#jobs
- job 间传值（outputs）：https://docs.github.com/zh/actions/writing-workflows/choosing-what-your-workflow-does/passing-information-between-jobs
- 失败时的表达式 failure() / always()：https://docs.github.com/zh/actions/learn-github-actions/expressions#status-check-functions

## 下一步行动

进入 L4：上下文与表达式 —— env / GITHUB_ENV / outputs 三者作用域对比，secrets 的取与藏。
