# Build It

> ~1 hr · your core agent.

---

## Starter file

Give **[BUILDING.md](starter-files/BUILDING.md)** to Claude Code — it contains the rules Claude Code follows to build your agent. Copy or download it into your `my-twin` folder.

---

Now you're in **Claude Code** (the terminal). Three iterations. **You write the prompts yourself** — figuring out what to ask for *is* the building. After each iteration: test it, then type `checkpoint` and hit Enter to save.

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

> After this, **you write the prompts** — deciding what to ask for *is* the building. (Stuck? Each iteration has an optional example you can expand.)

> Something breaks? Don't debug it alone — **paste the exact error back to Claude** and let it fix it.

## Iteration 0 — the brain

A terminal loop. Type a message, get a reply in your voice via `claude -p --session`, repeat until Ctrl+C.

Your kickoff prompt already starts this — make sure what Claude builds does all of these:

1. You run it with `node core.js`.
2. It reads your `PERSONA.md`.
3. It has an `askTwin(message)` function that calls `claude -p --session` and returns the reply.
4. It loops: waits for you to type → calls `askTwin()` → prints the reply → waits again.
5. `askTwin` is exported: `export function askTwin(...)`.
6. `package.json` exists with `"type": "module"`.

<details>
<summary>Show an example prompt (optional)</summary>

```
Build a Node.js program in core.js that I run from the terminal.
It reads my PERSONA.md, then loops: waits for me to type a message, sends my personality
+ my message to claude -p --session (via child_process), prints the reply in my voice,
then waits for the next message. It exits on Ctrl+C. No API key — use claude -p.

The askTwin function must be an export so other modules can import it later.
Make sure package.json has "type": "module". Plan it first, then build.
```

</details>

Run it:
```bash
node core.js
```

Talk to it for a minute. When it feels right, type `checkpoint` and hit Enter.

## Iteration 1 — clean terminal interface

Clean up the chat experience. The brain is working — now make it pleasant to use.

Write a prompt that asks Claude to:

1. Keep it running as a continuous chat that waits for each new message.
2. Make the interface clean — label the lines (like `You:` and your agent's name).
3. **No memory yet** — that's the next iteration.

Run it, chat with it, then type `checkpoint` and Enter.

## Iteration 2 — memory with extension points

Right now it forgets everything when you restart. Give it **cross-restart memory** with
named functions that capability modules can replace later.

Write a prompt that asks Claude to:

1. Create an `export function recallContext(query)` that loads relevant past context from
   `memory.json` (returns a string, or `""` if nothing relevant).
2. Create an `export function remember(content)` that saves content to `memory.json`.
3. Call `recallContext` before each `askTwin` call and prepend the result to the prompt.
4. Call `remember` after each reply with a summary of the turn.
5. Reuse the same session ID across restarts (save it to a file).

Run it, then test it: tell it something, **fully restart the app**, then ask about it — it should still know. Type `checkpoint` and Enter.

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
