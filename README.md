# agent-skills

Reusable AI agent skills following the open [Agent Skills](https://agentskills.io) (`SKILL.md`) specification. Designed to work across compatible coding agents, including Antigravity, Claude Code, Codex, Cursor, and Gemini CLI.

## Available Skills

| Skill | Description | Source |
| :--- | :--- | :--- |
| `code-review` | General-purpose code review: ripple-effect checks, dead-code/stale-doc cleanup, test adequacy, and production-readiness audit. | [`skills/code-review/SKILL.md`](./skills/code-review/SKILL.md) |

## Installation

### Recommended: Skills CLI (`npx skills`)

Requires Node.js (`npx`). Supports interactive selection of skills, target agents, and project vs. global scope.

```bash
# Preview available skills
npx skills add keanehui/agent-skills --list

# Install interactively (choose skills, agents, and scope)
npx skills add keanehui/agent-skills

# Install a specific skill globally
npx skills add keanehui/agent-skills --skill code-review --global
```

If the repository is private/restricted or you use SSH authentication:

```bash
npx skills add git@github.com:keanehui/agent-skills.git
```

To check for and apply updates later:

```bash
npx skills check
npx skills update
```

### Using an AI Agent

Ask your coding agent (Antigravity, Claude Code, Codex, Cursor, Gemini CLI, etc.) to install the skills for you. For example, to install all skills globally:

```text
Install the skills from https://github.com/keanehui/agent-skills globally.
```

Or to install a single skill into your current project only:

```text
Install the code-review skill from https://github.com/keanehui/agent-skills into this project.
```

### Antigravity (via local clone)

`npx skills add` supports Antigravity directly. If you prefer tracking a local `git clone` so `git pull` updates skills automatically:

- **Global (all projects):** register `<clone>/skills` in `~/.gemini/config/skills.json` using a **full absolute path** (Antigravity does not expand `~/` in `skills.json`). Merge into `"entries"` if the file already exists:

  ```json
  {
    "entries": [
      {
        "path": "/absolute/path/to/agent-skills/skills"
      }
    ]
  }
  ```

  Alternatively, symlink or copy individual skill folders into `~/.gemini/config/skills/`.

- **Per-project:** copy or symlink `skills/<skill-name>` into `<project>/.agents/skills/`, or register it in `<project>/.agents/skills.json`.

Start a new conversation after updating `skills.json` or adding skills so they load into the session context.

### Manual Setup (Any Compatible Agent)

Clone the repository and copy or symlink any `skills/<skill-name>/` directory into your agent's skills directory:

```bash
git clone https://github.com/keanehui/agent-skills.git
```

- **Project scope** (for a single repository):
  - Universal / Antigravity / Codex / Gemini CLI: `.agents/skills/<skill-name>`
  - Tool-specific directories: `.claude/skills/<skill-name>`, `.cursor/skills/<skill-name>`
- **Global scope** (available across all projects):
  - Universal / Codex: `~/.agents/skills/<skill-name>`
  - Antigravity: `~/.gemini/config/skills/<skill-name>`
  - Claude Code: `~/.claude/skills/<skill-name>`
  - Cursor: `~/.cursor/skills/<skill-name>`
  - Gemini CLI: `~/.gemini/skills/<skill-name>`

Using a symlink (`ln -s /absolute/path/to/agent-skills/skills/<skill-name> <target-skills-dir>/`) lets you pull updates with `git pull` instead of re-copying files.

## Repository Layout

```text
skills/
└── <skill-name>/
    ├── SKILL.md       # Required: YAML frontmatter (name, description) + instructions
    ├── scripts/       # Optional: helper executables
    ├── references/    # Optional: supporting docs loaded on demand
    └── assets/        # Optional: templates, schemas, or static data
```

## Contributing a Skill

1. Create a `kebab-case` directory under `skills/<skill-name>/` with a `SKILL.md` file.
2. Start `SKILL.md` with YAML frontmatter following the [Agent Skills](https://agentskills.io) specification:
   - `name` (required): lowercase alphanumeric and hyphens (`<= 64` characters), matching the directory name.
   - `description` (required): clear summary of what the skill does and when to trigger it (`<= 1024` characters).
   - `license`, `compatibility`, `metadata` (optional): include when relevant.
3. Keep skills portable so anyone with access can use them:
   - Never hardcode personal paths, usernames, credentials, or machine-specific assumptions.
   - Keep instructions tool-, OS-, and language-agnostic unless the skill explicitly targets a specific stack.
   - Keep `SKILL.md` concise; move bulky documentation into `references/` and link with relative paths.
4. Update the **Available Skills** table in `README.md`.

## License

[MIT](./LICENSE)
