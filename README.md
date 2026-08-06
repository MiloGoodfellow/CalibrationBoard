# CalibrationBoard - Prediction Market Data Modeling Pipeline

![Top forecasters by Kelly-criterion calibration](banner.png)

## Overview

Pulls the full public history off the [Manifold Markets](https://manifold.markets)
API and ranks forecasters by how well-calibrated their bets actually were, rather
than by profit - which mostly measures how much mana someone was willing to risk.

Each bet is scored with a Kelly-criterion metric against the bettor's balance at
the time of the bet, so a confident bet counts for more than a hedge. Users are
then ranked by the **lower bound of the 95% confidence interval** of their score,
not the mean, so a small number of lucky trades cannot reach the top of the board.

## Scale

- 6,200+ API requests across 24,000+ markets, 139,000+ bets and 5,800+ users
- Filtered to 341 resolved markets, excluding self-resolved and creator-bet markets
- 5,851 unique bettors scored; the top 30 are reported

The banner above shows the top 12.

## Pipeline

`TopManifold.ipynb`, in order:

1. Fetch topic groups, select by member count, fetch markets per group
2. Keep only resolved markets that pass `mconditions` (resolved, not creator-bet)
3. Fetch every bet on those markets, keep bets older than a day before resolution
4. Reconstruct each user's balance history to get the Kelly proportion per bet
5. Score each bet, then take the lower bound of each user's 95% CI
6. Sort and report the leaderboard

## Running it

```
pip install requests numpy scipy matplotlib
```

Every cell hits the live API, so a full run takes thousands of requests and the
results move as new markets resolve. The recorded outputs in the notebook are from
the run described above.

## Known rough edges

- Clipping is applied to the Kelly proportion where balance reconstruction goes
  negative; noted inline in the notebook as worth revisiting.
- `scipy.stats.t.interval` emits invalid-value warnings for users with a single
  scored bet; those users are filtered out before ranking.
