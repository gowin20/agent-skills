# Agent Skills

A local collection of [Agent Skills](https://agentskills.io/specification): folders of instructions, scripts, and resources that AI agents can discover and load on demand to perform specialized tasks.

## Repository layout

```text
agent-skills/
├── README.md            # This file
├── CONTRIBUTING.md      # Conventions for adding and editing skills
├── LICENSE              # MIT License, covering the whole repository
├── .claude-plugin/
│   ├── marketplace.json # Marketplace catalog (lists this repo as one plugin)
│   └── plugin.json      # Plugin manifest (name, version, metadata)
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
| [iiif-image-api-2](skills/iiif-image-api-2/SKILL.md) | IIIF Image API 2.0: image request URLs, `info.json`, tiling, and compliance levels 0-2. |
| [iiif-presentation-api-3](skills/iiif-presentation-api-3/SKILL.md) | IIIF Presentation API 3.0: Manifests, Canvases, Ranges, and painting/supplementing annotations, with an example Manifest. |
| [w3c-web-annotation-model](skills/w3c-web-annotation-model/SKILL.md) | W3C Web Annotation Data Model: bodies, targets, SpecificResource, selectors, states, motivations, and collections. |
| [strunk-white-style](skills/strunk-white-style/SKILL.md) | Writing, editing, and critiquing English prose by the rules of Strunk and White's *The Elements of Style*. |

## Installing as a plugin

This repository is a plugin marketplace. It contains a single plugin, `gowin20-agent-skills`, which bundles every skill in `skills/`. The format follows Claude Code's plugin layout, which VS Code and the GitHub Copilot CLI also read:

| File | Purpose |
|---|---|
| [.claude-plugin/marketplace.json](.claude-plugin/marketplace.json) | Marketplace `gowin20-skills`, with one plugin entry whose `source` is `./` (the repository root) |
| [.claude-plugin/plugin.json](.claude-plugin/plugin.json) | Plugin manifest. Skills are found automatically in the default `skills/` folder. |

### VS Code (agent plugins)

1. Add the marketplace in your **user** `settings.json` (**Chat › Plugins: Marketplaces**). Use either the GitHub repository or a local clone:

   ```jsonc
   "chat.plugins.marketplaces": [
     "gowin20/agent-skills",                 // GitHub (append #<branch|tag|sha> to pin)
     "file:///E:/Projects/agent-skills"      // or a local clone on disk (must be a git repo)
   ]
   ```

   You can also open the Command Palette, run **Chat: Open Customizations**, and go to **Plugins › Browse › Configure Marketplaces › Add Marketplace**.

2. Install the plugin: search `@agentPlugins` in the Extensions view, or go to **Chat: Open Customizations › Plugins › Browse**. Select **gowin20-agent-skills**, choose **Install**, and accept the trust prompt.

To load the plugin straight from a folder without a marketplace, register it with `chat.pluginLocations` (experimental):

```jsonc
"chat.pluginLocations": { "E:/Projects/agent-skills": true }
```

Plugin support requires `chat.plugins.enabled`. VS Code marks marketplace and local-plugin settings as experimental, and their names have changed between releases (for example, `chat.plugins.paths` became `chat.pluginLocations`).

### Claude Code

```bash
claude plugin marketplace add gowin20/agent-skills     # or a local path, e.g. E:/Projects/agent-skills
claude plugin install gowin20-agent-skills@gowin20-skills
```

### Validating the manifests

```bash
claude plugin validate .                              # marketplace.json
claude plugin validate .claude-plugin/plugin.json     # plugin.json
```

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
7. Bump `version` in [.claude-plugin/plugin.json](.claude-plugin/plugin.json) so installed copies of the plugin pick up the change. Update the plugin `description` in both manifests if it lists skills.

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
