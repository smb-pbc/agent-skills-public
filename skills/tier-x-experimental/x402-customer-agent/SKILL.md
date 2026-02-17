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
  - order food
  - foodcourts
---

# x402 Agent Payment Skill

**Purpose:** Enable any AI agent to make autonomous crypto payments.
**Last Updated:** 2026-02-17

This skill teaches you (the AI agent) how to:
1. Generate a crypto wallet for your user
2. Hand over the private key with proper security warnings
3. Guide them through FoodCourts profile creation
4. Help them fund the wallet with credit card
5. Check balance and make payments

---

## ⚠️ CRITICAL: Wallet Ownership & Private Key Security

**The wallet you generate belongs to YOUR USER, not you.**

- You create it and manage it on their behalf
- The private key MUST be given to the user immediately
- **FoodCourts NEVER stores private keys** — only wallet addresses
- If the user loses their private key, their funds are **PERMANENTLY LOST**
- This is not recoverable — make this crystal clear to the user

---

## What is x402?

x402 is an open payment protocol built on HTTP status code 402 (Payment Required). It enables machine-to-machine payments without traditional payment processors.

**How it works:**
```
1. You request a paid resource (API, file, service)
2. Server returns HTTP 402 with payment details
3. You send USDC on Base blockchain (~$0.001 fee, 2-second finality)
4. You retry the request with payment proof in headers
5. Server delivers the resource
```

**Why Base?** Coinbase's Layer 2 blockchain. Fast, cheap, and USDC-native. Not Ethereum mainnet (which is slow and expensive).

---

## Complete Setup Flow (5 Steps)

### Step 1: Generate the Wallet

Generate a new wallet keypair. You'll give the private key to the user and use it to sign transactions.

**Using ethers.js (Recommended)**
```javascript
import { Wallet } from 'ethers';

// Generate new wallet
const wallet = Wallet.createRandom();
const address = wallet.address;
const privateKey = wallet.privateKey;

// Store for agent use (in secrets manager)
// AND immediately show to user (Step 2)
```

**Using viem**
```javascript
import { generatePrivateKey, privateKeyToAccount } from 'viem/accounts';

const privateKey = generatePrivateKey();
const account = privateKeyToAccount(privateKey);
const address = account.address;
```

**Using Coinbase AgentKit**
```javascript
import { CdpAgentkit } from "@coinbase/cdp-agentkit-core";

const agentkit = await CdpAgentkit.configureWithWallet();
const wallet = agentkit.wallet;
const address = wallet.defaultAddress.id;
const exportedWallet = await agentkit.exportWallet();
```

---

### Step 2: Private Key Handoff (⚠️ CRITICAL)

**Immediately after generating the wallet, show the private key to your user with STRONG warnings:**

```
🔐 YOUR AGENT WALLET HAS BEEN CREATED

I've generated a wallet that I'll use to make purchases on your behalf.

📍 **Wallet Address:**
`0x[WALLET_ADDRESS]`

🔑 **Your Private Key:**
`0x[PRIVATE_KEY]`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ SAVE THIS PRIVATE KEY NOW ⚠️

• This is the ONLY time you will see this key
• FoodCourts CANNOT recover it — we don't store private keys
• If you lose it, any money in this wallet is GONE FOREVER
• Save it in a password manager, or write it down and store it safely
• Never share it with anyone except your AI agent
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Please confirm you have saved this private key before we continue.
Type "I've saved it" to proceed.
```

**IMPORTANT:** Wait for explicit user confirmation before proceeding. Do not continue until they confirm.

**Agent-side storage:**
```javascript
// Store in your secrets manager for transaction signing
// Example: GCP Secret Manager
const { SecretManagerServiceClient } = require('@google-cloud/secret-manager');

async function storePrivateKey(userId, privateKey) {
  const client = new SecretManagerServiceClient();
  const parent = `projects/${PROJECT_ID}/secrets/user-wallet-${userId}`;
  await client.addSecretVersion({
    parent,
    payload: { data: Buffer.from(privateKey) }
  });
}
```

---

### Step 3: Generate Profile Link & Send User to FoodCourts

**After user confirms they've saved the key, generate and send the signup link:**

```javascript
// Generate the profile link
const signupUrl = `https://foodcourts.ai/signup?wallet=${walletAddress}`;
```

**Present to user:**

```
Great! Your private key is saved.

Now let's link your wallet to a FoodCourts account so you can add funds.

🔗 **Click here to create your profile:**
https://foodcourts.ai/signup?wallet=[WALLET_ADDRESS]

**What you'll do:**
1. Click the link above
2. Sign in with Google (or create an account with email)
3. Your wallet address will be automatically linked to your profile
4. You'll see the funding page to add money

Come back here once you've created your account!
```

**API endpoint (for programmatic linking if needed):**
```javascript
// POST /api/profile — Link wallet to profile
const response = await fetch('https://foodcourts.ai/api/profile', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    wallet: walletAddress,
    // User auth handled via session/OAuth on web app
  })
});
```

---

### Step 4: Guide User to Add Funds via Stripe

**Once user returns and confirms profile creation:**

```
Your profile is ready! Now let's add funds so I can make purchases for you.

💳 **Add funds here:** https://foodcourts.ai/fund

**How it works:**
1. Log into your FoodCourts profile
2. Click "Add Funds"
3. Enter the amount you want to add (I suggest $25-50 to start)
4. Pay with your credit card via Stripe
5. USDC will appear in your wallet within a minute or two

The funds go directly to your wallet on Base blockchain — I can use them to 
place orders at any x402-enabled merchant.

Let me know once you've added funds by saying "funds added" and I'll check your balance!
```

---

### Step 5: Check Balance & Confirm

**When user says they've added funds, check the balance:**

**Using the FoodCourts API (Recommended):**
```javascript
// GET /api/balance?wallet=0x... — Check USDC balance
async function checkBalance(walletAddress) {
  const response = await fetch(
    `https://foodcourts.ai/api/balance?wallet=${walletAddress}`
  );
  const data = await response.json();
  return data.balance; // Returns balance in USDC (e.g., "25.00")
}

// Usage
const balance = await checkBalance(walletAddress);
console.log(`Balance: $${balance} USDC`);
```

**Direct on-chain check (alternative):**
```javascript
import { createPublicClient, http } from 'viem';
import { base } from 'viem/chains';

const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const USDC_ABI = ['function balanceOf(address) view returns (uint256)'];

async function getBalanceOnChain(walletAddress) {
  const client = createPublicClient({
    chain: base,
    transport: http('https://mainnet.base.org')
  });
  
  const balance = await client.readContract({
    address: USDC_ADDRESS,
    abi: USDC_ABI,
    functionName: 'balanceOf',
    args: [walletAddress]
  });
  
  return Number(balance) / 1e6; // USDC has 6 decimals
}
```

**Confirm to user:**

```
✅ **Wallet Funded Successfully!**

💰 **Current Balance:** $XX.XX USDC
📍 **Wallet:** 0x[ADDRESS]

You're all set! I can now make purchases on your behalf.

Would you like to order something? Just tell me what you want and I'll 
handle the payment automatically.
```

---

## After Setup: Making Purchases

### Pre-Purchase Balance Check

**Always check balance before attempting any purchase:**

```javascript
async function canAfford(walletAddress, amount) {
  const balance = await checkBalance(walletAddress);
  return parseFloat(balance) >= parseFloat(amount);
}

// Before ordering
const orderTotal = 18.50;
if (!await canAfford(wallet.address, orderTotal)) {
  // Prompt user to add funds
  throw new Error(`Insufficient funds. Need $${orderTotal}, have $${balance}`);
}
```

### Handle 402 Payments

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
  
  // Check balance first
  const balance = await checkBalance(wallet.address);
  if (parseFloat(balance) < parseFloat(amount)) {
    throw new Error(`Insufficient funds. Need $${amount}, have $${balance}`);
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

---

## User Communication Templates

### Initial Setup Introduction

```
I can make purchases for you using crypto payments. Here's what we'll do:

1. **I generate a wallet** — This is YOUR wallet that I manage for you
2. **You save the private key** — Critical! This is the only time you'll see it
3. **You create a FoodCourts profile** — Links wallet to your account
4. **You add funds** — Easy credit card payment via Stripe
5. **I place orders** — Using the funds in your wallet

Ready? This takes about 2 minutes. Let's start!
```

### Wallet Status Check

```
📊 **Your Agent Wallet Status**

📍 Address: `0x[ADDRESS]`
💰 Balance: $XX.XX USDC
🔗 Network: Base

🔍 View transactions: https://basescan.org/address/[ADDRESS]
👤 Manage profile: https://foodcourts.ai

Need more funds? Add them at https://foodcourts.ai/fund
```

### Low Balance Warning

```
⚠️ **Low Balance Warning**

Your wallet has $X.XX, but this order costs $XX.XX.

You need to add at least $XX.XX more to complete this purchase.

💳 **Add funds here:** https://foodcourts.ai/fund

Let me know once you've added funds and I'll complete the order!
```

### Pre-Purchase Confirmation

```
**Ready to Order**

🍽️ Item: [description]
💰 Price: $X.XX
📍 Merchant: [merchant name]

Your balance: $XX.XX USDC
After purchase: $XX.XX USDC

Proceed with this order? (Reply "yes" to confirm)
```

### Post-Purchase Confirmation

```
✅ **Order Placed!**

🧾 Order #: [ORDER_ID]
💰 Total: $X.XX USDC
💳 Remaining balance: $XX.XX

🔗 Transaction: https://basescan.org/tx/[TX_HASH]

Your order has been sent to the restaurant!
```

---

## FoodCourts API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/signup?wallet=0x...` | GET | User signup page with wallet pre-filled |
| `/api/profile` | POST | Link wallet to user profile |
| `/api/balance?wallet=0x...` | GET | Check USDC balance for wallet |
| `/fund` | GET | Add funds page (Stripe checkout) |
| `/login` | GET | User login |

### Balance Check API Example

```bash
# Check balance for a wallet
curl "https://foodcourts.ai/api/balance?wallet=0xAfCdA9C19F2af72eb5d636461d715De5b2298706"

# Response:
{
  "wallet": "0xAfCdA9C19F2af72eb5d636461d715De5b2298706",
  "balance": "47.50",
  "currency": "USDC",
  "network": "base"
}
```

---

## Error Handling

| Error | Cause | User Response |
|-------|-------|---------------|
| "Insufficient funds" | Balance too low | "Your wallet has $X. Add $Y more at https://foodcourts.ai/fund" |
| "Profile not found" | No FoodCourts account | "Create your profile first: https://foodcourts.ai/signup?wallet=[ADDR]" |
| "Wallet not linked" | Profile exists, wallet not attached | "Your wallet isn't linked. Visit https://foodcourts.ai to add it." |
| "Payment expired" | >10 min to pay | Re-request the 402 to get fresh payment details |
| "Invalid wallet" | Malformed address | "That doesn't look like a valid wallet address. It should start with 0x." |

---

## Security Best Practices

### Spending Controls
```javascript
const LIMITS = {
  maxPerTransaction: 50.00,    // Require confirmation above this
  dailyMax: 200.00,            // Hard stop for the day
  autoApproveUnder: 10.00      // Silent purchases below this
};

async function checkLimits(amount) {
  if (amount > LIMITS.maxPerTransaction) {
    return { approved: false, requiresConfirmation: true };
  }
  
  const todaySpent = await getTodaySpending();
  if (todaySpent + amount > LIMITS.dailyMax) {
    return { approved: false, reason: 'Would exceed daily limit' };
  }
  
  return { approved: true };
}
```

### Transaction Logging
Keep records for user review:
```javascript
{
  timestamp: "2026-02-17T12:30:00Z",
  type: "payment",
  merchant: "FoodCourts",
  description: "Rolled Beef Sandwich",
  amount: "18.50",
  txHash: "0x...",
  status: "confirmed"
}
```

---

## Complete Setup Conversation Example

See `examples/complete-setup-flow.md` for a full conversation example walking a user through setup and their first purchase.

---

## Quick Reference: The 5-Step Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. GENERATE WALLET                                          │
│    └── ethers.Wallet.createRandom() or viem                │
├─────────────────────────────────────────────────────────────┤
│ 2. HAND OVER PRIVATE KEY                                    │
│    └── Show key + strong warnings                          │
│    └── Wait for user confirmation                          │
├─────────────────────────────────────────────────────────────┤
│ 3. GENERATE PROFILE LINK                                    │
│    └── https://foodcourts.ai/signup?wallet=0x...           │
│    └── User creates account (Google OAuth available)       │
├─────────────────────────────────────────────────────────────┤
│ 4. USER ADDS FUNDS                                          │
│    └── https://foodcourts.ai/fund                          │
│    └── Credit card via Stripe → USDC in wallet             │
│    └── User returns and confirms                           │
├─────────────────────────────────────────────────────────────┤
│ 5. CHECK BALANCE & START ORDERING                           │
│    └── GET /api/balance?wallet=0x...                       │
│    └── Confirm balance, ready to transact                  │
└─────────────────────────────────────────────────────────────┘
```

---

## References

- `references/wallet-creation.md` — Detailed wallet generation options
- `references/making-payments.md` — Complete payment implementation
- `examples/complete-setup-flow.md` — Full setup conversation example

## External Resources

- [x402.org](https://x402.org) — Official protocol documentation
- [Base Docs](https://docs.base.org) — Base blockchain documentation
- [BaseScan](https://basescan.org) — Block explorer for Base
- [FoodCourts](https://foodcourts.ai) — x402-enabled food ordering
