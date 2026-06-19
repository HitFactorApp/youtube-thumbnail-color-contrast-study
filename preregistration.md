# Pre-registration: Thumbnail Color & Contrast vs. Views

> The analysis plan, fixed and publicly tagged (`v0-preregistration`) before any query relating
> colorfulness or contrast to views. The tagged commit is the audit trail. Deviations after this
> point are allowed but logged in Transparent Changes at the bottom.

**Status:** PRE-REGISTERED (committed + tagged 2026-06-17)
**Design:** [design-spec.md](./design-spec.md)

## Prior exposure

First look. We have not computed any within-channel relationship between thumbnail colorfulness or
contrast and views on this (or any) cohort. Our loose prior is that the "make it pop" advice is at
least weakly real (some positive slope), but we register both slopes as directional-positive only
because that is the claim under test; the private prior is not the registered hypothesis. We
pre-register to stop ourselves searching specifications after seeing the outcome; the real protection
is the sealed whole-channel holdout below.

## Could the predictors be circular?

No. Colorfulness (Hasler–Süsstrunk) and RMS luminance contrast are deterministic functions of the
thumbnail image pixels. No views, watch-time, click, or engagement signal enters either formula at
any stage, and neither predictor was selected by VPD fit (there is no model and no training). The
risk is therefore not circularity and not a biased detector (there is no detector); it is **image
measurement fidelity** — a placeholder/garbage image, a resolution artifact, a color-space bug, or a
swapped thumbnail. All four are addressed by the measurement gate below.

| Predictor | Built from | Outcome signal in it? | Selected by VPD fit? | Predictive claim allowed? |
|---|---|---|---|---|
| Colorfulness (Hasler–Süsstrunk) | thumbnail pixels, fixed formula | no | no | yes (within-channel, conditional on the residual confounder §confounders) |
| RMS luminance contrast | thumbnail pixels, fixed formula | no | no | yes (same) |

## Sample conditioning

**None.** Every thumbnail has a colorfulness and a contrast value, so the cohort is the full
qualifying population — there is no predictor-defined subset and therefore no collider of the kind
a sample restricted to thumbnails that have a feature would have to bound. The only per-channel restriction is a **variation filter**: a channel
enters a slope only if it has ≥ `MIN_VIDEOS_PER_CHANNEL_FE = 10` qualifying videos *and*
non-degenerate within-channel variation in that predictor (a channel whose thumbnails are all
near-identical in color cannot identify a within-channel slope). This filters on the predictor's
*variance*, not its *level*, so it does not condition on high vs. low color.

**The variation filter is not asserted benign — it is made falsifiable.** It changes who the
estimand is about (channels that vary their thumbnail color/contrast), and that selection is benign
for the *slope* only if the slope does not itself vary with how much a channel varies its thumbnails.
A channel that deliberately varies color may A/B-test packaging and have a steeper (or, if already
optimized, flatter) within-channel slope, which would make the estimated slope a selected-high (or
selected-low) value, not just a less generalizable one. We therefore **pre-commit to the SD-tercile
falsification test:** split qualifying channels into terciles of their own within-channel predictor
SD and report the slope in each tercile. A flat slope across terciles confirms the filter is benign;
a slope that trends with the channel's variation is effect-modification by the selection variable and
is disclosed as a bias direction, not a generalization footnote. This is run on the exploration set
and reported on the holdout.

## Hypotheses

- **H1 (primary, colorfulness):** Within the same channel and controlling for age, a thumbnail that
  is **more colorful** (higher Hasler–Süsstrunk colorfulness) is associated with **higher** views.
- **H2 (primary, contrast):** Within the same channel and controlling for age, a thumbnail with
  **higher RMS luminance contrast** is associated with **higher** views.
- Direction registered positive because that is the "make it pop" claim; the two predictors are
  tested and reported as **two separate headlines**, never summed.
- **Exploratory:** whether each slope differs by niche, channel size, and short vs. longer videos;
  whether it survives a `has_face` covariate, a text-density covariate, and the mutually-adjusted
  (colorfulness ⊥ contrast) decomposition; the within-channel decile shape of each.

## Primary analysis

- **Frame:** the full qualifying cohort on the study's frozen store (no predictor subset).
  `views ≥ 10`, `age ≥ 90 days`. Cohort = English-language **business** YouTube channels with ≥ 20
  qualifying videos, public + organic + regular long-form uploads (not Shorts), published 90 days to 5
  years ago. (~16,738 English+business channels in the classification set; the broad long-form corpus
  across them is the population.)
- **Locked inclusion constants:** `VIEW_FLOOR = 10`, `MIN_AGE_DAYS = 90`, `MIN_VIDEOS_PER_CHANNEL = 20`
  (cohort), `MIN_VIDEOS_PER_CHANNEL_FE = 10` (slope). Three further constants are **locked as rules
  now** (the rule is the pre-registration; the resolved number is filled in from the exploration set
  before any holdout read and recorded in the Transparent Changes / methodology, so none is a
  forking path):
  - **`SD_global` (the headline unit denominator):** the standard deviation of the predictor computed
    once across **all exploration-set videos** (video-level, not channel-averaged), one value per
    predictor. A descriptive statistic of the data, not a tuned choice.
  - **`MIN_PREDICTOR_SD_PER_CHANNEL` (the variation floor):** a channel enters a predictor's slope
    only if its within-channel SD in that predictor is **at or above the 5th percentile of the
    per-channel within-channel-SD distribution** (computed on exploration, per predictor). This drops
    only the bottom ~5% of channels whose thumbnails barely vary (a degenerate-variation cut, not a
    level cut); the count dropped is reported, and the SD-tercile check (§Sample conditioning) tests
    that this selection does not bias the slope.
  - **Resize target:** both metrics are computed on the thumbnail resized to a **480px short side**
    with **area-average (INTER_AREA) downscale**. 480 is the most universally available non-tiny
    YouTube thumbnail size (`maxresdefault`/1280 is missing for many older/smaller videos, so anchoring
    there would force fallbacks and break the resize-invariance gate); area-average is the correct
    interpolation for color statistics (it averages pixels rather than dropping them). The
    resize-invariance gate (§measurement gate) validates this choice against a second resolution.
- **Predictors (pinned formulas):**
  - **Colorfulness** `C`: on `rg = R − G`, `yb = 0.5·(R + G) − B` over all pixels,
    `C = sqrt(σ_rg² + σ_yb²) + 0.3·sqrt(μ_rg² + μ_yb²)` (Hasler–Süsstrunk 2003). Channel order RGB,
    8-bit values 0–255.
  - **Contrast** `K`: relative luminance `Y = 0.2126·R' + 0.7152·G' + 0.0722·B'` on **linearized**
    sRGB (`R',G',B'` via the standard sRGB piecewise transfer function, 2.4 gamma), then `K = std(Y)`
    across pixels (RMS contrast).
  - Both computed on the image resized to a **fixed 480px short side** (area-average downscale),
    over the full frame (letterbox bars not stripped in the primary metric).
- **Outcome:** `y = ln(views)`. **Single-measurement-clock:** the cohort capture records one frozen views
  snapshot per video in a short window, so age is *when we measured*, not a video property; raw VPD
  would smuggle measurement timing into the outcome. We use age-controlled `ln(views)` (algebraically
  the same model as age-controlled `ln(VPD)` with the artifact moved out of the outcome). The naive
  raw-VPD version is reported alongside as a cross-check.
- **Standardization (the headline unit — global SD, channel-invariant).** The predictor is
  standardized by a **single global SD computed once on the exploration set** (`SD_global`,
  recorded here once chosen, before any holdout read), not by each channel's own SD. A within-channel
  SD is noisy when estimated from ~9 residual degrees of freedom (CV ≈ 24% at 10 videos), and dividing
  the regressor by that noisy per-channel scale is multiplicative measurement error in the regressor
  that attenuates the slope worst on the small-n channels the median estimator weights most heavily;
  it also makes "+1 SD" mean a different raw amount of color in every channel, so the median and
  pooled estimators would be blending incommensurable units and their agreement gate would be
  hollow. A global SD is a constant: identical across channels, **not re-estimated inside the
  bootstrap**, so "+1 SD" is one fixed raw increment everywhere and the two estimators are in the
  same unit. The within-channel-SD-standardized slope is reported as a descriptive companion (below),
  never as the headline.
- **Model (per predictor):** channel fixed effects (within-channel demeaning) of `y ~ x + ln(age)`,
  where `x` is the predictor minus its channel mean, divided by `SD_global` (so the coefficient is
  the % change in views per +1 global-SD of the predictor, holding `ln(age)` fixed, identified purely
  within channel). Restricted to channels meeting `MIN_VIDEOS_PER_CHANNEL_FE` with within-channel
  variation in `x` above the variation floor.
- **Two co-equal estimators (per predictor):** (i) **median of per-channel slopes** (one channel,
  one vote) and (ii) the **pooled within-channel FE slope** (video-count weighted). A slope is
  claimed only when the two agree in sign and rough size; on divergence we report both and lead with
  the smaller-magnitude (conservative) one. We report the per-channel video-count distribution and
  flag any single channel whose leverage dominates the pooled slope.
- **Headline number (per predictor):** % change in views per **+1 global SD** of the predictor =
  `100·(exp(slope) − 1)`, with a whole-channel bootstrap 95% interval (10,000 resamples, fixed
  `seed = 0`, synthetic cluster ids so a channel drawn twice stays two clusters). The channel
  **demeaning** is re-done inside every resample (each drawn channel demeaned by its own resampled
  mean, so the within-channel centering is not treated as known); the **`SD_global` scale is fixed**
  (it is a constant from the exploration set, not a per-channel quantity, so there is nothing
  channel-specific to re-estimate). Computed on the holdout **once**; the exploration-set value is the
  confirming companion. Before the holdout read, the exploration interval half-width is checked against
  the ±10% bar (power go/no-go, below).
- **Two estimators in the same unit.** Both the median-of-per-channel slope and the pooled FE slope
  are now in the one global-SD unit, so their "agree in sign and rough size" gate compares like with
  like. A divergence means heavy-tail leverage from high-video-count channels pulling the
  video-weighted pooled slope away from the one-channel-one-vote median; on divergence we lead with
  the smaller-magnitude (conservative) one, as fixed in the design spec.
- **Confounder handling:** channel fixed effects absorb channel size, niche, audience, and house
  style; `ln(age)` removes the single-clock artifact and, because age relabels publish date under the
  single clock, absorbs much of publish-era confounding (so the publish-year sensitivity is
  corroboration, not an independent check — important here because thumbnail aesthetics trend strongly
  with era). We do not control click-through rate (mediator) or shelf placement (collider). **The
  unremovable residual:** colorfulness/contrast travel with other packaging choices we do not observe
  (topic, edit intensity, subject, branding, on-image text). Channel FE removes what is stable about a
  channel's style, not the within-channel correlation between "this video got a vivid thumbnail" and
  "this video was a bigger-swing topic." We cannot separate them, so we claim only the within-channel
  association, never the causal effect of recoloring a fixed thumbnail.

## Analyses fixed in advance

The two headlines are the **marginal within-channel SD slopes** for colorfulness and for contrast,
reported separately. These are fixed before the holdout is touched; each of the following can change
or veto how a headline is reported, and each appears in "what has to be true to publish":

1. **The two marginal slopes (headlines):** `ln(views) ~ x + ln(age)`, channel FE, full cohort, one
   fit per predictor.
2. **Decile shape guard (per predictor):** the within-channel decile curve of the predictor overrides
   the single slope if it is real and non-monotone in a way the slope hides (e.g. a colorfulness
   sweet-spot — helps then hurts). **Firing contract (pinned, single named contrast to avoid a
   multiple-comparison scan):** deciles are formed on the **pooled within-channel-demeaned** predictor
   (each video's predictor minus its channel mean), bin edges fixed on the exploration set; the
   age-controlled within-channel-residual mean `ln(views)` is computed per decile with whole-channel
   bootstrap intervals. The pinned contrast is a **turning point**, not a deviation from the curve
   midpoint (a monotone curve has every interior decile lying *between* its neighbours, so a
   midpoint contrast would mislabel a clean slope): for each interior decile (2–9) take its value
   minus the mean of its **two immediate neighbours**, pick the single decile whose local turn is
   largest *opposite* to the overall slope direction (a dip when the slope is up, a peak when down),
   and **fire iff that turn is opposite in sign to the overall slope AND its whole-channel bootstrap
   interval excludes zero**. One pre-named decile and one contrast, not a scan. On a fired guard the
   result is reported as "shape, see the curve," not led by the single slope. **Edge cases (fixed
   here):** if no interior decile has an opposite-signed turn (a clean, possibly noisy monotone curve),
   the candidate set is empty and the guard does not fire. The reference direction is the headline
   slope's sign; if the headline is a precise null or inconclusive (no signed slope to hide a shape
   behind), the guard is not evaluated.
3. **Mutually-adjusted decomposition (veto over the "two findings" framing).** Colorfulness and
   contrast are correlated, so two large marginal slopes can both reflect one shared "vividness"
   factor. Each slope is re-fit with the *other* predictor as a within-channel covariate, and the
   within-channel `C`↔`K` correlation `r_CK` is reported as a **gating** quantity. We lead with the
   marginal slopes, but the **two-separate-headlines framing is licensed only when both predictors
   clear the ±10%-per-SD bar in the *mutually-adjusted* fit.** Pre-committed: if a predictor's
   mutually-adjusted slope drops under the bar or flips sign while its marginal slope cleared it, that
   predictor is reported as "carried by shared vividness, not separable from [the other]," **not** as an
   independent positive finding. Separability tripwire, checked on exploration before the holdout read:
   if `r_CK` is high enough that the mutually-adjusted intervals inflate past the power bar (variance
   inflation computed on exploration), we declare up front that the design cannot separate the two and
   the honest headline is a single "vividness" statement. This is the decomposition's analogue of the
   power go/no-go, and it has veto power exactly like the decile guard.
4. **Within-channel-within-era slope (publish-year FE nested in channel) — confirmatory, with veto
   power.** This is the instrument for the sharpest within-channel confounder of an aesthetic
   predictor: a channel that both grew and got more colorful over the years has a within-channel time
   trend in colorfulness *and* in views that channel demeaning (a level removal) does not touch and
   that `ln(age)` (a single linear age slope) cannot flexibly absorb. We re-fit each headline with the
   demeaning done within **channel × publish-year** cells (publish-year FE nested in channel). It is
   *more* aggressive than `ln(age)` — it absorbs arbitrary year-to-year shifts — so it is the strong
   test, not corroboration: survival is the evidence the slope is not era-trend leakage; collapse is
   the finding that it was. Required to **agree in sign and rough size** with the headline to publish a
   positive. Nesting costs degrees of freedom (a channel needs predictor variation *within* a
   year-bucket to qualify), so we report this fit's qualifying-channel count and treat a large drop as
   itself informative.
5. **Letterbox-stripped fork (per predictor):** re-run with letterbox/pillarbox bars detected and
   excluded before the metric. A stable slope rules out a framing-bar artifact; a large move means the
   metric is partly measuring black bars.

Everything else is descriptive: FDR-corrected at `q = 0.10` within its family, cannot veto a
headline, logged in the lab-wide run registry.

- **`has_face` covariate on/off:** a vivid thumbnail may also be a face thumbnail; reuse an
  off-the-shelf detector's presence flag as a covariate. A check, not an expected mover.
- **Text-density covariate on/off:** high contrast often co-occurs with bold text overlays; an
  off-the-shelf text-area fraction as a covariate. A check, not a fix.
- **Length covariate / short-long split:** headline with and without `ln(duration)`, reported
  separately for short (≤ ~3 min) and longer videos.
- **Standardization companion:** the within-channel-SD-standardized slope beside the global-SD
  headline (it is the companion, never the headline; see the standardization note above).
- **SD-tercile slope (variation-filter falsification, §Sample conditioning):** the slope split by
  tercile of the channel's own within-channel predictor SD; a trend flags effect-modification by the
  selection variable.
- **Per-niche / per-channel-size spread:** each slope across niches and size bands (median + range).
- **Outlier legs:** winsorize views at the 99th percentile and drop the top 1% of channels.
- **Predictor within/between variance split (the global-SD interpretability disclosure).** On
  exploration, report the ratio of the median within-channel predictor SD to `SD_global` (equivalently
  the predictor's within-channel ICC), per predictor. This states how many typical within-channel SDs
  a +1-global-SD bump corresponds to, so "+1 global SD" is interpreted honestly (a fixed channel-
  invariant unit, not "one channel's own range") and a reader cannot mistake the headline for a swing
  no channel could execute within its own catalog. Computed before the holdout read.
- **Era-nested vs headline channel-set composition.** The within-channel-within-era veto (item 4)
  identifies only on channels with predictor variation *within* a publish-year bucket, a subset of the
  headline channels. We report that subset's size and its composition versus the headline set (size,
  niche, within-channel-SD tercile), alongside the d.o.f. drop, so a compositional shift toward the
  more-variable (deliberate-A/B-testing) channels is disclosed rather than hidden inside the veto.

## Outlier handling

We work in log space (outlier-robust by construction) and report each headline three ways (plain log,
views winsorized at the 99th percentile, top-1%-of-channels dropped), requiring agreement in sign and
rough size. Viral videos are signal and never dropped on outlier grounds; the only rows removed are
sub-floor view drops, decode failures (§measurement gate), and the cohort exclusions (non-business,
non-English, Shorts, too-recent/too-old, deleted/private).

## Practical-significance thresholds

- **Decision rule (per predictor):** (a) the 95% interval excludes 0, AND (b) the point reaches
  **±10% per +1 global SD**. At this N (a) is near-automatic, so (b) is the operative bar.
  "Present but trivial" = real but under ±10% per SD. "Precise null" = the interval includes 0 but
  lies entirely within ±10% per SD, so the design has bounded any effect under the practical bar.
  "Inconclusive" = ±10% per SD falls inside the interval (a power failure), kept rigidly distinct from
  a precise null and never rounded to a satisfying null.
- **±10% is per +1 global SD, a fixed channel-invariant increment.** `SD_global` is one constant from
  the exploration set, so the bar means the same raw amount of color/contrast in every channel. The
  cohort spans a few global SDs, so ±10% per SD implies a larger total swing across the full vividness
  range; the report spells out the unit (a per-SD rate on a standardized dose) so the figure is not
  mistaken for a per-step or per-doubling bar.
- **"Rough size" is pinned (no analyst discretion).** Everywhere two estimates are required to "agree
  in sign and rough size" — the two estimators, the within-era nested-FE veto, the outlier legs, the
  holdout-vs-exploration replication — **"rough size" means both estimates fall in the same
  decision-rule bucket**: both ≥ +10% per SD, both within ±10% per SD, or both ≤ −10% per SD. Same
  sign is implied by same bucket except across the trivial band, where a sign flip inside ±10% is by
  construction not decision-relevant. This is fixed here so "rough size" cannot be read generously or
  strictly after seeing the pair.
- **Decile shape guard:** a headline is reported as "shape, see the curve" if its within-channel
  decile curve is real and non-monotone in a way the single slope hides (interior decile opposite in
  sign to the overall slope, real by its own whole-channel interval).

## Locked holdout

- A fixed ~20% whole-channel holdout: a channel is held out iff a deterministic, reproducible function
  of its id places it in the held-out fifth. This rule regenerates the split exactly; it is
  whole-channel and locked when the cohort is captured, before any exploration.
- All exploration, the measurement gate, the within-channel SD-floor choice, the decile bin edges, and
  the power go/no-go run on the exploration side (the 80%).
- The two headline slopes are read off the holdout **once**, with the analysis code fixed in advance.
- **Within-channel FE + holdout reconciliation:** the FE coefficient is **refit inside the holdout
  channels** (option (a) of the protocol) — each predictor's within-channel slope is re-estimated on
  the held-out channels' own videos, never read off the exploration fit. The whole-channel split means
  a held-out channel's within-channel slope is never contaminated by exploration rows from the same
  channel.
- **Power go/no-go (locked threshold `POWER_HALFWIDTH_MAX = 7%` per SD).** On the exploration set,
  before the holdout read, each predictor's pooled-FE bootstrap interval half-width is checked against
  this threshold. If a predictor's half-width exceeds 7% per SD we declare it underpowered to separate
  a precise null from inconclusive on the single slope, and we lead with the decile curve and a
  bounded-magnitude statement rather than presenting that slope as confirmatory. This is the
  pre-committed expected fallback, fixed here so an inconclusive outcome reads as planned, not as
  improvisation after the read.

## Measurement gate (before any color/contrast-vs-views number)

No detector to validate; the gate is image fidelity and temporality. All run on the exploration side:

1. **Decode fidelity.** On a random 300-thumbnail sample, ≥ **99%** decode to a plausible image
   (non-degenerate size, non-constant pixels). A placeholder gray box reads as a real low-color
   thumbnail, so decode failures are **dropped, not imputed**. Because dropping (not imputing)
   selects on the missing rows, we additionally report the decode-failure rate **by publish-year and
   by views decile**, not just the aggregate pass rate: a flat rate closes the hole; a rate that
   trends with era or views means decode loss aliases the era threat (§4 below) and is disclosed as
   such.
2. **Resize invariance.** Both metrics computed at two short-side targets on the sample; rank
   correlation ≥ **0.98** across resolutions (the slope is rank-driven via within-channel demeaning).
   If a metric is resolution-fragile, fall back to the highest common resolution.
3. **Color-space / swatch validation.** The pipeline is validated on synthetic swatches with known
   answers **before any real image is scored**: a pure-gray luminance ramp must read `K > 0` and
   `C ≈ 0`; a saturated rainbow must read high `C`; a flat mid-gray must read `C ≈ 0, K ≈ 0`. This
   catches a silent BGR/RGB swap or a missing sRGB linearization (which would not crash but would bias
   both metrics).
4. **Temporality / thumbnail stability.** The fetched image may not be the one that earned the views
   (creators swap thumbnails). The capture records a content hash; a re-fetch at analysis time excludes
   any thumbnail that changed between capture and analysis. Where YouTube
   thumbnail-change history is unavailable at scale, the residual is **disclosed as a limitation** and
   a sensitivity is reported on the subset with the strongest stability evidence.

**Escalation rule.** The swatch validation plus a decoded-thumbnail spot-check suffices for a null,
trivial, or inconclusive holdout. If a headline comes back **present & meaningful** (CI excludes 0 and
the point reaches ±10% per SD), gates 1–3 become load-bearing for that positive claim and publication
then requires all of them documented. Committed now, before the holdout read.

## Statistical stance

At our scale statistical significance is automatic, so we de-emphasize p-values and report effect
sizes, channel-clustered intervals, and the practical thresholds above. Every relationship is
correlational; the data is observational and nothing was randomized. Each headline describes only
business+English channels with within-channel variation in that predictor; it does not generalize to
channels whose thumbnails are all one color/contrast, nor outside the business/creator-economy frame.

## Planned robustness checks

- [ ] Measurement gate passed (decode ≥99%, resize rank-corr ≥0.98, swatch validation, thumbnail-stability) before any outcome number.
- [ ] Headline holds under alternative minimum-age cutoffs (30 / 90 / 180 days).
- [ ] Each headline holds winsorized at the 99th percentile and dropping the top 1% of channels.
- [ ] Holdout replicates the exploration-set slope (sign + rough size) for each predictor.
- [ ] Decile curve is monotone and consistent with the slope sign (shape guard did not fire), per predictor.
- [ ] Mutually-adjusted decomposition: both predictors clear ±10% per SD in the adjusted fit, else the under-bar one is reported as "carried by shared vividness," not an independent finding; `r_CK` + separability tripwire reported.
- [ ] Within-channel-within-era slope (publish-year FE nested in channel) agrees in sign + rough size — confirmatory, can veto.
- [ ] SD-tercile slope check: slope flat across within-channel-SD terciles (else effect-modification by the variation filter disclosed).
- [ ] Letterbox-stripped fork: each slope stable with and without bar-stripping.
- [ ] `has_face` and text-density covariates on/off agree (text-density-adjusted contrast slope reported for the contrast headline specifically).
- [ ] Global-SD headline and within-channel-SD companion reported side by side.
- [ ] Median-of-per-channel and pooled-FE estimators (both in global-SD units) agree in sign + rough size; divergence ⇒ report both, lead with conservative.
- [ ] Naive (raw VPD) and age-controlled `ln(views)` reported side by side.
- [ ] Subgroup direction holds in a majority of niche/size tiers; a flip is reported, not suppressed.
- [ ] Deletion/private rate by predictor decile reported where observable, else flagged as unfixable survivorship limitation.

## What has to be true to publish

A slope is publishable as an **independent** positive ("more colorful helps" / "higher contrast
helps") finding only if the confirmatory bundle holds for that predictor: the holdout slope (in
global-SD units) meets the decision rule; the two estimators agree in sign and rough size; the decile
shape guard did not fire; the **mutually-adjusted slope still clears ±10% per SD** (else the predictor
is reported as "carried by shared vividness, not separable from the other," not as its own finding);
the **within-channel-within-era nested-FE slope agrees** in sign and rough size (else the slope is
reported as era-trend leakage); the letterbox fork does not overturn it; it holds under at least two
alternative specifications and within major subgroups; and the measurement gate passed. A slope that
flips, sits under the ±10% per-SD bar, comes back inconclusive, collapses under the era nesting, or is
absorbed by the other predictor is reported as exactly that — including the genuinely interesting "we
expected a positive and got a null," itself a publishable correction to "make it pop." A reported null
and a reported inconclusive are kept rigidly distinct: a private prior is never allowed to round an
underpowered inconclusive up to a satisfying null. The two predictors are adjudicated independently on
the decision rule — one can be present-and-meaningful while the other is a precise null — but the
two-separate-findings *framing* additionally requires both to survive the mutual adjustment.

## Data release plan (YouTube API ToS)

**Hard rule: NO raw YouTube data or resource IDs in cleartext.**

- **What we release:** aggregates (the within-channel decile curves and per-niche slope tables behind
  every chart) **plus** a de-identified row-level dataset.
- **Row-level columns:** `video_hash, channel_hash, colorfulness, contrast, log_age, has_face,
  publish_quarter, niche` — ids HMAC-SHA256 hashed with an unpublished salt. Described as "methods
  reproduction, not measurement verification." De-id rests on the withheld salt + the absence of raw
  YouTube data; both predictors are image-derived (published formulas), carrying no re-identification
  path.
- **Measurement defense** (hashed data can't verify labels): the measurement gate (swatch validation +
  decode + resize-invariance) and the pinned, reproducible formulas.
- [ ] Data-release reviewed against the **current** API ToS before the publish push.

---

## Resolved data-derived constants (recorded from exploration before the holdout read)

> The pre-registration fixes the RULES; two constants are resolved from the exploration set and recorded
> here before any holdout outcome is touched, per the methodology above. These are reported, not chosen:
> the rule that produced each is locked. Recorded 2026-06-18 from the exploration cohort (1,216,787
> videos passing the locked filters across 13,284 channels; estimand after the variation filter: 12,539
> channels, ~1.17M videos per predictor).

| Predictor | `SD_global` (headline-unit denominator) | Variation floor (P5 of per-channel within-channel SD) | Median within-channel SD / `SD_global` |
|---|---|---|---|
| Colorfulness | 28.2169 | 9.1266 | 0.69 (so +1 global SD ≈ 1.4 typical within-channel SDs) |
| Contrast | 0.0840 | 0.0309 | 0.75 (so +1 global SD ≈ 1.3 typical within-channel SDs) |

Pre-specified exploration tripwires (reported here for transparency; they gate interpretation, not the
estimate, and none tripped):

- **Power (go/no-go):** both predictors powered. Pooled-FE interval half-width per +1 global SD is 0.82%
  (colorfulness) and 0.71% (contrast), both well inside the registered ±7%-half-width go bar.
- **Separability (collinearity tripwire):** within-channel correlation between colorfulness and contrast
  is −0.014 (effectively orthogonal). Mutually-adjusting each predictor for the other moves its slope
  negligibly, so the two are reported as independent slopes, not a single blended "vividness" statement.

## Transparent Changes (post-registration deviations)

> A pre-registration is a plan, not a prison. Every change after the tag is logged here with date and
> reason. An empty section means none.

- <none yet>
