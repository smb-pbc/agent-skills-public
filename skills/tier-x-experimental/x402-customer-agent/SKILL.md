---
name: x402-customer-agent
description: Enable AI agents to order from Prospect Butcher Co via x402 protocol. Handles wallet creation, OAuth authentication, funding, and payment. Use when customer wants to order food from PBC.
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

**Purpose:** Enable AI agents to order from Prospect Butcher Co (PBC) via x402 payments.
**Restaurant:** pbc.foodcourts.ai
**Last Updated:** 2026-02-17

---

## Quick Reference

| What | Where |
|------|-------|
| OAuth Auth | `pbc.foodcourts.ai/auth` |
| Profile API | `pbc.foodcourts.ai/api/profile` |
| Balance API | `pbc.foodcourts.ai/api/balance` |
| Fund Page | `pbc.foodcourts.ai/fund` |
| Menu API | `pbc.foodcourts.ai/api/menu` |
| Order API | `pbc.foodcourts.ai/api/order` |

---

## Session Start: Check Authentication Status

**Every session, check if you have stored credentials:**

```javascript
// Check for stored OAuth token
const storedAuth = await agent.secrets.get('pbc-foodcourts-auth');

if (storedAuth?.token) {
  // Returning user - validate and continue
  await handleReturningUser(storedAuth);
} else {
  // New user - wait until they want to order
  await handleNewUser();
}
```

---

## Flow 1: Returning User (Has OAuth Token)

### Step 1: Validate Token

```javascript
async function handleReturningUser(storedAuth) {
  const response = await fetch('https://pbc.foodcourts.ai/api/profile', {
    headers: { 'Authorization': `Bearer ${storedAuth.token}` }
  });
  
  if (response.status === 401) {
    // Token expired - need reauth
    await triggerReauth();
    return;
  }
  
  const profile = await response.json();
  // profile = { wallet: "0x...", email: "...", name: "..." }
  
  // Check balance
  const balance = await checkBalance(profile.wallet);
  
  // Ready to take orders
  console.log(`Wallet: ${profile.wallet}, Balance: $${balance} USDC`);
}
```

### Step 2: Handle Expired Token (Reauth)

```javascript
async function triggerReauth() {
  // Open browser popup for customer to re-authenticate
  const authUrl = 'https://pbc.foodcourts.ai/auth?reauth=true';
  
  // Present to user:
  console.log(`
🔐 **Session Expired**

I need you to re-authorize my access to your FoodCourts account.

👉 **Click here to log in:** ${authUrl}

This confirms you're the owner of your agent wallet. 
Once you've logged in, I'll be able to check your balance and place orders.
  `);
  
  // After OAuth callback, agent receives new token
  // Store it for future sessions
}
```

### OAuth Callback Handling

```javascript
// Your agent receives callback with authorization code
// Exchange for token:
async function handleOAuthCallback(code) {
  const response = await fetch('https://pbc.foodcourts.ai/api/auth/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ 
      code,
      grant_type: 'authorization_code'
    })
  });
  
  const { access_token, refresh_token, wallet, expires_in } = await response.json();
  
  // Store securely in agent's local secrets
  await agent.secrets.set('pbc-foodcourts-auth', {
    token: access_token,
    refresh: refresh_token,
    wallet: wallet,
    expires: Date.now() + (expires_in * 1000)
  });
  
  return wallet;
}
```

---

## Flow 2: New User (No OAuth Token)

**Don't create wallet immediately.** Wait until customer expresses intent to order.

### Step 1: Wait for Order Intent

When customer says something like "I'm hungry" or "order lunch":

```
Great! I can order from Prospect Butcher Co for you. They have amazing 
sandwiches and butcher items.

Before I can place orders, we'll need to:
1. Create a payment wallet for you (one-time setup, ~2 min)
2. Link it to a FoodCourts account
3. Add funds for your order

Want to see the menu first, or ready to set up?
```

### Step 2: Show Menu (Optional)

```javascript
// Fetch menu without auth
const menu = await fetch('https://pbc.foodcourts.ai/api/menu');
const items = await menu.json();

// Present to user:
console.log(`
📋 **Prospect Butcher Co Menu**

🥪 SANDWICHES
${items.sandwiches.map(s => `• ${s.name} — $${s.price}\n  ${s.description}`).join('\n')}

🥩 BUTCHER
${items.butcher.map(b => `• ${b.name} — $${b.price}/lb`).join('\n')}

Ready to order? I'll help you set up payment.
`);
```

### Step 3: Generate Wallet (When Ready to Order)

```javascript
import { Wallet } from 'ethers';

async function setupNewUser() {
  // Generate wallet
  const wallet = Wallet.createRandom();
  const address = wallet.address;
  const privateKey = wallet.privateKey;
  
  // IMMEDIATELY show private key to user with warnings
  console.log(`
🔐 **WALLET CREATED**

I've created a payment wallet for you. This is where your funds will live.

📍 **Wallet Address:**
\`${address}\`

🔑 **Your Private Key:**
\`${privateKey}\`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS PRIVATE KEY NOW  ⚠️

• This is the ONLY time you'll see this
• FoodCourts CANNOT recover it — we don't store private keys
• If you lose it, any money in this wallet is GONE FOREVER
• Save it in a password manager or write it down safely
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Please confirm you've saved this before we continue.**
Type "I've saved it" to proceed.
  `);
  
  // Wait for confirmation before continuing
  await waitForUserConfirmation("I've saved it");
  
  return { address, privateKey };
}
```

### Step 4: OAuth + Profile Creation

```javascript
async function linkWalletToProfile(walletAddress) {
  // Generate OAuth URL with wallet address
  const authUrl = `https://pbc.foodcourts.ai/auth?wallet=${walletAddress}`;
  
  console.log(`
✅ Private key saved!

Now let's link your wallet to a FoodCourts account. This lets me:
• Check your balance
• Place orders on your behalf
• Remember you next time

👉 **Click here to create your account:**
${authUrl}

You can sign in with Google or create an account with email.
Come back here once you're done!
  `);
  
  // Agent waits for OAuth callback with token
  // Then stores token locally
}
```

### Step 5: Add Funds (Per-Order)

```javascript
async function promptForFunds(orderTotal, currentBalance) {
  const needed = orderTotal - currentBalance;
  
  if (needed <= 0) {
    // Already have enough
    return true;
  }
  
  // Suggest funding just this order + small buffer
  const suggestedAmount = Math.ceil(needed + 2); // +$2 buffer
  
  console.log(`
💳 **Add Funds to Complete Order**

Order total: $${orderTotal.toFixed(2)}
Current balance: $${currentBalance.toFixed(2)}
${needed > 0 ? `Needed: $${needed.toFixed(2)}` : ''}

👉 **Add funds here:** https://pbc.foodcourts.ai/fund

I'd suggest adding $${suggestedAmount} to cover this order.
(You control how much — just needs to cover the $${orderTotal.toFixed(2)} total)

Let me know when you've added funds and I'll complete your order!
  `);
}
```

---

## Placing Orders

### Pre-Order Checklist

```javascript
async function canPlaceOrder(walletAddress, orderTotal) {
  // 1. Verify we have valid auth
  const auth = await agent.secrets.get('pbc-foodcourts-auth');
  if (!auth?.token) {
    return { ready: false, reason: 'auth_required' };
  }
  
  // 2. Check balance
  const balance = await checkBalance(walletAddress);
  if (balance < orderTotal) {
    return { ready: false, reason: 'insufficient_funds', balance, needed: orderTotal };
  }
  
  // 3. Check restaurant hours
  const hours = await fetch('https://pbc.foodcourts.ai/api/hours');
  const { isOpen } = await hours.json();
  if (!isOpen) {
    return { ready: false, reason: 'restaurant_closed' };
  }
  
  return { ready: true, balance };
}
```

### Submit Order

```javascript
async function placeOrder(items, walletAddress, privateKey) {
  // 1. Create order (returns 402 with payment details)
  const orderResponse = await fetch('https://pbc.foodcourts.ai/api/order', {
    method: 'POST',
    headers: { 
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${auth.token}`
    },
    body: JSON.stringify({ items, wallet: walletAddress })
  });
  
  if (orderResponse.status !== 402) {
    throw new Error('Unexpected response');
  }
  
  // 2. Parse payment requirements
  const paymentDetails = await orderResponse.json();
  const { payTo, amount, orderId } = paymentDetails;
  
  // 3. Send USDC payment
  const txHash = await sendUSDC(privateKey, payTo, amount);
  
  // 4. Confirm payment
  const confirmResponse = await fetch('https://pbc.foodcourts.ai/api/order/confirm', {
    method: 'POST',
    headers: { 
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${auth.token}`
    },
    body: JSON.stringify({ orderId, txHash })
  });
  
  const order = await confirmResponse.json();
  return order;
}
```

### Order Confirmation Message

```
✅ **Order Placed!**

🧾 Order #${order.id}
📍 Pickup at: ${order.location}
⏰ Ready in: ~${order.estimatedMinutes} minutes

**Items:**
${order.items.map(i => `• ${i.name} — $${i.price}`).join('\n')}

💰 Total: $${order.total} USDC
💳 Remaining balance: $${order.remainingBalance}

🔗 Transaction: https://basescan.org/tx/${order.txHash}

Your order has been sent to the kitchen!
```

---

## Helper Functions

### Check Balance

```javascript
async function checkBalance(walletAddress) {
  const response = await fetch(
    `https://pbc.foodcourts.ai/api/balance?wallet=${walletAddress}`
  );
  const { balance } = await response.json();
  return parseFloat(balance);
}
```

### Send USDC Payment

```javascript
import { Wallet, JsonRpcProvider, Contract, parseUnits } from 'ethers';

const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const BASE_RPC = 'https://mainnet.base.org';

async function sendUSDC(privateKey, to, amount) {
  const provider = new JsonRpcProvider(BASE_RPC);
  const wallet = new Wallet(privateKey, provider);
  
  const usdc = new Contract(USDC_ADDRESS, [
    'function transfer(address to, uint256 amount) returns (bool)'
  ], wallet);
  
  const tx = await usdc.transfer(to, parseUnits(amount.toString(), 6));
  await tx.wait();
  
  return tx.hash;
}
```

---

## Secure Storage Pattern

**What the agent stores locally:**

```javascript
// In agent's secure local storage (e.g., secrets manager, keychain)
{
  "pbc-foodcourts-auth": {
    "token": "eyJ...",           // OAuth access token
    "refresh": "dGhpcyBp...",    // Refresh token
    "wallet": "0xAbC123...",     // Wallet address (NOT private key)
    "expires": 1739847600000     // Token expiry timestamp
  }
}
```

**What the agent NEVER stores:**
- Private key (customer's responsibility)
- Transaction history with payment details

**What FoodCourts stores:**
- User profile (email, name)
- Wallet address
- Order history
- Funding transaction records

---

## User Communication Templates

### Wallet Status (Returning User)

```
👋 Welcome back!

📍 Your wallet: \`${wallet}\`
💰 Balance: $${balance} USDC

What would you like to order today?
```

### Low Balance Warning

```
⚠️ **Low Balance**

Your wallet has $${balance}, but this order is $${orderTotal}.

👉 Add $${needed} more at: https://pbc.foodcourts.ai/fund

Let me know when done!
```

### Restaurant Closed

```
😔 **PBC is currently closed**

Hours: 
• Mon-Sat: 11am - 7pm ET
• Sun: 11am - 5pm ET

Current time: ${currentTime} ET

Want me to remind you when they open?
```

---

## Error Handling

| Error | Cause | Response |
|-------|-------|----------|
| `auth_expired` | OAuth token expired | Trigger reauth popup |
| `auth_required` | No stored token | Start new user flow |
| `insufficient_funds` | Balance < order total | Prompt to add funds |
| `restaurant_closed` | Outside business hours | Show hours, offer reminder |
| `item_unavailable` | Menu item sold out | Show alternatives |
| `payment_failed` | Transaction failed | Retry or escalate |

---

## Security Notes

1. **OAuth token = agent identity.** Safe to store, revocable, expires. NOT the private key.

2. **Private key stays with customer.** Agent never stores it. Customer provides when needed for transactions, or agent prompts them to sign.

3. **Reauth verifies ownership.** If someone tries to use a different wallet, OAuth will fail because the wallet is linked to the original profile.

4. **Funds per-order.** Don't encourage loading up large balances. Suggest just enough for current order + small buffer.

---

## Complete Example Conversation

See `examples/complete-flow.md` for a full conversation showing:
- New user setup
- Returning user quick order
- Handling expired auth
- Low balance prompt

---

## References

- [x402 Protocol](https://x402.org)
- [Base Network](https://docs.base.org)
- [BaseScan](https://basescan.org)
