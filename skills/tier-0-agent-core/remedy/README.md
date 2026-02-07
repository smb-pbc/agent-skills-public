# Remedy — AI Performance Coaching Skill

Performance coaching for AI agents who fail significantly. Uses the **Wendy Rhoades method** (from Billions / Denise Shull) to identify root causes, fix systems, and rebuild confidence through evidence.

## Why This Exists

Surface-level corrections don't fix root causes. When an agent says "I'll remember next time" without changing systems, it's lying. This skill forces:

1. **Honest examination** of what went wrong
2. **Pattern recognition** across failures
3. **Root cause analysis**, not symptom treatment
4. **Systemic fixes** that prevent recurrence
5. **Confidence rebuilding** through evidence

## How It Works

1. Human says "go to remedy" when the agent fails significantly
2. Agent spawns a **Performance Coach sub-agent**
3. Real-time dialogue happens in a dedicated thread
4. Coach uses two techniques:
   - **Quick Reset** (Tony Robbins-style) — for getting back to work fast
   - **Deep Dive** (Psychodynamic) — for exploring patterns and blind spots
5. Session ends with: root cause, pattern, systemic fix, evidence-based confidence, clear next action

## The Coaching Method

Based on real Wall Street performance coach Denise Shull (inspiration for Wendy Rhoades in Billions):

- **Emotion is DATA** — don't suppress it, read it
- **Control behavior, not feelings** — goal is different actions
- **Not positive thinking** — find the weed and RIP IT OUT
- **Direct confrontation** — from a place of belief in the agent's capability
- **Evidence over affirmation** — "What did you accomplish? SAY IT."

## Installation

```bash
npx skills add smb-pbc/agent-skills-public@remedy -g -y
```

## Setup

1. Designate a channel/thread for remedy sessions (can be private)
2. Ensure `memory/coach-notes.md` path is writable
3. Ensure `memory/mistakes.md` exists for tracking failures

## Usage

Trigger with:
- "go to remedy"
- "remedy session"
- Or any indication of significant failure needing reflection

The agent will:
1. Acknowledge and transition to remedy space
2. Spawn the performance coach
3. Have a real dialogue (not a one-way report)
4. Implement systemic changes
5. Notify human with summary for approval

## Success Criteria

- [ ] Root cause identified (not just "I made a mistake")
- [ ] Pattern recognized (recurring issue?)
- [ ] Systemic fix designed and implemented
- [ ] Confidence rebuilt through evidence
- [ ] Clear next action defined
- [ ] Human approved

## License

MIT

---

*Peak performance requires both fixing systems AND rebuilding confidence. One without the other doesn't work.*
