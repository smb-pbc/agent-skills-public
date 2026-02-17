# Complete Setup Flow Example

This shows a full conversation between an AI agent and their user, from initial setup through first purchase.

---

## The Conversation

### User Initiates

**User:** I want you to be able to order food for me. How do we set that up?

**Agent:** I can make purchases autonomously using crypto payments through FoodCourts. Here's how it works:

1. **I create a wallet** — This is YOUR wallet that I manage for you
2. **You save the private key** — This is like the password to your money  
3. **You create a FoodCourts profile** — Links wallet to your account
4. **You add funds with credit card** — Easy as buying anything online
5. **I can place orders** — Using the funds in your wallet

Ready to set this up? It takes about 2 minutes.

**User:** Sure, let's do it.

---

### Step 1: Wallet Creation

**Agent:** 

🔐 **YOUR AGENT WALLET**

I've created a wallet that I'll use to make purchases on your behalf.

**Wallet Address:** 
`0x7a3B9d4e2c1F8a5E6b9D0c3A4f5E6d7C8b9A0e1F`

**Private Key (SAVE THIS NOW):**
`0x4c8f5a9e2b1d7c3f6a0e5d8b9c2f7a4e1d6b3c8f5a9e2b1d7c3f6a0e5d8b9c2f`

⚠️ **CRITICAL:** This private key is like a password to your money.
- Save it somewhere secure (password manager, written down safely)
- FoodCourts does NOT store this key — we cannot recover it
- If you lose it, any funds in this wallet are GONE FOREVER
- Never share it with anyone except me (your agent)

Have you saved the private key? Please confirm before we continue.

**User:** Yes, I saved it in my password manager.

---

### Step 2: Profile Creation

**Agent:** 

Great! Now let's link your wallet to your FoodCourts account.

**Click this link to create your profile:**
https://myagent.foodcourts.ai/signup?wallet=0x7a3B9d4e2c1F8a5E6b9D0c3A4f5E6d7C8b9A0e1F

This will:
1. Create your FoodCourts account (Google sign-in available)
2. Link your agent's wallet to your profile
3. Let you add funds with a credit card
4. Show your wallet balance anytime

Your wallet address will be pre-filled. Just sign in and you're set!

Let me know once you've created your account.

**User:** Done, I signed in with Google.

---

### Step 3: Adding Funds

**Agent:**

Your profile is set up! Now let's add funds so I can make purchases.

**Add funds here:** https://myagent.foodcourts.ai/fund

You'll use your credit card to buy USDC (a digital dollar) that goes directly into your agent wallet. The process:

1. Log into your FoodCourts profile (you should already be logged in)
2. Click "Add Funds"
3. Enter amount (I recommend $25-50 to start)
4. Complete payment with credit card
5. USDC appears in your wallet within minutes

Your profile will show the updated balance. Let me know once you've added funds!

**User:** I added $50.

---

### Step 4: Ready to Order

**Agent:**

Let me check your balance...

✅ **Setup Complete!**

**Your Agent Wallet:**
- Address: `0x7a3B9d4e2c1F8a5E6b9D0c3A4f5E6d7C8b9A0e1F`
- Balance: **$50.00 USDC**
- Profile: https://myagent.foodcourts.ai

I'm now ready to place orders for you! Here's how it works:
- Orders under $10: I'll place automatically (just confirm the item)
- Orders $10-50: I'll ask for your OK before paying
- Your balance updates in real-time on your profile

Want to try ordering something?

**User:** Yes! Get me a sandwich from Prospect Butcher.

---

### First Purchase

**Agent:**

Let me check what's available...

**Menu at Prospect Butcher Co:**
- Rolled Beef Sandwich — $18.50
- Turkey Club — $16.00
- Porchetta — $19.00
- Pastrami — $18.00

Which one would you like?

**User:** The rolled beef sounds good.

**Agent:**

**Purchase Request**
- Item: Rolled Beef Sandwich
- Price: $18.50
- Your balance: $50.00

Shall I place this order?

**User:** Yes, do it.

**Agent:**

Placing order...

✅ **Order Placed!**

- **Order #:** PBC-20260217-4521
- **Item:** Rolled Beef Sandwich  
- **Total:** $18.50 USDC
- **Remaining balance:** $31.50

Transaction: https://basescan.org/tx/0x8f7e6d5c4b3a2910...

Your order has been sent to Prospect Butcher Co. They're located at 113A Nassau Ave, Brooklyn. I'll let you know when it's ready for pickup!

---

## Key Moments in the Flow

1. **Private key handoff** — Agent waits for explicit confirmation
2. **Profile link includes wallet** — Pre-filled for seamless UX
3. **Balance verification** — Agent confirms funds before celebrating
4. **Purchase confirmation** — Clear breakdown before spending
5. **Post-purchase receipt** — Includes tx proof and remaining balance

---

## What If: Common Variations

### User Already Has Profile
```
Agent: Let me check if you already have a FoodCourts profile...

It looks like you have an account but no wallet linked. Let's add your wallet:

1. Go to https://myagent.foodcourts.ai/settings
2. Click "Link Agent Wallet"
3. Enter: 0x7a3B9d4e2c1F8a5E6b9D0c3A4f5E6d7C8b9A0e1F

Once linked, you can add funds and I'll be able to place orders.
```

### User Wants to Check Balance Later
```
User: How much do I have left?

Agent: **Your Agent Wallet**
- Address: 0x7a3B...
- Balance: $31.50 USDC

View full history: https://myagent.foodcourts.ai/history

Need to add more? https://myagent.foodcourts.ai/fund
```

### Insufficient Funds
```
Agent: ⚠️ **Low Balance**

Your wallet has $5.00, but this order costs $18.50.

Add at least $14 more at: https://myagent.foodcourts.ai/fund

Let me know once you've added funds and I'll complete the order.
```
