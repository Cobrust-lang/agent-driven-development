# 入门指南

> **目标**: 30 分钟内让一个不熟悉 ADSD 的工程师在自己项目里开始用 ADRs + findings + sub-agent 派活的规范.

## 谁该读这份文档

- 你正在管理一个**多 agent 并行**的软件项目 (≥3 个 AI agent 同时干活)
- 你想避免 sediment / drift / silent regression 这些**多 agent 顽疾**
- 你已经会用 Claude Code / Cursor / 类似 IDE-agent 工具的基本操作
- 你有一个 git 项目可以套这套方法论

如果你只是写一个单 agent 的小脚本, ADSD 是 overkill, 跳过.

## 30 秒概览

ADSD 是从 Cobrust 项目 12 天密集开发实战 (2026-04-30 → 2026-05-12, ~278 commits) 提炼的**多 agent 工作纪律**, 把以下三件事做硬:

1. **决策捕获** — 每个跨文件的决定都写 ADR (Architecture Decision Record)
2. **失败捕获** — 每次"翻车 / 意外 / 死胡同"都写 Finding (负向结果)
3. **派活有谱** — D0-D5 难度矩阵 + dev/test pair 的 TDD 派活协议

加上**双语文档强制** + **wave + Tx 原子提交** + **F1-F30 反模式目录** + **release-readiness 上线前独立验证**, 就是 ADSD 全貌.

详细架构: [`concept-map.md`](./concept-map.md)

## 三种安装方式

### 方式 1 (推荐) — Claude Code plugin

```
/plugin marketplace add Cobrust-lang/agent-driven-development
/plugin install adsd@adsd
```

装完后, 命中"multi-agent dispatch / ADR drafting / F1-F30 failure mode"等关键词时, Claude 会自动激活 ADSD skill.

### 方式 2 — 个人 skill 目录 (回退方案)

```sh
mkdir -p ~/.claude/skills
git clone --depth 1 https://github.com/Cobrust-lang/agent-driven-development.git /tmp/adsd-src
cp -r /tmp/adsd-src/plugins/adsd/skills/agent-driven-development ~/.claude/skills/
rm -rf /tmp/adsd-src
```

### 方式 3 — 只读 (不装, 看 markdown)

直接读 [`plugins/adsd/skills/agent-driven-development/SKILL.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/SKILL.md), 30 分钟读完核心方法论. 不装也能学.

## 第一次实战 — 5 步落地

假设你有一个项目 `~/my-project/`, 想开始用 ADSD.

### 步骤 1: 创建项目 `CLAUDE.md` (宪法)

在 `~/my-project/CLAUDE.md` 写下 30 行的项目宪法, 至少包含:

- **项目身份** — 一行 pitch (是什么 + 谁用)
- **要保留的东西** (从其他语言 / 工具 / 工作流借鉴的良性属性)
- **要丢弃的东西** (明确反模式)
- **工程标准** — Elegant / Scientific / Efficient 各 3-5 条具体规定
- **里程碑表** — M0 (脚手架) → M1 → ... 现在 + 未来 6-12 个月

参考: ADSD 自己的 SKILL.md "Engineering standards" 段是模板.

### 步骤 2: 创建 `docs/agent/` + `docs/human/{zh,en}/` 目录骨架

```sh
cd ~/my-project
mkdir -p docs/agent/adr docs/agent/findings docs/agent/modules
mkdir -p docs/human/zh docs/human/en
```

把 ADSD 的 `templates/adr-template.md` 复制到 `docs/agent/adr/_template.md` 作为 ADR 起草模板. 同理 finding-template, snapshot-template.

### 步骤 3: 写 ADR-0001 (license 选择)

每个项目第一个 ADR 通常是 license 选择 (Apache+MIT dual, 或 BSL-1.1, 或 ...). 这是**强制走 ADR 流程**的开始 — 一次跨多文件的决定, 走完整流程: Context → Options → Decision → Consequences → Cross-references.

### 步骤 4: 建立 `MEMORY.md` 索引 (Claude Code auto-memory)

如果你用 Claude Code, 项目级 memory 在 `~/.claude/projects/<project-dir>/memory/`. 创建 `MEMORY.md` 索引, 一行一条:

```
- [Project identity preamble](identity.md) — read first when resuming a session
- [Subagent model tier rule](subagent_tiers.md) — D0-D5 matrix per ADSD
- [CTO operations runbook](runbook.md) — dispatch SOPs
```

详见 [`reference/cross-session-memory-architecture.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/cross-session-memory-architecture.md).

### 步骤 5: 第一次 sub-agent 派活 (用 ADSD D-matrix)

用 Claude Code 的 Agent tool 派一个具体任务. **prompt 必须含 difficulty self-rating**:

```
DIFFICULTY-RATING: D2 (multi-fn stdlib API new, single crate, ADR clear)
MODEL-DEV: sonnet
MODEL-TEST: sonnet
PAIR: yes

MISSION: 实现 <feature> 使得 <test_corpus> 全部通过.

REQUIRED READS:
- /abs/path/to/ADR-0XXX.md
- /abs/path/to/test_corpus.rs
- 见 reference/prompt-engineering-patterns.md PT2 (few-shot 输出格式)

REPORT FORMAT: [P7-COMPLETION] with verification block (paste raw cargo test output, no paraphrase)
```

详见 [`reference/prompt-engineering-patterns.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/prompt-engineering-patterns.md).

## 验证你装对了

跑这两条命令:

```sh
# 1. 验证 plugin 已激活
/plugin status adsd

# 2. 在 Claude Code 里问个问题, 含 ADSD 关键词
"我需要 plan 一个 multi-agent dispatch, 怎么用 D-matrix 评估难度?"
```

如果 Claude 自动引到 ADSD 的 reference, 装对了. 如果 Claude 用通用知识回答, skill 没激活.

## 下一步

- 读 [`concept-map.md`](./concept-map.md) 看 ADSD 完整概念图
- 撞坑了写 finding, 不要藏起来. F1-F30 catalogue 在 [`reference/failure-modes-catalogue.md`](https://github.com/Cobrust-lang/agent-driven-development/blob/main/plugins/adsd/skills/agent-driven-development/reference/failure-modes-catalogue.md), 你可能撞上同一个

## 常见问题

**Q: 我项目很小, 真的需要 ADR 吗?**
A: 跨 ≥2 文件的决定才写. 单文件修改不写. 修 bug 不写 (但写 finding).

**Q: zh + en 双语文档负担太重?**
A: ADSD 强制是因为它解决了"中国团队天然 multi-lingual"的真实问题. 单语项目可以放宽, 但 README + getting-started 双语建议保持.

**Q: D-matrix 太繁琐, 我每次都得想一遍?**
A: 头 5 次手动评估, 之后就成肌肉记忆. 跳过的代价是 model tier 错配 (F20 family) — 真撞坑过的项目觉得值.

**Q: 我用 OpenAI 不用 Anthropic?**
A: ADSD 是 LLM-agnostic. D-matrix / dev-test pair / evals-first 都 vendor-neutral. Claude Code plugin 部分只是发行渠道, 方法论本身不绑 Anthropic.
