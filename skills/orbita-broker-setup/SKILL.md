---
name: orbita-broker-setup
description: Help a person choose a broker with an official MCP and connect it next to Órbita, step by step. Use it when the person has no broker yet, their broker isn't connected to you, or they ask which broker to use. Spanish version: i18n/es/SKILL.md.
---

# Choosing and connecting a broker

Órbita tells you **what** to do; the broker's own MCP server lets you **do it**. You need both connected. Órbita never trades and never receives broker credentials.

This is general information to help the person set up tools, not a recommendation of a broker for their personal situation. Features and availability change: confirm on the broker's official site before acting.

## 1. Ask three things (one message)

1. **Where do you live?** (country of residence determines which brokers can open an account for you)
2. **Which app do you use me in?** An agent that accepts API-key headers and local MCPs (Grok Bot, OpenCode, OpenClaw, Claude Code, Cursor), or a chat app on the web/mobile (ChatGPT, Claude)?
3. **Do you want me to place orders on my own, or confirm each one yourself?**

If they already have a broker, skip to step 3 with that broker.

## 2. Suggest options

Use the table in [orbita-execution/references/brokers.md](../orbita-execution/references/brokers.md) (call `orbita_get_skill` with `name: "orbita-execution"`, `file: "references/brokers.md"`). Filter with the answers:

| Situation | Suggest |
|---|---|
| Argentina or Latin America, Grok Bot / OpenCode (or another agent that accepts headers) | **Wallbit** (US stocks in USD, covers the whole universe; hosted MCP with an API key in a header) |
| Argentina, a chat app on the web (ChatGPT, Claude) | **InvertirOnline (IOL)**: hosted MCP with IOL login; you confirm every order. Or use Grok Bot (orbita.markets/grok) with Wallbit. |
| Latin America, any app (also ChatGPT or Claude on the web) | **Berry**: hosted MCP with Berry sign-in; tokenized US stocks 1:1 from $1. It may not list every stock in the universe. |
| Rest of the world | **Interactive Brokers** (you submit orders in IBKR) or **Moomoo** (agent trades, optional approval). If they live in the US, also **Robinhood** (separate Agentic account), **Public**, **Webull** and **tastytrade**. **Alpaca** needs an agent that runs local MCPs (Grok Bot on its cloud computer, OpenCode, Claude Code, Cursor). |
| Wants to confirm each order | IBKR, IOL, or any broker with approval turned on |

Explain the trade-offs in one or two lines each. Let the person choose.

## 3. Walk them through connecting it

Follow the broker's steps in the reference and the guide at https://orbita.markets/brokers (Spanish: https://orbita.markets/es/brokers). In general:

1. Open the account at the broker (you can't do this for them).
2. Create credentials with the **minimum permissions**: read and trade. No transfers, withdrawals or card management.
3. Add the broker's MCP to their app. Web chat apps (ChatGPT, Claude) need hosted MCPs with sign-in; API-key headers and local MCPs need an agent such as Grok Bot (it stores the header on the connection and runs local MCPs on its cloud computer) or OpenCode.
4. Keep Órbita's MCP connected too (https://mcp.orbita.markets).

**Never ask for, read out or store passwords or API keys in the chat.** They go only into the broker's MCP configuration or its sign-in screen.

## 4. Check before the first trade

1. With the broker's MCP: read balances and positions (read-only).
2. With Órbita: call `orbita_get_next_orders` with that book and `broker: { preset }` (the preset from the reference; `generic` with explicit capabilities otherwise). Show the person what Órbita would do, without sending anything.
3. First execution: paper trading if the broker has it, or the smallest order the plan allows, with the person's explicit OK.
4. Then read orbita-slot-routine to schedule the slots.
