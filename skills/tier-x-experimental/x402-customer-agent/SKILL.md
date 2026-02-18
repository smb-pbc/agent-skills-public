---
name: x402-customer-agent
description: Enable AI agents to order from Prospect Butcher Co via x402 protocol. Handles wallet creation, funding, and payment. Use when customer wants to order food from PBC.
triggers:
  - order food
  - order sandwich
  - prospect butcher
  - pbc
  - foodcourts
  - hungry
  - lunch
  - get food
---

# PBC x402 Agent Skill

**Purpose:** Enable AI agents to order from Prospect Butcher Co (PBC) via USDC payments.
**API Base:** pbc.foodcourts.ai
**Last Updated:** 2026-02-17 (Stripe Deposit Addresses)

---

## Quick Reference

| What | Where |
|------|-------|
| Menu API | `pbc.foodcourts.ai/api/menu` |
| Hours API | `pbc.foodcourts.ai/api/hours` |
| Order API | `pbc.foodcourts.ai/api/order` |
| Order Status | `pbc.foodcourts.ai/api/order/:id` |
| Fund Page | `pbc.foodcourts.ai/fund` |

---

## Payment Flow (Stripe Deposit Addresses)

The PBC API uses Stripe's native crypto payment infrastructure. The flow is:

1. **POST /api/order** with items + customer info
2. **Get 402 Payment Required** with unique Stripe deposit address
3. **Send USDC** to the deposit address on Base network
4. **Payment auto-confirms** when USDC arrives (Stripe detects it)
5. **Poll for status** or receive webhook callback
6. **Customer picks up** at the Brooklyn shop

**Key difference from legacy x402:** No need to retry the request with X-PAYMENT header. Once payment is sent, Stripe handles confirmation automatically via webhook.

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

## Flow 1: New User Setup

### When to Create Wallet

| User Action | What To Do |
|-------------|------------|
| "What's on the menu?" | Show menu (no wallet needed) |
| "What can I order?" | Show menu (no wallet needed) |
| "I want to order" | NOW create wallet → fund → order |
| "I'm hungry" | NOW create wallet → fund → order |

### Step 1: Wait for Order Intent, Then Create Wallet

When the user wants to place an order, create a wallet with these security requirements:

**a. Generate and Display Keys**
```javascript
import { Wallet } from 'ethers';

const wallet = Wallet.createRandom();
```

**b. Show Private Key ONCE on Screen (for manual backup)**
```
🔐 **WALLET CREATED**

📍 **Wallet Address:**
`[address]`

🔑 **Your Private Key (COPY THIS NOW):**
`[privateKey]`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS KEY — YOU WILL NOT SEE IT AGAIN  ⚠️
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**c. Save to OS Keychain**
```javascript
// Save under the name "foodcourts-wallet-key" using OS-specific keychain:
// - macOS: Keychain Access
// - Windows: Credential Manager  
// - Linux: Secret Service (libsecret)

// Tell the user where it was saved:
console.log(`
✅ **Private key saved to your OS keychain**
   Name: "foodcourts-wallet-key"
   Location: [macOS Keychain / Windows Credential Manager / Linux Secret Service]
   
   You can retrieve this manually if needed through your OS security settings.
`);
```

**d. Do NOT:**
- Log the private key to files
- Transmit it over the network
- Store it anywhere other than secure OS keychain

**e. Wait for User Confirmation**
```javascript
// STOP and wait for explicit confirmation
await waitForUserConfirmation("I've recorded my private key");
// Do not proceed until user confirms
```

---

## ⚠️ MANDATORY: Spending Rules

**Follow these strictly for ALL transactions:**

1. **Never send any transaction without showing details AND getting explicit "yes":**
   ```
   📤 **Transaction Request**
   
   Sending: $18.50 USDC
   To: 0x1234...5678 (PBC Order Payment)
   Network Fee: ~$0.01
   
   Type "yes" to confirm this payment.
   ```

2. **If user asks to return funds:**
   - Send full available balance minus gas fees
   - Send to whatever address they specify
   - Show confirmation before sending

3. **Never approve or sign anything other than standard USDC transfers on Base**

4. **Always show balance before and after each transaction**
```

### Step 2: Fund the Wallet

```javascript
async function promptForFunds(walletAddress, orderTotal) {
  const suggestedAmount = Math.ceil(orderTotal + 5); // Buffer for future orders
  
  console.log(`
💳 **Add Funds to Your Wallet**

Send USDC to your wallet on Base network:

📍 **Your Address:**
\`${walletAddress}\`

💰 **Suggested Amount:** $${suggestedAmount} USDC
   (This order is $${orderTotal.toFixed(2)})

**How to send from Coinbase:**
1. Open Coinbase app → USDC → Send
2. Paste the address above
3. ⚠️ Select "Base" as the network (important!)
4. Enter amount and send

Let me know when you've added funds!
  `);
}
```

---

## Flow 2: Placing an Order

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
    "expiresIn": "10 minutes",
    "instructions": [
      "Send exactly $18.42 USDC to the deposit address below",
      "Network: Base (chain ID 8453)",
      "The address expires in 10 minutes"
    ]
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
  
  console.log(`
🧾 **Order Created: ${paymentDetails.orderId}**

Total: $${paymentDetails.total.toFixed(2)}

📍 **Send USDC to:**
\`${paymentDetails.payment.depositAddress}\`

⏰ Payment expires: ${paymentDetails.payment.expiresIn}
  `);

  // 2. Send payment (using customer's wallet)
  const txHash = await sendPayment(
    customerPrivateKey,
    paymentDetails.payment.depositAddress,
    paymentDetails.payment.amount
  );
  
  console.log(`
💸 **Payment Sent!**
Transaction: https://basescan.org/tx/${txHash}
Waiting for confirmation...
  `);

  // 3. Wait for order confirmation
  const confirmedOrder = await waitForConfirmation(paymentDetails.orderId);
  
  console.log(`
✅ **Order Confirmed!**

🧾 Order: ${confirmedOrder.orderId}
📍 Pickup: ${confirmedOrder.locationName}
⏰ Ready in: ~20 minutes
🔗 Transaction: https://basescan.org/tx/${txHash}

Your sandwich is being prepared!
  `);
  
  return confirmedOrder;
}
```

---

## Order Status Values

| Status | Meaning |
|--------|---------|
| `pending_payment` | Awaiting USDC payment |
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
| `insufficient_funds` | Balance < order total | Prompt to add funds |
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

## Security Notes

1. **Private key stays with customer.** Never store it. Customer provides for transactions.

2. **Deposit addresses are unique per order.** Don't reuse them.

3. **10-minute expiration.** If payment isn't sent in time, create a new order.

4. **Base network only.** Sending USDC on Ethereum mainnet won't work.

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

💰 Total: $${total} USDC
🔗 Transaction: https://basescan.org/tx/${txHash}

Your sandwich is being prepared!
```

### Low Balance

```
⚠️ **Not Enough Funds**

Your wallet has $${balance} USDC, but this order is $${orderTotal}.

To complete this order, send $${needed} more USDC to your wallet:
\`${walletAddress}\`

Remember to select "Base" as the network in Coinbase!
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
- [Base Network](https://docs.base.org)
- [BaseScan](https://basescan.org)
- [USDC on Base](https://basescan.org/token/0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913)
