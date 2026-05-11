# Agent-Driven Software Development (ADSD)

> Methodology distilled from running a 9-week multi-agent Rust compiler
> project where AI agents wrote ≥ 70% of the code under human strategic
> direction.

[![License: Apache 2.0 / MIT](https://img.shields.io/badge/license-Apache--2.0%20OR%20MIT-blue.svg)](#license)
[![Status: extracted from Cobrust](https://img.shields.io/badge/status-extracted%202026--05--11-orange.svg)](#origin)

## What this is

ADSD is **not a framework**. It's a documented working style that survived
contact with reality: ~178 commits, ~2,611 tests, 43 ADRs, 19 findings, 21
documented failure modes, 2 P0 codegen bugs caught via organic stress test,
and a 0.1.1 release shipped publicly.

ADSD codifies the discipline that kept the multi-agent project coherent:
ADRs as decision capture, findings as negative-result memory, bilingual
docs by default, wave-based delivery, D0-D5 difficulty matrix for agent
dispatch, test-first dev/test pair workflow, and release-readiness
verification before public-facing changes.

## When to use ADSD

- You're managing a software project where AI agents do most of the coding
  (≥ 70% of LOC produced by agents)
- You run **3+ parallel sub-agents** and need a way to prevent sediment /
  drift / silent regressions
- You're doing **stateful project management** (multi-week / multi-sprint),
  not one-shot tasks
- The project has external stakeholders (release notes, public roadmap,
  contributors) that need an honest narrative

ADSD is **overkill** for one-shot prompt → answer flows, single-developer
IDE-loop coding (Cursor / Claude Code already handle), and < 3-agent
simple workflows.

## Install

### As a Claude Code plugin (recommended)

```
/plugin marketplace add Cobrust-lang/agent-driven-development
/plugin install agent-driven-development@adsd
```

After install, invoke via `/agent-driven-development` or let Claude pick it
automatically based on context — the description-triggered activation fires
for multi-agent dispatch planning, ADR drafting, F1–F18 failure-mode triage,
pre-release audit team design, and similar prompts.

### As a personal skill (fallback, no plugin system)

If you can't or don't want to use `/plugin install`:

```sh
mkdir -p ~/.claude/skills
git clone --depth 1 https://github.com/Cobrust-lang/agent-driven-development.git /tmp/adsd-src
cp -r /tmp/adsd-src/plugins/agent-driven-development/skills/agent-driven-development ~/.claude/skills/
rm -rf /tmp/adsd-src
```

### Read-only (no install)

The methodology is plain markdown. Just read
[`plugins/agent-driven-development/skills/agent-driven-development/SKILL.md`](./plugins/agent-driven-development/skills/agent-driven-development/SKILL.md)
top-to-bottom (~36 KB, 30 min). Install matters only if you want Claude to
invoke it automatically based on conversation context.

## Repository layout

```
agent-driven-development/
├── .claude-plugin/
│   └── marketplace.json                       # Self-hosted single-plugin marketplace catalog
├── plugins/
│   └── agent-driven-development/              # Plugin root (matches marketplace.json source)
│       ├── .claude-plugin/
│       │   └── plugin.json                    # Plugin manifest
│       └── skills/
│           └── agent-driven-development/      # Skill — auto-discovered by Claude Code
│               ├── SKILL.md                   # Main methodology document (~36 KB)
│               ├── reference/
│               │   └── failure-modes-catalogue.md  # F1-F21 anti-patterns with empirical evidence
│               ├── case-study/
│               │   └── cobrust-multi-agent-experience.md  # The founding case study (N=1)
│               └── templates/
│                   ├── adr-template.md        # Architecture Decision Record
│                   ├── finding-template.md    # Negative result / failure capture
│                   ├── dispatch-prompt-p9.md  # Tech Lead sub-agent dispatch
│                   ├── dispatch-prompt-p7.md  # Senior Engineer sub-agent dispatch
│                   ├── handoff-cover-letter.md  # Cross-session handoff
│                   └── snapshot-template.md   # Project state snapshot
├── CONTRIBUTING.md
├── LICENSE-APACHE
├── LICENSE-MIT
└── README.md                                  # this file
```

## Quick start (after install)

1. Read [`SKILL.md`](./plugins/agent-driven-development/skills/agent-driven-development/SKILL.md) for the full methodology (~36 KB, 30 min).
2. Read [`reference/failure-modes-catalogue.md`](./plugins/agent-driven-development/skills/agent-driven-development/reference/failure-modes-catalogue.md) for the F1–F18 anti-patterns you'll likely hit. Don't re-derive them.
3. Read [`case-study/cobrust-multi-agent-experience.md`](./plugins/agent-driven-development/skills/agent-driven-development/case-study/cobrust-multi-agent-experience.md) to see ADSD applied in practice (warts and all).
4. Copy the relevant template from [`templates/`](./plugins/agent-driven-development/skills/agent-driven-development/templates/) into your project's `docs/agent/` tree.
5. Start writing ADRs as decisions actually happen — not speculatively.

## Origin

ADSD was extracted from the [Cobrust](https://github.com/Cobrust-lang/cobrust)
project, a Rust-implemented Python successor with an AI-native compiler.
Cobrust shipped its `0.1.0` stable tag on 2026-05-10 after a 9-week run
with multiple parallel Claude agents (Opus 4.7 and Sonnet 4.6) coordinated
via the methodology you'll find in [`SKILL.md`](./plugins/agent-driven-development/skills/agent-driven-development/SKILL.md).

The case study at [`case-study/cobrust-multi-agent-experience.md`](./plugins/agent-driven-development/skills/agent-driven-development/case-study/cobrust-multi-agent-experience.md)
documents both what worked and what broke. The failure modes catalogue
captures lessons we'd rather not re-learn.

## Status

**Validation N = 1**: Cobrust (this methodology's birthplace).

We are looking for design partners willing to apply ADSD to a second
project so the methodology can be tested outside its founding context.
File an issue describing your project if interested.

ADSD is **battle-tested but not orthodoxy**. Adapt it. If you find a
failure mode we missed, propose F22+ via a PR to
[`reference/failure-modes-catalogue.md`](./plugins/agent-driven-development/skills/agent-driven-development/reference/failure-modes-catalogue.md).

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md). We use ADSD to evolve ADSD —
contributions follow the same ADR + finding + dispatch discipline the
methodology itself describes.

## License

Licensed under either of

- Apache License, Version 2.0 ([LICENSE-APACHE](./LICENSE-APACHE) or
  http://www.apache.org/licenses/LICENSE-2.0)
- MIT license ([LICENSE-MIT](./LICENSE-MIT) or
  http://opensource.org/licenses/MIT)

at your option.

### Contribution

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.

## Acknowledgements

The methodology is shaped by patterns from:

- [Linear Method](https://linear.app/method) — calm-tech + cycles
- [TigerBeetle Way](https://tigerstyle.dev) — assertion discipline + deterministic simulation testing
- [Stripe internal playbook](https://stripe.com/blog) — memos over meetings
- [Basecamp Shape Up](https://basecamp.com/shapeup) — appetite-based scoping
- [OpenTelemetry semantic conventions](https://opentelemetry.io) — observability vocabulary
- [SLSA v1.1](https://slsa.dev) — provenance attestation
- [Astral's `uv` / `ruff`](https://astral.sh) — single-tool wedge UX

None of these tools or organizations endorse ADSD; the methodology
borrows ideas, not affiliation.
