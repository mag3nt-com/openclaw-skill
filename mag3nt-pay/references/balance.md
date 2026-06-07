# Balance — Check Card Balance

Always check balance before attempting a payment to avoid failed transactions.

## Check Balance

```typescript
const res = await fetch("https://mag3nt.com/api/cards", {
  headers: { Authorization: `Bearer ${process.env.MAG3NT_API_KEY}` },
});
const { cards } = await res.json();
const card = cards.find((c) => c.id === process.env.MAG3NT_CARD_ID);

console.log("Balance:", card.balance, "USDC");
console.log("Status:", card.status);
console.log("Network:", card.funding_network);
```

## Pre-Flight Check

Before paying, compare the required amount against the card balance:

```typescript
const requiredAmount = 0.10; // from the 402 challenge payload

if (card.balance < requiredAmount) {
  throw new Error(
    `INSUFFICIENT_FUNDS: Card balance is ${card.balance} USDC but ${requiredAmount} USDC is required. Please top up the agent card.`
  );
}
```

## Treasury Balance

To check total available funds across all cards:

```typescript
const res = await fetch("https://mag3nt.com/api/balance", {
  headers: { Authorization: `Bearer ${process.env.MAG3NT_API_KEY}` },
});
const balance = await res.json();

console.log("Available:", balance.available, "USDC");
console.log("Total funded:", balance.total_funded);
console.log("Total spent:", balance.total_spent);
```

## Important Notes

- Balance is denominated in **USDC**
- If the card status is not `ACTIVE`, payments will fail
- If the card is `FROZEN`, contact the card owner to unfreeze it
- The card's `funding_network` determines which chain payments settle on
