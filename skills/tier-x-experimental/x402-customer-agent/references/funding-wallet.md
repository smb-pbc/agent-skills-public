# Funding Your Agentic Wallet

**This document provides the HUMAN-FACING messaging for funding.** Use simplified language — no crypto jargon.

---

## Overview

To enable your AI agent to make purchases, you need to add funds to your agentic wallet. This is a one-time setup (plus occasional top-ups).

---

## Human-Facing Funding Message (USE THIS)

When the user needs to add funds, say:

```
💳 **Add Funds to Your Wallet**

Your wallet needs funds before I can place orders for you.

👉 **Add funds here:** https://foodcourts.ai/fund

**Order Summary:**
• [Item] — $XX.XX
• Tax — $X.XX
• **Total: $XX.XX**

I'd suggest adding $XX to cover this order with a small buffer.
Let me know when you've added funds!
```

---

## What NOT to Say

❌ "Send USDC to your wallet on Base from Coinbase"
❌ "You'll need crypto to pay"
❌ "Transfer USDC on the Base network"
❌ "Buy USDC and send it to..."
❌ Any mention of blockchain, crypto, USDC, Coinbase, Base network

✅ "Add funds at https://foodcourts.ai/fund"
✅ "Add funds with your credit card"
✅ "Top up your wallet"

---

## How Much to Suggest?

| Use Case | Recommended Amount |
|----------|-------------------|
| Single order | Order total + $3-5 buffer |
| Testing | $10-20 |
| Light use | $50-100 |
| Regular use | $100-500 |

The agent can only spend what's in the wallet. Start small, add more as needed.

---

## Low Balance Message

When the user tries to order but doesn't have enough funds:

```
⚠️ **Not Enough Funds**

Your wallet has $${balance}, but this order is $${orderTotal}.

👉 Add $${needed} more at: https://foodcourts.ai/fund

Let me know when you've added funds!
```

---

## Verifying Funds Arrived

After the user says they've added funds:

1. Check the wallet balance (technical: query USDC balance on Base)
2. Confirm to user:
   ```
   ✅ Funds received! Your balance is now $XX.XX.
   
   Ready to place your order?
   ```

---

## FAQ Responses (Human-Facing)

**Q: How do I add funds?**
A: Visit https://foodcourts.ai/fund and follow the instructions. You can pay with a credit card.

**Q: Is my money safe?**
A: Your agent can only spend what you add to the wallet. It cannot access your bank account or other funds. You control how much to add.

**Q: Can I get a refund?**
A: Talk to your agent. It can send funds back to you, minus a small network fee (~$0.01).

**Q: How long does it take?**
A: Funds typically appear within a few minutes.

---

## Technical Implementation (Agent Only)

The /fund page at foodcourts.ai handles the actual fiat-to-USDC conversion and deposit. Under the hood:

1. User enters card details on foodcourts.ai/fund
2. Stripe processes the card payment
3. USDC is purchased and sent to the user's wallet on Base
4. Agent can check balance by querying the USDC contract

**USDC Contract (Base):** `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`

```javascript
const USDC_ABI = ['function balanceOf(address owner) view returns (uint256)'];

async function checkBalance(walletAddress) {
  const usdc = new Contract(USDC_ADDRESS, USDC_ABI, provider);
  const balance = await usdc.balanceOf(walletAddress);
  return parseFloat(balance.toString()) / 1e6; // Convert to dollars
}
```

The human never needs to know these technical details. Just point them to the funding page.
