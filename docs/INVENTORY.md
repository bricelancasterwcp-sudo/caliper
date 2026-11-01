# Inventory of black-oxide's measurement harness

Measured 2026-09-02 against `~/workspace/oxide` (<https://github.com/bricelancasterwcp-sudo/black-oxide>) at `521f602f` (branch `v04-wave9-index-syntax`). Every number below was produced by the commands at the end of this file; re-run them rather than trusting the table.

## eval/*.py

| module | LOC | purpose | imports from eval/ | imports from src/ | test files | class | Oxide coupling |
|---|---:|---|---|---|---:|---|---|
| `__init__` | 18 | Evaluation harness package (SPEC.md Part IX, sections 44-46). | — | — | 0 | **n/a** | package docstring |
| `cost_census` | 297 | Cost census: where the oxide/rust token surplus actually sits. | eval, token_match, tokenizer_pin, train_corpus | — | 3 | **mixed** | `EVAL_SOURCE`/`LARGE_SOURCE` PairSource constants; census math generic |
| `deformation` | 208 | The §56 field-assignment deformation signature, with a pinned definition. | — | explicit.parser, parser, parser.parser | 2 | **oxide-bound** | imports src.parser (Oxide AST) |
| `degenerate` | 122 | Taxonomy of degenerate repairs (ownership probe follow-up). | — | — | 1 | **oxide-bound** | ownership-probe repair taxonomy (OX codes) |
| `demand` | 167 | Demand counters for the generation-friction taxonomy. | — | sema.types | 1 | **oxide-bound** | imports src.sema.types.BUILTINS |
| `demand_census` | 954 | Demand census over committed campaign replies and the training corpus | token_match, train_corpus | — | 1 | **mixed** | construct families are Oxide vocabulary; counting/gating generic |
| `difficulty_band` | 199 | The two-arm, per-class difficulty band. | shape_gate | — | 1 | **generic** | two-arm per-class band; arm names are labels only |
| `driver` | 778 | Phase 6a run driver (SPEC Part X, sections 5 and 6.4). | eval, extract, llamacpp, models, repair | — | 3 | **mixed** | `MODELS` ladder (ollama tags) + arm dispatch; run skeleton generic |
| `exp_campaign` | 202 | Drive one experiment arm against a llama-server: generation + probes. | eval, driver, llamacpp, models | — | 1 | **mixed** | `ARM_SPECS`: 12 baked arm names + gguf naming (ox/rs × 1.5/7/14) |
| `experiment_report` | 473 | Pre-registered endpoints for the RunPod fine-tune experiment. | cost_census, probe | — | 4 | **mixed** | `ARM_NAMES` (12) + `SIZES`; paired estimand math (§59.7) is generic |
| `extract` | 50 | Model output -> candidate source (SPEC Part X, section 6.2). | — | — | 1 | **generic** | model reply -> candidate source; no arm knowledge |
| `g0_report` | 462 | G0 diagnostic profiler (SPEC Part X, section 50.5's gate deliverable). | eval, driver | — | 2 | **mixed** | `ARMS=(oxide,explicit,rust)`; profiler generic |
| `harness` | 659 | Evaluation harness (SPEC.md section 45): importable module + CLI. | rustc_adapter | cli, codegen.rust, explicit.pipeline, source | 5 | **mixed** | THE ARM BOUNDARY: imports src (transpile/explicit_run/diagnostics); `ARMS`, `TASKS_PATH` |
| `learnability` | 79 | Learnability: uptake per unit corpus exposure (SPEC §62.1). | — | — | 1 | **generic** | uptake ÷ exposure (SPEC §62.1); pure arithmetic |
| `llamacpp` | 267 | A ``ModelClient`` backed by llama.cpp's ``llama-server``. | models | — | 1 | **mixed** | `GRAMMAR_FILES={oxide,explicit}` + card-drop guard naming oxide arms |
| `models` | 434 | Model clients for the Phase 6a ladder (SPEC Part X, section 6.1). | — | — | 3 | **mixed** | card-front-truncation guard names the oxide/explicit arms |
| `probe` | 547 | Ownership probe: prompt construction, scoring, and runner. | eval, extract, llamacpp, models | — | 5 | **oxide-bound** | ownership probe: Oxide defect classes + scoring |
| `probe_campaign` | 147 | Drive the ownership probe across arms and seeds, resumably. | probe | — | 2 | **generic** | resumable campaign driver; logic generic, consumes `probe` |
| `repair` | 126 | Repair-prompt construction (SPEC Part X, section 6.3). | eval | — | 3 | **generic** | repair-prompt builder; arm-agnostic |
| `rollup` | 581 | Phase 6a grid aggregation (SPEC Part X, sections 3 and 6.5). | eval, driver | — | 2 | **mixed** | grid aggregation; arm triple baked into row/column logic |
| `rustc_adapter` | 180 | rustc ``--error-format=json`` adapter (SPEC.md section 45). | — | — | 13 | **generic** | ORACLE plugin #1: rustc --error-format=json |
| `shape_gate` | 177 | The pre-flight corpus-shape gate. | — | — | 1 | **generic** | corpus-shape gate, closed bands |
| `token_match` | 355 | Matched supervised-token training corpora from the paired factory output. | tokenizer_pin, train_corpus | — | 5 | **mixed** | `ARMS=(oxide,rust)`; pair file names `oxide.ox`/`rust.rs` |
| `tokenizer_pin` | 115 | One tokenizer, pinned by content hash, attested from three checkpoints. | — | — | 2 | **generic** | one tokenizer pinned by content hash |
| `train_corpus` | 225 | Training-corpus loading and the contamination guard. | eval | — | 2 | **mixed** | contamination guard generic; pair validation calls harness per arm |
| `wave8_report` | 158 | Wave 8 Phase B endpoints, computed by command rather than by hand. | cost_census, experiment_report, token_match | — | 1 | **mixed** | wave-8 endpoints; surplus arithmetic generic |
| `wave8_screen` | 202 | Wave 8's 14B screen: is the cliff a 7B property or a language property? | experiment_report | — | 1 | **mixed** | `GUARD_ANCHORS`, `BAND_*`, `SEVEN_B_RATIO`, `LEXER_SHARE_BASELINE` baked |

### Totals by class (LOC)

| class | LOC | share |
|---|---:|---:|
| generic | 1073 | 13.1% |
| mixed | 6047 | 73.9% |
| oxide-bound | 1044 | 12.8% |
| n/a | 18 | 0.2% |
| **total** | **8182** | |

Reading: **generic + mixed = the bench.** Mixed modules are generic
logic with Oxide *constants* baked in (arm names, model ladder, guard
anchors, construct families); extraction means lifting those constants
into configuration or into the consumer. Only four modules import the
Oxide compiler or its defect taxonomy, and they stay.

`harness.py` is the single seam that matters: it is where an arm turns
a candidate source into an oracle verdict (`src.codegen.rust.transpile`
for oxide, `src.explicit.pipeline.run` for explicit, `rustc` directly
for rust; `eval/harness.py:141-144`). Everything above that line is
prompt rendering and everything below is the rustc oracle, and both are
already arm-agnostic in shape.

## scripts/runpod/*

| script | LOC | class | note |
|---|---:|---|---|
| `convert_quant.sh` | 12 | **generic** | HF dir -> bf16 -> q8_0 gguf, sha recorded |
| `merge_lora.py` | 28 | **generic** | LoRA merge to bf16 for GGUF conversion |
| `pod_setup.sh` | 50 | **mixed** | idempotent pod bring-up; clones black-oxide, builds llama.cpp CUDA, installs rustc (the oracle) |
| `runpod_api.sh` | 14 | **generic** | REST wrapper: pods/create/terminate |
| `serve_arm.sh` | 35 | **generic** | serve one gguf, run one arm, SIGKILL + port-down teardown; only `cd /workspace/oxide` binds it |
| `train_lora.py` | 142 | **mixed** | QLoRA recipe; renders prompts via `harness.build_prompt` and reads oxide's train tasks |
| `wave8_14b_screen.sh` | 77 | **runbook** | wave-specific; stays with its plan |
| `wave8_phaseb.sh` | 72 | **runbook** | wave-specific; stays with its plan |
| `wave9_rescreen.sh` | 51 | **runbook** | wave-specific; stays with its plan |

| class | LOC |
|---|---:|
| generic | 89 |
| mixed | 192 |
| runbook | 200 |
| **total** | **481** |

## Non-Python assets under eval/ (arm assets; move WITH the arm, i.e. stay)

`grammar/oxide.gbnf`, `grammar/explicit.gbnf`, `grammar/build.py`, `shots/{oxide,explicit}/`, `probes.jsonl`, `tasks.jsonl`, `tasks-large.jsonl`, `solutions/` (contamination reference, frozen), `references-v04/`, `references-large/`, `train/`, `results/` (the experimental record, tracked).

## Test suite

`1738/1741 tests collected (3 deselected) in 0.15s`: 47 test files, 17625 lines.

## Commands

```bash
cd ~/workspace/oxide
wc -l eval/*.py | sort -n                      # LOC
wc -l scripts/runpod/*
grep -nE '^\s*(from|import)\s+(eval|src)\b' eval/*.py   # coupling
for m in $(ls eval/*.py | xargs -n1 basename | sed 's/.py//'); do
  printf '%-20s %s\n' "$m" "$(grep -lE "eval\.$m\b|from eval import .*\b$m\b" tests/*.py | wc -l)"
done                                           # test files per module
grep -nE '^(ARMS|MODELS|ARM_|GUARD_ANCHORS|BAND_|SEVEN_B|LEXER_SHARE|EVAL_SOURCE|LARGE_SOURCE|GRAMMAR_FILES)' eval/*.py
PYTHONDONTWRITEBYTECODE=1 .venv/bin/pytest --collect-only -q | tail -1
grep -lE '^\s*type [A-Z]\w* *=' eval/*.py src/**/*.py     # PEP 695 users: src/parser/ast.py, src/sema/cfg.py; none in eval/
```

The inventory itself was generated by a script that runs these same
commands; the table is not hand-typed.
