# Contributing

## Conventions

- **One skill per folder** under `skills/`. The folder name must equal the skill's `name` field.
- **Names**: 1-64 characters, lowercase `a-z`, `0-9`, and hyphens only; no leading, trailing, or consecutive hyphens.
- **Keep `SKILL.md` lean**: under 500 lines and about 5,000 tokens. Move detailed material into `references/` and say in `SKILL.md` *when* to read each file.
- **Relative paths only** when a skill refers to its own files (for example `references/api.md`, `scripts/run.py`). Keep references one level deep from `SKILL.md`.
- **Line endings**: LF (enforced by `.gitattributes` and `.editorconfig`).
- **Markdown style**: follow `.markdownlint.json`. Line length is not enforced.
- **No secrets** in skills. Read credentials from environment variables at runtime.

## Checklist for a new or changed skill

- [ ] `name` matches the folder name and follows the naming rules
- [ ] `description` is under 1024 characters and says what the skill does **and** when to use it
- [ ] Frontmatter declares `license: MIT` (see the License section in `README.md`)
- [ ] `skills-ref validate skills/<skill-name>` passes
- [ ] Bundled scripts are non-interactive, document `--help`, and pin their dependencies
- [ ] Tested on at least one real task with an agent
- [ ] Skill is listed in the table in `README.md`

## Commits

Use short, imperative commit subjects that name the skill, for example `agent-skills-specification: clarify metadata field`.
