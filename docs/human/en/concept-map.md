# ADSD concept map

> Mermaid diagrams + short prose to unpack the full ADSD concept landscape at once.

## Top-level view

```mermaid
flowchart TB
    Constitution[CLAUDE.md constitution] --> Decisions{Need a decision?}
    Decisions -->|Yes, affects ≥2 files| ADR[ADR — decision record]
    Decisions -->|No, single file| InCode[Just code it]

    Implementation[Implementation work] --> Failure{Did something break?}
    Failure -->|Yes| Finding[Finding — failure record]
    Failure -->|No| Continue[Continue]

    State[Project state] --> Snapshot[snapshot.md — state snapshot]

    ADR --> Sprint[Sprint = Wave + Tx]
    Sprint --> Dispatch[Dispatch P9/P7 sub-agent]

    Dispatch --> Drating{D-Matrix assessment}
    Drating -->|D0 doc-only| Sonnet[sonnet solo]
    Drating -->|D1-D3 multi complexity| Pair[dev/test pair TDD]
    Drating -->|D4 ADR| OpusSolo[opus solo, P9 personal]
    Drating -->|D5 real LLM/consensus| OpusPair[opus dev + opus test]

    Pair --> CommitWave[Atomic commit + Wave merge]
    OpusPair --> CommitWave

    CommitWave --> ReleaseGate{Release artifact?}
    ReleaseGate -->|Yes| ReleaseReady[Release-readiness agent independent verify]
    ReleaseReady -->|GO| Tag[git tag v0.X.Y]
    ReleaseReady -->|BLOCK| Fix[fix-pack sprint]
    Fix --> ReleaseReady
```

## Three abstraction layers (slow → fast)

```mermaid
flowchart LR
    Strategy[Strategy — month-scale] --> Tactical[Tactical — week-scale]
    Tactical --> Execution[Execution — hour/day-scale]

    Strategy -.includes.-> Constitution[Constitution] & Wedge[Wedge / strategic direction] & Roadmap[Milestone roadmap]
    Tactical -.includes.-> ADRs[ADRs] & Findings[Findings] & Waves[Waves] & PreMortem[Pre-mortem]
    Execution -.includes.-> Dispatch[Sub-agent Dispatch] & Tx[Tx commits] & Gates[5-gate + 6th eval-gate] & Release[Release-readiness]

    Strategy -.through.- Tactical -.through.- Execution
```

- **Strategy layer**: CLAUDE.md rarely changes; month-scale decisions. Changing it = major project pivot.
- **Tactical layer**: ADR + Finding added weekly; milestone checkpoints.
- **Execution layer**: daily sprints, sub-agent dispatch, gate enforcement, atomic commits.

## Failure modes (F1 Sediment Family) panorama

```mermaid
flowchart TB
    F1[F1 Sediment Family — declared-without-enforcement] --> F1_0[F1.0 schema invariant]
    F1 --> F1_1[F1.1 snapshot HEAD freshness]
    F1 --> F1_2[F1.2 ADR roster completeness]
    F1 --> F16[F16 post-compaction identity drift]
    F1 --> F17[F17 sub-agent KPI self-report]
    F1 --> F18[F18 attribution policy scope]
    F1 --> F19[F19 install-not-tested]
    F1 --> F20[F20 constitution-vs-workflow]
    F1 --> F21[F21 cross-session identity overload]

    F1_0 -.via.-> Enforce0[snapshot-lint Inv]
    F1_1 -.via.-> Enforce1[pre-commit hook]
    F16 -.via.-> Enforce16[auto-memory identity preamble]
    F17 -.via.-> Enforce17[verification commands block in completion report]
    F19 -.via.-> Enforce19[release-readiness agent in clean shell]
    F20 -.via.-> Enforce20[D-matrix + dev/test pair workflow]
    F21 -.via.-> Enforce21[session-ID stamping convention]
```

Each F-pattern has a corresponding enforcement mechanism. The F1 Family core lesson: **declaring rules isn't enough; you must have machine / workflow enforcement**.

## Four-layer storage model (memory decision)

```mermaid
flowchart TB
    NewInfo{Where to write new info?} --> Type{What category?}
    Type -->|Identity / operative rule / SOP| L1[L1 auto-memory<br/>~/.claude/projects/<proj>/memory/]
    Type -->|Cross-file decision| L2A[L2 ADR<br/>docs/agent/adr/]
    Type -->|Failure / surprise / dead-end| L2B[L2 Finding<br/>docs/agent/findings/]
    Type -->|Project state fact| L2C[L2 Snapshot<br/>project_state_snapshot.md]
    Type -->|Mid-sprint working state| L3[L3 session scratch<br/>notes in messages]
    Type -->|Re-fetchable ephemeral output| L4[L4 ephemeral<br/>don't store]

    L1 -.auto-load.-> Session[Session start]
    L2A -.persists in.-> Repo[git history]
    L2B -.persists in.-> Repo
    L2C -.persists in.-> Repo
    L3 -.persists until.-> SessionEnd[Session end]
```

When unsure, **default to L3 scratch**. Promotion to L1/L2 is a deliberate decision at sprint-end, not in-flight.

## Dispatch protocol (dev/test pair pattern)

```mermaid
sequenceDiagram
    participant P9 as P9 Tech Lead
    participant Test as P7 Test Agent
    participant Dev as P7 Dev Agent

    P9->>P9: Assess D-rating (D1-D3 / D5 → pair)
    P9->>Test: spawn (TDD step 1 — write failing test corpus)
    Test-->>P9: [P7-TEST-CORPUS-READY] N tests, K fail
    P9->>P9: review test corpus (10 min)
    P9->>Dev: spawn (TDD dev step — implement + pass corpus)
    Dev-->>P9: [P7-DEV-COMPLETION] cargo test 0 fail
    P9->>P9: verify gate + atomic commit
    P9-->>CTO: [P9-MILESTONE-COMPLETION]
```

**Why a separate test agent + dev agent is mandatory**: a single agent writing impl + test has confirmation bias — the test verifies what the agent intended, not what the spec demands. Separate test agent eliminates the bias.

## Release closure (with release-readiness)

```mermaid
flowchart LR
    Code[Code Ready] --> Gate5[5-gate Green<br/>fmt+clippy+build+test+doc-cov]
    Gate5 --> Gate6[6th gate — Eval Delta Non-Regression]
    Gate6 --> ReleaseFile[Edit Release Notes / README]
    ReleaseFile --> ReleaseAgent[Spawn Release-readiness Agent<br/>clean shell + curl + cargo install --dry-run]
    ReleaseAgent --> Decision{GO or BLOCK?}
    Decision -->|GO| Tag[git tag v0.X.Y]
    Decision -->|BLOCK| Fix[Fix root cause]
    Fix --> ReleaseAgent
```

**F19 closure key**: don't let the agent that wrote the docs self-verify the docs. **Independent release-readiness agent in a clean shell** is the only robust F19 defense.

## Turning these diagrams into practice

Each diagram is a "practice script":

- Top-level view → follow this flow for a new project
- Three abstraction layers → team cadence, what to do daily/weekly/monthly
- F1 Family → consult this when you hit a wall, find the missing enforcement
- Storage four-layer → consult the decision tree before writing
- Dispatch protocol → P9 follows this sequence when initiating a sprint
- Release closure → mandatory path before any tag

See [`getting-started.md`](./getting-started.md) 5-step practice section to map these diagrams to concrete commands.
