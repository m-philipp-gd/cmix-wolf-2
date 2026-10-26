# cmix-wolf P14

Hutter Prize entry: a lossless compressor for `enwik9` (first 10^9 bytes of the
English Wikipedia dump). It is a GPL-3.0 derivative of **cmix-neif-beehive** by
`galqiwi` (<https://github.com/galqiwi/cmix-neif-beehive>, tag `cmix-neif-beehive`,
commit `92c07c0`), which is itself a derivative of **cmix-neif-1.0** by Marcus
Woelk (<https://github.com/marwoelk/cmix-neif-1.0>, commit `1c9ea5e`). All
upstream notices and attributions are retained (`LICENSE`, `NOTICE.md`).

P14 keeps Beehive's trained transformer, its weights, its compiled-in CM
settings, its prefetch tuning and the whole engine byte-for-byte unchanged, and
grafts four context-mixing levers from the cmix-wolf line (P13) onto it.

## Result

| quantity | value |
|---|---|
| `comp9` | 3,942,114 B, sha256 `c985a5a9d1dc2f1be5c30343d0ceaf1b42690d9b364b5421d6f1e877c450c38c`, md5 `df9c851f99587651e246ae6156a01d62` |
| `archive9` | 90,735,088 B, sha256 `a566968be434fd65cc90d1614c62bcfabdfc675fdfa8d6d6337b71dd722058e8` |
| `comp9.args` | 19 B, sha256 `67d0e2c61ec607bee22c7cc9f56368d6f57afba095e08f7e98bd57e2896f57b0` |
| **S = comp9 + archive9 + comp9.args** | **94,677,221 B** |
| vs cmix-neif-beehive (94,705,591) | **-28,370 B = -0.030%** (new leader of the line) |
| vs cmix-neif-1.0 (95,914,051) | -1,236,830 B = -1.290% |
| vs awarded record fx2-cmix-transformer (100,424,672) | -5,747,451 B |

The delta against the best pending entry (-0.030%) is below the 1% step; this
entry is submitted as a fully verified size increment of the cmix-wolf line on
top of Beehive's model, not as a payout step.

## Changes against cmix-neif-beehive

Four CM-layer levers, previously certified at full stream on the cmix-wolf P13
line, grafted onto Beehive's tree (`src/predictor.cpp`, `src/mixer/*`,
`src/models/fxcmv1.cpp`, `src/context-manager.cpp`, `src/id_exclude.h`,
`src/metablock*.h`, `src/word_label.cpp`, `src/ppm_logit_prior.h`, new headers
`src/port_cfg.h`, `src/neif_levers.h`, `src/b1_interp.h`, `src/ts_progress.h`):

- PRL: per-row mixer weight decay;
- E1: interpolated mixer-error feedback;
- E3b: timestamp-window progress features into the mixer (kN=9);
- IND: doubly-indirect context subset restored;
- kGate 0.35: PPMd logit-prior gate - proven **inert on this engine** (a
  control build at 0.55 produced a byte-identical archive on the fixture),
  kept only for source identity with P13.

## Evidence

Fixture (first 16 MB of Beehive's coded stream, sha `7b827308...`):

| build | out16 | note |
|---|---|---|
| Beehive published comp9 | `d94eecc6...` | exactly their published reference |
| our rebuild of their unchanged tree | same | 0 B toolchain drift, streams identical |
| P14 graft | 1,938,126 B = **-23,939 B** | decodes itself bit-exactly, 9.5% faster |

Full stream: solo encode on one pinned core, 34.8 h (program clock; the EU DST
rollback hides one wall-clock hour), rc=0, the `archive9` above. Peak RSS at
Beehive's 9.75 GiB. Full-stream bit-exact round-trip of this submitted
executable is the last gate; the release notes carry its verdict.

## Machines

Result reference: Beehive's AWS EC2 EPYC 7R13 (Geekbench5 single T=1292, limit
54.18 h; Beehive 52.14/50.96 h measured, byte-identical). Our desktop host
measured the full encode/round-trip on one pinned core; P14's decoder side is
within a few percent of Beehive's, projection ~46-49 h, below the limit.

## Build / use

`bash install.sh` (pinned clang/lld 18.1.3), then `bash build.sh`; a second
independent clean rebuild produced the byte-identical `comp9` (sha256 `c985a5a9...`).
Compress: `./comp9 -e enwik9 archive9`. Decompress: `chmod +x archive9 &&
./archive9` in an empty directory - the decoder is bundled, nothing is compiled.

## Docs

Section pointers to "README.md" in source files refer to the upstream README
(Beehive's convention, unchanged) - preserved as `README_cmix-neif-1.0.md`.
Beehive's own README: `doc/README_cmix-neif-beehive.md`. This line's changes:
`CHANGES.md`, `NOTICE.md`, build environment: `environment.txt`. The transformer
weights' training (Beehive's work, untouched here): `training/README.md` sec. 12.
