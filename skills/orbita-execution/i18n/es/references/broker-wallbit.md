# Broker: Wallbit

Anexo de la skill `orbita-execution` para quien opera en Wallbit con su propia API key. Lo general (tamaños, precios, flatten) está en la skill; aquí solo lo específico de Wallbit. Órbita nunca pide ni recibe tu key. Usa `broker: { preset: "wallbit" }` en `orbita_get_next_orders`.

Base: `https://api.wallbit.io`, header `X-API-Key`. Solo la cuenta de inversiones.

## Lo que la API acepta de verdad (probado en vivo)

| Orden | Funciona | Tamaño | Además |
|---|---|---|---|
| `MARKET` | Sí | `amount` (USD) **o** `shares` | — |
| `LIMIT` | Sí | **solo `shares`** | `limit_price`, `time_in_force` `DAY` o `GTC` |
| `STOP`, `STOP_LIMIT` | **No** (422 aunque la documentación las liste) | — | — |

- Nunca envíes `amount` y `shares` juntos.
- `currency` siempre `"USD"`.
- La respuesta `201` con `status: REQUESTED` **no es un fill**. Confirma en `GET /api/public/v1/transactions?page=1`: `PENDING` → `COMPLETED` o `FAILED`.
- **No hay cancelar ni OCO.** Un SELL pendiente **bloquea esas shares**: un segundo SELL sobre las mismas devuelve 422. Nunca pongas dos ventas sobre el mismo lote.
- LIMIT con shares fraccionales a veces devuelve 422 `shares must be an integer`. Prefiere shares enteras en LIMIT y deja la fracción residual bajo vigilancia (stop por software).
- Un BUY pendiente no descuenta efectivo hasta ejecutarse.
- Una orden `DAY` vence al cierre; `GTC` sigue.

## Libro en Wallbit

`GET /api/public/v1/balance/stocks`: el efectivo viene como un item con `symbol: "USD"`; el resto son posiciones (`symbol`, `shares`). Para MARKET BUY envía `amount` en USD; para LIMIT, `shares`.

**Las shares de `transactions` vienen redondeadas a 2 decimales** (una compra de 0,0192 acciones figura como 0,01; una de 0,0091, como 0). Toma la cantidad de la posición de `balance/stocks` y, para cada operación, calcula las shares como monto / `share_price`: así sale el costo promedio real y las `shares` correctas para `orbita_report_execution`.

## Take profit y stop loss en Wallbit

Como no hay STOP ni OCO, se usa el esquema de *stop por software* de la skill `orbita-execution` (Órbita lo calcula con el preset `wallbit`):

- **Take profit**: una sola orden `SELL LIMIT` por las shares enteras de la posición, al precio de `risk.takeProfitPctFromCost`.
- **Stop loss por software**: en cada slot, si `último ≤ costo × (1 + stopLossPctFromCost/100)` y hay shares libres → `SELL MARKET` por esas shares.
- **La trampa**: si el TP LIMIT bloquea todas las shares y el precio cae bajo el stop, el MARKET devuelve 422. Salidas: esperar que venza la DAY, que se llene el TP, o cancelar a mano en la app de Wallbit. Para reducir el riesgo, prefiere TP `DAY` y vuelve a colocarlo cada mañana si la posición sigue.

## Flatten

Antes de `risk.flattenBy`: vende todo con `MARKET` por shares. Si una venta está bloqueada por un LIMIT pendiente, el viernes por la mañana las `DAY` del jueves ya vencieron: usa `DAY` en los TP de los días previos a un flatten.

## Límites de la API

- En 429 respeta `Retry-After` y no reintentes más de una vez. Cloudflare 1015 es límite por IP, no una caída de Wallbit.
- Por slot alcanza con 1× `balance/stocks` y 1–2 páginas de `transactions`.
- Errores: 400 fondos insuficientes · 401 key inválida · 403 falta scope `trade` · 404 símbolo · 412 KYC o cuenta bloqueada · 422 validación.
