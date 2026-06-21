# Digital Twin — Core Modules

The modules below are the building blocks of a digital twin agent. For each module, you have two paths:

- **Use ours** — a ready-made implementation derived from Junior (our production agent). Paste a link into Claude Code, the agent installs it, done.
- **Build your own** — each module has a clear interface. Swap in your own implementation as long as it fulfills the same contract.

The twin runs as a single Node.js process. It listens on a messaging platform, manages conversation state, spawns an LLM subprocess to think, and uses MCP tools to act. Memory and scheduling run alongside.

```
┌─────────────────────────────────────────────────────────┐
│                    Your Digital Twin                     │
│                                                         │
│  ┌────────────────────┐   ┌──────────────────────┐      │
│  │ Platform Adapter   │──▶│  Agent Core           │      │
│  │ (Slack / Discord / │   │  (LLM subprocess)     │      │
│  │  Terminal)          │   └────────┬─────────────┘      │
│  │  + session state   │            │                     │
│  └────────────────────┘   ┌────────┤                     │
│       ▲                   ▼        ▼                     │
│  ┌──────────┐       ┌──────────┐  ┌──────────────────┐  │
│  │Scheduler │       │  Memory  │  │  MCP Tools        │  │
│  │(cron/    │       │ (recall +│  │  (capabilities)   │  │
│  │ timers)  │       │  learn)  │  └──────────────────┘  │
│  └──────────┘       └──────────┘                         │
└─────────────────────────────────────────────────────────┘
```

Participants build the core first (terminal chat loop — iterations 0-2 in BUILDING.md). Everything below layers onto that foundation.

---

## How Modules Get Installed

Every module is designed for **agent-driven installation**. Participants don't wire things up by hand — they paste a repo link (or a one-liner) into Claude Code and the agent does the rest.

Each module ships:
- **`PLAN_OF_ACTION.md`** — an ordered checklist the coding agent follows step by step
- **A guide file** — full code + instructions the agent reads and executes
- **A `SKILL.md`** — teaches the agent when and how to use the new capability
- **Reference code** — working implementation files the agent copies into the project

The install flow:
```
Participant pastes a repo link into Claude Code
     │
     ▼
Claude reads the README → follows PLAN_OF_ACTION.md
     │
     ├── Installs dependencies
     ├── Copies reference code into the project
     ├── Wires it into the twin's startup
     ├── Walks the participant through any manual steps (Slack app, Gmail auth)
     └── Verifies it works end-to-end
```

This is the pattern for every module. Participants learn to install capabilities by pasting a link — a skill that transfers to any future module or MCP server.

---

## 1. Agent Core (built by participants)

**What it does:** The brain. Takes a message + persona + context, sends it to an LLM, returns a reply in the user's voice.

**Why it matters:** Everything else feeds into or out of this. The persona prompt, conversation history, recalled memories, and tool results all get composed into one prompt that goes to the LLM. The quality of the twin lives or dies here.

### Participants build this themselves

The Agent Core is what participants create in iterations 0-2 of BUILDING.md. It's not a module you install — it's the foundation you write:

- **Iteration 0** — a `core.js` with a terminal loop that calls `claude -p --session` and gets replies in the user's voice
- **Iteration 1** — clean up the chat interface (labeled lines, formatting)
- **Iteration 2** — add memory (session reuse across restarts + `memory.json` backup)

By the end of iteration 2, the participant has an `askTwin(message)` function and a `recallContext(query)` / `remember(content)` pair. These are the **extension points** that every module plugs into:

```
export function askTwin(message)        ← Platform Adapter calls this
export function recallContext(query)    ← Memory module replaces this
export function remember(content)       ← Memory module replaces this
main()                                  ← Scheduler hooks into startup here
```

The code is structured so that modules can `import { askTwin } from "./core.js"` and use the brain without rewriting it. BUILDING.md enforces this shape.

### How modules upgrade it

When participants install modules, the core gets upgraded implicitly:

- **Platform Adapter** (Slack) — replaces the terminal `readline` loop with a Slack listener, but keeps calling `askTwin()`. Upgrades the subprocess from `execSync` to `spawn()` with timeout, streaming, and per-thread session IDs.
- **Memory module** — replaces the simple `memory.json` functions with engram or Junior's SQLite+FTS5 store. Same `recallContext` / `remember` interface, better implementation.
- **Scheduler** — calls `startScheduler()` at startup, which fires `askTwin()` for due jobs.
- **Capabilities** — MCP servers are configured via `claude mcp add` or `.mcp.json`. The LLM discovers and uses them during `askTwin()` calls — no code change to the core.

### Key design decisions

- **Subprocess, not SDK.** Participants are signed into Claude Code — `claude -p` uses their subscription with zero setup. No API key, no billing surprises.
- **`--session` for context.** Claude maintains conversation history at the CLI level. No need to re-send full history on every call.
- **Exportable brain.** `export function askTwin` so modules can import it rather than rewrite it.

---

## 2. Platform Adapter (Messaging + Sessions)

**What it does:** Connects the twin to a chat platform and manages conversation state. The platform determines how messages arrive, how replies are sent, and what a "conversation" looks like.

**Why it matters:** The twin needs a face. Without this, it's a script you run manually. With it, people can DM your twin and get a reply — that's the demo moment.

### How sessions work per platform

| Platform | What is a conversation? | Session key | History |
|---|---|---|---|
| **Terminal** | There's only one | `"default"` | Array in memory, saved to `memory.json` on exit |
| **Slack** | A thread (or DM) | `thread_ts` | Messages stored per-thread, loaded on each turn |
| **Discord** | A channel or DM thread | `channel.id` | Same — messages stored per-channel |
| **Web UI** | A chat session | Session cookie / ID | Same pattern, different key |

The pattern is always the same: **conversation ID → message history → inject into prompt**. The platform just determines where the ID comes from.

### Use ours (derived from Junior)

Junior's platform layer is a **Slack bot built on `@slack/bolt`** with a full session manager (`src/session/manager.ts`). What we extract:

**Slack integration:**
- **Socket Mode** — no public URL needed. Connects outbound, works behind firewalls, on laptops, no ngrok.
- **Event routing** — incoming messages go through a priority chain: attention gate → command handler → agent dispatch. The bot knows when it's being talked to vs. when humans are talking to each other.
- **Thread-scoped replies** — every reply goes in the same thread. Follow-ups in a thread don't need another @mention.
- **Attention gating** — if two humans are talking in a thread without @mentioning the bot, it goes dormant. Reactivates when addressed.
- **Agent identity** — the bot posts with a configurable name and icon. The twin has its own Slack persona.

**Session management:**
- **State machine per thread** — `idle → busy → draining → idle`. Prevents double-responses and race conditions.
- **Buffer-drain** — new messages while the agent is thinking get queued. When the current turn finishes, queued messages are combined into one follow-up prompt. The agent never misses a message.
- **Persistence** — sessions survive restarts via SQLite. An in-memory store exists for development/testing. Factory pattern: `createSessionStore(config)` selects the backend.
- **Thread commands** — `!cancel`, `!reset`, `!status` for controlling the bot in Slack.

**Source:** `src/session/manager.ts`, `src/session/types.ts`, `src/session/store/`, `src/index.ts` (event handler registration)

**What we simplify:** Strip multi-agent session support (that's the Sub-Agents module). Strip tmux driver mode. Keep the Slack bot + session state machine + persistence core.

### Build your own

The platform adapter contract:

```
Platform event (message arrives)
     │
     ▼
Platform Adapter
     │
     ├── Extract conversation ID (thread_ts, channel.id, "default")
     ├── Load history for that conversation
     ├── Call Agent Core with (message, history, persona)
     ├── Post reply back to the platform
     └── Save updated history
```

Swap it for any platform:
- **Discord** — `discord.js`, use `channel.id` as the session key
- **Telegram** — `node-telegram-bot-api`, use `chat.id` as the session key
- **WhatsApp** — Twilio or Baileys, use phone number as the session key
- **Web UI** — Express endpoint with a session ID in the request

The simplest version: a `Map<conversationId, Message[]>` and a `readline` loop. No state machine needed if you only handle one conversation at a time.

### Key design decisions

- **Session = conversation ID + state.** Junior's session manager tracks more than just history — it knows whether the agent is busy, queues messages, and drains them. This prevents the "sent 3 messages, got 3 separate replies" problem.
- **Socket Mode for Slack.** No public URL. No ngrok. Works on a laptop.
- **Upgrade in place.** The terminal twin *becomes* the Slack bot. The brain stays the same; only the face changes.

---

## 3. Capabilities (MCP + Skills)

**What it does:** Gives the agent capabilities beyond conversation — reading email, posting to Slack, searching the web, querying Mixpanel. Capabilities are added through **MCP servers** that the LLM discovers and calls automatically.

**Why it matters:** A twin that can only chat is a novelty. A twin that can check your email, draft a reply, and schedule a follow-up is something you use the next day. More importantly — learning to add capabilities via MCP is a skill that **scales**. Once you know the pattern, you can plug in any MCP server from a growing ecosystem without writing integration code.

### How it works

The twin's brain is `claude -p`, which has built-in MCP support. You register tool servers, and the LLM automatically discovers and calls them when the conversation needs it.

```
Your twin calls claude -p
     │
     claude -p sees configured MCP servers + connectors
     │
     ├── Gmail connector → read inbox, draft replies
     ├── Google Calendar connector → check schedule, create events
     ├── Slack MCP → read channels, post messages
     ├── Mixpanel MCP → query analytics, check dashboards
     └── ... any MCP server you add
```

The LLM decides when to call a tool based on the conversation. You never write tool-calling logic — the LLM handles it.

### Use ours (derived from Junior)

Junior runs a **shared HTTP MCP server** (`src/mcp/slack-server.ts`) inside its main process. All spawned agent turns connect to it. What we extract:

**MCP server architecture:**
- **Single shared server** at `localhost:3456` — one process, one server, all conversations share it. No per-session server overhead.
- **Stateless transport** — each HTTP request gets a fresh MCP transport with run context (channel, thread, agent name) parsed from URL query params. No session stickiness.
- **16 registered tools** across categories: Slack communication (send, read, search, upload), agent orchestration (dispatch, search), memory (recall, consolidate), permission (human-in-the-loop), worktree management.
- **Human-in-the-loop approval** — a `request_permission` tool that posts Allow/Deny buttons to Slack. The LLM blocks until the human responds. Wired to Claude's `--permission-prompt-tool` contract.

**Per-agent tool scoping:**
- Each agent definition declares which MCP servers and tools it can access via `permissions.mcp` in frontmatter.
- `allowedMcpServers(session)` checks the agent's permissions and only wires up declared servers.
- Conditional injection: MongoDB and Mixpanel MCPs are only available when the active agent's permissions declare them. Slack MCP is always available.

**Source:** `src/mcp/slack-server.ts`, `src/mcp/context.ts`, `src/mcp/mongodb-proxy.ts`, `src/runners/mcp-config.ts`

**What we simplify:** Strip the shared HTTP server down to just the MCP config generation. For buildathon twins, the simpler path is `claude mcp add` / `.mcp.json` — the LLM's built-in MCP support handles the rest. We keep human-in-the-loop and per-agent scoping as opt-in features.

### Three ways to add capabilities

**1. Claude Connectors (zero setup)**

Some capabilities are pre-built into Claude and just need a one-time web auth:

1. Go to [claude.ai/customize/connectors](https://claude.ai/customize/connectors)
2. Connect Gmail, Google Calendar, or other services
3. Done — `claude -p` can now use those tools automatically

No code, no config files, no MCP server to run. The tools just appear.

**2. MCP Servers (the universal plug)**

For everything else, add an MCP server:

```bash
claude mcp add mixpanel npx @anthropic/mixpanel-mcp-server
claude mcp add slack npx @anthropic/slack-mcp-server
claude mcp add <name> npx <package-name>
```

Or configure all at once via `.mcp.json` in your project folder:

```json
{
  "mcpServers": {
    "mixpanel": {
      "command": "npx",
      "args": ["@anthropic/mixpanel-mcp-server"]
    }
  }
}
```

Once configured, every `claude -p` call can use these tools. Want Notion? Same pattern. Want a database? Same pattern. The ecosystem does the work.

**3. Skills (orchestration layer)**

A **Skill** is a `.claude/skills/<name>/SKILL.md` file that teaches the agent *when* to use a capability and *how* to compose a workflow around it. Skills sit on top of MCP — providing the "when" and "how" while MCP provides the "what."

```markdown
---
name: email-replies
description: Read unread Gmail and draft replies in the agent's voice.
  Trigger when the user asks to read their email and draft replies.
---

# Email replies
1. Use the Gmail tools to fetch unread emails (read-only, never send)
2. Draft a reply to each in the agent's voice (from PERSONA.md)
3. Save all drafts to drafts/replies-<date>.md
```

### Build your own

Write your own MCP server using `@modelcontextprotocol/sdk`:

```javascript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
const server = new McpServer({ name: "my-tools", version: "1.0.0" });

server.tool("check_weather", { city: { type: "string" } }, async ({ city }) => {
  const res = await fetch(`https://wttr.in/${city}?format=j1`);
  return { content: [{ type: "text", text: JSON.stringify(await res.json()) }] };
});
```

Or skip MCP entirely and give the twin helper scripts it calls via the Bash tool.

### What capabilities to add

| Capability | How to add it | What it enables |
|---|---|---|
| **Gmail** | Claude connector (web auth) | Read inbox, draft replies in your voice |
| **Google Calendar** | Claude connector (web auth) | Check schedule, create events, find free slots |
| **Slack actions** | MCP server | Read channels, send/edit/delete messages |
| **Mixpanel** | MCP server | Query analytics, check dashboards, track events |
| **Web search** | MCP server (Brave/Tavily) | Search the web for current info |
| **Database** | MCP server (Postgres/MongoDB) | Query and manage data |
| **File system** | Built into Claude Code | Read/write local files |
| **Custom API** | Write your own MCP server | Wrap any internal or proprietary API |

### Key design decisions

- **MCP as the primary pattern.** It's a universal plug — learn it once, add any capability. Bespoke scripts for each capability don't scale; MCP does.
- **Connectors for the common stuff.** Gmail and Calendar are one-click web auth. No server to run.
- **Skills for orchestration.** A Skill teaches the agent when and how to use capabilities in combination. The capability itself comes from MCP or a connector.
- **Additive.** Start with zero capabilities (chat only). Add them one `claude mcp add` at a time.

---

## 4. Memory

**What it does:** Stores what the twin learns — facts, preferences, past conversations, corrections — and recalls relevant context before each reply. The twin gets smarter the more you use it.

**Why it matters:** Without memory, the twin is a goldfish. It asks the same questions, forgets your preferences, and gives generic answers. With memory, it knows your manager's name is Rahul, that you prefer bullet points, and that you discussed the Q3 roadmap last Tuesday.

### Use ours (derived from Junior)

Junior's memory system (`src/memory/`) is a **graph-based knowledge store** backed by SQLite with FTS5 full-text search. No embeddings, no vector database, no API key. What we extract:

**Storage model:**
- **SQLite + FTS5** — one file on disk (`data/memory.db`). Zero infrastructure. WAL mode with `PRAGMA synchronous = NORMAL` for performance.
- **Typed nodes** — `event` (raw observations), `lesson` (distilled learnings with `appliesWhen` context), `fact` (curated knowledge), `entity` (named things), `tag` (classifications), `summary` (auto-generated consolidations).
- **Typed edges** — `lesson_from`, `same_topic`, `follows_up`, `contradicts`, `supersedes`, `merged_from`, `mentions`, `tagged_as`. Edges connect memories into a knowledge graph.

**Ingestion pipeline:**
- **Auto-capture** — every Slack message, runner result, and routing decision is ingested automatically via `MemoryIngestor`.
- **Heuristic tagging** — incoming content is auto-tagged (frontend, backend, PR review, correction) based on regex patterns + accepted rules.
- **Entity extraction** — file paths, names, and other entities are extracted and linked.
- **Provenance chain** — every memory links back to its source record. Nothing is orphaned.

**Recall:**
- **Multi-signal retrieval** — FTS5 BM25 full-text match + tag/entity filter + graph traversal (recursive edge walk up to depth 3, decaying weight ×0.7 per hop).
- **Scoring** — `activation + kindBoost + importance + frequency + recency`. Procedures and routing memories score highest; raw events score lowest. Recency decays linearly over 90 days.
- **Usage tracking** — `use_count++` and `last_used_at` on every recall. Proven memories surface more.

**Consolidation engine:**
- **Archive** — old (30d+), low-importance, never-used events → inactive.
- **Promote** — repeated corrections on the same value → auto-promoted to routing memory fact.
- **Propose rules** — repeated tag corrections → draft auto-tagging rules for human review.
- **Mark stale** — unused facts older than 30 days → inactive.
- **Merge** — `mergeLessons(ids)` / `mergeFacts(ids)` combine duplicates, create `supersedes` edges.

**Interface:**
- `recall(query, options) → Memory[]`
- `ingest(source, content) → void`
- `consolidate() → void`
- Full CLI: `recall`, `add-lesson`, `add-fact`, `add-event`, `merge-lessons`, `consolidate`, etc.

**Source:** `src/memory/store.ts`, `src/memory/sqlite.ts`, `src/memory/ingestion.ts`, `src/memory/factory.ts`, `src/memory/cli.ts`

**What we simplify:** Strip the consolidation engine's rule lifecycle (draft → accepted → rejected). Strip the full node/edge type system down to `fact`, `event`, `lesson`. Keep the SQLite + FTS5 core, auto-capture, and scored recall.

### Build your own

The memory contract:
- `recallContext(query) → string` — retrieve relevant memories, formatted for prompt injection
- `remember(content, importance) → void` — save something

**Simplest version — JSON file (iteration 2):**

```javascript
import { writeFileSync, readFileSync } from "fs";

export function remember(content) {
  const mem = loadMemory();
  mem.push({ content, timestamp: new Date().toISOString() });
  writeFileSync("memory.json", JSON.stringify(mem, null, 2));
}
export function recallContext(query) {
  const mem = loadMemory();
  // simple filter — modules replace this with FTS5, engram, etc.
  const relevant = mem.filter(m => m.content.toLowerCase().includes(query.toLowerCase()));
  return relevant.map(m => m.content).join("\n");
}
function loadMemory() {
  try { return JSON.parse(readFileSync("memory.json", "utf8")); }
  catch (_) { return []; }
}
```

**Other options:**
- **engram** — SQLite + associative recall, offline, no API key
- **Vector store** (Chroma, Pinecone) — semantic similarity instead of keyword search
- **Any store** — as long as it implements `recall` + `remember`, the rest of the system doesn't care

### Key design decisions

- **No embeddings required.** FTS5 keyword search is fast, free, and good enough. No embedding API, no vector DB.
- **SQLite = zero infrastructure.** One file on disk. Survives restarts. Copy to backup.
- **Auto-capture, selective recall.** Everything gets ingested (cheap). Only relevant memories get recalled per turn (scored and filtered).
- **Self-correcting.** When the twin gets corrected, the correction is stored and the old fact is superseded. Accuracy improves over time.

---

## 5. Scheduler

**What it does:** Runs tasks at specific times or on recurring schedules — "send me a summary at 9am every morning," "post to #general at 5pm," "remind me about the meeting in 2 hours."

**Why it matters:** This is what makes the twin useful the next day. A twin that only responds to messages is reactive. A twin that proactively sends your daily digest, fires off a scheduled Slack message, and reminds you about deadlines is an assistant.

### Use ours (derived from Junior)

Junior's workflow system (`src/workflows/`) is a **markdown-defined, hot-reloading automation engine**. What we extract:

**Workflow definitions (markdown-as-config):**
- Each scheduled job is a `*.workflow.md` file with YAML frontmatter (name, triggers, outputs, permissions, runner config) and a prompt body. No code needed to define a job.
- Frontmatter schema: `name`, `enabled`, `description`, `ownerSlackUserIds`, `triggers[]`, `outputs[]`, `runner` (provider, model, timeout), `permissions.tools[]`, `permissions.repos[]`, `concurrency` (skip or parallel).

**Registry + hot-reload:**
- `WorkflowRegistry` scans workflow directories and watches for file changes (FSWatcher with debounce).
- Invalid file edits don't evict previously-loaded definitions (last-known-good semantics).
- Overlay support: org-private workflows in `agents-org/workflows/` override public ones.

**Scheduler:**
- Cron-based via `cron-parser`. Reconciles all timers when the registry reloads.
- Concurrency guard: `skip` mode (default) prevents overlap; `parallel` allows it.
- Timer overflow handling (>2^31ms clamping).

**Trigger types:**
- `schedule` — cron expression + timezone
- `command` — registered as `!<command>` in Slack
- `slack-event` — fires on messages in a specific channel, with optional regex pattern match

**Executor:**
- Spawns a CLI subprocess per workflow run with the workflow prompt + runtime context JSON.
- Idle-interrupt-resume: if the runner goes silent for `idleTimeoutMs`, SIGINT → re-spawn with a continuation prompt (up to `maxIdleInterrupts` times).
- Writes markdown artifacts to `data/workflow-runs/<name>/<date>-<runId>.md`.
- Posts results to configured Slack channels/threads.

**Store:**
- Interface-driven: `InMemoryWorkflowStore` (tests) and `SqliteWorkflowStore` (production).
- Tracks workflow state (active/stopped/invalid, version hash, next/last run) and individual run records.

**Source:** `src/workflows/types.ts`, `src/workflows/registry.ts`, `src/workflows/scheduler.ts`, `src/workflows/executor.ts`, `src/workflows/controller.ts`, `src/workflows/store.ts`

**What we simplify:** Strip overlay support, multi-provider runners, and the Slack command controller (`!workflow run/stop/show`). Keep markdown definitions, cron scheduling, hot-reload, and the executor core.

### Build your own

The scheduler contract:
- `register(workflow) → void`
- `runNow(name) → void`
- `getNextRun(name) → Date`

**Simplest version — setTimeout:**

```javascript
setTimeout(() => {
  sendSlackMessage(channel, "Hey — reminder about the meeting!");
}, 2 * 60 * 60 * 1000);
```

**Other options:**
- **node-cron** — lightweight in-process cron, no persistence
- **System crontab** — `crontab -e` and call your script
- **Cloud scheduler** — AWS EventBridge, Google Cloud Scheduler

### Key design decisions

- **Markdown-as-config.** A scheduled job is a readable, editable, version-controllable file — not a database entry or API call. Participants define jobs by writing prose, not code.
- **Same brain.** Scheduled workflows use the same LLM subprocess and persona as interactive conversations. The twin sounds like you whether replying to a DM or posting a scheduled digest.
- **Hot-reload.** Edit a workflow file, save it, it takes effect. No restart needed.
- **Concurrency control.** A slow-running workflow won't double-fire. The scheduler skips the next tick if the previous run is still going.

---

## 6. Sub-Agents — ADVANCED

> This module is a stretch goal. Most participants won't need it during the buildathon. It's documented here for completeness and for participants who finish early.

**What it does:** Lets the twin delegate specialized work to other agents — a "researcher" that digs into a topic, a "writer" that drafts long-form content. The main agent orchestrates; sub-agents execute.

### What Junior provides

Junior has a full multi-agent system (`src/agents/`, `src/support/`):

- **Markdown agent definitions** — each agent is a `.md` file with YAML frontmatter (`name`, `description`, `tools`, `model`, `permissions.intent`, `context.*` flags) and a system prompt body. 9 agents defined: default, lead, build, review, frontend, architect, pm, reproducer, thinker.
- **Common preamble composition** — agents declare a `common:` profile (e.g., `core`, `building-philosophy`). Common files are loaded and composed into the system prompt.
- **Resolution order** — target repo's `.claude/agents/` → org overlay `agents-org/` → fallback `.claude/agents/`.
- **Dispatch via directives** — the main agent writes `!researcher <prompt>` in its response; the session manager intercepts and spawns the target agent in the same thread.
- **Dispatch allow-lists** — orchestrators (lead, default) can dispatch any agent. Workers have restricted dispatch rights enforced in code (`WORKER_DISPATCH_ALLOW`).
- **Per-agent identity** — each agent posts to Slack with its own name and icon emoji.
- **Per-agent tool scoping** — each agent only has access to declared tools and MCP servers.
- **Multi-agent sessions** — each agent gets its own `AgentSession` under the thread's `ThreadSession`, with independent sessionId, status, and pending message buffer.

**Source:** `src/agents/router.ts`, `src/agents/loader.ts`, `src/support/agents.ts`, `src/support/router.ts`, `.claude/agents/*.md`

### Simpler alternatives (try these first)

- **Single agent** — skip sub-agents entirely. One agent handles everything.
- **Skills** — a `SKILL.md` gives the agent specialized instructions for specific tasks without spawning a separate agent.
- **Multiple prompts** — switch the system prompt based on the task type, but use the same agent process.

---

## Module Dependencies

Not every module needs every other module. Here's what depends on what:

```
Agent Core ◄── required by everything (it's the brain)
     │
     ├── Platform Adapter (needs Agent Core to generate replies)
     │
     ├── Memory (wired into dispatch — recall before, remember after)
     │
     ├── Scheduler (auto-starts in twin, fires claude -p for due jobs)
     │
     ├── Capabilities (MCP servers + connectors discovered by claude -p)
     │
     └── Sub-Agents (ADVANCED)
```

### Build order (what to tackle when)

| Phase | Modules | What you have |
|---|---|---|
| **Iterations 0-2** | Agent Core + Terminal | A terminal chat that sounds like you and remembers |
| **Iteration 3** | + Platform Adapter (Slack) | People can DM your twin on Slack |
| **Capabilities** | + MCP / Connectors | Your twin can read email, check calendar, query Mixpanel |
| **Capabilities** | + Memory (level 2+) | Your twin recalls relevant past context automatically |
| **Capabilities** | + Scheduler | Your twin does things on its own schedule |
| **Advanced** | + Sub-Agents | Multi-agent delegation for complex tasks |

### Minimum viable twin

**Agent Core + Terminal** — a chat loop that replies in your voice. ~50 lines of code. This is iteration 0.

### Recommended for "use it tomorrow"

**Agent Core + Slack + Memory + Scheduler + a capability or two** — a bot that replies in your voice, remembers context, does things on a schedule, and can read your email. This is the sweet spot.

### Full system

All modules. Multi-agent orchestration, full memory engine, workflow scheduler. This is Junior.

---

## How Modules Connect — A Complete Turn

Here's what happens when someone DMs the twin "remind me to email Rahul at 5pm about the roadmap":

```
1. PLATFORM     Slack event arrives → extract thread_ts → session: idle → busy
2. MEMORY       recall("Rahul", "email", "roadmap") → Rahul = PM, discussed roadmap in Q3
3. AGENT CORE   Compose prompt: persona + history + memories + message → claude -p
4. CAPABILITY   LLM calls scheduler tool via MCP: schedule("email Rahul", "17:00")
5. SCHEDULER    Job registered for 5pm today
6. AGENT CORE   LLM returns: "Done — I'll email Rahul at 5 about the roadmap."
7. PLATFORM     Reply posted to Slack thread → session: busy → draining → idle
8. MEMORY       Auto-ingest: [scheduled email to Rahul re: roadmap at 5pm]

--- At 5pm ---

9.  SCHEDULER   Job fires → spawns claude -p with the prompt
10. MEMORY      recall("Rahul", "roadmap") → PM, Q3 context, prefers bullet points
11. AGENT CORE  Compose email with persona + memories → claude -p
12. CAPABILITY  LLM uses Gmail connector to draft email (read-only, saved to file)
13. PLATFORM    Posts confirmation to Slack: "Drafted the roadmap email to Rahul"
14. MEMORY      Auto-ingest: [drafted email to Rahul about roadmap, 5pm June 21]
```

Every module has a role. None of them know about each other's internals — they communicate through simple interfaces.
