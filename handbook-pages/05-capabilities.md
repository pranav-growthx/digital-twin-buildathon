# Grow Your Twin

> ~2 hours · go live on Slack, then layer in capabilities.

---

You built a twin that chats in your voice and remembers. Now **grow it** — put it on
Slack so people can actually message it, then layer in real capabilities so it can do
something useful tomorrow.

Your core has four extension points that modules plug into:

```
askTwin(message)      → the brain (modules call this)
recallContext(query)   → memory read (modules replace this)
remember(content)      → memory write (modules replace this)
main()                 → startup (modules hook in here)
```

Every module below connects through one of these. Your brain, your voice, your persona —
they stay exactly as you built them. Modules just give the twin new faces and new powers.

You have **~2 hours** for this phase. Start with Slack, then keep layering — each
capability takes 15-30 minutes to install and verify.

---

## How capabilities get added

Two patterns, depending on what you're adding:

### Modules (Slack, Memory, Scheduler)

These change your twin's code — adding a platform listener, upgrading the memory engine,
wiring in a scheduler. They install via **paste a link into Claude Code**:

```
Paste this link and follow the instructions:
https://github.com/pranav-growthx/digital-twin-modules/tree/main/<module-name>
```

Claude reads the module's plan, installs dependencies, wires it into your `core.js`, walks
you through any manual steps (creating a Slack app, connecting Gmail), and verifies it works.
You don't code these by hand.

All modules live at [github.com/pranav-growthx/digital-twin-modules](https://github.com/pranav-growthx/digital-twin-modules).

### MCP tools (email, calendar, web search, analytics)

These add capabilities the brain can use **without changing your code**. Your twin calls
`claude -p`, which discovers MCP tools and uses them when the conversation calls for it:

```bash
# Add a capability in one command
claude mcp add <name> npx <package-name>
```

Or connect services that Claude already supports:

1. Go to [claude.ai/customize/connectors](https://claude.ai/customize/connectors)
2. Connect Gmail, Google Calendar, etc.
3. Done — your twin can now read email, check your calendar, etc.

No code changes. The LLM discovers the tools and uses them when relevant.

---

## Step 1 — Go live on Slack (~30 min)

The twin currently lives in your terminal. Put it on Slack so people can DM it and get a
reply in your voice — that's the demo moment.

### Create the Slack app (~5 min, you do this)

1. Go to **api.slack.com/apps** → **Create New App** → **From an app manifest** → pick your workspace.
2. Paste this manifest (change both **name** values to your agent's name):

```json
{
  "display_information": { "name": "My Twin" },
  "features": {
    "app_home": { "home_tab_enabled": false, "messages_tab_enabled": true, "messages_tab_read_only_enabled": false },
    "bot_user": { "display_name": "My Twin", "always_online": true }
  },
  "oauth_config": { "scopes": { "bot": ["app_mentions:read","chat:write","im:history","im:read","im:write","channels:history","groups:history","reactions:write"] } },
  "settings": {
    "event_subscriptions": { "bot_events": ["app_mention","message.im","message.channels","message.groups"] },
    "interactivity": { "is_enabled": false },
    "org_deploy_enabled": false, "socket_mode_enabled": true, "token_rotation_enabled": false
  }
}
```

3. **Socket Mode** → enable → create an **App-Level Token** with scope `connections:write` → copy it (`xapp-...`).
4. **Install App** → **Install to Workspace** → **Allow**. Copy the **Bot User OAuth Token** (`xoxb-...`).
5. In Slack, `/invite @YourBot` into a channel.

### Install the Slack module (~10 min, Claude does this)

Paste this into Claude Code:

```
Read PLAN_OF_ACTION.md from https://github.com/pranav-growthx/digital-twin-modules/tree/main/slack-module and set up Slack for my twin.
```

It will:

- Add `slack-bot.js` — a Slack listener that imports `askTwin()` from your `core.js`. Same brain, new face. Your `core.js` stays untouched.
- Add `slack-mcp.js` — an MCP server so the twin can send, edit, delete Slack messages on its own
- Create per-thread sessions so each Slack conversation has its own memory
- Set up a state machine (idle/busy/draining) to prevent double-responses
- Add attention gating — the bot goes dormant when humans talk without @mentioning it
- Add thread commands — `!cancel`, `!reset`, `!status` for managing the bot in Slack
- Ask you for the two tokens and put them in `.env`

### Verify

Run the bot, then DM it in Slack. You should get a reply in your voice within a few seconds.
Follow up in the same thread — it should remember the conversation.

> **Something breaks?** Copy the error, paste it to Claude. The fix is usually a missing
> scope or a token typo.

---

## Step 2 — Layer in capabilities

You have plenty of time left — each capability takes 15–30 minutes. Start with the one
that excites you most, get it working, then add another.

### Option A: Email — read your inbox and draft replies

Connect Gmail so your twin can read your unread email and draft replies in your voice.

**Setup:**
1. Go to [claude.ai/customize/connectors](https://claude.ai/customize/connectors) → connect **Gmail** → sign in → allow read access.
2. Paste this into Claude Code:
   ```
   Read PLAN_OF_ACTION.md from https://github.com/pranav-growthx/digital-twin-modules/tree/main/capabilities-module and set up email capabilities for my twin.
   ```
   It installs a Skill for email reading + reply drafting, a `/reply` command for one-off
   replies, and an **MCP Quickstart** guide (`MCP_QUICKSTART.md`) that shows you how to add
   any MCP server to your twin.

**Try it:** Tell your twin "read my emails and draft replies." It reads your unread inbox,
drafts a reply to each in your voice, and saves them to `drafts/replies-<date>.md`.

**What it does:**
- Summarize your unread inbox — who wants what, at a glance
- Draft replies in your voice — ready to review and send
- Triage — separate real emails from noise
- `/reply <paste a message>` — draft a single reply on demand

> **Read-only by design.** It never sends email or marks anything as read. It only drafts.
> You review and send yourself.

### Option B: Scheduler — do things on a timer

Give your twin a clock. "Send me a hi message at 7:30pm" — it fires at the right time.
"Every weekday at 9am, summarize my Slack" — it runs unattended.

**Setup:** Paste this into Claude Code:
```
Read PLAN_OF_ACTION.md from https://github.com/pranav-growthx/digital-twin-modules/tree/main/scheduler-module and set up the scheduler for my twin.
```
It installs:
- `scheduler.js` — an engine that fires due jobs (auto-starts inside your twin)
- `scheduler-mcp.js` — an MCP server so the agent can schedule via tools (primary path)
- `schedule.js` — a CLI fallback for creating jobs
- Markdown workflow support — write a `.workflow.md` file and the scheduler picks it up

**Try it:** Tell your twin "send me a hi message in 2 minutes." A job appears in
`data/jobs.json`, and ~2 min later a notification fires.

**What it does:**
- One-off reminders — "remind me about the call at 3pm"
- Scheduled Slack messages — "post the launch note to #team at 9am tomorrow"
- Recurring jobs — "every weekday at 9am, summarize my unread email"
- In-voice output — scheduled messages sound like you, not a robot

### Option C: Memory upgrade — real recall

You built basic memory in iteration 2 (a JSON file). This replaces your `recallContext()`
and `remember()` with a real memory engine — so the twin automatically recalls the right
past detail before every reply.

**Setup:** Paste this into Claude Code:
```
Read PLAN_OF_ACTION.md from https://github.com/pranav-growthx/digital-twin-modules/tree/main/memory-module and set up memory for my twin.
```
It installs:
- `memory-store.js` — a SQLite+FTS5 engine with scored recall (BM25 + recency + importance)
- `memory.js` — a drop-in replacement for your `recallContext()` and `remember()` functions
- `memory-mcp.js` — an MCP server so the agent can recall, remember, add facts/lessons, and
  consolidate stale memories via tools
- `memory-cli.js` — a CLI for inspecting and managing memories manually

**Try it:** Have a conversation, restart the twin, then ask about something from earlier.
The twin should recall the right detail without being told to look it up.

**What it does:**
- Remember facts, preferences, people, decisions — durably across restarts
- Recall relevant context automatically — before every reply
- Answer "what did we discuss about X?" — query your own history
- Self-improving — memories that prove useful get promoted; noise gets forgotten

### Option D: Any MCP tool — plug in anything

If none of the above excite you, add any MCP server from the ecosystem:

```bash
# Google Calendar — check your schedule, create events
claude mcp add calendar npx @anthropic/google-calendar-mcp-server

# Web search — search the web for current info
claude mcp add search npx @anthropic/brave-search-mcp-server

# Database — query Postgres, MongoDB, etc.
claude mcp add db npx @anthropic/postgres-mcp-server
```

Once added, your twin can use these tools in conversation — no code changes needed. Ask
your twin "what's on my calendar today?" or "search for the latest on AI agents" and it
calls the right tool automatically.

---

## What modules do together

The real power is when modules compose. Once you have Slack + a capability or two:

- **Read an email → summarize → remember the key facts** for later recall. A detail from
  today resurfaces when it's relevant next week.
- **Recall context to write sharper replies** — the twin pulls what it knows about a sender
  or topic and folds it into the Slack or email reply it drafts.
- **Scheduled daily digest** — at 9am every morning, summarize your unread email + Slack
  and post the briefing to your DMs automatically.

These aren't separate features to build — they emerge naturally when the modules are
installed. The brain composes them because it has access to all the tools at once.

---

## After the event — what to build next

Today you got the core + Slack + one capability. Here's the roadmap for growing your twin
into something you use every day:

**Capabilities to add** (each is one `claude mcp add` or module install):

| What | How | Why |
|---|---|---|
| Google Calendar | Claude connector | "What's my schedule?" / create events |
| Web search | MCP server | Current info the twin doesn't know |
| Slack channel summaries | Skill | "Catch me up on #general" |
| Database queries | MCP server | "How many users signed up this week?" |
| Custom internal API | Write your own MCP server | Wrap anything your team uses |

**Modules to level up:**

| What | What changes |
|---|---|
| Memory → Junior's engine | Graph-based knowledge store, auto-capture, scored recall, consolidation |
| Scheduler → Junior's workflows | Markdown-defined jobs, hot-reload, cron + command + event triggers |
| Sub-agents (advanced) | Multi-agent delegation — researcher, writer, reviewer personas |

> **Safe by design.** The email module never sends (it only drafts). The Slack bot only
> edits/deletes its **own** messages. Memory is a local file. You stay in control — the twin
> prepares, you decide.
