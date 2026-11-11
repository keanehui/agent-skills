# agent-skills

Personal collection of reusable AI agent skills following the [Agent Skills](https://agentskills.io) `SKILL.md` specification.

## Available Skills

| Skill | Description | Source |
| :--- | :--- | :--- |
| `code-review` | General-purpose code review: ripple-effect checks, dead-code/stale-doc cleanup, test adequacy, and production-readiness audit. | [`skills/code-review/SKILL.md`](./skills/code-review/SKILL.md) |

## Repository Layout

Add new skills as `kebab-case` subdirectories under `skills/`:

```text
skills/
└── <skill-name>/
    ├── SKILL.md       # Required: frontmatter (name, description) + instructions
    ├── scripts/       # Optional: helper executables
    └── references/    # Optional: supporting docs loaded on demand
```

Keep `name` in `SKILL.md` frontmatter identical to the directory name.

## Usage

### Antigravity

Register `skills/` in `~/.gemini/config/skills.json` so all skills in this repository load automatically:

```json
{
  "entries": [
    {
      "path": "~/Developer/agent-skills/skills"
    }
  ]
}
```

### Skills CLI

```bash
npx skills add keanehui/agent-skills
```
