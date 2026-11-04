# VERIFY · <task-id>

- 申报 SHA：`<sha>`（head 变了本报告作废）
- 检出方式：`git worktree add --detach <临时路径> <sha>`
- 验收人：verifier ｜ 时间：YYYY-MM-DD HH:MM
- 结论：**PASS / FAIL**（n/m 项通过）

## 红基线
<builder 申报的测试在改动前确实失败：命令 + 失败摘要>

## 逐条验收
| # | 验收项（PLAN 原文） | 命令 / 操作 | 关键输出 | 对照（未防护时） | 结果 |
|---|---|---|---|---|---|
| ① | … | `…` | … | … | PASS |

## 修复项
[FIX] <可直接派发的修复描述 + 复现命令>

## 证据
`evidence/<task-id>/`：报告、脚本、截图（不放 token / lock / 运行时目录）
