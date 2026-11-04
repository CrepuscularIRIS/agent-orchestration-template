# 看板（GitHub Projects）

PLAN 是唯一真相，看板只是视图。两者不一致时以 PLAN 为准，并修正看板。

## 列 ↔ PLAN status

| 看板列（Status） | PLAN status | 进入条件 | 谁移动 |
|---|---|---|---|
| Backlog | `todo` | 写进 PLAN，依赖未全部 done | director 建卡 / 主会话 |
| Ready | `ready` | 依赖全部 done，未被问题阻塞 | 主会话（tick） |
| Running | `running` | 已派 builder / frontend-dev | 主会话 |
| Verify | `verify` | builder 回报 SHA，已派 verifier | 主会话 |
| Panel | `panel` | 等闸门：G1 / G2 / GU / G3 / D 投票 | 主会话 |
| Merged | `merged` | 已合入 main、全量回归绿，review（S / M）尚未覆盖 | 主会话 |
| Done | `done` | review 已覆盖（或 review 列为 `–`） | 主会话 |

`blocked` / `frozen` 的任务留在原列，加 label `blocked`，卡片正文链接对应 QUESTIONS 编号。

## 卡片约定
- 一条 PLAN 任务一张 issue，标题 `<id> <一句话>`，用 `.github/ISSUE_TEMPLATE/task.md`。
- 修复任务 `T-xx.fN` 单独开卡，正文链接原任务和 VERIFY / REVIEW。
- PR 正文写 `Closes #<issue>`，PR 也加进看板，与 issue 同列。

## 用 gh 建看板（参考）
```bash
gh project create --owner <owner> --title "<项目名> board"
# 默认 Status 字段的选项改成上表七列（GraphQL updateProjectV2Field，或在网页 Settings 里改）
gh project link <number> --owner <owner> --repo <owner>/<repo>
gh project item-add <number> --owner <owner> --url <issue 或 PR url>
```
