# agent-customizations

My personal, reusable customization files for AI coding tools — GitHub Copilot, Claude,
Cursor, and any agent that reads the open `AGENTS.md` standard or `SKILL.md` skills.

The goal: write a rule or skill **once**, keep it version-controlled here, and reuse it
across every project and every AI tool.

## What's inside

| Type | File | Loaded |
|------|------|--------|
| **Always-on instructions** | [`AGENTS.md`](./AGENTS.md) | In context for **every** request |
| **On-demand skills** | [`.github/skills/<name>/SKILL.md`](./.github/skills) | Discovered by description, body loaded **when relevant** |

> **Key difference:** `AGENTS.md` is read on every turn, so keep it short. Skills stay out
> of context until the agent decides they're relevant — that's where detailed, task-specific
> knowledge belongs.

## Layout

```
agent-customizations/
├── AGENTS.md                       # Always-loaded global instructions
├── .github/
│   └── skills/
│       └── java-spring-backend/
│           └── SKILL.md            # On-demand Java/Spring/JUnit skill
├── LICENSE
└── README.md
```

## How to use it in your projects

Pick whichever fits your tool. Symlinking keeps every project in sync with this repo.

### Option A — Per project (recommended for teams)
Copy or symlink the files into the project you're working on:

```bash
# from inside a target project
ln -s /Users/NREDDY5/Documents/VSCode/agent-customizations/AGENTS.md ./AGENTS.md
ln -s /Users/NREDDY5/Documents/VSCode/agent-customizations/.github/skills ./.github/skills
```

### Option B — Personal, all projects (Copilot / Claude / agents)
Skills are also discovered from personal folders. Symlink a skill into the right home dir:

```bash
mkdir -p ~/.copilot/skills ~/.claude/skills ~/.agents/skills
ln -s /Users/NREDDY5/Documents/VSCode/agent-customizations/.github/skills/java-spring-backend ~/.copilot/skills/java-spring-backend
```

| Tool | Personal skills folder |
|------|------------------------|
| GitHub Copilot | `~/.copilot/skills/<name>/` |
| Claude | `~/.claude/skills/<name>/` |
| Generic agents | `~/.agents/skills/<name>/` |

## Adding a new skill

1. Create `.github/skills/<lowercase-hyphenated-name>/SKILL.md`.
2. The `name:` in the frontmatter **must match the folder name**.
3. Write a keyword-rich `description:` — it's the only part always read, so trigger words
   here determine whether the skill ever gets loaded.
4. Keep `SKILL.md` focused; put long material in a `references/` subfolder.

## Conventions

- Skill folder + `name` field are lowercase, hyphenated, and match exactly.
- Descriptions follow the "What it does. Use when…" pattern with concrete keywords.
- One skill = one coherent job. Split unrelated concerns into separate skills.
