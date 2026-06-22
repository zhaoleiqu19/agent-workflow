# workspace-init Skill 设计 spec

- 日期: 2026-06-22
- 状态: 待用户审阅
- 所属 repo: `~/projects/agent-workflow`
- 本 spec 范围: 两个工作流 skill 中的**第二个**(workspace-init)。第一个(handoff)已完成,见 `2026-06-22-handoff-skill-design.md`。

## 1. 目标与定位

为单人多工具开发者(重度 Claude Code + Codex)提供"开新项目 / 接手他人 repo"时的一键工作流脚手架。
用户用几句话描述一个 repo,`workspace-init` 在当前目录搭出一套默认结构:以 **AGENTS.md 为唯一真源**的约定文件、指向它的 CLAUDE.md、空的 `handoff.md` 桩、最小 README、按技术栈的 `.gitignore`,以及 `docs/superpowers/{specs,plans}` 工作流目录;并在 greenfield 场景下完成 `git init` 与(可选)`gh repo create` + push。

与另外两者的分工:
- **handoff** = 易变在途状态;只读 git、只写 `handoff.md`,不 commit/push/建仓。
- **memory** = 慢变事实;本 skill 不写 memory。
- **workspace-init** = 一次性脚手架;**会** `git init` / commit / `gh repo create` / push(这正是当初从 handoff 划分出来留给它的职责)。

## 2. 范围(in / out)

In(本 skill 做):
- 在**当前目录**就地 scaffold 一套完整骨架(见 §4)。
- **交互问答**收集项目特定信息后填入 AGENTS.md。
- **两种模式自动判别**(greenfield / adopt,见 §3)。
- greenfield 下:`git init` + 初始 scaffold commit +(可选)`gh repo create` + push。
- `.gitignore` 从 `github/gitignore` 按栈拉取,失败回退内置最小版。
- 幂等:绝不覆盖已存在文件,跳过并报告。

Out(本 skill **不**做):
- 不写 memory。
- adopt 模式下不 `git init`、不碰 remote、不 `gh repo create`/push、不 commit(见 §3)。
- 不生成大量 skill(不做 pedrohcgs 式 52-skill 堆叠);保持最小骨架。
- 不维护本地 `.gitignore` 全量模板(改为运行时拉取 + 最小回退)。
- v1 只做 Claude Code 的 `SKILL.md`;Codex 镜像以后再说(产出物本身是通用文件)。

## 3. 两种模式(自动判别)

运行时先探测当前目录:

| 探测到 | 模式 | 行为概要 |
|---|---|---|
| 非 git 仓库(空/新目录) | **greenfield** | 全套:写文件 + `git init` + 初始 commit +(可选)`gh repo create` + push |
| 已是 git 仓库(尤其有 remote) | **adopt(叠加)** | 只补缺失的工作流文件;**绝不** `git init`、**绝不**碰 remote、**绝不** `gh repo create`/push;改动**不 commit**,留给用户审完自行提交到其 fork/分支 |

**判别依据**:当前目录是否已是 git 工作树(`git rev-parse --is-inside-work-tree`)。

adopt 模式细则(场景:clone 别人的 repo 到本地,想叠加自己的工作流):
- 只新增缺失的:`handoff.md`(空桩)、`docs/superpowers/{specs,plans}`。
- **不动**对方已有的 `AGENTS.md` / `CLAUDE.md` / `README.md` / `.gitignore`。
- 仅当"已有 `AGENTS.md` 但缺 `CLAUDE.md`"时,补一行 `@AGENTS.md` 的 `CLAUDE.md`。
- 两者都没有时**不**强加我们的 `AGENTS.md`(生成 AGENTS.md 内容是 greenfield 的交互行为,不属于叠加)。
- 改动一律不 commit、不 push,完成后报告"已叠加哪些文件,请自行 review 并提交"。

## 4. 产出物(greenfield 完整骨架)

幂等:逐个写,已存在的一律跳过并报告;绝不覆盖。

```
AGENTS.md                         # 唯一真源,交互问答后填入(§5)
CLAUDE.md                         # 只有一行: @AGENTS.md
handoff.md                        # 空的结构桩(handoff 的固定标题骨架,值留空)
README.md                         # 极简: 标题 + 一句话描述 + 常用命令
.gitignore                        # 从 github/gitignore 按栈拉,失败回退内置最小版
docs/superpowers/specs/.gitkeep
docs/superpowers/plans/.gitkeep   # 空目录靠 .gitkeep 让 git 跟踪
```

### AGENTS.md 内容结构(survey 驱动,短,目标 <~60 行)

依据 2026 CLAUDE.md/AGENTS.md 最佳实践:短、命令优先、带理由、不重复 linter。

1. 一句话项目描述
2. Tech Stack(语言 / 框架 / 版本)
3. **Commands**(build / test / lint / run 的**精确**调用 —— ROI 最高段)
4. Architecture(3–5 个关键目录,指向文件而非长篇描述)
5. Conventions(几条带"为什么"的规则;不写 linter 能确定性强制的风格)
6. Current Work → 一行指向 `handoff.md`

### handoff.md 桩

写入 handoff skill 定义的固定英文标题骨架(`Current Truth` / `User Goal` / … / `For the Next Session`),值留空,使新 repo 一开始 `/handoff resume` 就有结构可读。

## 5. 运行流程

1. **模式判别 + 幂等扫描**:判 greenfield / adopt;列出 cwd 已存在的目标文件,后续跳过。
2. **交互问答**:项目描述、技术栈、build/test/lint/run 四个命令、仓库可见性(默认 private)、是否建 GitHub。
   - adopt 模式下问答收缩:不需要可见性 / 建仓(远程一律跳过)。
3. **写文件**:按模式写应写的文件,跳过已存在的并报告。
4. **.gitignore**:按栈从 `github/gitignore` 拉取;网络失败 → 内置最小版(greenfield 且文件不存在时)。
5. **git**:greenfield 且非 git → `git init` + 一次初始 scaffold commit;adopt → 跳过,不 commit。
6. **gh**(仅 greenfield 且用户选了建仓):运行时检测 gh 是否安装 + 登录。
   - 有 → `gh repo create`(private 默认)+ push。
   - 无 → 打印安装 / `gh auth login` 指引,**跳过远程**,留一个本地就绪的 repo。
7. **报告**:建了什么 / 跳过了什么 / 下一步(如安装 gh、填 AGENTS.md 占位、`/handoff` 用法)。

## 6. 已定默认值

- 命令名:`/workspace-init`;`disable-model-invocation: true`(仅用户主动调用)。
- 位置:在**当前目录**就地 scaffold(新或空皆可);不自动新建子目录。
- 双文件策略:**AGENTS.md 为源**,CLAUDE.md 仅一行 `@AGENTS.md`(覆盖早先 handoff 笔记里"两份相同副本"的设定)。
- 填充方式:**交互问答**后填入 AGENTS.md。
- 模板存放:**全部内联**在 `SKILL.md`(与 handoff 一致;模板不多)。
- `.gitignore` 来源:运行时从 `github/gitignore` 拉取,失败回退内置最小版。
- GitHub 建仓:保留 `gh repo create`,**运行时检测**,缺失则优雅降级为本地 scaffold。
- commit 策略:greenfield 自动做一次初始 commit;adopt 不 commit。
- README:极简(标题 + 一句话 + 常用命令)。
- 安装位置:用户级符号链接 `~/.claude/skills/workspace-init` → 本 repo,新会话生效。

## 7. 安全与边界

- **绝不覆盖**已存在文件(幂等跳过 + 报告)。
- **绝不写** API key / secret / PII。
- **绝不推到不属于你的 remote**:adopt 模式整体跳过远程操作;greenfield 只对自己新建的仓 push。
- 所有外部依赖统一优雅降级:gh 缺失、`github/gitignore` 拉取失败,都退回本地 scaffold,不中断。

## 8. 开放 / 待定(不阻塞 v1)

- Codex 侧 SKILL 镜像 —— 结构稳定后再做(产出物通用)。
- 是否支持"新建子目录"形态(给名字参数)—— v1 先只做当前目录。
- adopt 模式下两者都缺 AGENTS.md 时是否提供"顺手生成"选项 —— v1 不做,保持叠加最小。
- 团队/多人协作下 handoff.md、`.local.md` 的取舍 —— 单人 v1 不处理。
