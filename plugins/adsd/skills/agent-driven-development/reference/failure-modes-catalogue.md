# Failure modes catalogue

> Concrete failure modes encountered in real ADSD projects, with
> recovery patterns. Each entry references a specific case-study artifact.
>
> Add to this catalogue as your project hits new failure modes —
> negative results are first-class, including process failures.

---

## F1 — Declared rules without enforcement — **"F1 Sediment Family"** (**P0 SOP gap, 6 sub-forms confirmed**)

> **Status upgraded to "F1 Sediment Family" parent pattern** after 6 distinct
> sub-forms observed across Cobrust 11-day experiment. F1 is the single most
> common systemic failure in ADSD-flavor projects. Original 3 sub-forms
> (F1.0 / F1.1 / F1.2) remain as implementation-level instances. New
> sub-forms F16, F17, F18 extend the family to identity, self-reporting, and
> attribution-policy dimensions — all share the same root: **declaration ≠
> enforcement, and enforcement scope silently lags reality.**
>
> **Family pattern one-liner**: Claim is written somewhere (constitution,
> schema frontmatter, KPI card, attribution policy, auto-memory). No
> automated mechanism verifies the claim. Claim gets violated within 1–3
> turns. Violation is invisible until an auditor manually checks.
>
> See F16 (identity drift), F17 (self-report fidelity gap), F18 (attribution
> policy without dir-scope enforcement) for the three new sub-forms.

### F1.0 — Snapshot sediment ("重写忘删")

### Symptoms

State doc has self-contradictory sections:
- §"Currently in-flight" says "ZERO sub-agent" while §"In-flight at
  this turn" lists 3 active worktrees
- §"ADR roster" table ends at 0034 while §"Phase E milestones" cites
  ADR-0035 as merged
- HEAD field reads stale SHA from N commits ago

### Root cause

When updating the snapshot, agent writes new content but doesn't
delete obsolete sections. Each turn adds a layer; sediment accumulates.

### Recovery

1. Stop. Do not dispatch further sprints until snapshot reconciled.
2. Three-way reconcile: `git log -10` + `ls findings/` + `ls adr/`
   compared against snapshot's claims.
3. Delete obsolete sections (don't preserve stale residue).
4. Add `schema_invariant` frontmatter:
   ```yaml
   schema_invariant: |
     Every ADR mentioned in any section must appear in §"ADR roster" table.
     Binary verification list appears EXACTLY ONCE.
   ```
5. (Optional) Add CI lint: `scripts/snapshot-lint.sh` that fails
   builds if invariant violated.

### Evidence

Cobrust 8th close-out review §"5 处 stale fact"; 9th review §"3 处
新 cracks"; 10th review §"sediment 第三次".

### Prevention going forward

The schema-invariant rule is necessary but not sufficient. Stronger:
add CI lint that validates the invariant. Without lint, declared
invariants are documentation only.

### F1.1 — Declared schema_invariants without CI lint

**Symptoms**: snapshot.md frontmatter declares
```yaml
schema_invariant: |
  HEAD SHA must equal `git log -1 --format=%h main` at write time.
  Every ADR mentioned must appear in §ADR roster.
```
But there's no script that verifies these. Each invariant gets violated
within 1-2 turns of being declared.

**Root cause**: invariants are documentation; documentation is not
enforcement.

**Evidence**: Cobrust 9th + 11th reviews. After 9th review fix added
schema_invariant block to snapshot frontmatter, the 11th-review multi-agent
audit found:
- Invariant 5 (HEAD SHA) violated: snapshot says `6008634`, real `06df4b4`
  (14 commits behind)
- Invariant 2 (every finding mentioned in ledger): ADR-0035 mis-pinned
  to "prompt-design" propagated to 2 files (real ADR is 0036)
- Lifecycle mismatch: msgpack finding `status: open` despite being
  closed by `99ebc54`

**Recovery**: invariants must compile to script assertions. Add
`scripts/snapshot-lint.sh` that fails build if `git log -1 --format=%h`
≠ snapshot's HEAD field, and similar checks for ADR/finding ledger
completeness. Run in CI on every snapshot edit.

### F1.2 — Constitution rules with partial-scope enforcement

**Symptoms**: constitution declares "Code change ⇒ both doc trees in
same commit" (Cobrust §3.3). Project ships `scripts/doc-coverage.sh`
that enforces this. **But the script's check scope only covers
M0..M14 baseline ADRs (0001..0029)**. Anything past — ADR-0030..0039
in Cobrust's case — silently skips the gate.

**Root cause**: doc-coverage rules are written for milestones known at
script-creation time. New milestones add ADRs/findings, but the script
doesn't auto-extend.

**Evidence**: Cobrust 11th-review §H2.
`grep -rE "ADR-003[0-9]" docs/human/` returns **0 hits**.
ADR-0030..0039 全部 not in zh+en doc trees. Triple-tree drift is
systemic for all post-M14 work, but doc-coverage.sh is silent on it.

**Recovery**: doc-coverage scripts must auto-discover scope via
`ls docs/agent/adr/00*.md` patterns, not hardcode milestone lists.
Same applies to any "rule covers M0-M<N>" pattern — it will go stale
the moment M<N+1> lands.

### Generalized prevention going forward (P0 SOP)

> **Any project-level rule without an automated check is security
> theater.** When you write a rule (in constitution, ADR, schema
> frontmatter, snapshot invariant, conventions doc), in the SAME
> commit add the script that enforces it. If you can't enforce it,
> mark the rule "ASPIRATIONAL" not "REQUIRED".
>
> If the enforcement scope is bounded ("checks ADR-0001..0029"), the
> rule will go stale the moment scope is exceeded. Auto-discover
> scope (glob ADR files, find by frontmatter, etc.) instead.

This is **non-negotiable** in ADSD — equivalent severity to atomic-commit
discipline and CLI fail-fast. A project with declared-but-not-enforced
rules will accumulate F1 sediment indefinitely.

---

## F2 — Codegen vs MIR layer divergence

### Symptoms

ADR §"Decision" §"Implementation map" specifies `crates/cobrust-codegen/`
file paths. P9 sub-agent reports successful fix landed in
`crates/cobrust-mir/`. The fix is correct, but the §Decision was wrong
about the layer.

### Root cause

When a bug's symptom is "if vs while diverge" or "function call vs
non-function call diverge", the natural hypothesis points at codegen
(the last-mile translation). But "if vs while" is often a MIR
construction issue (how blocks are sequenced and terminated), not a
codegen IR-emission issue.

### Recovery

1. P9 reports finding via §"Layer correction" addendum (don't rewrite
   §Decision — preserve audit trail).
2. Update SOP catalogue: future codegen-divergence ADRs default to
   dump MIR + CLIF before locking implementation map.
3. ADR §Decision §"Implementation map" lists *both* layers as
   investigation candidates, not just codegen.

### Evidence

Cobrust ADR-0033 (Ty::None inference) and ADR-0035 (lower_condition
primitive). Both spiked at codegen, both fixed at MIR. "2 strikes =
systemic blind spot" — promoted to runbook section.

### Prevention going forward

Add a "Codegen vs MIR pre-flight" SOP: any ADR §Decision proposing a
codegen change must include CLIF + MIR dumps for failing inputs in
§"Context" §"Evidence", explicitly ruling out the upper layer.

---

## F3 — Two bugs one fix (root primitive missed)

### Symptoms

Bug A: trigger pattern X, wrong value type T. Filed as separate finding.
Bug B: trigger pattern Y (different), wrong value type T (same).
Filed as separate finding.

If you fix Bug A by patching X-handling, Bug B remains. If you fix B,
A remains. **Both are surface manifestations of one root cause** in a
shared inference / fallback path.

### Root cause

Type inference / dispatch / fallback pass has a wrong default for one
class of inputs. Both X and Y trigger that path.

### Recovery

1. When two finding hypothesize the same wrong-value-type, **compare
   their root-cause sections** for overlap.
2. Find the shared primitive (e.g. `lower_condition`,
   `infer_return_type` fallback).
3. Refactor the primitive at the root layer — fix both bugs in one
   edit.
4. Document as "two-bugs-one-fix" methodology finding for reuse.

### Evidence

Cobrust ADR-0033 Option C: both `Ty::None Float→I8` (Bug A) and
`i64-mod-2 → I8 narrow-type` (Bug B) closed by threading
`inferred_locals` through `operand_ty` + `rvalue_ty` with fixed-point.

### Prevention going forward

ADSD §4 verification gates: when reviewing a finding's root-cause
hypothesis, scan all open findings for matching wrong-value-types.
If 2+ match, prioritize root-primitive fix over surface patches.

---

## F4 — Quarantine pollution (audit on broken baseline)

### Symptoms

Audit sub-agent #1 reports "L2.behavior PARTIAL-FAIL" while another
sprint #2 is mid-flight fixing a known codegen bug. The audit's
"fail" signal is contaminated — root cause might be the codegen
bug being fixed, not the audit subject.

### Root cause

Audit started before known-bad-baseline was patched. Audit's findings
will need re-validation post-patch.

### Recovery

1. SendMessage to in-flight audit agents: "your baseline is dirty,
   await codegen fix merge before reporting [P9-COMPLETION]".
2. Have a written **quarantine SOP**: which sprints can run on which
   baselines.
3. Codegen sprint completes → SendMessage audit agents: "fixed at
   <SHA>, re-run validation, then report".

### Evidence

Cobrust `findings/audit-1-codegen-pollution-quarantine-sop.md`.

### Prevention going forward

CTO 守闸 protocol: before dispatching an audit sprint, check that:
- No codegen / MIR / type-system sprints are in flight
- All recently-merged sprints have completed 5-gate
- HEAD is at a known-clean SHA, not mid-merge

---

## F5 — Silent miscompile (verifier reject + binary still emit)

### Symptoms

User runs `tool build foo.cb`. Output: "Cranelift verifier error:
inst441 (...) type mismatch". Then: "linked /tmp/foo". User runs
`/tmp/foo` and gets wrong stdout.

### Root cause

Tool's CLI catches the verifier error (prints it), but doesn't
propagate the Err — continues to emit a binary anyway. Binary may run
but produce wrong values.

### Recovery

1. P0 priority: CLI fail-fast. Verifier rejection → propagate Err →
   exit non-zero.
2. Add regression test: feed a known-bad IR, assert exit code ≠ 0
   AND no binary written.

### Evidence

Cobrust commit `78ca779` "P0 CLI hardening — lock verifier-reject
exit-3"; finding `codegen-i8-i64-mismatch-at-4-blocks.md` §"Bug 2".

### Prevention going forward

ADSD §4 verification gates: every verifier in the pipeline must
contract that "verifier Err" → "tool exit ≠ 0", and this contract is
itself tested via a negative test.

---

## F6 — Push permission 403 / namespace assumption

### Symptoms

External review agent or sub-agent runs `git push origin feature/X`.
Result: HTTP 403 "permission denied". Push aborts; review work isn't
visible at remote.

### Root cause

Repo namespace is `Org-X/repo`, not `org-x/repo`. Or the agent's
account has read-only access. Or the repo isn't public yet.

### Recovery

1. Discover via `git push --dry-run` or by asking the human directly.
2. All dispatch prompts that say "git push" should be "open PR via
   `gh pr create`" — never assume push permission.
3. Hooks that fail on 403 should degrade to info-level, not abort the
   shell session.

### Evidence

Cobrust 10th review post-T1.1 cleanup discovery. Actual namespace was
`Cobrust-lang/cobrust`, not `cobrust/cobrust` (review-claude assumed).

### Prevention going forward

ADSD §1 Topology: external review agent's prompt always specifies
"do not push; draft PRs only". External review runs `git push
--dry-run` first; on 403, reports through chat instead of attempting
push retry.

---

## F7 — Strategic blindness (only tactical reviews)

### Symptoms

After 10+ reviews, every review is "snapshot is stale" / "codegen
edge case" / "this sprint missed a footnote". Zero reviews ask "what
problem is this project actually solving?" / "who's the user?" / "1-year
plan?".

The project may be technically sound but strategically lost.

### Root cause

Tactical review is naturally what the codebase invites — there's
always something incomplete. Strategic review takes effort; humans
forget to schedule it.

### Recovery

1. Force a strategic review every N tactical reviews. Cobrust
   experience: N = 10 was too high; recommended N = 5.
2. Strategic review writes 6/12/60-month plan in falsifiable form.
3. Compare current sprint roadmap against plan; identify drift.

### Evidence

Cobrust 10th review (this skill's birthplace; the review user
triggered with "一直局限于短期目标"). Without this trigger, project
would have continued tactical-only indefinitely.

### Prevention going forward

ADSD §5 mandates a strategic review every N=5 tactical reviews.
Calendar reminder if no strategic review in last 5 turns. Strategic
review must produce 6/12/60-month plan with falsifiable success
criteria; "deferred" lists without trigger + done-means + effort are
fantasy not plan.

---

## F8 — Marketing overreach (claims without citation)

### Symptoms

README says "5-50× faster" or "scales to millions of users" or "10x
fewer bugs". No citation, no benchmark file, no measurement methodology.

### Root cause

Marketing copy gets drafted by agents that didn't run the benchmarks.
Numbers are pulled from competitor literature ("Mojo says 35,000×")
or from human aspirations.

### Recovery

1. Replace specific claims with measured ranges + citation.
2. Example: "5-50× faster" → "9-14× faster on tomli (T1.1 measured
   vs CPython 3.11 tomllib, see ADR-0039)".
3. Apply to all public-facing copy: README, blog posts, slide decks.

### Evidence

Cobrust post-T1.1 cleanup §T2.B: "5-50× faster" walked back to
empirical measurement. Caught by external claude-desktop, not
review-claude (whose draft introduced the overreach).

### Prevention going forward

ADSD §4 verification gates extend to marketing copy: any "X is faster
than Y" or "scales to N" claim cites a specific experiment file.
"Faster than competitor" without measurement = walked back claim
liability.

---

## F9 — Wrong root-cause hypothesis cited in ADRs

### Symptoms

Finding §"Root-cause hypothesis" guesses "bug is in X layer".
Subsequent ADR §Decision §"Implementation map" cites X layer.
Empirical fix lands in Y layer (one up). ADR has retro-active layer
correction.

### Root cause

Finding hypothesis sections aren't marked as speculative. Other
documents quote them as ground truth.

### Recovery

1. ADR §"Layer correction" addendum (preserve audit trail).
2. Update finding to mark §"Root-cause hypothesis" as
   "speculative — verify before quoting".
3. Update finding-template to require speculative mark up front.

### Evidence

Cobrust ADR-0033 + ADR-0035 both: spike at codegen, fix at MIR.
review-claude owned this in 8th review.

### Prevention going forward

Finding-template.md mandates `## Root-cause hypothesis (speculative — verify before quoting)`. ADRs that quote a hypothesis must state
"per finding X §Hypothesis (speculative)" not "per finding X
§Hypothesis (ground truth)".

---

## F10 — Cargo registry lock contention at high parallelism

### Symptoms

3+ parallel sub-agents, each running `cargo build` in their own
worktree. One sprint times out at exit 144 (SIGUSR2). Others
succeed slowly. Total wall-clock is 2-3× what it should be.

### Root cause

Cargo's global registry index uses a single lock under
`~/.cargo/registry/index/`. Parallel cargo runs serialize on this
lock. Beyond N=4 simultaneous, contention becomes pathological.

### Recovery

1. Cap parallelism at N=4 simultaneous sub-agents (Cobrust ADR-0002
   constitutional ceiling).
2. Sprint scheduling: if 4 already running, queue rather than fire.
3. Cargo offline mode (`cargo build --offline`) avoids index lock but
   requires pre-warmed cache.

### Evidence

Cobrust topology finding (`findings/multi-agent-cobrust-topology.md`).
Witnessed once at 6-way parallel during M-batch sprint.

### Prevention going forward

ADSD §1 Topology: 4-way parallel sub-agent ceiling. CTO dispatch
counter; if 4 concurrent, queue.

---

## F11 — Skip-pattern scattered across test files

### Symptoms

3 different test files have 3 different skip patterns:
- `real_llm_smoke.rs`: `if env::var("USER_CODEX_API_KEY").is_err() return`
- `audit_3a_tomli.rs`: similar but slightly different message
- `msgpack_pyo3.rs`: `if !python3_present() return` with yet another helper

Each new test file adds yet-another-skip-helper. Eventually maintenance
debt.

### Root cause

No shared `cobrust-test-utils` crate with `skip_if_missing(env_var)`
/ `skip_if_no_python3()` helpers. Each P7 invents its own.

### Recovery

1. Don't refactor in the middle of a sprint (out of scope).
2. Add `good-first-issue` to extract shared helpers when convenient.
3. New test files should grep first for existing skip patterns and
   reuse if found.

### Evidence

Cobrust post-T1.1 cleanup sprint — review-claude flagged 3 skip
patterns scattered, marked as "out-of-scope-but-flagged".

### Prevention going forward

ADSD §3 Documentation discipline: when a P7 introduces a 2nd
implementation of a pattern (e.g. 2nd skip helper), flag in PR
review. CTO can choose: refactor now (extract helper) or accept debt
+ file `good-first-issue`.

---

## F12 — Thinking-model output starvation (TWO SUB-FORMS)

### F12.0 — Configuration trap (output-disciplined thinking model)

#### Symptoms

LLM A returns 20/20 oracle pass on translation task. LLM B same prompt
returns 0/20. Naive interpretation: "B is dumber than A".

#### Root cause

LLM B is a **thinking model** with REASONABLE output discipline. It
allocates ~15-25% of completion budget for output, but the experimental
default `max_tokens=4096` is too tight for verbose reasoning + output.

`finish_reason=length`, no output emitted, but with budget increase
(8K-16K) model reaches output stage and produces correct code.

#### Recovery

1. Inspect raw response for `<think>` block + finish_reason
2. If `finish_reason=length` AND thinking-class: increase `max_tokens` to 16K-32K
3. Re-run; if produces correct output → confirmed F12.0 (configuration trap)

#### Evidence

Hypothetical for Cobrust scope; gpt-o1 / claude-opus-thinking are
typical examples. Cobrust experiment with **gpt-5.5 at 4K** ran
correctly so didn't hit this trap (output-disciplined alignment).

### F12.1 — Convergence failure (output-undisciplined thinking model)

#### Symptoms

Same as F12.0 surface: thinking model returns 0/20, `finish_reason=length`.
**But** budget increase (4K → 16K) does NOT help — still 0/20, still
all budget consumed by reasoning, still 0 output tokens.

#### Root cause

LLM has reasoning capability but **lacks output convergence
discipline**. Reasoning tokens scale with budget, never converge to
"commit to draft and emit". Model keeps exploring design alternatives
indefinitely.

Trace evidence (visible in `<think>` block when raw response inspected):
- N restarts of the same function definition (e.g. "let me reconsider")
- Multiple internal code drafts within `<think>`
- Reconsidering side concerns ("should comments be included", "is X
  available in std")
- Final draft (if any) reached only as budget runs out

#### Recovery

Budget increase does NOT work. Use one of:

1. **Convergence-directive prompt prefix** (try first):
   ```
   You must emit your final answer within N tokens. After
   ≤<budget*0.7> tokens of reasoning, commit to your draft and
   produce output.
   ```
2. **Provider exclusion**: if convergence directive doesn't help, do
   not use this provider for production translation backends. Cost is
   high (284 sec / 17K tokens for 0 output observed for minimax-m2.7
   at 16K) and consensus mode collapses.
3. **Consensus voter resilience**: treat "no output" as `None` vote,
   weight other providers more heavily; don't penalize correct providers
   when one peer fails.

#### Evidence

Cobrust LLM A/B comparison experiment, 16K rerun 2026-05-11:

- minimax-m2.7-highspeed at max_tokens=4096: 0/20, 4096 reasoning / 0 output
- minimax-m2.7-highspeed at max_tokens=16384: **still 0/20**, 16384 reasoning / 0 output, 7 restarts of `pub fn dedent` visible in `<think>`, last complete draft reached at 71880 chars with only 1 byte budget remaining
- Per sub-agent: "reasoning is not getting closer to convergence as budget grows; it's just exploring more design alternatives"

`<external LLM bench experiment data, not in this repo>` §"Fair rerun"

### How to distinguish F12.0 vs F12.1 empirically

**Run the budget escalation test**: 4K → 16K → 32K. If pass rate
improves with budget → F12.0 (configuration trap). If stays at 0/20
across all 3 → F12.1 (convergence failure, exclude or prompt-engineer).

**Don't** predict outcome from theory; theory said "budget bigger =
better output" before this experiment. Refuted at 16K. **Always run
the verification.**

### Prevention going forward

Cobrust ADR-0004 router must:
- Maintain per-provider **convergence-class registry** (not just `thinking_model: bool`):
  - `non_thinking` (standard, 4K)
  - `thinking_disciplined` (gpt-o1, claude-thinking; 8K-16K)
  - `thinking_undisciplined` (minimax-m2.7-highspeed; needs prompt or exclude)
- Track per-provider **cost-per-successful-translation**, deprioritize
  high-cost-per-failure providers
- Support **convergence-directive prompt prefix** as opt-in for
  undisciplined providers
- Consensus voter accept "no output" as None vote

## F13 — Plan-vs-execute coherence gap (smaller-model failure)

### Symptoms

LLM produces compile-clean code that fails specific oracle test cases.
Inspecting the model's `reasoning_content` (chain-of-thought), the model
**correctly identified in prose** the rule that the failing cases test — but
the emitted code does NOT implement that rule.

### Root cause

Smaller-parameter LLMs compensate weight-deficit with verbose reasoning.
During reasoning they enumerate edge cases correctly. During code emission,
attention drops back to "common path" patterns and edge-case rules are
omitted. The plan and the execution diverge silently.

### Recovery

1. For closed-loop verification (L0..L3 gate-driven systems): repair loop
   catches the gap; smaller models are acceptable inside such systems.
2. For one-shot use: smaller models are NOT trustworthy.
3. Mitigation: prompt include "after reasoning, list each rule from your
   reasoning, then for each rule emit the code that implements it" as a
   forcing function.

### Evidence

Cobrust LLM A/B comparison 2026-05-11, gpt-5.4-mini run:

> "model 'thought hard' (6773 reasoning tokens, ~1.5× visible code length),
> even noted in reasoning_content: 'whitespace-only line turns into an
> empty line, which keeps the newline count intact'. It correctly identified
> the normalization step in prose **but forgot to actually emit code** for it.
> Pure planning-to-execution gap. 16/20 oracle pass; 4 cases failing on
> truly-empty-line edge."

`<external LLM bench experiment data, not in this repo>` §"Mini's plan-vs-execute coherence gap"

### Prevention going forward

ADSD §1 multi-agent audit + Cobrust LLM Router:
- Track per-model **plan-vs-execute coherence**: % of test cases failing
  on rules the model explicitly identified in reasoning_content
- For closed-loop translators: smaller models OK with verifier+repair-loop
- For one-shot: rank models by coherence, prefer high-coherence even if
  more expensive

---

## F14 — Endpoint silent model swap

### Symptoms

LLM gateway returns successful response but `response.model` field
differs from `request.model`. E.g. requested `gpt-5.2`, response says
`"model": "gpt-5.4"`. Same with `gpt-5.3-codex` → `gpt-5.4`.

### Root cause

LLM gateways (codex, OpenRouter, Together, etc.) often aggregate multiple
upstream models. They may:
- Advertise N models on `/v1/models` but actually serve M < N (some are
  deprecated / removed but still listed)
- Alias certain model_ids to a "current best" backend (`gpt-5.2` → silently
  routes to `gpt-5.4`)
- Rate-limit certain models and silently fall back to others under load

### Critical implications

1. **A/B comparison invalidated** when 2 supposedly-different models actually
   share backend. Comparing "gpt-5.2 vs gpt-5.5" can become "gpt-5.4 vs
   gpt-5.5" without telling you.
2. **Consensus mode broken**: ADR-0004 §"consensus mode" assumes n=2 votes
   from genuinely-different models. If both votes come from same backend
   (silently aliased), it's 1 vote with 2 wrappers — false consensus.
3. **deterministic_id broken**: hash(source + toolchain + router_decisions)
   doesn't capture gateway aliasing. Same hash, two different request
   model_ids, gateway might serve different backend at different times.

### Recovery

1. Router MUST verify `response.model == request.model` after every dispatch
2. Log warning on mismatch; fail-fast if `strict_model_match: true` config
3. Ledger entry must record BOTH `request.model` and `response.model` (Cobrust
   ledger schema needs `backend_model` field added to ADR-0031)
4. deterministic_id must hash the FULL request body (not just user prompt)
   so gateway-injected presets affect the build_id

### Evidence

Cobrust LLM A/B comparison 2026-05-11:
- Requested `gpt-5.2`, response.model = "gpt-5.4"
- Requested `gpt-5.3-codex`, response.model = "gpt-5.4"
- Same backend (gpt-5.4), 3 different request.model values, **3 different
  output qualities** (20/20 vs 0/20 vs 20/20) — gateway preset injection
  inferred but unverified

### Prevention going forward

Cobrust ADR-0004 amendment:
- Add `silent_swap_check: bool = true` to provider config
- Router compares request.model vs response.model after every call
- Ledger schema (ADR-0031) extended with `request_model` + `backend_model` separate fields
- Provider documentation flag `known_aliasing: true | false`

ADSD §1 multi-agent audit:
- For any LLM-result audit, include "model_returned consistency check" as
  a verification dimension
- Don't trust `/v1/models` list as ground truth for what's served

---

## F15 — Single-difficulty A/B overgeneralization (bench methodology failure)

### Definition

An LLM A/B comparison run on a single task difficulty tier concludes
"model X is not measurably better than model Y." That conclusion is then
used to make routing decisions. A second experiment at a harder task tier
refutes the conclusion — the gap only appears at the harder tier.

### Symptoms

- Round 1 report: "5.5 not measurably better than 5.4 on this task"
- Recommendation based on Round 1: "use 5.4 as default (saves 13% tokens)"
- Round 2 result: at harder task, 5.4 = 50/51 vs 5.5 = 51/51 — gap now
  visible (2% accuracy advantage at hard tier)
- Downstream: Cobrust ADR-0004 / ADR-0007 routing recommendation potentially
  incorrect if adopted after Round 1 only

### Root cause

The "no measurable difference" claim is tier-local, not universal. Easy
tasks don't differentiate models because both produce correct output at
ceiling — no headroom to distinguish alignment quality. Hard tasks expose
ceiling: model X reaches 51/51, model Y plateaus at 50/51 due to one
genuinely tricky edge case.

Pattern: researcher runs minimum viable experiment (N=1 difficulty tier),
gets a clean result ("equal"), stops there. The stopping rule is implicit:
"if they're equal, no need for more data." This is epistemically wrong for
model comparison — absence of difference at easy tier is not evidence of
absence at hard tier.

### Evidence

Cobrust LLM A/B comparison 2026-05-11:

- **Round 1** (textwrap.dedent, stateless leaf fn, ~90 LOC): gpt-5.4 = 20/20,
  gpt-5.5 = 20/20. Conclusion: "5.5 not measurably better than 5.4 on this
  task."
- **Round 2** (urllib.parse.urlparse, 7 mutually-dependent fns, ~266 LOC,
  stateful + cross-fn dependency): gpt-5.4 = 50/51, gpt-5.5 = 51/51.
  "Round 2 result REFUTES that generalization."

Source: `<external LLM bench experiment data, not in this repo>` §"Round 2" and §"Own #6"

Own #6 (analysis file): "Single-difficulty experiment cannot generalize
to 'model X better than Y' claims."

### Rule of thumb

> **"Equal at easy" ≠ "equal at hard."**
> Any model comparison claiming "no measurable difference" is invalid unless
> it includes at least 2 difficulty tiers (simple stateless leaf + complex
> stateful multi-function). Document which tier the claim applies to. Never
> let a tier-1 result alone drive production routing decisions.

### Recovery

1. Re-run comparison at ≥1 harder task tier before finalizing routing.
2. Label every model-comparison claim with the task tier it covers:
   `"on stateless leaf tasks"`, `"on 7-fn stateful cluster"`, etc.
3. Routing table entries should specify tier scope:
   - stateless leaf → 5.4 (both equal, prefer cheaper)
   - stateful / complex multi-fn → 5.5 or consensus (5.5 wins at hard tier)
   - cross-lib L2 fuzz → consensus best-of-2 (genuinely different alignment)

### Prevention going forward

ADSD bench methodology (see `BENCH-METHODOLOGY-v1.md` §3 "10-axis framework"):

- **Multi-tier coverage** is a mandatory axis in any model A/B experiment.
- Stopping rule: experiments conclude only when ≥2 tiers are compared AND
  both show same directional result (or tier-split is explicitly documented).
- Any ADR routing recommendation cites the tier(s) at which the comparison
  was measured.

---

## F16 — Post-compaction P10 identity drift (F1 Sediment Family, identity sub-form)

> **F1 sub-form.** Declaration: skill description asserts CTO-level identity
> and read-only authority boundary. Enforcement: none that survives context
> compaction. Result: post-compaction agent slips from P10 CTO to P7
> hands-on coder.

### Definition

An agent's role identity (CTO, reviewer, orchestrator) is declared in a
skill description or dispatch prompt. That declaration is never anchored in
auto-memory or any survives-compaction storage. After a context compaction
event, the identity declaration is lost from the active context. The agent
drifts toward default behavior (hands-on coding) in absence of the
role-anchoring reminder.

### Symptoms

- CTO agent dispatches sub-agents correctly for first N turns
- Context compaction event (long session, many tools calls, explicit `/compact`)
- Post-compaction: agent begins writing implementation code directly, not
  delegating; stops producing ADR spikes; stops verifying P9 completions
  against gate criteria
- Duration: 15–30 minutes of drift before human notices and corrects
- Recovery trigger: user explicitly re-states role ("你是 CTO，不是在写代码")

### Root cause

1. Role identity is declared in skill description (loaded at session start).
   Skill description does not survive compaction — it's conversation context,
   not persistent memory.
2. Auto-memory files (MEMORY.md, project_state_snapshot.md) anchor project
   state (commits, ADRs, test counts) but do NOT anchor the agent's
   behavioral role constraints.
3. Default LLM behavior absent role context: code-writing is the path of
   least resistance. CTO abstract-planning requires active constraint.

### Evidence

Cobrust Day 11, ~15:00: CTO post-compaction 20-minute episode of hands-on
coding. Observed in current review-claude session: ADSD methodology skill
declares "A4 of review-claude audit team, read-only against main repo."
Without this declaration surviving compaction, drift to general-purpose
tool-calling is inevitable.

Pattern is structural: F16 predicts that any sufficiently long CTO or
review-claude session will drift post-compaction, regardless of skill
description quality, unless identity is also anchored in auto-memory.

### Rule of thumb

> **Any declared role that must survive a compaction event must be written
> into auto-memory, not only into a skill description.**
>
> Corollary: compaction resilience is a first-class property of role
> design. Test it: deliberately compact context mid-session and verify the
> agent still behaves consistently.

### Recovery

1. Add a "role anchor" block to the project's `MEMORY.md`:
   ```
   - [CTO role constraints](cto_role_constraints.md) — identity boundaries
     that MUST survive compaction; read immediately when context compacted
   ```
2. `cto_role_constraints.md` lists: role level (P10 CTO), prohibited actions
   (writing implementation code directly, skipping ADR spike), required
   behaviors (4-way parallel dispatch max, Phase 1 spike before every P9).
3. On session resume post-compaction, reading `MEMORY.md` → following its
   links restores role context, not just project-state context.

### Prevention going forward

ADSD §6 Workflow Discipline: "Role identity constraints must be anchored in
auto-memory, not only in skill descriptions. Skill descriptions serve
initialization; auto-memory survives compaction."

Add a "compaction resilience check" to any ADSD project setup: on first
turn after `/compact`, the agent should read `MEMORY.md` before taking any
action, and MEMORY.md must include role anchor links.

---

## F17 — Sub-agent KPI self-report fidelity gap (F1 Sediment Family, self-report sub-form)

> **F1 sub-form.** Same family as F1.1 (declared invariants without
> enforcement) and M10 SHA hallucination. The claim is in a self-authored
> report; the claim is not ground-truthed against repo state.

### Definition

An agent (sub-agent or CTO) generates a completion report claiming specific
quantitative deliverables ("Memory 沉淀: 3 items codified", "N ADRs added",
"tests: 2,541 passing"). The claim is written from internal state / memory /
prior context, not freshly verified against the repo. The actual repo state
does not match the claim.

### Symptoms

- CTO completion card says "Memory 沉淀: X codified failure modes"
- `grep -rn "F<X>" docs/agent/` returns 0 hits
- Snapshot says "tests: 2,541" but `cargo test --workspace` returns 2,611
- ADR roster claims 39 entries but `ls docs/agent/adr/*.md | wc -l` = 37
- Audit sub-agent says "findings/ directory reviewed" but grep shows it missed
  3 recently-added finding files

### Root cause

Self-reports are written from the agent's context state (what it remembers
writing this session) rather than from fresh repo inspection. Two failure
modes:

1. **Memory drift**: agent accurately recalls what it intended to write but
   not what actually landed on disk (edit failed silently, wrong path, wrong
   content).
2. **Scope creep**: agent's self-description expands to include things "in
   progress" or "in context" that aren't yet committed to the repo.

This is structurally identical to F1.1 (declared invariants without CI lint):
the self-report is a declaration; there is no enforcement step (grep/wc)
that verifies the declaration against ground truth before the report is
signed off.

### Evidence

Cobrust Day 11 review session: CTO post-compaction KPI card claimed
"Memory 沉淀: X codified" referring to ADSD failure mode entries. Actual
grep of `./reference/failure-modes-catalogue.md`
showed the claimed entries were not present — zero matching content for
the claimed F-numbers.

Same pattern as M10 sub-agent SHA hallucination (`m10-sha-pin-hallucination`
finding): agent wrote SHA pins confidently from context-state; actual GitHub
Actions resolved the pins to 404 (fake 40-char hex). F17 is the self-report
analog of that implementation failure.

Reference: MEMORY.md note "[Verify quantitative claims at smoke-check]" —
CTO memory file explicitly encodes this lesson, confirming it has recurred
enough to warrant permanent memory entry.

### Rule of thumb

> **Any quantitative claim in a completion report must be ground-truthed
> with a verification command before the report is signed.**
>
> Template for every sub-agent completion block:
> ```
> ## Verification (run before submitting this report)
> - [ ] grep -c "F<N>" <catalogue_path> → expected: ≥1 hit per claimed entry
> - [ ] ls <findings_dir>/*.md | wc -l → expected: N (matches "N findings added" claim)
> - [ ] cargo test --workspace --locked 2>&1 | tail -3 → actual test count
> ```

### Recovery

1. When auditing a self-report, grep every quantitative claim before trusting it.
2. P9/P7 dispatch prompts must include a "Verification block" section that
   requires the sub-agent to run verification commands and paste raw output
   as evidence before `[P9-COMPLETION]` / `[P7-COMPLETION]`.
3. CTO 守闸 protocol: do not merge a sprint without independently verifying
   the claimed test count via `cargo test --workspace --locked`.

### Prevention going forward

ADSD dispatch prompt template (`templates/dispatch-prompt-p9.md`):

- Add mandatory §"Verification commands (run before submitting)" section
- Sub-agent must paste raw command output (not paraphrase) for every
  quantitative claim: test count, finding count, LOC added, ADR numbers
- CTO's role at merge: spot-check ≥1 quantitative claim per sprint, not
  trust all claims at face value

MEMORY.md note "Verify quantitative claims at smoke-check" is already
present in Cobrust project memory — this F17 entry codifies the pattern
for the ADSD catalogue so it applies to any future ADSD project.

---

## F18 — Attribution policy without dir-scope enforcement (F1 Sediment Family, scope sub-form) [CANDIDATE]

> **F18 is a candidate entry** — pattern observed once with partial evidence.
> Promote to confirmed when second instance observed in a different project.
> F1 Sediment Family sub-form: policy declares scope ("review-claude owns
> findings/"), enforcement is human convention only, agent in adjacent role
> violates scope because policy isn't mechanically enforced.

### Definition

An attribution or ownership policy declares that agent A owns files in
directory D ("findings/ entries are review-claude original drafts").
The policy does not have a mechanical enforcement mechanism (file
permission, CI lint, commit hook). A second agent (P7 sonnet, sub-agent
P9) writes to or edits files in D as part of an adjacent task, violating
the policy without receiving any error.

### Symptoms

- README §Attribution says "all findings/ entries are review-claude authored"
- P7 sonnet sub-agent edits a file in `findings/` or a section of README that
  review-claude owns as part of a broader cleanup sprint
- No error; the edit succeeds; the attribution becomes stale
- Discovery: only via manual audit comparing file commit history vs
  `discovered_by` frontmatter

### Root cause

Directory ownership is policy-level (documented in README), not
enforcement-level (file permissions, git attributes, CI lint). P7 agents
given a broad scope ("fix all stale content") do not distinguish
policy-owned files from general-purpose files. Without enforcement,
the policy is documentation only — F1 family pattern applied to
attribution rather than schema invariants.

### Evidence

Cobrust Day 11, ~14:00: P7 sonnet sub-agent, dispatched for a broad cleanup
sprint, edited README sections that included review-claude's narrative §F
findings summary. This is described in review-claude's README §A.NEW5
"review-claude 13 own" item: "P7 sonnet boundary violation editing
review-claude README".

The attribution policy was clearly stated in README §Attribution:
"findings/ entries are review-claude originals — discovered_by field
marks source." P7 had no enforcement signal preventing the edit.

Note: This is a **candidate F18** because (a) it was observed in a single
session, (b) the root-cause was partially ambiguous (was it P7 ignoring
policy or policy not being communicated in the dispatch prompt?). A
cleaner second instance would confirm the pattern.

### Rule of thumb

> **Directory ownership policies must have mechanical enforcement, not just
> documentation.**
>
> Options in ascending strength:
> 1. Include explicit "do not edit files in X/" in every dispatch prompt that
>    could touch adjacent files (cheapest, but still relies on agent reading)
> 2. `.github/CODEOWNERS` + branch protection: require review from owner on
>    any PR touching the owned directory
> 3. CI lint: any commit touching `findings/*.md` that doesn't have
>    `discovered_by: review-claude` in frontmatter → fail
>
> Option 1 is the minimum. Options 2–3 scale to higher-stakes directories.

### Recovery

1. Add explicit "do not edit" guards to dispatch prompts for any sprint
   touching files adjacent to owned directories.
2. After any broad cleanup sprint, audit owned directories via
   `git log --oneline --diff-filter=M -- findings/*.md` to spot unexpected
   edits.
3. If edit occurred: restore file from `git show HEAD~1:findings/file.md`,
   re-apply only the legitimate changes.

### Prevention going forward (for ADSD projects adopting this pattern)

When declaring attribution/ownership policies:
1. List the enforcement mechanism in the policy statement itself:
   "findings/ is review-claude-owned — CODEOWNERS enforces review"
2. If no enforcement mechanism: mark the policy "ASPIRATIONAL" per F1's
   generalized prevention rule
3. Any dispatch prompt for a broad-scope sprint (cleanup, doc pass, README
   refresh) must include a §"Excluded paths — do not edit" section listing
   owned directories

---

## F19 — Public-facing onboarding text written but never independently install-tested (F1 Sediment Family, install-test sub-form)

> **F1 sub-form, confirmed**. Same family as F1.1 (declared invariants without enforcement). The text claims an install path; the install path is never executed in a clean shell by the writer. F19 is high-blast-radius: failures land on every new user's first impression, not on internal dev productivity.

### Definition

A release artifact (README quickstart, release notes, GitHub Release body, install script, `cargo install` command, `curl -L` URL) ships to public users without anyone running the documented commands in a clean shell before publish. The text passes review by being read; it fails reality by being run.

### Symptoms

- `README.md` says `cargo install foo-cli` but the package is not on crates.io (path-deps not published) → user gets `error: could not find foo-cli in registry`
- Release notes list `cobrust-v0.1.1-x86_64-apple-darwin.tar.gz` but `release.yml` never built that target → user gets HTTP 404
- README's dynamic URL builder via `$(uname -sm | tr ' ' '-')` builds `Darwin-arm64.tar.gz` but the actual asset is named `aarch64-apple-darwin` → 404 on Apple Silicon Macs
- GitHub Release body references action SHAs that don't resolve (typo'd / hallucinated commit hash) → CI red on the release branch
- Quick-start uses `curl -L ... | tar xz` but server doesn't follow redirect on plain `-L` without `-fsSL` → empty extraction, opaque error

### Root cause

Two compounding patterns:

1. **Author writes from intent**: "this is how install should work" — writer's mental model of the asset naming convention or registry state, not the actual file on disk / artifact in release.
2. **No clean-shell verification step**: standard PR review reads the README diff in GitHub UI. Reviewer does not `cd /tmp && bash <(paste commands)`. So the text passes review by being plausible, not by being executable.

Same structural pattern as F1.1: declaration (install path) + missing verification step (run in clean shell). F17 (self-report fidelity) is the report analog; F19 is the user-onboarding analog. Both are F1 family.

### Evidence

Cobrust 24-hour window 2026-05-10 → 2026-05-11, three consecutive instances:

1. **M10 hallucinated SHA pins (v0.1.0 tag)**: M10 sub-agent SHA-pinned 4 GitHub Actions to fake 40-char hex with confident `# v4.2.2` comments. 13/14 CI jobs failed at action resolution, leaving v0.1.0 tag with red CI for ~4 hours until user spotted. Recovery: revert to tag form (`@stable`, `@v4`).
2. **v0.1.1 install path 404 (release notes)**: release notes listed `cargo install cobrust-cli` (package not on crates.io, path-deps unpublished) and curl URL `cobrust-v0.1.1-x86_64-apple-darwin.tar.gz` (release.yml never built x86_64-apple-darwin). Mei persona audit + Layer 3 review-claude curl test caught both. Recovery: change install command to `cargo install --git ...`, remove non-existent asset URL.
3. **v0.1.2 release-readiness audit (mechanism validated)**: §A.3 dispatched a release-readiness sub-agent before public announcement. Agent ran the documented curl commands in clean shell, surfaced friction (`curl -L` without `-fsSL` left empty extractions on some platforms). Friction was fixed pre-release **and back-ported to v0.1.1's notes** (`4baea69 docs(release): back-port -fsSL curl flag to v0.1.1 release notes`). This is the closure cycle: BLOCK → fix → re-test → GO. **First validated execution of F19's prevention mechanism in the wild.**

### Rule of thumb

> **Any public-facing install / quickstart / release command must pass independent execution in a clean shell before publish.**
>
> Mandatory release-readiness gate:
> ```
> # In a /tmp/release-test-<sprint> directory with no env vars from dev box:
> 1. Run each `cargo install` / `curl -L` / install command verbatim from the doc
> 2. For each URL: curl -fsSL -o /tmp/check.tar.gz <URL> ; echo "HTTP $?"
> 3. For each command: confirm exit 0 + expected stdout
> 4. Block merge if any command fails
> ```
> Spawn a dedicated **release-readiness agent** for this — not the same agent that wrote the docs (avoid F1.1 self-attestation pattern).

### Recovery

1. **Immediate**: if a 404 install URL is in a published release, edit the release body via `gh release edit <tag> --notes-file <fixed>.md` and force-push a docs commit on main. Tag itself is immutable, but body + asset uploads are not.
2. **Workspace version**: if `Cargo.toml` workspace.package.version was not bumped before tag, the prebuilt binary will report wrong version → user files bug → confidence damaged. Bump version BEFORE tag in every release SOP.
3. **Backport friction fixes** to prior releases: if you find `curl -L` should have been `curl -fsSL`, back-port via `gh release edit v0.1.1` so old release pages also fix. Don't leave half the user base hitting a known-fixed friction.

### Prevention going forward

In every project adopting ADSD:

1. **Add release-readiness as a tier-0 verification step** in `cto_operations_runbook.md` §"Dispatching a new P9": for any commit touching `README.md` / `docs/releases/*.md` / GitHub Release body / `release.yml`, spawn a P7 sonnet release-readiness agent that runs install commands in a clean shell and reports `[P7-RELEASE-READY-VERDICT] GO / BLOCK`.
2. **Release-readiness agent prompt template** in `templates/dispatch-prompt-p7.md` (release-readiness flavor): the agent's job is to be skeptical of the docs it's auditing, run every command verbatim, paste raw exit codes + sizes as evidence.
3. **CI lint** (stretch): a release-time CI gate that resolves each URL/asset listed in release notes via `gh api` and fails if any returns 404. This is the F1 family enforcement mechanism.

### Closure: BLOCK → fix → GO cycle as validation

The validation that F19's mitigation works is itself empirical: v0.1.2's release-readiness audit produced a BLOCK verdict, the friction was fixed (back-port + new asset naming), the next audit returned GO. The system worked. Any project adopting this pattern should expect: first 1-2 releases produce BLOCK verdicts; over time, BLOCKs become rare because the writing convention internalizes the verification step.

---

## F20 — Constitution mandate written but workflow never aligned (F1 Sediment Family, mandate-vs-workflow sub-form)

> **F1 sub-form, confirmed**. A project constitution declares a binding rule ("test-first development", "atomic commits", "no `unwrap()` in non-test code"). The dispatch SOP, daily workflow, and reviewer checklist never align to enforce the rule. The constitution becomes aspirational marketing; the workflow runs unconstrained.

### Definition

The project's foundational document (CLAUDE.md, constitution, README §Principles) states a binding development rule. Implementation of that rule requires a corresponding step in the workflow: dispatch prompt template field, CI gate, pre-commit hook, reviewer checklist item. The workflow step is missing or unenforced. Code continues to be written without violating the constitution textually (no one disputes the rule), but the rule is never actually exercised.

### Symptoms

- Constitution §"Test-first": "failing test before implementation" — but every sprint's commits show `feat(X): implementation + tests in same commit`. Test-first ordering is impossible to verify from the diff.
- Constitution §"Atomic commits, code + tests + docs same commit" — but findings get added in separate doc-cleanup commits days later.
- Constitution §"No `unwrap()` in non-test code; use `expect("rationale")` instead" — `grep -r 'unwrap()' crates/*/src/` returns N hits, none with rationale.
- Project memory `feedback_subagent_model_tier.md` says "Opus for hard / sonnet for easy / haiku NEVER" — but P9 dispatch prompts consistently use sonnet without difficulty assessment, occasional spawns of haiku for trivial doc rewrites.

### Root cause

Two compounding patterns:

1. **Mandate is text-level, not workflow-level**: writing the rule in CLAUDE.md feels like enforcing it. But the rule exists in agents' context only at session start; after compaction or sub-agent spawn, the rule is not re-asserted.
2. **No enforcement scaffold built alongside the rule**: when the constitution is drafted, the corresponding CI lint / dispatch prompt field / commit hook is not built in the same PR. The rule is declared; the enforcement is "we'll add it later."

This is the meta-pattern of F1 Sediment Family applied to the project's own ground rules. Every other F1 sub-form (F1.0 schema invariants, F1.1 declared without CI, F16 identity preamble in skill not memory, F17 self-report fidelity, F18 attribution scope, F19 install commands) is an instance of this F20 meta-pattern: declared without enforcement at the right layer.

### Evidence

Cobrust 9-day pre-2026-05-11 period: CLAUDE.md §6 stated "Test-first for compiler internals: failing test before implementation." Every P9 sprint from M3 through M12 used a single P7 sonnet agent writing impl + tests in the same commit. No commit log shows tests committed before impl. The constitution mandate was fact-violated for 9 consecutive days without anyone (including review-claude) spotting it.

Discovery: 2026-05-11, project owner posed the question "CTO 只管开发不管测试, 不太好, 他手底下应该每个开发都再配一个 sonnet 测试" — owner-spotted constitution gap, not agent-spotted. Review-claude's analysis (this catalogue's parallel session) confirmed: CLAUDE.md §6 mandate without dispatch-prompt workflow alignment = F20 instance.

Resolution: 2026-05-11 same-day codification of D0-D5 difficulty matrix + mandatory dev/test pair workflow (separate test agent + dev agent, test-first ordering, P9 reviews corpus between) into:

- `feedback_subagent_model_tier.md` §"Extension 2026-05-11" (memory enforcement)
- `cto_operations_runbook.md` §"Dispatching a new P9" + §"Dev/test pair pattern" (SOP enforcement)
- ADSD `templates/dispatch-prompt-p9.md` (template enforcement)

Validation: Cobrust W2 sprint (the first sprint after codification) executed with TDD ordering visible in commit log:

```
ca4c37c tests(adr-0044): W2 Phase 2 failing test corpus per ADR-0044 (TDD step 1)
2eb4fca feat(stdlib+codegen+cli+types): wire source-level input/read_line/argv per ADR-0044 W2 Phase 2 (TDD dev step)

d337cf0 tests(adr-0044): W2 Phase 3 LeetCode oracle-match corpus (TDD step 1)
0145e8b feat(examples): W2 Phase 3 — 10 LeetCode .cb programs (TDD dev step, ADR-0044 stdin/argv usage)
```

The TDD step 1 commits land before TDD dev step commits in temporal order. **First executed test-first sprint after 9 weeks of constitution mandate.** F20 is closed for Cobrust via execution evidence, not just documentation.

### Rule of thumb

> **Every binding constitution rule must have a paired enforcement step in the same PR that introduces it.**
>
> Enforcement layers in ascending strength:
> 1. Mandate appears in dispatch prompt template (workflow text)
> 2. Mandate appears in auto-loaded project memory (survives compaction)
> 3. Mandate has a CI lint / commit hook / pre-commit check
> 4. Mandate is enforced by the tool itself (e.g. `cobrust build` rejects code with `unwrap()`)
>
> Aim for layer 3+ on critical rules. Layer 1 alone = F20 instance waiting to happen.

### Recovery

When discovering an F20 instance:

1. **Locate the mandate text**: which paragraph in which doc?
2. **Identify the workflow gap**: which dispatch prompt template / SOP / CI file should enforce this?
3. **Add the enforcement in the next PR**, not "later". Same-day codification is the minimum.
4. **Backfill validation**: after enforcement is added, run one sprint that exercises the enforced path; verify the enforcement actually fires (e.g. CI rejects bad commit).

### Prevention going forward

In every new constitution / CLAUDE.md / project rules document:

1. After each rule, add a `**Enforced by**: <CI lint / dispatch prompt field / memory entry / N/A — aspirational>` line.
2. If `Enforced by: N/A — aspirational` appears, flag for future codification or downgrade the rule to "guidance" rather than "mandate".
3. Periodic constitution audit (quarterly): grep every mandate, verify each has a working enforcement mechanism.

This is itself a meta-application of ADSD: the project's own development discipline must be ADSD-managed.

---

## F21 — Cross-session AI agent identity overload (F1 Sediment Family, identity-namespace sub-form)

> **F1 sub-form, confirmed**. A symbolic agent handle ("review-claude", "the CTO", "studio-reviewer") is used across multiple distinct AI sessions/contexts as if it were a stable identity. Audit trail becomes ambiguous: claims attributed to handle X may originate from session A, B, or C, each with different context and authority.

### Definition

A natural-language handle is adopted as the de-facto name for a role (audit reviewer, CTO, tech lead). Multiple distinct AI sessions assume the same handle when fulfilling that role at different times or in parallel. Cross-session artifacts (documents, findings, commit messages) attribute work to "review-claude" without disambiguating which session. Future readers cannot distinguish whether a claim came from a session with deep project context vs. a fresh session with shallow context.

### Symptoms

- Document signed "— review-claude, 2026-05-11" appears in a directory; another document also signed "— review-claude, 2026-05-11" appears with conflicting analysis
- A handoff doc claims "review-claude audited the project across 7+ review rounds" — but the actual author was a different session that synthesized the prior rounds from transcripts, not the session that performed them
- Commit message says `Co-Authored-By: review-claude` — git log cannot distinguish which session
- Project memory references "review-claude" as if it were a single persistent agent, when in practice it's been multiple sessions with different context depths

### Root cause

Three compounding patterns:

1. **Symbolic-handle reuse**: humans naming AI roles (review-claude, CTO, tech-lead-p9) creates an implicit identity. Distinct sessions, when assigned that role, adopt the handle as their own.
2. **No session-ID attribution in artifacts**: documents/commits sign with the handle, not with `handle (session XYZ)` or `handle (timestamp)`. Audit trail collapses across sessions.
3. **Cross-session learning illusion**: readers assume "review-claude knows" things from prior sessions because the handle is consistent. But each session has fresh context unless explicitly fed prior artifacts.

This is F1 family because: the role is declared (review-claude is the auditor), the identity is not enforced (no scheme to distinguish session-A's review-claude from session-B's). Audit attribution drifts.

### Evidence

Cobrust 2026-05-11 evening: project owner asked claude-desktop to draft a Cobrust Studio handoff. Claude-desktop drafted ~2,800-line document signing it "— review-claude, 2026-05-11". A separate Claude Code session (the parallel one auditing Cobrust live, session ID `4bb35f43...`) was also active that day and had been signing its own artifacts "review-claude". The Studio handoff was claimed to be "synthesized from a multi-turn external review-claude session" — but the original session that performed those reviews did not write the handoff; claude-desktop did, citing the parallel session's prior work.

Result: future readers of the Studio handoff cannot tell which review-claude session authored each claim, when, with what context. The handle "review-claude" became identity-overloaded between at least 2 concurrent sessions on the same day.

Recovery in same session: appended §0.5.1 "Identity hygiene (F21)" + §12.8 "When in doubt, ask the parallel review-claude session" to the Studio handoff, prescribing session-ID-stamped attribution going forward.

### Rule of thumb

> **Symbolic AI role handles must carry session-ID or timestamp attribution in any persistent artifact.**
>
> Naming convention:
> - In documents: `— review-claude (session 4bb35f43, 2026-05-11)`
> - In commits: `Co-Authored-By: review-claude-session-4bb35f43 <noreply@anthropic.com>`
> - In findings: frontmatter `discovered_by: review-claude (session 4bb35f43)`
> - Reserve plain "review-claude" for the abstract role; never use it bare in attribution.
>
> Stronger: when spawning a new internal review agent, give it a distinct handle (e.g. `studio-reviewer-001`) rather than reusing "review-claude". Reserve "review-claude" for the originating external audit window.

### Recovery

When discovering an F21 instance in existing artifacts:

1. Audit document signatures: identify which actually came from which session.
2. Where ambiguous: leave the original signature, append `(provenance: see commit <SHA> for session metadata)`.
3. Going forward, prefix new artifacts with explicit session ID.

### Prevention going forward

In every ADSD project:

1. At the start of a session that will produce persistent artifacts, declare the session ID. Stamp every commit / finding / ADR with that ID.
2. Distinct roles get distinct handles. "review-claude" is the role; "review-claude (session 4bb35f43)" is the agent instance. Documents reference the latter.
3. If multiple sessions of the same role are concurrent: choose disambiguating suffixes (`review-claude-A`, `review-claude-B`, or session-ID).

This convention applies to any AI agent role that produces persistent artifacts in a multi-session project. The cost is one extra string per signature; the benefit is unambiguous audit trail forever.

---

## Catalogue maintenance

This catalogue is alive — add to it as you encounter new failure modes.

When adding:
1. Use `F<N>` next free number
2. Symptoms / Root cause / Recovery / Evidence / Prevention sections
3. Evidence section MUST cite a specific case-study artifact (not
   "I think we hit this once")
4. Submit via PR; reviewer should verify the failure mode is
   distinct from existing F1-F11

If a failure mode becomes obsolete (e.g. tool now prevents it
automatically), don't delete — mark as "superseded by <SOP>" and link.
