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

## Iteration 0 — the brain (~20 min)

A terminal loop. Type a message, get a reply in your voice via `claude -p --session`,
repeat until Ctrl+C. Your kickoff prompt already starts this.

When it's done, run it:

```bash
node core.js
```

Talk to it for a minute. **The real test:** text it something a friend would say. Does the
reply sound like you? If the code runs but the voice is off, tell Claude what's wrong —
*"too formal"*, *"shorter replies"*, *"more sarcasm"* — the fix is in the prompt, not the code.

When it feels right, type `checkpoint` and hit Enter.

## Iteration 1 — clean terminal interface (~15 min)

The brain works — now make it pleasant to use. Paste this:

```
Let's continue to iteration 1. Clean up the terminal interface — labeled lines, clean
formatting. Follow BUILDING.md.
```

You should see your agent's name on its replies and `You:` on yours. Chat with it, then
type `checkpoint` and Enter.

## Iteration 2 — memory (~20 min)

Right now the twin forgets everything when you restart. Paste this:

```
Let's continue to iteration 2. Add memory with the extension points from BUILDING.md —
recallContext, remember, and session persistence.
```

This adds `recallContext()` and `remember()` — the same functions that memory modules will
replace later with a real engine. For now they use a simple JSON file.

**The test:** tell it something specific ("my favourite coffee is filter kaapi"), fully
restart the app (`Ctrl+C`, then `node core.js`), then ask "what's my favourite coffee?"
If it remembers, it works. Type `checkpoint` and Enter.

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
