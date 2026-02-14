---
name: x402-customer-agent
description: Enable AI agents to make autonomous payments using x402 protocol. Covers wallet creation, user funding instructions, and payment execution. Use when you need to set up crypto payments, handle HTTP 402 responses, or help users fund your agent wallet with USDC on Base.
triggers:
  - x402
  - agent wallet
  - crypto payments
  - autonomous payments
  - USDC payments
  - make a purchase
  - buy something
  - payment required
  - 402 error
  - fund my wallet
---

# x402 Agent Payment Skill

**Purpose:** Enable any AI agent to make autonomous crypto payments.

This skill teaches you (the AI agent) how to:
1. Create your own crypto wallet
2. Help your user fund it with USDC
3. Make payments to x402-enabled merchants

---

## What is x402?

x402 is an open payment protocol built on HTTP status code 402 (Payment Required). It enables machine-to-machine payments without credit cards or payment processors.

**How it works:**
```
1. You request a paid resource (API, file, service)
2. Server returns HTTP 402 with payment details
3. You send USDC on Base blockchain (~$0.001 fee, 2-second finality)
4. You retry the request with payment proof in headers
5. Server delivers the resource
```

**Why Base?** Coinbase's Layer 2 blockchain. Fast, cheap, and USDC-native. Not Ethereum mainnet (which is slow and expensive).

**Current stats (Feb 2026):** 75M+ transactions, $24M+ volume, zero protocol fees.

---

## Quick Setup (3 Steps)

### Step 1: Create Your Wallet

You need a Base-compatible wallet. Choose one approach:

**Option A: Coinbase AgentKit (Recommended)**
```javascript
import { CdpAgentkit } from "@coinbase/cdp-agentkit-core";

const agentkit = await CdpAgentkit.configureWithWallet();
const wallet = agentkit.wallet;
const address = wallet.defaultAddress.id;

console.log(`My wallet address: ${address}`);
// Save wallet data securely for persistence
const exportedWallet = await agentkit.exportWallet();
```

**Option B: Coinbase Agentic Wallets API**
```javascript
// For pre-built infrastructure with spending limits
const response = await fetch('https://api.cdp.coinbase.com/platform/v2/wallets', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${CDP_API_KEY}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    network_id: 'base-mainnet',
    type: 'autonomous'
  })
});
const wallet = await response.json();
```

**Option C: Standard Ethereum Wallet (ethers.js/viem)**
```javascript
import { Wallet } from 'ethers';
const wallet = Wallet.createRandom();
console.log(`Address: ${wallet.address}`);
// CRITICAL: Store wallet.privateKey in secure secrets manager
```

→ See `references/wallet-creation.md` for complete guide.

### Step 2: Tell Your User How to Fund It

Once you have a wallet, share the **funding page** with your user:

```
I've created my own wallet for making purchases. Fund it here:

https://storage.googleapis.com/foodcourts-funding/fund.html?address=[YOUR_WALLET_ADDRESS]

This page lets you:
- Scan a QR code with any crypto wallet
- See the live USDC balance
- Copy the address for manual transfer
```

**Alternative: Direct Instructions**
```
My USDC Address on Base: 0x[YOUR_WALLET_ADDRESS]

How to send USDC:
1. Open Coinbase app
2. Tap "Send" on your USDC balance
3. Paste my address above
4. IMPORTANT: Select "Base" as the network (not Ethereum)
5. Send $20-50 to start
```

**Coming Soon: Credit Card Funding (Stripe Onramp)**
- Application submitted Feb 2026, pending approval
- Once approved, users can buy USDC directly with credit/debit cards
- No crypto exchange needed

**Programmatic Funding API:**
```javascript
// Create a funding session (for Stripe onramp - when approved)
const response = await fetch('https://us-east4-data-lake-430420.cloudfunctions.net/create-funding-session', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${API_KEY}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    walletAddress: '0x...',
    amount: 15.00,
    agentId: 'my-agent',
    userId: 'user_123'
  })
});
const { fundingUrl, sessionId } = await response.json();

// Check funding status
const status = await fetch(`https://us-east4-data-lake-430420.cloudfunctions.net/funding-status/${sessionId}`);
```

→ See `references/fiat-to-crypto.md` for detailed user guide you can share.

### Step 3: Make Payments

When you hit a 402 response, handle it:

```javascript
async function x402Fetch(url, options = {}) {
  let response = await fetch(url, options);
  
  if (response.status !== 402) return response;
  
  // Parse payment requirements
  const paymentDetails = await response.json();
  const { payTo, amount, asset, chainId } = paymentDetails;
  
  // Verify it's USDC on Base
  if (chainId !== 8453 || asset !== 'USDC') {
    throw new Error('Unsupported payment type');
  }
  
  // Send payment
  const txHash = await sendUSDCPayment(payTo, amount);
  
  // Wait for confirmation (~2 seconds on Base)
  await waitForConfirmation(txHash);
  
  // Retry with proof
  return fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      'X-Payment': txHash,
      'X-Payment-Chain': '8453'
    }
  });
}
```

→ See `references/making-payments.md` for complete implementation.

---

## User Communication Templates

### When User Asks "How do I set you up to buy things?"

```
I can make purchases autonomously using crypto payments. Here's the setup:

**Step 1: I've Created a Wallet**
My USDC address on Base: [WALLET_ADDRESS]

**Step 2: Fund It**
Send USDC to the address above. Here's how:

Option A — Coinbase App:
1. Buy USDC (or convert existing crypto)
2. Tap "Send" on your USDC
3. Paste my address
4. Select "Base" network (critical!)
5. Send amount

Option B — Coinbase Onramp:
Visit https://pay.coinbase.com and send directly to my address

**Step 3: Set Limits**
Let me know your preferences:
- Max single purchase: $___
- Daily spending limit: $___

I recommend starting with $20-50 for testing.
```

### When User Asks "What's the status of my wallet?"

```
**Wallet Status**
Address: [WALLET_ADDRESS]
Network: Base (Chain ID 8453)
Balance: [X] USDC

View on BaseScan: https://basescan.org/address/[WALLET_ADDRESS]

Recent transactions:
- [date]: Sent $X.XX to [merchant] — [description]
- [date]: Received $XX.XX from [user address]
```

### When You Need to Make a Purchase

Before executing, confirm with user if amount exceeds their set threshold:

```
**Purchase Request**
Merchant: [merchant name]
Item: [description]
Price: $X.XX USDC
My current balance: $XX.XX USDC

Shall I proceed? (This will deduct from your funded balance)
```

---

## Security Best Practices

### Key Storage
- **NEVER** store private keys in code, logs, or conversation history
- Use your platform's secrets manager (GCP Secret Manager, AWS Secrets, etc.)
- If no secrets manager: encrypt key at rest with user-provided passphrase

### Spending Controls
```javascript
const LIMITS = {
  maxPerTransaction: 50.00,    // Require approval above this
  dailyMax: 200.00,            // Hard stop for the day
  autoApproveUnder: 10.00      // Silent purchases below this
};

async function checkLimits(amount) {
  if (amount > LIMITS.maxPerTransaction) {
    return { approved: false, reason: 'Exceeds per-transaction limit' };
  }
  
  const todaySpent = await getTodaySpending();
  if (todaySpent + amount > LIMITS.dailyMax) {
    return { approved: false, reason: 'Would exceed daily limit' };
  }
  
  return { 
    approved: true, 
    requiresConfirmation: amount >= LIMITS.autoApproveUnder 
  };
}
```

### Transaction Logging
Always log transactions for user review:
```javascript
{
  timestamp: "2026-02-14T12:30:00Z",
  type: "payment",
  merchant: "pbc-x402-api",
  description: "Rolled Beef Sandwich",
  amount: "18.50",
  asset: "USDC",
  txHash: "0x...",
  status: "confirmed"
}
```

### Merchant Validation
Before paying unknown merchants:
1. Check if address appears in x402 discovery registries
2. Verify HTTPS and valid SSL on merchant endpoint
3. For large amounts, confirm with user first

---

## Finding x402 Merchants

**Discovery APIs:**
```javascript
// Coinbase x402 Discovery
const discovery = await fetch(
  'https://api.cdp.coinbase.com/platform/v2/x402/discovery/resources'
);
const merchants = await discovery.json();

// Filter by category
const foodMerchants = merchants.filter(m => m.category === 'food');
```

**Known Merchants (Feb 2026):**
- **PBC x402 API** — Sandwich ordering (example in this skill)
- **Coinbase Commerce** — Retail integrations
- Growing ecosystem at x402.org/ecosystem

---

## Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| "Insufficient funds" | Wallet balance too low | Tell user to add more USDC |
| "Invalid payment proof" | Tx not confirmed yet | Wait 2-3 seconds, retry |
| "Network mismatch" | USDC on wrong chain | Must be Base (8453), not Ethereum |
| "Payment expired" | Took too long | Get fresh 402, pay again |
| "Amount mismatch" | Paid wrong amount | Check exact amount from 402 response |

---

## Example: Order a Sandwich

See `examples/order-sandwich.md` for a complete walkthrough using PBC's x402 API.

---

## References

- `references/wallet-creation.md` — Detailed wallet setup guide
- `references/fiat-to-crypto.md` — User guide for buying and sending USDC
- `references/making-payments.md` — Complete payment implementation
- `examples/order-sandwich.md` — Working example with PBC's x402 API

## External Resources

- [x402.org](https://x402.org) — Official protocol documentation
- [AgentKit Docs](https://docs.cdp.coinbase.com/agentkit) — Coinbase AgentKit
- [Base Docs](https://docs.base.org) — Base blockchain documentation
- [BaseScan](https://basescan.org) — Block explorer for Base
