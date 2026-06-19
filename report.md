# Does a More Colorful or Higher-Contrast Thumbnail Get More Views?

"Make the thumbnail pop" is standard advice: crank the color, push the contrast, grab the eye in
the feed. We tested both halves of that advice on **1,602,471 qualifying videos from 16,716
English-language business and creator channels** (1,532,870 videos across 16,681 channels enter
the analysis after the settling filters), comparing each channel's punchier thumbnails against its
own flatter ones. The answer is the same for both, and it is not what the advice promises: within
a channel, **making a thumbnail more colorful does essentially nothing to views, and making it
higher-contrast helps only a trivial amount.** Both effects are small enough to sit under the
threshold we set in advance as the bar for mattering. Neither is big enough to act on.

## What we did

Every thumbnail was scored by two fixed formulas: **colorfulness** (how far the colors spread
from gray) and **luminance contrast** (the spread of bright-to-dark). Both are deterministic
pixel measurements, not models, so they are exactly reproducible from any image. Each channel's
more-colorful and higher-contrast videos were then compared against that same channel's flatter
ones at the same age. Comparing a channel with itself is the central design choice: it removes
channel size, niche, and audience, so the result cannot be explained by "bigger channels use
punchier thumbnails." The two predictors are reported as **two separate slopes** (they turned out
to be uncorrelated within a channel, so they don't have to be blended). The full method is in
[`methodology.md`](./methodology.md); the analysis plan was registered and timestamped before any
view count was queried.

Effects are reported as the **percent change in views per +1 standard deviation** of the
predictor, where one SD is a large, deliberately chosen step: about 1.3 to 1.4 times the amount a
typical channel already varies its own thumbnails, so it is a swing any channel could comfortably
execute within its own catalog. We set **±10% per SD** in advance as the bar for a practically
meaningful effect (our judgment of the smallest within-channel view change a team would reorganize
work around), fixed before we saw any result so the verdicts can't be a moving goalpost.

## Key findings

1. **More color → no meaningful effect on views (a precise null).** Held-out estimate
   **+1.2% per +1 SD of colorfulness** (95% CI **−0.4% to +2.9%**). The interval includes zero and
   lies **entirely inside the ±10% bar**, so this is not "we couldn't tell" — the design resolved
   the question: any within-channel effect of colorfulness on views is bounded well under the
   practical threshold, by a wide margin (the bar is several times the top of the interval, and we
   were powered to resolve effects a fraction of it). The two estimators agree (pooled +1.2%,
   per-channel median +1.2%).

2. **More contrast → a real but trivially small effect.** Held-out estimate **+1.9% per +1 SD of
   contrast** (95% CI **+0.5% to +3.3%**). The interval excludes zero, so this is a real, positive
   association — higher-contrast thumbnails do get slightly more views within a channel — but it
   is **far under the ±10% bar**. A full standard-deviation jump in contrast is tied to under two
   percent more views. Real, but not big enough to act on.

3. **Both results replicated out of sample.** The held-out numbers (computed once, on a fifth of
   the channels never used while building the analysis) match the exploration set they were built
   on: colorfulness +2.0% on exploration vs +1.2% held out; contrast +1.7% vs +1.9%. Same sign,
   same trivial size on both halves.

4. **Color and contrast are independent.** Within a channel they are essentially uncorrelated
   (−0.01), and adjusting each for the other moves its slope by a fraction of a point. So these
   are two genuinely separate findings, not one "vividness" effect double-counted.

5. **No targetable sweet spot.** Sorting each channel's thumbnails into ten levels and plotting
   age-adjusted views shows a gentle few-percent climb that flattens through the middle and ticks
   down only at the most extreme level (total swing about seven points top to bottom, for both
   predictors). The pre-registered check for a real interior turning point came back clean for
   both, so this is a weak, broadly flat relationship, not a peak you can aim for.

6. **It is not a time-trend artifact.** Restricting the comparison to videos from the same channel
   *and the same publish year* leaves both slopes essentially unchanged (colorfulness +1.2%,
   contrast +1.6%), so the result is not an artifact of color or contrast drifting across eras.

7. **The whole-cohort estimate says the same thing, more precisely.** Pooling both halves of the
   study (a descriptive companion, not the confirmatory headline) gives colorfulness **+0.8% per
   SD** (95% CI +0.2% to +1.3%) and contrast **+1.7%** (+1.1% to +2.4%) — tighter intervals around
   the same trivial effects, both far under the bar.

8. **The cross-channel comparison is why this design is necessary, for color.** Pooled across
   *different* channels, more colorful thumbnails look like a **+15%** win; compare each channel
   against itself and that collapses to **+1.9%**. The cross-channel number reflects that bigger,
   higher-view channels use more colorful thumbnails, so color inherits credit for the channel's
   reach. Contrast shows no such gap (cross-channel and within-channel are both near zero), so for
   contrast there is no channel-size confound to undo — the effect is simply small.

## What this means

If you are choosing where to spend effort on a thumbnail, **raw color saturation and overall
contrast are not where the wins are.** Within a channel, pushing either one is associated with
only low-single-digit view changes for a large change, and for color the honest answer is no
effect worth acting on. This
does not say color and contrast are irrelevant to making a *good* thumbnail; it says that **the
overall colorfulness and contrast of the image, by themselves, do not predict views** once you
compare a channel to itself. Whatever makes one of a channel's thumbnails outperform another's, it
is not how vivid or punchy the image is overall.

A caution on reading these as advice: this is a **correlation, not an experiment.** We did not
randomize thumbnails. The result is the cleanest observational estimate we can make — channel held
constant, age held constant, on a million-plus videos and replicated out of sample — but it
describes what *is* associated with views, not what *would happen* if you turned up the saturation
on a given video.

## Robustness

The headline held under the pre-registered checks. Splitting by length, the small contrast effect
lives in **longer videos (+1.8%, 95% CI +0.3% to +3.3%)** and disappears on short ones (−0.6%,
interval spanning zero); colorfulness is a null on both partitions. The within-publish-year
re-estimate (finding 6) and the no-sweet-spot shape guard (finding 5) both held, and the two
estimators (pooled and per-channel median) agree throughout.

## How to read the size of these numbers

"+1.9% per +1 SD" means: take a thumbnail at a channel's typical contrast, raise its contrast by
one full standard deviation (a large, visible jump — about 1.3 to 1.4 times what a typical channel
already varies its own thumbnails),
and the model associates that with under 2% more views. For colorfulness the same large jump is
associated with no change worth acting on. For comparison, our earlier studies found within-channel
packaging effects in the same low-single-digit range; effects this small are real findings about
what *doesn't* drive views as much as the advice claims, not buttons to push for a view boost.

---

*Method, limitations, and the pre-registered plan: [`methodology.md`](./methodology.md),
[`design-spec.md`](./design-spec.md), [`preregistration.md`](./preregistration.md). The binned
tables behind every figure are in [`aggregates/`](./aggregates/).*
