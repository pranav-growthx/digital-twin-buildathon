# Troubleshooting

> **Help** — Stuck? Start here.

---

**Your AI is your debugger.** Almost anything that breaks, you fix the same way: copy the **exact error** and paste it to Claude — in your build session, or at [claude.ai](https://claude.ai) — and ask it to fix it. It wrote the code; it can fix the code.

## The move that fixes most things

> Paste this to your AI, with the error:
> *"I ran this and got an error — fix it: [paste the full error message]"*

Then run it again. Repeat if it's not fixed the first time. Stay calm — this back-and-forth **is** building.

---

## A few specifics worth knowing

**"Address already in use" (a port is busy)**
Something's already running on that port. Stop the other one (press **Ctrl+C** in its terminal), or ask your AI to *"run it on a different port."*

**Weird characters / emoji error (Windows)**
Don't fix this by hand — tell your AI: *"make the code print emojis correctly on Windows."* It'll handle it in the code.

**"command not recognized" / "not found"**
Usually a tool isn't installed yet, or your terminal needs a restart. **Close and reopen your terminal** and try again. Still stuck? Paste it to Claude.

**Replies stop / "usage limit reached"**
Switch to Sonnet — type **`/model sonnet`** — and keep test messages short. Limits refill over time, so take a short break if needed.

---

**Doesn't sound like you?** That's not a bug — open the [Demo](06-demo.md) step and tune the personality.

**Compare with a working build.** If you're really stuck — especially on Slack — open the reference build and borrow what you need: [github.com/luvishg-growthx/buildathon-build-your-personal-ai-agent](https://github.com/luvishg-growthx/buildathon-build-your-personal-ai-agent)

**Still stuck?** Grab a volunteer.
