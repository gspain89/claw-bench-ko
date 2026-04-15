# ClawBench-KO

OpenClaw Korean Agent Benchmark -- LLM agent's ability to process Korean data, generate Korean documents, and understand Korean systems.

10 tasks across 3 categories, graded by automated checks and LLM judge.

Inspired by [PinchBench](https://github.com/pinchbench/skill).

## Prerequisites

- [OpenClaw](https://github.com/openclaw) installed and configured
- At least one model registered in OpenClaw (`openclaw config`)
- A judge model registered (default: `anthropic/claude-opus-4-6`) — required for `llm_judge` and `hybrid` tasks (6/10 tasks)
- Python 3.10+

## Quick Start

```bash
git clone https://github.com/gspain89/claw-bench-ko.git
cd claw-bench-ko

# Run all tasks (e.g. qwen/qwen3.5-27b)
python3 runner.py --model <openclaw-model-id>

# Run specific tasks
python3 runner.py --model <openclaw-model-id> --task addr_parse,num_convert

# Use a different judge model
python3 runner.py --model <openclaw-model-id> --judge <judge-model-id>

# Run automated-only tasks (no judge model needed)
python3 runner.py --model <openclaw-model-id> --task addr_parse,num_convert,phone_normalize

# Dry run (list tasks without executing)
python3 runner.py --model <openclaw-model-id> --dry-run
```

## Tasks

| ID | Task | Category | Grading | Difficulty |
|----|------|----------|---------|------------|
| addr_parse | Korean address parsing | Data Processing | automated | medium |
| num_convert | Korean number conversion | Data Processing | automated | medium |
| phone_normalize | Phone number normalization | Data Processing | automated | easy |
| csv_transform | Bank transaction CSV (EUC-KR) | Data Processing | hybrid | hard |
| meeting_minutes | Meeting minutes generation | Document Generation | llm_judge | medium |
| biz_email | Business email writing | Document Generation | llm_judge | medium |
| news_summary | News briefing summary | Document Generation | llm_judge | medium |
| invoice_gen | Tax invoice generation | Korean System | hybrid | hard |
| resume_parse | Resume parsing | Korean System | hybrid | medium |
| regulation_extract | Legal requirement extraction | Korean System | hybrid | hard |

## Options

```
--model           OpenClaw model ID (required)
--judge           Judge model ID (default: anthropic/claude-opus-4-6)
--task            Run specific tasks (comma-separated)
--runs N          Repeat each task N times (default: 1)
--output-dir      Custom output directory
--dry-run         List tasks without running
--verbose         Show detailed logs
--no-fail-fast    Continue even if first task scores 0
--no-judge        Skip judge calls — see "Deferred Judging" below
--save-artifacts  Persist per-task workspace/session/stdout/prompt for analysis
--skip-preflight  Skip the OpenClaw model-registration check
```

## Grading Types

- **automated** (4 tasks): File existence, JSON structure, field value matching. Deterministic.
- **llm_judge** (3 tasks): LLM judge scores 0-100 based on rubric. Evaluates Korean language quality, content accuracy, formatting.
- **hybrid** (3 tasks): Weighted combination of automated checks + LLM judge.

## Deferred Judging (`--no-judge`)

Use `--no-judge` when you want to run the agent now and grade later (e.g. parallel batch runs, manual review, third-party judges, or to avoid judge cost during smoke tests).

- `llm_judge` tasks: marked `pending_manual_review` with `score=0.0`. Grade them later from the saved artifacts.
- `hybrid` tasks: only the `automated` portion is graded; the judge weight stays as `pending_manual_review` (status `partial_automated_only`).
- `automated` tasks: graded normally.
- `--no-fail-fast` is implied for tasks left in `pending_manual_review` / `partial_automated_only` — the runner will not abort on a 0 score for those.
- Combine with `--save-artifacts` so you have the workspace + transcript needed for offline judging.

## Artifact Snapshots (`--save-artifacts`)

Stores per-task evidence under `results/<model>_<timestamp>/artifacts/<task_id>/`:

```
artifacts/<task_id>/
├── workspace/        # Final workspace state (excluding .git, .openclaw)
├── session/          # OpenClaw session JSONL (turns, tool calls, messages)
├── prompt.txt        # The exact prompt sent to the agent
├── agent_stdout.txt  # Agent CLI stdout
├── agent_stderr.txt  # Agent CLI stderr
└── meta.json         # task_id, grading_type, agent_returncode, duration, etc.
```

Snapshots are written before the next task wipes the agent's session, so all evidence survives the run.

## Output

Results are saved to `results/<model>_<timestamp>/`:

- `results.json` — scores, grading breakdown, run metadata
- `artifacts/` — per-task snapshots (only when `--save-artifacts` is passed)

## Dependencies

No pip packages required. Uses Python 3.10+ standard library only. Requires OpenClaw CLI on PATH.
