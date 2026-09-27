# Brokers with an official MCP

Annex to the `orbita-execution` skill. Your agent needs **two MCP servers**: Órbita's (the plan) and your broker's (execution). Órbita never receives broker credentials: they go only to the broker's MCP. Use the `preset` below in `orbita_get_next_orders`. Details change; always check the broker's official documentation.

| Broker | MCP | Access | Who sends the order | `preset` |
|---|---|---|---|---|
| Wallbit | Official, hosted: `https://mcp.wallbit.io/mcp` | API key in the `X-API-Key` header (create it with read and trade permissions only) | Your agent | `wallbit` (see [broker-wallbit.md](broker-wallbit.md)) |
| Berry (Latin America; not US, Canada or EU) | Official, hosted: `https://connect.berry.app/mcp` | Berry sign-in (OAuth) | Your agent | `berry` (no stops or API cancels, like Wallbit; tokenized US stocks 1:1, may not list every stock in the universe) |
| InvertirOnline (IOL) | Official, hosted: `https://mcp.invertironline.com` | IOL login + 2FA | **You confirm every order**; read-only by default (reconnect to enable management) | `iol` |
| Interactive Brokers | Official, hosted: `https://api.ibkr.com/v1/api/mcp` | OAuth | **You submit it from an IBKR platform**; the agent drafts it | `ibkr` |
| Moomoo | Official via Moomoo OpenAPI (hosted with OAuth, or local OpenD) | OAuth / OpenD login | Your agent (optional approval) | `moomoo` |
| Alpaca | Official, **local only** (`github.com/alpacahq/alpaca-mcp-server`, `uvx`); paper trading by default | API key + secret | Your agent | `alpaca` |
| Robinhood (US) | Official, hosted: `https://agent.robinhood.com/mcp/trading` | OAuth; a separate Agentic account | Your agent, only in that account | `robinhood` |
| Public (US markets) | Official, hosted: `https://mcp.public.com/mcp` | OAuth | Your agent | `public` |
| Webull | Official (`github.com/webull-inc/webull-mcp-server`), local; sandbox by default | App key + secret | Your agent, after a preview | `webull` |
| tastytrade | Official, local (`github.com/tastytrade/tastytrade-mcp`) | API credentials | Your agent, after a dry run of the same order | `tastytrade` |

## How to run a slot with a broker MCP

1. Read your book **from the broker's MCP** (cash, positions with average cost, open orders).
2. Call `orbita_get_next_orders` with that book and `broker: { preset }`.
3. Send each order **with the broker's MCP**, in `step` order, keeping each `ref`.
4. If your broker requires a human (IOL, Interactive Brokers), show the person the orders with prices and let them confirm or submit; never pretend an order was sent.
5. Confirm fills with the broker's MCP and report them with `orbita_report_execution`.

## Rules

- Never paste broker credentials into Órbita or into the chat; configure them only in the broker's MCP.
- Start with a read-only check (balances) and, where available, paper trading or a small order.
- If a preset's capabilities don't match what your broker accepts, pass them explicitly (`supportsStopOrders`, `supportsOco`, `supportsCancel`, `fractional`).
