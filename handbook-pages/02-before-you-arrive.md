# Before You Arrive

> **Setup**

Install everything tonight — about 15 minutes.

---

Spend about **15 minutes tonight** so you arrive ready to build. You only need two things: **Node.js** and **one AI tool** (Claude Code recommended). Once Claude is installed, it can set up anything else for you on the day — so this list is short on purpose.

> You don't need to understand all of it. Install it, run the check at the bottom, and you're set. Mentors will fix anything that didn't work tomorrow.

> Use the **Tool** and **Computer** sections below to match what you have.

## 1. Install Node.js

### Windows

1. Go to **nodejs.org** and download the **LTS** version.
2. Run the installer (keep clicking Next).
3. Open your **terminal (PowerShell)** — you'll use it in the next step.

### macOS

1. Go to **nodejs.org**, download the **LTS** version, and install it.
2. Open the **Terminal app** — you'll use it in the next step.

## 2. Install your AI tool

Install **Claude Code first** — once it's running and signed in, it can install everything else you need (even Python) for you.

### Claude Code

In your terminal:

```bash
npm install -g @anthropic-ai/claude-code
```

Then start it and sign in with your Claude account:

```bash
claude
```

Sign in when it asks, then type `/exit` to return to the terminal.

### Codex (alternative)

In your terminal:

```bash
npm install -g @openai/codex
```

Then sign in:

```bash
codex login
```

## 3. Python (optional)

You only need Python if you choose the Python path tomorrow — and your AI tool can install it for you once it's running. **You can skip this for now.**

## Final check

In your terminal, confirm Node is installed:

```bash
node --version
```

You should see a version like `v20.x` (not an error). Then check your AI tool replies:

**Claude Code:**

```bash
claude -p "say hi in one word"
```

**Codex:**

```bash
echo "say hi in one word" | codex exec -
```

> Got a version number and a one-word reply? You're ready — see you tomorrow.

> We may share a one-step setup script on the day, but doing this tonight means you can start building right away.
