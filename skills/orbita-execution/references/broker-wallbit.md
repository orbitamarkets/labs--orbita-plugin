# Broker: Wallbit

Annex to the `orbita-execution` skill for anyone trading at Wallbit with their own API key. The general flow (sizes, prices, flatten) is in the skill; this covers only Wallbit specifics. Órbita never asks for or receives your key. Use `broker: { preset: "wallbit" }` in `orbita_get_next_orders`.

Base: `https://api.wallbit.io`, header `X-API-Key`. Investment account only.

## What the API really accepts (tested live)

| Order | Works | Size | Also |
|---|---|---|---|
| `MARKET` | Yes | `amount` (USD) **or** `shares` | — |
| `LIMIT` | Yes | **`shares` only** | `limit_price`, `time_in_force` `DAY` or `GTC` |
| `STOP`, `STOP_LIMIT` | **No** (422 even though the docs list them) | — | — |

- Never send `amount` and `shares` together.
- `currency` is always `"USD"`.
- A `201` with `status: REQUESTED` **is not a fill**. Confirm in `GET /api/public/v1/transactions?page=1`: `PENDING` → `COMPLETED` or `FAILED`.
- **No cancel and no OCO.** A pending SELL **locks those shares**: a second SELL on them returns 422. Never place two sells on the same lot.
- Fractional LIMIT sometimes returns 422 `shares must be an integer`. Prefer whole shares on LIMIT and leave the fractional residue under watch (software stop).
- A pending BUY does not debit cash until it fills.
- A `DAY` order expires at the close; `GTC` stays.

## Book at Wallbit

`GET /api/public/v1/balance/stocks`: cash comes as an item with `symbol: "USD"`; the rest are positions (`symbol`, `shares`). For a MARKET BUY send `amount` in USD; for LIMIT, `shares`.

## Take profit and stop loss at Wallbit

With no STOP and no OCO, Órbita uses the *software stop* scheme of the `orbita-execution` skill (the `wallbit` preset does this):

- **Take profit**: a single `SELL LIMIT` for the whole shares of the position at the take profit level.
- **Software stop**: every slot, if `last ≤ cost × (1 + stopLossPctFromCost/100)` and there are free shares → `SELL MARKET` for those shares.
- **The trap**: if the TP LIMIT locks all shares and the price drops below the stop, the MARKET sell returns 422. Ways out: wait for the DAY order to expire, for the TP to fill, or cancel manually in the Wallbit app. To reduce the risk, prefer `DAY` take profits and re-place them each morning if the position remains.

## Flatten

Before `flattenBy`: sell everything with `MARKET` by shares. If a sale is locked by a pending LIMIT, Thursday's DAY orders have already expired by Friday morning: use `DAY` take profits the days before a flatten.

## API limits

- On 429 honor `Retry-After` and retry at most once. Cloudflare 1015 is an IP rate limit, not a Wallbit outage.
- Per slot, 1× `balance/stocks` and 1–2 pages of `transactions` is enough.
- Errors: 400 insufficient funds · 401 invalid key · 403 missing `trade` scope · 404 symbol · 412 KYC or account locked · 422 validation.
