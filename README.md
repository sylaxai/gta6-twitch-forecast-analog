# gta6-twitch-forecast-analog

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/sylaxai/gta6-twitch-forecast-analog/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/sylaxai/gta6-twitch-forecast-analog/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

An analog / shape-based forecaster for Grand Theft Auto VI's launch-day Twitch
concurrent viewers, built for make.fun's `gta6-twitch-ccu-forecast-01` bounty.

## What it does

`main.ts` (Node 22, standard library only) reads one case (`/input/<id>.in`) and
writes `/output/<id>.out`: for each of hours 2 to 24 of the release day (UTC),
five quantiles (5%, 25%, 50%, 75%, 95%) of that hour's Twitch concurrent-viewer
count in the "Grand Theft Auto VI" category.

## How it works

Every past big launch in the case file contributes its release-day *shape*: the
hour-by-hour ratio of its viewers to its own hour-1 print. The forecast anchors
that shape to this launch's observed hour 1:

- **Pool:** comparables with a real hour 1 (hour-1 viewers at least 2% of the
  launch's release-day peak), so pre-unlock hours do not poison the shape.
- **Magnitude weighting:** each comparable is weighted by
  `exp(-1.5 * |ln(v1_target) - ln(v1_comparable)|)` — big launches forecast big
  launches. GTA VI is the biggest launch ever, so the pool concentrates on the
  biggest past launches (ELDEN RING, Cyberpunk 2077, Diablo IV, ...).
- **Median shape:** per hour, the magnitude-weighted mean of the comparables'
  log-ratios, with a clamped linear trend on `ln(v1)` so the median extrapolates
  gently toward the biggest launches, then smoothed over neighbouring hours.
- **Spread:** the magnitude-weighted standard deviation of the log-ratios,
  scaled by 0.85 and combined with a 0.05 floor term, so the 5%/95% band is
  honest about how much launch days actually vary.

The program is fully deterministic (same case, same bytes), offline, and runs
in well under the 8-second cap.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

## Validation

Leave-one-out over the 37 launch comparables in the kit (each launch forecast
from the other 36): mean pinball loss on the log scale **0.220**, against
**0.297** for the house persistence baseline on the same targets. On the six
biggest launches (hour-1 viewers ≥ 200k, the closest analogs to GTA VI):
**0.121–0.132** against **0.226** for the baseline.
