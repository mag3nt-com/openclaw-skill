# AP2 Pay — Mandate-Based Recurring Payments

AP2 (Agent-to-Agent Payment Protocol) is used for recurring or pre-authorized payments. Detection is described below, but **payment uses the same universal endpoint** as all protocols.

## Detection

An HTTP 402 response with an `AP2-Challenge` header indicates AP2:

```typescript
if (response.status === 402) {
  const ap2Header = response.headers.get("AP2-Challenge");
  if (ap2Header) {
    // This is an AP2 challenge — pay via POST /api/pay
  }
}
```

## Pay

AP2 payments use the same universal endpoint as x402 and MPP. The engine auto-detects the protocol:

```typescript
const payment = await fetch("https://mag3nt.com/api/pay", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.MAG3NT_API_KEY}`,
  },
  body: JSON.stringify({
    card_id: process.env.MAG3NT_CARD_ID,
    card_token: process.env.MAG3NT_CARD_TOKEN,
    url: "https://data.example.com/v1/market-feed",
    method: "POST",
    body: { symbols: ["BTC", "ETH"], interval: "1m" },
  }),
});

const result = await payment.json();
// result.protocol === "ap2"
console.log("Paid:", result.amount_debited, "USDC via", result.protocol);
```

See `references/x402-pay.md` for full request/response schema (same endpoint, same shape).

## Notes

- AP2 is ideal for **recurring purchases** and **pre-authorized spending limits**
- The mandate defines the maximum amount the merchant can pull per transaction
- If the mandate is exhausted, the agent will need to create a new one
