# Making x402 Payments

Complete implementation guide for handling 402 responses and executing payments.

**Note:** This is a technical reference for agent implementation. Human-facing messaging should NOT expose these details. See SKILL.md for user-facing templates.

---

## The x402 Flow

```
┌─────────────┐     Request      ┌─────────────┐
│    Agent    │ ───────────────► │   Merchant  │
│             │                  │             │
│             │ ◄─────────────── │             │
│             │   402 Payment    │             │
│             │   Required       │             │
│             │                  │             │
│   Parse     │                  │             │
│   Payment   │                  │             │
│   Details   │                  │             │
│             │                  │             │
│             │   USDC Payment   │             │
│             │ ──────────────►  │  (on-chain) │
│             │                  │             │
│             │   Retry with     │             │
│             │   Payment Proof  │             │
│             │ ───────────────► │             │
│             │                  │             │
│             │ ◄─────────────── │             │
└─────────────┘   200 + Resource └─────────────┘
```

---

## Standard 402 Response Format

When a merchant requires payment, they return:

```http
HTTP/1.1 402 Payment Required
Content-Type: application/json

{
  "x402": {
    "version": "1",
    "payTo": "0x1234567890abcdef1234567890abcdef12345678",
    "amount": "5.00",
    "asset": "USDC",
    "chainId": 8453,
    "description": "API access fee",
    "validUntil": "2026-02-14T13:00:00Z",
    "metadata": {
      "orderId": "abc123"
    }
  }
}
```

**Fields:**
- `payTo`: Merchant's wallet address on Base
- `amount`: Price in USDC (human-readable, e.g., "5.00")
- `asset`: Token type (always "USDC" for x402)
- `chainId`: Network (8453 = Base)
- `validUntil`: Payment window expiration
- `metadata`: Merchant-specific data (include in retry if present)

---

## Complete Implementation (JavaScript)

```javascript
import { Wallet, Contract, JsonRpcProvider, parseUnits } from 'ethers';

// Configuration
const BASE_RPC = 'https://mainnet.base.org';
const USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913';
const USDC_ABI = [
  'function transfer(address to, uint256 amount) returns (bool)',
  'function balanceOf(address owner) view returns (uint256)'
];

class X402Client {
  constructor(privateKey) {
    this.provider = new JsonRpcProvider(BASE_RPC);
    this.wallet = new Wallet(privateKey, this.provider);
    this.usdc = new Contract(USDC_ADDRESS, USDC_ABI, this.wallet);
  }

  /**
   * Get current balance in dollars
   */
  async getBalance() {
    const balance = await this.usdc.balanceOf(this.wallet.address);
    return parseFloat(balance.toString()) / 1e6;
  }

  /**
   * Send payment
   */
  async sendPayment(toAddress, amountUSD) {
    const amountWei = parseUnits(amountUSD.toString(), 6);
    
    // Check balance first
    const balance = await this.usdc.balanceOf(this.wallet.address);
    if (balance < amountWei) {
      throw new Error(`Insufficient funds. Have: ${balance / 1e6}, need: ${amountUSD}`);
    }

    const tx = await this.usdc.transfer(toAddress, amountWei);
    const receipt = await tx.wait();
    
    return receipt.hash;
  }

  /**
   * Make x402-aware fetch request
   */
  async fetch(url, options = {}) {
    // First request
    let response = await fetch(url, options);
    
    // Not a 402? Return as-is
    if (response.status !== 402) {
      return response;
    }

    // Parse payment requirements
    const body = await response.json();
    const payment = body.x402;
    
    if (!payment) {
      throw new Error('402 received but no x402 payment details');
    }

    // Validate payment request
    this.validatePayment(payment);
    
    // Execute payment
    const txHash = await this.sendPayment(payment.payTo, payment.amount);

    // Wait for confirmation (~2 seconds on Base)
    await this.waitForConfirmation(txHash);

    // Retry with payment proof
    const retryOptions = {
      ...options,
      headers: {
        ...options.headers,
        'X-Payment': txHash,
        'X-Payment-Chain': '8453'
      }
    };

    // Include metadata if present
    if (payment.metadata?.orderId) {
      retryOptions.headers['X-Payment-OrderId'] = payment.metadata.orderId;
    }

    return fetch(url, retryOptions);
  }

  /**
   * Validate payment request
   */
  validatePayment(payment) {
    // Must be USDC
    if (payment.asset !== 'USDC') {
      throw new Error(`Unsupported asset: ${payment.asset}`);
    }

    // Must be on Base
    if (payment.chainId !== 8453) {
      throw new Error(`Unsupported chain: ${payment.chainId}`);
    }

    // Check expiration
    if (payment.validUntil) {
      const expiry = new Date(payment.validUntil);
      if (expiry < new Date()) {
        throw new Error('Payment request expired');
      }
    }

    // Sanity check amount
    const amount = parseFloat(payment.amount);
    if (isNaN(amount) || amount <= 0 || amount > 10000) {
      throw new Error(`Suspicious amount: ${payment.amount}`);
    }
  }

  /**
   * Wait for transaction confirmation
   */
  async waitForConfirmation(txHash, maxWaitMs = 30000) {
    const startTime = Date.now();
    
    while (Date.now() - startTime < maxWaitMs) {
      const receipt = await this.provider.getTransactionReceipt(txHash);
      if (receipt && receipt.status === 1) {
        return receipt;
      }
      await new Promise(r => setTimeout(r, 1000));
    }
    
    throw new Error(`Transaction not confirmed within ${maxWaitMs}ms`);
  }
}

// Usage
const client = new X402Client(process.env.AGENT_PRIVATE_KEY);

const response = await client.fetch('https://api.merchant.com/paid-endpoint', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ query: 'something' })
});

const data = await response.json();
console.log(data);
```

---

## Python Implementation

```python
import os
import time
import requests
from web3 import Web3

BASE_RPC = 'https://mainnet.base.org'
USDC_ADDRESS = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913'
USDC_ABI = [
    {
        "name": "transfer",
        "type": "function",
        "inputs": [
            {"name": "to", "type": "address"},
            {"name": "amount", "type": "uint256"}
        ],
        "outputs": [{"type": "bool"}]
    },
    {
        "name": "balanceOf",
        "type": "function",
        "inputs": [{"name": "owner", "type": "address"}],
        "outputs": [{"type": "uint256"}]
    }
]


class X402Client:
    def __init__(self, private_key: str):
        self.w3 = Web3(Web3.HTTPProvider(BASE_RPC))
        self.account = self.w3.eth.account.from_key(private_key)
        self.usdc = self.w3.eth.contract(
            address=Web3.to_checksum_address(USDC_ADDRESS),
            abi=USDC_ABI
        )

    def get_balance(self) -> float:
        balance = self.usdc.functions.balanceOf(self.account.address).call()
        return balance / 1e6

    def send_payment(self, to_address: str, amount_usd: float) -> str:
        amount_wei = int(amount_usd * 1e6)
        
        # Build transaction
        tx = self.usdc.functions.transfer(
            Web3.to_checksum_address(to_address),
            amount_wei
        ).build_transaction({
            'from': self.account.address,
            'nonce': self.w3.eth.get_transaction_count(self.account.address),
            'gas': 100000,
            'gasPrice': self.w3.eth.gas_price
        })
        
        # Sign and send
        signed = self.account.sign_transaction(tx)
        tx_hash = self.w3.eth.send_raw_transaction(signed.rawTransaction)
        
        # Wait for confirmation
        receipt = self.w3.eth.wait_for_transaction_receipt(tx_hash, timeout=30)
        
        return receipt.transactionHash.hex()

    def fetch(self, url: str, method: str = 'GET', **kwargs) -> requests.Response:
        response = requests.request(method, url, **kwargs)
        
        if response.status_code != 402:
            return response
        
        # Parse payment requirements
        payment = response.json().get('x402')
        if not payment:
            raise ValueError('402 but no x402 details')
        
        # Execute payment
        tx_hash = self.send_payment(payment['payTo'], float(payment['amount']))
        
        # Retry with proof
        headers = kwargs.get('headers', {})
        headers['X-Payment'] = tx_hash
        headers['X-Payment-Chain'] = '8453'
        kwargs['headers'] = headers
        
        return requests.request(method, url, **kwargs)


# Usage
client = X402Client(os.environ['AGENT_PRIVATE_KEY'])
response = client.fetch('https://api.merchant.com/endpoint', method='POST', json={'query': 'data'})
print(response.json())
```

---

## Using @x402/fetch (Simplest)

If you prefer a drop-in solution:

```bash
npm install @x402/fetch
```

```javascript
import { createX402Fetch } from '@x402/fetch';

const x402fetch = createX402Fetch({
  privateKey: process.env.AGENT_PRIVATE_KEY,
  rpcUrl: 'https://mainnet.base.org'
});

// Now just use it like regular fetch
const response = await x402fetch('https://merchant.com/paid-api');
const data = await response.json();
```

---

## Spending Limits & User Confirmation

Implement spending controls to protect your user:

```javascript
class SpendingLimits {
  constructor(config = {}) {
    this.maxPerTx = config.maxPerTransaction || 50;
    this.dailyMax = config.dailyMax || 200;
    this.autoApproveUnder = config.autoApproveUnder || 5;
    this.todaySpent = 0;
    this.lastReset = new Date().toDateString();
  }

  async checkAndApprove(amount, confirmFn) {
    // Reset daily counter if new day
    const today = new Date().toDateString();
    if (today !== this.lastReset) {
      this.todaySpent = 0;
      this.lastReset = today;
    }

    // Hard limits
    if (amount > this.maxPerTx) {
      throw new Error(`Amount $${amount} exceeds max per transaction ($${this.maxPerTx})`);
    }

    if (this.todaySpent + amount > this.dailyMax) {
      throw new Error(`Would exceed daily limit. Spent today: $${this.todaySpent}, limit: $${this.dailyMax}`);
    }

    // Auto-approve small amounts
    if (amount < this.autoApproveUnder) {
      this.todaySpent += amount;
      return true;
    }

    // Require confirmation for larger amounts
    const approved = await confirmFn(`Approve payment of $${amount}?`);
    if (approved) {
      this.todaySpent += amount;
    }
    return approved;
  }
}

// Usage
const limits = new SpendingLimits({
  maxPerTransaction: 100,
  dailyMax: 500,
  autoApproveUnder: 10
});

// In your x402 flow:
const payment = response402.json().x402;
const approved = await limits.checkAndApprove(
  parseFloat(payment.amount),
  async (msg) => {
    // Your confirmation mechanism (e.g., ask user via chat)
    return await askUserForConfirmation(msg);
  }
);

if (!approved) {
  throw new Error('Payment not approved by user');
}
```

---

## Error Handling

```javascript
async function safeX402Fetch(client, url, options) {
  try {
    return await client.fetch(url, options);
  } catch (error) {
    if (error.message.includes('Insufficient funds')) {
      return {
        error: 'insufficient_funds',
        // Human-facing message:
        userMessage: 'Your wallet needs more funds. Add funds at https://foodcourts.ai/fund',
        balance: await client.getBalance()
      };
    }
    
    if (error.message.includes('not confirmed')) {
      return {
        error: 'tx_timeout',
        userMessage: 'Payment is taking longer than expected. Please wait a moment.'
      };
    }
    
    if (error.message.includes('expired')) {
      // Retry to get fresh payment request
      return safeX402Fetch(client, url, options);
    }
    
    throw error;
  }
}
```

---

## Transaction Logging (Internal)

Log transactions for transparency and debugging:

```javascript
const txLog = [];

function logTransaction(tx) {
  txLog.push({
    timestamp: new Date().toISOString(),
    type: tx.type,
    to: tx.to,
    amount: tx.amount,
    txHash: tx.hash,
    merchant: tx.merchant,
    description: tx.description,
    status: tx.status
  });
  
  // Persist to storage
  saveToStorage('tx_log', txLog);
}

// After successful payment
logTransaction({
  type: 'payment',
  to: payment.payTo,
  amount: payment.amount,
  hash: txHash,
  merchant: new URL(url).hostname,
  description: payment.description,
  status: 'confirmed'
});
```

---

## Quick Reference (Technical - Agent Only)

| Header | Purpose |
|--------|---------|
| `X-Payment` | Transaction hash proving payment |
| `X-Payment-Chain` | Chain ID (8453 for Base) |
| `X-Payment-OrderId` | Merchant's order ID if provided |

| Status | Meaning |
|--------|---------|
| 402 | Payment required |
| 200 | Success (after valid payment) |
| 400 | Payment invalid or already used |
| 410 | Payment request expired |

| Network Details | Value |
|-----------------|-------|
| Network | Base |
| Chain ID | 8453 |
| Token | USDC |
| USDC Contract | 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 |
| USDC Decimals | 6 |

---

## Next Steps

- See `examples/order-sandwich.md` for a complete working example
- See `references/funding-wallet.md` for user funding instructions
