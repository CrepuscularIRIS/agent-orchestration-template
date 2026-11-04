# workflows/

放多 agent 的确定性流程脚本，例如多模型 code review（`mm-review`）。设计与规则见 `docs/harness/REVIEW.md`。

本模板不附脚本，只约定接口，按你的 agent 框架实现：

| 项 | 约定 |
|---|---|
| 调用 | `args: "<S|M|L> <slug> <task-id 或名字> <ref 范围>"`，由主会话调用 |
| 输出文件 | `work/<slug>/REVIEW-<task-id>.md`，头部写 base / head SHA |
| 返回主会话 | 只一行：`verdict | critical/major 数 | 报告路径` |
| 读取 | `work/REVIEW-ALLOWLIST.md`（命中的不再报）、`CLAUDE.md`、`agent-rules.md` |
| 临时目录 | 复现实验只在约定的临时目录里做，结束前删除 |
