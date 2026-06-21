# Troubleshooting

> Stuck? Start here.

---

**Your AI is your debugger.** Almost anything that breaks, you fix the same way: copy the
**exact error** and paste it to Claude Code and ask it to fix it. It wrote the code; it can
fix the code.

## The move that fixes most things

> Paste the full error to Claude Code — include context:
> *"I ran `node core.js` and got this error. I was trying to [what you were doing].
> I just changed [what you changed]. Here's the full error: [paste the full stack trace]"*

The more context you give, the faster the fix. Then run it again. Repeat if needed. This
back-and-forth **is** building.

---

## Common JavaScript errors — what they mean

| Error | What it means |
|-------|---------------|
| `Cannot read property 'x' of undefined` | You're accessing `.x` on something that doesn't exist yet. The variable is `undefined` when this line runs. |
| `Module not found: 'xyz'` | Package not installed. Run `npm install xyz`. |
| `X is not a function` | You're calling something that isn't callable. Check your imports — did you import the right thing? |
| `Unexpected token` | Syntax error — a missing bracket, comma, or typo near the line mentioned. |
| `SyntaxError: Cannot use import statement` | Your `package.json` is missing `"type": "module"`. Add it. |
| `ERR_MODULE_NOT_FOUND` | The file path in your `import` is wrong, or you forgot the `.js` extension. ESM requires full paths. |

---

## Core build (iterations 0-2)

**`claude -p` says "command not found"**
Claude Code isn't installed or isn't on PATH. Run `claude --version` to check. If it fails,
go back to [Before You Arrive](02-before-you-arrive.md) and install it.

**`claude -p "say hello"` doesn't reply / hangs**
You're not signed in. Run `claude` (without `-p`) and complete the login flow, then try again.

**"Replies stop" / "usage limit reached"**
Switch to Sonnet — type `/model sonnet` in Claude Code. Keep test messages short. Limits
refill over time, so take a short break if needed.

**Exports not working — `import { askTwin }` fails**
Make sure your `package.json` has `"type": "module"`. Without it, Node.js treats `.js` files
as CommonJS and `import`/`export` won't work.

**Twin doesn't sound like you**
That's not a code bug — tweak the prompt. Tell Claude Code *"too formal"* or *"shorter
replies"* or *"more sarcasm"* and let it update PERSONA.md.

---

## Slack

**"An API error occurred: invalid_auth"**
Your bot token is wrong or expired. Go to **api.slack.com/apps** → your app → **OAuth &
Permissions** → copy the **Bot User OAuth Token** (starts with `xoxb-`) again. Paste it
into `.env`.

**Bot is running but doesn't reply in Slack**
Three things to check:
1. Did you `/invite @YourBot` into the channel?
2. Are you @mentioning it (or DMing it)? It won't reply to general chatter.
3. Are the event subscriptions set up? Check **Event Subscriptions** → `app_mention` and
   `message.im` should be listed.

**"Address already in use" (port conflict)**
Something's already running on that port. Stop the other process (Ctrl+C in its terminal),
or ask Claude Code to *"run it on a different port."*

**Bot replies but sounds generic / not in my voice**
The Slack module uses the same brain you built. If the voice is off, the fix is in PERSONA.md,
not in the Slack code. Tell Claude Code what to change.

---

## Capabilities (MCP / Connectors)

**Gmail tools not available — "mcp__claude_ai_Gmail__ not found"**
The Gmail connector isn't connected. Go to
[claude.ai/customize/connectors](https://claude.ai/customize/connectors) and connect Gmail.
This is web-only auth — it won't appear under `/mcp` in the terminal.

**MCP server not connecting**
Run `claude mcp list` to see what's configured. If the server isn't listed, re-run
`claude mcp add <name> npx <package>`. If it's listed but failing, check that `npx` can
run the package: `npx <package-name> --help`.

**Scheduler isn't firing jobs**
Check three things:
1. Is the scheduler running? You should see `Scheduler active` in the startup log. If not,
   make sure `startScheduler()` is called in your startup.
2. Is the job in `data/jobs.json`? If not, the skill didn't create it — try again.
3. Is the time right? Jobs use local time, no timezone suffix. Run `date` to check your
   system clock.

**"Permission denied" / tool blocked when running via Slack bot**
Your `claude -p` call needs `--permission-mode bypassPermissions` since the bot runs
unattended and can't answer permission prompts. Check the spawn args in your bot file.

---

## General

**"command not recognized" / "not found"**
A tool isn't installed, or your terminal needs a restart. **Close and reopen your terminal**
and try again. Still stuck? Paste it to Claude Code.

**Weird characters / emoji errors (Windows)**
Tell Claude Code: *"make the code handle Unicode correctly on Windows."* Don't fix it by hand.

**Everything was working, now it's broken**
Did you install a new module? Restart the twin — modules wire into startup, and a running
process won't pick up the changes until restarted.

---

## Emergency — roll back to your last checkpoint

If everything is broken and you can't figure out what went wrong, roll back to your last
working commit:

```bash
# See your checkpoints
git log --oneline

# Find the last one that was working (e.g. "feat: iteration 2 memory")
# Copy its commit ID, then:
git checkout <commit-id>
git checkout -b attempt-2
```

You're back to working. Try again from there. This is why we checkpoint after every
iteration — you always have a safe point to return to.

---

**Still stuck?** Grab a mentor.
