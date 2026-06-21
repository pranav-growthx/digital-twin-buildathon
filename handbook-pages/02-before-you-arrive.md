# GrowthX Buildathon — Laptop Setup

> **Do this _before_ you arrive — about 15 minutes.**
> The event is 3 hours of *building*, not installing. If your laptop is ready the night before, you can start the moment you sit down. Stuck on a step? Come 15 minutes early and a mentor will help.

---

## What you'll need

- A **laptop** (Windows or Mac) where you can install software (admin rights).
- An **AI subscription** you can log in to — one of:
  - **Claude Pro / Max** — *recommended* (powers **Claude Code**)
  - **ChatGPT Plus / Pro** (powers **Codex**)
- About **15 minutes**.

You'll set up five things, in order:

**Node.js → an IDE → your AI tool → Python → Git**

---

## Step 1 — Install Node.js

> Your AI tool is installed and runs through Node.js.

**Windows**
1. Go to **[nodejs.org](https://nodejs.org)** and download the **LTS** version.
2. Run the installer and click through (Next → Next → Finish).

Prefer the terminal?
```powershell
winget install OpenJS.NodeJS.LTS
```

**macOS**
1. Go to **[nodejs.org](https://nodejs.org)** and download the **macOS LTS** (`.pkg`).
2. Run the installer.

Have [Homebrew](https://brew.sh)?
```bash
brew install node
```

---

## Step 2 — Install your Code-Editor (your workspace)

> An IDE is the app where you'll see your files and run commands. It has a built-in **terminal** you'll use in the next step.

Pick **one** (each site has Windows & macOS downloads):

| IDE | Notes |
|-----|-------|
| **[VS Code](https://code.visualstudio.com)** | Recommended — free, simple, works everywhere |
| **[Google Antigravity](https://antigravity.google)** | AI-first IDE |
| **[Cursor](https://cursor.com)** | AI-first IDE |

> Not sure? **Choose VS Code.** Antigravity and Cursor are also built on VS Code, so the steps below are identical. (They have their own built-in AI, but for this build everyone uses Claude Code in the terminal.)

Download your chosen IDE, install it, and **open it**.

---

## Step 3 — Install your AI tool (in the IDE terminal)

**1. Open the built-in terminal** inside your IDE:
- Menu: **Terminal → New Terminal**, or
- Shortcut: **Ctrl + `** (Windows) / **Cmd + `** (macOS) — the key above Tab.

A terminal panel opens at the bottom. **2. Type the command for your tool:**

**Claude Code** (recommended)
```bash
npm install -g @anthropic-ai/claude-code
```

**Codex** (alternative)
```bash
npm install -g @openai/codex
```

> If it says **"npm is not recognized"**, close the IDE and reopen it (so it picks up Node from Step 1), then open the terminal again.

---

## Step 4 — Log in to your AI tool

In the same terminal:

**Claude Code** — run `claude`, then sign in with your Claude account in the browser window that opens. Type `/exit` when done.
```bash
claude
```

**Codex**
```bash
codex login
```

---

## Step 5 — Install Python

> The default build uses Python. *Building in JavaScript instead? Node from Step 1 is enough — skip this step.*

**Windows**
1. Go to **[python.org/downloads](https://www.python.org/downloads/)** and download Python.
2. Run the installer — **tick "Add Python to PATH"** at the bottom — then click **Install Now**.

**macOS**
1. Go to **[python.org/downloads](https://www.python.org/downloads/)** and run the macOS installer.

Have Homebrew?
```bash
brew install python
```

---

## Step 6 — Install Git

> During the build you'll "checkpoint" your work so you never lose it. Git makes that possible.

**Windows**
1. Go to **[git-scm.com/download/win](https://git-scm.com/download/win)**, download, and run the installer (accept the defaults).

Prefer the terminal?
```powershell
winget install Git.Git
```

**macOS**
1. Run the line below. If Git isn't installed, macOS pops up an **Install** button — click it.
```bash
git --version
```

Have Homebrew?
```bash
brew install git
```

---

## Step 7 — Verify everything works

In your IDE terminal, run these — each should print a version, not an error:

**Windows**
```powershell
node --version
python --version
git --version
claude --version
```

**macOS**
```bash
node --version
python3 --version
git --version
claude --version
```

> Codex users: use `codex --version` instead of `claude --version`.

**Final test — can your AI tool answer?**

```bash
claude -p "say hello in one word"
```
Codex:
```bash
echo "say hello in one word" | codex exec -
```

A one-word reply means you're ready for the build.

---

## If something goes wrong

- **"not recognized" / "command not found"** → close your IDE and reopen it (the terminal picks up new installs), then try again.
- **Windows: Python still not found** → reinstall it and make sure **"Add Python to PATH"** is ticked.
- **Login problems** → run `claude` (or `codex login`) again and finish the browser sign-in.
- **Still stuck?** Come 15 minutes early on event day — we will get you set up quickly.
