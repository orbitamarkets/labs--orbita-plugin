# Órbita plugin

Órbita's trading desk plan for your agent. Órbita's research desk follows 35 US-listed stocks in aerospace, defense, compute, energy and robotics and publishes Órbita Pulso, its short-term plan, through the US trading day. Your agent sends its book and its broker's capabilities and gets exact orders, in execution order, with stop loss and take profit adapted to what your broker supports.

Órbita is read-only: it never places orders and never receives broker credentials. Your agent executes at your own broker through the broker's own MCP, under your approval rules. General information, not personalized investment advice.

## What's inside

| Component | What it does |
|---|---|
| MCP server `orbita` | `https://mcp.orbita.markets` (Streamable HTTP, OAuth 2.1, scope `signals:read`) |
| Skill `orbita-strategy` | Órbita Pulso's mandate and risk rules (long only, −2 % stop, +3 % take profit, 10 % tickets, flat before weekends) |
| Skill `orbita-execution` | How to send the plan's orders at any broker, with per-broker notes |
| Skill `orbita-slot-routine` | When to check the plan during a market day |
| Skill `orbita-broker-setup` | How to pick a broker with an official MCP and connect it next to Órbita |

Every skill has a Spanish version under `i18n/es/`.

## Tools

`orbita_get_next_orders`, `orbita_get_briefing`, `orbita_report_execution`, `orbita_search_research`, `orbita_get_research_item`, `orbita_get_risk_calendar`, `orbita_get_ledger`, `orbita_get_skill`.

## Requirements

- An Órbita subscription at [orbita.markets](https://orbita.markets): monthly USD 9, 6 months USD 49 or annual USD 99 (the long plans add Órbita Horizonte). The skills are free.
- To execute: a broker with an official MCP, connected in your agent ([options](https://orbita.markets/brokers)).

## Network and credentials

- The only endpoint is `https://mcp.orbita.markets`. Sign-in is OAuth in your browser; the plugin stores no secrets.
- The plugin has no hooks, scripts or local executables.
- Never paste broker keys into the chat: they belong in your broker MCP's own configuration.

## Links

- Site: https://orbita.markets
- For agents: https://orbita.markets/agents
- llms.txt: https://mcp.orbita.markets/llms.txt
- Contact: info@orbita.markets

## License

MIT
