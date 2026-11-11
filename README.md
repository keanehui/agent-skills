# agent-skills

Reusable AI agent skills following the open [Agent Skills](https://agentskills.io) (`SKILL.md`) specification. An agent skill is a folder containing a `SKILL.md` file that compatible coding agents (Antigravity, Claude Code, Codex, Cursor, Gemini CLI, and others) discover by metadata and load on demand when your request matches its description.

**Contents:** [Quick start](#quick-start) · [Available skills](#available-skills) · [Installation](#installation) · [Usage](#usage) · [Updating and removing](#updating-and-removing) · [Contributing](#contributing) · [License](#license)

## Quick start

```bash
npx skills add keanehui/agent-skills
```

Follow the prompts to choose skills, target agents, and project vs. global scope, then start a new session so your agent discovers the installed skills.

## Available skills

| Skill | Description |
| :--- | :--- |
| [`code-review`](./skills/code-review/SKILL.md) | General-purpose code review: change-impact (ripple effect) check, dead-code and stale-doc cleanup, test adequacy, and production-readiness audit. |

## Installation

### Option 1: Skills CLI (recommended)

Requires Node.js (`npx`). Lists, installs, and links skills across supported agents:

```bash
# Preview available skills without installing
npx skills add keanehui/agent-skills --list

# Install a specific skill globally across all projects
npx skills add keanehui/agent-skills --skill code-review --global
```

For SSH-only setups: `npx skills add git@github.com:keanehui/agent-skills.git`

### Option 2: Ask your agent

Paste a prompt into your coding agent (Antigravity, Claude Code, Codex, Cursor, Gemini CLI, etc.):

```text
Install all skills from https://github.com/keanehui/agent-skills globally.
```

Or for a single skill in your current project only:

```text
Install the code-review skill from https://github.com/keanehui/agent-skills into this project.
```

### Option 3: Manual setup (clone and link)

Clone the repository and copy or symlink any `skills/<skill-name>/` directory into the target path for your agent:

```bash
git clone https://github.com/keanehui/agent-skills.git
```

| Agent | Project scope | Global scope |
| :--- | :--- | :--- |
| Antigravity | `.agents/skills/<skill-name>` | `~/.gemini/config/skills/<skill-name>` |
| Claude Code | `.claude/skills/<skill-name>` | `~/.claude/skills/<skill-name>` |
| Codex | `.agents/skills/<skill-name>` | `~/.agents/skills/<skill-name>` |
| Cursor | `.agents/skills/<skill-name>` or `.cursor/skills/<skill-name>` | `~/.agents/skills/<skill-name>` or `~/.cursor/skills/<skill-name>` |
| Gemini CLI | `.agents/skills/<skill-name>` or `.gemini/skills/<skill-name>` | `~/.agents/skills/<skill-name>` or `~/.gemini/skills/<skill-name>` |

Symlinking (e.g. `ln -s /absolute/path/to/agent-skills/skills/code-review <target-skills-dir>/`) keeps your installed skills in sync when you run `git pull`.

**Antigravity: register the whole clone** — instead of linking individual skills, you can register `<clone>/skills` in `~/.gemini/config/skills.json` (merging into `"entries"` if the file already exists) using a **full absolute path** (`~/` is not expanded):

```json
{
  "entries": [
    {
      "path": "/absolute/path/to/agent-skills/skills"
    }
  ]
}
```

## Usage

Start a new session after installing so your agent loads the skill metadata.

Your agent triggers `code-review` automatically when you ask it to review code or diffs, audit code quality, trace ripple effects, clean up dead code or stale docs, assess test adequacy, or verify release readiness (it avoids triggering on routine feature work or isolated bug fixes unless you ask for a broader review). Example prompt:

```text
Review my uncommitted changes for ripple effects, dead code, stale docs, and missing tests.
```

## Updating and removing

- **Installed via `npx skills`:**

  ```bash
  npx skills check
  npx skills update
  npx skills remove code-review --global   # omit --global for project installs
  ```

- **Installed via clone / symlink / `skills.json`:** run `git pull` inside your clone to update; to uninstall, delete the copied folder or symlink (or remove its entry from `skills.json`).

## Contributing

Each skill lives in its own `kebab-case` directory under `skills/`:

```text
skills/
└── <skill-name>/
    ├── SKILL.md       # Required: YAML frontmatter + instructions
    ├── scripts/       # Optional: helper executables (none used yet)
    ├── references/    # Optional: supporting docs loaded on demand (none used yet)
    └── assets/        # Optional: templates, schemas, or static data (none used yet)
```

**Required by the [Agent Skills](https://agentskills.io) specification:**

- `name` in `SKILL.md` frontmatter: lowercase alphanumeric and hyphens (`<= 64` characters), matching `<skill-name>`.
- `description` in `SKILL.md` frontmatter: non-empty (`<= 1024` characters) describing what the skill does and when to trigger it.
- Optional frontmatter fields (`license`, `compatibility`, `metadata`) may be included when needed.

**Repository conventions:**

- Keep skills portable: never hardcode personal paths, usernames, credentials, or machine-specific assumptions.
- Keep instructions tool-, OS-, and language-agnostic unless a skill explicitly targets a specific stack.
- Keep `SKILL.md` focused; move bulky reference material into `references/` and link with relative paths.
- Update the [Available skills](#available-skills) table when adding or renaming a skill.

## License

[MIT](./LICENSE)
