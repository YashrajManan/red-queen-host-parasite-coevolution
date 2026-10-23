# Red Queen Dynamics in Host-Parasite Coevolution

### Does coevolution automatically produce perpetual oscillation, or only under specific conditions?

## 1. Pitch

"Coevolution produces Red Queen dynamics" is often stated as if it's a blanket property of host-parasite
systems. This project tests that claim directly for two real, standard coevolutionary genetic architectures
— matching-allele (MA) and gene-for-gene (GFG) — by sweeping their parameters and classifying the resulting
dynamics with an independently validated spectral method, rather than assuming the outcome from the model's
name.

## 2. Research question

Under exactly what parameter conditions does each of the MA and GFG coevolutionary models produce sustained
perpetual oscillation versus settle to a stable equilibrium, and does the literature's expectation that MA
favors Red Queen dynamics more readily than GFG actually hold up when both are swept across comparable
parameter ranges?

## 3. Biological background

**Matching-allele (MA):** two host genotypes, two parasite genotypes. A parasite genotype can only
successfully exploit the matching host genotype — a simplified model of specific immune recognition/evasion.
Host fitness falls with the frequency of the matching parasite genotype; parasite fitness rises with the
frequency of the matching host genotype. This symmetric genotype-vs-genotype matchup is the textbook setup
expected to produce perpetual chasing dynamics.

**Gene-for-gene (GFG):** host alleles `{R, r}`, parasite alleles `{V, v}`. `R` resists `v` specifically, but
`V` overcomes any host — an asymmetric matchup (one allele beats everything, unlike MA's symmetric pairing).
`R` carries a resistance cost even when unneeded; `V` carries a virulence cost even against hosts that didn't
need defeating. The literature generally expects these costs to pull GFG toward a stable polymorphic
equilibrium rather than sustained oscillation, in contrast to MA.

## 4. Dataset

None — the standard, disclosed exception for this cluster. The two coevolutionary models and their swept
dynamics are the object of study, not an external dataset.

## 5. Methods

- Discrete-generation frequency-recursion simulations for both MA and GFG, each with a recurring-mutation
  term preventing permanent fixation.
- A spectral (periodogram) oscillation/equilibrium classifier: late-time burn-in, linear detrending (to
  avoid mistaking a slow monotonic approach to equilibrium for low-frequency oscillation), a variance
  pre-check (near-zero late-time variance is equilibrium by definition, skipping the spectral step), and a
  peak-power-fraction threshold for the oscillation verdict.
- The classifier validated against four synthetic signals with known ground truth (pure sine, exponential
  decay, pure noise, slow monotonic trend) before being trusted on the real simulated dynamics.
- Parameter sweeps: MA over selection strength `s` and mutation rate `mu`; GFG over cost `c = cR = cV`.
- A confirmatory re-run at 3.3x the generation count for any GFG edge case found, checking that late-time
  variance is stable rather than still decaying (ruling out a slow transient misread as sustained
  oscillation).

## 6. Pipeline

1. Implement and sanity-check the MA model (frequencies stay valid probability distributions).
2. Implement and sanity-check the GFG model (same property).
3. Build and validate the spectral classifier against 4 synthetic ground-truth cases.
4. Apply the validated classifier to representative MA and GFG runs; confirm the qualitative contrast.
5. Sweep MA's selection strength and mutation rate.
6. Sweep GFG's cost parameter.
7. Compare MA's and GFG's oscillation fractions directly; re-confirm any GFG edge case at a longer run.
8. Export the hero figure and CSVs.

## 7. Results

The classifier passed all 4 ground-truth checks (sine → oscillating; decay, noise, trend → equilibrium).
The representative-parameter contrast matched the literature's expectation: MA (`s=0.8, mu=0.01`) oscillates
(`power_frac=0.951`); GFG (`cR=cV=0.1`) sits at equilibrium (`power_frac=0.000`).

**MA sweep.** Oscillation requires selection strength `s≥0.2` (no oscillation at `s=0.05` or `0.1`;
oscillating from `s=0.2` through `0.8`) and mutation rate `mu≤0.05` (oscillating at `mu=0.001, 0.01, 0.05`;
equilibrium from `mu=0.1` upward).

**GFG sweep.** Equilibrium at 5 of 6 swept costs (`c=0.001, 0.01, 0.05, 0.1, 0.5`), but a genuine oscillating
exception at `c=0.3` (`power_frac=0.674`). Confirmed non-transient by re-running at `N=5000`: variance over
the last 500 generations (`0.01389`) and the last 2000 generations (`0.01386`) are nearly identical.

**Direct comparison.** MA oscillates across 67% of its swept selection-strength range; GFG oscillates across
17% of its swept cost range — real, measured support for the general claim that MA genetics favors Red Queen
dynamics more readily than GFG, with a confirmed, non-monotonic exception in GFG at an intermediate cost.

## 8. Figures

`results/hero_red_queen.png` — three panels: representative MA vs. GFG trajectories, MA's two oscillation
thresholds (selection strength and mutation rate), and GFG's oscillation-vs-cost curve showing the `c=0.3`
exception.

## 9. Limitations & honest scope

Both models here are the simplest real versions of their architecture: two genotypes per species (not
many-allele), deterministic discrete-generation recursions (not a stochastic finite-population model where
genetic drift could interact with the oscillation/equilibrium boundary found here). A real bug was found and
fixed during this build — a clipping bound typo (`1e6` instead of `1e-6`) that silently switched off host-side
selection in the MA model without throwing any error, caught only by comparing printed numbers line-by-line
against an independent reference solution. Extending to many-allele versions of both models and to a
stochastic finite-population simulation are real, disclosed next steps, not claims already covered here.

## Reproducible code

`red_queen_host_parasite_coevolution.ipynb` — runs top to bottom locally (CPU only, no GPU or dataset
download needed).

## Key tools

NumPy (both model simulations), SciPy (`scipy.signal.periodogram`, `scipy.signal.detrend` for the spectral
classifier), pandas (sweep result tables), Matplotlib (hero figure).

## Skills demonstrated

Discrete-generation evolutionary-dynamics simulation; building and independently validating a classification
method against synthetic ground truth before trusting it on real output; parameter sweeps with explicit
threshold-finding rather than single-point checks; catching a silent, non-error-throwing numerical bug via
line-by-line cross-verification against a reference solution; honest documentation of a genuine non-monotonic
exception rather than smoothing it into a cleaner-sounding narrative.

## Stretch goals

Many-allele extensions of both MA and GFG; a stochastic finite-population version to test whether genetic
drift shifts the oscillation/equilibrium boundaries found here; a post hoc qualitative comparison against a
real published multi-year host-parasite allele-frequency time series (e.g. *Daphnia*-*Pasteuria*).
