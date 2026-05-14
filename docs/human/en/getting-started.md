# Getting started

> **Goal**: in 30 minutes, an engineer unfamiliar with ADSD has the ADRs + findings + sub-agent dispatch discipline running on their own project.

## Who should read this

- You're managing a project with **multi-agent parallelism** (≥3 AI agents working concurrently)
- You want to avoid the multi-agent endemic ailments: sediment / drift / silent regression
- You already use Claude Code / Cursor / similar IDE-agent tools at a basic level
- You have a git project to apply this methodology to

If you're writing a single-agent small script, ADSD is overkill. Skip.

## 30-second overview

ADSD is the multi-agent working discipline distilled from 12 days of intensive Cobrust development (2026-04-30 → 2026-05-12, ~278 commits), codifying:

1. **Decision capture** — every cross-file decision becomes an ADR (Architecture Decision Record)
2. **Failure capture** — every "this broke / surprised / dead-ended" becomes a Finding (negative result)
3. **Dispatch discipline** — D0-D5 difficulty matrix + dev/test pair TDD protocol

Plus **bilingual docs mandate** + **wave + Tx atomic commits** + **F1-F30 anti-pattern catalogue** + **release-readiness pre-publish independent verification**. That's the full picture.

Detailed architecture: [`concept-map.md`](./concept-map.md)

## Three install paths

### Method 1 (recommended) — Claude Code plugin

```
/plugin marketplace add Cobrust-lang/agent-driven-development
/plugin install adsd@adsd
```

Once installed, when a prompt mentions "multi-agent dispatch / ADR drafting / F1-F30 failure modes" etc., Claude auto-activates the ADSD skill.

### Method 2 — Personal skill directory (fallback)

```sh
mkdir -p ~/.claude/skills
git clone --depth 1 https://github.com/Cobrust-lang/agent-driven-development.git /tmp/adsd-src
cp -r /tmp/adsd-src/plugins/adsd/skills/agent-driven-development ~/.claude/skills/
rm -rf /tmp/adsd-src
```

### Method 3 — Read-only (no install, just markdown)

Read [`plugins/adsd/skills/agent-driven-development/SKILL.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/SKILL.md) top-to-bottom (~30 min) for the full methodology. No install required to learn.

## First real use — 5 steps

Assume you have a project at `~/my-project/` and want to start with ADSD.

### Step 1: Create the project `CLAUDE.md` (constitution)

Write a ~30-line project constitution at `~/my-project/CLAUDE.md` with at minimum:

- **Project identity** — one-line pitch (what + who uses it)
- **What you keep** (good properties borrowed from other tools / languages / workflows)
- **What you drop** (explicit anti-patterns)
- **Engineering standards** — Elegant / Scientific / Efficient with 3-5 concrete rules each
- **Milestone roadmap** — M0 (scaffold) → M1 → ... 6-12 months out

Reference: ADSD's own SKILL.md "Engineering standards" section is a template.

### Step 2: Create `docs/agent/` + `docs/human/{zh,en}/` skeleton

```sh
cd ~/my-project
mkdir -p docs/agent/adr docs/agent/findings docs/agent/modules
mkdir -p docs/human/zh docs/human/en
```

Copy ADSD's `templates/adr-template.md` to `docs/agent/adr/_template.md` as your ADR drafting template. Same for finding-template, snapshot-template.

### Step 3: Write ADR-0001 (license choice)

Every project's first ADR is typically the license choice (Apache+MIT dual, or BSL-1.1, or ...). This is **the start of mandatory ADR flow** — one cross-multifile decision running through the complete process: Context → Options → Decision → Consequences → Cross-references.

### Step 4: Build `MEMORY.md` index (Claude Code auto-memory)

If you use Claude Code, project-level memory lives in `~/.claude/projects/<project-dir>/memory/`. Create the `MEMORY.md` index with one-line hooks:

```
- [Project identity preamble](identity.md) — read first when resuming a session
- [Subagent model tier rule](subagent_tiers.md) — D0-D5 matrix per ADSD
- [CTO operations runbook](runbook.md) — dispatch SOPs
```

See [`reference/cross-session-memory-architecture.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/cross-session-memory-architecture.md).

### Step 5: First sub-agent dispatch (using ADSD D-matrix)

Use Claude Code's Agent tool to dispatch a concrete task. **The prompt MUST include difficulty self-rating**:

```
DIFFICULTY-RATING: D2 (multi-fn stdlib API new, single crate, ADR clear)
MODEL-DEV: sonnet
MODEL-TEST: sonnet
PAIR: yes

MISSION: implement <feature> such that <test_corpus> all passes.

REQUIRED READS:
- /abs/path/to/ADR-0XXX.md
- /abs/path/to/test_corpus.rs
- see reference/prompt-engineering-patterns.md PT2 (few-shot output format)

REPORT FORMAT: [P7-COMPLETION] with verification block (paste raw cargo test output, no paraphrase)
```

See [`reference/prompt-engineering-patterns.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/prompt-engineering-patterns.md).

## Verify you installed correctly

Run these two checks:

```sh
# 1. Verify plugin activated
/plugin status adsd

# 2. In Claude Code, ask a question with ADSD keywords
"I need to plan a multi-agent dispatch, how do I use the D-matrix to assess difficulty?"
```

If Claude auto-references ADSD's `reference/` files, you installed correctly. If Claude answers from general knowledge, the skill didn't activate.

## Next steps

- Read [`concept-map.md`](./concept-map.md) for the complete ADSD concept diagram
- When you hit a wall, write a finding. Don't hide it. F1-F30 catalogue is at [`reference/failure-modes-catalogue.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/failure-modes-catalogue.md); you may have hit the same one

## FAQ

**Q: My project is small. Do I really need ADRs?**
A: Only for decisions affecting ≥2 files. Single-file modifications don't write ADRs. Bug fixes don't write ADRs (but do write findings).

**Q: Bilingual docs feel burdensome?**
A: ADSD mandates this because it addresses the real "Chinese teams are natively multilingual" problem. Single-language projects can relax this; but the README + getting-started bilingual pair is recommended.

**Q: D-matrix is tedious, do I need to evaluate every time?**
A: Manual evaluation for the first 5 times; after that it becomes muscle memory. Skipping costs you model-tier mismatch (F20 family) — projects that hit it think it's worth it.

**Q: I use OpenAI not Anthropic?**
A: ADSD is LLM-agnostic. D-matrix / dev-test pair / evals-first are all vendor-neutral. The Claude Code plugin is just a distribution channel; the methodology itself doesn't bind to Anthropic.
