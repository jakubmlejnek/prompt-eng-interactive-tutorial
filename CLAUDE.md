# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Anthropic's **Prompt Engineering Interactive Tutorial** — an educational course
delivered entirely as Jupyter notebooks. There is no application to build, no
test suite, and no package to ship. "Running" the project means executing
notebook cells in order; "correctness" is judged by per-exercise grading
functions embedded in the notebooks, not by CI.

The course is 9 chapters plus an appendix, intended to be worked through in
numbered order. It uses Claude 3 Haiku at `temperature=0.0` for deterministic
results.

## Three parallel editions (keep them in sync)

The exact same curriculum exists in three directories that differ **only** in
how they call Claude. When editing lesson content, exercises, or hints, apply
the equivalent change to all three editions unless the change is genuinely
specific to one API surface:

| Directory | Calls Claude via | Client setup |
|---|---|---|
| `Anthropic 1P/` | Anthropic 1P API | `anthropic.Anthropic(api_key=API_KEY)` |
| `AmazonBedrock/anthropic/` | Bedrock via Anthropic SDK | `AnthropicBedrock(aws_region=AWS_REGION)` |
| `AmazonBedrock/boto3/` | Bedrock via raw boto3 | `boto3.client('bedrock-runtime', ...)` + manual JSON body |

Notebooks are numbered identically across editions (`01_` … `10_`), so the
corresponding file in each directory is the canonical place to mirror a change.
Note minor filename inconsistencies already exist (e.g. spaces vs underscores,
`Empirical` misspellings) — match the existing name in each directory rather
than renaming.

## How a tutorial session runs

1. **The `00_Tutorial_How-To` notebook must be run first.** It sets and
   `%store`s the shared variables that every other notebook reads back with
   `%store -r`:
   - `Anthropic 1P`: stores `API_KEY` and `MODEL_NAME` (`claude-3-haiku-20240307`).
   - `AmazonBedrock`: stores `MODEL_NAME` (`anthropic.claude-3-haiku-20240307-v1:0`)
     and `AWS_REGION` (auto-detected from the boto3 session).
2. Every chapter notebook opens with a **Setup cell** that re-imports the stored
   variables, `pip install`s dependencies, and defines a `get_completion()`
   helper. This is why notebooks must be run top-to-bottom — later cells depend
   on the client and helper defined in Setup.
3. Each chapter follows the structure: **Lesson → Exercises → Example
   Playground**.

## Key conventions

- **`get_completion()`** is the single entry point for calling Claude in every
  notebook. Its signature varies slightly by edition (e.g. `system_prompt` vs
  `system` parameter, JSON body assembly for boto3), but its role is identical.
  Reuse it rather than calling the client directly inside lesson/exercise cells.
- **Exercise cells** isolate the editable surface to a single `PROMPT` variable
  (commented `# this is the only field you should change`) followed by a
  `grade_exercise(text)` function that returns a bool — usually a regex match
  against required substrings. Preserve this pattern when adding exercises:
  learners edit only the prompt, and grading must be deterministic.
- **Hints** live in `Anthropic 1P/hints.py` and `AmazonBedrock/utils/hints.py`
  as module-level strings named `exercise_<chapter>_<n>_hint`. Bedrock notebooks
  import them via `from utils import hints` (after appending `..` to
  `sys.path`); the 1P edition imports `hints` from the same directory. A hint's
  text states exactly what the grading function checks for — keep hint text and
  the corresponding `grade_exercise` regex consistent.
- Prompt-template variables use **single-mustache** placeholders like `{TOPIC}`,
  `{QUESTION}` substituted via Python f-strings/`.format()`.

## Dependencies

- `Anthropic 1P`: only `anthropic` (installed inline via `!pip install anthropic`).
- `AmazonBedrock`: pinned in `AmazonBedrock/requirements.txt`
  (`awscli`, `boto3`, `botocore`, `anthropic`, `pickleshare`); the Bedrock
  How-To installs them with `%pip install -qr ../requirements.txt` and then
  restarts the kernel. Bedrock also requires valid AWS credentials and model
  access in the configured region.

## Editing notebooks

- These are real `.ipynb` JSON files. Edit the actual cell sources; do not
  hand-write malformed JSON. Prefer Jupyter or a notebook-aware tool over raw
  text edits to avoid corrupting cell metadata.
- Don't commit executed output or API keys. `API_KEY` is set inline by the
  learner in the How-To notebook (placeholder `"your_api_key_here"`) — leave the
  placeholder in place.

## Contributing (per AmazonBedrock/CONTRIBUTING.md)

Work against the latest `main`; keep PRs focused on a single change without
incidental reformatting; ensure notebooks run cleanly top-to-bottom before
opening a PR. Report security issues through AWS's vulnerability reporting page
rather than public GitHub issues.
