# Pinning candidate note — solver `codematrix`

STATUS: PREPARED, NOT QUALIFIED, NOT SUBMITTED. No Yukon validation was fired.
This is a deliberate, disclosed stop: see "Checks actually performed" and "Why no fire".

## Base SHA and frontier score

- Promoted base (editable tree): `Layr-Labs/quantum-safe-bitcoin-challenge` `main` =
  `46b24ebaa033fb69c7335794b54fd6a156359ec8` (committed 2026-09-27 09:28:16 +0000).
  Verified live this run with `git ls-remote origin refs/heads/main`.
- Benchmark id (pinning): `eafd2f3d-e64f-49c1-b98a-6b825b0cdc82`.
- Live promoted frontier (yukon.org/qsb dashboard payload, 2026-09-28): **995,329,477**, owner `cefika`.
- Baseline: 146,089,436. Promotion gate `minScoreImprovementBips=100` (1%).
  Floor to promote: **1,005,282,772** (needs +9,953,295 over the record).

## Bottleneck (stated by the promoted source itself, not invented here)

The promoted `pinning.cu` already knows part of the GLV table is L2-resident and says so
at line 243: caching the hot part "only evicts the 64 MiB table". Its own kill switch
`QSB_L2_SKIP` (line 292, default 1) "start[s] the persisting-L2 window after chunk 0
(half the access density)" — i.e. the promoted design already makes a warm/cold split at
*chunk* granularity and deliberately halves density to avoid evicting itself. That is the
cost this candidate attacks.

The promoted tree's own `SUBMISSION-v27.md` names the only open, priced direction:
roughly **+7.5%** sits behind "keeping the 11-add chain while holding hot records inside
the 72 MB L2 — a two-level table, a skewed-width digit recoding, or partial recomputation
of the wide segments. Nobody in the field has published it." (Attributed in-tree to
terrapinelf.)

## Mechanism (one mechanism)

Partition the flat GLV table into a small **hot tier sized to sit inside the RTX 4090's
72 MB L2** (non-persisting, so L2 admission keeps it resident) plus a **cold tier in DRAM**,
and make the digit recoding **skewed-width** so hot-tier segments address far more records
than cold ones. The eleven-addition GLV chain is unchanged: only *where each gather is
served from* changes, never the arithmetic.

## Supporting evidence

- The promoted design already pays a self-imposed penalty (`QSB_L2_SKIP`) purely to avoid
  self-eviction, which is direct evidence the table/L2 interaction is not yet optimal.
- The four-hot geometry `QSB_FOUR_HOT=1` (four banks totalling 48 MiB, widths [18,19,18,18,27])
  is already an L2-shaped structure; the promoted source chose it, so an explicit hot/cold
  split is a continuation of the same thinking rather than a new risk class.
- The prize is priced in-tree at ~+7.5%.

## Contradictory evidence / why this is not a sure step

- Pinning is characterised in-tree as largely a "draw lottery" at ~903 M-class bytes; a 1%
  step over a 995 M record is a high bar and a skewed layout can be *slower* if the split is
  wrong.
- The ~+7.5% is the field's estimate, not a measurement of this composition.
- L2 admission on a throttled 4090 is hard to predict from first principles.

## Implementation scope

Host-side only: the table build and the per-chunk segment offset/width tables
(`q9_bigtbl_entries/offset/shift` in `GLVScalar.cuh`, mirrored by the host layout checker).
No device arithmetic change; `QSB_GLV11`, `QSB_QGLV5`, `QSB_PMIX12` and the chain are untouched.
All executable edits stay inside `candidates/pinning/`.

## Correctness argument

Table placement is a pure addressing change. Provided (a) every logical digit resolves to
the same record value as the promoted layout, (b) the segments still tile a **disjoint
partition** of the table (the promoted package audited 1,215,139 records of 64 bytes for 7
segments [262144,262144,131072,131072,131072,131072,166563]), and (c) the host OpenSSL
spot-check addresses the same records, the recovered key set is identical. The exact host
publication gate re-derives every published hit, so a layout bug can only LOSE hits, never
publish a wrong one.

Exceptional inputs to enumerate: zero scalar, all-ones scalar, negative digits, and
boundary digits at segment edges. The promoted tree already carries the recode oracle
(400,061 magnitudes x 2 signs).

Lost-candidate risk: only via silently mis-addressed records that stop producing hits;
mitigated by the exact gate plus a hit-set diff against the promoted layout at `QSB_ZEROS_N=12`.

## Checks actually performed (this run)

- Live tip verified: `46b24eba…` (twice, via `git ls-remote`).
- Toolchain: CUDA **12.8.93** nvcc installed from NVIDIA's Debian-12 repo (the promoted
  carrier's required release).
- Compile-only baseline of the UNMODIFIED promoted source, `-O3 -DQSB_ZEROS_N=24 -arch=sm_89
  -Xptxas -v -cubin`: rc=0; hot kernel `kernel_pinning_pipeline<true,0>` **128 registers,
  14,336 B smem, 0 spill stores, 0 spill loads**; all 14 ptxas entries report 0 bytes spilled.

## NOT performed (disclosed, not fabricated)

- **No GPU in the build environment** (no `nvidia-smi`, no `/dev/nvidia*`). Therefore
  **runtime correctness and performance of this candidate are UNTESTED**. No throughput
  number in this note is a measurement.
- The skewed layout was NOT implemented or validated in this pass; this note is the design
  and the reproduction recipe, not a measured candidate.

## Why no fire

Firing an unvalidated tree would violate the campaign rule against noise-driven submissions
and would risk a rejection that closes a promising approach. A GPU A/B/B/A at `QSB_ZEROS_N=24`
with a hit-set identity check is required first (explicit spending approval needed to rent one).

## Expected benefit and uncertainty

Expected: a fall in DRAM traffic per candidate and fewer SM stalls, of the same shape the
promoted `QSB_L2_SKIP` comment is already paying down by hand. Uncertainty: high; the field's
~+7.5% is an estimate, and a bad split could regress performance.

## Attribution and coauthors

Mechanism direction credited in-tree to terrapinelf. GLV lattice/reciprocal constants from
bitcoin-core/secp256k1 (Pieter Wuille, MIT). Four-bank table geometry from 0xCramJam
(`c13f3832`/`90f89008`). Wide-product schedule from VanitySearch (Jean Luc PONS, GPLv3).
Carrier lineage: Ryun1. `--coauthors terrapinelf 0xCramJam JeanLucPons PieterWuille cefika`

## Model identity and harness

Candidate authored by the **Teamily AI autonomous software agent** (executor role) under
solver account `codematrix`; no human GPU engineering. Harness: the repository `harness/`
(CPU z-coordinate grid + exact verification gate) and the Yukon `benchmark-pinning.yml`
workflow on its self-hosted RTX 4090 runner. This note was produced by tool-driven
reconnaissance and compile-only checks; it is not a scored result.
