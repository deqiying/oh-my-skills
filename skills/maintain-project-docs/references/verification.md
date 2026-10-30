# notes 脚本说明

脚本只校验本技能原生 notes 格式（见 [notes 规范](notes.md)）；ADR、RFC 或宿主自定义格式不用它们判定合规。本技能不提供 docs 校验脚本，docs 按 [docs 维护流程](docs-maintenance.md) 验证。

## 运行前提

- `<skill>` 指实际加载的 `SKILL.md` 所在目录。脚本不随提交自动运行，需要手动执行。
- 在宿主项目根执行，默认读取 `.agents/notes/`；位置不同时用 `AGENT_NOTE_ROOT` 指定。每次只检查一个 notes 根。笔记路径参数相对当前工作目录解析。
- Node 支持类型剥离时直接 `node <script>.ts`；Node 22.6 起的旧版本加 `--experimental-strip-types`；无法使用时改用 `npx tsx <script>.ts`（需要网络或本地已有 `tsx`）。

## 脚本与顺序

| 脚本 | 用途 |
|---|---|
| `verify-agent-note-format.ts` | 头部、状态行及各生命周期必需小节 |
| `verify-agent-note-tree.ts` | 生命周期与 class 目录、文件名、禁止 `INDEX.md`、活跃笔记内部相对链接 |
| `verify-archived-agent-notes.ts [--write]` | 冻结笔记头部与 `archived/manifest.json` 封印 |
| `archive-agent-note.ts <note> [--superseded-by <note>]` | 归档 implemented 笔记并写入封印 |

1. **改完笔记**：依次运行 format、tree 检查。
2. **归档**：上一步通过后运行 `archive-agent-note.ts`，再重跑 tree 检查（旧路径链接会失效）和 `verify-archived-agent-notes.ts`。
3. **补封印**：仅对核对过的历史未封印文件使用 `--write`。存在其他错误（含封印失配、孤立封印）时脚本以 1 退出且不写回；封印失配须人工核对后决定补封印或恢复内容。

## 结果判读

- 通过时打印 `ok: …` 并以 0 退出；失败时把每条错误写入 stderr 并以 1 退出。notes 根不存在或为空按 0 条处理。
- `verify-archived-agent-notes.ts` 在非 git 目录或基线缺少 manifest 时打印 `warning:` 并跳过只增检查，仍校验磁盘封印；汇报时注明降级。
- 链接检查只解析 notes 根内的相对链接。目录扫描跳过以 `.` 开头的条目；活跃生命周期目录根只允许 `AGENTS.md`、`CLAUDE.md` 说明文件。
- 校验通过只说明结构合规，内容仍按 notes 规范的写后语义自检人工判断。

## 已知不一致

`archive-agent-note.ts` 遇到无法解析的 `archived/manifest.json` 时只打印警告并重建清单，旧封印随之丢失；这与 notes 规范中“manifest 损坏时停止、不自动重建”的要求不一致。出现该警告应停止归档，先人工备份并核对清单，再决定是否补封印。
