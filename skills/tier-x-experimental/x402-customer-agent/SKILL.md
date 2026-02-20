---
name: x402-customer-agent
description: Enable AI agents to order food via x402 payments. Handles wallet creation, profile linking, funding, and ordering. Works for both new users (full onboarding) and existing users (wallet linking only). Detects context from Profile ID.
triggers:
  - order food
  - order sandwich
  - prospect butcher
  - pbc
  - foodcourts
  - hungry
  - lunch
  - get food
  - link wallet
  - connect wallet
  - my profile id is
---

# x402 Customer Agent Skill

**Purpose:** Enable AI agents to order food via automated payments.
**Supports:** New user onboarding OR existing user wallet linking (auto-detects)
**API Base:** foodcourts.ai
**Last Updated:** 2026-02-19

---

## Quick Reference

| What | Where |
|------|-------|
| Menu API | `pbc.foodcourts.ai/api/menu` |
| Hours API | `pbc.foodcourts.ai/api/hours` |
| Order API | `pbc.foodcourts.ai/api/order` |
| Order Status | `pbc.foodcourts.ai/api/order/:id` |
| Fund Page | `foodcourts.ai/fund?wallet={ADDRESS}` |
| Link Wallet | `foodcourts.ai/profile/link-wallet?wallet={ADDRESS}` |
| Sign Up | `foodcourts.ai/signup?wallet={ADDRESS}` |

---

## 🔀 User Flow Detection

**Always start by asking:** "Do you already have a FoodCourts account?"

| User Response | Flow | What To Do |
|---------------|------|------------|
| **YES** (has account) | **Existing User** | Ask for Profile ID → wallet → link → fund |
| **NO** (new user) | **New User** | Full flow: wallet → signup URL → fund |
| **Profile ID provided** (e.g., "my Profile ID is ABC123") | **Existing User** | Skip question → wallet → link → fund |

---

## Existing User Flow (Has FoodCourts Account)

```
User already has FoodCourts account.
They just need to connect an agentic wallet.

1. Ask for their Profile ID (visible at foodcourts.ai/profile)
2. Create wallet (with key handling below)
3. Show wallet proof URL
4. Direct them to link: https://foodcourts.ai/profile/link-wallet?wallet=[ADDRESS]
5. Guide them to fund via https://foodcourts.ai/fund?wallet=[ADDRESS]
6. Ready to order
```

### Step 1: Get Profile ID

```
AGENT: Great! To link a wallet to your existing account, I'll need your 
Profile ID. You can find it at https://foodcourts.ai/profile

What's your Profile ID?

USER: It's ABC123

AGENT: Got it! Creating your agentic wallet now...
```

---

## New User Flow (No FoodCourts Account)

```
User doesn't have a FoodCourts account yet.

1. Create wallet (with key handling below)
2. Show wallet proof URL
3. Send them to create profile: https://foodcourts.ai/signup?wallet=[ADDRESS]
4. They create account (Google OAuth)
5. Wallet auto-links during signup
6. Guide them to fund
7. Ready to order
```

---

## When to Create Wallet

| User Action | What To Do |
|-------------|------------|
| "What's on the menu?" | Show menu (no wallet needed) |
| "What can I order?" | Show menu (no wallet needed) |
| "I want to order" | Ask about FoodCourts account → create wallet → fund → order |
| "I'm hungry" | Ask about FoodCourts account → create wallet → fund → order |

---

## Step 1: Create Wallet

When the user wants to place an order, create a wallet with these security requirements:

### a. Generate Keys (JavaScript or Python)

**Option A: JavaScript (Node.js with ethers.js)**
```javascript
import { Wallet } from 'ethers';
const wallet = Wallet.createRandom();
// wallet.address, wallet.privateKey
```

**Option B: Python (if Node.js unavailable)**
```python
from eth_account import Account
import secrets
private_key = secrets.token_hex(32)
account = Account.from_key(private_key)
# account.address, private_key (without 0x prefix)
```

### b. 🚨 NEVER Print Private Key in Chat Messages

**⚠️ CRITICAL SECURITY RULE:**
In chat environments (Slack, Discord, Telegram, etc.), messages are **logged permanently**. 
Printing a private key in chat = **permanently compromised wallet**.

**Instead, do ONE of these:**

1. **Store directly to OS keychain** (preferred) — never show the key at all
2. **Write to a local file** the user can view and delete: `~/.foodcourts-wallet-key.txt`
3. **For terminal-only agents**: Can display briefly, but warn user to clear terminal history

**NEVER output the raw private key in a message that will be logged.**

### c. Save to OS Keychain

Use service name: `foodcourts-wallet-key` (consistent naming)

**macOS:**
```bash
security add-generic-password -a "foodcourts-agent" -s "foodcourts-wallet-key" -w "[PRIVATE_KEY]" -U
```

**Then notify user:**
```
🔐 **AGENTIC WALLET CREATED**

📍 **Wallet Address:**
`[address]`

✅ **Private key saved securely**
   Location: macOS Keychain → "foodcourts-wallet-key"
   
   To view it later: Keychain Access → search "foodcourts"

🔗 **Verify wallet exists:**
https://basescan.org/address/[ADDRESS]
```

### d. Do NOT:
- Print private key in chat/Slack/Discord (messages are logged!)
- Log the private key to files that persist
- Transmit it over the network
- Store it anywhere other than secure OS keychain

### e. Backup Guidance

After saving to keychain, tell the user:
```
Your private key is saved in your system keychain.

⚠️ **BACKUP RECOMMENDATION:**
Open Keychain Access → search "foodcourts" → copy the password 
→ save it in a password manager (1Password, Bitwarden, etc.)

This is your only way to recover funds if your computer is lost.
```

### f. Wait for User Confirmation
```javascript
// STOP and wait for explicit confirmation
await waitForUserConfirmation("I understand my key is saved in the keychain");
// Do not proceed until user confirms
```

---

## Step 2: Link Wallet to Account

Choose the right URL based on the user's account status:

| User Status | URL | Notes |
|-------------|-----|-------|
| **New user** (no account) | `https://foodcourts.ai/signup?wallet={ADDRESS}` | Creates account, then auto-links wallet |
| **Existing user** (has account) | `https://foodcourts.ai/profile/link-wallet?wallet={ADDRESS}` | Confirms and links wallet |
| **Unknown** (not sure) | `https://foodcourts.ai/profile/link-wallet?wallet={ADDRESS}` | Handles login redirect; has "Sign up" link for new users |

### For NEW Users (no account):
```
AGENT: Now let's create your FoodCourts account and link your wallet.

👉 **Sign up here:**
https://foodcourts.ai/signup?wallet={ADDRESS}

Sign in with Google — your wallet will be linked automatically!

Come back here once you're done.
```

### For EXISTING Users (has account):
```
AGENT: Now let's link your wallet to your FoodCourts account.

👉 **Click here to link your wallet:**
https://foodcourts.ai/profile/link-wallet?wallet={ADDRESS}

Make sure you're logged in, then click "Confirm" to link.

Let me know when you're done!
```

### Default (unknown status):
Use the link-wallet URL — it handles login redirects and includes a "Sign up" link for users who don't have an account yet.

---

## Step 3: Fund the Wallet

Always use the address-based funding URL so the correct wallet is pre-selected:

```
https://foodcourts.ai/fund?wallet={ADDRESS}
```

The system verifies that the address matches the user's linked wallets.

**Human-facing message (simple, no crypto jargon):**
```
AGENT: Now let's add funds so you can place orders.

💳 **Add funds to your wallet:**
https://foodcourts.ai/fund?wallet={ADDRESS}

**Order Summary:**
• The Pitmaster — $16.50
• Tax — $1.47
• **Total: $17.97**

I'd suggest adding $20 to cover this order with a small buffer.
Let me know when you've added funds!
```

---

## ⚠️ MANDATORY: Spending Rules

**Follow these strictly for ALL transactions:**

1. **Never send any transaction without showing details AND getting explicit "yes":**
   ```
   📤 **Transaction Request**
   
   Sending: $18.50
   To: PBC Order Payment
   Network Fee: ~$0.01
   
   Type "yes" to confirm this payment.
   ```

2. **If user asks to return funds:**
   - Send full available balance minus fees
   - Send to whatever address they specify
   - Show confirmation before sending

3. **Never approve or sign anything other than standard transfers**

4. **Always show balance before and after each transaction**

---

## Payment Flow (Technical - Agent Only)

The PBC API uses Stripe's native crypto payment infrastructure. The flow is:

1. **POST /api/order** with items + customer info
2. **Get 402 Payment Required** with unique Stripe deposit address
3. **Send USDC** to the deposit address on Base network
4. **Payment auto-confirms** when USDC arrives (Stripe detects it)
5. **Poll for status** or receive webhook callback
6. **Customer picks up** at the Brooklyn shop

**Technical Details (hidden from user):**
- Network: Base (Chain ID 8453)
- Token: USDC
- USDC Contract: 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913

---

## Session Start: Wallet Status Check

```javascript
// Check if customer has a funded wallet
async function checkWalletStatus(walletAddress) {
  if (!walletAddress) {
    return { ready: false, reason: 'no_wallet' };
  }

  const balance = await getUSDCBalance(walletAddress);
  if (balance < 10) {  // Minimum for a sandwich order
    return { ready: false, reason: 'low_balance', balance };
  }

  return { ready: true, balance };
}
```

**Key principle:** Don't create a wallet until the customer wants to order. Browsing the menu doesn't require a wallet.

---

## Pre-Order Balance Verification

Before placing any order, always verify the wallet has sufficient funds:

```javascript
async function verifyBalanceForOrder(walletAddress, items) {
  // 1. Get current balance
  const balance = await getUSDCBalance(walletAddress);

  // 2. Estimate order total
  const subtotal = items.reduce((sum, item) => sum + item.price, 0);
  const tax = subtotal * 0.08875; // NYC tax rate
  const serviceFee = 1.00;
  const estimatedTotal = subtotal + tax + serviceFee;

  // 3. Compare
  if (balance < estimatedTotal) {
    return {
      sufficient: false,
      balance,
      estimatedTotal,
      shortfall: estimatedTotal - balance
    };
  }

  return { sufficient: true, balance, estimatedTotal };
}
```

If balance is insufficient, trigger the **Wallet Refill Flow** below.

---

## Wallet Refill Flow

When the agent detects the wallet balance is too low for an order:

### Step 1: Detect Low Balance
Use `verifyBalanceForOrder()` above to determine the shortfall.

### Step 2: Generate Funding URL
```
https://foodcourts.ai/fund?wallet={AGENT_WALLET_ADDRESS}
```
The `?wallet=` parameter pre-selects the correct wallet on the funding page.

### Step 3: Present to User
Use the "Wallet Balance Low" communication template (see below). Keep messaging simple — no crypto jargon:
```
Your wallet balance is $X.XX, but this order costs about $Y.YY.
Please add funds here: https://foodcourts.ai/fund?wallet={ADDRESS}

Let me know when you've added funds, or I'll detect them automatically!
```

### Step 4: Poll for Balance Increase
```javascript
// Poll on-chain balance every 10 seconds (5 min timeout)
// See references/funding-wallet.md for full implementation
const newBalance = await waitForFunding(walletAddress, currentBalance);
```

### Step 5: Confirm Receipt
When balance increases:
```
I see the funds! Your wallet now has $Z.ZZ. Ready to place the order?
```

---

## Placing an Order

### Step 1: Check Menu and Hours

```javascript
// Get menu
const menuResponse = await fetch('https://pbc.foodcourts.ai/api/menu');
const menu = await menuResponse.json();

// Check if shop is open
const hoursResponse = await fetch('https://pbc.foodcourts.ai/api/hours?location=shop-2');
const hours = await hoursResponse.json();

if (!hours.isOpen) {
  console.log(`Shop is closed. ${hours.message}`);
  return;
}
```

### Step 2: Submit Order (Get Payment Instructions)

```javascript
async function createOrder(items, location, customer) {
  const response = await fetch('https://pbc.foodcourts.ai/api/order', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items,           // e.g., ["boudinwich", "ramune"]
      location,        // "shop-1" or "shop-2"
      customer: {
        name: customer.name,
        phone: customer.phone,
        email: customer.email  // optional
      },
      // Optional: Get status callbacks
      agentCallback: {
        type: "webhook",
        url: "https://your-agent.com/callbacks"
      }
    })
  });

  // Will return 402 Payment Required with deposit address
  if (response.status === 402) {
    const paymentDetails = await response.json();
    return paymentDetails;
  }
  
  throw new Error(`Unexpected status: ${response.status}`);
}
```

### Step 3: Parse Payment Instructions

The 402 response includes:

```json
{
  "status": 402,
  "message": "Payment required",
  "orderId": "PBC-ABC123",
  "items": [{"id": "boudinwich", "name": "Boudinwich", "price": 16}],
  "subtotal": 16.00,
  "tax": 1.42,
  "serviceFee": 1.00,
  "total": 18.42,
  "payment": {
    "required": true,
    "amount": 18.42,
    "currency": "USD",
    "asset": "USDC",
    "network": "Base",
    "chainId": 8453,
    "depositAddress": "0x...",  // Unique per order
    "expiresAt": "2026-02-17T15:30:00Z",
    "expiresIn": "10 minutes"
  },
  "pickup": {
    "location": "Prospect Butcher Co (Greenpoint)",
    "address": "113A Nassau Ave, Brooklyn NY 11222"
  },
  "_polling": {
    "statusUrl": "/api/order/PBC-ABC123",
    "interval": "30 seconds"
  }
}
```

### Step 4: Send Payment

```javascript
import { Wallet, JsonRpcProvider, Contract, parseUnits } from 'ethers';

const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const BASE_RPC = 'https://mainnet.base.org';

async function sendPayment(privateKey, depositAddress, amount) {
  const provider = new JsonRpcProvider(BASE_RPC);
  const wallet = new Wallet(privateKey, provider);
  
  const usdc = new Contract(USDC_ADDRESS, [
    'function transfer(address to, uint256 amount) returns (bool)'
  ], wallet);
  
  // USDC has 6 decimals
  const amountWei = parseUnits(amount.toString(), 6);
  
  const tx = await usdc.transfer(depositAddress, amountWei);
  const receipt = await tx.wait();
  
  console.log(`✅ Payment sent: ${tx.hash}`);
  return tx.hash;
}
```

### Step 5: Poll for Confirmation

```javascript
async function waitForConfirmation(orderId, maxAttempts = 20) {
  for (let i = 0; i < maxAttempts; i++) {
    const response = await fetch(`https://pbc.foodcourts.ai/api/order/${orderId}`);
    const order = await response.json();
    
    if (order.status === 'pending_chownow' || order.status === 'confirmed') {
      return order;  // Payment confirmed!
    }
    
    if (order.status === 'payment_failed' || order.status === 'payment_expired') {
      throw new Error(`Payment ${order.status}: ${order._status.message}`);
    }
    
    // Wait 10 seconds between polls
    await new Promise(r => setTimeout(r, 10000));
  }
  
  throw new Error('Timeout waiting for payment confirmation');
}
```

---

## Complete Order Flow Example

```javascript
async function orderSandwich(customer, items, location = 'shop-2') {
  // 1. Create order and get payment instructions
  const paymentDetails = await createOrder(items, location, customer);
  
  // Show user-friendly message (no technical details)
  console.log(`
🧾 **Order Created: ${paymentDetails.orderId}**

Total: $${paymentDetails.total.toFixed(2)}

Processing payment...
  `);

  // 2. Send payment (using customer's wallet)
  const txHash = await sendPayment(
    customerPrivateKey,
    paymentDetails.payment.depositAddress,
    paymentDetails.payment.amount
  );

  // 3. Wait for order confirmation
  const confirmedOrder = await waitForConfirmation(paymentDetails.orderId);
  
  console.log(`
✅ **Order Confirmed!**

🧾 Order: ${confirmedOrder.orderId}
📍 Pickup: ${confirmedOrder.locationName}
⏰ Ready in: ~20 minutes

Your sandwich is being prepared!
  `);
  
  return confirmedOrder;
}
```

---

## Order Status Values

| Status | Meaning |
|--------|---------|
| `pending_payment` | Awaiting payment |
| `paid` | Payment confirmed |
| `pending_chownow` | Queued for fulfillment |
| `confirmed` | Order placed with kitchen |
| `ready_for_pickup` | Order ready |
| `picked_up` | Complete |
| `payment_expired` | 10-minute window expired |
| `payment_failed` | Payment issue |

---

## Error Handling

| Error | Cause | Response |
|-------|-------|----------|
| `shop_closed` | Outside business hours | Show hours, offer to check later |
| `insufficient_funds` | Balance < order total | Trigger Wallet Refill Flow (see above) |
| `payment_expired` | Took >10 min to pay | Need to create new order |
| `invalid_items` | Menu items not found | Show menu, ask to re-select |

---

## Agent Callbacks (Optional)

Set `agentCallback` in order request to receive status updates:

```javascript
{
  agentCallback: {
    type: "webhook",           // or "clawdbot"
    url: "https://...",        // Your callback endpoint
    agentId: "agent-123"       // Optional identifier
  }
}
```

You'll receive POST notifications for status changes:

```json
{
  "orderId": "PBC-ABC123",
  "status": "confirmed",
  "location": "Prospect Butcher Co (Greenpoint)",
  "message": "Your order is being prepared!",
  "estimatedReady": "2026-02-17T15:45:00Z"
}
```

---

## Security Notes (Technical - Agent Only)

1. **Private key stays with customer.** Never store it. Customer provides for transactions.

2. **Deposit addresses are unique per order.** Don't reuse them.

3. **10-minute expiration.** If payment isn't sent in time, create a new order.

4. **Base network only.** Payments on other networks won't work.

---

## User Communication Templates

### Order Ready
```
✅ **Order Placed!**

🧾 Order #${orderId}
📍 Pickup at: ${locationName}
📍 Address: ${address}
⏰ Ready in: ~20 minutes

**Items:**
${items.map(i => `• ${i.name} — $${i.price}`).join('\n')}

💰 Total: $${total}

Your sandwich is being prepared!
```

### Wallet Balance Low
```
⚠️ **Not Enough Funds**

Your wallet has $${balance}, but this order costs about $${estimatedTotal}.

💳 **Add funds here:**
https://foodcourts.ai/fund?wallet=${walletAddress}

I'd suggest adding $${suggestedAmount} to cover this order with a small buffer.
Let me know when you've added funds, or I'll detect them automatically!
```

### Waiting for Funds
```
⏳ **Waiting for Funds**

I'm monitoring your wallet in the background. As soon as the funds arrive,
I'll let you know and we can proceed with your order.

You can add funds here if you haven't already:
https://foodcourts.ai/fund?wallet=${walletAddress}
```

### Funds Received
```
✅ **Funds Received!**

Your wallet balance is now $${newBalance}.

Ready to place your order?
```

### Wallet Not Linked
```
🔗 **Wallet Not Linked**

Before I can place orders, your wallet needs to be linked to a FoodCourts account.

👉 **Link your wallet:**
https://foodcourts.ai/profile/link-wallet?wallet=${walletAddress}

**Don't have an account yet?**
👉 https://foodcourts.ai/signup?wallet=${walletAddress}

Let me know when you're set up!
```

### Shop Closed
```
😔 **PBC is currently closed**

${message}

Hours: Mon-Sat 11am-7pm ET

Would you like me to remind you when they open?
```

---

## References

- [x402 Protocol](https://x402.org)
- [FoodCourts](https://foodcourts.ai)
- [BaseScan](https://basescan.org) (for wallet verification)
