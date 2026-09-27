---
name: orbita-execution
description: How to execute Órbita's plan at your own broker, whichever it is. Use it every time you send orders from orbita_get_next_orders, or if your broker has restrictions (no STOP, no OCO, no cancel, no fractional shares). Spanish version: i18n/es/SKILL.md.
---

# Executing Órbita's plan

You trade **your** account with **your** credentials. Órbita never asks for them or receives them: do not pass them to any Órbita tool.

## The flow

1. Read your book at the broker: cash, positions with average cost, open orders. Also read the **last price** of each stock you hold and of each stock in the current plan (Órbita does not supply prices).
2. Call `orbita_get_next_orders` with `book`, `prices` (for example `{"NVDA": 224.1}`) and `broker` (your capabilities, or `preset` if your broker is listed). Stops and price triggers are evaluated with your prices; if some are missing, the response lists them in a `SEND_PRICES` alert: fetch them and call again. Órbita never stores or logs your book or prices.
3. Execute `orders` **in `step` order**:
   - `CANCEL`: cancel the matching open order (symbol, side, price).
   - `MARKET`: if it has `amountUsd`, send by amount; if it has `shares`, by shares.
   - `LIMIT` / `STOP` / `OCO`: with the given prices and `timeInForce`.
4. Keep the `ref` of every order you send. If you call again and the same `ref` shows up, **do not send it again** unless the previous one was rejected or expired.
5. Confirm fills at your broker. "Accepted" is not "filled".
6. Check `watches`: conditions you monitor yourself (software stops or take profits, when your broker lacks them natively). Every slot, if the condition is met, do what `then` says.
7. Handle `alerts`, especially `critical` ones: they need an action outside the API (for example, cancelling in the broker's app).
8. Call again at `checkAgainAt`, or earlier if a buy filled: protection (take profit / stop) is computed from the updated book.
9. Report what happened with `orbita_report_execution`: one entry per `ref` with status, shares, average price and time. Reporting the same `ref` again updates it. Órbita stores only what you report (never your book or credentials).

Every alert, reason and skipped action carries a stable `code` (e.g. `SHARES_LOCKED_NO_CANCEL`, `SKIP_TRIGGER_NOT_MET`) besides its translated text: branch on the code, show the text.

## If you do not send your book

`orbita_get_next_orders` without `book` returns the plan in relative terms (percent of book and of position). You can translate it yourself with [references/order-math.md](references/order-math.md), but sending the book is safer: the server applies stops, flatten, rounding and locked shares consistently.

## Broker capabilities

| Capability | What changes |
|---|---|
| `supportsOco` | Protection as a single OCO order (TP + SL) |
| `supportsStopOrders` without OCO | Native STOP for the SL; the TP becomes a `watch` |
| neither | TP as a LIMIT; the SL becomes a `watch` (software stop) |
| `supportsCancel` = false | An open sell locks those shares: Órbita will not propose another sell on them and warns you when a manual cancel is needed |
| `fractional` | `none`: whole shares; `market_only`: fractions only on MARKET; `all`: fractions always |

Brokers with an official MCP and their presets: [references/brokers.md](references/brokers.md). Wallbit specifics: [references/broker-wallbit.md](references/broker-wallbit.md).
