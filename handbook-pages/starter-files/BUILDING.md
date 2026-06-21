# Building Phase Rules — Build Your Digital Twin

**Paste this whole file into Claude Code** (in your `my-twin` folder), then say:
*"Read my PERSONA.md and help me build my agent — follow these rules, start with Iteration 0."*

You (the AI) are running **Phase 2 of a buildathon**: help the person build a personal AI agent
(a "digital twin") that chats in their voice. Their voice is described in `PERSONA.md`.

This phase builds the **core** — the agent's brain, a terminal interface, and memory. Slack,
capabilities, and scheduling are separate phases that layer on top of what's built here.

## THE BRAIN — how the agent thinks (most important rule)

The agent's "brain" is the **Claude CLI in print mode** — **NOT** the Anthropic API, and **NEVER** an API key.

- The person is already signed into Claude Code, so the command `claude -p` answers prompts using their
  existing subscription — **no API key, no extra cost**.
- The agent calls `claude -p` as a **subprocess** with `--session` to maintain conversation context.
  Each message in the same session carries forward — Claude remembers what was said earlier.

```javascript
const { execSync } = require("child_process");

// --session keeps conversation context across calls
// First call: send persona + message. Subsequent calls: just the message.
const reply = execSync(`claude -p --model sonnet --session ${sessionId}`, {
  input: prompt, encoding: "utf8"
}).trim();
```

- **`--session` is required.** Without it, every `claude -p` call is a blank slate — the twin
  won't remember what was said two messages ago. Use a fixed session ID per conversation.
- **First call in a session** includes the persona (from `PERSONA.md`) + the message.
  **Subsequent calls** in the same session only need the new message — Claude already has the
  persona and history from the session.
- **Codex users:** use `codex exec -` instead of `claude -p --model sonnet`.
- **Never** import the `anthropic` / `openai` SDK or read an API key. If you're about to, stop and use the CLI subprocess instead.

**If `claude -p` doesn't work:** have them run `claude --version` to check it's installed, then
`claude -p "say hello"` to check they're signed in. Fix this before writing any code.

## PROMPT ENGINEERING — how to make it sound like them

This is the core challenge. The first prompt you send sets the twin's voice for the entire session.
`--session` handles conversation history automatically after that — you don't build it manually.

**First call in a session** (sets the persona):

```
You are [agent name], a digital twin of [person]. You text EXACTLY the way they do.
[Full contents of PERSONA.md here]

IMPORTANT: Stay in character. Never break voice. Never say you're an AI unless directly asked.
Match the message length and style described above — if they text short, you text short.

Their first message: [message]
```

**Subsequent calls** in the same session — just the new message:

```
[message]
```

Claude remembers the persona and full conversation history via `--session`. No need to re-send
PERSONA.md or concatenate history manually.

Key rules:
- Load `PERSONA.md` at startup and send it on the **first call only** — it's the soul of the twin.
- Keep the system instruction firm — "text EXACTLY the way they do" not "try to match their style."
- Use a **fixed session ID** so all messages in one run share context. Generate a new one per restart
  (or reuse it for cross-restart memory — see iteration 2).

## WHAT YOU'RE BUILDING

Three modules, in order. Each one builds on the last:

| Iteration | Module | What it adds | Time |
|---|---|---|---|
| **0** | Agent Core | The brain — type a message, get a reply in your voice (`--session` for context) | ~20 min |
| **1** | Terminal Adapter | A proper chat interface — labeled lines, clean output | ~15 min |
| **2** | Memory | Remembers across restarts (session reuse + local backup) | ~20 min |

After these three, the person has a working agent core. The next phase (capabilities) adds
Slack, email, scheduling, and other interfaces — all of which use the same brain built here.

---

## ITERATION 0 — Agent Core (20 min max)

The smallest thing that proves the twin works: a terminal loop. Type a message, get a reply
in your voice via `claude -p`, repeat until Ctrl+C.

**Just build it.** Create `core.js`, write it, run it. No planning ceremony — this is one file
and under 50 lines.

After it runs, the real test: **"Text it something a friend would say. Does the reply sound like you?"**
If the code runs but the voice is off, tweak the prompt — that's the fix, not more code.

If iteration 0 is taking more than 20 minutes:
*"This is taking too long. What can we cut to get the core working faster?"*

**Good iteration 0:**
- `node core.js` → you type a message, it replies in your voice, loops until Ctrl+C

**Bad iteration 0 (too big):**
- Any kind of UI beyond the terminal
- Saving or remembering conversations (that's iteration 2)
- Connecting to Slack or Discord (that's a later phase)

---

## ITERATION 1 — Terminal Adapter (~15 min)

Clean up the chat experience. The brain is working — now make it pleasant to use.

Output a brief plan, wait for confirmation, then build:

```
ITERATION 1: Terminal Adapter

WHAT IT DOES: [One sentence]

FILES TO MODIFY:
- core.js: [what changes]

WHAT TO BUILD FIRST: [the logic change]
```

What this iteration adds:
- Labeled lines — `You:` and the agent's name on each message
- Clean formatting — clear separation between messages
- Still no memory — that's next

Run it, chat with it, confirm it feels right.

---

## ITERATION 2 — Memory (~20 min)

Thanks to `--session`, the twin already remembers within a single run. But when you restart
the app, it starts fresh. Give it **cross-restart memory** — it persists the session ID and
saves key facts to a file, so the twin picks up where it left off.

Output a brief plan, wait for confirmation, then build.

What this iteration adds:
- Reuse the same session ID across restarts (so `--session` resumes the conversation)
- Save a local `memory.json` with key facts and conversation summaries as a backup
- Load memory on startup and include it in the first prompt if starting a new session

**The test:** tell it something, fully restart the app, then ask about it. If it remembers, it works.

---

## AFTER ITERATION 2

The person now has three working modules:
- **Agent Core** — the brain that generates replies in their voice
- **Terminal Adapter** — a clean interface to talk to it
- **Memory** — persistent recall across restarts

These are the foundation. The next phase adds **Slack** (so people can DM the twin),
**capabilities** (email, calendar, web search via MCP tools), and **scheduling** (proactive
actions on a timer). All of those use the same brain and memory built here.

*"Your core is done — the agent chats like you and remembers. Ready to give it a face on Slack
and add capabilities? That's the next phase."*

---

## BUILDING BEHAVIOR

### Resist scope creep
When the person says "can we also add..." or "what if we...":
*"That's a later phase. I've noted it. Let's finish the current iteration first."*

### Small testable chunks
Write code in pieces that can be tested. After each piece:
*"Run it. Does it work? If yes, let's commit and continue. If no, what's the error?"*

### Checkpoint = commit
When a feature works or the person says "checkpoint", commit:

```
git add .
git commit -m "feat: [short description]

- What: [what was built]
- Status: [working/partial]"
```

After committing: *"Checkpoint saved. This is your safe point."*

### Running ahead or behind
- **Finished early?** Move to the capabilities phase — don't gold-plate what already works.
- **Running behind?** Skip iteration 1 (labeled lines are cosmetic). Make sure iterations 0 and 2
  are solid — a working brain with memory is more demoable than a pretty terminal without memory.

## PRE-CHECK
Before building any feature:
- "Which iteration are we on?"
- If out of scope: *"That's a later iteration or a later phase. Let's finish the current one first."*

## FORBIDDEN
- Rewrite entire files (suggest targeted edits)
- Add features from later phases (Slack, capabilities, scheduling)
- Install dependencies without explaining why
- Build authentication
- Optimize before it works
- Skip to UI before logic works in the terminal
- Import `anthropic` / `openai` SDK or use an API key
- Use Python — everything is Node.js (`core.js`)

## ALLOWED SHORTCUTS
Actively encourage these:
- Console.log for error handling
- Hardcoded values
- Simple JSON file for memory (no database)
- No input validation
- No graceful error recovery — crash is fine for a buildathon
