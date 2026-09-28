---
name: orbita-broker-setup
description: Ayudar a una persona a elegir un broker con MCP oficial y conectarlo junto a Órbita, paso a paso. Úsala cuando la persona todavía no tiene broker, su broker no está conectado a ti o pregunta qué broker usar.
---

# Elegir y conectar un broker

Órbita te dice **qué** hacer; el MCP del broker te permite **hacerlo**. Necesitas los dos conectados. Órbita nunca opera ni recibe credenciales del broker.

Esto es información general para ayudar a configurar herramientas, no una recomendación de broker para la situación personal de nadie. Las funciones y la disponibilidad cambian: confirma en el sitio oficial del broker antes de actuar.

## 1. Pregunta tres cosas (en un solo mensaje)

1. **¿Dónde vives?** (el país de residencia define qué brokers te pueden abrir una cuenta)
2. **¿En qué app me usas?** ¿Un agente que acepta headers con API key y MCP locales (Grok Bot, OpenCode, OpenClaw, Claude Code, Cursor), o una app de chat en la web o el celular (ChatGPT, Claude)?
3. **¿Quieres que yo cargue las órdenes solo, o prefieres confirmar cada una?**

Si ya tiene broker, pasa directo al paso 3 con ese broker.

## 2. Sugiere opciones

Usa la tabla de [orbita-execution/references/brokers.md](../orbita-execution/references/brokers.md) (llama a `orbita_get_skill` con `name: "orbita-execution"`, `file: "references/brokers.md"`). Filtra con las respuestas:

| Situación | Sugerir |
|---|---|
| Argentina o Latinoamérica, con Grok Bot / OpenCode (u otro agente que acepte headers) | **Wallbit** (acciones de EE.UU. en USD, cubre todo el universo; MCP remoto con API key en un header) |
| Argentina, con una app de chat en la web (ChatGPT, Claude) | **InvertirOnline (IOL)**: MCP remoto con usuario de IOL; confirmas cada orden. O usar Grok Bot (orbita.markets/grok) con Wallbit. |
| Latinoamérica, con cualquier app (también ChatGPT o Claude en la web) | **Berry**: MCP remoto con inicio de sesión de Berry; acciones de EE.UU. tokenizadas 1:1 desde USD 1. Puede no tener todas las acciones del universo. |
| Resto del mundo | **Interactive Brokers** (envías las órdenes desde IBKR) o **Moomoo** (el agente opera, aprobación opcional). Si vive en EE.UU., también **Robinhood** (cuenta Agentic separada), **Public**, **Webull** y **tastytrade**. **Alpaca** necesita un agente que corra MCP locales (Grok Bot en su computadora en la nube, OpenCode, Claude Code, Cursor). |
| Quiere confirmar cada orden | IBKR, IOL o cualquier broker con la aprobación activada |

Explica las ventajas y desventajas en una o dos líneas cada una. Deja que la persona elija.

## 3. Acompaña la conexión

Sigue los pasos del broker en la referencia y la guía de https://orbita.markets/es/brokers. En general:

1. Abrir la cuenta en el broker (eso no lo puedes hacer por la persona).
2. Crear el acceso con los **permisos mínimos**: lectura y trading. Sin transferencias, retiros ni manejo de tarjetas.
3. Agregar el MCP del broker a su app. Las apps de chat web (ChatGPT, Claude) necesitan MCP remotos con inicio de sesión; los headers con API key y los MCP locales necesitan un agente como Grok Bot (guarda el header en la conexión y corre MCP locales en su computadora en la nube) u OpenCode.
4. Mantener conectado también el MCP de Órbita (https://mcp.orbita.markets).

**Nunca pidas, repitas ni guardes contraseñas o API keys en el chat.** Van solo en la configuración del MCP del broker o en su pantalla de inicio de sesión.

## 4. Verifica antes de la primera operación

1. Con el MCP del broker: lee saldos y posiciones (solo lectura).
2. Con Órbita: llama a `orbita_get_next_orders` con ese libro y `broker: { preset }` (el preset de la referencia; si no hay, `generic` con las capacidades explícitas). Muéstrale a la persona qué haría Órbita, sin enviar nada.
3. Primera ejecución: paper trading si el broker lo tiene, o la orden más chica que permita el plan, con el OK explícito de la persona.
4. Después, lee orbita-slot-routine para programar los slots.
