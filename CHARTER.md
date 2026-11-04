# 工作流章程

本仓库的每一次改动都走同一条路：需求先由 director 写成带可证伪 DONE 条款的 BRIEF，经双席盲评通过后拆成 PLAN 任务；每条任务由主会话派给一个 agent，在独立 worktree 和 `feat/<task-id>` 分支上测试先行地完成，回报 commit SHA；verifier 在该 SHA 上独立验收并留下证据，作者从不自评；通过后按逐文件清单合并、立即全量回归，PLAN 与看板同步移动，code review 在合并后按风险档位补上，失败项变成 `.f` 修复任务而不是原地重试。每个 PR 对应 PLAN 中的一个任务 id，正文按 `.github/PULL_REQUEST_TEMPLATE.md` 填写，只改该任务区域内的文件，不夹带凭证、内网地址或运行时产物；需要人拍板的事写进 `work/QUESTIONS.md`，不停工等待；踩过的坑以「现象 → 根因 → 规则」写进 `work/LEARNINGS.md`，让下一次不再踩。
