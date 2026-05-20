# priority-check

A grounding tool for people who lose hours to FOMO, ADHD-driven hyperfocus, or just having too much fun on the wrong thing.

It rates your current Claude conversation or project as **High**, **Medium**, or **Low** priority against tiers you define -- then prepends a single line to the response so you stay aware of where your attention is going without breaking your flow.

```
Priority: Low -- creative side project, unrelated to your high-priority areas.
```

That's it. One line. A signal, not a blocker.

---

## Who it's for

- People with ADHD or FOMO-driven attention drift who want a lightweight external check
- Knowledge workers and freelancers where time discipline directly affects output
- Anyone who has ever looked up from a side project at 1am and felt regret
- Users who want their AI assistant to surface trade-offs rather than silently enable drift

---

## How it works

On first use, the skill walks you through a short setup:

1. What topics are **High** priority for you?
2. Any **Medium** priority topics? (optional -- Claude infers if not)
3. Any **Low** priority topics you tend to over-invest in? (optional -- Claude infers if not)
4. Run automatically at the start of every new chat? (yes/no)

It then generates a settings block for you to paste into your Claude preferences. After that, it runs silently against those tiers whenever invoked.

For topics not in your list, the skill infers the tier: topics related to your High areas are Medium, everything else is Low. Universal-importance topics (legal, medical, financial, family) always default to High regardless of your configuration.

---

## Commands

| Invocation | What happens |
|------------|-------------|
| `priority-check` | Rates the current conversation (default) |
| `priority-check chat` | Same as above, explicit |
| `priority-check project` | Rates the current Claude project, with a trade-off note |
| `priority-check help` | Shows your current settings and command reference |
| `priority-check reset` | Re-runs the questionnaire to update your tiers |

Aliases also work: `fomo check`, `priority check`, `is this worth my time`.

---

## Settings block

After setup, your preferences will contain a block like this:

```
=====
%%PRIORITY CHECK SETTINGS v1%%
high: career, software engineering, parenting, health
medium: fitness, home management, personal finance, cooking
low: gaming, creative side projects, random rabbit holes
run_on_chat_start: true
At the start of any substantive new conversation, invoke the priority-check skill to rate the topic.
=====
```

You can edit this manually at any time, or run `priority-check reset` to regenerate it via the questionnaire.

---

## Installing

### Claude.ai (web/desktop)

1. Download this repo as a ZIP: **Code → Download ZIP**
2. In Claude.ai: **Settings → Customize → Skills → Upload**
3. Upload the ZIP

### Claude Code

```bash
gh skill install mvsd/mvsd-ai-skills priority-check --agent claude-code --scope user
```

Or manually:

```bash
mkdir -p ~/.claude/skills/priority-check
curl -o ~/.claude/skills/priority-check/SKILL.md \
  https://raw.githubusercontent.com/mvsd/mvsd-ai-skills/main/skills/priority-check/SKILL.md
```

Restart your Claude Code session after installing.

---

## Compatibility

Built for Claude.ai and Claude Code. Uses the open SKILL.md standard and should work with other SKILL.md-compatible agents (OpenClaw, Codex CLI, etc.) though these are untested.

---

## Known limitations

**Settings require a manual paste step.** After setup or reset, the skill generates 
a config block you paste into your Claude preferences. A future version aims to 
remove this friction. See [issue #1](../../issues/1) for status.

## License

MIT