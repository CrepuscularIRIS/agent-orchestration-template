# CLAUDE.md · 项目主控文件

`<项目名>`。用 `<语言>` 交流。**人主导需求和节奏**，AI 按本文件和 `context.md` 推进；推进开发前主会话先加载 `.claude/skills/orchestrate`。

## 1. 先读
| 文件 | 内容 |
|---|---|
| `context.md` | 项目边界：目标、非目标、沙箱、凭证、人与 AI 的分工。**所有 agent 必读** |
| `agent-rules.md` | 所有 agent 的通用规则、每个角色的准入与产出 |
| `.claude/skills/orchestrate/SKILL.md` | 主会话编排手册：一层派发、触发表、tick、合并 |
| `docs/harness/PROTOCOL.md` | 阶段、闸门、PLAN 格式、panel 汇总规则 |
| `docs/harness/MODELS.md` | 角色 → 模型映射，派发参数，提示词模板。**派发前必读** |
| `docs/harness/REVIEW.md` | 多模型 code review 档位与规则 |
| `docs/harness/board.md` | 看板列与 PLAN status 的对应 |
| `work/LEARNINGS.md` | 跨需求经验。开工前读一遍 |
| `work/<slug>/` | 每个需求的状态文件。只认文件，不认聊天记录 |

## 2. 角色与模型槽位
具体模型在 `docs/harness/MODELS.md` 填。原则：

| 槽位 | 承担角色 | 要求 |
|---|---|---|
| primary model | director、frontend-dev、panel 席 A、review 深度 finder | 判断力最强；额度通常最少，不让它检索、轮询、抄写 |
| judge model | scout、panel 席 B、review 后端 finder 与交叉验证 | 与 primary 来源不同（不同厂商或不同训练路线），保证盲评有差异 |
| worker model | 主会话、builder、verifier、review 安全 / 主力 finder | 便宜、额度多，可多开并行 |

决策核心 = primary + judge 两席；代码交给 worker。

## 3. 方法论
- 先写测试（红 → 绿 → 重构）；先找根因再修；先有证据再宣布完成。
- **复用阶梯**，停在第一级能用的：① 直接依赖在维护中的上游 ② 整仓集成（不改上游，用插件 / 配置 / 包装层补）③ 复制改造（文件头写 `来源: <url>@<sha>:<path>`）④ 逆向（行为只存在于闭源产物）⑤ 自建（写明前四级为什么不行，并有 scout 的「搜过了，确实没有」）。
- 「在维护」= 最近 6 个月有 commit 或 release，issue 有人回应。
- 作者不自评：方案走 panel，代码走 verifier + code review。外部事实派 scout 查，不凭记忆回答。
- 互不依赖的派发放在同一条消息里。

## 4. 审查强度
见 `context.md` §7。

## 5. 硬约束
- 一层派发：只有主会话派 agent。
- 并行 agent 一律用独立 worktree：`<worktree 根>/<task-id>`，分支 `feat/<task-id>`；并行期禁用 `git stash`。
- 合并按逐文件清单 checkout + 交叠检查，禁止整树覆盖；清单排除 `work/**/LOG.md`、`work/LEARNINGS.md`；合并后立即全量回归。
- 进程：只清理沙箱内自己产生的进程，禁止 `pkill` / `killall` / `-P` / 负数 PID / `sudo` / 重启 / 注销（`context.md` §4）。建议用 PreToolUse hook 强制执行。
- 凭证不进模型，也不进容器。
- 报告（VERIFY / REVIEW / evidence）一律写主仓 `work/`，不写 worktree 内的 `work/`。
