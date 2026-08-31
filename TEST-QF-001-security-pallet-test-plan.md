---
id: TEST-QF-001
uns_version: 0.1
type: test-plan
title: "QF Security-Critical Pallet Test Plan — spin-anchoring & spin-polkadot"
domain: triarch
status: draft
created: 2026-05-21
evidence_status: green
priority: high
tags: [test-plan, qf-network, spin-polkadot, spin-anchoring, finality, security]

pipeline:
  stage: draft
  next_stage: review

relationships:
  supports:
    - "ENG-QF-001"   # remediation artifact for the one RED finding
  references:
    - "pallets/spin-anchoring/src/lib.rs"
    - "pallets/spin-polkadot/src/lib.rs"
    - "pallets/spin/src/tests.rs (existing test pattern to mirror)"
    - "pallets/claim/src/{mock,tests}.rs (mature pattern in-repo)"

x_triarch:
  purpose: "Convert the one RED finding (untested security-critical code) into a scoped, executable remediation."
  note: "Doubles as the old-dev handoff agenda — every OPEN QUESTION is a specific question for the original author (zotho/team), not 'explain stuff'."

x_fsg:
  formation: small-team
  tempo: medium
  zone: middle

lifecycle:
  events:
    - "2026-05-21 created — derived from line-by-line read of both pallets"
---

# TEST-QF-001 — Test Plan for QF's Untested Security-Critical Pallets

## Context

Two of QF's novel, security-critical pallets ship with **zero tests**:
`spin-anchoring` and `spin-polkadot`. Both are small and legible (anchoring ~75
LOC, spin-polkadot ~251 LOC), and the repo already has a mature test pattern to
mirror (`pallet/claim`, `pallet/spin` both have `mock.rs` + `tests.rs`). This is
**additive remediation** — tests written against existing, readable code. It is
the most fixable finding in ENG-QF-001.

This plan does double duty: each **OPEN QUESTION** is a precise handoff question
for the original author (the `TODO(zotho)` notes are authored uncertainty — gold
for a warm handoff).

---

## Part A — `spin-anchoring` (the SecureUpTo watermark)

**What it does:** stores `SecureUpTo` (highest securely-anchored fastchain block)
and a `Relayer` account. `note_anchor_verified(up_to)` advances the watermark if
`up_to > prev`; callable by root or the configured relayer. `set_relayer` is
root-only.

### Invariants to test
| # | Invariant | Test |
|---|---|---|
| A1 | Monotonicity — `SecureUpTo` never decreases | submit decreasing `up_to`, assert no change, no event |
| A2 | Advance on strictly greater | submit greater `up_to`, assert storage updated + `SecureFinalityAdvanced` emitted |
| A3 | Equal value is a no-op | submit `up_to == prev`, assert no event, no write |
| A4 | Relayer auth | non-relayer signed origin → `BadOrigin` |
| A5 | Root bypass | root origin always allowed regardless of relayer |
| A6 | set_relayer is root-only | signed origin → `BadOrigin`; root succeeds |
| A7 | Genesis | relayer set at genesis is honored on first call |

### Open questions (handoff agenda)
- **OQ-A1:** Is there *any* path where `note_anchor_verified` should reject an
  `up_to` that exceeds the actual fastchain height? Today it trusts the caller
  fully — the watermark can be advanced past reality by a compromised relayer.
  Was that an accepted trust assumption (relayer = trusted) or a gap?

---

## Part B — `spin-polkadot` (the GRANDPA finality bridge — THE security core)

**What it does:** `submit_finality_proof` verifies a GRANDPA justification from the
fastchain against a trusted authority set, enforcing the 2/3 supermajority, then
records `LastFinalized`. `set_authority_set` (root/relayer) installs the trusted
set. `set_relayer` is root-only.

### Invariants to test
| # | Invariant | Test |
|---|---|---|
| B1 | Happy path | valid justification, ≥2/3 weight, good sigs → `LastFinalized` updated + `FinalityProofAccepted` |
| B2 | Supermajority threshold | weight just *below* 2/3 → `InsufficientWeight`; exactly 2/3 → accepts |
| B3 | Bad signature | one invalid precommit sig → `BadSignature` |
| B4 | Unknown authority | precommit from non-set signer → `UnknownAuthority` |
| B5 | set_id mismatch | `expected_set_id` ≠ stored → `AuthoritySetMismatch` |
| B6 | Replay / monotonic | `target_number <= LastFinalized.number` → `AlreadyFinalized` |
| B7 | Empty precommits | empty list → `NoPrecommits` |
| B8 | Uninitialized set | no authority set → `AuthoritySetNotInitialized` |
| B9 | Mismatched targets | precommit target ≠ commit target → `MismatchedTargets` |
| B10 | Dedup | same authority signs twice → counted once (no double weight) |
| B11 | Relayer auth | non-relayer → `BadOrigin` |
| B12 | Overflow guards | crafted weights → `ComputationOverflow` not panic |

### Open questions (handoff agenda — these are the REAL ones)
- **OQ-B1 (RESOLVED — forgery branch):** The in-code `TODO(zotho)`: *"how do we
  verify that target_hash is hash of target_number block?"* TRACED 2026-05-21:
  `check_message_signature` verifies over `finality_grandpa::Precommit { target_hash,
  target_number }` (finality-grandpa v0.16.3) — both fields together. A
  `(fake_hash, real_number)` pairing cannot collect valid signatures because no
  honest authority signed it. **Fake-hash forgery is NOT possible; the binding is
  provided by the existing signature check.** A redundant NC interlock is NOT
  needed for this concern. Confidence 🟢 on the forgery branch.
- **OQ-B1b (CONFIRMED unused; likely-safe simplification — needs author nod):**
  TRACED 2026-05-21: `votes_ancestries` appears exactly 3 times in the whole repo
  (struct field, Debug fmt, size const) and is NEVER read in `submit_finality_proof`.
  Ancestry is accepted but not validated. HYPOTHESIS (strong): the verifier forces
  every precommit to the *identical* `(target_hash, target_number)` via the
  `MismatchedTargets` check, so there is no ancestry tree to walk — the stricter
  same-target constraint subsumes the divergence that `votes_ancestries` guards
  against. If correct, omitting ancestry validation is SAFE, not a hole.
  Confidence 🟡 — inferred from protocol semantics + the same-target check, NOT
  from line-by-line upstream comparison (upstream verifier not retrievable in
  assessment env). MUST get a cryptographer or original-author confirmation before
  trusting. **Handoff question:** "The same-target enforcement — does that
  intentionally make votes_ancestries validation unnecessary, or was ancestry
  validation dropped?" One sentence closes this for the author.
  Test implication: add a B-series test asserting mixed-target precommits are
  rejected (locks in the property the safety argument depends on).
- **OQ-B2:** `TODO(zotho)` — recomputing `total_weight` every call. Perf only, or
  is there a correctness reason it isn't cached with the authority set?
- **OQ-B3:** `TODO(zotho)` — "only first seen" dedup. Confirm intended GRANDPA
  semantics (first valid precommit per authority wins) vs. any equivocation
  handling expectation.
- **OQ-B4:** No equivocation detection (same authority, conflicting precommits in
  one round). Out of scope by design, or a known TODO?
- **OQ-B5:** Authority-set rotation — `set_authority_set` is manual via root/relayer.
  How is set rotation kept in sync with the fastchain's actual GRANDPA set changes?
  What happens if they desync?

---

## Execution Notes
- Mirror the existing in-repo pattern: add `mock.rs` (mock runtime) + `tests.rs`
  per pallet. `pallet/claim` is the richest reference (31 test fns).
- B-series needs GRANDPA test fixtures (keypairs, signed precommits). Substrate's
  `sp_consensus_grandpa` test helpers + the existing `pallet_grandpa` test
  scaffolding upstream are the source to crib from.
- **Cannot be run in current assessment env** (no Rust toolchain). This plan is
  written to be executed on QF-capable hardware. Status of "does the existing
  code even compile" remains 🔴 Unknown until then.

## Definition of Done (for this plan)
Every invariant above has a passing test; every OPEN QUESTION is either answered
by the original author or converted to a documented, accepted risk. At that point
the ENG-QF-001 RED finding (untested security-critical code) flips to YELLOW
(tested, residual design questions documented) or GREEN (tested + questions
resolved).
