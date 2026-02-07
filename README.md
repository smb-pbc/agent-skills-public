# 🤖 SMB Agent Skills

**Turn your AI from a chatbot into a business operator.**

---

## 🚀 Brand New to This?

**Step 1:** Copy this URL: `https://github.com/smb-pbc/agent-skills-public`

**Step 2:** Paste it into ChatGPT, Claude, or any AI chat

**Step 3:** Say: *"Read this and help me understand what AI agents can do for my small business. I'm completely new to this."*

That's it. The AI will read this page and guide you through everything — explained for your specific type of business.

---

## 💡 What You Might Not Know About AI

You've probably used ChatGPT or Claude. You type, it types back. It's helpful for questions and writing.

**But that's only 1% of what AI can do.**

Right now, your AI is like a really smart person stuck in a room with no phone, no computer, and no access to anything. They can talk to you through a slot in the door, but they can't actually DO anything.

**AI agents are different.** They can:
- Actually read your emails (not just talk about email)
- Actually check your sales numbers (not just suggest you check them)
- Actually pause that ad that's wasting money (not just tell you to pause it)
- Actually send that invoice reminder (not just draft it for you to copy-paste)

**This repo helps you get there.** It's a collection of "skills" — pre-built instructions that teach AI how to connect to and use your business tools.

---

## 🤔 How Does This Work?

Three things need to happen for AI to actually do things in your business:

| Step | What It Means | How Long |
|------|---------------|----------|
| 1. **AI runs somewhere with access** | Install a tool like Claude Code on your computer | 15 min |
| 2. **AI gets credentials** | Store API keys so AI can connect to Gmail, Square, etc. | 30 min |
| 3. **AI learns your tools** | Add "skills" that teach it how to use each service | 5 min each |

**This repo handles Step 3.** Steps 1-2 are one-time setup — the AI reading this can walk you through it.

---

## 🛠️ Tools You'll Need

To use these skills, you'll need one of these "CLI AI agents" (AI that runs on a computer, not just in a browser):

| Tool | Best For | Difficulty |
|------|----------|------------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Beginners, Mac/Linux users | Easy |
| [Clawdbot](https://github.com/clawdbot/clawdbot) | Slack/Discord users, always-on agent | Medium |
| [Codex CLI](https://github.com/openai/codex) | OpenAI/GPT users | Easy |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | Google ecosystem users | Easy |

**Don't have any of these yet?** That's fine. Ask the AI helping you to walk you through setting up Claude Code — it's free to start and takes about 15 minutes.

---

## 🎯 What Your First Week Could Look Like

**Day 1-2: Connect your email**

Once AI can read your inbox, something magical happens — it can actually SEE your business. Not in theory. Actually see what you deal with every day.

**Day 3-5: AI observes**

You don't do anything. AI reads 2 weeks of your emails and learns:
- Who emails you most
- What questions keep coming up
- What's taking your time

**Day 5-7: AI tells YOU what to automate**

Instead of you guessing what would help, AI shows you:

> "You answered 23 emails about delivery times last week. Want me to create an auto-reply?"
>
> "You have 5 vendors you email every Monday. Want me to track those and remind you?"
>
> "You got 8 customer complaints about the same issue. Want me to flag those instantly?"

**You pick which automations to try.** AI suggests based on your actual patterns. You decide.

---

## 🔐 You Control the Access

This is important: **you decide how much access AI gets.**

| Level | What AI Can Do | Good For |
|-------|----------------|----------|
| **Read-only** | See your emails, can't send | Starting out, observing |
| **Draft** | Write emails, but you approve before sending | Testing automations |
| **Full** | Send on your behalf | After you trust it |

Start with read-only. Expand later. You're always in control.

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
