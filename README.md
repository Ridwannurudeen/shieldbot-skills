# ShieldBot Skills

Claude Code skills for [ShieldBot Security](https://shieldbotsecurity.online) — the autonomous transaction firewall for BNB Chain and EVM networks.

## What This Repo Is

This is a **skills repository** for the [BNB Chain Skills Hub](https://github.com/bnb-chain/skills-hub). It contains a `SKILL.md` file that teaches Claude Code (and other AI agents) how to use ShieldBot's public API to scan smart contracts, detect honeypots, analyze deployer campaigns, simulate transactions, and monitor threats across 7 EVM chains.

## Installation

Add this repo as a Claude Code skill:

```bash
claude skill add https://github.com/Ridwannurudeen/shieldbot-skills
```

Or reference the `SKILL.md` directly in your project's `.claude/skills/` directory.

## What You Can Do

Once installed, your AI agent can:

- **Scan any contract** — risk score, honeypot detection, flag analysis, tax breakdown
- **Analyze transactions** — simulate before signing, classify risk, surface danger signals
- **Investigate deployers** — detect serial scammers, map deployment campaigns
- **Monitor threat feeds** — real-time threat intelligence across all supported chains
- **Check URLs for phishing** — domain age, brand impersonation, phishing database lookup
- **Audit wallet approvals** — find risky token allowances, recommend revocations
- **Ask security questions** — conversational AI advisor for DeFi security guidance

## Quick Start

```bash
# Scan a token on BSC
curl -s -X POST https://api.shieldbotsecurity.online/api/scan \
  -H "Content-Type: application/json" \
  -d '{"address": "0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82", "chainId": 56}'
```

## Supported Chains

| Chain | ID | Native Token |
|-------|-----|-------------|
| BNB Smart Chain | 56 | BNB |
| Ethereum | 1 | ETH |
| Base | 8453 | ETH |
| Arbitrum | 42161 | ETH |
| Polygon | 137 | MATIC |
| Optimism | 10 | ETH |
| opBNB | 204 | BNB |

## Repository Structure

```
SKILL.md                       # Core skill definition (frontmatter + instructions)
references/
  api-reference.md             # Full API endpoint documentation
  chains.json                  # Chain configuration data
  risk-flags.json              # Risk flag definitions and severity levels
README.md                      # This file
LICENSE                        # MIT
```

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/scan` | POST | Contract security scan |
| `/api/firewall` | POST | Transaction analysis + simulation |
| `/api/campaign/{address}` | GET | Deployer campaign analysis |
| `/api/threats/feed` | GET | Real-time threat intelligence |
| `/api/phishing?url=` | GET | URL phishing check |
| `/api/rescue/{wallet}` | GET | Wallet approval audit |
| `/api/agent/chat` | POST | AI security advisor |

No API key required. Rate limit: 30 requests/minute.

## Links

- [ShieldBot Website](https://shieldbotsecurity.online)
- [ShieldBot Product Repo](https://github.com/Ridwannurudeen/shieldbot)
- [BNB Chain Skills Hub](https://github.com/bnb-chain/skills-hub)

## License

MIT
