---
name: mag3nt-pay
version: 1.0.3
description: "Pay for APIs and services, receive payments, and manage agent wallets using Mag3nt USDC virtual cards. Activate when the agent needs to pay for a resource, encounters an HTTP 402 response, a payment challenge header (x402, MPP, AP2, Pay Link), needs to check wallet balance, or set up payment credentials for autonomous transactions."
user-invocable: true
disable-model-invocation: false
allowed-tools: ["Bash(node *)", "Bash(npm *)", "Bash(npx *)", "Bash(curl *)"]
---

# Mag3nt Pay

Pay for APIs and services, receive payments, and manage agent wallets using USDC virtual cards. Supports Base, Solana and most EVMs. Settles on Solana, Base and Tempo.

## Preflight

Confirm credentials are set:

```bash
[[ -z "$MAG3NT_API_KEY" ]] && echo "ERROR: Set MAG3NT_API_KEY (starts with mag3nt_live_)" && exit 1
[[ -z "$MAG3NT_CARD_ID" ]] && echo "ERROR: Set MAG3NT_CARD_ID" && exit 1
[[ -z "$MAG3NT_CARD_TOKEN" ]] && echo "ERROR: Set MAG3NT_CARD_TOKEN" && exit 1
```

If credentials are missing, tell the user to visit [mag3nt.com](https://mag3nt.com) to create an account, generate an API key, and fund an agent card.

## When to Activate

Activate this skill when ANY of these occur:

1. An HTTP response has status code **402**
2. A response contains a `PAYMENT-REQUIRED` header
3. A response contains a `WWW-Authenticate` header with `Payment request=`
4. A response contains an `AP2-Challenge` header
5. A response body contains a `paylink` object with a `code` field
6. The user asks to "pay for", "buy access to", or "unlock" an API
7. The user asks to check their Mag3nt balance or card status

## How Payment Works

Mag3nt has **one universal payment endpoint**: `POST https://mag3nt.com/api/pay`

For x402, MPP, and AP2 protocols, you do NOT need to detect or decode the protocol yourself. Just pass the target URL and the engine auto-detects the protocol and settles on-chain.

For **Pay Links** (paylink object in 402 body), use `POST https://mag3nt.com/api/pay/{code}/settle` with card credentials. No API key required.

## Protocol Detection and Routing

When you receive an HTTP 402, inspect the response in this order:

1. **Body contains `paylink` object** → Pay Link → read `references/paylink-pay.md`
2. **Any protocol header** (PAYMENT-REQUIRED, WWW-Authenticate, AP2-Challenge) → use the universal pay endpoint → read `references/x402-pay.md`

Always check balance first. Read `references/balance.md` before any payment.

## Reference Routing

| Task | Read This First |
|------|----------------|
| Set up credentials | `references/setup.md` |
| Check card balance | `references/balance.md` |
| Pay a Pay Link (most common, no API key needed) | `references/paylink-pay.md` |
| Pay any x402 / MPP / AP2 protected API | `references/x402-pay.md` |
| MPP-specific detection notes | `references/mpp-pay.md` |
| AP2-specific detection notes | `references/ap2-pay.md` |

## Payment Flow

1. **Detect** — Agent receives HTTP 402
2. **Buyer-Agent Preflight** — Before any autonomous spend, extract and verify:
   - **Price / cap unit** — exact amount and currency from the 402 challenge
   - **Policy / version hash** — if the merchant provides terms or a policy hash, log it
   - **Revocation / dispute path** — merchant contact or refund endpoint
   - **Allow / caution / block** — if the amount exceeds the user's stated limit, STOP and ask for confirmation
3. **Check** — Verify card balance >= required amount via `GET /api/cards`
4. **Pay** — Call `POST /api/pay` with `{ card_id, card_token, url }` (or `POST /api/pay/{code}/settle` for Pay Links). The response includes a `transaction_id` and `settlement.tx_hash` as cryptographic charge-evidence.
5. **Receipt** — Log the `transaction_id`, `amount_debited`, `protocol`, and `tx_hash` for audit
6. **Retry** — Re-send the original request with the payment receipt header

## Error Handling

- **Insufficient balance (403)**: Tell the user the current balance and required amount. Suggest topping up at mag3nt.com.
- **Card frozen or inactive**: Tell the user to check their card status in the dashboard.
- **Already settled (410)**: The payment was already made. Proceed directly to retrying the request.
- **No payable option (422)**: The target URL does not offer any payment protocol. Tell the user.
- **Network mismatch**: Ensure the card's network matches what the merchant expects. The network is set when the card is created.

## Important

- All payments are in **USDC** on the network configured for the card
- Card credentials never leave the local environment
- The agent should always confirm the payment amount with the user before paying amounts > $1.00
- Every payment returns a `transaction_id` and on-chain `tx_hash` — always log these as charge-evidence
- Pay Links are the most common protocol — most merchants use them
- API base URL is `https://mag3nt.com` (not api.mag3nt.com)
