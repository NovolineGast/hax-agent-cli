# Batch Mode

> [!WARNING]
> **Parts of this document have drifted from the current codebase** (v1.6.0).
> - The standalone `src/batch.js` module no longer exists; batch mode is now
>   implemented directly in `src/cli.ts` (`runBatch`).
> - **Stdin piping is no longer supported.** `--batch` takes the prompt as a
>   CLI argument, or pairs with `--input <file>`. `echo "..." | hax-agent --batch`
>   will exit with an error asking for a prompt string or `--input`.
> - The multi-turn markers (`---multi---` / `@@@multi@@@`) described below are
>   **not handled** by the current implementation — the whole input is sent as a
>   single turn.
>
> Verified current usage:
>
> ```bash
> hax-agent --batch "explain this code"      # prompt as argument
> hax-agent --batch --input task.txt         # prompt from file
> ```

Batch mode lets you run HaxAgent non-interactively from scripts, CI pipelines, and automated workflows.

## Quick start

```bash
# Pass a single prompt as an argument (stdin piping is NOT supported)
hax-agent --batch "Explain the src/cli.ts file"

# Read the prompt from a file
hax-agent --batch --input example-prompts.txt --output result.md

# Choose a different model
hax-agent --batch --input tasks.txt --model claude-sonnet-4-20250514
```

## CLI flags

| Flag | Description |
|------|-------------|
| `--batch` | Enable batch (non-interactive) mode |
| `--model <name>` | Override the model (default: settings.model) |
| `--input <path>` | Read prompts from a file instead of stdin |
| `--output <path>` | Write the response to a file instead of stdout |
| `--raw` | Output raw text only (no token summary footer) |
| `--no-raw` | Include token/cost summary footer |

## Input formats

### Single turn (default)

When the input does **not** start with a multi-turn marker, the **entire content** is sent as a single prompt:

```
Write a Python script that monitors CPU usage and logs warnings to a file.
```

### Multi turn

When the first line is exactly `---multi---` or `@@@multi@@@`, each following non-empty line is sent as a **separate turn** — the model sees the conversation history from previous turns:

```
---multi---
Add unit tests for the parseBatchInput function.
Now update the README to document the multi-turn format.
Finally, create a CI workflow that runs the new tests.
```

Each turn receives the full conversation context from all prior turns.

### Choosing a marker

Both `---multi---` and `@@@multi@@@` behave identically:

- **`---multi---`** — conventional, human-readable choice. Preferred for hand-written prompt files.
- **`@@@multi@@@`** — less likely to collide with markdown frontmatter or YAML separators. Good for machine-generated input.

## Exit codes

| Code | Meaning |
|------|---------|
| `0` | All turns completed successfully |
| `1` | Error: missing input, file read/write failure, or a turn failed |

## Examples in this directory

- **`single-turn.txt`** — A single code-review prompt, no marker.
- **`example-prompts.txt`** — Four programming tasks using `---multi---`, each building on the previous turn.
