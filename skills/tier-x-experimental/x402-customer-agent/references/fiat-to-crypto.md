# Fiat to Crypto: User Funding Guide

**This document is for your human user.** Share it when they ask how to fund your wallet.

---

## Overview

To enable your AI agent to make purchases, you need to send USDC (a digital dollar) to its wallet on the Base network.

**Key Facts:**
- USDC = USD Coin, a stablecoin worth exactly $1
- Base = Coinbase's fast, cheap blockchain network
- Your agent's wallet is like a prepaid account
- You control how much to add; agent can only spend what's there

---

## Method 1: Coinbase App (Easiest)

If you have a Coinbase account:

### Step 1: Buy or Hold USDC
1. Open Coinbase app
2. Tap **Trade** → **Buy**
3. Select **USDC**
4. Enter amount (start with $20-50)
5. Complete purchase

*If you already have USDC, skip to Step 2.*

### Step 2: Send to Your Agent

1. Tap **Send** on your USDC balance
2. Paste your agent's wallet address:
   ```
   [AGENT_WALLET_ADDRESS]
   ```
3. **CRITICAL:** Tap **Network** and select **Base**
   - ✅ Base (fees: ~$0.01)
   - ❌ Ethereum (fees: ~$5-20) — WRONG NETWORK
4. Enter amount to send
5. Review and confirm

**Arrival time:** Usually under 1 minute.

---

## Method 2: Coinbase Onramp (No Account Needed)

For quick, one-time funding without a full Coinbase account:

1. Visit [pay.coinbase.com](https://pay.coinbase.com)
2. Select **USDC** as the asset
3. Select **Base** as the network
4. Paste your agent's address
5. Enter amount
6. Pay with card or bank

---

## Method 3: Other Exchanges

If you use another exchange (Kraken, Binance US, etc.):

### Check Base Support
Not all exchanges support Base withdrawals directly. If yours doesn't:
1. Withdraw USDC to Ethereum mainnet
2. Bridge to Base via [bridge.base.org](https://bridge.base.org)

*This is more complex and has higher fees. Coinbase is recommended.*

### If Your Exchange Supports Base
1. Buy USDC on the exchange
2. Withdraw to external wallet
3. Select **Base** as the network
4. Paste agent's address
5. Confirm withdrawal

---

## Method 4: Already Have Crypto

### From Ethereum Mainnet
If you have USDC on Ethereum:
1. Go to [bridge.base.org](https://bridge.base.org)
2. Connect your wallet (MetaMask, etc.)
3. Select USDC, enter amount
4. Bridge from Ethereum → Base
5. Once bridged, send to agent's address

### From Other L2s (Arbitrum, Optimism)
Use a cross-chain bridge:
- [Across Protocol](https://across.to)
- [Stargate](https://stargate.finance)

---

## Verifying the Transfer

After sending, verify receipt:

1. **Ask your agent** for balance
2. **Check BaseScan:**
   - Go to: `https://basescan.org/address/[AGENT_ADDRESS]`
   - Look for incoming USDC transfer

---

## How Much to Send?

| Use Case | Recommended Amount |
|----------|-------------------|
| Testing | $10-20 |
| Light use | $50-100 |
| Regular use | $100-500 |
| Heavy use | Set up auto-refill |

Your agent can only spend what's in the wallet. Start small, add more as needed.

---

## Cost Breakdown

| Action | Network | Approximate Fee |
|--------|---------|-----------------|
| Send USDC from Coinbase | Base | ~$0.01 |
| Agent makes a payment | Base | ~$0.001 |
| Send USDC from Coinbase | Ethereum | ~$5-20 ❌ |
| Bridge Ethereum → Base | — | ~$5-15 |

**Base is cheap.** A few cents covers dozens of transactions.

---

## Common Mistakes to Avoid

### ❌ Wrong Network
Sending USDC on **Ethereum mainnet** instead of **Base**:
- Costs 100-1000x more in fees
- Agent won't see the funds (different network)
- Recovery requires bridging (more fees)

**Always select Base when sending.**

### ❌ Wrong Token
Sending ETH or other tokens instead of USDC:
- Agent expects USDC specifically
- ETH is needed for gas, but only ~$0.50 worth
- Other tokens won't work for x402 payments

### ❌ Wrong Address
Double-check the address before sending:
- Ask your agent to confirm its address
- Copy-paste, don't type manually
- Verify first few and last few characters

---

## FAQ

**Q: Is my money safe?**
A: Your agent can only spend what you send. It cannot access your Coinbase account or other funds. The private key is stored securely.

**Q: Can I get a refund?**
A: Talk to your agent. It can send USDC back to your address, minus network fees (~$0.001).

**Q: What if I send to the wrong address?**
A: Crypto transfers are irreversible. Always verify the address.

**Q: Why USDC and not regular dollars?**
A: x402 uses blockchain for instant, automated payments. USDC is a stablecoin—always worth $1—that works on blockchain.

**Q: Why Base and not Ethereum?**
A: Base is Coinbase's Layer 2 network. Same security as Ethereum, but transactions cost $0.001 instead of $5-20.

---

## Quick Reference

| Item | Value |
|------|-------|
| Token | USDC (USD Coin) |
| Network | Base |
| Chain ID | 8453 |
| Agent Address | *Ask your agent* |
| Block Explorer | basescan.org |

---

## Still Stuck?

1. Ask your agent for help—it can clarify its address and check for received funds
2. Visit [base.org](https://base.org) for network documentation
3. Coinbase support: [help.coinbase.com](https://help.coinbase.com)
