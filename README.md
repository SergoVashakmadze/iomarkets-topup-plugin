# IoMarkets Topup — Claude Code plugin

Buy real-world things for your principal and pay per order in **USDC on Algorand** via
[x402](https://x402.org): **travel eSIMs, and mobile airtime and data top-ups** delivered to a
phone number.

**Status: technology demonstration.** It has been tested on real data with real money in a limited
pilot, and it is not offered as a commercial service until the required licences, penetration testing
and security audits are complete. Payments are real: they settle USDC on Algorand mainnet.

No account, no card, no API key handed to anyone. Money settles on chain *before* anything is
bought, a delivery that fails is refunded on chain automatically, and every terminal order
carries an ed25519-signed receipt naming both transactions.

**Available in this pilot:** travel eSIMs, and airtime and data top-ups. `type: "bill"` and
`type: "payout"` are implemented end to end — same quote, same settlement, same signed receipt — but
each needs a supplier that is not currently wired, so they are **not** advertised.
[`GET /v1/catalog?type=<type>`](https://iomarkets.app/v1/catalog?type=topup) is the authoritative
answer and returns an empty list for anything unavailable.

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
