# agent-skills

Agent skills I use day to day with coding agents (Claude Code, Codex, Pi, and similar).

Each skill is a small, self-contained folder with a `SKILL.md` entrypoint.
Agents that support the [Agent Skills](https://agentskills.io) layout can load these directly.

## Skills

| Skill | When to use it |
| --- | --- |
| [`lightweight`](./lightweight/) | Audit a product for bloat, latency, memory, and bundle size without cutting quality |
| [`spike`](./spike/) | Research a technical topic and write a findings doc before building |
| [`x-post`](./x-post/) | Draft an X/Twitter post (or thread) in a human voice |

### lightweight

Performance and footprint audit skill.
Measures before ranking, prioritizes hot paths, and refuses to treat tests, types, auth, or error handling as fat to trim.
Findings need evidence (`file:line` or a measured metric).

### spike

Short research loop for "should we / can we / how would we" questions.
Investigates code, small experiments, and primary sources, then writes a standalone results Markdown file with findings, options, recommendation, and open questions.

### x-post

Drafts posts like a voice memo typed out - one beat, specific, with a rough edge.
Produces a few ready-to-paste candidates at different energies (dry / playful / blunt).
Voice checklist lives in [`x-post/VOICE.md`](./x-post/VOICE.md).

## Layout

```text
.
├── lightweight/
│   ├── SKILL.md
│   └── references/
├── spike/
│   ├── SKILL.md
│   └── agents/
└── x-post/
    ├── SKILL.md
    └── VOICE.md
```

## Install

Copy or symlink the skill folders into your agent's skills directory, for example:

```bash
# Claude Code / compatible agents
ln -s "$PWD/lightweight" ~/.agents/skills/lightweight
ln -s "$PWD/spike" ~/.agents/skills/spike
ln -s "$PWD/x-post" ~/.agents/skills/x-post
```

Exact paths depend on the agent.
Point the agent at each skill's `SKILL.md`.

## License

MIT
