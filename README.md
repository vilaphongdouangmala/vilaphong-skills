# vilaphong-skills

My personal [Claude Code](https://docs.claude.com/en/docs/claude-code) configuration, shared publicly. Two things live here:

- **[`CLAUDE.md`](#claudemd--global-guidelines)** — behavioral guidelines Claude follows in every project.
- **[`skills/`](#skills)** — installable agent skills.

Take whichever you want; the install steps for each are below.

---

## `CLAUDE.md` — global guidelines

Behavioral instructions that reduce common LLM coding mistakes (think before coding, keep changes surgical, prefer simplicity). Adapted from [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills/blob/main/CLAUDE.md), with one added section on reading GitLab MCP attachments (§5).

Claude Code automatically reads `~/.claude/CLAUDE.md` and merges it into **every** project on your machine. Set it up once:

```bash
# Optional: back up an existing global file first
cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.backup 2>/dev/null || true

# Install
mkdir -p ~/.claude
curl -fsSL https://raw.githubusercontent.com/vilaphongdouangmala/vilaphong-skills/main/CLAUDE.md \
  -o ~/.claude/CLAUDE.md
```

Open Claude Code in any project and it now follows these instructions. Re-run the `curl` to update later.

---

## Skills

Reusable [agent skills](https://docs.claude.com/en/docs/claude-code/skills) under [`skills/`](./skills). Each is a directory with a `SKILL.md` that Claude loads on demand when your request matches it.

### Available

| Skill | What it does |
|-------|--------------|
| [`upward-comms`](./skills/upward-comms) | Rewrite engineer-to-engineer content for engineering-org leadership (VPs, directors, PMs, release managers) and shape it for the channel: ticket comment, chat post, async standup, email, or meeting talking-points. Platform-agnostic, draft-only. |

### Install

Copy a skill directory into your Claude Code skills folder — `~/.claude/skills/` for every project, or `<project>/.claude/skills/` for one:

```bash
git clone https://github.com/vilaphongdouangmala/vilaphong-skills.git
cp -R vilaphong-skills/skills/upward-comms ~/.claude/skills/
```

Start a new Claude Code session and the skill is available.
