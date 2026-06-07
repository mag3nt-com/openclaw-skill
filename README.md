# mag3nt-pay — Agent Payment Skill

[![OpenClaw Skill](https://img.shields.io/badge/OpenClaw-Skill-blue?style=flat-square)](https://clawhub.ai/skills/mag3nt-pay)
[![Hermes Agent](https://img.shields.io/badge/Hermes-Compatible-purple?style=flat-square)](https://github.com/NousResearch/hermes-agent)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](./LICENSE)

**Give your AI agent a wallet.** This skill teaches agents to autonomously pay for any API that returns HTTP 402 — using USDC via Mag3nt virtual cards. Supports Base, Solana and most EVMs. Compatible with OpenClaw, Hermes Agent, and any framework supporting the [agentskills.io](https://agentskills.io) standard.

---

## Install

**OpenClaw:**
```bash
openclaw skill install mag3nt-pay
```

**Hermes Agent:**
```bash
hermes skills tap add mag3nt-com/openclaw-skill
```

**Manual (any agentskills.io framework):**
Copy the `mag3nt-pay/` directory into your agent's skills folder.

## Setup (60 seconds)

1. Get a free API key at [mag3nt.com](https://mag3nt.com)
2. Create an agent card in the dashboard
3. Fund it with USDC (Base, Solana, or any supported EVM)
4. Set your env vars:

```bash
export MAG3NT_API_KEY=mag3nt_live_your_key
export MAG3NT_CARD_ID=mag3nt_your_card_id
export MAG3NT_CARD_TOKEN=tok_your_card_token
```

5. Test it:

```bash
openclaw agent --message "Check my Mag3nt card balance"
```

## What It Does

When your agent calls any API and gets back an **HTTP 402 Payment Required**, this skill automatically:

1. **Detects** the payment protocol from response headers
2. **Decodes** the challenge (amount, recipient, network)
3. **Checks** the agent's card balance
4. **Pays** using the correct protocol (x402, MPP, AP2, or Pay Link)
5. **Retries** the original request with the payment receipt

The agent handles all of this without human intervention.

## Supported Protocols

| Protocol | Header / Signal | Use Case |
|----------|----------------|----------|
| **Pay Link** | `paylink` object in 402 body | Universal — any merchant |
| **x402** | `PAYMENT-REQUIRED` header | Single pay-per-request |
| **MPP** | `WWW-Authenticate: Payment` | Streaming micropayments |
| **AP2** | `AP2-Challenge` header | Recurring / mandated payments |

## How It Works

```
Your Agent                    Paid API                   Mag3nt
    │                            │                         │
    ├── GET /api/data ──────────►│                         │
    │◄── 402 + challenge ────────┤                         │
    │                            │                         │
    │  [mag3nt-pay skill activates]                        │
    │                            │                         │
    ├── Check balance ──────────────────────────────────────►
    │◄── Balance: $50 USDC ─────────────────────────────────┤
    │                            │                         │
    ├── Pay $0.10 ──────────────────────────────────────────►
    │◄── Receipt: tx_abc123 ────────────────────────────────┤
    │                            │                         │
    ├── GET /api/data ──────────►│                         │
    │   + X-Payment: tx_abc123   │                         │
    │◄── 200 OK + data ──────────┤                         │
    │                            │                         │
```

## Example Conversations

**Pay Link (most common):**
```
You:   "Run a background check on this company via screening.example.com"

Agent: Calling the screening API...
       Received HTTP 402 — Pay Link detected in response body.
       Amount: 0.10 USDC | Merchant: screening.example.com
       Card balance: 49.90 USDC — sufficient.
       Settling Pay Link PL_abc123...
       Payment settled. Retrying request...
       Done. Here are the screening results: [...]
```

**x402 (single pay-per-request):**
```
You:   "Fetch the climate risk report from data.example.com"

Agent: Calling the data API...
       Received HTTP 402 — x402 protocol detected (PAYMENT-REQUIRED header).
       Amount: 0.25 USDC | Recipient: 0x1BD3...9D00
       Card balance: 49.65 USDC — sufficient.
       Payment settled. Transaction: tx_8f2a...
       Retrying with receipt...
       Done. Here's the climate risk report: [...]
```

**MPP (streaming micropayments):**
```
You:   "Stream inference results from compute.example.com"

Agent: Calling the compute API...
       Received HTTP 402 — MPP protocol detected (WWW-Authenticate header).
       Amount: 0.05 USDC per request | Session-based billing.
       Card balance: 49.40 USDC — sufficient.
       Payment session opened. Session: sess_7d1e...
       Streaming results: [...]
```

**AP2 (recurring mandate):**
```
You:   "Subscribe to the real-time market feed at data.example.com"

Agent: Calling the market feed API...
       Received HTTP 402 — AP2 protocol detected (AP2-Challenge header).
       Amount: 1.00 USDC | Mandate: recurring, max 5.00 USDC/day.
       Card balance: 48.40 USDC — sufficient.
       Mandate executed. Transaction: tx_3c9f...
       Market feed connected: [...]
```

## Directory Structure

```
mag3nt-pay/
├── SKILL.md              # Skill definition (frontmatter + instructions)
├── references/
│   ├── setup.md          # SDK configuration and credentials
│   ├── balance.md        # Balance check before paying
│   ├── x402-pay.md       # x402 single-request payments
│   ├── mpp-pay.md        # MPP streaming micropayments
│   ├── ap2-pay.md        # AP2 mandate-based payments
│   └── paylink-pay.md    # Pay Link universal payments
└── scripts/
    └── quickstart.mjs    # Run a balance check in one command
```

## Quick Test

After setting your env vars, run the quickstart script directly:

```bash
node mag3nt-pay/scripts/quickstart.mjs
```

This checks your card balance and confirms the SDK is working.

## Requirements

- Node.js 18+ or Python 3.8+
- `@mag3nt/sdk` (npm) or `mag3nt` (PyPI)
- A Mag3nt account with a funded agent card

## Links

- **Dashboard:** [mag3nt.com](https://mag3nt.com)
- **API Docs:** [docs.mag3nt.com](https://docs.mag3nt.com)
- **SDK (npm):** [@mag3nt/sdk on npm](https://www.npmjs.com/package/@mag3nt/sdk)
- **SDK (Python):** [mag3nt on PyPI](https://pypi.org/project/mag3nt/)
- **Protocol Specs:** [mag3nt-agent-skills](https://github.com/mag3nt-com/agent-skills)

## License

MIT — see [LICENSE](./LICENSE)
