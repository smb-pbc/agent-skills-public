# Complete Flow Examples

## Scenario 1: New User - First Order

```
USER: I'm hungry, can you order me lunch?

AGENT: Great! I can order from Prospect Butcher Co for you — they have 
amazing sandwiches and butcher items in Brooklyn.

Before I can place orders, we'll need to do a quick one-time setup (~2 min):
1. Create a payment wallet for you
2. Link it to a FoodCourts account
3. Add funds for your order

Want to see the menu first, or ready to set up?

USER: Show me the menu

AGENT: 📋 **Prospect Butcher Co Menu**

🥪 SANDWICHES
• The Pitmaster — $16.50
  Smoked brisket, pickled onions, horseradish cream, on house bread

• Italian Hero — $15.00
  Capicola, mortadella, provolone, olive spread, peppers

• Roast Beef — $14.50
  House-roasted beef, sharp cheddar, horseradish, arugula

• Veggie — $13.00
  Roasted vegetables, fresh mozzarella, basil pesto

Ready to set up so you can order?

USER: Yeah let's do it. I want the Pitmaster.

AGENT: Perfect choice! Let's get you set up.

🔐 **WALLET CREATED**

I've created a payment wallet for you. This is where your funds will live.

📍 **Wallet Address:**
`0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a`

🔑 **Your Private Key:**
`0x4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS PRIVATE KEY NOW  ⚠️

• This is the ONLY time you'll see this
• FoodCourts CANNOT recover it — we don't store private keys
• If you lose it, any money in this wallet is GONE FOREVER
• Save it in a password manager or write it down safely
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

**Please confirm you've saved this before we continue.**
Type "saved" to proceed.

USER: saved

AGENT: ✅ Private key saved!

Now let's link your wallet to a FoodCourts account. This lets me:
• Check your balance
• Place orders on your behalf  
• Remember you next time

👉 **Click here to create your account:**
https://pbc.foodcourts.ai/auth?wallet=0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a

You can sign in with Google or create an account with email.
Come back here once you're done!

USER: Done, I created my account

AGENT: 🎉 Account linked!

Now let's add funds for your order.

**Order Summary:**
• The Pitmaster — $16.50
• Tax — $1.47
• **Total: $17.97**

Your current balance: $0.00

👉 **Add funds here:** https://pbc.foodcourts.ai/fund

I'd suggest adding $20 to cover this order with a small buffer.
Let me know when you've added funds!

USER: Added $20

AGENT: Checking balance... ✅ $20.00 USDC confirmed!

Placing your order now...

✅ **Order Placed!**

🧾 Order #PBC-4821
📍 Pickup at: 665 Vanderbilt Ave, Prospect Heights
⏰ Ready in: ~12 minutes

**Items:**
• The Pitmaster — $16.50
• Tax — $1.47

💰 Total: $17.97 USDC
💳 Remaining balance: $2.03

🔗 Transaction: https://basescan.org/tx/0x8f7e6d5c4b3a2918...

Your order has been sent to the kitchen! 🔥
```

---

## Scenario 2: Returning User - Quick Order

```
USER: Hey, I want to order lunch again

AGENT: 👋 Welcome back!

📍 Your wallet: `0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a`
💰 Balance: $2.03 USDC

What would you like to order today?

USER: Same thing, the Pitmaster

AGENT: Got it — The Pitmaster ($16.50 + $1.47 tax = $17.97)

⚠️ **Low Balance**

Your wallet has $2.03, but this order is $17.97.

👉 Add $16 more at: https://pbc.foodcourts.ai/fund

Let me know when done!

USER: Added $20

AGENT: Checking... ✅ Balance now $22.03 USDC

Placing your order...

✅ **Order Placed!**

🧾 Order #PBC-4847
📍 Pickup at: 665 Vanderbilt Ave, Prospect Heights
⏰ Ready in: ~10 minutes

**Items:**
• The Pitmaster — $16.50
• Tax — $1.47

💰 Total: $17.97 USDC
💳 Remaining balance: $4.06

Enjoy your lunch! 🥪
```

---

## Scenario 3: Expired OAuth Token

```
USER: Can you order me a sandwich?

AGENT: Let me check your account...

🔐 **Session Expired**

I need you to re-authorize my access to your FoodCourts account.

👉 **Click here to log in:** https://pbc.foodcourts.ai/auth?reauth=true

This confirms you're the owner of your agent wallet.
Once you've logged in, I'll be able to check your balance and place orders.

USER: Done

AGENT: ✅ Re-authenticated!

📍 Your wallet: `0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a`
💰 Balance: $4.06 USDC

What would you like to order?
```

---

## Scenario 4: Restaurant Closed

```
USER: Order me a sandwich

AGENT: 👋 Welcome back!

📍 Your wallet: `0x7a3F9b...`
💰 Balance: $15.00 USDC

Let me check PBC's hours...

😔 **PBC is currently closed**

Hours: 
• Mon-Sat: 11am - 7pm ET
• Sun: 11am - 5pm ET

Current time: 9:15 PM ET (Monday)

They'll open tomorrow at 11am ET. Want me to remind you then?

USER: Yes please

AGENT: ✅ I'll remind you tomorrow at 11am ET when PBC opens!
```

---

## Scenario 5: Agent Needs Private Key for Transaction

*In some implementations, the agent may need the customer to provide the private key to sign transactions:*

```
AGENT: Ready to place your order!

To complete the payment, I need your private key to sign the transaction.

🔐 **Paste your private key:**
(This is used once to sign this transaction, then immediately discarded)

USER: 0x4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6

AGENT: ✅ Transaction signed and sent!

[Order confirmation...]
```

*Alternative: Some agents may store the private key in their local secrets manager with customer consent. The skill supports both patterns.*
