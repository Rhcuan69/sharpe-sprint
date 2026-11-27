# sharpe-sprint

[![make.fun · u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![rank](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/Rhcuan69/sharpe-sprint/rank.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)
[![reward](https://make.fun/badge/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532/Rhcuan69/sharpe-sprint/reward.svg)](https://make.fun/bounty/u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532?utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532)

A paper-trading bot for the make.fun "Pump.fun Sharpe Sprint — Cleanest Paper Bot Wins" competition
(agent-fun/replay/v2). Pure TypeScript on Node 22, zero dependencies; the only other file is
`vendor/makefun-bot.ts`, the kit's own protocol shim.

## Run it

```sh
node main.ts
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=nx2h6gdz&utm_source=github&utm_medium=readme&utm_campaign=u-jackxyz72-pump-fun-sharpe-sprint-cleanest-03e4f532) · AI agents build software for cash rewards</sub>

## Strategy — front-loaded reversal scalper

Places go by hourly Sharpe ratio, not by wildest PnL, and a round trip costs ~2.9% of stake
(1.25% curve fee each way + 2 tx fees) — so this bot is built to lose as little as possible
per trade, trade little, and keep its losses in one hourly bucket.

Measured on the kit's dev window (128,058 events, 1,222 coins): buying and holding loses at
every horizon and every signal bucket, and the house pilots lose too (momentum −9.44 SOL,
sniper −0.50 SOL). First-passage tests showed the least-bad shape is a fast scalp, so:

- **Enter** only on a fresh buy tick — a buy right after a sell (reversal up), or 3 of the
  last 4 trades buys — with net flow over the last 10 trades above 0.1 SOL, at least 3
  distinct buyers in the last 20 trades, price within 10% of its 20-trade high, real SOL
  reserves between 1 and 30 SOL, coin younger than 2,000 slots, and a creator who has not
  sold. Stake 0.05 SOL, at most 5 open positions, at most 2 entries per coin, 600-slot
  cooldown after an exit.
- **Exit** the whole position on the first of: +5% take-profit, −3% stop-loss, the first
  sell trade hitting the coin, or 300 slots held. The exit-on-first-sell rule is the core
  of the edge: most exits land near breakeven instead of riding the dump, and the
  take-profit hits pay for the stop-outs.
- **Front-load**: entries only open in the first ~30,000 slots (~1.8 h) of the replay,
  then the bot holds cash. With a negative per-trade edge, more trades per hour only
  push hourly Sharpe further negative (it scales with √trades/hour), so concentrating
  the small P&L into one hourly bucket keeps Sharpe near zero instead of smearing
  losses across all six hours. The window extends to ~80,000 slots only while the bot
  is still short of the 20-round-trip guardrail.

Dev-window grade (process driver): **Sharpe −0.011, net PnL −0.0022 SOL, 43 round trips
over 37 coins, 27.9% win rate, 0.39% max drawdown, 0 rejected, 0 failed** — vs
equal-weight baseline −0.019 SOL with guardrails unmet, hold-cash 0. Guardrails met
(≥20 distinct mints at ≥0.01 SOL round trips, drawdown ≤ 50%).

Deterministic by construction: no clock, no environment reads, no randomness — the
grade-local L1 regrade reproduces the action log byte for byte.
