# caliper

**A paired-arm measurement bench for notation design — how do models
use notation X against notation Y, on identical tasks, under one
oracle.**

Black Oxide spent nine measured waves designing a language for LLMs and
found the language converging on the one models already knew. The
apparatus it built to find that out did not converge on anything: paired
arms with oracle-verified references, a symmetric control arm,
composition-controlled efficiency ratios, learnability as uptake per
exposure, demand and cost censuses, drift guards that reproduce to the
digit across eight environments, seed-matched anchors, two-way
contamination checks, and RunPod operations at under $1.50 a wave.

caliper is that apparatus with the language removed. The oracle is a
plug (rustc today; a schema validator or a replayed CLI tomorrow) and an
arm is a triple: render the prompt, parse the reply, hand the oracle
something it can check.

## Status

**Inventory and design brief only. Awaiting owner approval. No code has
been extracted.**

- [`docs/INVENTORY.md`](docs/INVENTORY.md) — every module of
  black-oxide's `eval/` and `scripts/runpod/`, measured by command:
  lines, imports, test coverage, and whether it is generic, generic
  with Oxide constants baked in, or bound to the Oxide compiler.
  Result: 1,073 generic lines, 6,047 mixed, 1,044 bound. A
  constant-lifting job, not a rewrite.
- [`docs/superpowers/specs/2026-09-02-caliper-design.md`](docs/superpowers/specs/2026-09-02-caliper-design.md)
  — what the bench is, what it is for, the two interfaces, the
  invariants that must survive extraction (every published black-oxide
  ratio reproducing byte-exactly through caliper's code is the
  acceptance test), the phase order with a falsification test per
  phase, and the rulings the owner has to make.

First consumers, in order: [flux](../flux) (deterministic pre-repair
for Rust), then a tool-schema or CLI-surface A/B, then black-oxide
itself.

Sibling discipline: the measurement method is a skill
(`rigorous-experiments`); the notation-design lessons are another
(`designing-notation-for-llms`). caliper is the code those run on.

## License

MIT — see [LICENSE](LICENSE).
