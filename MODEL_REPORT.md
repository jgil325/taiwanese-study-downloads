# Taiwanese Study 0.3.0: measured model and sampling results

September 30, 2026 · Apple M4 Pro · 48 GiB RAM · eight CPU workers.

All four modes are available. The three new policy families each trained from uniform through 100k, 1M, and 10M self-play deals, with two independent training seeds. Their model quality is still experimental. None meets the 0.02 points per opponent per hand equilibrium research target. More Monte Carlo samples improve the precision of EV against a frozen policy; they do not repair a weak policy.

## Frozen artifacts and independent attacks

Each final audit fixes a frozen average policy before drawing 128 independent decision observations. It uses 25,000 selection scenarios and a separate 25,000 validation scenarios per observation. Gain is measured against the policy-weighted baseline on paired validation outcomes. The SE below includes variation across observations. The attack is an attainable deviation diagnostic, rather than a full best-response solve. The conservative upper bound uses the method described in METHODS.md and remains far above the target.

| Family | Training seed | Training deals | Held-out attack gain | SE | Conservative 95% upper |
|---|---:|---:|---:|---:|---:|
| Classic | 42 | 262,144 | 2.390 | 0.061 | 6.056 |
| Jokers | 9303001 | 10,000,000 | 2.238 | 0.063 | 5.901 |
| Jokers | 9303002 | 10,000,000 | 2.182 | 0.063 | 5.843 |
| Revealed flops | 9303001 | 10,000,000 | 1.424 | 0.070 | 5.090 |
| Revealed flops | 9303002 | 10,000,000 | 1.348 | 0.067 | 5.007 |
| Flops + jokers | 9303001 | 10,000,000 | 1.396 | 0.067 | 5.053 |
| Flops + jokers | 9303002 | 10,000,000 | 1.527 | 0.069 | 5.189 |

The bundle uses seed 9303001 for each new family. Both completed seed runs are reported; the second seed is a robustness check. Classic preserves the previously released average strategy in a separately audited compact artifact. Blind-mode compact tables retain physical action probabilities and omit training histories; they are analysis-only. The two public-mode artifacts retain resumable current/average networks and historical buffers.

| Bundled family | Artifact ID | Compressed MiB | Unpacked MiB | Resumable |
|---|---|---:|---:|---|
| Classic | `selfplay-262144-f0cccff18f0de6c2` | 28.0 | 208.4 | No |
| Jokers | `selfplay-10000000-80ab33b881126b4d` | 891.7 | 2702.0 | No |
| Revealed flops | `selfplay-10000000-8265a0d0eae28d5c` | 3.8 | 28.9 | Yes |
| Flops + jokers | `selfplay-10000000-44feed8f61c062f0` | 3.7 | 28.9 | Yes |

Rules hashes and SHA256 payload digests appear in the accompanying measurement-summary.json. The joker face is W, in a warm plum color; physical joker IDs remain distinct inside the engine and saved data.

## Training stages and compute

Stage audits use independent true-chance decision observations. Most use 64 observations and 10,000 selection/validation scenarios; final audits use the larger budget above. Because observations and seeds differ, stage gain changes are descriptive and require their reported uncertainty.

| Family | Seed | Deals | Attack gain ± SE | Additional stage wall seconds | Peak footprint GiB |
|---|---:|---:|---:|---:|---:|
| Jokers | 9303001 | 100,000 | 2.687 ± 0.113 | 3.55 | 0.386 |
| Jokers | 9303001 | 1,000,000 | 2.458 ± 0.082 | 79.60 | 3.152 |
| Jokers | 9303001 | 10,000,000 | 2.243 ± 0.077 | 1205.71 | 11.843 |
| Jokers | 9303002 | 100,000 | 2.624 ± 0.101 | 5.78 | 0.379 |
| Jokers | 9303002 | 1,000,000 | 2.456 ± 0.102 | 53.14 | 3.147 |
| Jokers | 9303002 | 10,000,000 | 2.099 ± 0.090 | 899.42 | 11.844 |
| Revealed flops | 9303001 | 100,000 | 1.818 ± 0.131 | 8.47 | 0.042 |
| Revealed flops | 9303001 | 1,000,000 | 1.978 ± 0.129 | 78.59 | 0.065 |
| Revealed flops | 9303001 | 10,000,000 | 1.534 ± 0.097 | 826.49 | 0.065 |
| Revealed flops | 9303002 | 100,000 | 2.107 ± 0.169 | 11.47 | 0.041 |
| Revealed flops | 9303002 | 1,000,000 | 1.620 ± 0.115 | 119.57 | 0.064 |
| Revealed flops | 9303002 | 10,000,000 | 1.348 ± 0.067 | See stage log | See stage log |
| Flops + jokers | 9303001 | 100,000 | 2.319 ± 0.147 | 10.03 | 0.042 |
| Flops + jokers | 9303001 | 1,000,000 | 1.851 ± 0.120 | 80.51 | 0.042 |
| Flops + jokers | 9303001 | 10,000,000 | 1.568 ± 0.109 | 875.59 | 0.041 |
| Flops + jokers | 9303002 | 100,000 | 2.235 ± 0.138 | 12.33 | 0.041 |
| Flops + jokers | 9303002 | 1,000,000 | 1.775 ± 0.114 | 131.27 | 0.064 |
| Flops + jokers | 9303002 | 10,000,000 | 1.527 ± 0.069 | See stage log | See stage log |

Public policies use Vector105, 48 features, 16 hidden units, two 32,768-record uniform historical reservoirs, 1,000 deals per frozen batch, 1,280 SGD fit steps per network/batch, linear iteration weights, and warm historical refits. Each deal updates both player roles, so 10M deals means 20M observation updates. Network tensors use f32; sampling probabilities and Monte Carlo statistics use f64. Fits have no momentum/hidden optimizer state. Fit loss is a training diagnostic, not a generalization or equilibrium guarantee.

The blind-joker table uses 5,000 deals/batch and uniform global-batch average weights. Its full 10M checkpoint reached about 12 GiB measured peak footprint, after smaller stages established growth; the compact bundled average uses about 3.15 GiB estimated model memory. The initial 8 GiB target was increased for this measured training workload on the 48 GiB machine. Checkpoint I/O is included in command wall times; engine-only training counters exclude some checkpoint overhead. Concurrent jobs and OS scheduling affect these timings. They are measurements, not exclusive-machine performance promises.

A further compute increase is deferred: the public families improve in these local stages but retain material attack gains. Larger architecture/buffer and independent development folds should be compared before paying for cloud training.

## Public generalization development pilot

The pilot excludes an entire canonical public-key fold: SHA256(key)[0] % 5 == 0 never trains. It records that conditioned chance distribution in its artifact contract and disallows silently resuming a development-fold artifact as a full-game solve. It compares two architectures, each warm/from-scratch, with 8,192 accepted development deals and 16 independent held-out public observations. Shared-action fits use 512 steps; Vector105 uses 1,280 for roughly comparable wall budgets. This is a small pilot, not a certification audit.

| Family | Architecture | Warm historical fit | Seconds | Held-out attack gain ± SE | Policy inference µs |
|---|---|---|---:|---:|---:|
| Revealed flops | shared_action | False | 1.46 | 2.595 ± 0.226 | 21.13 |
| Revealed flops | shared_action | True | 1.44 | 2.529 ± 0.215 | 30.59 |
| Revealed flops | vector105 | False | 1.57 | 2.543 ± 0.227 | 13.74 |
| Revealed flops | vector105 | True | 1.09 | 1.613 ± 0.192 | 9.40 |
| Flops + jokers | shared_action | False | 1.53 | 3.117 ± 0.266 | 24.31 |
| Flops + jokers | shared_action | True | 1.74 | 2.494 ± 0.217 | 27.53 |
| Flops + jokers | vector105 | False | 1.24 | 3.089 ± 0.272 | 10.24 |
| Flops + jokers | vector105 | True | 1.30 | 2.585 ± 0.278 | 12.92 |

Warm Vector105 was selected for lower ordinary-flop development attack gain and faster small-observation inference. The combined-mode architecture quality difference is unresolved in this pilot; its chosen model remains experimental. Release policies train separately on the complete true chance distribution, with fresh final audits.

## CPU and Metal pilot

Burn 0.18.0 was tested in an isolated development crate with f32 shared-action and Vector105 shapes (48 features, 16 hidden units). Batches 1/16/64/256 measured inference plus readback and forward/backward plus gradient readback. CPU/Metal outputs and parameter gradients agreed within 1e−4. These are deterministic tensor fixtures; they are not whole-model strategic quality tests. The report was repeated after sustained training finished, although light build/UI activity remained.

| Architecture | Batch | CPU inference µs | Metal inference µs | CPU gradient µs | Metal gradient µs |
|---|---:|---:|---:|---:|---:|
| shared_action | 1 | 54.8 | 883.5 | 136.4 | 1618.2 |
| shared_action | 16 | 193.9 | 912.4 | 381.7 | 3400.2 |
| shared_action | 64 | 840.4 | 928.2 | 2221.2 | 7398.6 |
| shared_action | 256 | 3054.0 | 2098.1 | 9102.3 | 18621.1 |
| vector105 | 1 | 41.8 | 319.7 | 108.1 | 898.3 |
| vector105 | 16 | 36.3 | 415.8 | 61.8 | 1009.2 |
| vector105 | 64 | 29.2 | 487.8 | 103.1 | 949.3 |
| vector105 | 256 | 53.8 | 435.1 | 289.8 | 4452.1 |

Production retains bounded CPU inference batches of 16 and the tested portable Rust implementation. Small-batch Metal dispatch/readback overhead does not justify adding a GPU dependency here. Burn is not included in the app runtime. Larger batches can favor Metal for some shapes; a larger-network training workload should be remeasured before changing the backend.

## Monte Carlo sample-size results

Each family benchmark uses its bundled frozen artifact, three independent seeds per structure/budget, and 1k/5k/10k/50k/100k/250k/1M scenarios in both fixed and hidden modes. Natural hands, one/two private jokers, public conflicts, monotone boards, and zero/one/two exposed discards are covered where applicable. Below are averages over the hidden-opponent structures; setup/model load is excluded. The selected maximum EV and its SE are descriptive and selection-dependent.

| Family | Independent draws | Seconds | Total EV SE | Leading-two paired SE |
|---|---:|---:|---:|---:|
| Jokers | 1,000 | 0.023 | 0.1998 | 0.1193 |
| Jokers | 5,000 | 0.043 | 0.0890 | 0.0546 |
| Jokers | 10,000 | 0.060 | 0.0629 | 0.0353 |
| Jokers | 50,000 | 0.242 | 0.0281 | 0.0173 |
| Jokers | 100,000 | 0.488 | 0.0198 | 0.0122 |
| Jokers | 250,000 | 1.160 | 0.0125 | 0.0069 |
| Jokers | 1,000,000 | 4.363 | 0.0063 | 0.0034 |
| Revealed flops | 1,000 | 0.014 | 0.1775 | 0.1242 |
| Revealed flops | 5,000 | 0.039 | 0.0788 | 0.0567 |
| Revealed flops | 10,000 | 0.069 | 0.0559 | 0.0401 |
| Revealed flops | 50,000 | 0.213 | 0.0250 | 0.0179 |
| Revealed flops | 100,000 | 0.434 | 0.0177 | 0.0127 |
| Revealed flops | 250,000 | 1.092 | 0.0112 | 0.0080 |
| Revealed flops | 1,000,000 | 4.331 | 0.0056 | 0.0040 |
| Flops + jokers | 1,000 | 0.007 | 0.1854 | 0.0878 |
| Flops + jokers | 5,000 | 0.027 | 0.0816 | 0.0391 |
| Flops + jokers | 10,000 | 0.046 | 0.0579 | 0.0276 |
| Flops + jokers | 50,000 | 0.242 | 0.0259 | 0.0123 |
| Flops + jokers | 100,000 | 0.477 | 0.0183 | 0.0087 |
| Flops + jokers | 250,000 | 1.100 | 0.0116 | 0.0055 |
| Flops + jokers | 1,000,000 | 4.796 | 0.0058 | 0.0028 |

Quick remains 100k, Standard 250k, and Deep 1M independent opponent draws. Actual SE and simultaneous refinement-valid grading determine confidence. Close choices can remain unresolved even at 1M. All 105 settings share each opponent draw, sampled setting, and board pair. Home Run joint events are counted directly; zero observed events receive bounded intervals rather than a zero-probability claim.

## Opponent reuse and calibration

K=1/4/16 pilots choose draw budgets from separate timings and report actual elapsed time. SE uses independent opponent-cluster means. A preselected probe action avoids choosing the lowest-variance action after seeing results. Only one representative pairs structure is used for this comparison; the result does not establish a universal K.

| Family | K | Seconds | Probe SE | Leading paired SE | Seconds × probe SE² |
|---|---:|---:|---:|---:|---:|
| Jokers | 1 | 0.573 | 0.01409 | 0.01056 | 0.00011382 |
| Jokers | 4 | 0.419 | 0.01389 | 0.00826 | 0.00008073 |
| Jokers | 16 | 0.464 | 0.01946 | 0.00828 | 0.00017595 |
| Revealed flops | 1 | 0.397 | 0.01366 | 0.01190 | 0.00007404 |
| Revealed flops | 4 | 0.473 | 0.01006 | 0.00731 | 0.00004786 |
| Revealed flops | 16 | 0.563 | 0.01197 | 0.00722 | 0.00008071 |
| Flops + jokers | 1 | 0.505 | 0.01395 | 0.00339 | 0.00009821 |
| Flops + jokers | 4 | 0.451 | 0.01285 | 0.00231 | 0.00007437 |
| Flops + jokers | 16 | 0.473 | 0.01450 | 0.00163 | 0.00009957 |

The default remains K=1 for broad independent opponent coverage. Reuse can improve efficiency on some holdings; the app exposes it without treating MK board pairs as independent opponents.

| Family | Covered / 32 fixed-action intervals | Standardized mean | Standardized SD | Reference EV ± SE |
|---|---:|---:|---:|---:|
| Jokers | 32 / 32 | 0.285 | 0.861 | -1.62426 ± 0.01152 |
| Revealed flops | 32 / 32 | -0.137 | 0.829 | -3.62254 ± 0.01111 |
| Flops + jokers | 30 / 32 | 0.628 | 0.873 | -2.32092 ± 0.01057 |

Action 0 was fixed in advance. Each reference uses an independent 250k-scenario run with its own SE; uncertainty is combined for comparison. Sharing that reference correlates the calibration comparisons. Thirty-two seeds are diagnostics, not proof of exact nominal coverage. Full per-case top-five stability, fixed-matchup results, Home Run counts, and seeds are retained in benchmark CSV/JSON.

## Conditional strategy stress diagnostics

Each declared stratum uses 16 independently generated true-chance observations conditioned on that stratum. Every attack uses 10k selection and separate 10k validation scenarios against the frozen average. The paired inner estimate and full observation are retained. Outer descriptive SE uses independent observations; strata overlap and cannot be pooled as a full-game estimate.

| Family | Conditioned stratum | Held-out attack gain | SE |
|---|---|---:|---:|
| Jokers | private_0 | 2.226 | 0.151 |
| Jokers | private_1 | 2.492 | 0.166 |
| Jokers | private_2 | 3.215 | 0.185 |
| Revealed flops | paired | 1.189 | 0.131 |
| Revealed flops | monotone | 2.163 | 0.318 |
| Revealed flops | connected | 1.910 | 0.165 |
| Revealed flops | overlapping_ranks | 1.472 | 0.213 |
| Flops + jokers | private_0 | 1.418 | 0.199 |
| Flops + jokers | private_1 | 1.468 | 0.171 |
| Flops + jokers | private_2 | 1.902 | 0.277 |
| Flops + jokers | paired | 1.294 | 0.212 |
| Flops + jokers | monotone | 1.364 | 0.184 |
| Flops + jokers | connected | 1.367 | 0.231 |
| Flops + jokers | overlapping_ranks | 1.293 | 0.154 |
| Flops + jokers | discard_1 | 1.560 | 0.157 |
| Flops + jokers | discard_2 | 1.423 | 0.245 |

These deliberately rare private-joker, public-discard, and flop-texture checks can reveal weaknesses hidden by natural-frequency averaging. They are small diagnostic samples and do not certify the conditional strategies.

## Reproduction and practical limits

The source repository records staged commands, checksummed rule/model contracts, full benchmark CSV/JSON, development-fold keys, CPU/Metal fixtures, independent attack observations, and conditional stress observations. `measurement-summary.json` accompanies the public release. Rule/evaluator correctness, Monte Carlo calibration, and model quality have separate evidence.

The app uses Monte Carlo for all equity/runout EV calculations. It scores individual showdowns exactly, retains all 105 settings and global suit/board/joker symmetries, and excludes hidden cards/future boards from policy inputs. Existing Classic sessions/models remain readable. The universal Mac binary includes arm64 and x86_64; native runtime verification was on Apple Silicon, not physical Intel hardware. It is ad-hoc signed and not Apple-notarized. Allow 6 GB of free disk and preferably 8 GB RAM. Updating the app preserves local models and saved hands.
