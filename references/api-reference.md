# ShieldBot API Reference

Complete endpoint documentation for the ShieldBot Security API.

**Base URL:** `https://api.shieldbotsecurity.online`

## Authentication

Public endpoints require no authentication. Rate limits apply per IP.

## Rate Limits

| Tier | Requests/min | Burst |
|------|-------------|-------|
| Public | 30 | 10 |

Rate-limited responses return HTTP `429` with a `Retry-After` header.

---

## Endpoints

### POST /api/scan

Perform a security scan on a smart contract.

**Request Headers:**
- `Content-Type: application/json`

**Request Body:**
```json
{
  "address": "string (required) — Contract address (0x...)",
  "chainId": "integer (required) — Chain ID (see Supported Chains)"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "address": "0x...",
    "chainId": 56,
    "risk_score": 75,
    "classification": "HIGH_RISK",
    "flags": ["hidden_mint", "ownership_not_renounced"],
    "honeypot": {
      "is_honeypot": false,
      "buy_tax": 5.0,
      "sell_tax": 12.0,
      "transfer_tax": 0.0
    },
    "token_info": {
      "name": "Example Token",
      "symbol": "EXT",
      "total_supply": "1000000000000000000000000",
      "owner": "0x..."
    },
    "contract_info": {
      "verified": true,
      "is_proxy": false,
      "has_source": true
    },
    "scanned_at": "2026-03-17T12:00:00Z"
  }
}
```

**Error Response (400):**
```json
{
  "success": false,
  "error": "Invalid address format"
}
```

---

### POST /api/firewall

Analyze a transaction for security risks with optional simulation.

**Request Body:**
```json
{
  "to": "string (required) — Destination contract address",
  "from": "string (required) — Sender wallet address",
  "value": "string (optional) — Transaction value in hex (default: \"0x0\")",
  "data": "string (optional) — Transaction calldata in hex",
  "chainId": "integer (required) — Chain ID"
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "classification": "HIGH_RISK",
    "risk_score": 82,
    "danger_signals": [
      "Interacting with unverified contract",
      "Unlimited token approval requested"
    ],
    "simulation": {
      "success": true,
      "gas_estimate": 85000,
      "asset_changes": [
        {
          "token": "0x...",
          "symbol": "USDT",
          "amount": "-1000.00",
          "direction": "out"
        }
      ]
    },
    "recommendation": "BLOCK",
    "contract_info": {
      "verified": false,
      "risk_score": 90
    }
  }
}
```

---

### GET /api/campaign/{address}

Analyze a deployer's contract deployment history.

**Path Parameters:**
- `address` (required) — Deployer wallet address

**Query Parameters:**
- `chainId` (optional) — Filter by chain (default: all chains)

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "deployer": "0x...",
    "total_contracts": 15,
    "contracts_deployed": [
      {
        "address": "0x...",
        "name": "ScamToken",
        "chain_id": 56,
        "risk_score": 95,
        "deployed_at": "2026-03-10T08:00:00Z",
        "status": "rugpulled"
      }
    ],
    "campaign_risk": "CRITICAL",
    "pattern": "serial_rugpull",
    "stats": {
      "avg_risk_score": 88,
      "rugpulled_count": 12,
      "active_count": 3
    }
  }
}
```

---

### GET /api/threats/feed

Retrieve real-time threat intelligence.

**Query Parameters:**
- `chainId` (optional) — Filter by chain ID
- `limit` (optional, default: 20, max: 100) — Number of results
- `severity` (optional) — Filter: `low`, `medium`, `high`, `critical`
- `offset` (optional, default: 0) — Pagination offset

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "threats": [
      {
        "id": "threat_abc123",
        "type": "honeypot",
        "address": "0x...",
        "chain_id": 56,
        "severity": "critical",
        "description": "Honeypot detected: sell tax 100%",
        "timestamp": "2026-03-17T11:30:00Z",
        "metadata": {
          "sell_tax": 100,
          "victims": 45
        }
      }
    ],
    "total": 150,
    "limit": 20,
    "offset": 0
  }
}
```

---

### GET /api/phishing

Check a URL against phishing databases.

**Query Parameters:**
- `url` (required) — URL to check (URL-encoded)

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "url": "https://suspicious-site.com",
    "is_phishing": true,
    "confidence": 0.95,
    "reasons": [
      "Domain registered 2 days ago",
      "Impersonates PancakeSwap",
      "Known phishing kit detected"
    ],
    "domain_info": {
      "registration_date": "2026-03-15",
      "registrar": "NameCheap",
      "similar_to": "pancakeswap.finance"
    }
  }
}
```

---

### GET /api/rescue/{wallet}

Scan wallet token approvals and flag risky allowances.

**Path Parameters:**
- `wallet` (required) — Wallet address to scan

**Query Parameters:**
- `chainId` (optional, default: 56) — Chain to scan

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "wallet": "0x...",
    "chain_id": 56,
    "total_approvals": 23,
    "risky_approvals": 5,
    "approvals": [
      {
        "token": {
          "address": "0x...",
          "symbol": "USDT",
          "name": "Tether USD"
        },
        "spender": {
          "address": "0x...",
          "label": "Unknown Contract",
          "verified": false
        },
        "allowance": "unlimited",
        "risk_level": "HIGH",
        "reason": "Spender is unverified contract with no transaction history"
      }
    ],
    "recommendation": "Revoke 5 risky approvals immediately"
  }
}
```

---

### POST /api/agent/chat

Conversational AI security advisor.

**Request Body:**
```json
{
  "message": "string (required) — User's question",
  "context": {
    "chainId": "integer (optional) — Relevant chain",
    "address": "string (optional) — Relevant contract address"
  }
}
```

**Success Response (200):**
```json
{
  "success": true,
  "data": {
    "response": "PancakeSwap is a well-established DEX on BNB Chain...",
    "references": [
      {
        "type": "contract",
        "address": "0x...",
        "label": "PancakeSwap Router v2"
      }
    ]
  }
}
```

---

## Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad request — invalid parameters |
| 404 | Not found — address not found on specified chain |
| 429 | Rate limited — retry after `Retry-After` seconds |
| 500 | Internal server error |

All error responses follow this format:
```json
{
  "success": false,
  "error": "Human-readable error message"
}
```

## Supported Chains

| Chain | chainId | Explorer |
|-------|---------|----------|
| BNB Smart Chain | 56 | bscscan.com |
| Ethereum | 1 | etherscan.io |
| Base | 8453 | basescan.org |
| Arbitrum | 42161 | arbiscan.io |
| Polygon | 137 | polygonscan.com |
| Optimism | 10 | optimistic.etherscan.io |
| opBNB | 204 | opbnbscan.com |
