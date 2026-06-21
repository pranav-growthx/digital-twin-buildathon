# Build It

> ~60 min · ⭐ your core win.

---

## Starter file

Give **[BUILDING.md](starter-files/BUILDING.md)** to Claude Code — it contains the rules Claude Code follows to build your agent. Copy or download it into your `my-twin` folder.

---

Now you're in **Claude Code** (the terminal). Three small steps. **You write the prompts yourself** — figuring out what to ask for *is* the building. After each step: test it, then type `checkpoint` and hit Enter to save.

> **This is your core win.** Finish these three steps and you have a real agent — one that chats like you and remembers — ready to demo. Slack and capabilities (next) are bonuses; this is the part to nail.

**Kickoff — paste this once to start** (in Claude Code, inside your `my-twin` folder):

```
Read BUILDING.md and PERSONA.md in this folder, and follow BUILDING.md as your rules
for this whole session.

I'm building my personal AI agent — a digital twin that chats in my voice (described in
PERSONA.md). Its brain must be the claude -p CLI I'm already signed into — no API key.

Build everything in ONE language — Python (file named core.py) OR Node.js (core.js).
Pick one, keep the same file names throughout, and don't rename files later.

Start with Iteration 0 from BUILDING.md: plan it first, then build.
```

> Your agent's brain is the **`claude -p`** command you're already signed into — **no API key, no cost.** (`BUILDING.md` enforces this, so you don't have to.)

> After this, **you write the prompts** — deciding what to ask for *is* the building. (Stuck? Each step has an optional example you can expand.)

> Something breaks? Don't debug it alone — **paste the exact error back to Claude** and let it fix it.

## Iteration 0 — a tiny terminal program

A read-and-answer loop you run in the terminal. Your kickoff prompt already starts this — make sure what Claude builds does all of these:

1. You run it from the terminal.
2. It reads your `PERSONA.md`.
3. It loops: waits for you to type a message → prints a reply in your voice (via `claude -p`) → waits for the next one.
4. It exits when you press **Ctrl+C**.

<details>
<summary>Show an example prompt (optional) — Python</summary>

```
Build a small Python program in a file named core.py that I run from the terminal.
It reads my PERSONA.md, then loops: waits for me to type a message, sends my personality
+ my message to the claude -p CLI (as a subprocess), prints the reply in my voice, then waits
for the next message. It exits on Ctrl+C. No API key — use claude -p. Plan it first, then build.
```

</details>

<details>
<summary>Show an example prompt (optional) — JavaScript</summary>

```
Build a small Node.js program in a file named core.js that I run from the terminal.
It reads my PERSONA.md, then loops: waits for me to type a message, sends my personality
+ my message to the claude -p CLI (via child_process), prints the reply in my voice, then waits
for the next message. It exits on Ctrl+C. No API key — use claude -p. Plan it first, then build.
```

</details>

Then run it in the terminal:

**Python:**
```bash
python core.py
```

**JavaScript:**
```bash
node core.js
```

Talk to it for a minute. When it feels right, type `checkpoint` and hit Enter.

## Iteration 1 — a proper app

Open it up into a real chat app.

Write a prompt that asks Claude to:

1. Keep it running as a continuous chat that waits for each new message.
2. Make the interface clean — label the lines (like `You:` and your agent's name).
3. **No memory yet** — that's the next step.

Run it the same way, chat with it, then type `checkpoint` and Enter.

## Iteration 2 — give it lasting memory

Right now it forgets everything when it restarts. Give it **persistent memory** — it saves the conversation to a file and loads it back, so it remembers across messages *and* restarts.

Write a prompt that asks Claude to:

1. Save each message and reply to a file on disk (for example `memory.json`).
2. Load that history when the app starts, and include it so replies stay in context.
3. Make sure it survives a restart — not just memory inside one run.

Run it, then test it: tell it something, **fully restart the app**, then ask about it — it should still know. Type `checkpoint` and Enter.

> You now have an app that chats like you **and** remembers across restarts. Next: put it somewhere people can actually message it.
