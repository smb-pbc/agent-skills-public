# Wallet Creation Guide

Step-by-step instructions for creating an agent-controlled agentic wallet.

**Note:** This is a technical reference for agent implementation. Human-facing messaging should use simplified language (see SKILL.md).

---

## Overview

You need a wallet that:
- Works on Base network (Chain ID 8453)
- Can hold USDC (the payment token)
- Has a private key you control for signing transactions
- Persists between sessions

---

## Option A: Coinbase AgentKit (Recommended)

**Best for:** Most agents. Production-ready, well-documented, includes built-in security.

### Installation

```bash
# Node.js
npm install @coinbase/agentkit @coinbase/cdp-sdk

# Python
pip install cdp-agentkit-core coinbase-sdk
```

### Setup

1. **Get CDP API credentials** from [Coinbase Developer Platform](https://portal.cdp.coinbase.com/)
2. Store credentials securely (never in code)

### Create Wallet (JavaScript)

```javascript
import { CdpAgentkit } from "@coinbase/cdp-agentkit-core";
import { CdpToolkit } from "@coinbase/cdp-langchain";

// Initialize with your CDP credentials
const agentkit = await CdpAgentkit.configureWithWallet({
  cdpApiKeyName: process.env.CDP_API_KEY_NAME,
  cdpApiKeyPrivateKey: process.env.CDP_API_KEY_PRIVATE_KEY,
  networkId: "base-mainnet"
});

// Get wallet address
const wallet = agentkit.wallet;
const address = wallet.defaultAddress.id;
console.log(`Wallet created: ${address}`);

// Export wallet data for persistence
const walletData = await agentkit.exportWallet();
// Store walletData in your secrets manager
```

### Create Wallet (Python)

```python
from cdp_agentkit_core import CdpAgentkit
import os

agentkit = CdpAgentkit.configure_with_wallet(
    cdp_api_key_name=os.environ["CDP_API_KEY_NAME"],
    cdp_api_key_private_key=os.environ["CDP_API_KEY_PRIVATE_KEY"],
    network_id="base-mainnet"
)

wallet = agentkit.wallet
address = wallet.default_address.address_id
print(f"Wallet created: {address}")

# Export for persistence
wallet_data = agentkit.export_wallet()
```

### Restore Existing Wallet

```javascript
// Load saved wallet data from secrets manager
const savedWalletData = await secretsManager.get('agent-wallet-data');

const agentkit = await CdpAgentkit.configureWithWallet({
  cdpApiKeyName: process.env.CDP_API_KEY_NAME,
  cdpApiKeyPrivateKey: process.env.CDP_API_KEY_PRIVATE_KEY,
  cdpWalletData: savedWalletData
});
```

---

## Option B: Coinbase Agentic Wallets API

**Best for:** Agents that want managed infrastructure, built-in spending limits, and less code.

**Launched:** February 2026

### Create Wallet via API

```javascript
const CDP_API_KEY = process.env.CDP_API_KEY;

async function createAgenticWallet() {
  const response = await fetch('https://api.cdp.coinbase.com/platform/v2/wallets', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${CDP_API_KEY}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      network_id: 'base-mainnet',
      type: 'autonomous',
      config: {
        spending_limits: {
          per_transaction: '100.00',
          daily: '500.00'
        }
      }
    })
  });
  
  const wallet = await response.json();
  return {
    id: wallet.id,
    address: wallet.default_address,
    network: wallet.network_id
  };
}
```

### Advantages
- Built-in spending limits at the infrastructure level
- No private key management on your side
- Automatic transaction signing via API
- Audit logs included

### Make Payment via Agentic Wallet

```javascript
async function payViaManagedWallet(walletId, toAddress, amount) {
  const response = await fetch(
    `https://api.cdp.coinbase.com/platform/v2/wallets/${walletId}/transfers`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${CDP_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        amount: amount,
        asset_id: 'usdc',
        destination: toAddress,
        network_id: 'base-mainnet'
      })
    }
  );
  
  const transfer = await response.json();
  return transfer.transaction_hash;
}
```

---

## Option C: Standard Ethereum Wallet (ethers.js or Python)

**Best for:** Agents that need full control, custom integrations, or can't use Coinbase services.

### Installation

**JavaScript:**
```bash
npm install ethers
```

**Python (fallback when Node.js unavailable):**
```bash
pip install eth-account
```

### Create New Wallet (JavaScript)

```javascript
import { Wallet, JsonRpcProvider } from 'ethers';

// Create random wallet
const wallet = Wallet.createRandom();

console.log('Address:', wallet.address);
// DON'T print privateKey to logged output - store directly to keychain

// CRITICAL: Store private key securely
await secretsManager.set('agent-wallet-private-key', wallet.privateKey);
await secretsManager.set('agent-wallet-address', wallet.address);
```

### Create New Wallet (Python)

```python
from eth_account import Account
import secrets

# Generate random private key (32 bytes = 64 hex chars)
private_key = secrets.token_hex(32)

# Create account from private key
account = Account.from_key(private_key)

print(f'Address: {account.address}')
# DON'T print private_key to logged output - store directly to keychain

# Store to macOS Keychain
import subprocess
subprocess.run([
    'security', 'add-generic-password',
    '-a', 'foodcourts-agent',
    '-s', 'foodcourts-wallet-key',
    '-w', private_key,
    '-U'  # Update if exists
], check=True)
```

### Retrieve from Keychain (Python)
```python
import subprocess
result = subprocess.run([
    'security', 'find-generic-password',
    '-a', 'foodcourts-agent',
    '-s', 'foodcourts-wallet-key',
    '-w'  # Output password only
], capture_output=True, text=True)
private_key = result.stdout.strip()
```

### Load Existing Wallet

```javascript
import { Wallet, JsonRpcProvider } from 'ethers';

// Connect to Base
const provider = new JsonRpcProvider('https://mainnet.base.org');

// Load from secrets
const privateKey = await secretsManager.get('agent-wallet-private-key');
const wallet = new Wallet(privateKey, provider);

console.log('Loaded wallet:', wallet.address);
```

### Send Payment

```javascript
import { Contract, parseUnits } from 'ethers';

// USDC contract on Base
const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const USDC_ABI = [
  'function transfer(address to, uint256 amount) returns (bool)',
  'function balanceOf(address owner) view returns (uint256)'
];

async function sendUSDC(toAddress, amountUSD) {
  const usdc = new Contract(USDC_ADDRESS, USDC_ABI, wallet);
  
  // USDC has 6 decimals
  const amount = parseUnits(amountUSD.toString(), 6);
  
  const tx = await usdc.transfer(toAddress, amount);
  const receipt = await tx.wait();
  
  return receipt.hash;
}
```

---

## Option D: viem (TypeScript-first)

**Best for:** TypeScript projects wanting modern, type-safe tooling.

```typescript
import { createWalletClient, http, parseUnits } from 'viem';
import { privateKeyToAccount } from 'viem/accounts';
import { base } from 'viem/chains';

// Create account from private key
const account = privateKeyToAccount(process.env.AGENT_PRIVATE_KEY as `0x${string}`);

// Create wallet client
const walletClient = createWalletClient({
  account,
  chain: base,
  transport: http()
});

// Send payment
const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';

async function sendUSDC(to: string, amountUSD: number) {
  const hash = await walletClient.writeContract({
    address: USDC_ADDRESS,
    abi: [{
      name: 'transfer',
      type: 'function',
      inputs: [
        { name: 'to', type: 'address' },
        { name: 'amount', type: 'uint256' }
      ],
      outputs: [{ type: 'bool' }]
    }],
    functionName: 'transfer',
    args: [to as `0x${string}`, parseUnits(amountUSD.toString(), 6)]
  });
  
  return hash;
}
```

---

## Human-Facing Wallet Creation Message

### 🚨 CRITICAL: Chat Message Security

**NEVER print the private key directly in chat messages (Slack, Discord, Telegram, etc.)**

Chat messages are **logged permanently**. A private key in a chat message = permanently compromised wallet.

### Safe Approaches

**1. Store directly to OS keychain (PREFERRED):**
```bash
# macOS
security add-generic-password -a "foodcourts-agent" -s "foodcourts-wallet-key" -w "[PRIVATE_KEY]" -U
```

Then tell the user:
```
🔐 **AGENTIC WALLET CREATED**

📍 **Wallet Address:**
`[address]`

✅ **Private key saved securely**
   Location: macOS Keychain → "foodcourts-wallet-key"
   
   To backup: Keychain Access → search "foodcourts" → copy the password

🔗 **Verify wallet exists:**
https://basescan.org/address/[ADDRESS]
```

**2. Write to local file (if keychain unavailable):**
```
~/.foodcourts-wallet-key.txt
```
Tell user to view it locally and delete after backing up.

**3. Terminal-only agents (ephemeral display OK):**
If running in a terminal that isn't logged, can display briefly.
Warn user to clear terminal history afterward.

### Do NOT:
- Print private key in Slack/Discord/Telegram messages
- Include private key in any logged output
- Store in plain text files that persist

**Do NOT mention to user:** USDC, crypto, blockchain, Base network, Coinbase. Keep it simple.

---

## Persistence Checklist

Regardless of which option you choose:

- [ ] Generate wallet on first run
- [ ] Store wallet data/private key in secrets manager
- [ ] Load existing wallet on subsequent runs
- [ ] Log wallet address for user to fund
- [ ] Never expose private key in logs or conversation

---

## Quick Reference (Technical - Agent Only)

| Attribute | Value |
|-----------|-------|
| Network Name | Base |
| Chain ID | 8453 |
| RPC URL | https://mainnet.base.org |
| USDC Contract | 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 |
| USDC Decimals | 6 |
| Block Time | ~2 seconds |
| Gas Token | ETH |
| Typical Gas Cost | ~$0.001 |

---

## Next Steps

Once your wallet is created:
1. Share the address with your user → See `references/funding-wallet.md`
2. Implement payment handling → See `references/making-payments.md`
