<div align="center">

# 検証 · Kensho

### An industry-grade Web3 security assessment playbook

EVM smart contracts and DeFi, plus Rust and Solana consensus.
Prove every finding. Never assume.

<br/>

`authorize` · `scope` · `recon` · `analyze` · `prove` · `rate` · `report` · `disclose`

</div>

---

## What Kensho is

**Kensho (検証)** means verification. That single idea is the spine of this playbook: a finding is not real until it is proven on a fork, in a harness, or by `eth_call`, and a live system is never exploited.

Kensho is an end-to-end methodology for finding, proving, rating, and responsibly disclosing vulnerabilities in Web3 systems. It is written so a solo researcher can run it top to bottom, and so a team can adopt it as a shared standard. Every step ships with concrete commands, decision gates, quality bars, and fill-in templates.

## Who it is for

- Independent security researchers and bug bounty hunters.
- Audit teams and security firms that want a repeatable, defensible process.
- Protocol engineers who want to understand how their systems get attacked.

## Two tracks, one spine

| Track | Surface |
| :--- | :--- |
| **A** | EVM smart contracts and DeFi protocols |
| **B** | Rust and Solana consensus and validator code |

Both run through the same lifecycle and hold to the same discipline.

## The lifecycle

| Phase | What happens |
| :--- | :--- |
| 0 | Authorize and scope |
| 1 | Intake |
| 2 | Verify audit status |
| 3 | Recon and mapping |
| 4 | Analysis |
| 5 | Proof of concept |
| 6 | Severity and risk rating |
| 7 | Reporting |
| 8 | Disclosure and fix review |

## What is inside

- **Rules of engagement.** Authorization, verification safety, and integrity, stated as hard rules.
- **A bug-class library.** Ten EVM classes and ten consensus classes, each with pattern, detection, and fix.
- **Reusable detection heuristics.** Grep-level patterns that map to real historical exploits.
- **A severity taxonomy.** Impact times likelihood, with the bound always stated, and centralization separated from a permissionless exploit.
- **Proof-of-concept and reporting standards.** Reproducible from one command, asserted in numbers, one finding per report.
- **A full Rust and Solana consensus track.** Scope discipline, the consensus prize list, and a method for building and running a dynamic byzantine harness.
- **Tooling and automation.** An AI audit tool stack, a multi-agent fleet pattern, and environment gotchas learned in the field.

## How to use it

**As a Claude Code or Agent skill.** Place this folder in your skills directory and invoke it by name (`kensho`). The agent runs the lifecycle for you.

**As a manual playbook.** Open [`SKILL.md`](SKILL.md) and work top to bottom. Every step is copy-paste ready.

## Principles that never bend

1. Authorized scope only.
2. Fork, harness, or `eth_call` proof only. Never exploit a live system.
3. Honest severity, always. State the bound.
4. Separate centralization risk from a permissionless exploit.
5. Kill your own findings when they do not hold up.
6. Private until patched. Ask, never threaten.

## Repository contents

| File | Purpose |
| :--- | :--- |
| [`SKILL.md`](SKILL.md) | The full playbook |
| `README.md` | This overview |
| `LICENSE` | Attribution terms |

## Author

Created and maintained by **Duke** ([@dukedotsol](https://x.com/dukedotsol), GitHub [cryptoduke01](https://github.com/cryptoduke01)). Built from real offensive security engagements across EVM DeFi and Rust and Solana consensus. Field-tested, not theoretical.

## License

Licensed under **Creative Commons Attribution 4.0 International (CC BY 4.0)**. You are free to share and adapt this work, including for commercial use, as long as you keep attribution to Duke ([@dukedotsol](https://x.com/dukedotsol)). See [`LICENSE`](LICENSE).

<div align="center">
<br/>

*A finding is not real until it is proven.*

</div>
