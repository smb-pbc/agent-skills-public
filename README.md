# 🤖 SMB Agent Skills

**Give your AI agent superpowers for running a small business.**

This is a collection of open-source skills that turn an AI agent from a chatbot into a business operator. Built and battle-tested by [SMB PBC](https://github.com/smb-pbc) — a team running real brick-and-mortar businesses with AI agents.

---

## 🎯 The Vision

Most people think of AI as something you chat with. That's 1% of what's possible.

An AI agent with the right skills can:
- Read your emails and surface what matters
- Check your sales while you sleep
- Manage your ad spend and cut waste
- Track inventory and alert you before stockouts
- Generate reports without you touching a spreadsheet

**The difference between a chatbot and an AI operator is skills.**

This repo is the playbook for building that operator, one skill at a time.

---

## 📊 The SMB Agent Maturity Model

We organize skills into **tiers** — each tier unlocks more capability:

| Tier | Name | What It Unlocks |
|------|------|-----------------|
| **0** | Agent Core | Skills that make your agent smarter (coaching, memory, self-improvement) |
| **1** | Foundation | Security, authentication, the plumbing that makes everything else work |
| **2** | Communication | Email, calendar, Slack — how your agent talks to the world |
| **3** | Business Ops | POS, accounting, inventory — real business data at your fingertips |
| **4** | Growth | Ads, marketing, social media — grow your business on autopilot |
| **5** | Automation | Workflows that chain everything together — your agent runs itself |

**Start at Tier 0 or 1. Build up from there.**

Most businesses stall at "chatbot" because they skip the foundation. Don't skip the foundation.

---

## 📁 What's Here

### Tier 0: Agent Core
*Skills that make your agent better at being an agent.*

| Skill | Description | Status |
|-------|-------------|--------|
| [remedy](./skills/tier-0-agent-core/remedy/) | Performance coaching when your agent fails. Wendy Rhoades-style coaching to identify root causes, fix systems, and rebuild confidence. | ✅ Ready |

### Tier 1: Foundation
*The plumbing. Set this up first.*

| Skill | Description | Status |
|-------|-------------|--------|
| [secrets-manager](./skills/tier-1-foundation/secrets-manager/) | Store API keys securely. Guides you through GCP, AWS, Azure, 1Password, Doppler, or Vault setup. **Start here.** | ✅ Ready |

### Tier 2: Communication
*Connect your agent to email, calendar, messaging.*

| Skill | Description | Status |
|-------|-------------|--------|
| *Coming soon* | Gmail, Google Calendar, Slack enhancements | 🔜 Planned |

### Tier 3: Business Operations
*Your agent reads real business data.*

| Skill | Description | Status |
|-------|-------------|--------|
| *Coming soon* | Square POS, QuickBooks, BigQuery analytics | 🔜 Planned |

### Tier 4: Growth & Marketing
*Your agent drives growth.*

| Skill | Description | Status |
|-------|-------------|--------|
| *Coming soon* | Google Ads, Mailchimp, social media | 🔜 Planned |

### Tier 5: Advanced Automation
*Your agent runs itself.*

| Skill | Description | Status |
|-------|-------------|--------|
| *Coming soon* | Daily digests, workflow builder, health monitoring | 🔜 Planned |

---

## 🚀 Installation

Using [skills.sh](https://skills.sh):

```bash
# Install a skill globally
npx skills add smb-pbc/agent-skills-public@tier-1-foundation/secrets-manager -g -y

# Or just the skill name if unique
npx skills add smb-pbc/agent-skills-public@secrets-manager -g -y
```

Or copy the skill folder directly to your agent's skills directory.

---

## 🛤️ Recommended Path

**Week 1: Get the Foundation Right**
1. Install `secrets-manager` and set up secure API key storage
2. Get your first API key working (Gmail or Square are good starts)

**Week 2: Connect Communication**
3. Add email integration (read vendor emails, customer inquiries)
4. Add calendar integration (never miss a delivery or meeting)

**Week 3-4: Business Operations**
5. Connect your POS (Square, Shopify, etc.)
6. Connect your accounting (QuickBooks, Xero)
7. Your agent can now answer "How did we do last week?"

**Month 2+: Growth**
8. Connect ad platforms (Google Ads, Meta)
9. Add inventory monitoring
10. Your agent starts finding opportunities and problems before you do

---

## 🤝 Contributing

Have a skill that's been battle-tested and could help other businesses?

**Quality bar:**
- ✅ Proven through real use (not theoretical)
- ✅ Documentation is clear (a non-technical person can follow it)
- ✅ No hardcoded secrets or personal info
- ✅ Works with any agent/human pair

Open a PR and tell us how you've used it.

---

## 💡 Philosophy

1. **Skills > Prompts.** Anyone can write a prompt. Skills are documented, tested, and reusable.

2. **Foundation first.** Flashy integrations fail without solid plumbing (auth, secrets, error handling).

3. **Battle-tested only.** We don't publish theoretical skills. Everything here has been used in real businesses.

4. **Non-technical friendly.** Your agent should guide you through setup, not expect you to know DevOps.

5. **Open source wins.** The SMB AI revolution shouldn't be locked behind enterprise paywalls.

---

## 📜 License

MIT — use it, fork it, improve it, share it.

---

*Built by [SMB PBC](https://github.com/smb-pbc) — small business operators building the future of AI-powered business.*
