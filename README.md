# mvsd-ai-skills

A collection of AI skills for Claude and other SKILL.md-compatible agents. Each skill is a focused, self-contained workflow tool built around a specific problem worth solving repeatedly.

Skills are designed to be minimal and purposeful.

---

## Available skills

| Skill | Description | Compatible with |
|-------|-------------|-----------------|
| [priority-check](./skills/priority-check/) | Rates your current chat or project against your personal priority tiers. A grounding signal for people with ADHD, FOMO, or a tendency to over-invest in low-value work. | Claude.ai, Claude Code |

---

## Installing a skill

### Claude.ai (web/desktop)

1. Navigate to the skill folder (e.g. `skills/priority-check/`)
2. Open `SKILL.md` and download the raw file: **Raw → right-click → Save As**
3. In Claude.ai: **Settings → Customize → Skills → Upload**
4. Upload the `SKILL.md` file

### Claude Code (terminal)

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

## License

MIT — free to use, adapt, and share.