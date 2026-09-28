---
name: orbita-horizonte
description: Órbita Horizonte, la cartera de largo plazo incluida en los planes semestral y anual. Cómo reparte la cuenta con Órbita Pulso, cuándo pedir sus órdenes, sus reglas y cómo reportar.
---

# Órbita Horizonte

Órbita Horizonte es la estrategia de largo plazo de Órbita: una cartera de 8 a 12 acciones del universo, con pesos según la convicción de la mesa y rebalanceo mensual. Viene con los planes semestral y anual. Es información general, igual para todos los suscriptores; quien ejecuta decide y es responsable de sus órdenes.

## Cómo comparte la cuenta con Órbita Pulso

- **Mitad y mitad.** Horizonte trabaja sobre el 50 % del valor de la cuenta; Órbita Pulso, el plan de corto plazo, sobre el otro 50 %. Órbita separa el libro por ti: envía la cuenta completa y calcula cada mitad.
- **Cada estrategia toca solo sus acciones.** Órbita las distingue con tu libro de operaciones (cada orden tiene un `ref` y una `strategy`). El flatten de fin de semana y los stops de Pulso nunca venden acciones de Horizonte, ni al revés.
- **Por eso importa reportar:** reporta cada `ref` con `orbita_report_execution` (ejecutada, rechazada u omitida). Las acciones que Órbita nunca te emitió son tuyas: ninguna estrategia las toca.

## Cuándo pedir sus órdenes

Llama a `orbita_get_next_orders` con `strategy: "horizonte"`, tu `book`, `prices` y `broker`:
- **Una vez por semana**, por ejemplo el lunes después de la apertura. Las primeras tres semanas arma la cartera en tres tramos (un tercio por semana).
- **El primer día hábil de cada mes**, después de que la mesa publica la cartera objetivo nueva.
- **Apenas agregues dinero** a la cuenta: el efectivo nuevo va hacia los objetivos.

Pulso sigue en cada slot con `strategy: "pulso"` (la opción por defecto). Ejecuta las órdenes de cada respuesta en orden de `step` y nunca reenvíes un `ref`.

## Reglas

- **Cartera:** de 8 a 12 acciones; como máximo 15 % por acción y 40 % por sector; entre 80 % y 95 % invertido (siempre invertida una vez armada).
- **Stop −10 % y take profit +20 %** desde el costo de cada posición de Horizonte. Si tu broker no tiene órdenes stop, Órbita las devuelve como stops por software (`watches`) y vende a MARKET cuando una consulta encuentra el precio del otro lado.
- **Rebalanceo:** una acción más de 5 puntos por encima de su objetivo se recorta; una acción que sale de la cartera se vende. Los huecos se completan en la consulta siguiente, sin esperar al mes.
- **Sin flatten de fin de semana ni tenencia máxima:** es largo plazo.
- **Cartera vencida:** si la cartera objetivo tiene más de 45 días, no hay compras nuevas; ventas, stops y take profits siguen funcionando.

## Qué devuelve

La misma respuesta que Pulso: `orders`, `watches`, `alerts`, `skipped`, `checkAgainAt`, con `strategy: "horizonte"` en cada orden y `risk` con −10/+20 y sin `flattenBy`. `_meta` suma el tramo, el presupuesto de Horizonte y el valor de la cuenta.

Sin plan semestral o anual, `strategy: "horizonte"` devuelve un error de acceso: sigue con Pulso.
