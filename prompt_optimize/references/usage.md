# Prompt Optimizer Usage

This reference is for an AI agent orchestrating the prompt optimizer. See `SKILL.md` for the full step-by-step flow.

## Prerequisites

Before starting, the agent must have:

- `prompt`: prompt text or a prompt file path.
- `dataset`: JSONL, CSV, or Excel evaluation data with human gold answers.
- `business_goal`: natural-language optimization goal.
- `model`: target model ID (litellm format, e.g. `gpt-4o`, `claude-sonnet-4-20250514`, `deepseek/deepseek-v4-flash`).
- `api_key`: API key for the target model (literal or env var name).
- `out`: a run artifact directory.

**Model configuration is mandatory.** If the user hasn't provided model and API key, the agent must ask before proceeding. Do not fall back to agent self-inference.

## CLI subcommands

Call the wrapper by absolute path. Each prints JSON to stdout. Let `SKILL=<skill-dir>`.

### validate-inputs

Check dataset format + gold presence. Exit 2 if invalid.

```bash
python $SKILL/scripts/run_prompt_optimizer.py validate-inputs \
  --dataset <path> [--input-column C] [--gold-column C] [--sample-size N]
```

### load-dataset

Load + normalize all rows. With `--strip-gold`, output only `{id, input}` for feeding to run-inference.

```bash
python $SKILL/scripts/run_prompt_optimizer.py load-dataset \
  --dataset <path> [--input-column C] [--gold-column C] [--strip-gold]
```

### run-inference

Call the user's real model API to produce predictions. Input rows must NOT contain gold.

```bash
python $SKILL/scripts/run_prompt_optimizer.py run-inference \
  --prompt <path-or-inline> \
  --dataset <path>            # {id, input} rows only \
  --model <model-id> \
  --api-key <key-or-env-var> \
  [--base-url <url>] \
  [--prompt-mode system|template] \
  [--concurrency N] \
  [--timeout seconds] \
  [--output <path>]
```

- `--prompt-mode system` (default): prompt = system message, input = user message.
- `--prompt-mode template`: prompt contains `{input}` placeholder, rendered as user message.
- Stdout: JSONL `{"id": "1", "prediction": "..."}`.
- Stderr: progress JSON lines `{"completed": N, "total": N, "tokens_in": N, "tokens_out": N, "errors": N}`.
- Exit 2 if `--api-key` is missing or unresolvable.

### score-predictions

Judge `is_correct` for each row. When predictions lack gold, use `--dataset` to merge from original file.

```bash
python $SKILL/scripts/run_prompt_optimizer.py score-predictions \
  --task-type <T> --predictions <file|inline-json> [--dataset <path>]
```

### compute-metrics

Compute every metric in the task_type pool.

```bash
python $SKILL/scripts/run_prompt_optimizer.py compute-metrics \
  --task-type <T> --results <file|inline-json>
```

### check-guardrails

Decide candidate eligibility vs baseline.

```bash
python $SKILL/scripts/run_prompt_optimizer.py check-guardrails \
  --payload <file|inline-json>
```

## What the agent does itself (no Python)

- **Eval plan**: read `assets/prompts/eval_plan_generation.md`; decide task_type, primary/secondary metrics, guardrails. Write `eval_plan.json`.
- **Attribution + gradients**: read `assets/prompts/badcase_attribution.md`; analyze wrong cases.
- **Rewrite suggestions**: read `assets/prompts/rewrite_suggestion.md`; generate edit suggestions.
- **Strategy grouping**: read `assets/prompts/strategy_grouping.md`; group suggestions into strategies.
- **Candidate generation**: read `assets/prompts/candidate_generation.md`; produce candidate prompts.
- **Report**: read `assets/prompts/best_round_report.md`; write `best_round_report.md`.

The agent does NOT produce predictions itself. Inference is always via `run-inference`.

## check-guardrails payload shape

```json
{
  "candidate_metrics": {"micro_recall": 1.0, "micro_f1": 1.0},
  "baseline_metrics": {"micro_recall": 0.8, "micro_f1": 0.89},
  "baseline_correct_ids": ["1", "2", "3", "4"],
  "candidate_results": [{"id": "1", "is_correct": true}],
  "eval_plan": {
    "primary_metric": "micro_recall",
    "secondary_metrics": ["micro_f1", "exact_match", "label_regression_rate"],
    "guardrails": {"max_correct_to_wrong_rate": 0.05, "min_secondary_metric_ratio": 0.95, "no_empty_prediction": true}
  }
}
```

## Artifact reading order

1. `eval_plan.json` — task type, metrics, guardrails.
2. `baseline_results.jsonl` / `baseline_metrics.json` — baseline errors and scores.
3. `round_{N}/wrong_cases_attributed.jsonl`, `gradients.jsonl`, `rewrite_suggestions.jsonl`, `strategy_groups.json` — the optimization chain.
4. `round_{N}/candidate_scores.json` — candidate metrics, guardrail results, selection.
5. `round_history.jsonl` — round-by-round advancement.
6. `best_round_report.md` — human-readable rationale.
7. `final_prompt.md` — the prompt to hand off.

## Exit codes

- `0`: success.
- `2`: input/boundary error — explain and request corrected input.
- `1`: unexpected runtime failure — report command and stderr.
