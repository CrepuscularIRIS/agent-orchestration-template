# 角色 → 模型映射

## §1 填表（项目负责人填写）

| 槽位 | 你的模型（完整 ID） | 派发参数 | 用在 |
|---|---|---|---|
| primary model | `<填>` | `<如 model="..." 或 agent frontmatter>` | director、frontend-dev、panel 席 A、review 深度 finder |
| judge model | `<填，与 primary 来源不同>` | `<填>` | scout、panel 席 B、review 后端 finder 与交叉验证 |
| worker model | `<填>` | `<填>` | 主会话、builder、verifier、review 安全 / 主力 finder |

注意：
- agent frontmatter 里 `model: inherit` 表示跟随主会话；同一个 agent 文件要派成两个席位时，**派发时显式传模型参数**，漏传两席会退化成同一个模型。
- 框架内置的模型别名可能被环境变量重映射，跑完核对 transcript 里的实际模型 ID。
- 设置了全局「子 agent 统一模型」的环境变量时，它会覆盖每个 agent 的模型配置，双席会失效。

## §2 为什么这样编排

| 规则 | 为什么 |
|---|---|
| 主会话不写代码、不写方案 | 主会话的上下文要撑过整个需求，自己写代码会把上下文塞满 |
| 一层派发 | 嵌套慢、孙代停不掉、中间汇报乱发 |
| 双席盲评，模型来源不同 | 单个模型有固定盲点，且倾向认同自己写的东西；来源不同的模型出错方式不同 |
| 作者不自评 | verifier 和 panel 的存在就是为了推翻「已完成」 |
| 先 scout 再写 | 重写现成的轮子等于引入新 bug |
| 一条任务一个 agent、一个 worktree | 便宜模型在单一明确任务上最可靠；共享检出会互相覆盖 |
| 无冲突任务一次全部派出 | 串行派发浪费墙钟时间 |
| 证据绑定 SHA | 代码改了，旧证据就不再证明任何事 |
| 修复最多 2 轮 | 同一提示词试第 3 次结果基本不变，需要换路线 |
| 判断型模型不做检索 / 轮询 / 抄写 | 它的额度最贵；检索交给 scout |

## §3 派发提示词模板

```
任务：<id> <一句话>
slug：<slug>    主仓：<绝对路径>    worktree：<worktree 根>/<id>（分支 feat/<id>）
必读：context.md、agent-rules.md、work/LEARNINGS.md、work/<slug>/PLAN.md 中 <id> 行、REUSE.md 对应行
完成标准：<PLAN 验收列原文，不改写>
为什么重要：<这条任务服务于哪条 DONE 条款>
约束：报告写主仓绝对路径；禁用 git stash；有构建步骤先构建再跑 e2e；只结束自己 spawn 的进程；凭证不进被测进程 env
回报：≤15 行，按 agent-rules.md §4
```

派给判断型模型的任务可以省略步骤，但必须保留「为什么重要」。
