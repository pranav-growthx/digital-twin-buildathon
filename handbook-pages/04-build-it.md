# Build It

> ~1 hr · your core agent.

---

## Starter file

Give **[BUILDING.md](starter-files/BUILDING.md)** to Claude Code — it contains the rules Claude Code follows to build your agent. Copy or download it into your `my-twin` folder.

---

Now you're in **Claude Code** (the terminal). You'll build three things, one at a time:
the brain, a clean interface, and memory. After each one: test it, then type `checkpoint`
to save your progress.

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

The smallest thing that proves the twin works. You type a message, it replies in your
voice, and it loops. The kickoff prompt starts this — BUILDING.md tells Claude to build
a terminal loop powered by `claude -p --session`.

**What's happening under the hood:** Your agent calls `claude -p` as a subprocess — the
same CLI you're already signed into. The `--session` flag keeps conversation context, so
the twin remembers what was said earlier in the same run. No API key, no cost — it uses
your existing subscription.

When it's done, run it:

```bash
node core.js
```

Talk to it for a minute. **The real test:** text it something a friend would say. Does the
reply sound like you? If the code runs but the voice is off, tell Claude what's wrong —
*"too formal"*, *"shorter replies"*, *"more sarcasm"* — the fix is in the prompt, not
the code.

When it feels right, type `checkpoint` and hit Enter.

## Iteration 1 — clean terminal interface (~15 min)

The brain works — now make it pleasant to use. Right now the output is raw text with no
structure. You want labeled lines so you can tell who said what — `You:` on your messages,
the agent's name on its replies.

Tell Claude to move to iteration 1 — clean up the terminal interface. You should see your
agent's name on its replies and clear separation between messages.

Run it, chat with it, then type `checkpoint` and Enter.

## Iteration 2 — memory (~20 min)

Right now the twin forgets everything when you restart. `--session` keeps context within a
single run, but close the app and it's gone.

This iteration adds two things:

1. **Session persistence** — save the session ID to a file so `--session` picks up where it
   left off across restarts.
2. **Memory functions** — `recallContext(query)` loads relevant past context before each
   reply, `remember(content)` saves a summary of each turn. For now these use a simple JSON
   file. Later, when you install the memory module, these same functions get replaced with a
   real search engine — but the interface stays the same.

Tell Claude to move to iteration 2 — add memory with `recallContext` and `remember`.

**The test:** tell it something specific ("my favourite coffee is filter kaapi"), fully
restart the app (`Ctrl+C`, then `node core.js`), then ask "what's my favourite coffee?"
If it remembers, it works. Type `checkpoint` and Enter.

> You now have an agent that chats like you **and** remembers across restarts. The three
> functions it exports — `askTwin`, `recallContext`, `remember` — are the extension points
> that every module in the next phase plugs into.

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
