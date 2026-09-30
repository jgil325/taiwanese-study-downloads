# Four-mode calculation, policies, and validation

September 30, 2026. Rules are implemented as complete versioned contracts (`src/rules.rs`) and SHA256-bound model manifests. Classic history retains its original rules ID. New caches include the full observation, rules contract, frozen content-addressed model, seed, sample/group budget, sampler version, and grading look.

## Rules and exact evaluation

All four modes keep 105 physical 1/2/4 arrangements, Hold'em construction for Top/Middle, exactly two private plus three board cards for Bottom, zero-point ties, 1/2/3 per-board row weights, and eight points for six outright wins. Two disjoint boards use one deck. Maximum payoff is ±20 per opponent.

Private hands are dealt before public flops. In either joker mode, board dealing permanently removes and replaces jokers; successive replacements are handled. Combined mode exposes the exact pre-setting discard set E, including an observed empty set. Later replacements are not policy inputs. The API rejects omitted E in fixed combined observations instead of silently treating omission as None.

For each selected physical five, one/two jokers maximize a legal standard hand with distinct virtual rank/suit identities. A joker may represent a card elsewhere in the physical deal. Five of a kind is forbidden. Assignments can differ across boards and rows. The optimized feasibility kernel enumerates standard rank categories; ordinary combinations retain the pinned upstream evaluator. Independent substitution tests use a separate rank classifier and cover generated five-, six-, and seven-card holdings. Slow substitution witnesses are used only when inspecting an actual runout, showing both physical and represented cards and preserving Omaha construction.

## Conditional Monte Carlo

Hero H contains seven private cards; F contains six ordinary public flop cards; E is the exact exposed discard set. Each independent scenario draws one opponent hand, samples that hand's setting from its frozen policy, and samples one board pair. All 105 hero arrangements use this same scenario.

| Mode | Uniform compatible opponent pool | Ordinary cards available for boards |
|---|---:|---:|
| Classic | 45 | 38, draw ten |
| Blind jokers | 47 | 38+h+o, draw ten |
| Revealed flops | 39 | 32, draw four completions |
| Flops + jokers | 41−e | 32+h+o, draw four completions |

Here h/o are private joker counts and e=|E|. No joker is returned to the deck. Uniform conditional opponent sampling given exact H/F/E is valid under the hands-first permanent-replacement protocol; independent reduced-deck enumeration verifies this posterior, including informative empty E. Public flops without a discard observation would define a different mixture and are deliberately not silently accepted.

Opponent policy inference receives only its own H and public F/E. Scoring may inspect both private hands and completed boards, but strategy selection cannot. Pure observation-only interfaces, information-boundary tests, frozen batches, and worker-invariant seeded results enforce this separation.

Welford moments on complete payoffs preserve dependence between rows, boards, and bonuses. Total EV SE is s/sqrt(N); paired arrangement differences use their own moments. With K=4/16 boards per opponent, the M independent opponent-group means determine SE; MK outcomes are not treated as independent. Home Run event counts, bounded cluster-rate probability intervals, and bounded total EV intervals guard against overconfidence with zero or rare events. Simultaneous, refinement-valid conservative bounds retain neutral labels for unresolved alternatives.

## Symmetry and learned policies

Blind modes use exact canonical private-hand tables. Blind jokers has 7,106,606 possible classes. Public observations jointly canonicalize H/F/E with one global suit permutation, preserve board membership, allow board swaps, and relabel equal-power joker identities. Remaining action automorphisms are averaged exactly. All 105 physical actions remain available.

Public modes require generalization. Separate regret and average-policy networks use 48 features and 16 hidden ReLU units. Forty features losslessly encode canonical private/public card zones; eight add legal made-hand strengths and rank/suit matches on each known flop. These features never use an opponent's hand or turn/river. Shared per-action scoring and a vector of 105 outputs are both implemented and tested. Model artifacts record architecture, dimensions, precision, evaluator/canonicalization/action versions, training method, weighting, sampler, and resume state.

Training samples the true variant chance distribution and both player roles. Current policies are frozen for a complete batch. All-action regret targets subtract the frozen acting policy's payoff against the sampled opponent. Two independent bounded reservoirs preserve historical regret targets and current-strategy targets with uniform reservoir inclusion. Public targets use linear iteration weights and weighted losses. The average strategy uses cross-entropy against historical distributions, separately from regret regression. Sampling propensity is true chance and own reach is one in this one-decision game; no off-policy importance correction is needed.

Fits can restart from scratch or continue warm while refitting the historical buffer. Both are compared in development pilots. Warm continuation is recorded explicitly and has no hidden optimizer/momentum state. Deterministic fit seeds, networks, both reservoirs, seen counts, weights, and budgets are checkpointed; resume agrees across worker counts. Parameter gradients are checked against independent finite differences. Reservoir inclusion and historical mixture weighting have separate tests. Exact small Bayesian/RPS games test tabular regret matching; an independent RPS fixture tests generalized regression and average-policy recovery. These fixtures validate mechanics, not full-game convergence.

Burn 0.18.0 was evaluated separately with CPU and Metal backends, f32 inference and gradient readback, and batches 1/16/64/256. It is not a runtime dependency. The production backend is deterministic bounded-batch Rust CPU inference, avoiding GPU/framework startup and keeping Intel/macOS portability. See measured pilot files and RESULTS.md. Learned approximation is an explicit source of strategic error beyond Monte Carlo SE.

## Independent audit bound

A frozen model and audit budgets/seeds are fixed before a certification audit. Outer observations come independently from the true variant distribution. One independent seed selects an attack; another measures its gain relative to the policy-weighted baseline on paired scenarios. Reported attack gain and SE are diagnostics and are generally lower than the true best-response gap.

For one observation, let d_a=Q(a)−sum_b p_b Q(b), and g=max_a d_a. Each sampled paired deviation is in [−40,40]. Allocate alpha_inner/105 to each action's empirical Bernstein upper bound, and let U=clip(max_a upper(d_a),0,40). Conditional on the observation, P(g>U)≤alpha_inner. Because g,U are bounded by 40, E[g|observation]≤E[U|observation]+40 alpha_inner, even on failed inner intervals. The implementation uses alpha_inner=0.025/predeclared_outer_N and the usual sample-variance Bernstein radius with range width 80.

The U observations are i.i.d. in [0,40]. An outer empirical Bernstein upper bound at alpha=.05 on E[U], plus 40 alpha_inner, is a conservative 95% bound on the game's expected best-response gap. This uses an expectation correction, not an assumption that every inner interval succeeds. Cancelled audits return upper bound 40. The outer budget now supports up to 1,000,000 observations; a zero-gap fixture confirms that the mathematical design can support 0.02 with adequate inner precision. Actual models do not reach this target. Development comparisons, post hoc selection, and repeated audits are not certification; a fresh predeclared independent audit is required for such a claim.

## Artifacts and user data

Format v2 artifacts bind complete rule/model contracts and payload checksums. The separate v1 reader loads the original public-release binary without rewriting its identity. Old saved results default to Classic and remain immutable. Existing databases receive a one-time `VACUUM INTO` backup before variant access.

Compact blind-mode average artifacts store normalized f32 action probabilities; sampled probability agreement is tested to <1e−7 against f64 histories, and the compact artifact receives its own audit. They are analysis-only. Full checkpoints retain regrets/history and are resumable. Public artifacts retain bounded training buffers. Atomic checkpoint retention retires only intermediates created by the current run after a successful replacement; existing milestones are preserved.

The desktop package installs one checksummed compatible artifact per family and writes an app-owned recommendation catalog. It never copies developer sessions and never replaces existing user models or notes. A damaged existing bundled artifact is preserved and reported. Tests cover four-family installation, repeat installation, digest failure cleanup, database backup, public context persistence, and shutdown checkpointing.
