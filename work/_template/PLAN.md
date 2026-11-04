# PLAN · <slug>

v1 · YYYY-MM-DD · director · 输入：`BRIEF.md`、`REUSE.md`、`work/LEARNINGS.md`

## 0. 决策记录
| # | 选了 | 放弃 | 为什么 | 记录（panel / Q 编号） |
|---|---|---|---|---|

## 1. 通用约定（全部任务适用）
- 测试命令：`<…>`；e2e 前先构建。
- 合并后跑所在包全量，绿了才算合并完成。
- 报告写主仓 `work/<slug>/…`。
- 「拦住了」类断言成对带差分对照（未加防护时同一探针命中）。

## 2. 任务图
status ∈ todo / ready / running / verify / panel / merged / done / blocked / frozen；review ∈ S / M / L / GU / –。

| id | 任务 | owner | 区域 | deps | status | review | 捷径（REUSE 决策行） | 验收（可证伪） | evidence |
|---|---|---|---|---|---|---|---|---|---|
| T-01 | 示例：新增 X 接口 | builder | `src/api/x/`、`tests/api/x.test.ts` | – | todo | S | 复用 `<包@版本>` | ① `<测试命令>` 先红后绿；② 输入 … 返回精确值 …；③ 差分：未改动的 main 上同一用例失败 | |
| T-02 | 示例：X 页面 | frontend-dev | `src/ui/x/` | T-01 | todo | GU | 复用 `<组件库>` | ① 与参照截图并排对照；② e2e 走通旅程 1 | |

## 3. 修订账
| 版本 | 改动 | 回应哪条 panel 发现 |
|---|---|---|
