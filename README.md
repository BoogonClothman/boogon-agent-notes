# Agent Notes

A portable skill that teaches an AI coding agent to keep **durable decision records** — the why, the rejected alternatives, and the consequences — in any repository.

Most documentation says what the code does. An Agent Note says why it is that way, what lost, and what the decision cost. This repository packages that practice as a single Markdown skill, adapted from the Agent Notes system used by [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness).

[中文简介](#中文简介)

## What is in here

```text
.agents/skills/agent-notes/
├── SKILL.md                 the procedure an agent follows
├── references/
│   ├── contract.md          the project-facing rules; copy to .agents/notes/README.md
│   └── calibration.md       worked delete/keep/archive examples
└── templates/
    ├── note-proposed.md
    ├── note-implemented.md
    └── note-rejected.md
docs/
└── agent-notes-research-report.md   background on the DeepSeek Harness system (Chinese)
```

`SKILL.md` is the entry point. The references and templates are loaded only when needed.

## The idea in 30 seconds

- **One note, one decision.** Two independent decisions are two notes that link to each other.
- **The path is the metadata.** A note lives at `{lifecycle}/{class}/yyyy-mm-dd-topic.md`. Moving it between lifecycle folders *is* the status change; there is no index to maintain.
- **Six fixed classes.** `feature`, `bug-fix`, `simplification`, `architecture`, `process`, `testing`. The set is part of the contract: no project configuration, no seventh folder, no `refactor` class.
- **Alternatives are mandatory.** Every note records what lost and why. Alternatives are recorded, never invented.
- **Archive is frozen.** Implemented notes that stop guiding future work move to `archived/{class}/` and are sealed by an append-only SHA-256 manifest. After sealing, they are never edited, translated, moved, or deleted.
- **Machines check structure; humans judge meaning.** The contract specifies mechanical checks for layout, format, links, pairs, and seals. Whether a rationale is worth keeping is a judgment call, supported by the calibration examples.

## Install

### Project-level (pin the version with the repository)

```powershell
New-Item -ItemType Directory -Force ".agents\skills" | Out-Null
Copy-Item -Recurse "<path-to-this-repo>\.agents\skills\agent-notes" ".agents\skills\"
```

```bash
mkdir -p .agents/skills
cp -R <path-to-this-repo>/.agents/skills/agent-notes .agents/skills/
```

Commit `.agents/skills/agent-notes/`. A project-level copy takes precedence over a user-level copy.

### User-level (all repositories)

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
Copy-Item -Recurse "<path-to-this-repo>\.agents\skills\agent-notes" "$HOME\.agents\skills\"
```

```bash
mkdir -p ~/.agents/skills
cp -R <path-to-this-repo>/.agents/skills/agent-notes ~/.agents/skills/
```

DeepSeek Harness also discovers `~/.dsh/skills/` and `<repo>/.dsh/skills/`; either location works. `.agents/skills/` is the portable convention and is also shared by other agent harnesses.

### From GitHub

```bash
git clone https://github.com/BoogonClothman/boogon-agent-notes.git
cp -R boogon-agent-notes/.agents/skills/agent-notes ~/.agents/skills/
```

### Verify

The skill appears as `agent-notes` in a skill-aware agent's catalog. Invoke it by name (`/agent-notes` where supported), or ask:

> Use the agent-notes skill to record the decision behind this change.

If it does not appear, check that the directory is named `agent-notes`, that `SKILL.md` has non-empty `name` and `description` frontmatter matching the directory, and that you copied `references/` and `templates/` too — the skill needs them for adoption.

## Adopt it in a repository

With the skill installed, ask the agent to adopt Agent Notes. It will:

1. copy `references/contract.md` to `.agents/notes/README.md`;
2. create `proposed/`, `implemented/`, and `rejected/`, each with the six fixed classes, plus `archived/`;
3. add the directory instructions and an empty `archived/manifest.json`;
4. record the adoption itself as the first `process` note.

After that, for every non-trivial change the agent decides whether a note is warranted, sweeps for an existing owner before drafting, and writes at most one note in the right lifecycle and class.

## What it is not

- Not a documentation generator, changelog, or task tracker.
- Not a CLI or a runtime dependency. It is a Markdown procedure with no scripts.
- Not a semantic classifier. The mechanical rules are specified; deciding whether a note is worth keeping remains a review judgment.

The contract describes the checks a project should enforce. This repository does not bundle a checker: enforcement belongs to the adopting project, which can wire the checks into its CI.

## Design notes

- **Procedure, not scaffolding.** The skill is an operating procedure: what to do, in what order, when to stop, and what to report. It is not a collection of templates with no judgment.
- **Hardcoded classes.** The six classes are part of the contract. A seventh class is a contract revision, not a folder, a setting, or a per-project convention.
- **No centralized index.** The lifecycle/class tree is the inventory; adding `INDEX.md` is a contract violation.
- **Frozen history.** The archive uses an append-only manifest so a later cleanup cannot silently rewrite the record.
- **Progressive disclosure.** `SKILL.md` carries the procedure; the contract, calibration examples, and templates are read when needed.

## Background

The [research report](docs/agent-notes-research-report.md) documents the DeepSeek Harness implementation this skill was adapted from: its layout, gates, lifecycle rules, archive seal, and the decisions behind them. It is background, not the contract.

## License and attribution

MIT. See [LICENSE](LICENSE) and [NOTICE](NOTICE).

The Agent Notes system, its vocabulary, and the calibration examples are adapted from [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness), Copyright (c) 2026 DeepSeek, MIT.

## 中文简介

这是一份可移植的 **Agent Notes** skill：让 AI agent 在任意仓库里记录“为什么这样做、放弃了什么、代价是什么”，而不是把决策理由散落在代码、提交记录和聊天里。

- **结构**：`{lifecycle}/{class}/yyyy-mm-dd-topic.md`，生命周期为 `proposed` / `implemented` / `rejected`，类别固定为 `feature`、`bug-fix`、`simplification`、`architecture`、`process`、`testing` 六种。
- **安装**：把 `.agents/skills/agent-notes/` 复制到目标项目的 `.agents/skills/`（随仓库锁版本），或用户级的 `~/.agents/skills/`（所有项目可用）。
- **开始**：在目标仓库里让 agent “用 agent-notes skill 引入 Agent Notes 机制”；它会建立 `.agents/notes/`，并把“引入机制”本身记成第一篇笔记。
- **边界**：skill 只提供流程和契约，不附带检查器；机械检查由采用它的项目接进 CI。
- **来源**：改编自 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) 的 Agent Notes 体系（MIT）。
