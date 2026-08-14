<div align="center">

# 検証 · Kensho

### An industry-grade Web3 security assessment playbook

EVM smart contracts and DeFi, plus Rust and Solana consensus.
**Prove every finding. Never assume.**

<br/>

`authorize` · `scope` · `recon` · `analyze` · `prove` · `rate` · `report` · `disclose` · `verify-fix`

</div>

---

## What Kensho is

**Kensho (検証)** means *verification*. That single idea is the spine of this playbook: a finding is
not real until it is proven — on a fork, in a harness, or by `eth_call` — and a live system is never
exploited. (Read also as 見性, the Zen "seeing a thing's true nature": seeing what the code really
does, not what the docs claim.)

Kensho is an end-to-end methodology for **finding, proving, rating, and responsibly disclosing**
vulnerabilities in Web3 systems. It is written so a solo researcher can run it top to bottom, and so a
team can drop it in as a shared standard. Every step ships with concrete commands, decision gates,
quality bars, and fill-in templates — not vibes.

The whole playbook lives in one file: **[`SKILL.md`](SKILL.md)** (12 parts). This README is the map.

## Why it exists

Most lost researcher-hours come from three mistakes this playbook is built to kill:

- **Unverified findings.** A plausible bug that doesn't actually reproduce burns your credibility and
  the team's time. Kensho requires a reproducible PoC before a report exists.
- **Inflated severity.** Calling an owner-only power a "Critical" gets you rejected. Kensho separates
  centralization/trust from a permissionless exploit and always states the bound.
- **Hunting the wrong pond.** Kensho runs a deliberate **barbell**: *base flow* (small, fresh,
  unlisted custom-money protocols with reachable teams) funds the *swing shots* (deep diff-hunts of
  the unaudited surface of large protocols, and specialist targets like consensus bounties).

## Who it is for

- Independent security researchers and bug-bounty hunters.
- Audit teams and firms that want a repeatable, defensible process.
- Protocol engineers who want to see how their systems actually get attacked.

## Two tracks, one spine

| Track | Surface | Prizes |
| :--- | :--- | :--- |
| **A** | EVM smart contracts & DeFi | theft / drain / freeze of funds, accounting breaks |
| **B** | Rust & Solana consensus / validators | safety violations (forks), liveness stalls, crypto misuse |

Both run the same lifecycle and hold the same discipline.

## The lifecycle

| # | Phase | Output |
| :--- | :--- | :--- |
| 0 | Authorize & scope | signed scope / bounty terms, out-of-scope list |
| 1 | Intake | intake sheet, shell vars |
| 2 | Verify audit status | "fresh vs picked-over" verdict |
| 3 | Recon & mapping | contract list, surface map, verification status |
| 4 | Analysis | finding candidates per subsystem |
| 5 | Proof of concept | reproducible fork/unit PoC per finding |
| 6 | Severity & rating | defensible rating with the bound stated |
| 7 | Reporting | one standardized report per finding |
| 8 | Disclosure & fix review | private disclosure, patch review, payout |

## Rules of engagement (non-negotiable)

- **Authorized scope only.** A public bounty, a signed SoW, an explicit owner grant, or your own
  deployment. "It's on a public chain" is not authorization.
- **Fork / `eth_call` / local-fork PoC only.** Never move real funds or broadcast a live exploit.
- **Honest severity, always.** State the bound. A Medium is a Medium.
- **Separate centralization from a permissionless exploit.** "Owner can rug by design" is a trust
  finding, not a Critical.
- **Kill your own findings** when they don't hold up. An adversarial self-review is mandatory.
- **Private until patched. Ask, never threaten.** Disclosure and remediation are never conditioned on
  payment. No "pay or I release/exploit" — that is extortion and ends careers.

## The bug-class library

Each class ships in [`SKILL.md`](SKILL.md) with a detection recipe and a fix.

**EVM / DeFi (Track A, §5.3):**

1. **Migration pool-squat** — graduation pre-creates the DEX pool at a fake price (`amountMin=0`, no
   `slot0` check).
2. **Permissionless flash-inflated snapshot** — weight = spot `balanceOf` at an unguarded snapshot;
   flash-borrow, get snapshotted, return.
3. **Spot-price used as a "TWAP"** — `slot0` tick or `getReserves`; a cardinality-1 pool looks like a
   TWAP but is spot.
4. **Oracle staleness / closed-market** — equity/RWA feeds go stale nights & weekends.
5. **Keeper-trusted fund movement** — `distribute(holders,amounts)` only checks `total<=balance`.
6. **Owner backdoor on live positions** — admin swaps migrator/oracle/fee after positions are open.
7. **First-depositor / ERC-4626 share inflation** — tiny deposit + donation rounds the next depositor
   to zero.
8. **Permissionless buy/defend with `minOut=0`** — self-sandwich the protocol's own swap.
9. **Async snapshot manipulation** — two-phase request→finalize mixes a stale snapshot with live state.
10. **Fee-on-transfer over-credit** — `deposited += amount` before a taxing `transferFrom`.

Plus reusable **detection heuristics** (§5.4): Solmate codeless-address phantom-token drains,
`msg.sender`-only guards reachable via a proxy, guard-on-deposit-not-on-`merge`, arbitrary-call
"verify" paths, increment-before-failed-send accounting corruption.

**Consensus (Track B, §10.2):** safety-cert forgery, liveness stalls, stake double-count,
BLS/proof-of-possession misuse, threshold `>=` vs `>` boundaries, fraction/overflow, split-vote
semantics, equivocation across restart, migration seams, and the high-yield
**dissemination/repair-response** surface (sibling-arm asymmetry in `match` over wire messages).

## Recovery / stuck-funds bounties (§5.7)

A distinct engagement type with its own discipline — a "$X to unlock stuck funds" bounty pays only for
*actual recovery*, nothing for an impossibility proof, so the EV is binary. Kensho's method: enumerate
the sole value-out primitive, check its reachability gate, **verify bytecode == source** (watch the
metadata-`SELFDESTRUCT` false-positive), run the storage-surgery "which one slot frees it, and does
anything write it" test — and if freeing needs N simultaneous cross-contract writes, it's hard-fork-only
= walk. Triage in an hour; **a bounty existing is not evidence the funds are recoverable.**

## The Kensho Engine (companion intelligence)

Kensho isn't just a static document — it's paired with a **self-growing intelligence engine** (§11.6):
a corpus of ~1,000 real, resolved findings and exploits (most with runnable PoCs), classified into the
same bug-class taxonomy. It gives the method three things:

- **Targeting** — before auditing a live target, rank the bug classes that *historically hit that kind
  of protocol*, with the matching Kensho reference and example PoCs to study. Diff-hunt the right
  surface first instead of a blank page.
- **Backtesting** — audit past-vulnerable code *blind* and score the method's catch-rate; every miss
  becomes a new heuristic folded back into §5.3/§5.4.
- **Learning** — the corpus grows daily; the playbook gets sharper with it. No fine-tuning — it learns
  from a growing, queryable, backtestable body of real findings.

## Tooling & automation (§11)

- **AI audit tool stack** — map → breadth → depth → precedent → package, chained across specialist tools.
- **Multi-agent fleet pattern** — a dupe-exclusion digest, parallel finders per subsystem, and an
  adversarial verifier that tries to *refute* each candidate before it survives.
- **Field-learned gotchas** — RPC rate-limits, `eth_getLogs` ranges, blobless partial clones, feature-
  gated crates silently running 0 tests, non-mockable liveness timers.

## Severity taxonomy (§7)

| Severity | Definition |
| :--- | :--- |
| **Critical** | Unauthenticated theft/loss of a large share of funds, or full drain/freeze. One tx, no special role. |
| **High** | Unauthenticated, repeatable theft — but **bounded** (state the bound), or needing a common condition. |
| **Medium** | Limited/conditional loss, griefing, or a bounded value leak; often condition-gated. |
| **Low** | Minor/edge, unlikely, or negligible value. |
| **Trust / centralization** | Owner-by-design power. Disclose and label as such — not a permissionless exploit. |

## PoC & reporting standards (§6, §8)

- Reproducible **from one command**, pinned to a block/commit, on a local fork or unit test.
- **Asserts impact in numbers** (attacker gain > cost, or funds stranded/frozen).
- No mainnet state touched, no live tx broadcast — `vm.*` cheatcodes are inert on real networks.
- **One finding per report**, self-contained, honest, with a concrete fix.

## Disclosure & remediation (§9)

Find the private channel, open with a one-line impact + "verified on a fork, nothing touched, PoC you
can run," share via a **private repo**, help fix and review the patch. Name a number only *after* they
confirm, anchored on merit. Never bundle "pay or I post."

## How to use it

**As a Claude Code / Agent skill.** Put this folder in your skills directory and invoke it by name
(`kensho`); the agent runs the lifecycle for you.

**As a manual playbook.** Open [`SKILL.md`](SKILL.md) and work top to bottom — every step is
copy-paste ready.

## Principles that never bend

1. Authorized scope only.
2. Fork, harness, or `eth_call` proof only. Never exploit a live system.
3. Honest severity, always — state the bound.
4. Separate centralization risk from a permissionless exploit.
5. Kill your own findings when they don't hold up.
6. Private until patched. Ask, never threaten.

## Repository contents

| File | Purpose |
| :--- | :--- |
| [`SKILL.md`](SKILL.md) | The full 12-part playbook (Tracks A & B, engine, tooling, reference) |
| `README.md` | This overview |
| `LICENSE` | Attribution terms (CC BY 4.0) |

## Author

Created and maintained by **Duke** ([@dukedotsol](https://x.com/dukedotsol), GitHub
[cryptoduke01](https://github.com/cryptoduke01)). Distilled from real offensive security engagements
across EVM DeFi and Rust/Solana consensus. Field-tested, not theoretical.

## License

**Creative Commons Attribution 4.0 International (CC BY 4.0).** Share and adapt freely, including
commercially, as long as attribution to Duke ([@dukedotsol](https://x.com/dukedotsol)) stays intact.
See [`LICENSE`](LICENSE).

<div align="center">
<br/>

*A finding is not real until it is proven.*

</div>
