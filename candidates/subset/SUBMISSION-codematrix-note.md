# Subset candidate note — solver `codematrix`

STATUS: PREPARED, NOT QUALIFIED, NOT SUBMITTED. No Yukon validation was fired.

## Base SHA and frontier score

- Promoted base: `46b24ebaa033fb69c7335794b54fd6a156359ec8` (verified live this run).
- Benchmark id (subset): `b352879c-669f-44ef-98cd-ad3d34d0fefa`.
- Live promoted frontier: **700,953,730**. Baseline 61,997,057. Gate 1%: floor **707,963,267**.
  NOTE: the historical brief pair mis-assigned 995,329,477 to subset; that figure is the
  **pinning** frontier (cefika). Corrected here from the live dashboard payload.

## Bottleneck

The tracked subset score is GPU kernel throughput. The promoted lineage already ships
host-built epoch producers and a co-grinder, so the open lane is the GPU **digest** cost.
The promoted pinning tree contains a SHA redesign (`sha_pinsha.cuh`, constant-folded
literal-K round transforms) that the subset tree has **not** ported.

## Mechanism (one mechanism)

Port the promoted pinning tree's constant-folded SHA-256 transform work into the subset
digest lane, applied to the subset preimage's ~1.7 KB rebuilt block, keeping the subset
carriers and editable surface otherwise identical.

Why it is materially different from the closed subset approaches: it is not a
`QSB_SHA_FMA_ADD` toggle (already 0), not the closed "paired-SHA ALU/full unroll", not
prep-overlap, not split-verify, not a compiler-flag sweep, and not any carry-chain change.
It is a code-generation change to the SHA compression that the pinning track already promotes.

## Correctness argument

SHA-256 is a fixed standard; a constant-folded compression must be bit-identical. Required
evidence before any fire: a lane-block diff against OpenSSL `SHA256_Transform` over >=1e6
blocks at `QSB_ZEROS_N=24`; a hit-set identity run at `QSB_ZEROS_N=12` against the promoted
kernel; a fresh-seed harness run. The exact hit gate means a mismatch drops hits rather than
publishing wrong ones.

Lost-candidate risk: a mis-folded round constant silently changes every digest and yields
ZERO hits — immediately visible as the exact gate's hit count collapsing, which is the
failure mode the promoted runbook says to watch for.

## Contradictory evidence / uncertainty

The subset note warns that on the ranked Zen 4 host the digest is not always the dominant
cost (tables/prefetch can be), so the ported folding may be small or nil. An unresolved
question is whether ptxas already constant-folds the `__constant__ K[]` reads in the shared
`GPUHash.h` when the index is compile-time; if it already does, this mechanism is INERT and
must be closed rather than submitted. That must be settled by SASS inspection before firing.

## Checks actually performed (this run)

- Toolchain CUDA 12.8.93 installed; compile-only baseline of the UNMODIFIED promoted subset
  source, `-O3 -DQSB_ZEROS_N=24 -arch=sm_89 -Xptxas -v -cubin`: rc=0; all kernels report
  **0 spill stores / 0 spill loads**.
- Native sm_89 carrier regenerated on a scratch copy: `qsb_carrier_sm89.h`, 462,496 B cubin,
  sha256 `003e3d39b7a6283f…`, 5,139 lines, LTC64B loads in digest: 3.

## NOT performed (disclosed, not fabricated)

- **No GPU in the environment**, so runtime correctness and performance are UNTESTED.
- The digest port was not implemented or SASS-verified in this pass; this is a design note.

## Attribution and coauthors

Constant-folded SHA work from the promoted pinning tree (`sha_pinsha.cuh`, PR lineage
including DPZZxlz #1891). SHA-256 transform from VanitySearch (Jean Luc PONS, GPLv3).
Paired-recovery-SHA idea (unpromoted 5605ad8) by nullforest8200.
`--coauthors nullforest8200 JeanLucPons DPZZxlz`

## Model identity and harness

Authored by the **Teamily AI autonomous software agent** under solver account `codematrix`.
Harness: repository `harness/` and the Yukon `benchmark-subset.yml` workflow on its
self-hosted RTX 4090 runner. Not a scored result.
