# Design Spec: Thumbnail Color & Contrast vs. Views

> Phase 0 + Phase 1 deliverable. Written before the pre-registration and before any query relating
> color or contrast to views. This is the locked design for how two continuous image predictors
> (colorfulness and luminance contrast) enter the model and how the headline is read off a sealed
> holdout exactly once. The study draws its own cohort (its own sealed store and its own
> whole-channel holdout) and runs a content-only image pass to compute the two predictors. Both
> passes complete, and the measurement gate passes, before any outcome number is computed.

## Research question (Phase 0)

**The question:** Among regular long-form uploads, does a thumbnail that is **more colorful**, or one
with **higher luminance contrast**, relate to more views-per-day, _within the same channel_? The two
predictors are reported as two separate within-channel dose-response slopes, not blended into one
index.

**Why it matters:** "Make the thumbnail pop" is accepted YouTube wisdom, and is asserted as an obviou
thumbnail design tactic to drive clicks and ultimately views.
At the same time, we're aware of no large-scale, channel-controlled evidence behind it. Creators and
agencies cite it to justify edits; nobody has shown whether, _holding the channel fixed_, a more colorful
or higher-contrast thumbnail actually moves views. A defensible within-channel number
(a slope, a sign, and a magnitude under a pre-declared practical bar) settles a benchmark gap and is
directly quotable.

**Why we can answer it:** We have ~8.7M videos / ~176K channels with thumbnails we can fetch, joined
to a frozen views snapshot. Colorfulness and contrast are computed deterministically from the
thumbnail image by fixed, off-the-shelf formulas (no model, no training, no selection on outcome), so
there is no winner's-curse / selection-on-outcome risk in the predictor itself. The scale lets us
compute the slope _within_ each channel and aggregate across thousands of channels, which is what
makes "holding the channel fixed" credible rather than anecdotal.

**Question-selection checklist:**

- [x] On-brand (needs our thumbnail corpus + views snapshot at scale)
- [x] Citable (the answer is a concrete % change in views per +1 SD of colorfulness / contrast)
- [x] Defensible (framed as a within-channel relationship, no unsupported causal claim)
- [x] Counter-intuitive or benchmark-filling (tests a near-universal piece of advice that has no
      channel-controlled evidence)
- [x] Written down before peeking at outcome distributions

## Study design (Phase 1)

### 1. Observational, not experimental

Observational. Every finding is correlational by construction. Colorfulness and contrast are two of
many packaging choices a creator makes, and they travel with others we do not observe (topic, edit
style, subject, branding). The within-channel design strengthens the association; it does not make it
causal. We say "associated with," never "causes."

### 2. The two predictors: colorfulness and luminance contrast

Two continuous image statistics, computed once per thumbnail from the fetched image, **before any
outcome is touched**. Each is reported as its own within-channel dose-response slope; they are never
combined into a single composite (a composite would conflate two distinct things and make it
impossible to defend which one carries the result).

- **Colorfulness — Hasler–Süsstrunk (2003).** The standard perceptual colorfulness metric. On the
  opponent-color channels `rg = R − G` and `yb = 0.5·(R + G) − B`, colorfulness is
  `sqrt(σ_rg² + σ_yb²) + 0.3·sqrt(μ_rg² + μ_yb²)`, where σ and μ are the standard deviation and mean
  over all pixels. It rewards both a wide spread of colors and high saturation. Chosen over mean HSV
  saturation because saturation alone ignores hue variety (a uniformly deep-blue frame is highly
  saturated but not colorful); colorfulness is the metric the "make it colorful" advice is actually
  about. **Why this specific formula matters for replication:** the coefficients (`0.3`) and the
  opponent-channel definitions are fixed by the paper; we pin them in the pre-reg so the predictor is
  bit-for-bit reproducible.

- **Luminance contrast — RMS contrast.** Convert to relative luminance
  `Y = 0.2126·R + 0.7152·G + 0.0722·B` (Rec. 709, on linearized sRGB), then contrast is the standard
  deviation of `Y` across pixels (root-mean-square contrast). Chosen over Michelson contrast
  (`(Ymax − Ymin)/(Ymax + Ymin)`), which is dominated by a single bright or dark pixel and is
  unstable on photographic images; RMS uses the whole distribution. We pin the luminance weights and
  the sRGB linearization (the `2.4`-gamma piecewise transfer function) in the pre-reg.

- **Both are continuous doses, reported per +1 global SD.** Colorfulness and contrast are dose
  variables with a natural direction: the advice is literally "more." So the headline is a slope,
  expressed as the **% change in views per +1 global SD** of each predictor (per SD rather than per
  doubling because the units are additive, not multiplicative). The SD is a **single global
  constant computed once on the exploration set**, identical across channels and not re-estimated
  inside the bootstrap. We do **not** standardize by each channel's own within-channel SD for the
  headline: a per-channel SD estimated from ~9 residual d.o.f. is noisy (CV ≈ 24% at 10 videos), and
  dividing the regressor by that noisy scale is multiplicative measurement error that attenuates the
  slope worst on the small-n channels the median estimator weights most, while also making "+1 SD"
  mean a different raw amount of color in every channel (so the two estimators would blend
  incommensurable units). The within-channel-SD-standardized slope is reported as a descriptive
  companion only. Both transforms are locked in §8.

- **Pixel scope and resize, pinned now.** Both metrics are computed over the full thumbnail, on the
  image resized to a **480px short side with area-average (INTER_AREA) downscale** (locked in the
  pre-reg; 480 is the most universally available non-tiny YouTube thumbnail size, area-average is the
  correct interpolation for color statistics) so resolution differences across thumbnails do not
  change the statistic. Alpha/letterbox bars, if present, are not stripped in the primary metric
  (stripping is a researcher choice that could be gamed); a letterbox-stripped sensitivity fork is
  pre-registered in §8.

### 3. Unit and clustering

The row is the video; the **channel is the resampling unit**. Videos in a channel are not
independent, so every interval resamples whole channels (10,000-resample bootstrap, fixed seed),
never videos. The headline is within-channel (channel fixed effects), so it cannot be dismissed as
"the channels that use colorful thumbnails are just bigger / in flashier niches."

A within-channel dose slope only exists for a channel whose videos _vary_ in the predictor. We
require a channel to have at least **`MIN_VIDEOS_PER_CHANNEL_FE = 10`** qualifying videos with
**non-degenerate within-channel variation** in the predictor — its within-channel SD at or above the
**5th percentile** of the per-channel within-channel-SD distribution (`MIN_PREDICTOR_SD_PER_CHANNEL`,
set on exploration per predictor) — so a channel whose thumbnails are all near-identical in color does
not inject a high-leverage noise slope into the bootstrap. The two predictors qualify channels
independently (a channel can vary in contrast but not colorfulness), so the colorfulness slope and
the contrast slope rest on different — overlapping — channel sets; we lock and report both counts.
The estimand therefore skews toward channels that _vary_ their thumbnail color/contrast; that is a
generalization caveat, stated in the headline, not a bias in the slope.

**Two co-equal estimators, reported side by side** (the rule carried over from the earlier thumbnail
studies). A result is claimed
only if both agree in sign and rough size:

- **Median of per-channel slopes (one channel, one vote).** Fit the dose slope within each qualifying
  channel, then report the median across channels. Immune by construction to a few high-volume
  channels: a channel with 500 videos and one with 12 each contribute one slope to the median. This
  is the standard-preferred estimator for an early study because the resampling unit (channel) is
  also the unit of the point estimate.
- **Pooled within-channel fixed-effects slope.** The pooled coefficient from demeaning `log(views)`
  and the predictor within channels, then regressing. Video-count weighted, so high-volume channels
  carry more leverage; reported as the companion to the median, not as a standalone headline, because
  the per-channel video-count distribution is heavy-tailed.

**A result is claimed only when the median and the pooled slope agree in sign and rough size; on
divergence we report both and lead with the smaller-magnitude (more conservative) one**, a rule fixed
here, not after seeing the pair. We report the distribution of per-channel video counts and flag any
single channel whose leverage dominates the pooled slope.

**Bootstrap procedure (fixed here).** Each of the 10,000 resamples draws whole
channels with replacement and assigns each draw a synthetic cluster id (a channel drawn twice is two
clusters, not one collapsed group). The channel fixed-effects **demeaning** is **re-done inside every
draw** (each resampled channel is demeaned by its own resampled mean before the slope is fit);
demeaning once on the full sample and bootstrapping the already-demeaned rows would treat the
within-channel centering as known and understate the interval. The `SD_global` scale is a fixed
constant from the exploration set, so it is **not** re-estimated inside the draw (there is nothing
channel-specific to re-estimate). The same whole-channel resample feeds both estimators, and because
both are in the one global-SD unit their agreement gate compares like with like.

### 4. The selection structure (what conditioning we do and don't do)

This study does **not** condition the sample on a creator choice. **Every thumbnail has a color and a
contrast value**, so there is no "colorful subset" to restrict to; the cohort is the full qualifying
population, and the two selection caveats below are stated, not waved away.

- **No predictor-defined subset.** Colorfulness and contrast are defined for every image, so we do
  not condition on the predictor. There is no collider induced by keeping only thumbnails that have
  the feature (the failure mode when a study restricts to, say, thumbnails that use a face), because
  no such restriction is made.
- **The within-channel-varying restriction is on the predictor's variance, not its level (a
  variation filter, made falsifiable — not asserted benign).** Requiring within-channel variation (§3)
  drops channels whose thumbnails are near-uniform in color/contrast. A channel with no variation
  cannot identify a within-channel slope, so this is a variation filter, not a level cut, and it does
  not condition on high vs. low color. But it changes _who the estimand is about_ (channels that vary
  their thumbnails), and that selection is benign for the _slope_ only if the slope does not itself
  vary with how much a channel varies — a channel that deliberately varies color may A/B-test
  packaging and carry a steeper or flatter within-channel slope, which would make the estimate a
  selected value rather than just a less-generalizable one. We therefore do not assert benignity; we
  **pre-register the SD-tercile falsification test** (split qualifying channels into terciles of their
  own within-channel predictor SD, report the slope in each): flat across terciles confirms benign; a
  trend is effect-modification by the selection variable, disclosed as a bias direction. We also report
  the fraction of channels dropped for low within-channel variance, separately per predictor.
- **The real residual is co-varying packaging (the core limitation, §6).** A more colorful thumbnail
  is rarely _only_ more colorful — it often comes with a different topic, a louder edit, a different
  subject, brighter branding. Channel fixed effects absorb whatever is stable about a channel's style,
  but not the within-channel correlation between "this video got a colorful thumbnail" and "this video
  was also a bigger-swing topic." We cannot separate these, so we cannot claim the color did it rather
  than something that traveled with it. This is named as the headline's stated limitation, not bounded
  away.

### 5. Outcome: views, age-controlled (single measurement clock)

The cohort sits on a single measurement clock (the capture records
one frozen views snapshot per video within a short window), so a video's age is effectively _when we
measured it_, and raw views-per-day would smuggle measurement timing into the outcome. The headline
outcome is therefore **`log(views)` controlling for `log(age)`, inside channel fixed effects**. This
is algebraically the same model as an age-controlled views-per-day regression (moving from
views-per-day to views shifts only the age coefficient by 1, leaving the predictor slopes identical),
with the timing artifact moved out. We report the raw views-per-day version alongside as a
cross-check.

Because the capture date is nearly fixed, `log(age)` doubles as a publish-era axis: controlling for
it removes the timing artifact and absorbs a chunk of era confounding for free. The flip side is that
the publish-year sensitivity check is not fully independent of the headline control; we report it as
corroboration, not orthogonal evidence.

**An era confound specific to _this_ study (and why the era check is confirmatory, not descriptive).**
Thumbnail aesthetics have a strong temporal trend: average colorfulness and contrast on YouTube have
drifted over the years (the "MrBeast-ification" of packaging). The threat that channel fixed effects
do **not** reach is a _within-channel time trend_: a channel that both grew and got more colorful over
the years has colorfulness and views trending up together _inside_ the channel. Channel demeaning
removes the channel _level_, not a within-channel trend, and `log(age)` is a single linear age slope
that cannot flexibly absorb a channel-specific growth trajectory correlated with its color
trajectory. So `log(age)` handles the cross-sectional era confound but not this one. We therefore
**promote the within-channel-within-era slope (publish-year fixed effects nested in channel) to the
confirmatory bundle with veto power** (§8): it is _more_ aggressive than `log(age)` (it absorbs
arbitrary year-to-year shifts), so survival is the strong evidence the slope is not era-trend leakage,
and collapse is the finding that it was. This is the sharpest within-channel confounder for an
aesthetic predictor, so it is decision-relevant either way — not corroboration.

Limitations: views, not watch-time; a single capture; deleted and private videos are unobservable
(survivorship). The within-channel survivorship threat: if a creator deletes its weak low-color
videos more readily than its weak high-color videos (or vice versa), the surviving rows are
differentially upward-selected and the slope inherits a survivorship gradient that channel fixed
effects cannot remove, because it lives in the within-channel residual. We state this as an unfixable
limitation at the within-channel level. We work in medians and log space because views are
outlier-dominated.

### 6. Confounders — DAG

Headline relationship: `thumbnail colorfulness / contrast → views`, within channel.

| Variable                                                                           | Role                         | Control?                                                                                                                                                                                                                    |
| ---------------------------------------------------------------------------------- | ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Channel size / niche / audience / house style                                      | confounder                   | Absorbed by channel fixed effects.                                                                                                                                                                                          |
| Video age / measurement clock                                                      | artifact                     | Removed by the explicit `log(age)` term (§5).                                                                                                                                                                               |
| Publish era — cross-sectional                                                      | confounder                   | Largely absorbed by `log(age)` under the single clock (age relabels publish date).                                                                                                                                          |
| Publish era — **within-channel time trend**                                        | confounder FE cannot reach   | **Not** absorbed by channel FE (a level removal) or `log(age)` (a linear age slope). Caught by the within-channel-within-era nested-FE slope, **promoted to confirmatory with veto power** (§5, §8).                        |
| **Co-varying packaging** (topic, edit intensity, subject, branding, on-image text) | **core residual confounder** | Colorfulness/contrast travel with other choices we do not observe. Not removable by design; the headline's stated limitation (§4).                                                                                          |
| Face presence                                                                       | possible covariate           | A colorful thumbnail may also be a face thumbnail. Reported with and without a `has_face` covariate from an off-the-shelf detector; a check, not an expected mover. |
| On-image text / graphic density                                                    | possible covariate           | High contrast often co-occurs with bold text overlays. We pre-register a text-density covariate (an off-the-shelf detector's text-area fraction) reported with and without; a check, not a fix.                             |
| Video length / format                                                              | weak proxy                   | Sensitivity covariate, with and without.                                                                                                                                                                                    |
| Click-through rate                                                                 | mediator                     | No (on the causal path from packaging to views).                                                                                                                                                                            |
| Recommendation-shelf placement                                                     | collider                     | Never (controlling it manufactures correlation).                                                                                                                                                                            |
| Thumbnail-swap history (the measured image ≠ the one that earned the views)        | temporality threat           | Detected and handled by the freshness/stability rule (§7); a swapped thumbnail breaks the link between the measured color and the earned views.                                                                             |

Big Three (always): `log(subscribers)` / channel — absorbed by FE; niche — absorbed by FE; temporal
era — cross-sectional part absorbed by `log(age)`, within-channel-trend part caught by the
confirmatory within-era nested-FE slope. **The headline leads with the within-channel (fixed-effects)
version.** Controlling a mediator (CTR) or collider (shelf placement) would inject bias — not done.

### 7. Measurement gate (before any outcome number)

The predictor is computed by a fixed formula, not a trained model, so there is no detector to
validate. The gate is instead about **image fetch fidelity and temporality**, the two ways a
deterministic metric can still be wrong:

- **(a) Fetch fidelity.** The metric is only as good as the bytes we fetched. We confirm on a
  hand-checked sample that the fetched image is the real thumbnail (not a placeholder, not a 404
  gray box, not a resized-to-1px sentinel), and that decode succeeded. **Pre-committed: ≥ 99% of a
  random 300-thumbnail sample decode to a plausible image (non-degenerate size, non-constant pixels);
  failures are dropped, not imputed.** A placeholder gray box has near-zero colorfulness and contrast
  and would masquerade as a real low-color thumbnail, so this is load-bearing.
- **(b) Resize invariance.** Confirm the two metrics are stable across the resolutions thumbnails
  ship in (YouTube serves several sizes). Compute both metrics at two short-side targets on the same
  sample; **pre-committed: rank correlation ≥ 0.98 across resolutions** (the slope is rank-driven via
  the within-channel demeaning, so rank stability is what matters). If a metric is resolution-fragile,
  fall back to the highest common resolution before any outcome read.
- **(c) Temporality — thumbnail freshness/stability.** The stored/fetched thumbnail may not be the one
  that earned the views (creators swap thumbnails), which breaks temporality for an image-derived
  predictor. Where thumbnail-change history exists, we prefer videos whose thumbnail has been stable;
  where it does not, we **disclose it as a limitation** and (pre-registered) report a sensitivity on
  the subset with the strongest stability evidence. The cohort capture records a content hash so
  a re-fetch at analysis time can detect a thumbnail that changed _between capture and analysis_ and
  exclude it.
- **(d) Color-space correctness.** A silent BGR/RGB channel swap or a missing sRGB linearization would
  not crash but would bias colorfulness and contrast systematically. We pin the channel order, the
  sRGB→linear transfer function, and the luminance weights in the pre-reg, and validate the pipeline
  on a set of synthetic swatches with known colorfulness/contrast (a pure-gray ramp must read contrast
  > 0 and colorfulness ≈ 0; a saturated rainbow must read high colorfulness) before any real image is
  > scored. This is the deterministic analogue of the detector gate.

On any failure: narrow to where the metric is trustworthy (highest common resolution, stable-thumbnail
subset, decode-clean images) or report inconclusive. No color/contrast-vs-views number is computed
before (a)–(d) pass.

**Escalation rule.** A spot-check of decoded thumbnails and the synthetic-swatch validation suffices
for a null, trivial, or inconclusive holdout. If the holdout comes back present and meaningful, the
quantitative gates (decode rate, resize rank-correlation, swatch validation) become load-bearing for
a positive "color/contrast matters" claim, and publication then requires all of them documented. This
is committed now, before the holdout read.

### 8. Thresholds and decision rules (fixed in advance)

At this scale everything is statistically significant, so the threshold is the study.

- **Headline = two within-channel, age-controlled dose slopes** (colorfulness; luminance contrast),
  each expressed as the **% change in views per +1 global SD** of the predictor (`100·(exp(slope) − 1)`
  on the predictor standardized by `SD_global`, §2), with a whole-channel bootstrap 95% interval. Each
  is reported as the two co-equal estimators of §3 (median-of-per-channel and pooled FE), now both in
  the same global-SD unit; a slope is claimed only when they agree in sign and rough size, and on
  divergence we lead with the more conservative one. The two predictors are reported as **two separate
  headlines**, never summed — subject to the collinearity veto below.
- **Decision rule (per predictor):** (a) the 95% interval excludes 0, AND (b) the point reaches
  **±10% per +1 global SD** to count as meaningful. At this N, (a) is near-automatic, so (b) is the
  operative bar. "Present but trivial" = real but under ±10%. "Precise null" = the interval includes
  0 but lies entirely within ±10%, so the design has bounded any effect under the practical bar.
  "Inconclusive" = ±10% sits inside the interval (a power failure), kept rigidly distinct from a
  precise null.
- **Decile shape guard (mechanical, single pinned contrast).** A single slope can hide a non-monotone
  dose-response (e.g. a colorfulness sweet spot — more color helps up to a point, then garish hurts).
  To keep this falsifiable and free of a multiple-comparison scan, the trigger is one pre-named
  contrast — a **turning point** (not a deviation from the curve midpoint, which would mislabel a
  clean monotone slope): form deciles on the **pooled within-channel-demeaned** predictor (bin edges
  fixed on exploration), compute the age-controlled within-channel-residual mean `log(views)` per
  decile with whole-channel bootstrap intervals, take each interior decile (2–9) minus the mean of its
  **two immediate neighbours**, pick the single decile whose local turn is largest opposite to the
  overall slope, and **fire iff that turn is opposite in sign to the overall slope AND its
  whole-channel bootstrap interval excludes zero.** On a trigger the result is reported as "shape, see
  the curve," not led by the single slope. Confirmatory: it can change how the result is led and
  appears in "what has to be true to publish."
- **Collinearity veto over the "two findings" framing.** Colorfulness and contrast are correlated (a
  vivid image is often contrasty), so two large _marginal_ slopes can reflect one shared "vividness"
  factor. We report each slope (i) marginally and (ii) with the other predictor as a within-channel
  covariate, and we report their within-channel correlation `r_CK` as a **gating** quantity. We lead
  with the marginal slopes, but **the two-separate-headlines framing is licensed only when both
  predictors clear ±10% per SD in the _mutually-adjusted_ fit.** If a predictor's adjusted slope drops
  under the bar or flips while its marginal cleared it, that predictor is reported as "carried by
  shared vividness, not separable from the other," not as an independent finding. Separability
  tripwire (checked on exploration before the holdout): if `r_CK` inflates the adjusted intervals past
  the power bar, we declare up front the design cannot separate the two and the honest headline is a
  single "vividness" statement. This has veto power exactly like the decile guard.

The headline is the **two marginal within-channel slopes**, fixed in advance. The analyses that can
change or veto how a slope is reported, all computed on the holdout: the slope itself; the decile
shape guard; the collinearity / mutually-adjusted decomposition veto; the within-channel-within-era
nested-FE slope (era leakage); and the letterbox-stripped fork (§2).

Everything else is descriptive, FDR-corrected at `q = 0.10` within its family, logged in the run
registry, and cannot veto the headline: the `has_face` covariate on and off, the text-density
covariate on and off, the length covariate on and off, short and long split, the SD-tercile slope
check (§4), per-niche and per-channel-size spread, the within-channel-SD-standardized companion,
winsorize and drop-top-1%.

### 9. Locked holdout (by channel)

The study locks a whole-channel holdout: a channel is held out if a deterministic, reproducible
function of its id places it in the held-out fifth (~20%). The split is whole-channel (a channel is
entirely in or entirely out), so a within-channel slope on the holdout is never contaminated by
exploration rows from the same channel.

All exploration, the measurement gate, the power go/no-go, and threshold confirmation run on the
exploration side; the two headline slopes are computed on the holdout **once**, with the analysis code
fixed in advance.

**Power go/no-go before the holdout read (locked threshold).** On the exploration set we first
compute each slope's bootstrap interval half-width. **If a predictor's half-width exceeds
`POWER_HALFWIDTH_MAX = 7%` per SD, we declare the study underpowered to separate a precise null from
inconclusive on that predictor, and we lead with the decile curve and a bounded-magnitude statement
rather than presenting the single slope as confirmatory.** This is the pre-committed expected
fallback, fixed here so an inconclusive outcome reads as "we said this might happen," not improvisation
after the read.

### 10. Generalization

The frame is English-language business and creator-economy YouTube channels from our own funnel,
skewed mid-to-large: not a random sample of YouTube. The two slopes are reported as a spread across
niches (median and range), not one blended number. The colorfulness and contrast distributions are
reported for the full cohort and for the within-channel-varying subset that drives each slope, so the
disclosed representativeness matches the estimand.

### 11. Data release (de-identified, ToS-compliant)

Per the lab protocol: a de-identified row dataset (one-way keyed hashes of channel and video id,
secret salt never published; no raw ids, titles, or thumbnails), the binned aggregate tables (the
within-channel decile curves and per-niche slopes behind every chart), and a replication notebook.
The released rows carry the two predictor values (colorfulness, contrast) and `log(age)`; both
predictors are derived from the image by published formulas, not from any YouTube id, so they carry no
re-identification path. De-identification exists to satisfy the YouTube API terms (no republishing
data keyed to real ids).

## Data access

The cohort is materialized once into a sealed, read-only working store with an immutable manifest
(one row per cohort video, carrying a frozen view-count snapshot and its capture date so the outcome
cannot drift), a deterministic whole-channel holdout flag, and a committed fingerprint that seals the
sample. A separate content-only pass records the two image predictors, a decode-status flag, a
content hash (the thumbnail-stability guard), a face-detection flag, and a text-density measure. The
view snapshot is read once; no outcome is related to any predictor before the pre-registration is
publicly tagged. Only the fingerprint, this design, the pre-registration, the report, and the
aggregate tables are tracked or released; the working store and raw rows are not.

Cohort definition (locked): regular long-form uploads (not Shorts), public, organic traffic,
published 90 days to 5 years ago, not hidden or deleted, on English-language business /
creator-economy channels, each channel with at least 20 qualifying videos. The frozen view count is
the latest available snapshot per video.

> Implementation details (exact source columns, the sealed-store engine, the freeze and image-pass
> mechanics, the holdout hash, and credential handling) live in the gitignored
> `.internal/implementation.md`, which never crosses to the public repo.

## Sign-off

- [x] Within-channel association; no causal claim; co-varying packaging named as the unremovable residual
- [x] Two continuous predictors locked: Hasler–Süsstrunk colorfulness + RMS luminance contrast, formulas pinned
- [x] Headline unit = **global SD** (channel-invariant constant, not re-estimated in bootstrap); within-channel-SD slope demoted to companion (B1)
- [x] Unit (video) + clustering (channel as resampling unit, whole-channel bootstrap, re-demean inside draw, SD fixed)
- [x] Two co-equal estimators (median-of-per-channel + pooled FE), same global-SD unit, claim only on agreement; heavy-tail leverage addressed
- [x] Estimand = within-channel dose slope; "who it's about" = channels varying their thumbnail color/contrast, ≥10 qualifying videos; variation filter made falsifiable via SD-tercile check (S2)
- [x] Outcome = age-controlled `log(views)`, single-clock artifact removed; within-channel-time-trend era threat caught by **confirmatory** within-era nested-FE slope with veto power (S1)
- [x] Selection structure: no predictor-defined subset (no faced-collider problem); variation filter falsifiable, not asserted benign
- [x] Confounders classified; co-varying packaging is the residual; `has_face` + text-density as checks; within-channel survivorship named
- [x] Measurement gate locked (decode fidelity + by-era/views, resize invariance, temporality/thumbnail-stability, color-space/swatch validation) before any outcome number
- [x] Threshold = ±10% per +1 global SD; real/trivial/precise-null/inconclusive rule; single-contract decile shape guard (S3); collinearity decomposition **veto** over two-findings framing (B2); power go/no-go; analyses that can veto named
- [x] Fresh whole-channel holdout, prior-seen + prior-study channels forced to exploration, read once; ToS-checked de-identified release
- [ ] → ready to write the pre-registration, tagged before touching color/contrast-vs-views
