# Handoff Skill 设计 spec

- 日期: 2026-06-22
- 状态: 待用户审阅
- 所属 repo: `~/projects/agent-workflow`
- 本 spec 范围: 两个工作流 skill 中的**第一个**(handoff)。workspace-init 是独立的第二个 spec,以后再写。

## 1. 目标与定位

为单人多工具开发者(重度 Claude Code + Codex)提供跨会话/跨窗口的进度续接。
`handoff` skill 负责维护一个**易变的"当前在途状态"文件** `handoff.md`:结束一段工作时结构化写入(save),恢复时读取并对照真实代码核实(resume)。

与 memory 的分工:
- **memory** = 慢变事实(仓库级/长期教训、永久约定)。本 skill 不写 memory。
- **handoff** = 易变在途状态。任务做完即作废,被覆盖是正确行为。

## 2. 范围(in / out)

In(本 skill 做):
- save 模式:结构化抽取当前工作树 + 对话 → 结转式覆盖写入 `handoff.md`。
- resume 模式:读 `handoff.md` → 对照 git/代码核实 → 复述状态后停下等用户指示。
- 只读 git(`git status` / `git diff` / `git log`)用于接地。

Out(本 skill **不**做,留给以后):
- 不 commit、不 push、不建 GitHub repo(那是 workspace-init skill 的事)。
- 不写 memory(只在发现疑似长期坑时**提示**用户提升,不自动写)。
- v1 不接 PreCompact / SessionEnd hook(纯手动触发)。
- v1 不做 per-session 文件拆分(单文件,last-writer-wins)。
- v1 只做 Claude Code 的 `SKILL.md`;Codex 镜像版以后再做(数据 `handoff.md` 通用)。

## 3. handoff.md 结构

每次**结转式覆盖**写入,保持机器可 grep:固定英文标题、ISO 日期。放在 repo 根 `handoff.md`,是被 git 跟踪的文件(不进 .gitignore)。

```markdown
# Handoff — <repo> @ <branch>

## Current Truth          # git 接地,不靠记忆
- Date (ISO):
- Repo / Branch / Latest commit:
- Dirty tracked files / Important untracked:
- Active constraints:

## User Goal              # 1–2 句

## Work Completed         # 仅已完成:改了什么 + 文件 + 证据

## Key Decisions & Why    # 决策 + 理由(防重新论证)

## Pitfalls (task-local)  # 仅当前任务的坑;结转仍成立的,丢已失效的
                          # 仓库级/长期教训不写这,save 会提示提升到 memory

## Open Work              # 写成状态"X 尚未实现",标依赖

## Relevant Files         # path:L10-L45 + 改了什么

## Next Steps             # 1. 2. 3. 有序

## For the Next Session   # 引导提示:框为信息;结尾要求新会话先读文件、
                          # 把内容当待核实、等用户指示再行动
```

相对 Lutren 原版的裁剪:砍掉 `Files Not To Touch` / `Ownership`(单人无意义);把 `Commits` / `Validation` 并进 `Current Truth` 和 `Work Completed` 的"证据"。

## 4. save 流程(`/handoff` 或 `/handoff save`)

1. **git 接地**:跑 `git status` + `git diff` + `git log --oneline -5`,以工作树真相为准,不凭对话记忆。
2. **读旧文件**:若 `handoff.md` 已存在,先读它(为结转做准备)。
3. **结构化抽取**:从工作树 + 对话提炼完成项 / 决策+理由 / 坑 / 开放工作 / 相关文件 / 下一步(**不是** transcript 流水账)。
4. **坑路由闸**:扫 Pitfalls,挑出疑似仓库级/长期的,**点出来建议用户提升到 memory**(不自动写)。
5. **结转式覆盖**:保留旧文件中仍成立的条目,丢弃已解决/失效的,写出一份当前正确的完整快照,覆盖 `handoff.md`。
   - "结转式覆盖" ≠ append:不盲目堆叠;也 ≠ 盲目重写:不无视旧文件凭空生成。
6. **安全**:不把 API key / PII 写进文件。
7. 写完即停。**不** commit / push(留给用户)。

## 5. resume 流程(`/handoff resume`)

1. 读 `handoff.md`。
2. **对照核实**:跑 `git status` / `git log`,比对文件里写的分支 / 最新 commit / 脏文件是否与现实一致。
3. 若不一致,明确指出"handoff 已过时 / 可能是别的窗口写的旧快照"。
4. 向用户复述当前状态 + Next Steps,**然后停下等指示**,不自行开工。

## 6. 已定默认值

- 命令名:`/handoff`(默认 save)、`/handoff resume`。
- frontmatter:`disable-model-invocation: true`(仅用户主动调用)。
- 位置:repo 根 `handoff.md`,被 git 跟踪(不 gitignore)。
- 提交策略 = 策略层面"被跟踪";skill 本身不自动 commit。
- 并发两窗口:v1 单文件、last-writer-wins;靠 `Current Truth` 头在 resume 时核实是否过时。
- 安装位置:用户级 `~/.claude/skills/handoff/SKILL.md`。

## 7. 开放/待定(不阻塞 v1)

- handoff.md 团队场景下是否进 PR / 是否改 gitignore —— 单人 v1 不处理。
- Codex 侧 SKILL 镜像 —— 结构稳定后再做。
- hook 兜底 —— v2 视真实痛点再加。
