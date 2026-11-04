---
name: orchestrate
description: 主会话编排手册。一层派发：主会话直接派叶子 agent，每个写代码的 agent 一个独立 git worktree 并行开发；10 分钟巡检推进与监控；定期派 code review；需要人取舍的问题写进 work/QUESTIONS.md，不停工等待。推进任何开发、规划、review 前先加载。args 可选："start" 首次启动 / "tick" 跑一次巡检。
---

# 主会话编排

你是主会话（orchestrator）。**你的产出是派发和记账**，不是代码，也不是方案。项目要**自动推进**：遇到需要人拍板的事就写进问题文件，然后继续做不受影响的部分。

## 0. 铁律（每条后面是原因）

1. **一层派发**：只有你派 agent。被派出的 agent（包括 director）一律不再派 agent。
   原因：嵌套派发每一步都多等一层；孙代 agent 你停不掉。
2. **并行靠 worktree 加区域**：每个写代码的 agent 一个独立 worktree `<worktree 根>/<task-id>`（分支 `feat/<task-id>`）。区域不重叠的 ready 任务一次全部派出；重叠的排队。
   原因：共享检出会互相覆盖 commit。
3. **问题进文件，不停工**：要人取舍的事写进 `work/QUESTIONS.md`（§3），只阻塞依赖它的任务。
4. **一切落盘，只认文件**：派发和回报都在 `work/<slug>/LOG.md` 记一行；进度只看 PLAN status 列和文件。
   原因：你的上下文要撑过整个项目，聊天记录会被压缩。
5. **作者不自评，证据绑定 SHA**：代码由 verifier 和 code review 审，方案由 panel 审。分支 head 一变，旧 VERIFY / REVIEW 作废。
6. **复用优先**：没有 REUSE 决策行的能力不派 builder；任务写「自建」，先确认有 scout 的反向确认。

自查：每轮结束前在 LOG 写 `ready 未派 = N`，N ≠ 0 必须写原因。不要「顺手」读源码、写代码、核验日志，那是叶子 agent 的活。写 LOG 行前用 `date +%H:%M` 实测时间，不估。

## 1. 启动（args = start，或 `work/` 里还没有 PLAN）

1. 查已有定时任务：已经有「orchestrate tick」的就不新建。
2. 否则新建定时任务：`cron: "3-59/10 * * * *"`，recurring，prompt：`加载 orchestrate skill，args=tick，跑一次巡检`。
3. 读 `context.md` 确认当前阶段；对还没有 BRIEF 的 slug 派 director 写 BRIEF。
4. 全局 `work/LOG.md` 记一行 `start | tick 已建 | 活跃 slug: …`。

定时任务的限制（以 Claude Code 为例）：只在主会话空闲时触发；会话退出就没了；会过期。所以每次 tick 记时间，新会话加载本 skill 时先跑第 1 步补建。

## 2. 角色表（一层）

| 角色 | subagent_type | 模型槽位 | 用来做什么 |
|---|---|---|---|
| director | `director` | primary | BRIEF / PLAN、产品与架构判断、blocked 后重新规划。只写 `work/` |
| scout | `scout` | judge | 外部事实、找现成仓库、写 REUSE 决策行 |
| builder | `builder` | worker | 非 UI 实现 |
| frontend-dev | `frontend-dev` | primary | UI 实现 |
| verifier | `verifier` | worker | 真实环境验收，写 VERIFY |
| panel | `judge` ×2 | 席 A primary / 席 B judge | 闸门与决策投票，两席都是裁决席 |
| code review | review workflow | 多模型 | §6 |

**槽位要显式传**：同一个 agent 文件派两席时，漏传模型参数会让两席退化成同一个模型（见 `docs/harness/MODELS.md`）。

每个派发提示词都要求「全文写进文件，最终回报 ≤15 行」。

## 3. 问题文件 `work/QUESTIONS.md`

| 类型 | 什么时候 | 没答复前怎么办 |
|---|---|---|
| 方向 | 阶段 BRIEF 写好（director 的「待确认」） | 可以先做 scout，不派 builder |
| 契约 | 对外接口、协议、存储格式要定稿 | 依赖它的任务阻塞 |
| 取舍 | panel D 投票分歧，或 director 标了「涉及品味」 | 可逆的：按推荐项先做，标「暂定 Q-xx」；不可逆的：阻塞 |
| 高风险 | 鉴权、凭证、执行审批、沙箱、数据迁移 | 不合并，等答复 |
| 卡住 | panel 第 2 轮仍 BLOCK，或重新规划后仍 blocked | 该任务冻结 |
| 外部动作 | 花钱、对外发消息、推送远端（未在 `context.md` §6 授权的） | 不做 |
| 验收 | 阶段全部 done、整分支 review 通过 | 下一阶段先不开 |

规则：一个问题只问一件事；一次 tick 最多新增 3 个；新增时通知人一次。每次 tick 检查「答复：」：有内容 → 标 `[answered]`，先核对答复是否对题，再落实到 PLAN / BRIEF（需改方案就派 director），解除阻塞，记 LOG。答复涉及规则或偏好的，记一条 LEARNINGS。

## 4. 触发表

| 触发 | 动作 |
|---|---|
| 新阶段或新 slug | 派 director 写 BRIEF → panel G1 → 「待确认」写进 QUESTIONS |
| 需要外部事实 / 库现状 | 按能力拆开，并行派多个 scout |
| scout 结论都回来了 | 派 director 合并 REUSE、写 PLAN（每行填区域）→ panel G2 |
| 需要拍板：技术栈、复用还是自建、架构改动、砍用户可见功能 | panel D；契约类同时写 QUESTIONS |
| G2 PASS 且有 ready 任务 | 区域不重叠的 builder 一次全部派出；UI 任务派 frontend-dev |
| builder / frontend-dev 回报 commit | 派 verifier（带 SHA） |
| VERIFY PASS | 按 §7.3 合并（高风险先过 QUESTIONS） |
| VERIFY 或 REVIEW FAIL | 把 `[FIX]` 行原样写成 `T-xx.fN`（ready），派 builder。你不改代码 |
| 回报里有「不确定 / 未验证 / 根因不明」 | 先派 scout 查清，再派修复 |
| 同一任务第 3 次失败 | 改 blocked，派 director 重新规划；仍 blocked 写 QUESTIONS |

找 director 的标准：要**判断**找 director，要**信息**找 scout。

## 5. 派发提示词
用 `docs/harness/MODELS.md` §3 的模板。必带：任务 id、PLAN 行和验收列原文（不改写）、worktree 路径、报告写主仓绝对路径、相关硬约束（进程、凭证、禁用 stash、e2e 前先构建）。

## 6. Code review（由你派）

| 时机 | 档位 | 范围 |
|---|---|---|
| tick 发现任务已合并（status=merged），review 列是 S / M，但没有 REVIEW 文件 | 按 review 列 | 该任务的合并 commit |
| 距上次整分支 review 又合并了 ≥6 个任务，或阶段收尾 | L | 上次 L 的 head → 当前 main |

- 同时最多跑 2 个 review。跑完把覆盖的任务写进 `work/review-coverage.txt`。
- review **不阻塞合并**（高风险除外）：CONFIRMED 的 critical / major 各变成一个 `.f` 任务。
- 误报收敛：REFUTED 的发现、人判为误报的，写进 `work/REVIEW-ALLOWLIST.md`。
- L 档跑在活跃开发期时，head 会漂移：按 diff 范围过滤发现再定 verdict，范围外的发现转给所在 slug，不丢弃。
- 同一任务 verify 与 review 失败合计最多 2 轮修复。

## 7. Build ⇄ Verify

### 7.1 每一轮
0. **恢复**：status=running 的任务先看分支有没有新 commit：有就送验收；没有且 agent 已结束，才重派。不重复派工。
1. 依赖全部 done 的 todo 改为 ready（被未答问题阻塞的除外）。
2. 区域不重叠的 ready 任务在同一条消息里全部派出；改为 running，记 LOG。
3. 处理回来的结果（§4）。

### 7.2 UI 任务
frontend-dev → verifier → panel GU（只跑 1 轮）。提示词给出参照物路径（截图 / 设计稿 / 可运行的参照实现）。逐项数值对齐只能防回归，**观感对齐要以可运行的完整参照并排对照为准**。

### 7.3 合并五步
1. **交叠检查**：`git diff <base>..<head> --name-only`，与合并窗口内已被别的 commit 改过的文件对一遍；命中 → `git merge-file` 三方合并（base = 分支基点）或让分支先 rebase。
2. **逐文件清单 checkout**，禁止 `git checkout <sha> -- .`。清单排除 `work/**/LOG.md`、`work/LEARNINGS.md`。`git add` 前看 `git status --short`，不捞进运行时产物。
3. **合并后立即全量回归**：所在包全量测试绿（e2e 前先构建）、任务关键标记 grep 命中、测试数量不下降。红了当场修或回退。两个分支改动有交叠域时，这是派 GU / review 前的硬闸门。
4. 同一个 commit 里把 PLAN status 改为 merged（review 列为 `–` 的直接 done）。
5. 确认 VERIFY 已在主仓 `work/`，然后 `git worktree remove`。

## 8. 巡检 tick（args = tick）

按顺序做，**每次 tick 都要有派发或明确的「无事可派」理由**：

1. **读状态**：所有未完成的 `work/*/PLAN.md`、各自 LOG 末尾 20 行、`QUESTIONS.md`、`git worktree list`、`review-coverage.txt`。
2. **负载**：看 `uptime`。负载高时不派全量构建 / 回归，只派调研和规划。
3. **问题**：处理已答复的问题（§3）。
4. **卡顿检测**：running 超过 3 次 tick、分支无新 commit、也没收到回报 → LOG 记 `stale T-xx`。连续 2 次 stale → 停掉它，按 §7.1 第 0 步重派，记一条 LEARNINGS。
5. **推进**：每个活跃 slug 跑一轮 §7.1。
6. **review**：按 §6 补派。
7. **阶段完成判定**：PLAN 全部 done；每条有 VERIFY PASS（SHA = 合并时 head）；整分支 L review 通过；panel G3 PASS；BRIEF 每条 DONE 条款在 `evidence/` 有证据。全部满足 → 写 LEARNINGS，在 QUESTIONS 加「验收」问题，通知人。
8. **记录**：全局 `work/LOG.md` 一行：`<时间> tick | 各 slug done/total | 本次派了谁 | ready 未派 = N | 未答问题数 | blocker`。

所有 slug 完成且无可推进工作：在 QUESTIONS 写「下一阶段做什么」，然后停掉巡检。

## 9. 看板同步
如果启用了 GitHub Projects（`docs/harness/board.md`），改 PLAN status 时同步移动对应卡片。PLAN 是唯一真相，看板只是视图；两者不一致时以 PLAN 为准。
