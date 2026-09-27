# Order math (what orbita_get_next_orders does)

Reference for anyone translating the plan on their own or auditing the result.

- **Book** = cash + Σ (shares × last price) of positions **in the universe**. Anything outside the universe does not count.
- **Evaluation order:** flatten → stops and max hold → plan sells → plan buys → protection.
- **Flatten:** from one hour before `flattenBy`, everything at MARKET (cancelling what locks shares, if the broker allows). Plan actions are not executed.
- **Stop:** if `last ≤ cost × (1 + stopLossPctFromCost/100)`, sell everything at MARKET.
- **Max hold:** if `maxHoldSessions` business days passed since `openedAt`, sell at MARKET.
- **Plan sells:** `pct_of_position` × shares; `exit` = the whole position. Only free shares (not locked by another open sell).
- **Plan buys:** USD = book × `pct_of_book` / 100, capped at available cash (minus open buys). Below `minTicketPctOfBook` it is skipped. `open` is skipped if there is already a position; any buy is skipped if there is already an open buy for the symbol. Entries only during the session or up to 90 minutes before the open.
- **LIMIT price:** `abs` = that price; `pct_from_cost` = cost × (1 + v/100); `pct_from_last` = last × (1 + v/100). Rounded to the cent.
- **Trigger:** `last_at_or_above` / `last_at_or_below` against the last price; if unmet, the action is skipped.
- **Shares:** floored to 4 decimals if the broker accepts fractions for that order type; otherwise to whole shares. If the result is 0, it is skipped.
- **Protection:** TP = cost × (1 + takeProfitPctFromCost/100); SL = cost × (1 + stopLossPctFromCost/100). `DAY` if the flatten is less than 24 h away, `GTC` otherwise.
