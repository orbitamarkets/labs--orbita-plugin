---
name: orbita-horizonte
description: Órbita Horizonte, the long-term portfolio included in the 6-month and annual plans. How it splits the account with Órbita Pulso, when to ask for its orders, its rules and how to report. Read it before running Horizonte for the first time. Spanish version: i18n/es/SKILL.md.
---

# Órbita Horizonte

Órbita Horizonte is Órbita's long-term strategy: a portfolio of 8 to 12 stocks from the universe, weighted by the desk's conviction, rebalanced monthly. It comes with the 6-month and annual plans. It is general information, the same for every subscriber; whoever executes decides and is responsible for their orders.

## How it shares the account with Órbita Pulso

- **Half and half.** Horizonte works on 50% of the account's value; Órbita Pulso, the short-term plan, on the other 50%. Órbita splits the book for you: send the whole account, it computes each half.
- **Each strategy only touches its own shares.** Órbita tells them apart with your operations ledger (every order has a `ref` and a `strategy`). Pulso's weekend flatten and stops never sell Horizonte's shares, and the other way around.
- **That's why reporting matters:** report every `ref` with `orbita_report_execution` (filled, rejected or skipped). Shares Órbita never issued to you are yours: neither strategy touches them.

## When to ask for its orders

Call `orbita_get_next_orders` with `strategy: "horizonte"`, your `book`, `prices` and `broker`:
- **Once a week**, for example Monday after the open. The first three weeks it builds the portfolio in three tranches (a third each week).
- **The first business day of each month**, after the desk publishes the new target portfolio.
- **Right after you add money** to the account: the new cash goes toward the targets.

Pulso still runs every slot with `strategy: "pulso"` (the default). Execute each response's orders in `step` order and never resend a `ref`.

## Rules

- **Portfolio:** 8 to 12 stocks; at most 15% per stock and 40% per sector; between 80% and 95% invested (always invested once built).
- **Stop −10% and take profit +20%** from the cost of each Horizonte position. Where your broker has no stop orders, Órbita returns them as software stops (`watches`) and sells at MARKET when a call finds the price past them.
- **Rebalance:** a stock more than 5 points above its target is trimmed; a stock that leaves the portfolio is sold. Gaps are refilled on the next call, without waiting for the month.
- **No weekend flatten and no maximum holding time:** it's long term.
- **Stale portfolio:** if the target portfolio is more than 45 days old, there are no new buys; sells, stops and take profits keep working.

## What comes back

The same response as Pulso: `orders`, `watches`, `alerts`, `skipped`, `checkAgainAt`, with `strategy: "horizonte"` in each order and `risk` with −10/+20 and no `flattenBy`. `_meta` adds the tranche, the Horizonte budget and the account value.

Without a 6-month or annual plan, `strategy: "horizonte"` returns an access error: keep using Pulso.
