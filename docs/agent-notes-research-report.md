# deepseek-harness Agent Notes 体系研究报告

> 调研对象:[deepseek-ai/deepseek-harness](https://github.com/deepseek-ai/deepseek-harness)(公开仓库,commit [`22ddee37`](https://github.com/deepseek-ai/deepseek-harness/commit/22ddee3792402c3e70ec5f86a63c3e3e895a2f60))
> 调研范围:`.agents/notes/` 全部规则与脚本、`.agents/skills/` 结构及其与 Notes 的协作关系
> 报告日期:2026-10-26
> 许可说明:DeepSeek Harness 以 MIT 许可开源;本报告对其公开文件做摘要与引用,权威内容以该仓库为准。

---

## 1. 执行摘要

deepseek-harness 的 **Agent Notes** 是一套"由 agent 撰写的 RFC/ADR(架构决策记录)体系":

- **一句话定位**:记录代码和普通文档承载不了的东西——*为什么这么决策、放弃了什么*。
- **核心设计**:双轴路径编码(生命周期 × 决策类别)+ 严格文件格式 + 全部机械化门禁执法 + 冻结归档区。
- **规模**:活跃区 375 篇(proposed 45 / implemented 322 / rejected 8),冻结归档 926 篇(2778 个封印工件),时间跨度 2026-06-11 → 2026-10-23。
- **配套**:21 个仓库级 Skills(`.agents/skills/`),其中一个 `dsh-archive-agent-notes` 专门负责 Notes 的归档判定工作流。

这套体系最值得借鉴的一点:**所有规则都有对应的可执行门禁(verify-* 脚本),没有任何一条规则依赖"自觉遵守"**;而语义判断(如"这篇笔记还值不值得保留")则明确交给带校准样例的 Skill,而不是试图写成代码。

---

## 2. 设计哲学(从其制度自身的历史笔记中提炼)

Agent Notes 制度本身也用 Agent Notes 记录决策(见 §12 演进史),从中可提炼出贯穿始终的原则:

1. **机械规则交给门禁,语义判断交给评审/Skill。** 例如"非平凡变更必须写笔记"这条规则明确拒绝了 CI diff 分类器——"机械检查无法可靠判断语义是否平凡,只会制造误报和表面合规"。
2. **路径即元数据(path-encoding)。** 状态和类别编码在路径里,标签与存储是同一个东西,不存在"文件头说 A、文件夹在 B"的失同步问题。
3. **封闭集合(closed set)。** 生命周期、类别、格式词汇都是枚举白名单,新增一个类别必须同时改代码常量和文档,`mkdir` 一个新文件夹过不了门禁。
4. **拒绝中心化索引。** 明令禁止 `INDEX.md`:它复制路径元数据、制造无关变更的合并热点,树浏览 + 仓库搜索已覆盖发现需求。
5. **记录"打败了什么"是强制项。** 每篇笔记必须有 `## Alternatives considered`,防止后人重新翻案——"没有记录战胜了谁的决策,会招致 Agent Notes 本要防止的翻案"。
6. **历史即快照,封印后不可变。** 归档区用 SHA-256 manifest 封印,连"顺手修个链接"都不允许,例外必须以精确哈希写进代码。

---

## 3. 目录结构与双轴编码

```
.agents/
├── notes/                      # Agent Notes 本体
│   ├── README.md               # 制度总契约(布局/分类/格式/归档/写作判据)
│   ├── README.zh.md            # 中文对照 + .i18n.yaml sidecar
│   ├── AGENTS.md               # 目录级 agent 指令(新笔记必做 supersession 检查)
│   ├── proposed/{class}/       # 提案:实现前(或仅部分实现)
│   ├── implemented/{class}/    # 已落地:必须随代码同步更新事实
│   ├── rejected/{class}/       # 已否决:仅在能防止"诱人错误"时保留
│   └── archived/{class}/       # 冻结归档:只有 implemented 能进
└── skills/                     # 仓库级 agent Skills(见 §11)
```

### 3.1 两个轴

路径格式:**`{lifecycle}/{class}/yyyy-mm-dd-topic-title.md`**

- **生命周期(顶层目录)** = 笔记的状态,随状态移动:
  - `proposed/` — 实现前的提案评审
  - `implemented/` — 决策已发布;**必须与实际发布内容保持同步**(代码改了文件路径/包名/默认值,同一次变更里更新笔记;只更新事实,不重写决策)
  - `rejected/` — 提案被否决;仅当否决理由仍能防止一个"诱人且有意义的错误"时保留,否则整组三件套删除
- **类别(二级目录)** = 决策种类,封闭六类(定义在 `scripts/agent-note-tree.ts` 的 `AGENT_NOTE_CLASSES` 常量):

| 类别 | 覆盖范围 |
|---|---|
| `feature` | 面向用户或模型的新能力 |
| `bug-fix` | 修正缺陷或补上 postmortem 暴露的缺口 |
| `simplification` | 删除代码/行为/表面积而不新增能力 |
| `architecture` | 关于**发布源码**的结构决策——包之间关系、运行时词汇 |
| `process` | 代码**周围**的工具、政策、工作流——门禁、包管理器、vendoring |
| `testing` | 测试基础设施与策略 |

`architecture`/`process` 的分界线:architecture 是"我们发布的源码",process 是"周围的工具与流程"。**刻意不设 `refactor` 类**——它与 simplification 几乎完全重叠,而后者的判别式("可观察行为是否变化?")已经覆盖。

### 3.2 命名与交叉引用细节

- 文件名日期 = 话题**首次提出**的日期(git 历史可查);状态行不带任何日期,其余交给 git。
- 笔记间交叉引用**必须用相对 markdown 链接**(`[topic](../../implemented/architecture/2026-…-….md)`),禁止裸文本/数字引用——这样引用可被机械校验,且在生命周期目录间移动时存活。
- **禁止 `INDEX.md`**:树本身就是清单,门禁直接报错("centralized Agent Note indexes are forbidden")。
- 归档区被 `.rgignore`(`/.agents/notes/archived/`)排除出默认 ripgrep 搜索——冻结的事实可能过期,不应混入活跃决策的发现路径;历史查询需显式指名归档目录。

### 3.3 当前分布(统计快照)

| 区域 | feature | bug-fix | simplification | architecture | process | testing | 合计 |
|---|---|---|---|---|---|---|---|
| proposed | 9 | 1 | 12 | 15 | 6 | 2 | **45** |
| implemented | 46 | 14 | 14 | 194 | 36 | 18 | **322** |
| rejected | 0 | 0 | 7 | 1 | 0 | 0 | **8** |
| archived | 307 | 182 | 92 | 185 | 121 | 39 | **926** |

archived 的 926 篇 × 3(英/中/sidecar)= manifest 中的 2778 个封印工件。implemented 中 architecture 占比最高(194/322),符合"结构性决策最需要长期权威"的预期。

---

## 4. 文件格式契约(机器可校验)

由 `pnpm run verify-agent-note-format`(`scripts/verify-agent-note-format.ts`,doc-sync 的一部分)强制。格式本身的取舍理由记录在归档笔记 `2026-07-05-uniform-agent-note-format` 中。

### 4.1 头部块

每个活跃笔记前三行必须**精确**为:

```markdown
# Agent Note: <title>

Status: <status>
```

第 2、4 行必须是空行;`Status:` 全文只允许出现一次(第 3 行)。取值三选一,且必须与所在生命周期目录一致(门禁交叉核对):

- `Status: proposed`
- `Status: implemented`
- `Status: rejected — <一行理由>`

状态行**不带日期、不带括注**:文件名承载首次提出日期,其余归 git;唯一带内容的是拒绝理由——因为"被否决笔记的结论正是读者来找的事实"。

### 4.2 各生命周期的正文骨架

正文第一节必须是 `## Problem`(动机,脱离解决方案也能独立成立)。之后按生命周期分:

**proposed/**
```markdown
## Problem
## Proposal          ← 可以合法使用将来时:计划、迁移步骤、开放问题
…自定义技术章节…
## Alternatives considered
## Acceptance criteria   ← 什么可观察状态算完成
## Risks                 ← 既能出什么错,也包括明知放弃了什么
```

**implemented/**
```markdown
## Problem
## Decision          ← 现在时描述发布后的现实
…自定义技术章节…
## Alternatives considered
## Consequences      ← 既记录代价,也记录收益
```

implemented 中**禁止**出现提案时代词汇:`## Proposal`、`## Plan`、`## Migration plan`、`## Acceptance criteria` 一律门禁拒绝(对应 docs/AGENTS.md "slop checklist" 中"决策已发布还用 should 叙事"条目)。`## Testing`/`## Deferred`/`## Related` 若陈述现在时事实则允许。

**rejected/** — 提案的冻结件:保留提案时的章节(包括 Acceptance criteria/Plan),判决只活在 `Status:` 行上;仅头部块、`## Problem` 开头、`## Proposal` 和 Alternatives 强制项适用。

### 4.3 `## Alternatives considered` —— 强制项

每篇笔记必须有:每个真实备选方案及落选原因,一个加粗引领段落/一个 `### Why not <X>?` 小节。**"只记录、不发明"**:2026-07-05 之前、备选无法从记录中重建的老笔记,以一行精确注释替代,门禁仅对老文件接受:

```markdown
<!-- agent-note-format: alternatives-not-recorded (pre-format Agent Note) -->
```

该注释对文件名日期 ≥ 2026-07-05 的笔记失效(门禁校验日期)。历史上曾有 39 个遗留格式债务标记(`XXX: legacy ADR/RFC body format`),格式统一时全部清零,且该标记被列为禁用防止回潮。

### 4.4 双语三件套

每篇笔记是**三件套**:`foo.md`(英文)+ `foo.zh.md`(中文)+ `foo.i18n.yaml`(一致性 sidecar)。

- 中文章节逐节镜像英文结构;机器校验的头部 token(`# Agent Note: `、`Status:` 行)在中文里保持英文原样。
- `.i18n.yaml` 按**章节 slug** 记录两侧内容哈希(`en:`/`zh:` 各一个),由 `pnpm run verify-translation-pairing --write <pair>` 重录。双语契约见 `docs/i18n/README.md`:两种语言地位平等(中文先写也合法),绑定条件是"必须说同一件事"。
- 同步方法是"对着另一侧的 diff 最小化补丁",**禁止整篇重译**;门禁精确报出失配章节。
- 格式门禁跳过 `.zh.md`(配对门禁管它),避免双重执法。

---

## 5. 生命周期流转规则

在生命周期目录间移动文件 = 同一次变更中更新 `Status:` 行并重新满足目标骨架,否则格式门禁直接失败:

- **proposed → implemented**:`## Proposal` 重写为现在时 `## Decision`;`## Acceptance criteria` 和 `## Risks` 折进 `## Consequences`(或现在时 `## Testing`/`## Verification` 陈述现在钉住行为的验证);丢弃计划,只写实际落地的东西。
- **proposed → rejected**:`Status:` 行加否决理由,冻结文件。
- **笔记永远不能被编辑成"另一个决策"**:要取代就写新笔记并互链;完全取代的旧笔记可按 §7 的合并规则删除。
- **编辑 implemented 笔记跟踪其决策的新居所是被要求的,不是被禁止的**;但这是"更新事实",不是"重写决策"——决策反转需要新笔记 + 互链。

---

## 6. 何时写一篇笔记(写作判据)

根 `AGENTS.md` 的常设命令:"**仅为持久的决策理由创建 Agent Notes;机械/局部编辑豁免**"。`.agents/notes/README.md` 细化:

- **同一 PR 内**新增或更新,仅当代码、测试、现有文档解释不了的**持久决策理由**存在时。
- 重大未来工作提案 → `proposed/`;已做决策 → `implemented/`。
- **已有笔记拥有该决策时,更新它即满足规则**,禁止造重复记录。
- 机械/局部编辑(含本地 UI 展示与交互改动)豁免。
- **每篇新笔记触发 supersession 检查**(`notes/AGENTS.md`):写之前在活跃树中搜索覆盖同一决策/机制/被否备选的旧笔记,分类完全/部分取代,同 PR 内归档合格的 implemented 三件套;部分取代保持互链活跃;不允许"以后再清点"。
- **完全取代的合并规则**:吸收方必须先保住旧笔记中每一条独特的理由、备选、后果、验证要求、点名的覆盖缺口,修好所有入链,同变更删除中文对照与 sidecar;部分取代不够格。
- **功能新增笔记 → 后续移除笔记**的合并有更高门槛:功能须从生产代码/配置/schema/持久与线上格式/迁移/兼容行为中完全消失,无文档将其呈现为可用,无测试将其作为受控行为 exercised。移除方须保住原始动机、为何不再成立、完全移除的备选、放弃的能力、重新引入条件、"确实消失"的验证。

---

## 7. 归档与冻结(本体系最工程化的部分)

制度依据:`implemented/process/2026-07-26-frozen-agent-note-archive`(未归档,仍在活跃区)+ `archived/AGENTS.md` + skill `dsh-archive-agent-notes`。

### 7.1 准入与分流(三级出口)

对每篇 implemented 笔记按**未来价值**语义判定(词数/年龄只是分诊辅助,绝非标准):

1. **直接删除**:仅描述小 UI 调整或纯机械变更的三件套整组删除 + 修入链。本地 bug fix/性能/新能力/实质性行为决策**不因实现小而够格删除**。
2. **保持活跃**:理由、备选、否定性保证、持久/线上语义、所有权边界、安全规则、重新引入条件仍可能指导未来变更——无论多老多短。
3. **归档**:决策已完结、不太可能指导未来工作、但历史理由仍有保存价值。

proposed **永不归档**(过时提案应显式否决);rejected 仅在仍是"诱人错误的护栏"时保留,否则整组三件套删除。

### 7.2 归档动作(唯一的允许改动集)

路径 `archived/{class}/…`(**刻意省略 `implemented` 段**——只有 implemented 能进归档,冗余)。归档变更仅允许:

1. 移动完整三件套;
2. 在两个语言文件的 `Status: implemented` 下方插入完全一致的 `Archived: YYYY-MM-DD` 行;
3. 机械重录 sidecar 哈希;
4. 修复/删除入链(归档笔记的**出链永不校验、永不修复**)。

标题标点、空行布局、语言切换器措辞不阻塞归档(保留原样,不要求"顺便修整齐"——那会重写历史)。

### 7.3 冻结与封印机制(`scripts/verify-archived-agent-notes.ts` + `scripts/archived-agent-notes.ts`)

封存后三件套**永久冻结**:不编辑、不重排、不翻译、不更新、不移动、不删除,也不作为当前行为的权威。实现手段:

- **封闭类别树**:归档根只允许 `AGENTS.md`、`manifest.json` 和六个已知类别目录;每个类别目录必须存在,且只含常规文件。
- **append-only manifest**(`archived/manifest.json`,version 1):`{path: "sha256:<hex>"}` 全量封印每个工件。`--write` 模式先证明每一个既有封印未变,再**仅追加**新工件哈希;普通模式对任何未封印工件报错。
- **基线比对**:CI 通过 `DSH_ARCHIVE_BASE_REF` 提供可信的变更前 commit,本地用 HEAD——封印变更相对可信基线只能扩展不能修改(PR CI 还会先取完整历史,防浅克隆绕过基线 manifest)。
- **sidecar 双哈希校验**:归档三件套的 `.i18n.yaml` 必须含两侧文件的**当前 Git blob 哈希**(sha1 + `blob <len>\0` 前缀),归档器重算比对;中英归档日期必须一致,且不得早于文件名日期。
- **例外写进代码**:全库仅一处授权例外——`feature/2026-08-10-durable-workflow-runs-in-chat` 的 Figma 链接移除,三条精确的 before→after SHA-256 哈希硬编码在 `archived-agent-notes.ts`。例外不允许任何其他内容变更、反转或删除。
- 门禁成员:`pnpm run verify-archived-agent-notes`(doc-sync,quick)。

---

## 8. 机械化执法全景

| 门禁脚本 | pnpm script | 执法内容 |
|---|---|---|
| `scripts/agent-note-tree.ts` | (共享模块) | 双轴封闭集合的唯一真源;结构遍历器,报未知生命周期/类别、坏深度、坏文件名、INDEX.md |
| `scripts/verify-agent-note-classification.ts` | `verify-agent-note-classification` | 结构一致性;另封禁旧家 `docs/rfc`、`docs/rfcs` 防止逃逸 |
| `scripts/verify-agent-note-format.ts` | `verify-agent-note-format` | 头部块、Status 语法与目录一致性、必备/禁用章节、Alternatives 强制与 grandfather 规则、旧债务标记禁用(代码围栏内豁免) |
| `scripts/verify-archived-agent-notes.ts` | `verify-archived-agent-notes` | 归档封闭树、三件套完整性、Archive 元数据、sidecar 哈希、append-only 封印 manifest |
| `scripts/verify-translation-pairing.ts` | `verify-translation-pairing` | 双语逐节哈希配对(活跃笔记);归档区被此门禁排除、由上一条专属校验 |
| `scripts/verify-md-links.ts` | `verify-md-links` | 本地链接目标存在(笔记间相对链接因此可机械校验) |
| `scripts/verify-doc-refs.ts` | `verify-doc-refs` | 扫描 `packages/**` 下 TS 注释中的 `.agents/notes/….md` 根相对引用,防移动后注释悬空 |

前三个 + 配对 + doc-refs 均为 `doc-sync` 聚合的成员,其中笔记相关门禁标记 `quick: true`(同时进入轻量 `doc-quick`/`test:docs` 聚合)。运行模式为 tsx ESM、verify-don't-generate、首个违规即非零退出。

**设计取舍**:分类正确性(语义)不设门禁——"机械检查无法可靠判定平凡性";被否理由是否"仍值得保留"也不设门禁——交给 `dsh-archive-agent-notes` 的语义审计。

---

## 9. Skills 体系及其与 Notes 的分工

### 9.1 仓库级 Skills 概览(`.agents/skills/`,21 个)

每个 skill = 目录 + `SKILL.md`(YAML frontmatter:`name` + `description`,description 以 "Use when…" 开头声明触发条件)+ 可选 `references/`、`templates/`、`scripts/` 子目录。规模 288(dsh-error-handling)→ 2889 词(dsh-pr-description)。

命名有两族:`dsh-*`(deepseek-harness 专有工作流)和无前缀的(`agent-experience`、`cordis-styling-nitpicking`、`record-browser-gif`)。其中 `agent-experience/SKILL.md` 是**指向产品预设的符号链接**(`packages/preset/agent-preset/skills/agent-experience/SKILL.md`)——仓库技能与出货技能同源。

按职能分组:

- **Notes/文档治理**:`dsh-archive-agent-notes`(Notes 归档工作流)、`dsh-doc`、`dsh-prose-standard`、`dsh-translate-docs`、`dsh-trim-cot-leakage`
- **评审/交付**:`dsh-code-review`、`dsh-pr-description`、`dsh-pre-push-checks`、`dsh-merging-stacked-prs`、`dsh-create-upgrade-guide`
- **工程专项**:`dsh-ci-test-reliability`、`dsh-error-handling`、`dsh-session-projection`、`dsh-speed-up-perf`、`dsh-find-simplifications`、`cordis-styling-nitpicking`
- **UI/素材**:`dsh-client-ui-ux`、`dsh-responsive-ui`、`dsh-official-plugin-artwork`、`record-browser-gif`
- **产品镜像**:`agent-experience`

### 9.2 Skills 与 Notes 的关系

- **分工**:Notes 记录*一次性的决策事实*(为什么、放弃了什么);Skills 携带*可复用的工作流程*(怎么做)。AGENTS.md 里两者高频并置,如"错误处理:遵循 `dsh-error-handling` skill"+"理由链接到某篇 note"。
- **单向引用为主**:10 个 SKILL.md 链向 notes 树;活跃笔记中仅 2 篇链回 skills。Notes 是权威层,skill 是操作层。
- **`dsh-archive-agent-notes` 是两者的桥**:它把 §7 的制度变成一份带**校准样例**的操作手册——给出具体的删/留/归档判例及词数(如:折叠侧栏控制条 533 词 → 删;"事件溯源会话" 248 词 → 留,因为是奠基性权威),并规定汇报格式(保留/删除/归档/边界案例逐一列出)。**词数在样例中出现恰恰是为了证明"大小不是标准"**。
- **skill 自身也有治理提案**:`proposed/process/2026-07-13-human-review-skill-maintenance` 用一篇完整的 proposed 笔记规划了 `dsh-code-review` 的周期性人工评审维护流程(双评审器、采纳证据契约、私有工具不入库)——说明"skill 如何防止 checklist 膨胀/把 merged 当采纳"这类问题也走 Notes 流程决策。

### 9.3 技术底座(产品如何消费 skills)

- `.claude/skills` 是指向 `../.agents/skills` 的符号链接;根 `CLAUDE.md` 内容即 "AGENTS.md"——同一份内容服务 Claude Code/Codex/pi 等多种 agent 入口。
- `.agents/skills/.gitignore` 忽略 `*/agents/openai.yaml`(Codex 侧调用元数据为本地产物);`verify-skill-invocation-metadata` 门禁在该文件存在时校验其与 SKILL.md frontmatter(如 `disable-model-invocation` 布尔)的调用策略一致性。
- 运行时由 `packages/skill` 三件套承载:
  - `@deepseek-ai/dsh-skill` — 注册表 `ctx.skills`:多来源合并目录、按名加载、去重可预测、`modelInvocable`/`userInvocable` 双调用策略;
  - `@deepseek-ai/dsh-skill-filesystem` — 本地发现,来源带 rank(数值小者胜):project `.dsh/skills` 100 → project `.agents/skills` 200 → custom 300 → user `~/.dsh/skills` 400 → user `~/.agents/skills` 500 → bundled 600;
  - `@deepseek-ai/dsh-tool-skill` — 模型侧目录 + `skill` 加载工具:首个请求前注入"名称+截断描述"目录,模型按需 `skill` 工具加载全文(**渐进披露**,控制上下文)。用户可用 `/name` 调用 user-invocable 技能,注入同一份指令。

---

## 10. 与其他制度的咬合

- **根 AGENTS.md**:常设命令链接到 `notes/README.md#when-to-write-one` 和归档政策;"文档预算"章节明确 Agent Notes **在人类文档预算/slop 清单的范围之外**(唯一例外:implemented 笔记禁 spec-speak)。
- **docs/AGENTS.md 放置表**:明确"rationale → Agent Notes;迁移计划/验收清单 → 不属于笔记(发布后)"。放置三分法:bug → postmortem,理由 → Agent Notes,操作步骤 → cookbook,类型定义 → subsystems,包契约 → README,常设命令 → 根 AGENTS.md(附理由链接),评审强制规则 → defensive-patterns。
- **PR 流程**:`dsh-pr-description`/`dsh-code-review`/`dsh-pre-push-checks` 三个 skill 把笔记接入 PR 生命周期——评审 skill 明确"PR 实现了某篇 proposed 笔记时,同一 diff 内移动并重写为现在时,然后对照实现核验路径/名称/机制";"与 Agent Note 意见分歧是设计讨论,不是自动否决"。
- **i18n 契约**:笔记是双语语料的一部分,但归档区被演进中的配对门禁排除,改由专属封印校验——"演进标准不得制造改写历史的压力"。

---

## 11. 制度演进时间线(它自己是怎么长出来的)

| 日期 | 笔记(均已归档,除注明) | 决策 |
|---|---|---|
| 2026-06-11 | (docs 时期) | "机械质量门禁优于散文规范"成为仓库立场基调 |
| 2026-06-20 | `archived/process/2026-06-20-agent-note-classification` | 加第二轴(类别)路径编码;封禁 `Classification:` 行(路径已携带,行会与文件夹打架)、封禁 `refactor` 类(与 simplification 重叠)、封禁生成索引;两个门禁入 doc-sync |
| 2026-07-05 | `archived/process/2026-07-05-uniform-agent-note-format` | 统一文件内格式;拒绝刚性全模板(大型设计笔记的 8–15 个自定义章节是承重内容)、拒绝"无 Status 行"(自描述文件 + 门禁对齐目录)、拒绝带日期状态、拒绝无门禁约定("19 个文件证明了纯约定在此repo能达到什么");grandfather 注释一次清零 39 个债务标记 |
| 2026-07-19 | `archived/process/2026-07-19-remove-generated-agent-note-index` | 删除生成索引:树即清单;拒绝"冲突后重新生成"(不减少无关分支改同一工件)和"手工索引"(共享文件争用 + 完整性错误) |
| 2026-07-19 | `archived/process/2026-07-19-require-agent-notes-for-non-trivial-changes` | 非平凡变更必写/更新笔记;拒绝"耐久-有争议-令人惊讶"三重门槛(主观到大变更能被当作显而易见)、拒绝 `superseded/` 生命周期、拒绝 CI diff 分类门禁;确立完全取代合并规则与移除型合并的高门槛 |
| 2026-07-26 | `implemented/process/2026-07-26-frozen-agent-note-archive`(**仍活跃**) | 冻结归档区:三级出口(删/留/归档)、`.rgignore` 排除、manifest 封印、专属校验器、`dsh-archive-agent-notes` 拥有语义分类;supersession 检查前移到写笔记时 |
| 2026-09-04 起 | — | 早期制度笔记陆续按制度本身归档封印(如上三篇 2026-09-04/09-30 归档)——**吃自己的狗粮** |
| 2026-08-10 | `archived/feature/2026-08-10-durable-workflow-runs-in-chat` | 成为全库唯一封印例外(Figma 链接移除,三条精确哈希) |

---

## 12. 设计亮点提炼(可借鉴清单)

对正在设计 agent 提案/决策制度的项目(如采用本 skill 的仓库),按可操作性排序:

1. **先写封闭枚举,再写文档。** 生命周期、类别、章节词汇都定义为代码常量 + 文档同改要求;门禁拒绝任何集合外目录。新增类别是"改两处的刻意动作",不是 `mkdir`。
2. **格式门禁要"verify-don't-generate",且快。** 笔记相关三个门禁都是 `quick: true`,秒级反馈,能进最轻量聚合,贡献者才不会绕。
3. **强制 `## Alternatives considered` + "只记录不发明"。** 这是笔记区别于"实现说明"的本质;配 grandfather 注释处理存量,而不是伪造理由。
4. **状态不带日期,日期只在文件名。** 文件名 = 首次提出;git = 其余。门禁能校验日期格式,永远校验不了日期真伪——所以干脆不让写。
5. **禁中心化索引,把路径当索引。** 避免所有分支争一个生成文件;引用一律相对链接使其可校验、可随移动存活。
6. **归档 ≠ 删除 ≠ 活跃,三级出口 + 冻结封印。** 冻结的关键工程手段是 append-only SHA-256 manifest + 可信基线比对 + 例外硬编码精确哈希;"归档笔记出链永不修复"这条细节避免了无底洞。
7. **语义判断交给带校准样例的 Skill,不写成代码。** `dsh-archive-agent-notes` 的判例清单(删 533 词 / 留 248 词)是防止"按长度/年龄机械清理"的最有效手段;并要求汇报边界案例。
8. **取代/合并要保底。** 合并前必须保住每条独特理由/备选/后果/验证/覆盖缺口并修链;部分取代必须双链并存;"用 git 历史当唯一副本"被显式禁止。
9. **每篇新笔记即触发 supersession 审计**,由最新鲜的作者完成,不允许"以后统一清理"。
10. **双语不是翻译层,是配对契约。** 两侧平等、逐节哈希 sidecar、`--write` 需点名确认、同步靠最小补丁而非整篇重译——把"翻译一致性"变成可合并的 yaml diff。

---

## 13. 关键文件索引

| 路径 | 作用 |
|---|---|
| `.agents/notes/README.md` (+ `.zh.md`) | 制度总契约 |
| `.agents/notes/AGENTS.md` | 新笔记必做 supersession 检查 |
| `.agents/notes/implemented/AGENTS.md` | 随发布同步事实,禁止重写决策 |
| `.agents/notes/archived/AGENTS.md` | 冻结区规则与唯一例外说明 |
| `scripts/agent-note-tree.ts` | 双轴封闭集合真源 + 结构遍历器 |
| `scripts/verify-agent-note-{classification,format}.ts` | 结构/格式门禁 |
| `scripts/verify-archived-agent-notes.ts` + `scripts/archived-agent-notes.ts` | 归档封印门禁与纯函数 |
| `.agents/notes/archived/manifest.json` | 2778 个工件的 SHA-256 封印 |
| `.agents/skills/dsh-archive-agent-notes/SKILL.md` | 归档语义判定工作流(含判例) |
| `docs/AGENTS.md` | 放置表 + slop 清单(与笔记制度咬合) |
| `docs/i18n/README.md` | 双语配对契约 |
| `packages/skill/{skill,skill-filesystem,tool-skill}/` | 技能运行时三件套 |
| `scripts/run-gates.ts` (`docSyncLeafGates`) | 门禁在 CI 聚合中的接线 |
