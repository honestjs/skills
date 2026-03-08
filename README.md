# HonestJS Skills

Agent skills for the HonestJS ecosystem. These are designed for Cursor and other agents that support the SKILL.md format (e.g. `~/.cursor/skills/`, `~/.agents/skills/`).

## Layout

Each skill lives in its own directory named after the skill. Structure matches the [Hono skill](https://github.com/honojs/hono/tree/main/docs/skills) pattern:

```
skills/
├── README.md           # This file
└── honest/
    └── SKILL.md        # Honest.js framework skill
```

- **honest** — Build and maintain Honest.js apps: CLI, Application.create, decorators, modules, DI, pipeline (middleware, guards, pipes, filters), and ecosystem packages (middleware, pipes, class-validator-pipe, rpc-plugin, http-essentials).

## Using these skills

**Cursor (project):** Copy or symlink the skill directory into your project’s skills folder:

```bash
# From repo root
mkdir -p .cursor/skills
cp -r skills/honest .cursor/skills/
```

**Cursor (user):** Copy into your user skills directory:

```bash
cp -r skills/honest ~/.cursor/skills/
```

**Claude Code / agents:** Copy into the agents skills directory:

```bash
cp -r skills/honest ~/.agents/skills/
```

The agent will then have access to the skill when working with Honest, HonestJS, or related packages.

## Adding or editing skills

- One skill per directory; directory name = skill identifier (lowercase, hyphens).
- Required file: `SKILL.md` with YAML frontmatter (`name`, `description`) and markdown body.
- Keep `SKILL.md` under ~500 lines; use `references/` for long reference material if needed.
- See [Cursor create-skill](https://docs.cursor.com/skills/create-skill) and the Hono skill for structure and description guidelines.
