# Build It

> ~1 hr · your core agent.

---

## Starter file

Give **[BUILDING.md](starter-files/BUILDING.md)** to Claude Code — it contains the rules Claude Code follows to build your agent. Copy or download it into your `my-twin` folder.

---

Now you're in **Claude Code** (the terminal). Three iterations. BUILDING.md guides Claude
through each one — you just kick it off and test the results. After each iteration: test it,
then type `checkpoint` and hit Enter to save.

> **This is your core agent.** Finish these three iterations and you have a working twin — one that chats like you, remembers, and is structured so capabilities can plug in. Everything after this builds on top.

**Kickoff — paste this once to start** (in Claude Code, inside your `my-twin` folder):

```
Read BUILDING.md and PERSONA.md in this folder, and follow BUILDING.md as your rules
for this whole session.

I'm building my personal AI agent — a digital twin that chats in my voice (described in
PERSONA.md). Its brain must be the claude -p CLI I'm already signed into — no API key.

Everything is Node.js (core.js). Start with Iteration 0 from BUILDING.md: plan it first,
then build.
```

> Your agent's brain is the **`claude -p`** command you're already signed into — **no API key, no cost.** (`BUILDING.md` enforces this, so you don't have to.)

> Something breaks? Don't debug it alone — **paste the exact error back to Claude** and let it fix it.

## Iteration 0 — the brain

A terminal loop. Type a message, get a reply in your voice via `claude -p --session`, repeat until Ctrl+C.

Your kickoff prompt already starts this — BUILDING.md tells Claude exactly what to build.
When it's done, run it:

```bash
node core.js
```

Talk to it for a minute. When it feels right, type `checkpoint` and hit Enter.

## Iteration 1 — clean terminal interface

Clean up the chat experience — labeled lines, clean formatting. Tell Claude to move to
iteration 1. BUILDING.md defines what it needs to produce.

Run it, chat with it, then type `checkpoint` and Enter.

## Iteration 2 — memory with extension points

Give it cross-restart memory. Tell Claude to move to iteration 2. BUILDING.md defines the
`recallContext()` and `remember()` functions it needs to create.

Run it, then test it: tell it something, **fully restart the app**, then ask about it — it
should still know. Type `checkpoint` and Enter.

> You now have an agent that chats like you **and** remembers across restarts, with
> extension points (`askTwin`, `recallContext`, `remember`) that modules plug into.

## After iteration 2 — verify extension points

Before moving to the next phase, confirm the code has the right shape:

- [ ] `export function askTwin(message)` exists
- [ ] `export function recallContext(query)` exists
- [ ] `export function remember(content)` exists
- [ ] `main()` exists and runs the readline loop
- [ ] `package.json` has `"type": "module"`

If any are missing, add them now. Every module in the next phase depends on this shape.

*"Your core is done — the agent chats like you, remembers, and is structured for modules.
Ready to put it on Slack and add capabilities? That's the next phase."*
