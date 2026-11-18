# GTA VI launch-day Twitch forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/khan7271/gta6-twitch-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/khan7271/gta6-twitch-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)


Entry for make.fun competition `gta6-twitch-ccu-forecast-01`: forecast the
Twitch concurrent viewers in the GTA VI category for hours 2–24 of release
day (UTC), as five quantiles (5/25/50/75/95%) per hour. Score is pinball loss
on the log scale.

## Method

Size-kernel quantile fan over past big launches. `main.py` (Python 3.13,
stdlib only, offline, deterministic) reads one case, anchors on its hour-1
print `p1`, and takes the shape from the case's own `comparables` — past
launches with hourly SullyGnome counts:

- Each donor gives a log-ratio curve `ln(viewers_h / viewers_1)`.
- Donors are weighted by a Gaussian kernel on the log size ratio between
  their hour-1 print and this case's `p1` (bandwidth 0.7). The bandwidth
  widens automatically (up to 2.0) when the effective sample size falls
  below 6, so a print far outside the donor pool (GTA VI's will be) still
  listens to enough launches.
- The median curve is the kernel-weighted median of donor curves, blended
  60/40 with the plain median of all donors.
- The fan is each hour's empirical residual quantiles around that median
  (same kernel weights), shrunk by 0.9, with a minimum width floor
  (±0.25 log on the 5/95% tails, ±0.08 on the 25/75%) so the fan cannot
  collapse when a few donors dominate the weights.
- Falls back to the house persistence shape (0.15 log spread, widening
  with `sqrt(h-1)`) when a case has fewer than 3 usable donors.
- The GTA VI pre-release curve is excluded as a donor: its `day_start` is
  the August premiere, not a release day.

Things tried and rejected on leave-one-out over the 37 donor launches:
pre-release "hype" and momentum tilts, viewers-per-channel weighting, and
recency weighting — each looked good on the four practice cases and failed
the leave-one-out test, so none of them are in the entry.

## Local scores

Practice answers reconstructed from SullyGnome's public API ("Average
viewers" dataset, as the kit's `input/NOTICE.txt` describes); each case's
hour-1 value matches the case file exactly, which verifies the answers.

| model | pinball loss | median error | 90% coverage |
|---|---|---|---|
| this entry | **0.2265** | 0.329 log (~39%) | 0.88 |
| house baseline | 0.3871 | 0.478 log (~61%) | 0.78 |

Per practice case: Wolverine 0.345, RDR2 0.227, God of War Ragnarök 0.138,
Spider-Man 2 0.197. Leave-one-out over the 37 donor launches: 0.425 vs the
baseline's 0.930.

## Run

```sh
MAKEFUN_INPUT_DIR=/path/to/cases MAKEFUN_OUTPUT_DIR=/path/to/out python3 main.py
```

Reads every `*.in` in the input directory (sorted), writes one `*.out`
per case: `{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, ...]}`.
Four practice cases run in ~0.15s; well inside the 8s/case limit.
