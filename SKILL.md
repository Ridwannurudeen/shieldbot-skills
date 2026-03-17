---
name: shieldbot-security
description: Scan smart contracts, detect honeypots, check deployer campaigns, and monitor threats on BNB Chain and 6 EVM networks via ShieldBot's API. Use when user asks about contract safety, rug pull risk, honeypot detection, wallet approvals, phishing URLs, deployer history, or DeFi security on any supported EVM chain.
allowed-tools: Bash(curl:*), Bash(python:*), Bash(node:*), Bash(npx:*), WebFetch
model: opus
license: MIT
metadata:
  author: Ridwannurudeen
  version: '1.0.0'
---

# ShieldBot Security Skill

ShieldBot is an autonomous transaction firewall for BNB Chain and EVM networks. Three AI agents — Hunter (proactive threat sweeps), Sentinel (real-time monitoring), and Advisor (conversational guidance) — power a REST API that scans contracts, detects honeypots, analyzes deployer campaigns, simulates transactions, and delivers threat intelligence.

**Base URL:** `https://api.shieldbotsecurity.online`

No API key required. All endpoints are public.

## Quick Decision Guide

| User wants to... | Use this endpoint | Method |
|---|---|---|
| Check if a token is safe | `/api/scan` | POST |
| Analyze a transaction before signing | `/api/firewall` | POST |
| Investigate a deployer's history | `/api/campaign/{address}` | GET |
| See active threats on a chain | `/api/threats/feed` | GET |
| Verify if a URL is phishing | `/api/phishing` | GET |
| Audit wallet token approvals | `/api/rescue/{wallet}` | GET |
| Ask a DeFi security question | `/api/agent/chat` | POST |

## Architecture

```text
                          ShieldBot Security Platform
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│  ┌──────────┐   ┌──────────────┐   ┌──────────────┐                │
│  │  Hunter   │   │   Sentinel   │   │   Advisor    │                │
│  │  Agent    │   │   Agent      │   │   Agent      │                │
│  │          │   │              │   │              │                │
│  │ 30-min   │   │ Real-time    │   │ Conversational│               │
│  │ threat   │   │ transaction  │   │ security     │                │
│  │ sweeps   │   │ monitoring   │   │ guidance     │                │
│  └────┬─────┘   └──────┬───────┘   └──────┬───────┘                │
│       │                │                   │                        │
│  ┌────▼────────────────▼───────────────────▼───────┐                │
│  │              Analysis Engine                     │                │
│  │  Contract Scanner · Honeypot Detector            │                │
│  │  Deployer Graph · Tx Simulator · Phishing DB     │                │
│  └────────────────────┬────────────────────────────┘                │
│                       │                                             │
│  ┌────────────────────▼────────────────────────────┐                │
│  │              Multi-Chain RPC Layer               │                │
│  │  BSC · ETH · Base · Arbitrum · Polygon · OP     │                │
│  └──────────────────────────────────────────────────┘                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
        │
        ▼  REST API (https://api.shieldbotsecurity.online)
┌───────────────┐
│  Your Agent / │
│  Claude Code  │
│  / Frontend   │
└───────────────┘
```

## Quick Start

### curl

```bash
curl -s -X POST https://api.shieldbotsecurity.online/api/scan \
  -H "Content-Type: application/json" \
  -d '{"address": "0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82", "chainId": 56}' | python3 -m json.tool
```

### Python

```python
import requests

resp = requests.post("https://api.shieldbotsecurity.online/api/scan", json={
    "address": "0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82",
    "chainId": 56
})
data = resp.json()["data"]

print(f"Risk: {data['risk_score']}/100 ({data['classification']})")
print(f"Honeypot: {data['honeypot']['is_honeypot']}")
print(f"Flags: {', '.join(data['flags']) or 'None'}")
```

### JavaScript (Node.js)

```javascript
const resp = await fetch("https://api.shieldbotsecurity.online/api/scan", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    address: "0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82",
    chainId: 56
  })
});
const { data } = await resp.json();

console.log(`Risk: ${data.risk_score}/100 (${data.classification})`);
console.log(`Honeypot: ${data.honeypot.is_honeypot}`);
console.log(`Flags: ${data.flags.join(", ") || "None"}`);
```

---

## Core Endpoints

### 1. Contract Scan — `POST /api/scan`

Quick security scan of a smart contract. Returns risk score, classification, flags, and honeypot analysis.

> **IMPORTANT:** Always check `honeypot.is_honeypot` first. If true, the token cannot be sold — warn the user immediately regardless of risk score.

**Request:**
```json
{
  "address": "0xContractAddress",
  "chainId": 56
}
```

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `risk_score` | int (0-100) | Overall risk rating |
| `classification` | string | `SAFE` \| `LOW_RISK` \| `MEDIUM_RISK` \| `HIGH_RISK` \| `CRITICAL` |
| `flags` | string[] | Detected issues (e.g., `"hidden_mint"`, `"honeypot"`) |
| `honeypot.is_honeypot` | bool | Whether tokens can be sold |
| `honeypot.buy_tax` | float | Buy tax percentage |
| `honeypot.sell_tax` | float | Sell tax percentage |
| `honeypot.transfer_tax` | float | Transfer tax percentage |
| `token_info.name` | string | Token name |
| `token_info.symbol` | string | Token symbol |
| `token_info.total_supply` | string | Total supply (wei) |
| `token_info.owner` | string | Contract owner address |
| `contract_info.verified` | bool | Whether source is verified on explorer |
| `contract_info.is_proxy` | bool | Whether contract is upgradeable proxy |
| `contract_info.has_source` | bool | Whether source code is available |

**When to use:** User asks "Is this contract safe?", "Check this token", "Scan 0x...", "Is this a scam?", "Should I buy this token?"

**Complete example:**
```bash
# Scan CAKE token on BSC
curl -s -X POST https://api.shieldbotsecurity.online/api/scan \
  -H "Content-Type: application/json" \
  -d '{"address": "0x0E09FaBB73Bd3Ade0a17ECC321fD13a19e81cE82", "chainId": 56}'
```

---

### 2. Transaction Firewall — `POST /api/firewall`

Full transaction analysis with simulation. Classifies risk before a user signs. The simulation shows exactly what assets will move.

> **IMPORTANT:** If `recommendation` is `BLOCK`, warn the user immediately and explain why using the `danger_signals` array. Do NOT advise proceeding with blocked transactions.

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
| Field | Type | Description |
|-------|------|-------------|
| `classification` | string | Risk level of the transaction |
| `risk_score` | int (0-100) | Overall risk rating |
| `danger_signals` | string[] | Specific warnings (e.g., "Unlimited token approval") |
| `simulation.success` | bool | Whether simulation succeeded |
| `simulation.gas_estimate` | int | Estimated gas usage |
| `simulation.asset_changes` | object[] | What tokens/ETH will move and in which direction |
| `recommendation` | string | `PROCEED` \| `CAUTION` \| `BLOCK` |
| `contract_info.verified` | bool | Whether target contract is verified |

**When to use:** User wants to check a transaction before signing, asks "Is this tx safe?", "What does this transaction do?", "Should I approve this?"

**Complete example:**
```python
import requests

resp = requests.post("https://api.shieldbotsecurity.online/api/firewall", json={
    "to": "0x10ED43C718714eb63d5aA57B78B54704E256024E",  # PancakeSwap Router
    "from": "0xYourWallet",
    "value": "0x0",
    "data": "0x...",  # swap calldata
    "chainId": 56
})
result = resp.json()["data"]

if result["recommendation"] == "BLOCK":
    print("DANGER — Do not sign this transaction!")
    for signal in result["danger_signals"]:
        print(f"  - {signal}")
elif result["recommendation"] == "CAUTION":
    print("Warning — Review before signing:")
    for change in result["simulation"]["asset_changes"]:
        print(f"  {change['direction'].upper()}: {change['amount']} {change['symbol']}")
else:
    print("Transaction appears safe to sign.")
```

---

### 3. Deployer Campaign — `GET /api/campaign/{address}`

Analyzes a deployer's full history to detect serial scammers. Maps every contract an address has deployed and cross-references with known scam patterns.

> **IMPORTANT:** A deployer with `pattern: "serial_rugpull"` or `campaign_risk: "CRITICAL"` is a strong signal. Warn the user that the same wallet has deployed multiple scam tokens, even if the current token's scan looks clean — scammers often launch "clean" tokens initially, then rug later.

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `deployer` | string | Address analyzed |
| `total_contracts` | int | Number of contracts deployed |
| `contracts_deployed` | object[] | Array with address, name, chain_id, risk_score, status |
| `campaign_risk` | string | `LOW` \| `MEDIUM` \| `HIGH` \| `CRITICAL` |
| `pattern` | string | `"serial_rugpull"` \| `"pump_and_dump"` \| `"legitimate_developer"` \| `"unknown"` |
| `stats.avg_risk_score` | float | Average risk across all deployed contracts |
| `stats.rugpulled_count` | int | Number of confirmed rug pulls |
| `stats.active_count` | int | Number of still-active contracts |

**When to use:** User asks "Who deployed this?", "Is this developer legit?", "Check deployer history", "Has this wallet deployed scams before?"

**Complete example:**
```bash
# Check deployer history
curl -s "https://api.shieldbotsecurity.online/api/campaign/0xDeployerAddress" | python3 -m json.tool

# Filter to BSC only
curl -s "https://api.shieldbotsecurity.online/api/campaign/0xDeployerAddress?chainId=56"
```

---

### 4. Threat Feed — `GET /api/threats/feed`

Real-time threat intelligence feed. Returns recently detected threats across all monitored chains. Powered by the Hunter agent's 30-minute sweep cycles.

**Query parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `chainId` | int | all | Filter by chain |
| `limit` | int | 20 (max 100) | Number of results |
| `severity` | string | all | `low` \| `medium` \| `high` \| `critical` |
| `offset` | int | 0 | Pagination offset |

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `threats[].id` | string | Unique threat identifier |
| `threats[].type` | string | `"honeypot"` \| `"rugpull"` \| `"phishing"` \| `"exploit"` |
| `threats[].address` | string | Contract or wallet involved |
| `threats[].chain_id` | int | Chain where threat was detected |
| `threats[].severity` | string | Severity level |
| `threats[].description` | string | Human-readable description |
| `threats[].timestamp` | string | ISO 8601 detection time |
| `total` | int | Total matching threats |

**When to use:** User asks "What threats are active?", "Show recent scams on BSC", "Any new rug pulls?", "What's happening on chain?"

**Complete example:**
```bash
# Latest critical threats on BSC
curl -s "https://api.shieldbotsecurity.online/api/threats/feed?chainId=56&severity=critical&limit=5"

# All threats across all chains
curl -s "https://api.shieldbotsecurity.online/api/threats/feed?limit=50"
```

---

### 5. Phishing Check — `GET /api/phishing?url={url}`

Checks a URL against known phishing databases, brand impersonation detection, and domain age heuristics.

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `is_phishing` | bool | Whether the URL is flagged as phishing |
| `confidence` | float (0-1) | Detection confidence |
| `reasons` | string[] | Why it was flagged |
| `domain_info.registration_date` | string | When the domain was registered |
| `domain_info.registrar` | string | Domain registrar |
| `domain_info.similar_to` | string | Legitimate brand it may impersonate |

**When to use:** User shares a suspicious URL, asks "Is this site legit?", "Is this a phishing link?", "Can I trust this website?"

**Complete example:**
```bash
# Check a suspicious URL (URL-encode the parameter)
curl -s "https://api.shieldbotsecurity.online/api/phishing?url=https%3A%2F%2Fpancakesvvap.finance"
```

**Decision logic:**
```
if is_phishing == true AND confidence >= 0.8:
    → "This is almost certainly a phishing site. Do NOT connect your wallet."
elif is_phishing == true AND confidence < 0.8:
    → "This URL has phishing indicators. Proceed with extreme caution."
elif domain_info.registration_date < 30 days ago:
    → "This is a very new domain. Be cautious."
else:
    → "No phishing indicators detected."
```

---

### 6. Wallet Rescue — `GET /api/rescue/{wallet}`

Scans all token approvals for a wallet and flags risky allowances. Identifies contracts that have unlimited spending permission on the user's tokens.

> **IMPORTANT:** Unlimited approvals to unverified contracts are high-risk. For each risky approval, explain what the approved contract could do (drain all tokens of that type) and provide the revoke calldata: `approve(spender, 0)`.

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `wallet` | string | Address scanned |
| `chain_id` | int | Chain scanned |
| `total_approvals` | int | Total active approvals |
| `risky_approvals` | int | Number flagged as risky |
| `approvals[].token` | object | Token address, symbol, name |
| `approvals[].spender` | object | Spender address, label, verified status |
| `approvals[].allowance` | string | Amount approved (`"unlimited"` or value) |
| `approvals[].risk_level` | string | `LOW` \| `MEDIUM` \| `HIGH` \| `CRITICAL` |
| `approvals[].reason` | string | Why it's flagged |
| `recommendation` | string | Summary of recommended actions |

**When to use:** User asks "Check my approvals", "Am I exposed?", "Scan my wallet for risks", "What contracts can spend my tokens?"

**Complete example:**
```python
import requests

resp = requests.get("https://api.shieldbotsecurity.online/api/rescue/0xYourWallet?chainId=56")
data = resp.json()["data"]

print(f"Total approvals: {data['total_approvals']}")
print(f"Risky: {data['risky_approvals']}")

for approval in data["approvals"]:
    if approval["risk_level"] in ("HIGH", "CRITICAL"):
        print(f"\nREVOKE: {approval['token']['symbol']}")
        print(f"  Spender: {approval['spender']['address']} ({approval['spender']['label']})")
        print(f"  Allowance: {approval['allowance']}")
        print(f"  Reason: {approval['reason']}")
```

---

### 7. AI Advisor — `POST /api/agent/chat`

Conversational AI security advisor powered by the Advisor agent. Answers questions about DeFi security, explains scan results, and provides actionable guidance.

**Request:**
```json
{
  "message": "Is yield farming on PancakeSwap safe?",
  "context": {
    "chainId": 56,
    "address": "0xOptionalContractAddress"
  }
}
```

**Response fields:**
| Field | Type | Description |
|-------|------|-------------|
| `response` | string | AI-generated security advice |
| `references` | object[] | Related contracts or threats mentioned |

**When to use:** User asks general DeFi security questions, wants explanations of results, or needs guidance on best practices.

---

## Supported Chains

| Chain | chainId | Native Token | Explorer |
|-------|---------|-------------|----------|
| BNB Smart Chain | `56` | BNB | bscscan.com |
| Ethereum | `1` | ETH | etherscan.io |
| Base | `8453` | ETH | basescan.org |
| Arbitrum | `42161` | ETH | arbiscan.io |
| Polygon | `137` | MATIC | polygonscan.com |
| Optimism | `10` | ETH | optimistic.etherscan.io |
| opBNB | `204` | BNB | opbnbscan.com |

Always pass `chainId` as an integer. If the user doesn't specify a chain, default to BSC (`56`).

---

## Interpreting Results

### Risk Score Decision Tree

```text
risk_score
├── 0-20  (SAFE)       → "No issues detected. Contract appears safe."
├── 21-40 (LOW_RISK)   → "Minor concerns. Generally safe to interact with."
├── 41-60 (MEDIUM_RISK)→ "Proceed with caution. Review flags before interacting."
├── 61-80 (HIGH_RISK)  → "Significant red flags detected. Recommend avoiding."
└── 81-100 (CRITICAL)  → "Almost certainly a scam or honeypot. Do NOT interact."
```

### Flag Severity Reference

| Flag | Severity | What it means | User impact |
|------|----------|---------------|-------------|
| `honeypot` | CRITICAL | Cannot sell tokens after buying | Total loss of funds |
| `hidden_mint` | CRITICAL | Owner can mint unlimited tokens | Infinite dilution, dump |
| `blacklist` | HIGH | Owner can block addresses from selling | Selective rug pull |
| `proxy_contract` | HIGH | Contract logic can be changed any time | All bets are off |
| `high_sell_tax` | HIGH | Sell tax above 10% | Significant loss on every sell |
| `ownership_not_renounced` | MEDIUM | Owner retains admin privileges | Could change rules later |
| `liquidity_unlocked` | MEDIUM | LP tokens are not locked | Deployer can pull liquidity |
| `high_buy_tax` | LOW | Buy tax above 5% | Minor cost on entry |
| `no_source_code` | LOW | Source not verified on explorer | Cannot audit contract |

### Presenting Results to Users

When presenting scan results, always follow this order:

1. **Lead with the verdict** — classification + risk score
2. **Honeypot status** — this is the #1 concern for token buyers
3. **Critical flags** — any deal-breakers
4. **Tax breakdown** — buy/sell/transfer taxes
5. **Contract details** — verified, proxy, owner
6. **Recommendation** — clear action: buy/avoid/research more

Example output format:
```
## Token Scan: $EXAMPLE (0x...)

**Risk: 75/100 (HIGH_RISK)**

Honeypot: No
Buy Tax: 5% | Sell Tax: 12% | Transfer Tax: 0%

Flags:
- hidden_mint — Owner can mint unlimited tokens
- ownership_not_renounced — Owner retains admin access

Contract: Verified | Not a proxy
Owner: 0x...

Recommendation: AVOID — Hidden mint capability means the owner
can dilute your holdings to zero at any time.
```

---

## Example Workflows

### 1. Full Token Due Diligence (Most Common)

The complete workflow for when a user asks "Is this token safe to buy?"

```python
import requests

BASE = "https://api.shieldbotsecurity.online"
TOKEN = "0xTokenAddress"
CHAIN = 56

# Step 1: Scan the contract
scan = requests.post(f"{BASE}/api/scan", json={"address": TOKEN, "chainId": CHAIN}).json()["data"]

# Step 2: Immediate deal-breakers
if scan["honeypot"]["is_honeypot"]:
    print("STOP — This is a honeypot. You cannot sell after buying.")
    exit()

if scan["risk_score"] >= 80:
    print(f"STOP — Critical risk score: {scan['risk_score']}/100")
    print(f"Flags: {', '.join(scan['flags'])}")
    exit()

# Step 3: Check deployer history
if scan.get("token_info", {}).get("owner"):
    campaign = requests.get(f"{BASE}/api/campaign/{scan['token_info']['owner']}").json()["data"]
    if campaign["pattern"] == "serial_rugpull":
        print(f"STOP — Deployer is a serial scammer ({campaign['stats']['rugpulled_count']} confirmed rugs)")
        exit()

# Step 4: Present findings
print(f"Risk: {scan['risk_score']}/100 ({scan['classification']})")
print(f"Buy Tax: {scan['honeypot']['buy_tax']}% | Sell Tax: {scan['honeypot']['sell_tax']}%")
print(f"Flags: {', '.join(scan['flags']) or 'None'}")
print(f"Deployer risk: {campaign.get('campaign_risk', 'N/A')}")
```

### 2. Pre-Swap Safety Check

Before a user swaps tokens on a DEX:

```python
import requests

BASE = "https://api.shieldbotsecurity.online"

# Step 1: Scan the token being bought
scan = requests.post(f"{BASE}/api/scan", json={
    "address": "0xTokenToBuy",
    "chainId": 56
}).json()["data"]

# Step 2: If token scan passes, analyze the swap transaction
if scan["risk_score"] < 60 and not scan["honeypot"]["is_honeypot"]:
    firewall = requests.post(f"{BASE}/api/firewall", json={
        "to": "0xRouterAddress",
        "from": "0xUserWallet",
        "value": "0x" + hex(int(0.1 * 10**18))[2:],  # 0.1 BNB
        "data": "0xSwapCalldata",
        "chainId": 56
    }).json()["data"]

    if firewall["recommendation"] == "BLOCK":
        print("Transaction blocked — danger signals:")
        for signal in firewall["danger_signals"]:
            print(f"  - {signal}")
    else:
        print(f"Transaction risk: {firewall['risk_score']}/100")
        for change in firewall["simulation"]["asset_changes"]:
            print(f"  {change['direction']}: {change['amount']} {change['symbol']}")
```

### 3. Wallet Security Audit

Full wallet health check:

```python
import requests

BASE = "https://api.shieldbotsecurity.online"
WALLET = "0xYourWallet"

# Scan approvals across multiple chains
for chain_name, chain_id in [("BSC", 56), ("ETH", 1), ("Base", 8453)]:
    resp = requests.get(f"{BASE}/api/rescue/{WALLET}?chainId={chain_id}").json()["data"]

    if resp["risky_approvals"] > 0:
        print(f"\n{chain_name}: {resp['risky_approvals']} risky approvals found!")
        for a in resp["approvals"]:
            if a["risk_level"] in ("HIGH", "CRITICAL"):
                print(f"  REVOKE {a['token']['symbol']} → {a['spender']['address']}")
                print(f"    Allowance: {a['allowance']} | Reason: {a['reason']}")
    else:
        print(f"{chain_name}: Clean — no risky approvals.")
```

### 4. Threat Monitoring Dashboard

Build a threat monitoring loop:

```python
import requests
import time

BASE = "https://api.shieldbotsecurity.online"

while True:
    threats = requests.get(f"{BASE}/api/threats/feed", params={
        "chainId": 56,
        "severity": "critical",
        "limit": 10
    }).json()["data"]["threats"]

    for t in threats:
        print(f"[{t['severity'].upper()}] {t['type']} — {t['address']}")
        print(f"  {t['description']}")

    time.sleep(300)  # Poll every 5 minutes
```

### 5. Phishing URL Batch Check

Check multiple URLs from user reports:

```bash
# Check a list of suspicious URLs
for url in "https://pancakesvvap.finance" "https://app.uniswap.org" "https://bsc-airdrop.com"; do
  result=$(curl -s "https://api.shieldbotsecurity.online/api/phishing?url=$(python3 -c "import urllib.parse; print(urllib.parse.quote('$url', safe=''))")")
  phishing=$(echo "$result" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['is_phishing'])")
  echo "$url → Phishing: $phishing"
done
```

### 6. Multi-Chain Contract Comparison

When a token exists on multiple chains, scan all deployments:

```python
import requests

BASE = "https://api.shieldbotsecurity.online"
TOKEN_ADDRESSES = {
    "BSC": ("0xAddress1", 56),
    "ETH": ("0xAddress2", 1),
    "Base": ("0xAddress3", 8453),
}

results = {}
for chain, (addr, chain_id) in TOKEN_ADDRESSES.items():
    scan = requests.post(f"{BASE}/api/scan", json={"address": addr, "chainId": chain_id}).json()["data"]
    results[chain] = scan
    print(f"{chain}: Risk {scan['risk_score']}/100 | Honeypot: {scan['honeypot']['is_honeypot']}")

# Flag inconsistencies — different risk on different chains = suspicious
scores = [r["risk_score"] for r in results.values()]
if max(scores) - min(scores) > 30:
    print("WARNING: Risk scores vary significantly across chains — investigate further")
```

---

## Integration Patterns

### Building a Security Bot

```javascript
// Discord/Telegram bot pattern — respond to contract addresses
const BASE = "https://api.shieldbotsecurity.online";

async function handleMessage(address, chainId = 56) {
  // 1. Quick scan
  const scanResp = await fetch(`${BASE}/api/scan`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ address, chainId })
  });
  const { data: scan } = await scanResp.json();

  // 2. Build response
  let msg = `**${scan.token_info.symbol}** (${scan.classification})\n`;
  msg += `Risk: ${scan.risk_score}/100\n`;

  if (scan.honeypot.is_honeypot) {
    msg += `🚨 HONEYPOT — Cannot sell!\n`;
  }

  if (scan.flags.length > 0) {
    msg += `Flags: ${scan.flags.join(", ")}\n`;
  }

  msg += `Tax: Buy ${scan.honeypot.buy_tax}% / Sell ${scan.honeypot.sell_tax}%\n`;

  // 3. If risky, check deployer
  if (scan.risk_score > 50 && scan.token_info.owner) {
    const campResp = await fetch(`${BASE}/api/campaign/${scan.token_info.owner}`);
    const { data: campaign } = await campResp.json();
    if (campaign.pattern === "serial_rugpull") {
      msg += `\n⚠️ Deployer is a serial scammer (${campaign.stats.rugpulled_count} rugs)`;
    }
  }

  return msg;
}
```

### Embedding in a DApp

```javascript
// Pre-transaction hook for any EVM DApp
async function checkBeforeSign(txParams) {
  const resp = await fetch("https://api.shieldbotsecurity.online/api/firewall", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      to: txParams.to,
      from: txParams.from,
      value: txParams.value || "0x0",
      data: txParams.data || "0x",
      chainId: txParams.chainId
    })
  });
  const { data } = await resp.json();

  if (data.recommendation === "BLOCK") {
    throw new Error(`Transaction blocked: ${data.danger_signals.join("; ")}`);
  }

  if (data.recommendation === "CAUTION") {
    const proceed = confirm(
      `Warning (Risk ${data.risk_score}/100):\n${data.danger_signals.join("\n")}\n\nProceed?`
    );
    if (!proceed) throw new Error("User cancelled");
  }

  return data; // Safe to proceed
}
```

---

## Error Handling

All error responses follow this format:
```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

### HTTP Status Codes

| Code | Meaning | What to do |
|------|---------|------------|
| 200 | Success | Parse `data` field |
| 400 | Bad request | Check address format, chainId value |
| 404 | Not found | Address doesn't exist on the specified chain |
| 429 | Rate limited | Wait for `Retry-After` header seconds, then retry |
| 500 | Server error | Retry once after 5 seconds; if persistent, inform user |

### Rate Limits

| Tier | Requests/min | Burst |
|------|-------------|-------|
| Public | 30 | 10 |

When rate-limited, implement exponential backoff:
```python
import time
import requests

def safe_request(method, url, **kwargs):
    for attempt in range(3):
        resp = getattr(requests, method)(url, **kwargs)
        if resp.status_code == 429:
            wait = int(resp.headers.get("Retry-After", 2 ** attempt))
            time.sleep(wait)
            continue
        return resp.json()
    raise Exception("Rate limited after 3 retries")
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| `"Invalid address format"` | Address missing `0x` prefix or wrong length | Ensure 42-character hex address starting with `0x` |
| `"Chain not supported"` | Invalid chainId | Use one of: 56, 1, 8453, 42161, 137, 10, 204 |
| `"Address not found"` | Contract doesn't exist on specified chain | Verify chain — user may have the wrong network |
| Empty `flags` array | No issues detected | This is good — contract passed all checks |
| `simulation.success = false` | Transaction would revert | Contract may reject the call — check params |
| `honeypot.sell_tax = -1` | Tax couldn't be determined | Treat as suspicious — warn user |
| 429 status code | Rate limit exceeded | Implement backoff; batch requests if scanning multiple tokens |
| `campaign_risk = "unknown"` | Deployer has no history | New deployer — not necessarily bad, but less track record |

---

## Security Notes for Agents

1. **Never expose user wallet addresses** in responses visible to others
2. **Always URL-encode** the `url` parameter for phishing checks
3. **Default to BSC (chainId 56)** if the user doesn't specify a chain
4. **Cross-reference** scan results with deployer campaigns for higher confidence
5. **Rate limit awareness** — if scanning many tokens, space requests 2+ seconds apart
6. **Disclaimer** — always note that automated scans are not financial advice and users should DYOR
