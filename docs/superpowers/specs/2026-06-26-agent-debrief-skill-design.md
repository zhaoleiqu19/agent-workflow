# agent-debrief Skill 设计 spec

- 日期: 2026-06-26
- 状态: 待用户审阅
- 所属 repo: `~/projects/agent-workflow`
- 本 spec 范围: 新增一个用于 agent 协作后强审计复盘的 skill。它不是 handoff 的替代品,而是补上"我自己留下了什么"这一层。

## 1. 目标与定位

`agent-debrief` 帮助重度使用 Codex / Claude Code 等 agent 的用户,在一次任务结束或阶段性节点后,把"AI 帮我做完了"转化成"我理解了发生什么、为什么这么做、哪里有风险、我下次应该掌控什么"。

它解决的核心困境是:工作和学习越来越多交给 agent 完成后,用户容易有 out of control 的感觉。这个 skill 不只做总结,而是做**强审计复盘**:总结流程,提炼关键决策,审查证据与替代方案,指出风险和用户掌控点,最后沉淀可迁移经验与待补知识。

与已有 skill 的分工:
- **handoff** = 当前任务的易变续接状态,面向"下次怎么接着干"。
- **workspace-init** = 新 repo / adopt repo 的一次性工作流脚手架。
- **agent-debrief** = agent 协作后的学习与掌控复盘,面向"这次我自己留下了什么"。

## 2. 范围(in / out)

In(本 skill 做):
- 在任务结束时做默认复盘(end debrief)。
- 在阶段性节点做 checkpoint 复盘,例如设计结束、实现结束、debug 结束。
- 输出清晰的流程总结,可用文字流程图表示。
- 审计关键决策:决策、证据、替代方案、取舍、风险/弱假设。
- 提炼问题与解决路径、用户掌控点、可迁移经验、待补知识、后续追问。
- 默认先在对话中输出。
- 当用户要求"保存" / "写入日志"时,追加到 repo 根的单一日志文件 `agent-debrief.md`。

Out(本 skill 不做):
- 不替代 `handoff.md`,不记录下一会话续接所需的完整在途状态。
- 不写 memory;若发现长期原则,只建议用户考虑提升。
- 不自动 commit / push。
- 不把完整聊天记录转成流水账。
- 不把未经验证的推断包装成事实。
- 不保存 API key、token、secret、个人敏感信息。

## 3. 使用模式

`agent-debrief` 是用户主动调用型 skill。为了同时适配 Codex 和 Claude Code,frontmatter 只使用通用的 `name` 和 `description`;"用户主动调用"约束写入 description 和正文,不依赖 Claude-only 的 `disable-model-invocation` 字段。

支持两种模式:

| 模式 | 触发 | 重点 |
|---|---|---|
| end debrief(默认) | 任务基本完成后 | 最终发生了什么、哪些判断可迁移、哪些风险残留 |
| checkpoint debrief | 阶段性节点 | 当前确认了什么、下一阶段用户需要盯住什么、哪些决策尚未稳定 |

如果用户只说 `/agent-debrief`,默认使用 end debrief。
如果用户说"阶段性复盘"、"checkpoint"、"先复盘到这里",使用 checkpoint debrief。
如果用户说"保存"、"写入日志"、"append",在对话输出后追加到 `agent-debrief.md`。

## 4. 标准输出结构

标准输出采用固定结构,默认是中等深度。用户可要求"简短复盘"或"深度复盘"调整篇幅,但核心栏目不变。

### 4.1 Flow

用 5-9 个节点描述本轮协作怎么推进。避免聊天流水账,只保留关键推进路径。

示例:

```text
目标 -> 探索仓库 -> 确认方案 -> 实现修改 -> 遇到测试失败 -> 修正假设 -> 验证 -> 结果
```

### 4.2 What Changed

列出实际完成的事实,关联关键文件/模块。这里只讲事实,不展开解释。

### 4.3 Decisions Under Audit

这是核心部分。每个关键决策使用相同审计格式:

```text
决策:
证据:
替代方案:
取舍:
风险/弱假设:
```

目的不是证明 AI 做得对,而是让用户看清 agent 在哪些地方做了判断、这些判断靠什么支撑、哪里仍需要用户确认。

### 4.4 Problems & Fixes

记录遇到的问题、根因判断、解决办法和残留风险。若根因只是推断,明确标注。

### 4.5 Control Points

指出用户最应该掌控、追问或复核的点。例如:
- 架构边界是否符合长期方向。
- 测试覆盖是否覆盖真实风险。
- 某个默认选择是否只是沿用现有模式。
- 哪些地方 agent 没有充分证据。
- 哪些决定是用户明确拍板,哪些是 agent 默认推进。

### 4.6 Transferable Lessons

提炼下次能迁移的判断原则,不要只复述本项目细节。

### 4.7 Knowledge Gaps

指出如果用户想降低 out of control 感,应该补哪些具体知识。要求具体到概念或技术边界,避免"多学前端"这类空话。

### 4.8 Next Questions

给出 3-5 个用户可以继续追问 agent 的问题,帮助用户从被动接受结果转为主动理解过程。

## 5. 执行流程

1. **确认模式。** 判断是 end debrief 还是 checkpoint debrief;判断是否需要写入日志。
2. **先接地。** 如果在 repo 中运行,先读 `git status`;必要时读 `git diff`、`git log --oneline -5` 和相关文件。不能只凭聊天记忆复盘。
3. **抽取事实。** 从对话、git 状态、文件改动、测试结果中提取事实。
4. **区分事实与推断。** 文件改动、测试结果、用户明确选择是事实;架构意图、原因解释若没有直接证据,标成推断。
5. **做强审计。** 审查决策证据、替代方案、取舍、风险、弱假设和用户失控点。
6. **输出对话复盘。** 按标准结构生成复盘,篇幅根据用户要求调整。
7. **可选保存。** 如果用户要求保存,追加一条压缩版到 `agent-debrief.md`,不覆盖旧内容。
8. **停止。** 不自动 commit / push;如果发现长期原则,只建议用户考虑提升到 memory。

## 6. agent-debrief.md 日志格式

日志文件位置:repo 根 `agent-debrief.md`。

保存策略:追加,不覆盖。每条记录带日期、任务标题、模式和当前 git 状态摘要。保存版默认比对话版更压缩,防止日志膨胀。

建议结构:

```markdown
## 2026-06-26 — <task title>

- Mode: end | checkpoint
- Git: <branch>, <latest commit>, <dirty summary>

### Flow
<5-9 node flow>

### Decisions Under Audit
- <decision> — evidence: <...>; tradeoff: <...>; risk: <...>

### Problems & Fixes
- <problem> -> <fix> -> <residual risk>

### Control Points
- <what user should verify / own>

### Transferable Lessons
- <lesson>

### Knowledge Gaps
- <gap>

### Next Questions
1. <question>
```

## 7. 质量规则

- **不写流水账。** 复盘是结构化提炼,不是 transcript。
- **不装懂。** 未验证的原因解释必须标为推断。
- **强审计。** 主动指出替代方案、风险、弱假设和用户可能失去掌控的地方。
- **事实接地。** repo 中运行时优先以 git 和文件为事实来源。
- **证据优先。** 测试是否跑过、是否失败、是否跳过,必须说清楚。
- **日志克制。** 保存版默认压缩,避免长期日志不可读。
- **安全。** 不写 secret、token、API key、PII。
- **边界清楚。** 不自动 commit / push;不替用户写 memory。

## 8. 已定默认值

- skill 名称:暂定 `agent-debrief`。
- 默认模式:end debrief。
- 支持 checkpoint debrief。
- 默认输出位置:对话。
- 可选保存位置:repo 根 `agent-debrief.md`。
- 保存策略:追加,不覆盖。
- 默认深度:标准版。
- 审计姿态:强审计。
- frontmatter:只使用通用 `name` 和 `description`;用户主动调用约束写入 description/正文。

## 9. 开放 / 待定

- 最终命令名是否使用 `/agent-debrief`,还是更短的 `/debrief`。
- `agent-debrief.md` 是否应被 git 跟踪。v1 建议跟踪,因为这是用户想长期沉淀的学习账本;如果未来发现隐私压力大,可改为本地文件。
- 是否需要给"深度复盘"提供更严格的字数上限。
- 是否未来支持把高价值条目整理成 memory 候选清单。v1 只提示,不自动写。
