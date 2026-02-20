# Complete Flow Examples

## Scenario 1: New User - First Order

```
USER: I'm hungry, can you order me lunch?

AGENT: Great! I can order from Prospect Butcher Co for you — they have 
amazing sandwiches in Brooklyn.

Before I can place orders, we'll need to do a quick one-time setup (~2 min):
1. Create an agentic wallet for you
2. Link it to a FoodCourts account
3. Add funds for your order

**Do you already have a FoodCourts account?**

USER: No, I'm new

AGENT: Perfect! Let me set you up.

First, let me show you the menu so you know what you're getting into:

📋 **Prospect Butcher Co Menu**

🥪 SANDWICHES
• The Pitmaster — $16.50
  Smoked brisket, pickled onions, horseradish cream, on house bread

• Italian Hero — $15.00
  Capicola, mortadella, provolone, olive spread, peppers

• Roast Beef — $14.50
  House-roasted beef, sharp cheddar, horseradish, arugula

What looks good?

USER: I want the Pitmaster.

AGENT: Perfect choice! Let's get you set up.

🔐 **AGENTIC WALLET CREATED**

I've created a payment wallet for you. This is where your funds will live.

📍 **Wallet Address:**
`0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a`

🔑 **Your Private Key:**
`0x4c5d6e7f8a9b0c1d2e3f4a5b6c7d8e9f0a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS PRIVATE KEY NOW  ⚠️

• This is the ONLY time you'll see this
• FoodCourts CANNOT recover it — we don't store private keys
• If you lose it, any funds in this wallet are GONE FOREVER
• Save it in a password manager or write it down safely
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ **Your wallet has been created. You can verify it exists here:**
https://basescan.org/address/0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a

**Please confirm you've saved this before we continue.**
Type "saved" to proceed.

USER: saved

AGENT: ✅ Key secured!

Now let's create your FoodCourts account and link your wallet.

👉 **Click here to sign up:**
https://foodcourts.ai/signup?wallet=0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a

Sign in with Google or create an account with email.
Your wallet will be automatically linked!

Come back here once you're done!

USER: Done, I created my account

AGENT: 🎉 Account linked!

Now let's add funds for your order.

💳 **Add funds to your wallet:**
https://foodcourts.ai/fund?wallet=0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a

**Order Summary:**
• The Pitmaster — $16.50
• Tax — $1.47
• **Total: $17.97**

I'd suggest adding $20 to cover this order with a small buffer.
Let me know when you've added funds!

USER: Added $20

AGENT: Checking balance... ✅ $20.00 confirmed!

Placing your order now...

✅ **Order Placed!**

🧾 Order #PBC-4821
📍 Pickup at: 665 Vanderbilt Ave, Prospect Heights
⏰ Ready in: ~12 minutes

**Items:**
• The Pitmaster — $16.50
• Tax — $1.47

💰 Total: $17.97
💳 Remaining balance: $2.03

Your order has been sent to the kitchen! 🔥
```

---

## Scenario 2: Existing User - Has FoodCourts Account

```
USER: I want to order a sandwich

AGENT: Great! I can order from Prospect Butcher Co for you.

**Do you already have a FoodCourts account?**

USER: Yes, I have one

AGENT: Perfect! I just need to link an agentic wallet to your account.

What's your Profile ID? You can find it at https://foodcourts.ai/profile

USER: It's PROF-ABC123

AGENT: Got it! Creating your agentic wallet now...

🔐 **AGENTIC WALLET CREATED**

📍 **Wallet Address:**
`0x9b8C7d6E5f4A3B2c1D0e9F8a7B6c5D4e3F2a1B0c`

🔑 **Your Private Key:**
`0x1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3`

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  SAVE THIS PRIVATE KEY NOW  ⚠️
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ **Your wallet has been created. You can verify it exists here:**
https://basescan.org/address/0x9b8C7d6E5f4A3B2c1D0e9F8a7B6c5D4e3F2a1B0c

Type "saved" when you've backed up your key.

USER: saved

AGENT: ✅ Key secured!

Now let's link this wallet to your FoodCourts account.

👉 **Click here to link your wallet:**
https://foodcourts.ai/profile/link-wallet?wallet=0x9b8C7d6E5f4A3B2c1D0e9F8a7B6c5D4e3F2a1B0c

Make sure you're logged in, then click "Confirm" to link.

Let me know when you're done.

USER: Done

AGENT: ✅ Wallet linked!

Now let's add funds so you can order.

💳 **Add funds at:** https://foodcourts.ai/fund?wallet=0x9b8C7d6E5f4A3B2c1D0e9F8a7B6c5D4e3F2a1B0c

What would you like to order? Here's the menu:

🥪 SANDWICHES
• The Pitmaster — $16.50
• Italian Hero — $15.00
• Roast Beef — $14.50
• Veggie — $13.00

USER: Italian Hero

AGENT: Great choice!

**Order Summary:**
• Italian Hero — $15.00
• Tax — $1.33
• **Total: $16.33**

Have you added funds yet?

USER: Yes, I added $25

AGENT: Checking... ✅ Balance: $25.00

Placing your order...

✅ **Order Placed!**

🧾 Order #PBC-4856
📍 Pickup at: 665 Vanderbilt Ave, Prospect Heights
⏰ Ready in: ~15 minutes

**Items:**
• Italian Hero — $15.00
• Tax — $1.33

💰 Total: $16.33
💳 Remaining balance: $8.67

Enjoy your sandwich! 🥪
```

---

## Scenario 3: Returning User - Quick Order

```
USER: Hey, I want to order lunch again

AGENT: 👋 Welcome back!

📍 Your wallet: `0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a`
💰 Balance: $2.03

What would you like to order today?

USER: Same thing, the Pitmaster

AGENT: Got it — The Pitmaster ($16.50 + $1.47 tax = $17.97)

⚠️ **Low Balance**

Your wallet has $2.03, but this order is $17.97.

👉 Add $16 more at: https://foodcourts.ai/fund?wallet=0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a

Let me know when done!

USER: Added $20

AGENT: Checking... ✅ Balance now $22.03

Placing your order...

✅ **Order Placed!**

🧾 Order #PBC-4847
📍 Pickup at: 665 Vanderbilt Ave, Prospect Heights
⏰ Ready in: ~10 minutes

**Items:**
• The Pitmaster — $16.50
• Tax — $1.47

💰 Total: $17.97
💳 Remaining balance: $4.06

Enjoy your lunch! 🥪
```

---

## Scenario 4: Restaurant Closed

```
USER: Order me a sandwich

AGENT: 👋 Welcome back!

📍 Your wallet: `0x7a3F9b...`
💰 Balance: $15.00

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

## Scenario 5: User Wants Refund

```
USER: I want my money back from the wallet

AGENT: No problem! I can send your remaining balance back to you.

📍 Your wallet balance: $22.03

What address should I send the funds to? 

USER: Send it to my main wallet at 0x1234...5678

AGENT: 📤 **Return Funds Request**

Sending: $22.02 (balance minus ~$0.01 network fee)
To: 0x1234...5678

Type "yes" to confirm.

USER: yes

AGENT: ✅ **Funds Sent!**

$22.02 has been sent to your wallet.
Your FoodCourts wallet balance is now $0.00.

You can add funds again anytime at https://foodcourts.ai/fund?wallet=0x7a3F9b2C4d5E6f8A1b3C4D5e6F7a8B9c0D1e2F3a
```
