# Ejecutar el plan de Órbita

Operas **tu** cuenta con **tus** credenciales. Órbita nunca las pide ni las recibe: no se las pases a ninguna tool de Órbita.

## El flujo

1. Lee tu libro en el broker: efectivo, posiciones con costo promedio, órdenes abiertas. Lee también el **último precio** de cada acción que tienes y de cada acción del plan vigente (Órbita no provee precios).
2. Llama a `orbita_get_next_orders` con `book`, `prices` (por ejemplo `{"NVDA": 224.1}`) y `broker` (tus capacidades, o `preset` si tu broker está listado). Los stops y los triggers de precio se evalúan con tus precios; si falta alguno, la respuesta lo lista en una alerta `SEND_PRICES`: obtenlos y vuelve a llamar. Órbita usa tu libro y tus precios para calcular y los descarta: nunca los guarda ni los registra.
3. Ejecuta `orders` **en el orden de `step`**:
   - `CANCEL`: cancela la orden abierta que coincide (símbolo, lado, precio) y confirma que el broker la canceló. Una venta que necesita una cancelación nunca viene en la misma respuesta: después de las cancelaciones (aviso `CALL_AGAIN_AFTER_CANCEL`), vuelve a llamar con el libro actualizado y la venta llega calculada sobre lo que todavía tienes.
   - `MARKET`: si viene `amountUsd`, envíala por monto; si viene `shares`, por shares.
   - `LIMIT` / `STOP` / `OCO`: con los precios y el `timeInForce` indicados.
4. Guarda el `ref` de cada orden enviada. Si vuelves a llamar y aparece el mismo `ref`, **no la envíes de nuevo** salvo que la anterior haya sido rechazada o haya vencido. Deja alrededor de un segundo entre órdenes: algunos brokers (Wallbit) limitan las ráfagas y responden 429; espera unos segundos y vuelve a enviarla.
5. Confirma los fills en tu broker. "Aceptada" no es "ejecutada".
6. Revisa `watches`: son condiciones que vigilas tú (stops o take profits por software, cuando tu broker no los tiene nativos). En cada slot, si se cumple la condición, haz lo que dice `then`.
7. Atiende las `alerts`, sobre todo las `critical`: requieren una acción tuya fuera de la API (por ejemplo, cancelar en la app del broker).
8. Vuelve a llamar en `checkAgainAt`, o antes si se ejecutó una compra: la protección (take profit / stop) se calcula sobre el libro actualizado.
9. Reporta lo que pasó con `orbita_report_execution`: una entrada por `ref` con estado, shares, precio promedio y hora, incluidas las órdenes rechazadas y salteadas. Reportar de nuevo el mismo `ref` lo actualiza. Tus reportes arman el **libro de operaciones** con Órbita: el historial de la persona y qué acciones son de cada estrategia (`strategy` en cada orden). `orbita_get_ledger` lo muestra. Si una respuesta trae `REPORT_PENDING`, reporta esos refs; si trae `LEDGER_MISMATCH`, tu broker y el libro no coinciden (una venta fuera de Órbita o una orden sin reportar): avísale a la persona. Órbita guarda las órdenes que emite y lo que reportas, nunca tu libro ni credenciales.

Cada aviso, motivo y acción salteada trae un `code` estable (ej. `SHARES_LOCKED_NO_CANCEL`, `SKIP_TRIGGER_NOT_MET`) además del texto traducido: decide por el código y muestra el texto.

## Si no envías el libro

`orbita_get_next_orders` sin `book` devuelve el plan en términos relativos (porcentajes del libro y de la posición). Puedes traducirlo tú con [references/order-math.md](references/order-math.md), pero enviar el libro es más seguro: el servidor aplica stops, flatten, redondeos y bloqueos de shares de forma consistente.

## Capacidades del broker

| Capacidad | Qué cambia |
|---|---|
| `supportsOco` | Protección con una sola orden OCO (TP + SL) |
| `supportsStopOrders` sin OCO | STOP nativo para el SL; el TP queda como `watch` |
| ninguno de los dos | TP como LIMIT; el SL queda como `watch` (stop por software) |
| `supportsCancel` = false | Una venta abierta bloquea esas shares: Órbita no te propone otra venta sobre ellas y te avisa si hace falta cancelar a mano |
| `fractional` | `none`: shares enteras; `market_only`: fracciones solo en MARKET; `all`: fracciones siempre |

Brokers con MCP oficial y sus presets: [references/brokers.md](references/brokers.md). Particularidades de Wallbit: [references/broker-wallbit.md](references/broker-wallbit.md).
