# Agent Skills

A local collection of [Agent Skills](https://agentskills.io/specification): folders of instructions, scripts, and resources that AI agents can discover and load on demand to perform specialized tasks.

## Repository layout

```text
agent-skills/
├── README.md            # This file
├── CONTRIBUTING.md      # Conventions for adding and editing skills
├── LICENSE              # MIT License, covering the whole repository
├── skills/              # One sub-folder per skill
│   └── <skill-name>/
│       ├── SKILL.md     # Required: YAML frontmatter + Markdown instructions
│       ├── scripts/     # Optional: executable code
│       ├── references/  # Optional: docs loaded on demand
│       └── assets/      # Optional: templates, images, data files
└── debug/               # Git-ignored scratch space for local debugging
```

## Skills

| Skill | Description |
|---|---|
| [agent-skills-specification](skills/agent-skills-specification/SKILL.md) | The Agent Skills format: required and optional fields, structure, and a step-by-step guide to creating a new skill. |

## Using these skills

Agents discover skills by scanning known directories for sub-folders that contain a `SKILL.md`. To make these skills available, copy or symlink a skill folder into a location your agent scans, for example:

| Scope | Location |
|---|---|
| Project (cross-client convention) | `<project>/.agents/skills/<skill-name>/` |
| User (cross-client convention) | `~/.agents/skills/<skill-name>/` |
| Claude Code, project | `<project>/.claude/skills/<skill-name>/` |
| Claude Code, user | `~/.claude/skills/<skill-name>/` |

Check your agent's documentation for the exact paths it scans.

## Creating a new skill

Follow the step-by-step guide in [skills/agent-skills-specification/SKILL.md](skills/agent-skills-specification/SKILL.md#how-to-create-a-new-skill). Short version:

1. Create `skills/<skill-name>/` (lowercase letters, digits, and single hyphens).
2. Copy [the SKILL.md template](skills/agent-skills-specification/assets/SKILL.template.md) into it as `SKILL.md`.
3. Set `name` to match the folder name and write a specific `description` saying what the skill does and when to use it.
4. Set `license: MIT` (see [License](#license)).
5. Write the instructions, then validate with `skills-ref validate skills/<skill-name>`.
6. Add the skill to the table above.

## Validation

Use the reference validator from the [agentskills repository](https://github.com/agentskills/agentskills/tree/main/skills-ref):

```bash
skills-ref validate skills/<skill-name>
```

## The `debug/` folder

`debug/` is listed in `.gitignore`. Use it for test runs, eval outputs, and other local scratch files that should not be committed.

## License

This repository is licensed under the [MIT License](LICENSE).

As a general guideline, every skill created in this repository is also MIT-licensed. Declare it in each skill's frontmatter:

```yaml
license: MIT
```

Third-party material bundled in a skill (scripts, templates, reference docs) must be compatible with MIT. If a skill needs a different license, include its own `LICENSE.txt` in the skill folder and point to it, for example `license: Apache-2.0. LICENSE.txt has complete terms`.
