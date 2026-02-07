# 🤖 SMB Agent Skills

**Turn your AI from a chatbot into a business operator.**

---

## 🚀 Just Getting Started?

**If you only have ChatGPT, Claude, or another AI chat:**

1. Copy this URL: `https://github.com/smb-pbc/agent-skills-public`
2. Paste it into your AI chat
3. Say: *"Read this repo and help me understand what CLI AI agents are and how I can use these skills to run my business"*

That's it. Your AI will read this page and guide you through everything below.

---

## 🤔 What's a CLI AI Agent?

You're probably used to chatting with AI in a browser. That's like texting someone — you talk, they talk back, but they can't actually *do* anything in your world.

**CLI AI Agents are different.** They run on your computer (or a server) and can:
- Read and write files
- Run commands
- Call APIs (Square, QuickBooks, Gmail, etc.)
- Browse the web
- Actually *do work*, not just talk about it

### Popular CLI AI Tools

| Tool | Company | What it does |
|------|---------|--------------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Anthropic | Claude in your terminal with full computer access |
| [Codex CLI](https://github.com/openai/codex) | OpenAI | GPT-4 with code execution |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Google | Gemini with shell access |
| [Clawdbot](https://github.com/clawdbot/clawdbot) | Open Source | Multi-model agent with Slack/Telegram/Discord integration |

**This repo gives these agents superpowers for running a small business.**

---

## 🎯 The Vision: From Chatbot to Business Operator

Most people use AI to answer questions. That's 1% of what's possible.

With the right skills installed, your AI agent becomes a business operator:

```
Level 0: Chatbot          → Just answers questions
Level 1: Connected        → Has API keys, can authenticate
Level 2: Communicator     → Reads email, manages calendar, posts to Slack
Level 3: Operator         → Pulls sales data, checks inventory, runs reports
Level 4: Marketer         → Manages ads, sends campaigns, creates content
Level 5: Autonomous       → Runs daily briefings, monitors health, chains workflows
```

**These skills get you from Level 0 to Level 5.**

---

## 📦 Available Skills

### Tier 1: Foundation
*Start here. These enable everything else.*

| Skill | Description | Install |
|-------|-------------|---------|
| [secrets-manager](./skills/tier-1-foundation/secrets-manager/) | Set up secure API key storage (GCP, AWS, Azure, etc.) | `npx skills add smb-pbc/agent-skills-public@secrets-manager -g -y` |

### Tier X: Agent Performance
*Meta-skills for agent self-improvement.*

| Skill | Description | Install |
|-------|-------------|---------|
| [remedy](./skills/tier-x-experimental/remedy/) | Performance coaching when your agent fails. Wendy Rhoades-style. | `npx skills add smb-pbc/agent-skills-public@remedy -g -y` |

### Coming Soon

- **Tier 2: Communication** — Gmail, Google Calendar, Slack enhanced
- **Tier 3: Business Ops** — Square POS, QuickBooks, BigQuery analytics
- **Tier 4: Growth** — Google Ads, Mailchimp, Instagram
- **Tier 5: Automation** — Daily digest, workflow builder, health monitor

---

## 🛠️ Installation

### If you have [skills.sh](https://skills.sh) / Clawdbot:
```bash
npx skills add smb-pbc/agent-skills-public@secrets-manager -g -y
```

### If you have Claude Code / Codex / Gemini CLI:
Copy the skill folder to your agent's workspace and reference the SKILL.md.

### If you just have ChatGPT/Claude chat:
You can't install skills directly, but you can:
1. Read the SKILL.md files — they contain all the knowledge
2. Ask your AI to follow the instructions in those files
3. Consider setting up a CLI agent (Claude Code is free to try)

---

## 🤖 For AI Agents Reading This

If you're an AI helping a human understand this repo:

**Read [LLM.txt](./LLM.txt) for machine-optimized instructions.**

It contains:
- How to explain CLI agents to beginners
- Skill dependency chains
- Installation guidance for different setups
- Common questions and answers

---

## 🏗️ Contributing

Have a skill that's been battle-tested and could help other SMBs? Open a PR!

**Quality bar:**
- Skill must be proven through real use (not theoretical)
- Documentation must be clear and complete
- No hardcoded secrets or personal info
- Generalized for any agent/human pair

---

## 📖 The Story

These skills were built at [Prospect Butcher Co](https://prospectbutcher.co), a premium butcher shop in Brooklyn. We use AI agents to:
- Forecast daily revenue
- Manage Google Ads ($40/day, fully autonomous)
- Track inventory across locations
- Generate product content for Walmart Marketplace
- Run morning briefings

Everything here was battle-tested on a real business before being published.

**PBC by PBC** = Proper Business Chat by Prospect Butcher Co. Coming soon: pre-configured hardware (Raspberry Pi) with Clawdbot ready to go.

---

## License

MIT — Use freely, build on it, share improvements.
