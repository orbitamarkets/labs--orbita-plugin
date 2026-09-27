# Cálculo de órdenes (lo que hace orbita_get_next_orders)

Referencia para quien traduce el plan por su cuenta o quiere auditar el resultado.

- **Libro** = efectivo + Σ (shares × último precio) de las posiciones **del universo**. Lo que esté fuera del universo no cuenta.
- **Orden de evaluación:** flatten → stops y tenencia máxima → ventas del plan → compras del plan → protección.
- **Flatten:** desde una hora antes de `flattenBy`, todo a MARKET (cancelando lo que bloquee, si el broker deja). No se ejecutan acciones del plan.
- **Stop:** si `último ≤ costo × (1 + stopLossPctFromCost/100)`, vender todo a MARKET.
- **Tenencia máxima:** si pasaron `maxHoldSessions` días hábiles desde `openedAt`, vender a MARKET.
- **Ventas del plan:** `pct_of_position` × shares; `exit` = toda la posición. Solo shares libres (no bloqueadas por otra venta abierta).
- **Compras del plan:** USD = libro × `pct_of_book` / 100, limitado al efectivo disponible (descontando compras abiertas). Si queda por debajo de `minTicketPctOfBook`, se saltea. `open` se saltea si ya hay posición; cualquier compra se saltea si ya hay una compra abierta del símbolo. Entradas solo en sesión o hasta 90 minutos antes de la apertura.
- **Precio LIMIT:** `abs` = ese precio; `pct_from_cost` = costo × (1 + v/100); `pct_from_last` = último × (1 + v/100). Redondeo al centavo.
- **Trigger:** `last_at_or_above` / `last_at_or_below` contra el último precio; si no se cumple, la acción se saltea.
- **Shares:** hacia abajo a 4 decimales si el broker acepta fracciones para ese tipo de orden; si no, a entero. Si da 0, se saltea.
- **Protección:** TP = costo × (1 + takeProfitPctFromCost/100); SL = costo × (1 + stopLossPctFromCost/100). `DAY` si el flatten es en menos de 24 h, `GTC` si no.
