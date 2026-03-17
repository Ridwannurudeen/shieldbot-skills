# ShieldBot Skills

Claude Code skills for [ShieldBot Security](https://shieldbotsecurity.online) — the autonomous transaction firewall for BNB Chain and EVM networks.

## What This Repo Is

This is a **skills repository** for the [BNB Chain Skills Hub](https://github.com/bnb-chain/skills-hub). It contains a `SKILL.md` file that teaches Claude Code how to use ShieldBot's public API to scan smart contracts, detect honeypots, analyze deployer campaigns, and monitor threats.

## Installation

Add this repo as a Claude Code skill:

```bash
claude skill add https://github.com/Ridwannurudeen/shieldbot-skills
```

Or reference the `SKILL.md` directly in your project's `.claude/skills/` directory.

## What You Can Do

Once installed, Claude Code can:

- **Scan any contract** for rug pull risk, honeypot traps, and malicious patterns
- **Analyze transactions** before signing — classify risk and simulate outcomes
- **Check deployer history** to detect serial scammers
- **Monitor threat feeds** for real-time intelligence
- **Check URLs** for phishing attempts
- **Scan wallet approvals** and flag risky token allowances

## Supported Chains

| Chain | ID |
|-------|-----|
| BNB Smart Chain | 56 |
| Ethereum | 1 |
| Base | 8453 |
| Arbitrum | 42161 |
| Polygon | 137 |
| Optimism | 10 |
| opBNB | 204 |

## Repository Structure

```
SKILL.md                    # Core skill definition
references/api-reference.md # Full API endpoint documentation
README.md                   # This file
```

## Links

- [ShieldBot Website](https://shieldbotsecurity.online)
- [ShieldBot Product Repo](https://github.com/Ridwannurudeen/shieldbot)

## License

MIT
