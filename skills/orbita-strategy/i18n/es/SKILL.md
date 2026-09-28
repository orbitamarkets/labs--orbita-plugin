# Estrategia Órbita Pulso

Esta skill describe **Órbita Pulso**, la estrategia de corto plazo de Órbita (órdenes con `strategy: "pulso"`). Es la misma para todos los suscriptores y no se personaliza: el mismo plan, escalado al libro de cada uno. Es información general, no asesoramiento personalizado; quien ejecuta decide y es responsable de sus órdenes.

## Qué se opera

- **Long only.** Un SELL solo cierra o achica un long. Nunca shorts, margen, opciones, cripto ni instrumentos listados fuera de EE.UU. (por ejemplo, CEDEARs).
- **USD**, acciones listadas en EE.UU.
- **Universo (35), en cinco sectores:**
  - Aeroespacial (8): BA, HWM, TDG, LUNR, PL, RKLB, SPCE, SPCX (SpaceX).
  - Defensa (5): GD, LHX, LMT, NOC, RTX.
  - Cómputo (8): NVDA, AMD, AVGO, TSM, MU, ARM, CRWV, NBIS.
  - Energía (8): GEV, CEG, VST, CCJ, OKLO, SMR, BE, LEU.
  - Robótica (6): ISRG, TSLA, SYM, TER, ROK, SERV.

  Fuera de esta lista no se opera. La mesa revisa el universo periódicamente y puede sumar o sacar acciones; las tools siempre reflejan la lista vigente.
- **Solo la cuenta de inversiones** del broker. Nunca cuentas corrientes, de ahorro ni transferencias entre cuentas.

## Plazo y calendario

- Corto plazo, en sesión regular de EE.UU. (lunes a viernes, 9:30–16:00 ET).
- **Flatten:** antes de cualquier cierre de mercado de más de un día (fin de semana, feriado de NYSE), el libro queda **100% en efectivo**. La hora límite viene en el plan (`flattenBy`). Esta regla no depende del plan: si el plan no llega, igual se aplica.
- **Tenencia máxima** por posición: `maxHoldSessions` sesiones.

## Reglas de riesgo (por posición)

- **Take profit:** +3% sobre el costo promedio (`takeProfitPctFromCost`).
- **Stop loss:** −2% sobre el costo promedio (`stopLossPctFromCost`).
- **Ticket mínimo** de compra: 10% del libro (`minTicketPctOfBook`).

Los valores vigentes vienen siempre en el plan; si difieren de estos, mandan los del plan.

## El plan manda

- Se opera **solo lo que el plan indica**, cuando hay tesis y precio. No hay metas de cantidad de operaciones ni de posiciones abiertas.
- **El efectivo es una posición válida.** Un plan sin acciones es una decisión, no una falla.
- Si el plan está vencido, no se abren posiciones nuevas: solo stops, take profits y flatten.
- El porqué de cada decisión está en `orbita_get_briefing`.
