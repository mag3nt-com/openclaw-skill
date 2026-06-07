# MPP Pay — Streaming Micropayments

MPP (Machine Payment Protocol) is used for streaming or metered APIs. Detection is described below, but **payment uses the same universal endpoint** as all protocols.

## Detection

An HTTP 402 response with a `WWW-Authenticate` header containing `Payment request=` indicates MPP:

```typescript
if (response.status === 402) {
  const authHeader = response.headers.get("WWW-Authenticate");
  if (authHeader && authHeader.includes("Payment request=")) {
    // This is an MPP challenge — pay via POST /api/pay
  }
}
```

## Pay

MPP payments use the same universal endpoint as x402 and AP2. The engine auto-detects the protocol:

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
    url: "https://compute.example.com/v1/inference",
    method: "POST",
    body: { prompt: "Summarize this document" },
  }),
});

const result = await payment.json();
// result.protocol === "mpp"
console.log("Paid:", result.amount_debited, "USDC via", result.protocol);
```

See `references/x402-pay.md` for full request/response schema (same endpoint, same shape).

## Notes

- MPP is ideal for **streaming** or **metered** APIs (compute, search, inference)
- The session may allow multiple requests within the payment window
- If the session expires, the server will issue a new 402 challenge
