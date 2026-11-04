# 工程闭环协议

把一个需求做成「有验收证据的候选版本」的流程。编排规则（谁派谁、tick、合并）以 `.claude/skills/orchestrate/SKILL.md` 为准；本文件定阶段、PLAN 格式、panel 规则和停止条件。

## §0 铁律
1. **单一循环所有者**：主循环只由主会话的定时 tick 驱动，不叠加别的循环插件。
2. **主会话只调度和记账**：不写产品代码、不写方案、不自评。
3. **一切落盘**：每个阶段产出一个文件，下一阶段只认文件。
4. **作者不自评**：方案由 panel 审，代码由 verifier + code review 审。
5. **证据驱动**：只有带证据的结论才能推进状态；失败证据改写任务图，不原地重试同一个提示词。
6. **回执，不是承诺**：「已派出」不等于完成；证据绑定 SHA；状态文件读不到或解析不了按「未完成」处理；中断后先读 PLAN 和 LOG，接着处理 running 的任务，不重复派工。

## §1 状态目录 `work/<slug>/`

| 文件 | 写入者 | 内容 |
|---|---|---|
| `BRIEF.md` | director | 目标、用户、核心旅程、DONE 条款、非目标、范围模式、修订账 |
| `reuse/<能力>.md` → `REUSE.md` | scout → director 合并 | 每项能力的复用决策行 |
| `PLAN.md` | director 起草，主会话维护 status 列 | 任务图（§4） |
| `panels/<gate>-r<n>-<席>.md` → `panels/<gate>-r<n>.md` | judge 原文 → 主会话汇总 | 两席原文 / 机械汇总 + 必须回应清单 |
| `VERIFY-<task>.md`、`evidence/` | verifier | 验收结论与证据 |
| `REVIEW-<task>.md` | code review workflow | review 结论 |
| `LOG.md` | 主会话 | 一行一事件：时间、动作、派给谁、结果 |

## §2 阶段与闸门

```
Intake ─▶ G1 ─▶ Scout ─▶ Plan ─▶ G2 ─▶ Build ⇄ Verify ─▶ Merge ─▶ Review ─▶ G3 ─▶ Ship
```

| 阶段 | 进入条件 | 派谁 | 出口条件 |
|---|---|---|---|
| Intake | 人提出需求 | director | BRIEF 写完，DONE 条款每条可证伪 |
| G1 | BRIEF 写完 | panel G1 | PASS |
| Scout | G1 PASS | scout（按能力拆，可多开） | 每项能力在 REUSE 有决策行 |
| Plan | Scout 完成 | director | PLAN：每条任务有区域、依赖、owner、验收 |
| G2 | PLAN 写完 | panel G2 | PASS |
| Build | 任务 ready | builder×N / frontend-dev | 回报 commit SHA |
| Verify | 回报 commit | verifier | VERIFY PASS |
| Merge | VERIFY PASS | 主会话（合并五步） | 合并后全量回归绿 |
| Review | 合并后，review 列 S / M | code review | REVIEW；CONFIRMED critical / major → `.f` 任务 |
| G3 | 全部 done | verifier 整体 + review L → panel G3 | PASS |
| Ship | G3 PASS | 主会话 | 打 tag、写 LEARNINGS、停 tick |

**范围模式**（director 在 BRIEF 声明）：`EXPAND`（主动找出 10 星体验）/ `HOLD`（照单做好）/ `REDUCE`（砍到最小可用）。panel 审模式选得对不对，以及在该模式下做得好不好。

**不许偷偷挪球门**：DONE 条款经过 G1 后，任何修改都写进 BRIEF 末尾「修订账」：改了什么、为什么、哪次 panel 批准的。没有记录的修改，G3 一律 BLOCK。

## §3 tick
见 orchestrate skill §8。tick 只按落盘状态判断阶段，**绝不重派 status=running 的任务**。

## §4 PLAN 任务图格式

```
| id | 任务 | owner | 区域 | deps | status | review | 捷径 | 验收（可证伪） | evidence |
| T-01 | … | builder | src/api/ | – | todo | M | 复用 <包或 url@sha:path> / 无（理由） | `<测试命令>` 绿；… | |
```

- **区域列必填**：任务会改动的目录 / 文件。区域不重叠才能并行。
- **捷径列必填**：有没有现成库 / 插件 / 资产可用；写「无」必须给理由。
- owner ∈ builder / frontend-dev / scout。
- status ∈ `todo / ready / running / verify / panel / merged / done / blocked / frozen`（与看板列对应，见 `board.md`）。依赖全部 done 的 todo 变 ready。
- review ∈ `S / M / L / GU / –`，由 director 按风险填。
- 修复任务 id 写 `T-07.f1`，deps 指向原任务，验收列写来源（`来源 VERIFY-T-07`）。
- 一条任务最多 2 个 `.f` 修复轮；第 3 次失败改 blocked，交 director 重新规划。
- 验收要写死期望值：精确值或显式枚举，不写「被拒或不可见」二选一；「拦住了」类断言成对带差分对照。

## §5 Panel（双席盲评）

| 席 | 派发 | 模型槽位 | 权责 |
|---|---|---|---|
| A | `judge` | primary | 裁决席 |
| B | `judge` | judge | 裁决席 |

**派发**：两席在同一条消息里并行派，提示词完全一致（只有席位字母不同），互相看不到。主会话不在提示词里加自己的倾向；**必须显式传模型参数**。

**汇总**（机械执行，主会话不做判断）：
1. 任一席 BLOCK 且附证据 → **BLOCK**，回给作者，修完重开一轮。
2. 两席都 PASS → **PASS**。
3. 其余 → **REVISE**。
4. 两席带证据的 BLOCKER / HIGH 全部进「必须回应清单」，作者逐条写「已改 / 不改 + 理由」。
5. 最多 2 轮：第 2 轮仍 REVISE → 有条件通过，清单转成首批任务；第 2 轮 BLOCK → 写进 QUESTIONS 交人。
6. GU（UI）只跑 1 轮。

**决策题 D**：技术栈、复用还是自建（scout 拿不准时）、契约或架构修改、砍用户可见功能。两席投同一选项才通过；不一致 → 写 QUESTIONS。两个选项都可逆时先按改动更小、更易回退的做，标「暂定」。记录 `panels/D-<nn>-<主题>.md`。

## §6 停止与升级

| 信号 | 动作 |
|---|---|
| 任务第 3 次失败 / panel 第 3 轮 REVISE | blocked，派 director 重新规划（拆分、换路线、改用复用方案） |
| 重新规划后仍 blocked | 暂停，交人 |
| 需要产品取舍、花钱 | 交人，一次最多 3 个问题 |
| 完成判定满足 | 写 LEARNINGS，停 tick，通知人 |

## §7 方法论映射

| 阶段 | 角色 | 做法 |
|---|---|---|
| Intake | director | 先理解再发散，一次只问一个问题 |
| Build | builder / frontend-dev | 测试先行；独立 worktree |
| 遇到 bug / 修复任务 | builder | 先找根因再修 |
| Verify | verifier | 先有证据再下结论 |
| 回应审查 | 作者 | 逐条核实，不表演式同意 |

## §8 经验沉淀
每次 Ship、每个任务 blocked 之后，主会话在 `work/LEARNINGS.md` 追加：现象 → 根因 → 规则，每条 ≤3 行。director 每次 Intake 和 Plan 前先读。
