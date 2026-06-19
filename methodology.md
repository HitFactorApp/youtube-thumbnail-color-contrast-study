# Methodology: Thumbnail Color & Contrast vs. Views

> The full methods for the study. The plan was registered and publicly timestamped before any
> view data was queried (see `preregistration.md`, tag `v0-preregistration`). This document
> describes what we did; the [design spec](./design-spec.md) explains why each choice was made.

## The question

Within a single channel, does a **more colorful** thumbnail, or one with **higher luminance
contrast**, get more views than that same channel's less colorful or flatter thumbnails? "Make
the thumbnail pop" is standard packaging advice. We tested the two halves of "pop" separately,
as two distinct, deterministic image measurements, with the channel difference removed.

## Dataset

We analyzed **1,602,471 qualifying videos across 16,716 channels**. Of these, **1,532,870 videos
across 16,681 channels** enter the analysis after two locked filters (a 10-view floor and a
90-day minimum age, so views have settled), and that analysis frame is what every result and
every aggregate table use. The confirmatory headline is computed on a held-out fifth of those
channels (316,083 videos across 3,397 channels), the rest used to lock the plan. The channels are English-language business and creator-economy
channels drawn from our own dataset. They are **not a random sample of YouTube** (see
Limitations): they entered through our funnel and skew toward English-speaking,
business/creator-economy, mid-to-large channels. Every channel in the study has at least 20
qualifying videos.

A video qualifies if it is:

- a regular long-form upload (not a Short),
- public (not unlisted, private, hidden, or deleted),
- organic (we exclude videos our traffic classifier flags as primarily ad-driven, because
  ad-driven views don't test whether the thumbnail earned the click),
- published between 90 days and 5 years before measurement (90 days so views have settled;
  5 years so it stays comparable across thumbnail and algorithm eras),
- accompanied by a thumbnail image that loads.

"Long-form (not a Short)" is decided by the platform's own video-type label, not a duration
cutoff. As a result the cohort includes some short-duration (three minutes or less) regular
videos; these are kept, and we report the result separately for short- and longer-duration
videos as a robustness check (see Results).

## Outcome

The outcome is **views**, measured at a single point in time (one capture per video, with a
frozen snapshot), reported alongside each video's age so the comparison holds age constant (see
"Why views, age-controlled" below). We work in log space because view counts are heavily skewed
by viral videos; a single outlier cannot swing a log-based comparison.

## The two measurements

Both predictors are computed directly from the thumbnail pixels by a fixed formula. There is no
model, no training, and no human judgment in the measurement, so unlike a detector there is no
labeling error to validate against; the formulas are deterministic and reproducible from the
image alone. Each thumbnail is decoded, then resized so its short side is 480 pixels (so a
larger stored image and a smaller one give the same number), and the two values are read off the
pixels.

- **Colorfulness** is the Hasler–Süsstrunk metric: it measures how far the image's colors spread
  from gray, combining the spread and the average offset on the red–green and yellow–blue color
  axes. A grayscale image scores near zero; a vivid, saturated, multi-hued image scores high.
- **Contrast** is the root-mean-square luminance contrast: the standard deviation of perceived
  brightness across the image (brightness computed with the standard Rec. 709 luminance weights
  on linearized color). A flat, evenly lit image scores low; an image with bright highlights
  against deep shadows scores high.

Both are reported as the **percent change in views per +1 standard deviation** of the predictor,
where the standard deviation is one fixed, channel-invariant number computed once on the
exploration set. So "+1 SD" is the same size of step for every channel, and the colorfulness and
contrast numbers are on a comparable footing even though their raw units differ.

**Measurement validation (run before any color-to-views number).** The two formulas were checked
against synthetic test images with known answers: a flat gray field scores near-zero on both; a
gray brightness ramp scores near-zero colorfulness but a known nonzero contrast; a saturated
rainbow scores high colorfulness; pure-color fields confirm the luminance weighting orders green
above red above blue. All checks passed before the formulas were run on any thumbnail.

A thumbnail is dropped only if it fails to load or decodes to a degenerate (single-value) image.
On this cohort that loss is about 0.4%, and it is a property of the image fetch applied
identically to every video, so it cannot bias a within-channel comparison.

## The comparison

**The unit is the video; the channel is the cluster.** Videos in the same channel are not
independent (shared creator, audience, niche, cadence), so we never treat videos as independent
observations. Every confidence interval uses the **channel** as the resampling unit.

**The headline is within-channel.** We compare a channel's more-colorful (or higher-contrast)
videos to that same channel's less-colorful (or flatter) ones, which cancels every stable
channel trait (size, niche, audience, production quality) and so can't be dismissed as "big
channels just use punchier thumbnails." A within-channel slope only exists for channels whose
thumbnails actually vary in the predictor; a channel whose thumbnails are all equally colorful
contributes no information about colorfulness and drops out of that predictor's slope (it must
have at least 10 qualifying videos and a within-channel spread above a fixed floor, the 5th
percentile of the spread distribution, set on the exploration set).

**Why views, age-controlled.** Because the cohort is measured on a single clock, a video's age
is really *when we captured its views*, so the headline estimator removes age explicitly: a
channel-fixed-effects regression of **log(views) on the standardized predictor plus log(age)**.
Channel fixed effects absorb every stable between-channel difference; the age term removes the
measurement-timing component. This is the same model as an age-controlled views-per-day
regression (switching the outcome from views-per-day to views only shifts the age coefficient by
exactly one, leaving the predictor's slope identical).

**Two estimators, and we only claim what both agree on.** We report two slopes side by side: a
pooled fixed-effects slope (one regression over all channels at once) and the median of the
per-channel slopes (one slope per channel, then the median — one channel, one vote). A finding
is reported as real only when the two agree in sign and rough size; on disagreement we lead with
the smaller (more conservative) one.

**Are the two predictors separable?** Before the headline, we measured the within-channel
correlation between colorfulness and contrast. It is essentially zero (−0.01), and adjusting
each predictor for the other barely moves its slope, so the two are reported as independent
slopes rather than a single blended "vividness" number.

**Confidence intervals.** A whole-channel bootstrap: resample channels with replacement (never
videos), refit, repeat 10,000 times (fixed seed). Channels are the independent unit, so they are
the resampling unit. With more than a million videos, statistical significance is automatic for
any non-zero effect, so we de-emphasize p-values and report effect sizes, channel-clustered
intervals, and a practical-significance threshold (a ±10% effect per +1 SD) declared in advance.

## The holdout

A fixed ~20% of channels were set aside by a deterministic rule before any exploration, and the
headline numbers were computed on them **once**, with the identical analysis code, after the spec
and the two data-derived constants (the standardizing SD and the variation floor) were frozen on
the other ~80%. Channels we had looked at in any prior internal work were forced out of the
holdout. The exploration result is the confirming companion; the held-out number is the headline.

## Robustness and guards

The pre-registered checks that ran alongside the headline:

- **Decile shape guard.** A mechanical check for a single interior turning point (a "sweet spot"
  or a dip) in the dose-response curve, so a non-monotone shape can't hide inside a near-zero
  straight-line slope. It did not fire for either predictor.
- **Short vs. longer duration.** The headline re-estimated separately on videos three minutes or
  less and over three minutes.
- **Within-era check.** The slope re-estimated with channel-and-publish-year cells (a stricter
  control than log-age alone), to confirm the result is not an artifact of color or contrast
  trending over time.
- **Whole-cohort descriptive estimate.** After the holdout verdict was recorded, a pooled
  estimate over the entire cohort (both halves) is reported as a tighter-precision descriptive
  companion, explicitly not the confirmatory headline.

## Limitations (stated plainly)

- **Observational, not causal.** How colorful or punchy a thumbnail is, is a creator's choice; we
  randomized nothing. Within a channel, colorful and flat thumbnails still differ in bundled ways
  (topic, title, subject matter) we cannot fully separate. Read every result as "associated
  with," never "causes."
- **Sampling frame.** English business/creator-economy channels from our own funnel, skewed
  mid-to-large. The finding may not transfer to other languages, niches, or very small channels.
- **Outcome.** Views measured once; we cannot see watch-time, click-through rate, or impressions,
  and we cannot see videos that were deleted or made private (survivorship).
- **The measurements are image-level summaries.** Colorfulness and contrast are whole-image
  statistics; they do not capture *where* the color or contrast sits, the subject, the text, or
  composition. A thumbnail can be "vivid" in many different ways that these two numbers score the
  same.

## Reproducing the statistics

The `aggregates/` directory holds the binned tables behind every chart (the per-decile
dose-response curve for each predictor, the short/long split, the within-vs-cross-channel
contrast, and the headline slopes for the holdout, exploration, and whole-cohort sets). A
replicator can re-run our within-channel slope, the two estimators, the clustering, and the
bootstrap and catch a coding error. The two predictors are themselves fully reproducible: the
colorfulness and contrast formulas are standard and deterministic, so anyone can recompute them
from any thumbnail and get our numbers. The aggregates prove the *method*; the formulas, applied
to any image, prove the *measurement*.
