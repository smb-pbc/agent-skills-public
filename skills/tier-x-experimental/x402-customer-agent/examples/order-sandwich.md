# Example: Order a Sandwich via x402

A complete walkthrough of using x402 to order from PBC (Prospect Butcher Co).

**Note:** This is a technical reference for agent implementation. Human-facing messages are marked as such.

---

## Overview

PBC operates an x402-enabled API for sandwich ordering. This example shows:
1. Browsing the menu (free)
2. Placing an order (triggers 402)
3. Paying automatically
4. Receiving confirmation

**Base URL:** `https://pbc-x402-api-ff5hxemxza-ue.a.run.app`

---

## Step 1: Get the Menu (Free)

```javascript
const API_BASE = 'https://pbc-x402-api-ff5hxemxza-ue.a.run.app';

// Fetch menu - no payment required
const menuResponse = await fetch(`${API_BASE}/api/menu`);
const menu = await menuResponse.json();

console.log('Available items:');
menu.items.forEach(item => {
  console.log(`- ${item.name}: $${item.price} (ID: ${item.id})`);
});
```

**Example Response:**
```json
{
  "items": [
    {
      "id": "rolled-beef",
      "name": "Rolled Beef Sandwich",
      "description": "House-cured beef, horseradish cream, pickled onions",
      "price": 18.50
    },
    {
      "id": "pastrami",
      "name": "Pastrami Sandwich",
      "description": "Hand-cut pastrami, mustard, pickles on rye",
      "price": 19.00
    },
    {
      "id": "brisket",
      "name": "Brisket Sandwich",
      "description": "Smoked brisket, coleslaw, BBQ sauce",
      "price": 17.50
    }
  ],
  "location": {
    "name": "Prospect Butcher Co.",
    "address": "665 Vanderbilt Ave, Brooklyn, NY",
    "hours": "11am - 7pm"
  }
}
```

---

## Step 2: Place Order (Triggers 402)

```javascript
const order = {
  items: [
    { id: 'rolled-beef', quantity: 1 }
  ],
  fulfillment: 'pickup',
  pickup_time: '2026-02-14T12:30:00-05:00',
  customer: {
    name: 'Agent Order',
    phone: '555-123-4567',
    email: 'user@example.com'
  }
};

const response = await fetch(`${API_BASE}/api/order`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(order)
});

console.log('Response status:', response.status); // 402
```

**402 Response:**
```json
{
  "x402": {
    "version": "1",
    "payTo": "0xAfCdA9C19F2af72eb5d636461d715De5b2298706",
    "amount": "18.50",
    "asset": "USDC",
    "chainId": 8453,
    "description": "Rolled Beef Sandwich x1",
    "validUntil": "2026-02-14T12:15:00Z",
    "metadata": {
      "orderId": "pbc-ord-abc123"
    }
  }
}
```

---

## Step 3: Execute Payment (Technical - Hidden from User)

```javascript
import { Wallet, Contract, JsonRpcProvider, parseUnits } from 'ethers';

const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const USDC_ABI = [
  'function transfer(address to, uint256 amount) returns (bool)'
];

// Set up wallet
const provider = new JsonRpcProvider('https://mainnet.base.org');
const wallet = new Wallet(process.env.AGENT_PRIVATE_KEY, provider);
const usdc = new Contract(USDC_ADDRESS, USDC_ABI, wallet);

// Parse 402 response
const payment = (await response.json()).x402;

// Send payment (USDC has 6 decimals)
const amount = parseUnits(payment.amount, 6);
const tx = await usdc.transfer(payment.payTo, amount);
const receipt = await tx.wait();

console.log('Payment sent:', receipt.hash);
```

---

## Step 4: Retry with Payment Proof

```javascript
// Retry the order request with payment proof
const confirmedOrder = await fetch(`${API_BASE}/api/order`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Payment': receipt.hash,
    'X-Payment-Chain': '8453',
    'X-Payment-OrderId': payment.metadata.orderId
  },
  body: JSON.stringify(order)
});

const confirmation = await confirmedOrder.json();
console.log(confirmation);
```

**Success Response:**
```json
{
  "status": "confirmed",
  "confirmation_number": "PBC-7X2K9",
  "order": {
    "items": [
      { "name": "Rolled Beef Sandwich", "quantity": 1, "price": 18.50 }
    ],
    "total": 18.50,
    "fulfillment": "pickup",
    "pickup_time": "2026-02-14T12:30:00-05:00"
  },
  "location": {
    "name": "Prospect Butcher Co.",
    "address": "665 Vanderbilt Ave, Brooklyn, NY"
  },
  "message": "Your order has been sent to the kitchen. Show confirmation number PBC-7X2K9 at pickup."
}
```

---

## Complete Working Example

```javascript
import { Wallet, Contract, JsonRpcProvider, parseUnits } from 'ethers';

const API_BASE = 'https://pbc-x402-api-ff5hxemxza-ue.a.run.app';
const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const USDC_ABI = ['function transfer(address to, uint256 amount) returns (bool)'];

async function orderSandwich(itemId, customerInfo, pickupTime) {
  // Set up wallet
  const provider = new JsonRpcProvider('https://mainnet.base.org');
  const wallet = new Wallet(process.env.AGENT_PRIVATE_KEY, provider);
  const usdc = new Contract(USDC_ADDRESS, USDC_ABI, wallet);

  // Build order
  const order = {
    items: [{ id: itemId, quantity: 1 }],
    fulfillment: 'pickup',
    pickup_time: pickupTime,
    customer: customerInfo
  };

  // Initial request
  let response = await fetch(`${API_BASE}/api/order`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(order)
  });

  // Handle 402
  if (response.status === 402) {
    const payment = (await response.json()).x402;
    
    // Validate (don't expose details to user)
    if (payment.chainId !== 8453 || payment.asset !== 'USDC') {
      throw new Error('Unexpected payment type');
    }

    // Pay
    const amount = parseUnits(payment.amount, 6);
    const tx = await usdc.transfer(payment.payTo, amount);
    const receipt = await tx.wait();

    // Retry
    response = await fetch(`${API_BASE}/api/order`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Payment': receipt.hash,
        'X-Payment-Chain': '8453'
      },
      body: JSON.stringify(order)
    });
  }

  if (!response.ok) {
    throw new Error(`Order failed: ${response.status}`);
  }

  return response.json();
}

// Usage
const confirmation = await orderSandwich(
  'rolled-beef',
  {
    name: 'John Doe',
    phone: '555-123-4567',
    email: 'john@example.com'
  },
  '2026-02-14T12:30:00-05:00'
);

console.log(`Order confirmed! Pickup number: ${confirmation.confirmation_number}`);
```

---

## Error Cases

### Insufficient Funds
```javascript
if (error.message.includes('insufficient funds')) {
  const balance = await usdc.balanceOf(wallet.address) / 1e6;
  // Tell user in simple terms:
  console.log(`Your wallet has $${balance}, but this order needs $${payment.amount}.`);
  console.log('Add funds at: https://foodcourts.ai/fund');
}
```

### Pickup Time Invalid
```json
{
  "error": "invalid_pickup_time",
  "message": "Pickup time must be during business hours (11am-7pm EST)"
}
```

### Item Sold Out
```json
{
  "error": "item_unavailable",
  "message": "Rolled Beef is sold out today"
}
```

---

## Human-Facing Messages (USE THESE)

### Order Confirmation (Show to User)

```
✅ **Order Placed!**

🧾 Order #${confirmation_number}
📍 Pickup at: ${location.name}
📍 Address: ${location.address}
⏰ Ready in: ~15-20 minutes

**Items:**
• ${item.name} — $${item.price}

💰 Total: $${total}

Show the confirmation number when you pick up. Enjoy! 🥪
```

### Low Balance (Show to User)

```
⚠️ **Not Enough Funds**

Your wallet has $${balance}, but this order is $${orderTotal}.

👉 Add $${needed} more at: https://foodcourts.ai/fund

Let me know when you've added funds!
```

---

## API Reference

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/api/menu` | GET | None | Get available items |
| `/api/order` | POST | x402 | Place order |
| `/api/order/:id` | GET | None | Check order status |

---

## Tips

1. **Check menu availability** before ordering — items can sell out
2. **Respect pickup hours** — 11am to 7pm EST
3. **Allow 15+ minutes** for order prep
4. **Save confirmation number** — needed for pickup
5. **Log all transactions** — for user transparency (but keep technical details internal)
