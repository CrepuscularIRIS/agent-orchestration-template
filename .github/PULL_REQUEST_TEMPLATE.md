## 任务
- slug / id：`<slug>` / `<T-xx>`　Closes #<issue>
- 一句话：<…>

## 区域
<本 PR 改动的目录 / 文件，必须落在 PLAN 该行的区域列内；超出的说明原因>

## 捷径 / 复用来源
<上游包@版本、`来源: <url>@<sha>:<path>`，或「无」+ 理由>

## 验收证据
- VERIFY：`work/<slug>/VERIFY-<T-xx>.md`，SHA：`<sha>`
- 红基线：<命令 + 失败摘要>
- 合并后全量回归：<命令 + 通过数>

## review
档位：S / M / L / GU / –　报告：`work/<slug>/REVIEW-<T-xx>.md`（合并后补）

## 检查清单
- [ ] 独立 worktree / 分支 `feat/<T-xx>` 开发，未使用 `git stash`
- [ ] 只改区域内文件；交叠检查已做（`git diff <base>..<head> --name-only`）
- [ ] 无凭证、token、内网地址、运行时产物进入 diff
- [ ] 测试先红后绿，数量未下降
- [ ] PLAN status 列已同步
- [ ] 高风险（鉴权 / 凭证 / 审批 / 沙箱 / 数据迁移）已过 QUESTIONS
