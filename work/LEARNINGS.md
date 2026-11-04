# LEARNINGS

现象 → 根因 → 规则，每条 ≤3 行。director、builder、verifier 开工前读。下面 10 条来自真实项目，已去掉业务细节，作为起点保留。

1. **并行分支各自绿，合并后红** → 两个分支改动有交叠域（哪怕文件级零交集），各自验证不覆盖接缝 → 每次合并进 main 后立即跑所在包全量回归，红了当场修或回退；这是派 UI 评审 / review 前的硬闸门。
2. **按文件清单 checkout 合并时，把另一个修复整个盖掉** → 清单 checkout 不是 merge，同一文件被两个分支改过时静默覆盖 → 合并前 `git diff <base>..<head> --name-only` 与合并窗口内已改文件对一遍，命中就三方合并或先 rebase。
3. **两个 builder 的 stash 互相串位** → `refs/stash` 跨 worktree 共享，后 pop 的拿到对方栈顶 → 多 worktree 并行期间禁用 `git stash`，改用 `git diff > x.patch` + `git apply`。
4. **verifier 验收时 builder 正在 rebase，共享 worktree 出现冲突** → 验收与开发共用现场 → verifier 用 `git worktree add --detach` 检出申报的 SHA 独立验证，不动 builder 现场。
5. **VERIFY 报告写在 worktree 里，worktree remove 后丢失** → 报告路径用了相对路径 → 派发提示词写明报告一律写主仓绝对路径；合并前确认报告已在主仓再 remove。
6. **清理测试进程时按父进程批量杀，把整个桌面会话注销了** → 「不用某种写法」只禁了一种写法，模型换个参数就绕过，规则靠自觉 → 按目标限制：只允许清理沙箱内的字面 PID，`pkill` / `killall` / `-P` / 负数 PID 一律禁止，用 hook 强制执行。
7. **主会话把「未解出」转述成「不存在」，评审出现假冲突** → 转述时丢了限定词 → 写 LOG 或派发时，事实性结论引原文路径和行号，不改写限定词（有 / 无、未验证 / 不存在）。
8. **集成决策只验了「能构建」，到后期才发现在目标宿主里根本跑不起来** → spike 验的是构建，不是运行 → 凡集成决策，spike 验收必须包含「在目标宿主里实际启动并走通一条旅程」；只验构建的结论标「未实证」。
9. **整目录 `git add work/` 把运行时 token、lock 带进了 commit** → agent 写 evidence 的窗口期，整目录 add 会捞进运行时产物 → add 前看 `git status --short`；`.gitignore` 排除 evidence 下的运行时目录；evidence 只放报告、脚本、截图。
10. **PLAN status 列长期失修，worktree 堆积** → 合并后只记 LOG、没同步 PLAN，或 builder 自行合并 → 合并 commit 里同时更新 status 列；`git worktree remove` 是合并的最后一步，不能省；builder 不得自行合并。

<!-- 新条目追加在下面：YYYY-MM-DD 现象 → 根因 → 规则 -->
