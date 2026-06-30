# agent-debrief 重设计 spec:独立审计 + memory 闭环

- 日期: 2026-06-30
- 状态: 待用户审阅
- 所属 repo: `~/projects/agent-workflow`
- 取代关系: 在 `2026-06-26-agent-debrief-skill-design.md` 之上做架构级修订,不是新增 skill。原 spec 的目标与定位仍然成立,本 spec 改的是**执行机制**。

## 1. 背景:要解决的是架构缺陷,不是细节

现有 agent-debrief 让**做出决策的那个 agent、在同一段上下文里**审计自己。由此产生三个根本问题:

1. **审计者 = 被审计者(利益冲突)**。同一个 agent 对自己的决策有动机性推理,倾向事后构造合理化;看不到自己当时的盲区;"Decisions Under Audit" 的结构化模板反而让自我辩护显得更可信。
2. **产出无闭环**。Control Points / Transferable Lessons / Knowledge Gaps 产出后没有任何机制保证被消化或沉淀。同样的教训在每次 debrief 里被重新发现,从不进入长期记忆;下一次任务,agent 不知道上一次说过什么。
3. **"事实 vs 推断"靠自觉**。该划分是质量基石,却完全依赖 agent 自律标注,无任何强制——而 agent 恰恰最容易在"为什么"上把推断说成事实,尤其在审计自己时。

本次重设计针对前两点(并顺带结构性地解决第三点),通过两根支柱:**独立审计员**与**debrief→memory 沉淀数据流**。

## 2. 已确认的设计决策

| # | 决策点 | 选定方案 |
|---|---|---|
| D1 | 可移植性(独立 subagent 依赖 Claude Code 能力) | **可移植 + 优雅降级**:有 subagent 能力就派独立审计员;Codex 无此能力则降级为"禁用记忆的自审",并诚实声明降级 |
| D2 | 审计员收到的输入(独立性 vs 可用性) | **证据 + 中立决策清单**:git 证据 + 最终产物 + 只含"做了什么/谁拍板"的事实清单;**不给**理由、自我辩护、聊天记录 |
| D3 | memory 写入权限 | **起草 + 确认后写**:agent 起草完整格式的 memory 候选条目,问用户 [y/n/改],确认后才写文件并更新 MEMORY.md |
| D4 | 本轮改动范围 | **只改 agent-debrief**;handoff 保持现状("标记但不写 memory"),待本套机制跑顺后再复用 |

## 3. 三者关系的心智模型(指导本设计,本轮只落地 debrief 一侧)

区分 handoff / debrief / memory 的唯一干净的轴是**寿命**:

| | 寿命 | 给谁看 | 回答的问题 |
|---|---|---|---|
| handoff | 任务期间,任务完即弃 | 下一个 session | 现在干到哪、接下来干什么 |
| debrief | 一个时间点的快照 | 用户 | 刚才发生了什么、谁拍板、我该盯哪、学到什么 |
| memory | 永久,跨 session/项目 | 未来的 Claude | 有哪些长期成立的事实/偏好/教训 |

三者按一条**单向漏斗**流动,下游只存上游的蒸馏版、绝不复制:

```
原始事件 ──(干活中)──> handoff.md ──(任务结束反思)──> debrief ──(蒸馏长期项)──> memory
```

晋升判定口诀:
- 任务一结束就过期 → 不进 memory(顶多 handoff)
- 给用户"现在该动手"的时间点判断 → 留在 debrief(Control Points)
- 任何项目的未来 session 都该知道 → 起草成 memory 候选

## 4. 新流程:两阶段 + 闭环

```
[阶段0] 主 agent(有上下文)编"中立决策清单"
   - 每个决策点:做了什么 + 谁拍板 [user-chosen | agent-default]
   - 只写事实,不写为什么
        │
        ▼
[阶段1] 审计(独立性 = 有什么用什么)
   ├─ 有 subagent 能力:派全新 subagent 当审计员
   │     输入 = git证据 + 最终产物 + 决策清单(无理由、无聊天)
   │     输出 = 完整 debrief 主体
   └─ 无 subagent(Codex):主 agent 自审,但强制
         "只许从 git 证据 + 决策清单重新推导,禁止用'当时为什么'的记忆",
         并在输出顶部声明这是降级版自审
        │
        ▼
[阶段2] debrief 呈现给用户(默认聊天;显式要求才存 agent-debrief.md)
        │
        ▼
[阶段3] memory 晋升(闭环)
   - 把长期成立的教训/事实起草成完整 memory 候选条目(frontmatter + type + Why/How)
   - 问用户 [y/n/改];确认才写文件 + 更新 MEMORY.md
   - 用户否决则丢弃
```

**关键不变量:**
- 阶段0 的决策清单是隔离的钥匙——把"发生了什么"(主 agent 提供,带归属)与"判断好不好"(审计员独立做)切开。审计员拿不到自我辩护,故不会盖章。
- 可移植靠一个分支实现,两边都能跑,Codex 诚实声明降级。
- 阶段3 是新闭环,仅蒸馏长期成立项,且必须用户确认才落盘——不打破"agent 不静默写 memory"的精神,只是把"标记"升级成"起草+确认+写"。
- 职责归属:阶段1 的审计员(subagent)返回 debrief 后即退出;阶段2/3(呈现、与用户确认、写 memory)由**主 agent / 编排者**执行——它持有与用户对话和写入的能力,审计员只产出内容。

## 5. 审计员接口

**输入(阶段0 主 agent 打包):**
1. Git 证据:`git status` / `git log --oneline -10`;改动全集——未提交用 `git diff` (+ `--staged`),已提交用 `git diff <base>..HEAD` / `git show <commit>`。明确覆盖"已提交"的改动,不假设 `git diff` 是全集。
2. 最终产物:相关文件当前内容(审计依赖实现细节时才读)。
3. 中立决策清单:`"<决策>" — [user-chosen | agent-default]`,只含事实 + 归属。
4. **明确不给**:聊天记录、agent 的"为什么"、自我辩护。

**输出(审计员返回 = debrief 主体):**
- Flow:5-9 节点;允许并鼓励标注失败分支/回退(`Try A -(fail)-> Try B`)。
- What Changed:具体事实,不写理由。
- Decisions Under Audit:每条含 决策 / 证据 / 替代 / 权衡 / 风险弱假设 / **谁拍板**(新增字段,来自决策清单)。
- Problems & Fixes。
- Control Points:用户该 own/verify 的点。
- Transferable Lessons:阶段3 memory 候选来源。
- Knowledge Gaps。
- Next Questions:3-5 个。

**两条强制规则:**
- 事实/推断必须标注:git 证据 = 事实;"为什么/根因" = 推断,无直接证据即标 inference。审计员因本就拿不到理由,被结构性地逼着这么做(解决问题 #3)。
- 审计员不替 agent 辩护:任务是暴露判断、支撑强弱、可质疑处。

**小任务降级:** 单步小任务只回 Flow / What Changed / Control Points / Next Questions,其余段无内容则省略,不留空标题。

## 6. 去重边界(debrief 拥有什么 / 引用什么)

| 内容 | debrief 怎么处理 |
|---|---|
| 决策审计 / 控制点 / 知识缺口 / 下一步问题 | 独占,完整写 |
| 改动明细 | 引用 commit / `文件:行号`,不重贴大段 diff |
| 当前分支 / 脏文件 / 下一步执行状态 | 不写——handoff 的地盘 |
| 任务内用完即弃的坑 | 可提一句,归属仍是 handoff |
| 长期教训 / 用户偏好 / repo级的坑 | 产出候选 → 阶段3 晋升 memory,debrief 自身不长期保存 |

**保存到 agent-debrief.md 时**(仅用户显式要求):压缩版,追加不覆盖;教训若已晋升 memory,只写一句"已提升至 memory: [[slug]]"且不重复内容。

## 7. 范围(in / out)

In:
- 阶段0 决策清单编制;阶段1 独立审计(双路径);阶段2 呈现;阶段3 memory 起草+确认+写。
- 可移植分支与 Codex 降级声明。
- 吸收三个细节修复:已提交改动取证、小任务最小集、Flow 允许失败分支。

Out:
- 不动 handoff(下一轮再复用机制)。
- 不改 memory 的存储格式或目录(沿用现有 Claude Code memory 约定)。
- 不自动 commit / push;不静默写 memory;不存 secret/PII。
- 不引入过程内 checkpoint 拦截机制(那是另一个更大的项目,本轮不做)。

## 8. 已知遗留 / 非目标

- **事后 vs 过程中**:本设计仍是事后审计,治标。真正的"过程内拦截"(关键岔路口暂停确认)是独立的更大项目,本轮明确不做,仅记录为已知局限。
- **决策清单的残余偏见**:阶段0 由主 agent 编清单,它可能漏列某些决策。缓解:清单只写事实+归属、不写理由,把"判断"完全交给审计员;偏见限于"列没列",不污染"评得好不好"。
- **降级路径的真实独立性有限**:Codex 自审靠"禁用记忆"的纪律,非真正隔离;在该平台诚实声明,不假装等同。

## 9. 可移植性与验证约束

- frontmatter 仅用通用 `name` / `description`;平台差异(subagent 派发)写在正文的条件分支里,不依赖 Claude-only frontmatter 字段。
- SKILL.md 必须保持 **ASCII-only**:仓库的 `quick_validate.py`(Python 3.6 默认 ASCII)会在非 ASCII 字节上崩溃。破折号用 `-`,箭头用 `->`。
- 验证:`python3 ~/.codex-chatgpt/skills/.system/skill-creator/scripts/quick_validate.py skills/agent-debrief` 须通过。
