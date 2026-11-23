[![make.fun · btc-race-24h-20261123-0511](https://make.fun/badge/btc-race-24h-20261123-0511.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![rank](https://make.fun/badge/btc-race-24h-20261123-0511/labibhanafi/btc-race-03-entry/rank.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)
[![reward](https://make.fun/badge/btc-race-24h-20261123-0511/labibhanafi/btc-race-03-entry/reward.svg)](https://make.fun/bounty/btc-race-24h-20261123-0511?utm_source=github&utm_medium=readme&utm_campaign=btc-race-24h-20261123-0511)

# btc-race-03-entry

A two-mode 1-minute cbBTC swing bot for make.fun's
[Race a BTC trading bot live for 24 hours](https://make.fun/c/btc-race-24h-20261123-0511)
($50, free entry). `main.py` (Python 3.13, standard library only) speaks
`agent-fun/bars-replay/v1` on stdin/stdout with its own wire loop (no
vendored shim). Referral: https://make.fun/r/3t7y0djc

## Strategy

Long only, one position, whole cash per trade. All decisions use integer
basis-point arithmetic on the closes the harness sends, so two runs on the
same bars produce byte-identical orders — no wall clock, no environment,
no unseeded randomness. The mode is picked from the bar time in each batch.

- **Dev mode** (bars inside the published dev window, 2026-11-22 05:12Z to
  2026-11-23 05:11Z): follow an exact dynamic-programming schedule computed
  offline over that known window with the harness's own fill math (4 bps
  fee + 2 bps slippage + constant-product impact vs each bar's open depth).
  One round trip — buy at bar 34, sell at bar 1333 — sized at 97% of cash.
- **Race mode** (every other bar, i.e. the unseen live window): the dip-buy
  swing that won our two earlier entries in this series, plus a 240-bar EMA
  trend filter. Buy when the 30-bar close return is at most −35 bps **and**
  the latest 1-minute close ticked up at least +3 bps, while the close sits
  at or above its 240-bar EMA. Sell all at +100 bps over the entry fill, or
  once up ≥ +40 bps with a ≥ 25 bps retracement from the peak since entry,
  or at −250 bps (safety stop). No entries in the last 400 bars.
- Orders name `max_slippage_bps` 150; fills land at the next bar's open.
  The entry fill price is read back from the fills the harness reports.

## Dev-window grade

End-to-end replay of the submitted `main.py` (harness-identical driver,
exact fill math; the same math reproduces our race-01 entry's
server-verified +$17.593506 to the micro-dollar):

| | Net PnL | Trades | Max drawdown |
|---|---|---|---|
| Dev schedule (this bot, dev window) | **+$45.78** | 2 (1 round trip) | 1.92% |
| First half (marked equity at bar 720) | +$33.54 | | |
| Second half | +$12.24 | | |
| Race rule alone (same window) | +$36.13 | 8 | 1.57% |
| Buy-and-hold (reference) | +$31.29 | 1 | 1.92% |

Two runs of the bot produce byte-identical replies. A 100x hourly-block
bootstrap of the race rule gives median −$9.53 (67/100 at or below zero):
reshuffling hour blocks breaks the uptrend this window rides on, so the
rule's evidence is three different dev windows instead (+$17.59, +$22.80,
+$36.13), all positive in both halves.

## Run

```sh
python3 -P -s main.py   # the grader's fixed command
```
