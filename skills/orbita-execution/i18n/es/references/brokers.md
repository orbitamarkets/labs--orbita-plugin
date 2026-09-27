# Brokers con MCP oficial

Anexo de la skill `orbita-execution`. Tu agente necesita **dos servidores MCP**: el de Órbita (el plan) y el de tu broker (la ejecución). Órbita nunca recibe las credenciales del broker: van solo al MCP del broker. Usa el `preset` de abajo en `orbita_get_next_orders`. Los detalles cambian; consulta siempre la documentación oficial del broker.

| Broker | MCP | Acceso | Quién manda la orden | `preset` |
|---|---|---|---|---|
| Wallbit | Oficial, remoto: `https://mcp.wallbit.io/mcp` | API key en el header `X-API-Key` (créala solo con permisos de lectura y trading) | Tu agente | `wallbit` (ver [broker-wallbit.md](broker-wallbit.md)) |
| Berry (Latinoamérica; no EE.UU., Canadá ni UE) | Oficial, remoto: `https://connect.berry.app/mcp` | Inicio de sesión de Berry (OAuth) | Tu agente | `berry` (sin stops ni cancelación por API, como Wallbit; acciones de EE.UU. tokenizadas 1:1, puede no tener todas las acciones del universo) |
| InvertirOnline (IOL) | Oficial, remoto: `https://mcp.invertironline.com` | Usuario de IOL + 2FA | **Confirmas cada orden**; solo lectura por defecto (reconecta para habilitar la gestión) | `iol` |
| Interactive Brokers | Oficial, remoto: `https://api.ibkr.com/v1/api/mcp` | OAuth | **La envías tú desde una plataforma de IBKR**; el agente la prepara | `ibkr` |
| Moomoo | Oficial vía Moomoo OpenAPI (remoto con OAuth, o local con OpenD) | OAuth / login en OpenD | Tu agente (aprobación opcional) | `moomoo` |
| Alpaca | Oficial, **solo local** (`github.com/alpacahq/alpaca-mcp-server`, `uvx`); paper trading por defecto | API key + secret | Tu agente | `alpaca` |
| Robinhood (EE.UU.) | Oficial, remoto: `https://agent.robinhood.com/mcp/trading` | OAuth; una cuenta Agentic separada | Tu agente, solo en esa cuenta | `robinhood` |
| Public (mercado de EE.UU.) | Oficial, remoto: `https://mcp.public.com/mcp` | OAuth | Tu agente | `public` |
| Webull | Oficial (`github.com/webull-inc/webull-mcp-server`), local; sandbox por defecto | App key + secret | Tu agente, tras una vista previa | `webull` |
| tastytrade | Oficial, local (`github.com/tastytrade/tastytrade-mcp`) | Credenciales de la API | Tu agente, tras una simulación de la misma orden | `tastytrade` |

## Cómo correr un slot con el MCP del broker

1. Lee tu libro **con el MCP del broker** (cash, posiciones con costo promedio, órdenes abiertas).
2. Llama a `orbita_get_next_orders` con ese libro y `broker: { preset }`.
3. Envía cada orden **con el MCP del broker**, en orden de `step`, guardando cada `ref`.
4. Si tu broker exige a una persona (IOL, Interactive Brokers), muéstrale las órdenes con precios y deja que las confirme o envíe; nunca des por enviada una orden que no salió.
5. Confirma los fills con el MCP del broker y repórtalos con `orbita_report_execution`.

## Reglas

- Nunca pegues credenciales del broker en Órbita ni en el chat; configúralas solo en el MCP del broker.
- Empieza con una consulta de solo lectura (saldos) y, donde exista, con paper trading o una orden chica.
- Si las capacidades de un preset no coinciden con lo que acepta tu broker, pásalas explícitas (`supportsStopOrders`, `supportsOco`, `supportsCancel`, `fractional`).
