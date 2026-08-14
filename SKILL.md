---
name: kensho
description: >-
  KENSHO (検証) — industry-grade playbook for offensive Web3 security assessment and responsible
  disclosure, covering EVM smart contracts / DeFi and Rust/Solana consensus &
  validator code. Usable by individual researchers, audit teams, and security firms.
  Invoke when a target is named (project, handle, repo, or website) and the goal is
  to assess it for vulnerabilities and turn findings into responsible, paid
  disclosures. Runs a defined lifecycle: authorize -> scope -> recon -> analyze ->
  prove -> rate -> report -> disclose -> verify-fix. Verification is fork / eth_call /
  local-fork PoC ONLY; live systems are never exploited. Maintained by @dukedotsol.
---

# KENSHO (検証) — Web3 Security Assessment Playbook

*Kensho (検証): "verification / proof" — the discipline at the heart of this playbook:
every finding is proven on a fork, in a harness, or by `eth_call`, never assumed. (Also
read as 見性, the Zen "seeing a thing's true nature" — seeing what the code really does.)*

**Author & creator: Duke (@dukedotsol · GitHub cryptoduke01).**
Authored and maintained by Duke, distilled from real offensive security engagements
across EVM DeFi and Rust/Solana consensus. Field-tested, not theoretical. Teams and
individuals are free to adopt and adapt it; please keep this attribution intact.

An end-to-end, industry-grade methodology for finding, proving, rating, and
responsibly disclosing vulnerabilities in Web3 systems — EVM smart contracts and
DeFi protocols, and Rust/Solana consensus/validator code. Written to be adopted by a
solo researcher or dropped into a team's process as a shared standard. Every step has
concrete commands, decision gates, quality bars, and fill-in templates.

Two engagement tracks share one spine:
- **Track A — EVM smart contracts / DeFi** (Parts 3–9).
- **Track B — Rust / Solana consensus & validators** (Part 10).

Cross-cutting: rules of engagement (Part 1), lifecycle (Part 2), tooling & automation
(Part 11), reference library (Part 12).

---

# PART 1 — RULES OF ENGAGEMENT (non-negotiable; read once, apply always)

These are the professional and legal floor. They do not bend for revenge, urgency,
"I'll give it back," "just to prove it," or a client asking you to go further.

### Authorization & legality
1. **Only test what you are authorized to test.** Authorization = a public bug-bounty
   scope, a signed engagement/SoW, an explicit written owner grant, or your own
   deployment. "It's on a public chain" is not authorization to attack a live system.
2. **Read the program's scope, out-of-scope list, and severity→payout table BEFORE
   touching code.** This is step zero on any bounty. It routinely kills half your
   angles and prevents wasted (and on pay-to-submit platforms, expensive) reports.
3. **Respect data-handling and privacy.** Never exfiltrate real user data. Don't dox
   pseudonymous teams. Keep client material confidential.

### Verification safety
4. **Fork / `eth_call` / local-fork PoC ONLY. Never move real funds or broadcast a
   live-network exploit transaction.** A passing fork or unit test is the proof.
5. **Never handle a client's or user's private keys, and never broadcast transactions
   on their behalf.** Hand them the command to run themselves.
6. **Never publish a live, unpatched vulnerability.** Private channel + private repo
   until a fix ships. On competitive/first-to-file programs, publishing also forfeits
   the finding.

### Integrity
7. **Honest severity, always.** State the bound. A Medium is a Medium. Never inflate.
8. **Separate centralization/trust-risk from a permissionless exploit.** "Owner can
   rug by design" is a trust finding, not a Critical. Label it as such.
9. **Kill your own finding if it does not hold up.** Disproving yourself is the job;
   an adversarial self-review before submission is mandatory (Part 5, quality gate).
10. **Ask, never threaten.** No "pay or I release/exploit." That is extortion and ends
    careers. Disclosure and remediation are never conditioned on payment.
11. **Assess contracts, not narratives.** Day-one hype with no verifiable contract is a
    scam trap. Sketchy TLDs / brand-impersonation = assume phishing; never connect a
    wallet.

---

# PART 2 — ENGAGEMENT LIFECYCLE

The nine phases below map to the rest of this document. On a team, phases can be owned
by different people; the artifacts (intake sheet, surface map, finding tickets, PoCs,
report, disclosure thread) are the hand-off points.

| # | Phase | Output | Reference |
|---|---|---|---|
| 0 | Authorize & scope | signed scope / bounty terms, out-of-scope list | Part 1, Part 3 |
| 1 | Intake | intake sheet, shell vars | Part 3 |
| 2 | Audit-status verification | "fresh vs picked-over" verdict | Part 3 |
| 3 | Recon & mapping | contract list, surface map, verification status | Part 4 |
| 4 | Analysis | finding candidates (per subsystem) | Part 5 |
| 5 | Proof of concept | reproducible fork/unit PoC per finding | Part 6 |
| 6 | Severity & risk rating | defensible rating with stated bound | Part 7 |
| 7 | Reporting | standardized report per finding | Part 8 |
| 8 | Disclosure & remediation | private disclosure, fix review, payout | Part 9 |

**Strategy note (portfolio-level).** Two ponds, deliberately balanced ("barbell"):
- **Base flow:** small, fresh, unlisted custom-money protocols with reachable teams,
  found by following the money on new chains, disclosed direct. Less crowded, builds
  reputation and cash flow.
- **Swing shots:** deep *diff-hunts* of the unaudited surface of large protocols, and
  specialist targets (e.g. consensus bounties), for the high-value criticals.
The crowded platform route on hardened, audited code with hostile scope is usually the
*wrong* primary pond; enter it selectively, funded by the base.

---

# PART 3 — INTAKE, SCOPING & AUDIT-STATUS VERIFICATION

## 3.1 Intake sheet

```
PROJECT_NAME   : <e.g. hood.fun>
X_HANDLE       : <@handle>                    # to find the private disclosure channel
WEBSITE        : <https://...>                # app URL (where positions/tokens render)
DOCS           : <https://docs... or "none">
CHAIN          : <e.g. Base / Robinhood Chain>
RPC            : <https://... or "unknown">   # confirm in recon
CHAIN_ID       : <e.g. 8453 or "unknown">
EXPLORER       : <Blockscout/Etherscan base or "unknown">
KNOWN_ADDRS    : <addresses the client/user already has, else "none">
PRODUCT_TYPE   : <launchpad | lending/CDP | vault | perps | distribution/index | bridge | other>
PROGRAM        : <bounty/contest? platform? amount? scope URL? or "direct/none">
AUTHORIZATION  : <bounty scope URL | signed SoW | owner grant | own deployment>
NOTES          : <anything stated: "running a contest", "launched today", etc.>
RESEARCHER     : <you / your team>
```

Set reusable shell vars:
```bash
RPC="<RPC>"; CID="<CHAIN_ID>"; BS="<EXPLORER>/api/v2"
```

## 3.2 Audit-status verification (MANDATORY before investing time)

Automated "audited?" fields lie. Verify independently — a finding that duplicates a
published audit item is an instant rejection and a reputation hit.
```bash
curl -sL "https://<project>" | grep -iE "audit|security|bug bounty|immunefi|sherlock|cantina|code4rena"
# check for /security, /audits, /bug-bounty, /safe-harbor docs pages
# web-search "<project> audit"  (nothing indexed on a days-old launch = good sign)
# platforms: audits.sherlock.xyz, immunefi.com, code4rena.com, cantina.xyz
# grep the contract source comments for "audit" / "security-contact"
```
- **Genuinely fresh (best base-flow target):** bare landing page, no docs/security page,
  no bounty, zero search results, contract deployed days ago.
- **Picked-over:** full docs with Audits/Bug-Bounty/Safe-Harbor sections, a platform
  listing, published audit PDFs. The shallow bugs are gone and you compete with a crowd
  — either skip, or switch to the diff-hunt method (Part 5.6).

**Before submitting any finding, read the project's own published audits and grep them
for your bug class.** If it's there (even accepted/won't-fix), it's a duplicate.

---

# PART 4 — RECONNAISSANCE & TARGET MAPPING (Track A / EVM)

## 4.1 Ground truth
```bash
cast chain-id --rpc-url $RPC          # must match CHAIN_ID
cast block-number --rpc-url $RPC      # RPC alive?
```
Gate: RPC/chain won't resolve and no docs/verifiable contract exist → likely
vaporware/scam; report and stop. Real chain → continue.

## 4.2 Locate the core contract(s)
Frontends hide addresses in runtime config. Use in order:

**A. Trace a live token/position → its creator (most reliable).**
```bash
T=0x<token>
curl -s "$BS/addresses/$T" | python3 -c "import sys,json;print(json.load(sys.stdin).get('creator_address_hash'))"
```
**B. Grep the app bundle** (Vite `/assets/index-*.js`; Next `/_next/static/chunks/*.js`):
```bash
curl -sL "<WEBSITE>" -o app.html
J=$(grep -oE '/assets/[^"]+\.js' app.html | head -1); curl -s "<WEBSITE>$J" \
 | grep -oiE '(factory|launch|vault|pool|router|oracle|manager|locker)"?:"?0x[a-fA-F0-9]{40}' | sort -u
```
**C. Browser** — load the app, read network requests, pull `to` addresses from `eth_call`s.
**D. Explorer** — search Blockscout/Etherscan for the token/contract names.
**E. DefiLlama TVL adapter** (authoritative contract list):
`raw.githubusercontent.com/DefiLlama/DefiLlama-Adapters/main/projects/<slug>/index.js`.

## 4.3 Verification gate + surface map
```bash
C=0x<core>
cast balance $C --rpc-url $RPC                       # funds at risk
cast code $C --rpc-url $RPC | wc -c                  # complexity
curl -s "https://sourcify.dev/server/v2/contract/$CID/$C?fields=sources,compilation" -o src.json
python3 -c "import json;d=json.load(open('src.json'));print(d.get('match'),(d.get('compilation') or {}).get('name'))"
```
- **Verified** → pull sources from `src.json` and read (one verified handler often drags
  in its whole import tree: libs, calculators, siblings).
- **Unverified** → recover selectors from bytecode + real tx history:
```bash
cast code $C --rpc-url $RPC > code.hex
cast disassemble $(cat code.hex) \
 | awk '/PUSH4 0x/{s=$0;h=NR} /EQ/&&(NR-h<=2){print s}' | grep -Eo '0x[0-9a-fA-F]{8}' | sort -u \
 | while read x; do printf "%s %s\n" "$x" "$(cast 4byte $x|head -1)"; done
curl -s "$BS/addresses/$C/transactions" | python3 -c "import sys,json;from collections import Counter;d=json.load(sys.stdin);c=Counter((t.get('raw_input') or '0x')[:10] for t in d.get('items',[]));print(c.most_common(10))"
```
- **Resolve proxy → impl:** EIP-1967 slot `0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc` via `cast storage`.

## 4.4 Infrastructure fingerprints — "this is not a protocol, skip it"
Large balances are often infra/treasury, not exploitable protocol:
- **Gnosis Safe:** ~171-byte proxy, selector `a619486e` (masterCopy), singleton in slot 0.
- **EIP-7702 delegated wallet:** code = `0xef0100 || <20-byte impl>`. User wallet, not protocol.
- **Minimal proxy (EIP-1167):** exactly 45 bytes; read the impl and audit that.
- **Known audited frameworks (audit only the FORK MODS):** Morpho VaultV2, Veda
  BoringVault, Spark vaults, Native CreditVault, Balancer V2, Merkl/Angle distributor,
  and common launch-token templates. Recognize the base; hunt only the custom bolt-ons.

---

# PART 5 — VULNERABILITY ANALYSIS (Track A / EVM)

## 5.1 Authorization triage (find the free win first)
`eth_call` every state-changing admin/keeper function from an attacker address. No
revert = missing access control = likely Critical.
```bash
ATK=0x000000000000000000000000000000000000dEaD
for sig in "setOwner(address)" "setKeeper(address)" "setOracle(address)" "setFee(uint256)" "mint(address,uint256)" "withdraw(uint256)"; do
  printf "%-28s " "$sig"
  out=$(cast call $C "$sig" $ATK --from $ATK --rpc-url $RPC 2>&1)
  echo "$out" | grep -qi "revert\|error" && echo "guarded" || echo "OPEN <-- CHECK"
done
```

## 5.2 Read with attack questions (by PRODUCT_TYPE)
Hunt **one wrong assumption at the one moment value moves.**
Universal: who moves the money (every path); what happens when funds cross contracts
(migration/liquidation/distribution); can a stranger trigger it; can someone pay less /
receive more; external call before state update (reentrancy).
- **launchpad** → graduation/migration into the DEX pool; LP lock; fee split; anti-snipe.
- **lending/CDP** → oracle (staleness, manipulation, closed-market equities); liquidation
  math; health factor; LTV; interest; peg.
- **vault** → first-depositor / share inflation; rounding direction; donation attack.
- **perps** → own engine vs wrapper; oracle; funding; liquidation; margin accounting.
- **distribution/index** → snapshot weighting; permissionless crank; flash-inflatable
  balance; claim reentrancy; does it even custody funds.

## 5.3 Bug-class detection playbook (pattern → detection → fix)
1. **Migration pool squat (Uniswap v3/v4).** Graduation calls
   `createAndInitializePoolIfNecessary` with `amountMin=0` and no post-init price check
   → attacker pre-creates the pool at a fake price; migration dumps the raise. *Detect:*
   grep the call + missing `slot0()` check; confirm `getPool()==0` live. *Fix:* read
   `slot0()`, revert on price mismatch.
2. **Permissionless flash-inflated snapshot.** Weight = spot `balanceOf` at an unguarded
   snapshot; fee-free transfers → flash-borrow, get snapshotted, return, collect. *Fix:*
   time-weighted/checkpointed or Merkle snapshot.
3. **Spot-price oracle used as a "TWAP."** A `getPrice` that reads Uniswap v3/v4 `slot0`
   tick (instantaneous) instead of `observe()` time-weighting → single-tx manipulation.
   *Also:* a v3/v4 pool at `observationCardinality==1` returns a spot-equivalent without
   reverting — do not treat that as a real TWAP. *Fix:* real `observe()` window + a
   cardinality/history floor, or a Chainlink feed with staleness + decimals normalization.
4. **Oracle staleness / closed-market.** Equity/RWA feeds go stale nights/weekends; no
   buffer-widen/halt → wrong-price borrow/liquidation.
5. **Keeper-trusted fund movement.** `distribute(holders,amounts)` only checks
   `total<=balance` → compromised keeper drains anywhere. *Fix:* verify amounts on-chain.
6. **Owner backdoor on live positions.** Admin can swap migrator/oracle/fee on already-
   live positions. *Fix:* timelock or per-position snapshot/freeze. (Trust finding unless
   there's a permissionless path.)
7. **First-depositor / share inflation (ERC-4626).** Tiny deposit + direct donation
   inflates share price; next depositor rounds to 0. *Fix:* virtual/dead shares.
8. **Permissionless buy/defend with `minOut=0` and no on-chain floor.** A public
   buyback/defend forwards the caller's `minOut` to the router with no TWAP/spot floor →
   self-sandwich the protocol's own swap. *Tell:* a sibling function IS TWAP-floored and
   this one is not.
9. **Async snapshot manipulation (GMX/perp-integrated vaults).** Two-phase request→
   finalize; if finalize mixes a stale snapshot with live state, MEV profits between
   phases. *Look for:* `pending*Snapshot`/`pending*Price`/`borrowIndex` + a finalize handler.
10. **Fee-on-transfer over-credit.** `deposited[user]+=amount` then `transferFrom` → if
    the token taxes transfers, credited > received. *Only real if the token taxes plain
    wallet transfers* (many launch tokens tax only pool swaps → not FoT; verify `_update`).

## 5.4 Reusable detection heuristics (grep passes on any target)
- **Solmate `SafeTransferLib` + unchecked token address** → transfer to a codeless
  address returns success (phantom-token drain). (OZ `SafeERC20` differs.)
- **`msg.sender`-only guard reachable via a proxy/clone/module** → self-liquidation /
  identity spoof. Compare against a value the caller can't control.
- **Guard on one entry but not the aggregating op** (deposit guarded, `merge`/`split`/
  `transfer` not) → bypass to the same state.
- **Config/init address not checked against a registry/whitelist** → operator sets a
  never-slashable / malicious dependency.
- **A "verify" path that makes an external call to caller-supplied target+data** →
  arbitrary-call primitive.
- **State incremented before a low-level send whose bool return is ignored** → accounting
  corruption on send failure.

## 5.5 Quality gate (adversarial self-review — mandatory before PoC/report)
For each candidate, answer honestly:
- Is the trigger actually reachable by an unprivileged attacker over the wire?
- What exact upstream check might already stop it? (Go read it.)
- Is it a permissionless exploit or centralization-by-design?
- Is it a duplicate of a published audit/known issue?
- What is the honest severity and the bound?
A finding that can't survive this is not a finding.

## 5.6 Diff-hunt method (large / audited targets)
Do not re-hunt the combed audited core. Hunt what no auditor saw:
1. **Deployed-vs-audited diff.** Clone the audit repo (code-423n4/Sherlock/Cantina). Pull
   the deployed impl source from the explorer (Blockscout `/addresses/<proxy>` →
   `implementations` → `/smart-contracts/<impl>`; or Sourcify). Diff; hunt only the
   post-audit deltas — unaudited and uncrowded even on a huge protocol.
2. **Contracts NOT in the competitive-audit scope.** Contests often scope a few files; the
   rest (and the whole deployed set under an Immunefi bounty) are less picked-over.
3. **Post-audit features, integration seams between separately-audited components, and
   cross-chain accounting no single audit owned end to end.**

## 5.7 Recovery / stuck-funds bounties (distinct engagement — usually an unwinnable lottery)

A "recovery bounty" (e.g. Immunefi programs whose ONLY in-scope impact is "unlocking stuck funds")
pays a % of what you actually free (partial counts — e.g. 25% recovered → a min payout, scaling to the
cap for 100%). It pays **nothing for an impossibility proof.** So the entire EV is binary: can a
transaction move the funds, or not. Triage this in an hour before investing days.

**Triage first — a bounty existing is NOT evidence the funds are recoverable.**
- The team usually can't recover it either; they post the bounty *because* they're stuck ("we found the
  bug, recovery is the hard part" = they have no path and are outsourcing hope). Posting is free.
- **Time-open × balance-unmoved is the market's verdict.** A recovery bounty open for years with a large
  prize and the balance still untouched (`eth_getBalance` == the "stuck" figure, contract nonce ~1) means
  the entire hunter population already tried and failed. Strong prior: genuinely bricked.

**Method (verified source + a fork):**
1. Enumerate EVERY value-out primitive (`grep` all files for `call{value`, `.send`, `.transfer`,
   `selfdestruct`, `delegatecall`, payable fns). Old contracts often have exactly ONE ETH exit.
2. For each exit, find its reachability GATE (the state/role/time condition) and ask: can any tx from
   anyone (owner included) satisfy it *now*? Map every writer of the gating storage.
3. **Verify bytecode == verified source** — don't trust source-grep. Compile the verified source with the
   pinned solc and byte-compare to the deployed runtime (mask library-link addresses + the trailing CBOR
   metadata). Gotchas: a `SELFDESTRUCT` opcode hit is often a **false positive inside the metadata
   trailer** (data, not code); `DELEGATECALL`s to a `public library` (SafeMath-style pure math) are
   benign and move no ETH. No proxy/CREATE2/metamorphic = no code-injection recovery.
4. **Storage-surgery test (the decisive question):** which single storage slot, if overwritten, frees the
   funds — and does ANY on-chain function write it? If freeing requires **N simultaneous writes across
   multiple contracts** (proven by a god-mode fork test that rewrites slots and STILL reverts), the only
   "fix" is an L1 irregular state change / hard fork — not a hunter's tool, refused since The DAO =
   **unrecoverable, walk.**
5. Run it as an **adversarial multi-agent harness**: each agent's mandate is to REFUTE impossibility by
   building a fork PoC that extracts the ETH, across distinct vectors (state-machine revival via owner
   levers, dependency-contract exploitation, bytecode/low-level, storage surgery). All vectors failing
   with passing fork tests = a rigorous (but unpaid) impossibility proof.

**Freeze patterns in old TokenMarket / ICO crowdsale contracts (seen live — UTIX, 451 ETH, 4 permanent
locks, any one fatal):**
- Sole ETH exit is a `withdrawContractFund`-style send gated behind `State.Funding`.
- `Funding` needs `block.timestamp <= endsAt`; `setEndsAt`/`setStartsAt` self-`assert(block.timestamp <=
  endsAt/startsAt)` → **unextendable once the window closes.** Permanent.
- `setFinalizeAgent` `assert`s the agent is unset → a broken/insane finalizeAgent can never be re-pointed
  → stuck in `Preparing` forever (`getState` returns `Preparing` while `!finalizeAgent.isSane()`).
- **Inverted `require(extcodesize(multisig)==0)` (EOA-only) in the withdraw** → if the multisig is a Safe
  (a contract), every forward reverts. This is a common *original* stuck-funds cause; `setMultisig` is
  itself gated (`investorCount > MAX`), so it can't be repointed.
- One-way flags (`released`, `mintingFinished`) can't be reversed → the `invest → mint` path dies too.
- `weiRaised` is monotonic and `allocate()` can inflate it above the real ETH balance → the
  "withdraw `weiRaised`" branch becomes **mathematically unsatisfiable** (tries to send more than held).

**Verdict discipline:** most recovery bounties on truly-frozen funds are unwinnable — the honest,
high-value move is a fast rigorous triage, then walk. Wanting it more does not melt steel. Redirect the
same intensity to live, *reachable* targets.

---

# PART 6 — PROOF OF CONCEPT STANDARDS

A PoC is required and is the proof. Standards:
- **Reproducible from one command**, pinned to a block/commit, on a local fork or unit test.
- **Asserts impact in numbers** (attacker gain > cost, or funds stranded/frozen).
- **No mainnet state touched, no live tx broadcast.** Cheatcodes (`vm.*`) are inert on real
  networks by design.
- **Prefer real deployed addresses** on a fork over mocks; when a full fork is impractical
  (thousands of holders / slow RPC), replicate the *exact* vulnerable logic from verified
  source into a self-contained test and cite live on-chain magnitudes.

```bash
mkdir -p poc && cd poc && forge init --no-git . 2>/dev/null; rm -f src/Counter.sol test/*.sol
BLK=$(cast block-number --rpc-url $RPC)
forge test --fork-url $RPC --fork-block-number $BLK -vv
```
```solidity
// SAFE: local fork only. vm.* cheatcodes are inert on real networks. No mainnet state touched.
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;
import {Test, console} from "forge-std/Test.sol";
interface ITarget { /* exact fns from recon */ }
contract PoC is Test {
    address constant TARGET = 0x<core>;
    address attacker = address(0xA11CE);
    function test_exploit() public {
        // 1. record victim/pot state
        // 2. vm.deal / vm.prank attacker; run the value-moving sequence
        // 3. assert the loss in numbers: assertGt(gain, cost, "net profit"); // or funds stranded
    }
}
```

---

# PART 7 — SEVERITY & RISK RATING

Rate on **impact × likelihood**, state the bound, never round up. Align to the target
program's own table when one exists (many SC programs pay Critical/High only).

| Severity | Definition |
|---|---|
| **Critical** | Unauthenticated theft/loss of a large share of funds, or full drain/freeze. One tx, no special role. |
| **High** | Unauthenticated, repeatable theft of user funds but **bounded** (state the bound), or theft needing a common condition. |
| **Medium** | Limited/conditional loss, griefing, or bounded value leak; often condition-gated (e.g. oracle downtime). |
| **Low** | Minor/edge, unlikely, or negligible value. |
| **Trust / centralization** | Owner-by-design power. Disclose and label as such — not a permissionless exploit. |

For consensus/validator targets, map to the program's categories (e.g. Safety violation >
Liveness/Loss-of-availability > DoS), and demonstrate the claimed impact in the PoC.

---

# PART 8 — REPORTING STANDARD

One finding per report. Self-contained, reproducible, honest.
```markdown
# <PROJECT> — <SEVERITY>: <one-line title>
**Researcher:** <you>   **Date:** <date>   **Commit/Block:** <ref>
**Severity:** <Crit/High/Med/Low> — <why; state the bound>
**Status:** Verified on local fork / read-only on-chain. No mainnet state touched.
**Disclosure:** Private; live-exploitable now.  <!-- if applicable -->

## Affected components (<CHAIN>, chainId <CID>)
| Role | Address / file:line |
|---|---|

## Summary
<the one wrong assumption, plain language>

## Root cause
<exact code quoted, with file:line>

## Attack
1. ... 2. ... 3. ...

## Impact
Auth: <none/role> | Capital: <flash/zero/size> | Frequency: <once/every cycle> | Victims: <who> | Magnitude: <number/%>

## Proof of concept
<one-command repro> → <result: attacker gained X / Y stranded>

## Recommended fix
<one or more concrete options>

## Disclosure & remediation
Good-faith private disclosure. Not conditioned on payment — act now. Happy to walk the
team through it and review the fix.
```

---

# PART 9 — RESPONSIBLE DISCLOSURE & REMEDIATION

1. **Find the private channel** (security email, verified account DM, program intake).
   Never a public thread while live. For competitive programs, submit through the exact
   required channel (e.g. GitHub Security Advisory), one finding per submission, all detail
   inline, citing the exact commit; confirm the bug is still live on the target ref.
2. **First contact (short):** who you are, one-line impact, "verified on a fork, nothing
   touched, PoC you can run." Then send the report + PoC.
3. **Share via a PRIVATE repo** (add them as collaborator) — never public until patched.
   ```bash
   gh api user --jq .login   # confirm the RIGHT account FIRST
   gh repo create <project>-security-disclosure --private --source=. --push
   ```
4. **Skepticism is normal.** Point to the reproducible PoC. Stay calm, never threaten.
5. **Name the number AFTER they confirm**, anchored on merit (severity + funds at risk +
   clean disclosure + you helped fix). No program = discretionary; plan a fair range.
6. **Help fix and review the patch** — turns one bounty into repeat work.
7. **If unpaid:** you still have a portfolio piece. After the fix ships, a factual timeline
   post (receipts, no rage, no threats) builds your name. Never bundle "pay or I post."

**Disclosure with no social contact:** if the trace dead-ends at an anon deployer, send a
0-value tx with a neutral memo ("found a security issue, reach me @handle") to the deployer
+ active operator wallet. Never put the vulnerability in it (on-chain = public + permanent).

---

# PART 10 — RUST / SOLANA CONSENSUS & VALIDATOR AUDITING (Track B)

A different game from EVM DeFi: no "drain the vault." The prizes are **safety** (two
conflicting blocks finalized = fork), **liveness** (consensus stalls / loss of
availability), and crypto misuse. Targets: consensus clients (Agave/Alpenglow class),
validator code, BLS aggregation.

## 10.1 Scope discipline & setup
- The bounty repo (rules + advisory intake) is often separate from the CODE repo. Confirm
  which commit/branch is in scope; **moving-master** programs require the bug to be live at
  submit time — diff master right before firing.
- Sparse-clone the in-scope crates to keep it light:
  ```bash
  git clone --no-checkout --depth 1 --filter=blob:none <repo>
  git sparse-checkout set <crate1> <crate2> ...
  ```
  To actually BUILD/run a dynamic harness you'll need the full workspace: `git
  sparse-checkout disable` (fetches the remaining members).
- **Known issues are out of scope:** the program's issue tracker (labels like
  `blocking-ag`/`consensus-team`), public SIMDs/PRs/advisories, prior audits. A matching
  public entry predating your report = ineligible, usually with no partial tier.

## 10.2 Consensus bug classes (the prize list)
1. **Safety break (top):** forge a certificate or finalize below the honest threshold so
   two conflicting blocks finalize. Hunt the stake-threshold check + cert verification.
2. **Liveness break:** stall finalization — bad timeout logic, skip-vote miscount,
   parent-ready deadlock, one party blocking progress. (Only *permanent/unrecoverable*
   stalls are fundable; distinguish from transient delay.)
3. **Stake double-count:** the same validator's stake counted twice. Well-defended clients
   use a bitmap (a bit can't set twice) + disjointness checks on split-votes — so hunt
   **cross-certificate / cross-round accumulation** and the *build* side, not the bitmap.
4. **BLS misuse:** rogue-key if proof-of-possession not enforced (look for `PopVerified`-
   style gating); wrong message/domain (replay a sig across slot/round/epoch/shred_version);
   aggregation-API misuse (verifying an aggregate over a different signer set than claimed).
   `blst`/curve internals are usually out of scope — report upstream.
5. **Threshold / boundary:** wrong constant, or `>=` vs `>` at the exact threshold. Pin to
   the spec (SIMD); test the knife-edge exactly-at-threshold case.
6. **Fraction/arithmetic:** stake-fraction comparisons must widen (u128 cross-mult +
   checked ops). Watch a `saturating_add` hiding a wraparound.
7. **Split-vote / Base3 semantics (deep):** a cert that aggregates two vote sets toward one
   threshold — verify the protocol actually allows combining THOSE sets for THAT cert type.
   Combining the wrong sets toward a *finalizing* threshold would be a safety break.
8. **Equivocation / double-vote:** can a validator vote two conflicting things in a slot?
   Check vote-history dedup AND persistence-before-broadcast (equivocation across restart).
9. **Migration / handover seams:** a client transitioning consensus algorithms carries
   state across the switch — thresholds/stake sets/block-id scheme during transition.
10. **Dissemination / repair-response validation gaps (high yield on hardened targets):**
    the block-id/repair/turbine/shred surface, not the crowded cert crux, is often where the
    yield sits, because the crowd piles on the obvious verification path and the edges get
    less attention. **Highest-yield heuristic — sibling-arm asymmetry:** in a `match` over
    wire messages or response variants, compare adjacent handlers against each other; a
    common gap is one arm validating its input (length, bounds, domain) while an adjacent arm
    is missing an equivalent check. Also compare each handler's checks against what its own
    test suite actually exercises, since an untested branch is a frequent weak point. The
    untrusted entry point is the network RESPONSE path; the local insert/construction path is
    usually signature- or hash-verified and therefore trusted, so spend time on what a remote
    peer can send.

## 10.3 Method
1. **Safety crux first** (smallest crate): cert verification → stake check. Verify dedup
   (bitmap), disjointness (split-votes), threshold values, fraction overflow, `>=`/`>`.
2. **Semantics:** map cert_type → which vote sets are legitimately combinable (class 7).
3. **Liveness:** consensus pool — cert builder, parent-ready tracker, stake counters, timer
   manager. Look for a stall or a way one party blocks progress.
4. **Dissemination/repair** (class 10) — often where the yield is on an elite-hardened
   target: the crowd piles on the cert crux; the bug hides in an un-glamorous edge.
5. **Migration + integration seams.**
6. **Reward/credit accounting** if the client credits stake (double-credit, off-by-one over
   the reward window, forgeable reward cert).
7. **Node self-vote-safety:** vote history + persistence ordering (persist-before-broadcast
   prevents equivocation across restart).

## 10.4 Dynamic harness (worth building on hardened targets)
Static review confirms *shape*; a dynamic harness confirms *behavior* a static read can't
model. Many consensus clients are event-driven: drive the REAL state machine with an
adversarial vote/cert schedule and assert invariants.
- Reuse the crate's own test helpers (test context, signed-vote builders) — they live
  behind the crate's test module, so add your tests INSIDE it (the pool API is often
  `pub(crate)`, unreachable from an external integration test).
- Model the fault model precisely (e.g. strictly <20% byzantine = a bounded set of
  equivocators; honest nodes never double-vote) — otherwise you "break" it by bypassing the
  real defense, which is not a finding.
- Assert safety invariants: no finalize+skip same slot; no two conflicting notarizations; a
  fast-finalized block can't be skipped; split-vote thresholds are exact.
- **Build gotchas (real, from the field):** blobless partial clones need
  `sparse-checkout disable`; `librocksdb-sys` can abort with `dyld: @rpath/libclang.dylib`
  (symlink libclang into a dyld search path + set `LIBCLANG_PATH`); crates gated behind a
  feature flag silently run **0 tests** unless you pass the right `--features`.
- **Limitation:** if timers use non-mockable wall-clock (`Instant::now()`), you cannot prove
  *permanent* liveness stalls dynamically (can't fast-forward time; can't distinguish
  permanent from slow). Liveness is then best argued from a static trace of the recovery
  paths.

## 10.5 Coverage discipline
On a scoped program, build an explicit **coverage matrix**: every enumerated in-scope area
→ who reviewed it → result. "Touched everywhere" means every listed area has a reviewer and
a verdict, not a vibe. Name residuals honestly (surfaces you couldn't reach: non-mockable
liveness, external crates absent from the clone, mid-window master drift, multi-node
emergent behavior).

## 10.6 Reality check
Flagship consensus targets are elite-hardened, heavily audited, and crowded with career
researchers. The obvious cert crux is usually correct. Payable finds come from sustained,
specialist grinding of the harder/edge surfaces (split-vote semantics, liveness,
dissemination/repair, reward accounting), verified by both static trace and (where feasible)
a dynamic harness.

---

# PART 11 — TOOLING & AUTOMATION

## 11.1 AI audit tool stack (free unless noted; verify each repo before running)
| Lifecycle step | Tool | Role |
|---|---|---|
| Recon/map | **pashov/skills · x-ray** | threat model, invariants, entry points before reading |
| Read/diff | **pashov/skills · solidity-auditor** | fast security feedback on target/diff |
| Breadth | **Archethect/sc-auditor** | 6 parallel specialists + Devil's-Advocate trim |
| Depth | **0xiehnnkta/nemesis-auditor** | Feynman "why does this line exist" + coupled-state hunt |
| Precedent | **marchev/claudit** | MCP into Solodit's 20k+ findings for exact precedent |
| Heavy (paid) | **PlamenTSV/plamen** | 20–100 autonomous agents, PoC-gated, judge trims. EVM/Solana/Move |
| Package | **cholakovvv/foundry-poc-mainnet-fork** | finding → submission-ready fork PoC, real addresses |

Suggested chain: **map** (x-ray) → **breadth** (sc-auditor) → **depth** (nemesis) →
**precedent** (claudit) → **heavy, optional** (plamen) → **package** (foundry-poc) →
**disclose** (Part 9). Gap to fill with your own choice: an invariant/fuzz layer (Foundry
`invariant`, Echidna, Medusa, Halmos, Certora).

## 11.2 Multi-agent audit fleet (when authorized)
Pattern that scales coverage: (1) one agent builds a **dupe-exclusion digest** from all
findings files + out-of-scope + invariants; (2) parallel **finders**, one per subsystem
(prioritize un-competed contracts / edge surfaces), each handed the digest; (3) an
adversarial **verifier** per candidate that tries to REFUTE it (real? dupe? in-scope?
centralization-only? honest severity?). Run diverse lenses (standard classes + exotic:
cross-chain-replay, ERC-4337, curve/MEV economics, precision-DoS, whole-system invariant).
Filter every survivor against the program's out-of-scope before believing it. **Verify
agent output yourself** — read the load-bearing code before trusting a finding.

## 11.3 Environment / RPC gotchas
- Public RPCs rate-limit heavy fork traffic (403s under load) — pin `--fork-block-number`
  (caches state), loop-retry, poll `cast block-number` until it clears.
- `eth_getLogs` block-range limits — fetch factory deploy events in chunks, or deploy a
  fresh instance inside the fork test.
- `/tmp` and cwd can reset between shell calls — stage source into a persistent working dir.
- Toolchains can vanish mid-session — `command -v cast || foundryup` before relying on it.
- Background sub-agents can hit a watchdog and stall — take over foreground if so.
- New chains use **Blockscout**, not Etherscan; the `to`-addresses in the app's `eth_call`s
  are the contracts.

## 11.4 Platform economics (before you spend a submission)
- Read the out-of-scope list + severity→payout table first (step 0). Common scope kills:
  MEV/frontrun/sandwich out on single-sequencer L2s; first-depositor out if mitigated;
  fee-on-transfer out for standard ERC20; all trusted-role/admin actions out; every prior-
  audit finding out.
- Some platforms are **pay-to-submit** (a non-refundable fee per report) and rate-limit
  new accounts. Never spend a paid slot on a finding you wouldn't bet the fee is in-scope.

## 11.5 Writing style — strip AI-slop from reports/messages/posts
Cut these tells: "Real talk", "Not…Not…Not…", "It's not just X. It's Y.", "This isn't
about X, it's about Y.", "But here's the thing.", "The truth/reality is…", "What people
don't realize…", "Let's break it down.", "Here's the catch.", "Think about it.", "This
underscores/raises the question…", "The bigger picture…". No em dashes as a crutch, no
emoji in professional reports. Human tone, plain and precise.

## 11.6 The Kensho Engine (corpus + targeting + backtest) — the base-flow multiplier

A companion intelligence engine lives at `bounty-hunt-tracker/kensho-engine/` (stdlib Python, no
deps). It makes every hunt start from what has actually been exploited, not a blank page, and it
gets sharper daily. Three uses:

- **Targeting (run BEFORE reading a live target):** `python3 query.py target <protocol-type>`
  ranks the bug classes that historically hit that protocol type (lending → flash-loan/spot-oracle;
  token → reflection/transfer-tax; governance → takeover; vault → first-depositor/accounting;
  bridge → cross-chain), each with its Kensho §5.3/§5.4 ref and example PoCs. This is how you
  diff-hunt a listed/audited program fast (Part 5.6) — audit the historically-bug-prone surface
  first.
- **Backtest (the flywheel):** `backtest/run.py bench N [ptype]` emits a BLIND worklist (known class
  sealed); audit each (you or an agent fleet), then `score` for catch-rate. Every MISS is a method
  gap you turn into a new heuristic here in §5.3/§5.4. Corpus PoCs are runnable (DeFiHackLabs), so
  `verify` reproduces them on a fork.
- **Grow + learn:** `daily.py` pulls every source (DeFiHackLabs realized-exploits + DeFiVulnLabs
  patterns today; add `pull/<source>.py` for Solodit/C4/Sherlock) and dedupes idempotently;
  `learn.py report` regenerates the prioritization signal + taxonomy gaps + backtest misses. Wire
  `daily.py` to cron / the `schedule` skill so the corpus (and this playbook) compound over time.

`taxonomy.json` is the shared bug-class vocabulary (mapped to the refs in this file); keep it and
§5.3 in sync as new classes are learned.

---

# PART 12 — REFERENCE LIBRARY

- **crypto.training/hacks** — real-world hacks AND audit findings, each as a passing
  Foundry PoC (state + calldata + impact assertion), filterable by source. Study exploit
  structure before hunting a class; reuse the PoC skeletons. Registry format for publishing
  your own fork-proven findings: `github.com/sanbir/evm-hack-poc` +
  `evm-hack-registry`; source: `github.com/AuditWare/AuditVault`.
- **Seed exploit patterns** (each → a grep heuristic in Part 5.4): ZeroLend `merge()`
  bypassing a flash-vote guard; Karak unchecked `slashStore`; Solady ERC-6492
  `AllowSideEffects` arbitrary call; Royco + Solmate codeless-address phantom-token; Base
  `FeeVault` increment-before-failed-send; Licredity `msg.sender`-only guard via proxy.
- **Methodology:** wadgamaraldeen — *My Bug Hunting Methodology* (dated in parts, still
  useful for prompt design).
- **Productivity:** Matt Pocock skills (`/grill-me` → `/to-issues` → goal mode);
  obra/Superpowers capability pack.

---

---

## Authorship & attribution

Authored and maintained by **Duke — @dukedotsol (GitHub: cryptoduke01)**. This playbook
is the product of hands-on offensive security work; the methodology, bug-class library,
heuristics, and disclosure model are the author's own. Adopt and adapt it for your team or
solo practice — retain this authorship credit in copies and derivatives.

© Duke (@dukedotsol). Attribution required.

Adapt the intake, severity table, and disclosure steps to your team's process and each
program's rules. The discipline — authorized scope, fork-only proof, honest severity, kill
your own bad findings, private until patched — is the job.

— Duke (@dukedotsol)
