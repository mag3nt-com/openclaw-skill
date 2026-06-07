# Universal Pay — Pay Any 402-Protected API

The universal payment endpoint `POST /api/pay` handles all protocols (x402, MPP, AP2) in a single call. The engine automatically probes the target URL, detects the protocol, and settles on-chain.

## When to Use

Use this for **any** external API that returns HTTP 402 with x402, MPP, or AP2 headers. You do not need to detect or decode the protocol yourself — the engine does it.

## Pay via the API

```typescript
const res = await fetch("https://mag3nt.com/api/pay", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.MAG3NT_API_KEY}`,
  },
  body: JSON.stringify({
    card_id: process.env.MAG3NT_CARD_ID,
    card_token: process.env.MAG3NT_CARD_TOKEN,
    url: "https://api.example.com/data/report",
  }),
});

const result = await res.json();

if (result.success) {
  console.log("Transaction:", result.transaction_id);
  console.log("Amount:", result.amount_debited, "USDC");
  console.log("Protocol:", result.protocol);
  console.log("Network:", result.network);
  console.log("Settlement:", result.settlement.status);
}
```

## Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `card_id` | string | yes | Card identifier (`sx_...`) |
| `card_token` | string | yes | Card secret token (`tok_...`) |
| `url` | string | yes | The payment-protected URL to pay |
| `method` | string | no | HTTP method for probing (default: `POST`) |
| `body` | object | no | JSON body to send when probing the URL |
| `headers` | object | no | Additional headers for the probe request |

## Response

```json
{
  "success": true,
  "transaction_id": "txn_abc123",
  "amount_debited": 0.10,
  "merchant_amount": 0.095,
  "platform_fee": 0.005,
  "token": "USDC",
  "network": "eip155:8453",
  "protocol": "x402",
  "settlement": {
    "status": "COMPLETED",
    "tx_hash": "0xabc..."
  }
}
```

## Error Responses

| Status | Meaning |
|--------|---------|
| 400 | Invalid request (missing fields) |
| 403 | Insufficient balance or card inactive |
| 410 | Endpoint removed or resource gone |
| 422 | No payable option offered by merchant |

## Full Example — Detect 402 and Pay

```typescript
// 1. Call the external API
const apiRes = await fetch("https://api.example.com/data/report", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query: "climate risk analysis" }),
});

// 2. If 402, pay via Mag3nt
if (apiRes.status === 402) {
  const payment = await fetch("https://mag3nt.com/api/pay", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${process.env.MAG3NT_API_KEY}`,
    },
    body: JSON.stringify({
      card_id: process.env.MAG3NT_CARD_ID,
      card_token: process.env.MAG3NT_CARD_TOKEN,
      url: "https://api.example.com/data/report",
      method: "POST",
      body: { query: "climate risk analysis" },
    }),
  });

  const result = await payment.json();
  console.log("Paid:", result.amount_debited, "USDC via", result.protocol);
}
```

## Notes

- The engine auto-detects x402, MPP, and AP2 protocols from the target URL's response
- You do not need to parse challenge headers or decode payloads — the engine handles it
- The `protocol` field in the response tells you which protocol was used
- Settlement happens on-chain on the card's configured network
