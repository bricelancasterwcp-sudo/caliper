# caliper — design brief

**Status:** brief only, awaiting owner approval. No code has been
extracted. Working name is a placeholder (see open rulings).
**Date:** 2026-09-02. **Source of every number:** `docs/INVENTORY.md`
and the black-oxide files it cites, at `~/workspace/oxide` @ `521f602f`.

## 1. What the bench is

Black Oxide spent four weeks and nine waves building an apparatus to
answer one shape of question: *given two notations for the same tasks,
how do models use each?* The language turned out to converge on the
one models already knew; the apparatus did not, and it is the more
reusable of the two. caliper is that apparatus with the language
removed.

The parts, each with the oxide file that holds it today:

- **Paired arms on identical tasks.** Every task has one prompt and
  one committed `expected_stdout`; every arm has an oracle-verified
  reference solution (`eval/tasks.jsonl`, `eval/tasks-large.jsonl`,
  `eval/references-v04/`, `eval/references-large/`). A pair that does
  not reproduce its stdout byte-exactly is not a reference
  (`eval/train_corpus.py:157 validate_pair`).
- **A symmetric control arm.** Whatever is done to arm A is done to
  arm B — same corpus construction, same fine-tune recipe, same seeds
  (`scripts/runpod/train_lora.py`: "one recipe for all six runs"). The
  control is what let wave 8 call a 15× compile-rate gap a language
  property rather than a distribution-shift artifact
  (`eval/results/v04-wave8-phaseb/REPORT.md`, "Distribution shift").
- **Composition-controlled efficiency.** Tokens-to-green paired on
  `(seed, task)` and restricted to cells green in *both* arms, because
  each arm's own green set changes size and difficulty whenever pass@1
  moves (oxide `SPEC.md §59.7`). The uncontrolled ratio published in
  wave 1 was withdrawn when the controlled one read the opposite
  direction; the controlled construction is pinned by acceptance test
  against committed cells: 1.217 / 1.293 / 1.067
  (`tests/test_experiment_report.py:461-477`).
- **Learnability as uptake ÷ exposure** (oxide `SPEC.md §62.1`,
  `eval/learnability.py`, 79 lines, zero Oxide references). Raw uptake
  ranks `+=` first; the ratio shows it needed 13× the exposure of
  `reverse`.
- **Two censuses.** Demand: what the model reaches for and gets
  rejected on, counted over its own failing replies
  (`eval/demand_census.py`). Cost: where the reference pairs' token
  surplus sits, per construct (`eval/cost_census.py`). Neither eye
  suffices alone — wave 3 shipped constructs on cost evidence that
  drew zero uptake, and wave 2 deferred index-assign on demand
  evidence while it was the corpus's largest cost gap.
- **Drift guards that reproduce to the digit.** The untuned Rust
  control's pass@1 of 0.565 has reproduced in eight environments
  (`eval/results/v04-wave8-phaseb/REPORT.md:120`); pass@1 is invariant
  across GPU architectures, secondary metrics within one. A run whose
  guard misses its anchor publishes no ratio.
- **Seed-matched anchors.** A three-seed screen is compared to the
  committed cells restricted to those seeds, never to the ten-seed
  published figure (`eval/wave8_screen.py:21 GUARD_ANCHORS`; the
  published 0.745 reads 0.800 on seeds 1–3).
- **Contamination in both directions** — held-out tasks against
  training programs and against committed eval solutions
  (`eval/train_corpus.py:73 contamination_report`; the second
  direction was run by hand in wave 8 and is item 6 of that report's
  feed-forward, `eval/results/v04-wave8-large/REPORT.md:173`).
- **Ops at under $1.50 a wave.** Community RTX 3090 at $0.22/h; wave 5
  $0.36, wave 7 $0, wave 8 Phase B $0.81, the 14B screen $0.28
  (each wave's `REPORT.md`, first lines). Transfers verified by
  content hash against a `SHAS.txt`, because a truncated tar passed a
  file-count check (`eval/results/v04-wave8-14b-screen/REPORT.md`,
  provenance). Liveness and capacity checked on the machine, never
  from the API's fields.

## 2. What it is for

Any question of the form *how do models use notation X against
notation Y* where both can be oracle-checked:

1. **flux** (`~/workspace/flux`, scaffolded in parallel) — consumer
   #1. Rust with rustc's machine-applicable suggestions auto-applied
   before the model sees the residual, against plain Rust. Same oracle
   (rustc + expected_stdout), same paired estimands, same guards.
2. **sensorium's LLM-native query CLI**, or any tool-call / extraction
   schema — consumer #2, later. The oracle becomes a schema validator
   or a replayed command; the arms are two spellings of the surface.
3. **black-oxide itself**, once it consumes caliper instead of
   carrying the bench.

The measurement discipline (pre-registration, instrument honesty,
mutation-tested tests, long-run ops) is already a skill
(`~/.claude/skills/rigorous-experiments`). caliper is the *code* that
discipline runs on.

## 3. Extraction principle

Two interfaces, both already present in shape at `eval/harness.py`:

- **Oracle** — `check(source) -> Verdict{compiled, stdout, diagnostics}`.
  rustc today (`eval/rustc_adapter.py`, 180 lines, generic). A
  JSON-schema validator, a test runner, or a replayed CLI are other
  implementations. The bench never knows which.
- **Arm** — a triple: `render_prompt(task, shots, card) -> str`,
  `parse_reply(text) -> source` (today `eval/extract.py`),
  `to_oracle_input(source) -> (oracle_source, diagnostics)`. For the
  oxide arm the third step is `src.codegen.rust.transpile`
  (`eval/harness.py:141`); for rust it is identity. That third step is
  the *only* place the Oxide compiler is imported by anything the
  inventory calls generic or mixed.

Rule: everything `docs/INVENTORY.md` marks **generic** moves as-is;
everything marked **mixed** moves after its Oxide constants are lifted
into configuration (arm names, model ladder, guard anchors, construct
families, pair sources); everything marked **oxide-bound** stays in
black-oxide, which then pins caliper the way bloomery pins assay
(`PYTHONPATH` to a sibling checkout first, a package later).

Measured shape of the job: 1,073 generic lines, 6,047 mixed, 1,044
bound, over 26 modules; plus 89 generic / 192 mixed script lines
(`docs/INVENTORY.md`, totals). The mixed share is 74% — this is a
constant-lifting job, not a rewrite.

## 4. Invariants that must survive

These are the acceptance tests; the extraction is not done until each
passes through caliper's code against black-oxide's committed data.

1. **Every published ratio reproduces byte-exactly.** The six pinned
   today in `tests/test_experiment_report.py` (1.217 / 1.293 / 1.067,
   the 0.565 control, the v03 closing baseline, the census pairs) plus
   the wave-8 screen's 0.0652 and the two seed-matched anchors
   (`eval/results/v04-wave8-14b-screen/screen.json`).
2. **The tokenizer pin.** One tokenizer, content-hashed, attested from
   three checkpoints (`eval/tokenizer_pin.py`). Token counts are
   measurements only under it.
3. **None-vs-zero.** Unmeasured is `None` and named in a `dropped`
   list; measured-and-zero is `0.0` (`eval/wave8_screen.py:45-50`
   returns `rate: None` on an empty arm and the verdict reads
   `unmeasured`).
4. **Provenance completeness.** Every arm run writes `provenance.json`
   with gguf sha, llama.cpp commit, sampler, `num_ctx`, `num_predict`,
   seeds and `seeds_subset` (`eval/results/v04-wave8-14b-screen/
   results-large/tune-ox-14/provenance.json`). A serialization
   completeness test pins every field.
5. **Infrastructure ≠ subject.** Protocol-breaking replies raise
   (`ServerContextOverflowError`, `ModelError` in `eval/models.py`);
   they never score as data.

## 5. Python

Black Oxide requires 3.14 (`README.md:356`) and is stdlib-only. The
PEP 695 `type` statements are in `src/parser/ast.py` and
`src/sema/cfg.py` — **none in `eval/`**; the eval path needs ≥3.12 on
the pod only because `harness` imports `src`. An extracted caliper
therefore has no measured 3.14 dependency. Recommendation: **stdlib
only, floor 3.12** (what the RunPod path already runs the harness
under), verified by CI on 3.12 and 3.14. Flagged as a ruling below.

## 6. Extraction order

Each phase ends with a falsification test that must fail when the
phase's load-bearing line is broken, then pass.

| phase | moves | falsification |
|---|---|---|
| 0 pin | nothing; caliper's test suite gains fixtures that are black-oxide's committed cells for every ratio in §4.1 | delete the `(seed, task)` pairing key → the 1.217 test fails |
| 1 interfaces | `Oracle` and `Arm` protocols; `rustc_adapter` as oracle #1; a trivial `python -c` + `expected_stdout` oracle #2 | the paired estimand runs a toy python-vs-python pair end to end; swapping the oracle changes nothing in the estimand code |
| 2 clients | `models`, `llamacpp` with `GRAMMAR_FILES` and the card-drop guard injected per arm | the card-front-truncation guard fires for a configured arm and not for an unconfigured one |
| 3 estimands | `experiment_report` paired functions, `learnability`, `difficulty_band`, `shape_gate`, `token_match`, `tokenizer_pin` | phase-0 fixtures reproduce through the moved code; mutate a band width → its test fails |
| 4 drivers | `driver`, `exp_campaign`, `rollup`, `g0_report` with `MODELS` / `ARM_SPECS` / `ARMS` as config | black-oxide's 12-arm spec loads from config and names the same 12 arms in the same order |
| 5 censuses | `cost_census`, `demand_census` over a pluggable construct-family table | wave-8 census pairs (g01 272/265, g02 284/202) reproduce |
| 6 ops | `runpod_api.sh`, `serve_arm.sh`, `merge_lora.py`, `convert_quant.sh`, `pod_setup.sh` with the repo URL and oracle install as parameters | a dry pod bring-up for a repo that is not black-oxide |
| 7 consume | black-oxide pins caliper; its `eval/` shrinks to the bound modules + arm assets | black-oxide's suite (1738 collected today) green; §4.1 byte-identical |
| 8 probe | *optional*: the one-defect + diagnostic repair-scoring machinery from `eval/probe.py`, with the Oxide defect corpus staying | the +59.0pp qwen paired delta reproduces from committed probe results |

Phases 0–3 are worth doing even if nothing else is: they make the
estimands importable by flux without dragging the Oxide compiler.

## 7. Not in scope

- The Oxide language (`src/`), its cards, grammars, shots, and
  probes — arm assets, they stay.
- The ownership probe's defect corpus and taxonomy (`eval/probe.py`,
  `eval/degenerate.py`, `eval/deformation.py`, `eval/demand.py`).
- Wave runbooks (`wave8_phaseb.sh`, `wave8_14b_screen.sh`,
  `wave9_rescreen.sh`) — they belong to their plans.
- `train_lora.py` — generic in recipe but reads oxide's train tasks
  and renders through `harness.build_prompt`; revisit in phase 6 if a
  second consumer needs fine-tuned arms.
- Any new estimand. caliper ships what black-oxide measured with, not
  what it wished it had.

## 8. Open rulings for Brice

1. **Name.** "caliper" is a placeholder in the forge/instrument
   lineage (robigo, assay, bloomery, crucible, smithy).
2. **Python floor:** 3.12 (recommended, matches the pod path, no
   measured 3.14 need in `eval/`) or 3.14 (matches black-oxide).
3. **Does flux wait for caliper?** Recommended **no**: flux pins
   black-oxide's `eval/` directly by `PYTHONPATH` today, exactly as
   bloomery pinned assay; caliper extracts behind it and flux re-pins
   when phases 0–3 land. Nothing blocks on this brief.
4. **Library or template?** Recommended library (importable package,
   consumers pin a commit), because the invariants in §4 are about
   *shared code* reproducing shared numbers, which a copied template
   cannot promise.
5. **Phase 8** (probe machinery): in or out.
