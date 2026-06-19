# Thumbnail Color & Contrast vs. Views — Replication Package

A standalone replication package for the Hitfactor Labs study testing whether a **more colorful**
or **higher-contrast** thumbnail gets more views, measured *within a channel* across 1.6M YouTube
videos.

**Read the study:** https://hitfactorapp.com/labs/2026-thumbnail-color-contrast/

## Headline

Within a single channel, comparing each channel's punchier thumbnails against its own flatter ones:

- **Colorfulness → no effect worth acting on.** Held-out estimate +1.2% per +1 standard deviation
  (95% CI −0.4% to +2.9%), confidently inside the ±10% practical bar set in advance.
- **Contrast → real, but trivially small.** Held-out estimate +1.9% per +1 standard deviation
  (95% CI +0.5% to +3.3%), far under the bar.

Both replicated on a held-out fifth of channels never used while building the analysis. "Make the
thumbnail pop" is not a lever, at least not via overall color or contrast.

## What's in here

| File | What it is |
|---|---|
| `design-spec.md` | Why each design choice was made. |
| `preregistration.md` | The analysis plan, committed and tagged **before any view data was queried** (tag `v0-preregistration`). |
| `methodology.md` | What we did, in full. |
| `report.md` | The findings. |
| `aggregates/` | The binned tables behind every chart — the data artifact of the study. |

### `aggregates/`

- `headline-slopes.csv` — the within-channel slope per predictor for the holdout (confirmatory),
  exploration, and whole-cohort (descriptive) sets, with both estimators and bootstrap CIs.
- `colorfulness-decile.csv`, `contrast-decile.csv` — the dose-response curve: age-adjusted mean
  log-view residual and median predictor value per within-channel decile.
- `shortlong-split.csv` — the holdout slope split by video duration (≤180s vs >180s).
- `within-vs-cross.csv` — the within-channel vs across-channel slope, showing the channel-size
  confound for colorfulness.

## Reproducing

The two predictors are standard, deterministic image formulas — anyone can recompute them from any
thumbnail and get our numbers:

- **Colorfulness:** the Hasler–Süsstrunk metric.
- **Contrast:** root-mean-square luminance contrast (Rec. 709 luminance weights on linearized sRGB).

The aggregate tables let a replicator re-run our within-channel slope, the two estimators, the
channel-clustered bootstrap, and the decile curve, and catch a coding error. Raw rows and channel
identifiers are not republished.

## How this study was run

- **Plan first, data second.** The pre-registration was committed and publicly tagged before any
  outcome was queried.
- **A holdout we never touched.** A fixed ~20% of channels was set aside before exploration; the
  headline ran on it once, with the same code, after the plan was locked.
- **We resample channels, not videos.** Videos from one channel aren't independent, so every
  confidence interval treats the channel as the unit.
- **Magnitudes, not significance.** Past a million videos, anything non-zero clears significance, so
  we report effect sizes, intervals, and a ±10%-per-SD practical threshold set in advance.

## Citation

> Hitfactor (2026). *More colorful or higher-contrast thumbnails don't get more views: a
> within-channel study of 1.53M YouTube videos across 16,681 channels.*
> https://hitfactorapp.com/labs/2026-thumbnail-color-contrast/

## License

The aggregate data and text in this repository are released for replication and citation.
