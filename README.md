# IamYerhyer/gta6-twitch-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/IamYerhyer/gta6-twitch-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/IamYerhyer/gta6-twitch-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

An entry for make.fun's **Forecast GTA VI's launch-day Twitch viewers hour
by hour** (gta6-twitch-ccu-forecast-01): a program that, given everything
recorded up to the end of release day's first hour, forecasts five quantiles
(5%, 25%, 50%, 75%, 95%) of the Twitch "Grand Theft Auto VI" category's
concurrent viewers for each of hours 2–24 of 19 November 2026 (UTC).

## What the model does

`main.py` (Python 3, standard library only, no network) reads every case
file in the input directory and rebuilds the forecast from the case itself:

1. **Pool the comparables.** Each past launch in the case's `comparables`
   has its release day rebuilt as log-ratios to its own hour 1,
   `ln(viewers_h / viewers_1)` for h = 2..24.
2. **Shape and spread.** Per hour, the median log-ratio across launches is
   the typical day's shape; the median absolute deviation around it
   (×1.4826, then widened ×1.3) is the hour's log-spread.
3. **Forecast.** `q_tau(h) = p1 · exp(median_h + z_tau · sigma_h)`, where
   `p1` is the case's hour-1 print. The written file is then re-read and
   checked against the output contract before the run ends.

The pooled shape is the whole point: a midnight-local rolling launch dips
to ~0.73× of hour 1 around hour 4 (Europe asleep, the US not yet unlocked),
recovers through the UTC morning, and peaks at ~2.4× hour 1 around hour 19,
when the US audience is fully on — the pattern GTA VI's country-by-country
midnight unlock (New Zealand 11:00Z on the 18th, Europe 23:00–00:00Z, US
East 05:00Z, US West 08:00Z) sets up. A flat forecast pays for that shape on
every hour.

Calibration (leave-one-out over the 38 comparables in the practice cases,
scored with the contest's own pinball loss on the log scale): pooled model
≈ **0.79** vs the flat baseline's ≈ **0.93** on the same launches; ≈ **0.34**
on the midnight-local rolling subset GTA VI belongs to.

## Anchors from the comparables

All figures are SullyGnome hourly Twitch category averages, as bundled in
the competition kit's case data (see the kit's `input/NOTICE.txt` for
provenance):

| Launch | Hour 1 (UTC day) | Day peak (hourly avg) | Note |
| --- | ---: | ---: | --- |
| Cyberpunk 2077 (2020) | 851,999 | 1,069,278 | Largest peak in the set |
| ELDEN RING (2022) | 875,155 | 875,155 | Hour 1 was its peak — PC unlocked globally the night before |
| Diablo IV (2023) | 533,455 | 673,808 | |
| Hogwarts Legacy (2023) | 387,358 | 504,995 | Rolling midnight local |
| Ghost of Yōtei (2025) | 27,206 | 98,353 | PS5-only rolling launch, 3.6× rise |
| Marvel's Wolverine (2026, practice) | 23,357 | — | Practice case p00's hour 1 |
| Red Dead Redemption 2 (2018, practice) | 116,550 | — | Rockstar's previous launch, case p01 |

On the practice cases the program's median peaks at hour 19 at ≈2.4× each
case's hour-1 print (e.g. RDR2: 116,550 → ≈279,900). For GTA VI itself the
absolute numbers scale with whatever hour 1 prints; the shape is the
forecast.

## Run it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

reads every `<id>.in` in `MAKEFUN_INPUT_DIR` (default `/input`) and writes
`<id>.out` — `{"hours": [{"hour": h, "q": [q05, q25, q50, q75, q95]}, …]}`
for hours 2–24 — to `MAKEFUN_OUTPUT_DIR` (default `/output`), printing a
one-line summary per case. Deterministic: the same case always produces
the same bytes.

---

Built for [make.fun](https://make.fun/?ref=cfpg3p8n) — bounty
[gta6-twitch-ccu-forecast-01](https://make.fun/bounty/gta6-twitch-ccu-forecast-01).
Viewer figures: SullyGnome via the competition kit; this project is not
affiliated with Twitch, Rockstar Games, or SullyGnome.
