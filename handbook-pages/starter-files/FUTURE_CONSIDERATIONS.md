# Future Considerations — Digital Twin Buildathon

Ideas and questions parked for future iterations. Not part of v0.

---

## Personality Refinement via Real Texts

Dropped from the v0 interview to keep it fast (all tick-box, 15 min). This is the highest-signal
input for voice accuracy — bring it back in a future iteration once the twin is running and users
can feel where it sounds off.

**Proposed question (for a future "refine your twin" flow):**

```
Paste 5+ actual messages you've sent in any chat — group chats, DMs, whatever.
The more variety the better: a funny one, a serious one, a casual one.

> 1.
> 2.
> 3.
> 4.
> 5.
```

**Why it matters:** Tick-box answers capture *categories* of voice. Real texts capture *rhythm* —
sentence length, trailing punctuation, word choice, how someone starts and ends messages.
Two people with identical tick-box answers can text completely differently. This question closes that gap.

**When to introduce:** After v0, when users have used their twin and noticed it doesn't quite
sound like them. Frame it as "let's fine-tune" rather than front-loading the interview.

---

## Voice Output

Let the twin *speak* its replies using a text-to-speech API.

**Options:**
- **Sarvam AI** — Indian-language TTS with natural Hindi/English voices. API-based.
- **OpenAI TTS** — high-quality, multiple voices, simple API (`tts-1` / `tts-1-hd`).
- **Gemini TTS** — Google's text-to-speech via the Gemini API. Supports many languages.

**How it would work:** Pipe the twin's text reply through the TTS engine, play the audio
locally or send it as a voice message on Slack/Discord. Opt-in, not default — text replies
should always work without TTS installed.

**When to introduce:** After the twin's text voice is dialed in. Voice amplifies whatever
the text sounds like — if the text is generic, the voice will sound generic. Get the
personality right first.

---

## Module Ideas (post-buildathon)

Capabilities participants could build after the event, using the MCP + Skills patterns
they learned:

- **WhatsApp adapter** — same brain, new platform (via Baileys or Twilio)
- **Notion integration** — read/write Notion pages, query databases
- **GitHub assistant** — summarize PRs, draft review comments in your voice
- **Meeting prep** — before a calendar event, pull context from email + Slack + memory and
  draft a briefing
- **Daily journal** — scheduled nightly prompt that asks the twin to summarize what happened
  today, saved to a markdown file
- **Voice cloning** — train a TTS model on your actual voice recordings
- **Multi-platform sync** — same twin on Slack + Discord + Telegram, shared memory across all
