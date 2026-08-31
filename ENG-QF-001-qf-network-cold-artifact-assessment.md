---
id: ENG-QF-001
uns_version: 0.1
type: engagement
title: "QF Network — Cold Artifact Assessment (Diagnostic) v1"
domain: triarch
status: ready
created: 2026-05-21
evidence_status: yellow  # desk-pass findings logged 2026-05-21
priority: high
tags: [engagement, diagnostic, qf-network, blockchain, dependency-risk, dao]

pipeline:
  stage: intake
  next_stage: pre-hypothesis

relationships:
  implements:
    - "META-CAA-001"   # this engagement instantiates the CAA spec
  references:
    - "github.com/QuantumFusion-network/qf-solochain"
    - "github.com/QuantumFusion-network/qf-polkavm-sdk"
    - "telemetry.qfnetwork.xyz (live testnet signal)"
  related:
    - "FSG/Triarch QF-Network-compatibility requirement (attestation anchor)"

human_gates:
  - id: HG-QF-BIAS-001
    state: pending
    owner: Ahknu
    blocks_transition: "pre-hypothesis -> assessment"
    rationale: >
      Assessor holds a position in QF (on-chain participation, belief in thesis).
      Gut is compromised by definition. A cold second reader MUST be assigned to
      the go/no-go conclusion before it is trusted.
  - id: HG-QF-CLIENT-PAYS-001
    state: pending
    owner: Ahknu
    blocks_transition: "assessment -> any-repair-scope"
    rationale: >
      Resolve who is the client and who funds the engagement on turn one.
      QF has funding instability; if Triarch eats cost it is a doubling-down on a
      held position, not consulting. Token/equity payment deepens concentration
      risk. This must be answered before any work past assessment.
  - id: HG-QF-SCOPE-LOCK-001
    state: pending
    owner: Ahknu
    blocks_transition: "assessment -> repair"
    rationale: >
      Inherited from META-CAA-001 scope gate. Assessment does NOT auto-escalate to
      repair. Repair is a separate, separately-funded engagement requiring this
      gate to clear explicitly.

x_triarch:
  billable: true
  engagement_class: diagnostic
  client: "Triarch/Ahknu (adoption frame) — see HG-QF-CLIENT-PAYS-001"
  deliverable: "QF viability read + remediation-cost-map + DAO-readiness + go/no-go"
  scope_gate: "ASSESSMENT ONLY"

x_fsg:
  formation: small-team
  tempo: medium
  zone: middle
  kernel_invariants_touched: []

lifecycle:
  events:
    - "2026-05-21 created — instantiated from META-CAA-001 for QF Network read"
    - "2026-05-21 desk-pass findings logged — 5 preliminary findings, A3 risk downgraded, A4 moat-pivot surfaced"
    - "2026-05-21 v1 — done-definition locked (viability then cost), adoption frame, bias reader = named teammate TBD, Stage 1 verdict LEAN-YES provisional"
---

# ENG-QF-001 — QF Network Cold Artifact Assessment

## Pre-Hypothesis Node (state the prior before the read)

**What we expect to find, and why — logged for bias-checking:**

QF appears to be *running but not hardening*. The testnet produces blocks
(telemetry endpoint live, faucet exists, ~1,078 commits, 21 releases through
April), but every maturity signal reads early-stage: solochain at v0.1.32,
custom "SPIN" consensus at v0.1, a barely-born PolkaVM contract SDK, and an
effective bus factor of ~2 maintainers. The hypothesis is that the break is
**funding + headcount + protocol depth**, not a bug — i.e. a class of problem a
methodology can *map* but cannot *resolve* without funded specialists.

This prior is held by an assessor with a position in QF. **It must be checked by
a cold second reader (HG-QF-BIAS-001) before the conclusion is trusted.**

## Evidence Inventory

- `qf-solochain` (Rust, Substrate/Polkadot-SDK fork, custom SPIN consensus)
- `qf-polkavm-sdk` (smart-contract layer, ~3 stars — maturity flag)
- `polkadot-js-apps` fork, `faucet`, `qf-squid`, `substrate-telemetry` fork
- Commit history, release cadence, issue/PR activity
- Comms exhaust: X (@theqfnetwork), org discussions, contributor graph
- On-chain: validator set, wallet/token distribution
- Spec / whitepaper (locate and read)

## The Five Assessment Questions → Six Axes

| Your question | CAA Axis | Owner (RACI) | Methodology status |
|---|---|---|---|
| Where are we currently? | A1 Code/Arch + A2 Maintenance | TBD | Ported — solid |
| What are the gaps? | A1 + A2 + A3 | TBD | Mixed (A3 partial) |
| What would make it function? | A1–A3 cost-of-remediation | TBD | Ported deliverable |
| What's needed to go DAO? | A5 Governance/DAO design | **FSG-native** | Build, not port |
| Who plays what role? | RACI (this table) + A4 read | TBD | Native framing |
| *(reflexive)* Should we couple? | A6 Dependency risk | **FSG-native** | Native framing |

## Per-Axis Instantiation for QF

- **A1 Code & Architecture** — Rust quality, test coverage on consensus/runtime,
  coupling to upstream Parity (how much drift, how painful are runtime upgrades).
- **A2 Maintenance Health** — commit/release cadence trend, contributor diversity,
  bus factor as a hard number, issue responsiveness. Is it decelerating?
- **A3 Protocol Soundness** — **the deep one.** SPIN consensus soundness,
  finality/upgrade model, validator + token distribution (centralization),
  PolkaVM contract-layer readiness. *This is where "physics-based governance
  protocol, try me" gets tested. Determine if it maps to an adjacent solved
  problem or needs a distributed-systems specialist. Do not fake the verdict.*
- **A4 Forensic Intent** — read the team's real state from comms + commits. Are
  they actively building, coasting, or winding down? Intent ≠ press.
- **A5 DAO Readiness** — what governance must exist for QF to function as a DAO:
  token-holder rights, on-chain upgrade governance, treasury, validator
  incentives. FSG kernel-invariant thinking applies directly here.
- **A6 Dependency Risk** — Triarch/FSG currently treat QF as the attestation
  anchor (outbound ZK proofs for kernel integrity / governance attestation).
  Single-vendor exposure on an early, thinly-maintained chain under a compliance
  product's trust layer. **Decouple recommendation likely regardless of A1–A5
  outcome:** put attestation behind an interface so QF is one swappable backend.

## Engagement Definition (v1 — locked 2026-05-21)

**Decision this feeds:** Both — Stage 1 viability go/no-go, THEN Stage 2 cost scope.
**Stage 2 is gated behind a Stage 1 YES.** No cost work until viability clears.
**This session:** desk-pass only. No code clone. Live 100ms test is out of scope here.
**Bias reader (HG-QF-BIAS-001):** a named teammate — name TBD by Ahknu. Gate stays
OPEN until that person signs the Stage 1 verdict.

**Frame:** adoption/takeover, not third-party diligence. Funding = dropped (their
problem, possibly our leverage). Original HVM2 moat = dropped as risk; survives only
as a code-hygiene check (Finding 2 / Job 4). Primary axes are now A1/A2 (inheritance
cost) and A3 (does 100ms hold).

### The four verification jobs (what hardens this past desk-pass)
| # | Job | Confirms | Doable this session? | Confidence until done |
|---|---|---|---|---|
| 1 | Read SPIN pallet + validator election | Finding 1 | No (desk-pass only) | 🟡 |
| 2 | 100ms under real validator set | "still special" | No (no testnet here) | 🔴 Unknown |
| 3 | Inheritance-cost read (commit/contrib + code health) | Findings 4/5 | No (desk-pass only) | 🟡 |
| 4 | HVM2 residue check | Finding 2 | No (needs clone) | 🟢 once cloned |

---

## STAGE 1 — Viability: Provisional Desk-Pass Verdict (v1)

**Verdict: LEAN-YES on adoptability — PROVISIONAL.** Nothing in the desk pass is a
hard blocker to adopting QF as a substrate. The real gate is inheritance cost
(Stage 2) and the 100ms-under-load test, neither resolvable at desk-pass depth.

| Element | Read | Confidence |
|---|---|---|
| Finality (GRANDPA, not homegrown) | De-risked | 🟡 (README, not code) |
| SPIN leader-election soundness | Probably fine, unread | 🟡 |
| 100ms block time as real property | Claimed, unproven | 🔴 Unknown |
| Buildability (Substrate/PolkaVM) | Ordinary early-chain risk | 🟢 |
| Inheritance cost (can we carry it) | THE open question | 🔴 Unknown |
| HVM2 cleanly removed | Likely, unverified | 🟡 |
| Funding/treasury | Out of scope by decision | n/a |

**This verdict is not final until the named bias reader signs it (gate open).**

---

## Preliminary Findings — Desk Pass (NOT the full assessment)

> Status: first read from public artifacts only. No repo clone, no code read, no
> on-chain analysis yet. Subject to HG-QF-BIAS-001 (cold second reader). These are
> hypotheses with evidence, not conclusions.

### FINDING 1 — SPIN is far narrower than the "scary custom consensus" prior. *(A3 — DOWNGRADE RISK)*
The runtime README states SPIN is a PoS-based leader-election mechanism that grants
single block-authoring rights over a fixed sequence of slots, **with standard
GRANDPA finality**. This matters enormously: the genuinely dangerous surface on any
chain — finality — is **not homegrown here**. It's Parity's proven, audited GRANDPA
gadget. SPIN is "who authors the next N blocks" (a slot/leader scheme), not "how the
chain agrees on truth." That is a much smaller, much more mappable problem than
"novel consensus from scratch." **My earlier read overstated this. Correcting it.**
Residual A3 risk drops from Critical toward Medium pending a code read of the SPIN
pallet and validator-election wiring.

### FINDING 2 — The original moat (HVM2 parallel compute) appears to have been dropped. *(A4 — HIGH)*
QF's founding thesis (Whitepaper V1 "Genesis," Aug 2024) was built on **HVM2** —
Victor Taelin / Higher Order Co's interaction-combinator GPU evaluator — plus the
**Bend** language, with off-chain parallel compute verified on-chain via ZK proofs.
That was the differentiator. The *current* qf-solochain runtime shows **no HVM2** —
it's a PolkaVM / pallet-revive Substrate chain. Public messaging pivoted to "PolkaVM
L1" around late 2024. **The headline moat is gone or shelved; what remains is a
competent but not unique PolkaVM Substrate L1.** This is the single most important
strategic finding: you may be evaluating a project whose original reason-to-exist was
abandoned, leaving a me-too chain. A4 forensic work must confirm whether HVM2 is
dead, paused, or moved off the critical path.

### FINDING 3 — Funding ran through an ERC-20 token sale. *(A4 / A6 — HIGH, maps directly to the "money bs")*
A QuantumFusion (QF) ERC-20 exists on Ethereum with standard token-sale machinery
(AMM pair, max-buy/sell limits, treasury wallet). Stated use of proceeds: blockchain
dev, R&D, community. **This is where the "crybaby money bs" actually lives** — the
project is token-sale-funded, so runway is a function of treasury + token health, not
recurring revenue. If proceeds are depleted or the token has bled out, maintenance
deceleration is the expected symptom — which is exactly what "running but not
hardening" looks like. Confirm: treasury balance, token chart, sale vintage.

### FINDING 4 — Maintenance cadence looks thin; bus factor stands. *(A2 — MEDIUM/HIGH)*
Org repos show low past-year commit volume on the core chain repo and a small
contributor set, with one founder identity (Memechi Kekamoto) dominant across the
public record. Active repos (qf-solochain, qf-squid, forks) were still updated as
recently as late March 2026, so it is **not abandoned** — but the cadence reads like
a 1–2 person effort, not a funded core team. Confirm with the contributor graph and
commit-author histogram.

### FINDING 5 — Hard dependency on a moving Parity target. *(A1 — MEDIUM)*
QF forks polkadot-sdk and tracks specific stable tags (e.g. polkadot-stable2503), and
the contract layer is pallet-revive + PolkaVM — tooling Parity itself is still
stabilizing. Runtime upgrades chasing upstream are an ongoing maintenance tax, and
the contract SDK (qf-polkavm-sdk, ~3 stars) is immature. Buildability risk for *your*
dApp is real but ordinary for an early Substrate chain.

### NET PRELIMINARY READ
The prior ("running but not hardening; break is funding/headcount, not a bug") is
**largely surviving contact** — but the headline reason is *strategic* (Finding 2),
not *technical* (Finding 1 came back better than expected). The chain is more
buildable than first feared; the question is whether it's *worth* building on given
an abandoned moat and token-sale-dependent runway. **A6 decouple recommendation
hardens: do not anchor Triarch's attestation layer to QF as sole backend regardless.**

---

## RACI Skeleton (fill before bringing people in)

| Axis | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| A1 Code/Arch | TBD | Ahknu | | |
| A2 Maintenance | TBD | Ahknu | | |
| A3 Protocol *(specialist?)* | TBD | Ahknu | distributed-systems specialist | |
| A4 Forensic | TBD | Ahknu | | |
| A5 DAO design | TBD | Ahknu | law/gov partner | |
| A6 Dependency | Ahknu | Ahknu | | |
| Cold second read (bias) | TBD (not Ahknu) | Ahknu | | |

## Hard Gates Active on This Engagement

1. **HG-QF-BIAS-001** — cold second reader on the conclusion. Non-negotiable.
2. **HG-QF-CLIENT-PAYS-001** — who pays, answered turn one.
3. **HG-QF-SCOPE-LOCK-001** — assessment ≠ repair. No silent escalation.

## Definition of Done

A go/no-go on QF, backed by a per-axis risk read and a cost-of-remediation map,
with the dependency-decoupling recommendation (A6) delivered independently of the
fix verdict. Conclusion is **corroborated**, not "true" — revisable on new
evidence. Worst case: walk away with evidence instead of a hunch. Best case:
greenlight with eyes open and a funded, scoped repair engagement.
