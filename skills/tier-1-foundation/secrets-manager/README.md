# 🔐 Secrets Manager

**Give your AI agent secure access to all your business tools.**

This skill guides you through setting up a secure secrets vault so your agent can authenticate to Gmail, Square, QuickBooks, Google Ads, and any other API — without storing keys in plain text.

---

## Why This Matters

Without secure secrets management, your AI agent is just a chatbot. It can answer questions, but it can't *do* anything.

```
WITHOUT SECRETS:                     WITH SECRETS:
                                    
You ←→ Chat ←→ AI                   You ←→ AI ←→ Gmail
(just talking)                              ├→ Square
                                            ├→ Google Ads
                                            ├→ QuickBooks
                                            └→ Everything else
```

Every integration you'll ever add starts with secure API key storage. **This is step zero.**

---

## What You'll Set Up

1. **Choose a vault** — GCP, AWS, Azure, 1Password, Doppler, or HashiCorp Vault
2. **Store your first secret** — guided walkthrough
3. **Wire it to your agent** — so keys load at startup, never on disk
4. **Verify it works** — with included test scripts

---

## Supported Platforms

| Platform | Best For | Cost |
|----------|----------|------|
| **GCP Secret Manager** | Google users (Gmail, Ads, BigQuery) | 6 secrets free |
| **AWS Secrets Manager** | AWS shops | $0.40/secret/month |
| **Azure Key Vault** | Microsoft ecosystem | Pay-per-operation |
| **1Password Connect** | Teams on 1Password already | Business plan |
| **Doppler** | Developer-friendly | Free tier |
| **HashiCorp Vault** | Self-hosted, max control | Free (DIY) |

Not sure? The skill will help you choose.

---

## Installation

```bash
npx skills add smb-pbc/agent-skills-public@secrets-manager -g -y
```

Or copy this folder to your agent's skills directory.

---

## What's Included

```
secrets-manager/
├── SKILL.md              # Main guide (your agent follows this)
├── README.md             # This file
├── references/           # Deep-dive docs for each platform
│   ├── gcp-secret-manager.md
│   ├── aws-secrets-manager.md
│   ├── azure-key-vault.md
│   ├── 1password-connect.md
│   ├── doppler.md
│   ├── hashicorp-vault.md
│   └── platform-comparison.md
└── scripts/
    ├── verify_access.sh  # Check if everything's wired up
    └── test_secret.sh    # Create/read/delete a test secret
```

---

## After Setup

Once secrets are working, you can:
- Add API keys for any service (just store + wire + restart)
- Install Tier 2+ skills that need authentication
- Your agent becomes an *operator*, not just a chatbot

**Next recommended skill:** Gmail integration (Tier 2)

---

## Security Best Practices

- ✅ **Always** use the vault as single source of truth
- ✅ **Rotate** secrets quarterly
- ❌ **Never** store keys in .env files, code, or configs
- ❌ **Never** commit secrets to git

---

*Part of [SMB Agent Skills](https://github.com/smb-pbc/agent-skills-public) — open-source skills for AI-powered businesses.*
