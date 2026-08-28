# IoMarkets Topup — Claude Code plugin

Buy real-world things for your principal and pay per order in **USDC on Algorand** via
[x402](https://x402.org): mobile airtime and data top-ups, travel eSIMs, prepaid bills, and
bank / mobile-money / UPI payouts across 150+ countries.

No account, no card, no API key handed to anyone.

## Install

```
/plugin marketplace add SergoVashakmadze/iomarkets-topup-plugin
/plugin install iomarkets-topup
```

## What you get

- The **`iomarkets-topup` skill** — teaches the agent when and how to buy.
- The **hosted MCP server** at `https://iomarkets.app/mcp`, declared by the plugin. Nothing runs
  locally and nothing is cloned.

## How paying works

The server holds **no wallet**. A `buy` call returns an x402 `402 Payment Required` challenge and
the *caller* signs the USDC transfer from its own wallet — so no key is ever handed over, and the
agent cannot spend anything its principal has not authorised.

Every order settles on-chain **before** delivery is attempted. Each delivered *or* refunded order
carries an ed25519-signed receipt naming its on-chain settlement, and a delivery that fails is
refunded on-chain automatically.

Docs for agents: <https://iomarkets.app/agent.md>

## This repository

A public mirror of the plugin surface only — `.claude-plugin/` and `skills/`. The plugin points at
the hosted endpoint, so the server source is not needed here and is not published.
