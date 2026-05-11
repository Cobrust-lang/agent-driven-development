---
name: ADSD repo conventions
description: Meta-conventions for this repo itself. ADSD codifies how to manage AI-agent projects; this file applies ADSD discipline to the ADSD methodology itself. Agents contributing to this repo should read this first.
type: convention
version: 1.0.0
date: 2026-05-12
status: active
relates_to: [SKILL.md §"Documentation Discipline", README.md §Contributing, CONTRIBUTING.md]
---

# ADSD repo conventions

> ADSD is a methodology for managing AI-agent software projects. This repo IS such a project. Therefore, **ADSD applies to ADSD**. This file captures the meta-conventions specific to this repo's contributors (humans and AI agents).

## Repo structure (binding)

```
agent-driven-development/
├── .claude-plugin/marketplace.json     # Plugin marketplace catalog
├── plugins/
│   └── adsd/
│       └── skills/
│           └── agent-driven-development/   # The skill — auto-loaded by Claude Code
│               ├── SKILL.md                # Main methodology (~36 KB)
│               ├── reference/              # Deep-dive references (F-patterns, evals, prompts, etc.)
│               ├── case-study/             # Founding case study (Cobrust N=1)
│               └── templates/              # Templates for ADR / finding / dispatch / snapshot / handoff
├── docs/
│   ├── human/
│   │   ├── zh/                             # Chinese user docs — 与 en 一一对应
│   │   └── en/                             # English user docs — 1:1 parity with zh
│   └── agent/
│       ├── conventions.md                  # This file
│       ├── adr/                            # Meta-ADRs for ADSD itself
│       └── findings/                       # Findings about ADSD's evolution
├── scripts/
│   └── doc-coverage.sh                     # Enforces zh+en parity per ADSD §3 mandate
├── CONTRIBUTING.md                         # Human-facing contribution guide
├── LICENSE-APACHE / LICENSE-MIT
└── README.md                               # Entry point
```

**Binding constraints**:

1. Every file in `docs/human/zh/` MUST have a parallel file at `docs/human/en/` with the same filename. Enforced by `scripts/doc-coverage.sh`.
2. Every reference under `plugins/adsd/skills/agent-driven-development/reference/` MUST have YAML frontmatter (`name`, `description`, `type`, `version`, `date`, `status`, `relates_to`).
3. The SKILL.md `description` field is the auto-activation trigger — keep it keyword-dense and specific.
4. ADRs in `docs/agent/adr/` are zero-padded sequential (`0001-*.md`, `0002-*.md`, ...). Once accepted, an ADR is immutable; supersede via a new ADR.

## Frontmatter contracts

### Reference files (in `plugins/adsd/skills/agent-driven-development/reference/`)

```yaml
---
name: <Reference title>
description: <One-line trigger / summary>
type: reference
version: <semver>
date: <ISO date of last substantive edit>
status: active | deprecated | candidate
relates_to: [skill:SKILL.md §section, reference:other-file.md, ...]
---
```

### Meta-ADRs (in `docs/agent/adr/`)

```yaml
---
doc_kind: adr
adr_id: <NNNN, zero-padded>
title: <ADR title>
status: proposed | accepted | superseded | deprecated
date: <YYYY-MM-DD>
last_verified_commit: <SHA or TBD>
supersedes: [<adr_id>, ...]
superseded_by: [<adr_id>, ...]
relates_to: [<adr_id>, <finding-slug>, ...]
---
```

### Findings (in `docs/agent/findings/`)

```yaml
---
doc_kind: finding
finding_id: <slug>
last_verified_commit: <SHA>
status: open | closed | partial
discovered_by: <agent role + session ID>
dependencies: [adr:<NNNN>, finding:<slug>, ...]
---
```

## Bilingual docs mandate (ADSD §3 dogfood)

The skill's SKILL.md §3 mandates that every public item gets entries in:

- `docs/human/zh/<topic>.md`
- `docs/human/en/<topic>.md` (1:1 parity)
- Agent-facing schema (in this repo: SKILL.md + reference/)

This rule applies to ADSD itself. `scripts/doc-coverage.sh` enforces zh+en parity.

**Operative checks** (run by `doc-coverage.sh`):

1. Every `docs/human/zh/*.md` has a parallel `docs/human/en/*.md`
2. Every `docs/human/en/*.md` has a parallel `docs/human/zh/*.md`
3. Parallel files have identical filenames (case-sensitive)
4. (Future) Section headers are 1:1 between zh and en

CI fails if any check fails.

## When to add a new ADR vs amend SKILL.md vs add a finding

| Change type | Where | Trigger |
|---|---|---|
| New methodology rule | `docs/agent/adr/NNNN-<slug>.md` | The rule affects ≥2 reference files, templates, or SKILL.md sections |
| Refine existing reference | edit the reference file directly + note in commit | Single-file refinement |
| Document an ADSD evolution event | `docs/agent/findings/<slug>.md` | Real-world ADSD use surfaced a gap or worked unexpectedly well |
| Update SKILL.md | edit SKILL.md + cross-reference an ADR if it's a binding rule | Adds a new "Part N" or modifies an existing one |
| New cross-pollination ref (Anthropic / OpenAI / other) | `plugins/.../reference/<slug>.md` | New industry pattern worth adopting |

## When NOT to add an ADR

- Bug fix in a reference doc (typo, broken link)
- Updating frontmatter date / last_verified
- Adding an example to an existing section
- Re-organizing within a single file
- Translation update (zh ⟵→ en sync)

Per ADSD §"ADR vs Finding distinction": ADRs are forward-looking decisions; small refinements don't need them.

## Commit message format

```
<type>(<scope>): <short description> [vX.Y.Z]
```

- `<type>`: `feat`, `docs`, `fix`, `refactor`, `chore`
- `<scope>`: `skill`, `reference`, `case-study`, `templates`, `docs-zh`, `docs-en`, `meta`
- Include `[vX.Y.Z]` semver if the change is release-worthy

Examples:

```
feat(reference): add evals-first-development.md (v1.2.0)
docs(zh): translate getting-started.md to match en parity
fix(skill): correct cross-reference path after plugin layout migration
chore(meta): bump SKILL.md description for trigger keyword coverage
```

Sign with session ID per F21 (Cross-session identity overload):

```
Co-Authored-By: Claude Opus 4.7 (session XYZ) <noreply@anthropic.com>
```

## Identity hygiene (F21 closure)

Per F21 codification:

- Do NOT sign as bare "review-claude" or "ADSD-author" in commits or files
- Use session-stamped attribution: `review-claude (session 4bb35f43)` or `ADSD-author (session XYZ)`
- Reserve plain handles for the abstract role in narrative prose only

## Versioning policy

- **v1.0.x** — initial release, plugin migration
- **v1.1.x** — F19/F20/F21 codification
- **v1.2.x** — cross-pollination references (Anthropic + OpenAI)
- **v1.3.x** — bilingual docs + remaining G3/G5/G6/G8/G9/G10/G12 gaps

Semver bumps follow SemVer 2.0:

- MAJOR: breaking change to skill format, plugin layout, or canonical paths
- MINOR: new reference file, new template, new ADR
- PATCH: refinement, typo, frontmatter update, translation sync

## Cross-references

- `CONTRIBUTING.md` — human-facing contribution flow
- `plugins/adsd/skills/agent-driven-development/SKILL.md` §"Documentation Discipline" — methodology origin of these rules
- `plugins/adsd/skills/agent-driven-development/reference/failure-modes-catalogue.md` F1 family + F19 + F20 + F21 — the failure modes these conventions prevent
- `scripts/doc-coverage.sh` — machine enforcement of zh+en parity
