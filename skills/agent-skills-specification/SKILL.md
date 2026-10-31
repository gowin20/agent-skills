---
name: agent-skills-specification
description: >
  Use this skill when creating, editing, reviewing, or validating an Agent
  Skill (a folder containing a SKILL.md file), or when the user asks what
  agent skills are, how the SKILL.md format works, which frontmatter fields
  are required or optional, or how to organize scripts/, references/, and
  assets/. Covers the full agentskills.io specification (name and description
  rules, license, compatibility, metadata, allowed-tools, progressive
  disclosure, file references, validation with skills-ref) and a step-by-step
  process for building a new skill with a description that triggers reliably.
  Use it even when the user only says "make a skill", "turn this workflow
  into a skill", or "why isn't my skill being picked up".
license: MIT
metadata:
  spec-source: https://agentskills.io/specification
  version: "1.0"
---

# Agent Skills Specification

This skill explains the Agent Skills format and shows how to create a valid, effective skill.

## What an Agent Skill is

An Agent Skill is a **folder that contains a `SKILL.md` file**. It packages instructions, and optionally scripts, reference docs, and assets, that an agent can discover and load when a task calls for them. Anthropic originally developed the format and released it as an open standard ([agentskills.io](https://agentskills.io)). Many agent products can read it.

Skills give an agent:

- **Domain expertise** it does not have by default, such as company conventions, internal APIs, or gotchas.
- **Repeatable workflows**: a fixed procedure followed the same way every time.
- **Reuse across products**: one skill folder works in any compatible agent.

### How agents use skills (progressive disclosure)

Agents load skills in three tiers, so many skills can be installed without filling the context window:

| Tier | What is loaded | When | Budget |
|---|---|---|---|
| 1. Discovery / catalog | `name` + `description` only | At session start, for every installed skill | ~50-100 tokens per skill |
| 2. Activation | The full `SKILL.md` body | When a task matches the description | < 5,000 tokens recommended |
| 3. Resources | Files in `scripts/`, `references/`, `assets/` | Only when the instructions point to them | As needed |

Two things follow from this:

1. **The `description` alone decides whether the skill gets used.** The agent never sees the body until it decides to activate the skill.
2. **Everything in `SKILL.md` costs context on every activation.** Move detail that is only sometimes needed into separate files.

## Directory structure

```text
skill-name/
├── SKILL.md          # Required: YAML frontmatter + Markdown instructions
├── scripts/          # Optional: executable code the agent can run
├── references/       # Optional: extra documentation, read on demand
├── assets/           # Optional: templates, images, data files
└── ...               # Any other files or folders are allowed
```

Only `SKILL.md` is required. `scripts/`, `references/`, and `assets/` are recommended conventions, not requirements.

## SKILL.md format

`SKILL.md` must start with **YAML frontmatter** between `---` lines, followed by a **Markdown body**:

```markdown
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

Instructions go here...
```

## Frontmatter fields

### Summary

| Field | Required | Type | Constraints |
|---|---|---|---|
| `name` | **Yes** | string | 1-64 chars; lowercase `a-z`, `0-9`, `-`; no leading, trailing, or consecutive hyphens; must match the folder name |
| `description` | **Yes** | string | 1-1024 chars; non-empty; says what the skill does **and** when to use it |
| `license` | No | string | License name or a reference to a bundled license file |
| `compatibility` | No | string | 1-500 chars; environment requirements |
| `metadata` | No | map of string → string | Arbitrary extra properties |
| `allowed-tools` | No | string | Space-separated pre-approved tools (**experimental**) |

The specification defines no other fields. Some agent products add their own fields, such as Claude Code's `disable-model-invocation`. Those only work in that product, so put portable extras under `metadata`.

### `name` (required)

- 1-64 characters.
- Only lowercase alphanumerics (`a-z`, `0-9`) and hyphens (`-`).
- Must not start or end with a hyphen.
- Must not contain consecutive hyphens (`--`).
- **Must match the parent directory name.**

| Valid | Invalid | Why invalid |
|---|---|---|
| `pdf-processing` | `PDF-Processing` | Uppercase |
| `data-analysis` | `-pdf` | Leading hyphen |
| `code-review` | `pdf--processing` | Consecutive hyphens |
| `k8s-deploy` | `my_skill` | Underscore |

Prefer short, descriptive names, usually a noun phrase or a gerund (`processing-pdfs`, `release-notes`).

### `description` (required)

- 1-1024 characters, non-empty.
- Must describe **what the skill does** and **when to use it**.
- Include the specific keywords a user would actually say.

```yaml
# Poor: too vague to trigger reliably
description: Helps with PDFs.

# Good
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
```

The description is the main thing that decides whether a skill triggers. See [Writing the description](#step-4-write-the-description) below, and read [references/description-optimization.md](references/description-optimization.md) when tuning or testing one.

### `license` (optional)

The license that applies to the skill. Keep it short: either a license name or a pointer to a bundled file.

```yaml
license: Apache-2.0
# or
license: Proprietary. LICENSE.txt has complete terms
```

### `compatibility` (optional)

1-500 characters. Use it **only** when the skill has specific environment requirements, such as a target product, system packages, or network access. Most skills do not need it.

```yaml
compatibility: Designed for Claude Code (or similar products)
compatibility: Requires git, docker, jq, and access to the internet
compatibility: Requires Python 3.14+ and uv
```

### `metadata` (optional)

A map of **string keys to string values** for properties the spec does not define. Agent clients may read it. Use reasonably unique key names to avoid collisions, and quote values that YAML would otherwise parse as numbers or booleans.

```yaml
metadata:
  author: example-org
  version: "1.0"   # quoted so YAML keeps it a string
```

### `allowed-tools` (optional, experimental)

A **space-separated string** of tools the skill may run without asking for permission. Agents support this differently, and some ignore it.

```yaml
allowed-tools: Bash(git:*) Bash(jq:*) Read
```

### Full example with every field

```yaml
---
name: release-notes
description: >
  Drafts release notes from merged pull requests and commit history. Use when
  the user asks for release notes, a changelog entry, or a summary of what
  shipped since the last tag, even if they don't say "release notes".
license: MIT
compatibility: Requires git and the GitHub CLI (gh) with network access
metadata:
  author: platform-team
  version: "2.1"
allowed-tools: Bash(git:*) Bash(gh:*) Read
---
```

## Body content

The Markdown after the frontmatter holds the instructions. **There are no format restrictions.** Useful sections include:

- Step-by-step instructions or a checklist.
- Examples of inputs and outputs.
- Common edge cases and **gotchas**.
- Output templates.
- A list of available scripts and when to run them.

The agent loads the **entire** body when it activates the skill. Keep `SKILL.md` **under 500 lines and about 5,000 tokens**, and move longer material into referenced files.

## Optional directories

### `scripts/`

Executable code (commonly Python, Bash, or JavaScript; what runs depends on the agent). Scripts should:

- Be self-contained or clearly document their dependencies (pin versions).
- **Never prompt interactively.** Take input through flags, environment variables, or stdin, because an interactive prompt will hang the agent.
- Provide `--help`, give helpful error messages, and handle edge cases.
- Write data to stdout and diagnostics to stderr. Prefer structured output (JSON, CSV).

Read [references/scripts.md](references/scripts.md) before writing or bundling a script.

### `references/`

Extra documentation the agent reads on demand, for example `REFERENCE.md`, `FORMS.md`, `finance.md`, or `legal.md`. Keep each file focused: smaller files use less context.

### `assets/`

Static resources: document or config templates, images and diagrams, and data files such as lookup tables or schemas.

## File references

- Use **relative paths from the skill root**: `[the reference guide](references/REFERENCE.md)`, `scripts/extract.py`.
- Keep references **one level deep** from `SKILL.md`, and avoid chains of files that point to further files.
- Say **when** to load each file. "Read `references/api-errors.md` if the API returns a non-200 status" works better than "see references/ for details".

## Validation

Validate with the reference tool [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref):

```bash
skills-ref validate ./my-skill
```

It checks that the frontmatter is valid and that the naming rules are followed.

## Where agents look for skills

Clients scan project-level and user-level folders for subdirectories that contain a file named exactly `SKILL.md`. Common locations:

- `<project>/.agents/skills/` and `~/.agents/skills/` (cross-client convention)
- `<project>/.<client>/skills/` and `~/.<client>/skills/` (client-specific, for example `.claude/skills/`)

If two skills share a name, the project-level skill usually overrides the user-level one. Many clients only load project-level skills from trusted repositories.

---

## How to create a new skill

Follow these steps in order. Copy this checklist and tick items off as you go:

```text
- [ ] 1. Gather real source material
- [ ] 2. Choose the scope and name
- [ ] 3. Create the folder and SKILL.md
- [ ] 4. Write the description
- [ ] 5. Write the instructions (body)
- [ ] 6. Add scripts, references, and assets (if needed)
- [ ] 7. Validate
- [ ] 8. Test with an agent and refine
- [ ] 9. Install, register, and commit
```

### Step 1: Gather real source material

Skills written from generic knowledge produce vague advice like "handle errors appropriately". Base the skill on real expertise instead:

- A task you just completed with an agent: the steps that worked, the corrections you had to make, and the input and output formats.
- Project artifacts: runbooks, API specs, schemas, code review comments, incident reports, and git history.

Write down the **gotchas**: the non-obvious, environment-specific facts the agent would get wrong without being told.

### Step 2: Choose the scope and name

- **Scope it as one coherent unit of work.** If it is too narrow, a single task needs several skills loaded at once. If it is too broad, the skill is hard to trigger accurately. For example, "query and format reports from our analytics DB" is one skill, while adding "database administration" makes it too broad.
- **Pick the name**: lowercase, hyphen-separated, 64 characters or fewer, no `--`, and no leading or trailing hyphen.
- Check that no existing skill already covers this. If one does, extend it instead.

### Step 3: Create the folder and SKILL.md

```bash
mkdir -p skills/my-skill
cp skills/agent-skills-specification/assets/SKILL.template.md skills/my-skill/SKILL.md
```

Open `SKILL.md` and set `name:` to **exactly** the folder name (`my-skill`). Create `scripts/`, `references/`, or `assets/` only when you have something to put in them.

### Step 4: Write the description

The description is the only part the agent sees before it decides to use the skill, so give it the most care.

1. **Use imperative phrasing**: "Use this skill when..." rather than "This skill does...".
2. **State what the skill does**, naming the concrete capabilities (for example "compute summary statistics, clean messy data, generate charts").
3. **State when to use it**, describing the user's intent and the words they would actually use, including file types, tools, and domain terms.
4. **Be a little pushy.** Include situations where the user does not name the domain ("...even if they don't mention 'CSV'").
5. **Say what it is not for** if a neighbouring skill could be confused with it.
6. **Keep it to a few sentences**, well under 1024 characters.
7. **Use a YAML block scalar (`>`)** if the text contains a colon, or quote it. An unquoted `key: value: more` is the most common cause of a YAML parse error.

```yaml
# Before
description: Process CSV files.

# After
description: >
  Analyze CSV and tabular data files: compute summary statistics, add derived
  columns, generate charts, and clean messy data. Use this skill when the user
  has a CSV, TSV, or Excel file and wants to explore, transform, or visualize
  the data, even if they don't explicitly mention "CSV" or "analysis".
```

### Step 5: Write the instructions (body)

Use this order as a starting point:

1. **Title and one-line purpose.**
2. **Quick start / default procedure**: numbered steps for the common case.
3. **Gotchas**: environment-specific traps. Keep them in `SKILL.md`, not in a reference file, because the agent may not realize it needs to open that file.
4. **Output format / templates**: inline if short, or in `assets/` if long.
5. **Validation**: how the agent checks its own work (run a validator, fix the problems, repeat).
6. **Pointers to resources**, each with a condition for when to load it.

Writing principles:

- **Add what the agent lacks and leave out what it already knows.** For each line, ask: "Would the agent get this wrong without it?" If not, cut it.
- **Give a default, not a menu.** Pick one approach and mention alternatives briefly.
- **Match specificity to fragility.** Give exact commands for fragile or destructive operations ("Run exactly this; do not add flags"). Leave room for judgment, and explain why, on flexible tasks.
- **Teach procedures, not single answers.** Explain how to approach a class of problems.
- **Use checklists** for multi-step workflows, and **plan-validate-execute** for batch or destructive changes.
- Stay under **500 lines**.

### Step 6: Add scripts, references, and assets (if needed)

- **`scripts/`**: add a script when testing shows the agent rewriting the same logic each time. Make it non-interactive, give it `--help`, and pin its dependencies. See [references/scripts.md](references/scripts.md).
- **`references/`**: move detailed or rarely needed material here. Link each file from `SKILL.md` with a "read this when..." condition.
- **`assets/`**: put templates, schemas, and example files here.
- Use relative paths and keep references one level deep.

### Step 7: Validate

```bash
skills-ref validate skills/my-skill
```

If `skills-ref` is not available, check by hand:

- [ ] The file is named exactly `SKILL.md` and starts with `---` on line 1.
- [ ] The frontmatter parses as YAML (colons are quoted, or a block scalar is used).
- [ ] `name` matches the folder name and the naming rules.
- [ ] `description` is non-empty and 1024 characters or fewer.
- [ ] `compatibility` (if present) is 500 characters or fewer, and all `metadata` values are strings.
- [ ] Every relative link points to a file that exists.
- [ ] The body is under 500 lines.

### Step 8: Test with an agent and refine

1. Install the skill where your agent looks for skills (see [Where agents look for skills](#where-agents-look-for-skills)).
2. **Triggering**: try prompts that should activate the skill, using varied wording that does not name the skill, and near-miss prompts that should not. Read [references/description-optimization.md](references/description-optimization.md) for a structured eval method.
3. **Execution**: run real tasks and read the agent's **full trace**, not just the final output. Look for steps that wasted effort, instructions that did not apply, and places where the agent had to choose between options without a default.
4. When you have to correct the agent, add the correction to **Gotchas**.
5. Repeat until the skill triggers reliably and completes tasks without corrections.

### Step 9: Install, register, and commit

- Add the skill to the repository README's skill table.
- Commit the skill folder. Keep test output out of the repository (for example in a git-ignored `debug/` folder).
- Copy or symlink it into `.agents/skills/` or your agent's skills folder to use it.

---

## Gotchas

- **`name` must match the folder name exactly.** Some clients load the skill anyway with a warning, and others reject it.
- **Unquoted colons in `description` break YAML.** Use `description: >` or wrap the value in quotes.
- **`metadata` values must be strings.** `version: 1.0` is a float, so write `version: "1.0"`.
- **`allowed-tools` is a single space-separated string**, not a YAML list, and it is experimental.
- **A missing or empty description usually means the skill is skipped entirely**, not loaded with a warning.
- **Simple requests may not trigger a skill** even when the description matches. Agents tend to use skills for tasks they cannot easily do on their own.
- **Product-specific frontmatter fields are not portable.** Put portable extras under `metadata`.
- **Interactive scripts hang the agent.** Never call `input()`, `read`, or similar prompts.

## Additional resources

- [references/description-optimization.md](references/description-optimization.md): read when writing, testing, or debugging a description that triggers too often or not often enough.
- [references/scripts.md](references/scripts.md): read before adding or reviewing anything in `scripts/`.
- [assets/SKILL.template.md](assets/SKILL.template.md): copy this template when starting a new skill.
- Upstream: [Specification](https://agentskills.io/specification) · [Best practices](https://agentskills.io/skill-creation/best-practices) · [Optimizing descriptions](https://agentskills.io/skill-creation/optimizing-descriptions) · [Using scripts](https://agentskills.io/skill-creation/using-scripts)
