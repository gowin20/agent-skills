# Scripts in skills

## When you don't need a script

For a one-off tool call, put the command directly in `SKILL.md` and pin the version:

```bash
uvx ruff@0.8.0 check .
npx eslint@9 --fix .
pipx run 'black==24.10.0' .
go run golang.org/x/tools/cmd/goimports@v0.28.0 .
```

State prerequisites in `SKILL.md`, or in the `compatibility` field for runtime requirements. Once a command gets complicated, move it into a tested script.

## Referencing scripts

Use paths relative to the skill root, because the agent resolves them from there. List the scripts in `SKILL.md`:

```markdown
## Available scripts

- `scripts/validate.sh`: validates the input file. Run: `bash scripts/validate.sh "$INPUT_FILE"`
- `scripts/process.py`: transforms results. Run: `python3 scripts/process.py --input results.json`
```

## Self-contained dependencies

Declare dependencies inside the script so it runs without a separate install step:

- **Python (PEP 723)**: run with `uv run scripts/extract.py`.

  ```python
  # /// script
  # requires-python = ">=3.11"
  # dependencies = ["pdfplumber==0.11.4"]
  # ///
  ```

  Use `uv lock --script` if you need a lockfile.
- **Deno**: import with `npm:` or `jsr:` specifiers, then `deno run scripts/extract.ts`.
- **Bun**: use versioned imports (`"cheerio@1.0.0"`), then `bun run scripts/extract.ts`. Auto-install is disabled if a `node_modules` folder exists further up the tree.
- **Ruby**: use `require "bundler/inline"` with a `gemfile do ... end` block, then `ruby scripts/extract.rb`.

## Designing scripts for agents

- **No interactive prompts.** This is a hard requirement. Take input through flags, environment variables, or stdin. A script that waits for input will hang.
- **Provide `--help`.** It is the main way an agent learns how to call a script, so keep it short and accurate.
- **Write errors that help.** Say what was wrong and what is allowed:

  ```text
  Error: --format must be one of: json, csv, table.
  Received: "xml"
  ```

- **Produce structured output.** Prefer JSON, CSV, or TSV. Send data to **stdout** and progress messages and warnings to **stderr**.
- **Make scripts idempotent** where possible, and reject ambiguous input instead of guessing.
- **Make destructive operations safe:** provide `--dry-run`, and require `--confirm` or `--force` before doing damage.
- **Use meaningful exit codes**, each documented in `--help`.
- **Keep output a predictable size.** Agent tools may cut output off at around 10-30K characters. Support `--offset` or `--limit`, or write large results to a file with `--output`.
