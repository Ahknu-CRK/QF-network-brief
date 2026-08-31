---
id: BRIEF-QF-001
uns_version: 0.1
type: brief
title: "State of QF Network — Consolidated Assessment Brief (v1.4)"
domain: triarch
status: ready
created: 2026-05-21
author: Ahknu
evidence_status: yellow
priority: high
tags: [brief, qf-network, assessment, consolidation, handoff, decision-support]

pipeline:
  stage: review
  next_stage: decision

relationships:
  consolidates:
    - "ENG-QF-001"   # the engagement
    - "TEST-QF-001"  # the test plan / handoff agenda
  implements:
    - "META-CAA-001" # the engagement-type spec
  references:
    - "github.com/QuantumFusion-network/qf-solochain @ HEAD 2026-04-21"

x_triarch:
  purpose: "Single decision-support document for the team. Rolls up all findings to date so others can react. Feeds go/no-go on adopting QF as substrate."
  decision_owner: Ahknu
  scope_gate: "ASSESSMENT ONLY — no repair authorized. Adoption decision precedes any code work."

x_fsg:
  note: "Method (problem-solving rigor) was used throughout. FSG-the-governance-layer is NOT yet applied to QF and would only attach post-adoption, minimally, at the governance/trust layer — never inside the protocol filter logic."

lifecycle:
  events:
    - "2026-05-21 created — consolidates ENG-QF-001 + TEST-QF-001 + full repo read"
    - "2026-05-22 v1.1 — added qf-polkavm-sdk read (embryonic) + one-way claim/bridge finding (engagement-redefining); reordered next-actions, exit-path = remediation priority 1 if adopted"
    - "2026-05-22 v1.2 — ERC-20 market data pulled (6395 holders, live Uniswap liq); one-way severity narrowed HIGH->MED-HIGH, likely scenario b (no seizure), source confirmation still pending"
    - "2026-05-22 v1.3 — USER-VERIFIED DexScreener/Go+ data supersedes search: token economics corrected (price $0.02, liq $92K, mktcap $205K, 24h -37%, bleeding); honeypot CLEARED (sells work, not trapped); one-way severity MED-HIGH->MEDIUM; tokenomics locked (ownership renounced)"
    - "2026-05-22 v1.4 — TEAM FINDING CORRECTED: prior evaporating/1-dev read was a data-source error (read qf-solochain git log, missed the active spec-repo project board). Real status: small but active ~3-4 devs in sustaining/maintenance mode. QF reframed dying->idling. Also updates TL;DR."
---

# State of QF Network — Consolidated Brief (v1)

> **Read this first.** This is an assessment, not a verdict and not a repair. Every
> finding carries a confidence flag: 🟢 solid / 🟡 assumption / 🔴 known gap.
> All code findings are from a static read of the public repo at HEAD (last commit
> 2026-04-21). Nothing was compiled or run — no Rust toolchain in the assessment
> environment. The go/no-go decision is the reader's, not the assessor's.

## TL;DR (one paragraph)

QF Network is a **technically sound, MIT-licensed, actively-maintained PolkaVM
Substrate L1** whose original differentiator (HVM2 parallel compute) was dropped,
and whose **small team (~3–4) is alive and shipping but in sustaining/maintenance
mode, not advancing the vision**. The code asset is in better shape than its
outside reputation suggests. The central decision is not "is this junk worth
saving" and not "are the authors gone" (they're reachable and committing) — it is
**"is it worth adopting a sound-but-IDLING chain with a dead token, whose team
maintains rather than innovates, when our own moat doesn't require it?"** The
people who know the code can still answer questions; the open question is whether
the thing is worth building on at all given a $205K-mktcap token and a roadmap
that's all plumbing, no protocol advancement.

---

## What changed during assessment (intellectual honesty log)

Three early desk-pass alarms were **overturned by reading the actual repo.** Logged
here because the corrections matter as much as the findings:

| Early claim (desk pass) | Corrected finding (repo read) |
|---|---|
| Custom consensus is the scariest risk | 🟢 Finality is stock GRANDPA; SPIN is only leader-election |
| Bus factor ~2 | 🟢 Historically ~7–8 real engineers (then see team-decay below) |
| Repo looks thin / dying | 🟢 Steady cadence through Apr 2026; competent CI/CD |
| The `TODO(zotho)` is a vulnerability-shaped hole | 🟢 Not a forgery hole; binding provided by signature check |

The bias gate (assessor holds a position in QF) is the reason these were
re-checked rather than trusted. It worked.

---

## Findings by area

### 🟢 GREEN — the code asset is solid
- **Block time:** 100ms is real in code (`MILLI_SECS_PER_BLOCK`, "50ms compute /
  100ms block"). *Caveat: not yet verified under load — see RED.*
- **Finality:** stock Parity GRANDPA (`pallet_grandpa` wired in qf-runtime). The
  dangerous surface is NOT homegrown.
- **Architecture:** coherent two-runtime design — `qf-runtime` (fast solo
  authoring) + `parachain` runtime (Polkadot-anchored shared security).
- **Shared-security bridge:** `pallet-spin-polkadot` is built AND wired (into the
  parachain runtime), with a working TypeScript finality relayer. Not abandoned.
- **HVM2 residue:** cleanly gone — zero dead code in the execution path.
- **License:** MIT. Free to fork, modify, build commercially. *Code adoption is
  legally trivial.* (Token / network / trademark / DAO obligations are separate —
  see RED legal.)
- **CI/CD:** real — 4 workflows incl. containerized release pipeline.
- **Code hygiene:** 35 TODOs across 65 files (low), only 2 panic/unimplemented
  stubs, named-owner TODOs tracking real upstream-Parity work.
- **Security core (`submit_finality_proof`):** competent GRANDPA verification —
  relayer auth, set_id match, monotonic replay guard, per-precommit signature
  check, signer dedup, 2/3 supermajority, overflow-safe arithmetic.

### 🟡 YELLOW — assumptions / documented residual questions
- **Test coverage is inverted.** Mature/standard pallets are tested (claim: 31
  tests; spin: 6). The **novel security-critical pallets have ZERO tests**
  (`spin-anchoring`, `spin-polkadot`, `primitives/consensus-spin`). Fixable —
  additive — see TEST-QF-001 (19 invariants scoped).
- **`votes_ancestries` accepted but never validated** in the finality bridge.
  Strong hypothesis it is *safe* (the verifier forces all precommits to one
  identical target via `MismatchedTargets`, subsuming the ancestry check). Needs
  a cryptographer/author nod before trusting. One-sentence handoff question.
- **Parity fork is bleeding-edge** (`polkadot-stable2603`). Fresh, but the contract
  tooling (PolkaVM / pallet-revive) is the least battle-tested substrate, and
  chasing upstream is an ongoing maintenance tax that only happens with an active
  team.
- **38 `.unwrap()` calls** — modest; worth a pass to confirm none are in the
  runtime execution path.

- **Contract SDK (`qf-polkavm-sdk`) is real but embryonic.** Thin wrapper over
  Parity's `pallet-revive`: simplified compilation, allocator/panic handler, an
  `export` macro. Published on crates.io, one working example. Their own roadmap
  lists UNCHECKED: typed storage API, JS/TS client gen, address prediction, and
  **deployment/testing tools**. Last commit 2025-12-04 (~5 months stale — quieter
  than the chain). Verdict: you can deploy native-Rust contracts today, but you'd
  write against raw `pallet-revive-uapi` for anything real. Workable, NOT a head
  start. The mature part (pallet-revive) is Parity's, not QF's.

### 🔴 RED — known gaps (NOT guessed; require resources we don't have here)
- **⚠ ONE-WAY VALUE PATH — engagement-redefining, severity narrowed 2026-05-22 (v1.3 user-verified).**
  The repo contains a `claim` pallet (forked from Polkadot's Ethereum-claims
  pallet, GPL, 31 tests) that moves value Ethereum→QF. There is **NO outbound path
  anywhere in the chain repo**. Confirmed one-way 🟢. TRAPPED-FUNDS QUESTION NOW
  RESOLVED 🟢 via DexScreener/Go+ security panel (user-supplied, beats prior search
  data): the ERC-20 is **NOT a honeypot** — Go+ shows Honeypot: No, Can't-sell-all:
  No, Transfer-pausable: No, Mintable: No, Owner-can-change-balance: No, Ownership
  renounced: Yes, Owner balance: 0. Live sells are occurring. **ERC-20 holders are
  NOT trapped — they can sell.** The one-way issue is strictly about value migrated
  *to QF mainnet* having no return leg ("roach motel"), NOT seizure of ETH-side
  funds. Exit path remains remediation priority #1 IF adopted (highest rigor, not
  rushed), but this is no longer a user-harm emergency. Severity: MEDIUM.
- **TOKEN ECONOMICS — confirmed grim (v1.3 user-verified, corrects prior search data).**
  QF ERC-20 (0x6019…703d) actual figures from DexScreener: price **$0.02054**,
  liquidity **~$92K**, FDV/mkt cap **~$205K**, ~5,984–6,395 holders, 4 LP holders.
  24h: **−37.31%**, sell-weighted ($17K sell vs $5.2K buy vol, 36 sellers/25 buyers).
  (Prior brief versions cited $1–2.34 / ~$793K liquidity from search — THAT WAS
  WRONG; these user-pulled numbers supersede.) Implication: a $205K-mkt-cap,
  $92K-liquidity, actively-bleeding token **cannot fund a multi-dev team** —
  directly explains the team-decay finding. The "money bs" is real and structural.
  Note: ownership renounced + owner balance 0 means no rug lever, but also means
  **no one can unilaterally fix tokenomics** — it's locked as-is.
- **Team status — CORRECTED v1.4 (prior read was WRONG, data-source error).**
  Earlier versions reported "evaporating, ~1 active dev, silent since 2026-04-21"
  based on the `qf-solochain` git log. That was a BAD READ: active work moved to a
  separate `spec` repo project board, which is currently live. Real status: **small
  but ACTIVE team (~3–4), shipping right now** — artemiksion (fastchain runtime
  v120, chain-data PRs), khssnv/Khassanov (parachain runtime v3, coretime renewal,
  dapp-migration docs), Vsevolod-Rusinskiy/Orlov (testnet RPC, wallet UX), aardbol
  (infra). Board shows 36 Done, 12 in progress, items dated as recent as 2026-05-22.
  **BUT** — the active work is **sustaining/maintenance/migration plumbing** (faucet
  deploys, RPC URL updates, runtime version bumps, "get dapps to migrate onto QF,"
  logo-to-IPFS), NOT vision-advancement. No active work on SPIN, shared-security, or
  new protocol capability. **Revised read: QF is not DYING — it is IDLING.** Small
  team keeping a functional chain alive and courting migration, not innovating.
  Severity for adoption: still relevant (you'd inherit a sustaining-mode project, not
  a growing one) but NOT the "nobody left to ask" emergency prior versions implied —
  the people who know the code are still reachable and still committing.
  - Note: open bug #869 `UnknownTransaction::CannotLookup` (tx-validity) sits
    unassigned in Backlog — a concrete real-problem entry point if ever re-engaging.
- **Does it compile?** Unknown — no Rust toolchain in assessment env. Must run
  `cargo check` on QF-capable hardware.
- **Does 100ms hold under load?** Unknown — needs a multi-node testnet with real
  latency and GRANDPA finalizing. Config value ≠ proven property.
- **On-chain reality** — actual validator count, token/treasury balance, is the
  testnet currently producing blocks. Needs RPC/explorer access.
- **Team reachable for knowledge transfer?** Unknown — human contact. **This is
  the decider.** (zotho/artemiksion/Khassanov are the names to reach.)
- **Legal / token / DAO transfer** — who owns it, token-holder obligations, clean
  transfer/fork path. Needs the law/gov partner.
- **PolkaVM contract SDK maturity** — `qf-polkavm-sdk` not yet read; this is the
  layer a Triarch moat would deploy on. Last big checkable unknown.

---

## The decision this feeds

**Go/no-go on adopting QF as Triarch's substrate**, framed by one pivotal question:

> Can we secure knowledge transfer from the original engineers (esp. artemiksion /
> Khassanov / zotho) before they are fully gone?

- **If YES (warm handoff):** strong opportunity. Quality MIT code, motivated-to-exit
  owners, the test gap is scoped and additive, the open questions are
  sentence-answerable. Adopt, test-harden the two pallets, attach FSG governance
  minimally at the trust layer.
- **If NO (cold handoff):** expensive archaeology. You inherit a sophisticated
  custom-pallet / PolkaVM / forked-Parity chain with untested security-critical
  code and nobody to ask. Proceed only with eyes open and a real Rust/distributed
  -systems hire budgeted.

**Independent of the above:** the A6 dependency-decouple recommendation stands —
do not anchor Triarch's attestation layer to QF as sole backend. Put it behind an
interface; QF becomes one swappable option, not the foundation.

---

## Recommended next actions (proposed — decision owner picks)

1. **Bridge scenario RESOLVED** (Go+ panel: not a honeypot, sells work, ownership renounced — ERC-20 holders not trapped). Remaining: confirm there is no QF-mainnet→ETH return leg and scope an exit path IF adopted. No longer an emergency.
   it has no lock/escrow fn and 0x6019…703d holds no large locked balance (market
   data already points to scenario b / no seizure — confirm at source). Cheap, decisive.
2. **Reach the devs now.** Warm, specific, peer-level. The handoff agenda in
   TEST-QF-001 (the OQ list) is the script — lead with the GRANDPA/ancestry
   questions to signal you read the code. Time-sensitive (team going dark).
3. **`cargo check` on real hardware.** Cheapest hard fact about workability.
4. **On-chain pull** — validators, treasury, live block production.
5. **Legal pass** with the law/gov partner on token/DAO/transfer AND the one-way
   value-path liability.
6. **If adopted: a sanctioned exit path is remediation priority #1** — ahead of
   the moat. First in line, highest rigor, not rushed.
7. **Defer** any FSG-governance mapping and all other code repair until after
   go/no-go.

## Confidence on this brief
🟢 on everything readable from the repo (code quality, license, CI, architecture,
test gaps, finality-logic soundness on verifiable branches).
🟡 on the ancestry-safety inference (needs cryptographer/author).
🔴 on compile, load behavior, on-chain state, team reachability, legal — all
flagged, none guessed.
