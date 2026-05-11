# ADSD 概念图

> 用 mermaid 图表 + 简短文字, 把 ADSD 全套概念一图打散.

## 顶层视图

```mermaid
flowchart TB
    Constitution[CLAUDE.md 宪法] --> Decisions{需要决策吗?}
    Decisions -->|是, 跨 ≥2 文件| ADR[ADR — 决策记录]
    Decisions -->|否, 单文件| InCode[就在代码里改]

    Implementation[实施工作] --> Failure{出问题了吗?}
    Failure -->|是| Finding[Finding — 失败记录]
    Failure -->|否| Continue[继续]

    State[项目状态] --> Snapshot[snapshot.md — 状态快照]

    ADR --> Sprint[Sprint = Wave + Tx]
    Sprint --> Dispatch[Dispatch P9/P7 sub-agent]

    Dispatch --> Drating{D-Matrix 评估}
    Drating -->|D0 doc-only| Sonnet[sonnet solo]
    Drating -->|D1-D3 多复杂度| Pair[dev/test pair TDD]
    Drating -->|D4 ADR| OpusSolo[opus solo, P9 亲笔]
    Drating -->|D5 真 LLM/consensus| OpusPair[opus dev + opus test]

    Pair --> CommitWave[原子 commit + Wave merge]
    OpusPair --> CommitWave

    CommitWave --> ReleaseGate{Release artifact?}
    ReleaseGate -->|是| ReleaseReady[Release-readiness agent 独立验证]
    ReleaseReady -->|GO| Tag[git tag v0.X.Y]
    ReleaseReady -->|BLOCK| Fix[fix-pack sprint]
    Fix --> ReleaseReady
```

## 三层抽象 (从慢到快)

```mermaid
flowchart LR
    Strategy[战略层 — 月级别] --> Tactical[战术层 — 周级别]
    Tactical --> Execution[执行层 — 小时/天级别]

    Strategy -.包含.-> Constitution[宪法] & Wedge[Wedge / 战略方向] & Roadmap[Milestone 路线]
    Tactical -.包含.-> ADRs[ADRs] & Findings[Findings] & Waves[Waves] & PreMortem[Pre-mortem]
    Execution -.包含.-> Dispatch[Sub-agent Dispatch] & Tx[Tx commits] & Gates[5-gate + 6th eval-gate] & Release[Release-readiness]

    Strategy -.通过.- Tactical -.通过.- Execution
```

- **战略层**: CLAUDE.md 不常改, 月级别决策. 改 = 项目重大转向.
- **战术层**: ADR + Finding 每周新增, milestone 检查点.
- **执行层**: 每日 sprint, sub-agent 派活, gate 通过, atomic commit.

## 失败模式 (F1 Sediment Family) 全景

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

    F1_0 -.通过.-> Enforce0[snapshot-lint Inv]
    F1_1 -.通过.-> Enforce1[pre-commit hook]
    F16 -.通过.-> Enforce16[auto-memory identity preamble]
    F17 -.通过.-> Enforce17[verification commands block in completion report]
    F19 -.通过.-> Enforce19[release-readiness agent in clean shell]
    F20 -.通过.-> Enforce20[D-matrix + dev/test pair workflow]
    F21 -.通过.-> Enforce21[session-ID stamping convention]
```

每个 F-pattern 都有对应的 enforcement 机制. F1 Family 的核心 lesson: **声明规则不够, 必须有机器/工作流强制**.

## 四层 storage 模型 (memory 决策)

```mermaid
flowchart TB
    NewInfo{新信息要写哪?} --> Type{是哪类?}
    Type -->|身份 / 操作规则 / SOP| L1[L1 auto-memory<br/>~/.claude/projects/<proj>/memory/]
    Type -->|跨文件决策| L2A[L2 ADR<br/>docs/agent/adr/]
    Type -->|失败 / 意外 / 死胡同| L2B[L2 Finding<br/>docs/agent/findings/]
    Type -->|项目状态事实| L2C[L2 Snapshot<br/>project_state_snapshot.md]
    Type -->|本 sprint 工作中| L3[L3 session scratch<br/>消息中的笔记]
    Type -->|可再 fetch 的临时输出| L4[L4 ephemeral<br/>不存]

    L1 -.auto-load.-> Session[Session start]
    L2A -.持续到.-> Repo[git history]
    L2B -.持续到.-> Repo
    L2C -.持续到.-> Repo
    L3 -.持续到.-> SessionEnd[Session end]
```

不确定就**默认 L3 scratch**. 升级到 L1/L2 是 sprint 收尾时**主动决策**, 不在过程中.

## Dispatch 协议 (dev/test pair pattern)

```mermaid
sequenceDiagram
    participant P9 as P9 Tech Lead
    participant Test as P7 Test Agent
    participant Dev as P7 Dev Agent

    P9->>P9: 评估 D-rating (D1-D3 / D5 → pair)
    P9->>Test: spawn (TDD step 1 — 写 failing 测试集)
    Test-->>P9: [P7-TEST-CORPUS-READY] N tests, K fail
    P9->>P9: review test corpus (10 min)
    P9->>Dev: spawn (TDD dev step — 实现 + 通过 corpus)
    Dev-->>P9: [P7-DEV-COMPLETION] cargo test 0 fail
    P9->>P9: verify gate + atomic commit
    P9-->>CTO: [P9-MILESTONE-COMPLETION]
```

**为什么必须独立 test agent + dev agent**: 同一个 agent 写 impl + test 会有 confirmation bias — test 验证的是它自己想做的, 不是 spec 要求的. 独立 test agent 消除偏见.

## Release 闭环 (含 release-readiness)

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

**F19 闭环关键**: 不让写文档的 agent 自验文档. **独立 release-readiness agent 在 clean shell 跑** 是 F19 唯一 robust 防御.

## 怎么把这些图变成实战

每张图都是一种"实战剧本":

- 顶层视图 → 起新项目时按这条流程
- 三层抽象 → 团队节奏感, 每天/每周/每月各做什么
- F1 Family → 撞坑时查这张图, 哪个 enforcement 缺了
- Storage 四层 → 写东西前对照决策树
- Dispatch 协议 → P9 发起 sprint 时按此 sequence
- Release 闭环 → tag 前必走这条 path

参考 [`getting-started.md`](./getting-started.md) 的 5 步实战, 把这些图落到具体命令.
