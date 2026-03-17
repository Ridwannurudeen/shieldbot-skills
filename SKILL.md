---
name: shieldbot-security
description: Scan smart contracts, detect honeypots, check deployer campaigns, and monitor threats on BNB Chain and 6 EVM networks via ShieldBot's API. Use when analyzing contract safety, checking rug pull risk, or reviewing DeFi security.
allowed-tools: Bash(curl:*)
license: MIT
metadata:
  author: Ridwannurudeen
  version: '1.0.0'
---

# ShieldBot Security Skill

ShieldBot is an autonomous transaction firewall for BNB Chain and EVM networks. It provides real-time contract scanning, honeypot detection, deployer campaign analysis, transaction simulation, and threat intelligence through a public REST API.

**Base URL:** `https://api.shieldbotsecurity.online`

No API key is required for public endpoints.

## Quick Start

Scan a contract in one call:

```bash
curl -X POST https://api.shieldbotsecurity.online/api/scan \
  -H "Content-Type: application/json" \
  -d '{"address": "0x...", "chainId": 56}'
```

## Core Endpoints

### 1. Contract Scan — `POST /api/scan`

Quick security scan of a smart contract. Returns risk score, classification, flags, and honeypot analysis.

**Request:**
```json
{
  "address": "0xContractAddress",
  "chainId": 56
}
```

**Response fields:**
- `risk_score` (0-100): Overall risk rating
- `classification`: `SAFE` | `LOW_RISK` | `MEDIUM_RISK` | `HIGH_RISK` | `CRITICAL`
- `flags`: Array of detected issues (e.g., `"hidden_mint"`, `"honeypot"`, `"proxy_contract"`)
- `honeypot`: Object with `is_honeypot` boolean, `buy_tax`, `sell_tax`, `transfer_tax`
- `token_info`: Name, symbol, total supply, owner
- `contract_info`: Verified status, proxy detection, source availability

**When to use:** User asks "Is this contract safe?", "Check this token", "Scan 0x..."

### 2. Transaction Firewall — `POST /api/firewall`

Full transaction analysis with simulation. Classifies risk before a user signs.

**Request:**
```json
{
  "to": "0xContractAddress",
  "from": "0xUserWallet",
  "value": "0x0",
  "data": "0xCalldata",
  "chainId": 56
}
```

**Response fields:**
- `classification`: Risk level of the transaction
- `risk_score` (0-100)
- `danger_signals`: Array of specific warnings
- `simulation`: Simulated outcome (asset changes, gas estimate)
- `recommendation`: `PROCEED` | `CAUTION` | `BLOCK`

**When to use:** User wants to check a transaction before signing, asks "Is this tx safe?"

### 3. Deployer Campaign — `GET /api/campaign/{address}`

Analyzes a deployer's history to detect serial scammers. Maps all contracts deployed by an address and cross-references with known scam patterns.

**Response fields:**
- `deployer`: Address analyzed
- `contracts_deployed`: Array of contracts with risk scores
- `campaign_risk`: Overall deployer risk assessment
- `pattern`: Detected pattern (e.g., `"serial_rugpull"`, `"legitimate_developer"`)
- `total_contracts`: Number of contracts deployed

**When to use:** User asks "Who deployed this?", "Is this developer legit?", "Check deployer history"

### 4. Threat Feed — `GET /api/threats/feed`

Real-time threat intelligence feed. Returns recently detected threats across monitored chains.

**Query parameters:**
- `chainId` (optional): Filter by chain
- `limit` (optional): Number of results (default 20)
- `severity` (optional): Filter by severity (`low`, `medium`, `high`, `critical`)

**Response fields:**
- `threats`: Array of threat objects with `type`, `address`, `chain`, `severity`, `timestamp`, `description`

**When to use:** User asks "What threats are active?", "Show recent scams on BSC"

### 5. Phishing Check — `GET /api/phishing?url={url}`

Checks a URL against known phishing databases and heuristic analysis.

**Response fields:**
- `is_phishing`: Boolean
- `confidence`: Detection confidence (0-1)
- `reasons`: Array of detection reasons
- `domain_info`: Registration date, registrar, similarity to known brands

**When to use:** User shares a suspicious URL, asks "Is this site legit?"

### 6. Wallet Rescue — `GET /api/rescue/{wallet}`

Scans a wallet's token approvals and flags risky allowances that should be revoked.

**Response fields:**
- `wallet`: Address scanned
- `approvals`: Array of active approvals with `token`, `spender`, `allowance`, `risk_level`
- `risky_approvals`: Count of approvals flagged as risky
- `recommendation`: Suggested actions

**When to use:** User asks "Check my approvals", "Am I exposed?", "Scan my wallet"

### 7. AI Advisor — `POST /api/agent/chat`

Conversational AI security advisor. Answers questions about DeFi security, explains risks, and provides guidance.

**Request:**
```json
{
  "message": "Is yield farming on PancakeSwap safe?",
  "context": {
    "chainId": 56
  }
}
```

**Response fields:**
- `response`: AI-generated security advice
- `references`: Related threats or contracts mentioned

**When to use:** User asks general DeFi security questions

## Supported Chains

| Chain | chainId |
|-------|---------|
| BNB Smart Chain | `56` |
| Ethereum | `1` |
| Base | `8453` |
| Arbitrum | `42161` |
| Polygon | `137` |
| Optimism | `10` |
| opBNB | `204` |

Always pass `chainId` as an integer.

## Interpreting Results

### Risk Score Ranges
| Score | Meaning |
|-------|---------|
| 0-20 | Safe — no issues detected |
| 21-40 | Low risk — minor concerns, generally safe |
| 41-60 | Medium risk — proceed with caution |
| 61-80 | High risk — significant red flags |
| 81-100 | Critical — likely scam or honeypot |

### Key Flags to Watch For
- `honeypot` — Cannot sell tokens after buying
- `hidden_mint` — Owner can mint unlimited tokens
- `blacklist` — Owner can blacklist addresses from selling
- `proxy_contract` — Logic can be changed at any time
- `high_sell_tax` — Sell tax above 10%
- `ownership_not_renounced` — Owner retains privileged access
- `liquidity_unlocked` — LP tokens are not locked

## Example Workflows

### Audit a Token Before Buying
1. `POST /api/scan` with the token contract address
2. Check `honeypot.is_honeypot` — if true, do not buy
3. Check `risk_score` — above 60 means significant risk
4. Check `flags` for `hidden_mint`, `blacklist`, `high_sell_tax`
5. `GET /api/campaign/{deployer}` to check the deployer's history

### Check if a Contract is a Honeypot
1. `POST /api/scan` with the contract address
2. Read `honeypot.is_honeypot`, `honeypot.buy_tax`, `honeypot.sell_tax`
3. If `sell_tax > 50%` or `is_honeypot == true`, warn the user

### Scan Wallet Approvals for Risky Contracts
1. `GET /api/rescue/{wallet_address}`
2. List all `risky_approvals`
3. For each risky approval, recommend revoking via the token's `approve(spender, 0)` call

### Analyze a Suspicious Transaction
1. `POST /api/firewall` with the transaction parameters
2. Check `recommendation` — if `BLOCK`, warn the user immediately
3. Review `danger_signals` for specific threats
4. Check `simulation.asset_changes` to see what the tx would do
