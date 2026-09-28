---
name: orbita-strategy
description: Mandate and risk rules of Órbita Pulso, Órbita's short-term strategy (long only, USD, 35 US-listed names in aerospace, defense, compute, energy and robotics). Read it before trading for the first time or when you need to explain the strategy to a person. Spanish version: i18n/es/SKILL.md.
---

# Órbita Pulso strategy

This skill covers **Órbita Pulso**, Órbita's short-term strategy (orders tagged `strategy: "pulso"`). It is the same for every subscriber and not personalized: the same plan, scaled to each book. It is general information, not personalized advice; whoever places the orders decides and is responsible for them.

## What is traded

- **Long only.** A SELL only closes or reduces a long. Never shorts, margin, options, crypto or non-US listings.
- **USD**, US-listed stocks.
- **Universe (35), in five sectors:**
  - Aerospace (8): BA, HWM, TDG, LUNR, PL, RKLB, SPCE, SPCX (SpaceX).
  - Defense (5): GD, LHX, LMT, NOC, RTX.
  - Compute (8): NVDA, AMD, AVGO, TSM, MU, ARM, CRWV, NBIS.
  - Energy (8): GEV, CEG, VST, CCJ, OKLO, SMR, BE, LEU.
  - Robotics (6): ISRG, TSLA, SYM, TER, ROK, SERV.

  Nothing outside this list. The desk reviews the universe regularly and may add or remove names; the tools always reflect the current list.
- **Only the investment account** at the broker. Never checking or savings accounts, never transfers between accounts.

## Holding period and calendar

- Short term, during the regular US session (Monday to Friday, 9:30–16:00 ET).
- **Flatten:** before any market closure longer than a day (weekend, NYSE holiday) the book goes **100% cash**. The deadline is given in the plan (`flattenBy`). This rule does not depend on the plan: if no plan arrives, it still applies.
- **Max hold** per position: `maxHoldSessions` sessions.

## Risk rules (per position)

- **Take profit:** +3% above average cost (`takeProfitPctFromCost`).
- **Stop loss:** −2% below average cost (`stopLossPctFromCost`).
- **Minimum ticket** for a buy: 10% of the book (`minTicketPctOfBook`).

The current values are always given in the plan; if they differ from these, the plan wins.

## The plan rules

- Trade **only what the plan says**, when there is a thesis and a price. There are no targets for number of trades or open positions.
- **Cash is a valid position.** A plan with no actions is a decision, not a failure.
- If the plan is stale, do not open new positions: only stops, take profits and flatten.
- The why behind each decision is in `orbita_get_briefing`.
