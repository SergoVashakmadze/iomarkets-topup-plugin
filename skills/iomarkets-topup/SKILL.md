---
name: iomarkets-topup
description: Buy real-world things and send international payments for your principal with USDC on Algorand via x402 — mobile airtime/data top-ups, travel eSIMs, prepaid bills, and bank / mobile-money / UPI payouts in 150+ countries. Use when the user asks to recharge/top up a phone, buy mobile data or an eSIM, pay a prepaid bill, send money abroad (to a bank account, M-Pesa, GCash, UPI), check a USDC→local FX rate, or when an agent needs to purchase connectivity for itself. No account, no card; signed proof of delivery; automatic on-chain refunds.
version: 0.1.0
metadata:
  homepage: https://iomarkets.app
  agent_docs: https://iomarkets.app/agent.md
  tags: [x402, algorand, usdc, topup, airtime, esim, bills, agentic-commerce]
---

# IoMarkets Topup

Real-world checkout for agents. Four HTTP calls, or the `iomarkets-topup` MCP server (`pnpm mcp` in the repo).

## When to use
- "Top up / recharge +91…, +234…, +995… with N (local currency)"
- "Buy me an India / Türkiye / global eSIM for my trip"
- "Pay my prepaid electricity"
- "Send ₦20,000 to this account / 5,000 KES to this M-Pesa number / ₹2,000 to this UPI id" (international payment)
- "What's the USDC→INR rate?"
- The agent itself needs connectivity it can pay for.

## Procedure
1. **Discover** — `GET https://iomarkets.app/v1/lookup?phone=<E.164>` (operator + offers) or `GET https://iomarkets.app/v1/catalog?type=esim&country=IN`.
2. **Quote** — `POST https://iomarkets.app/v1/quote` with `{type, offerId, recipient:{phone}, amount}` (amount in the recipient's currency for range offers; omit for fixed bundles/eSIMs). You get `quoteId`, `price_usdc`, `delivers`, `expires_at`.
   For **international payments** (`type: "payout"`): `recipient.fields` must contain every key the offer's `requiredFields` lists, `sender: {name, country}` is the principal (ask if unknown; never invent), pass `payer` (your wallet address), and above $100/day the partner KYC `sender.reference` is required — tell the human to complete the partner's verification and give you the reference. `GET /v1/fx?to=NGN&amount=20000` gives an estimate first.
3. **Confirm with the human** — state exactly: *what* is delivered, *to whom* (number), and the *USDC price*. Never skip this for a purchase.
4. **Pay** — `POST https://iomarkets.app/v1/orders` with `{ "quoteId" }` using your x402 client (Algorand USDC, `exact` scheme). First response is 402 with the exact amount; retry with the payment signature. 202 → order.
5. **Poll** — `GET https://iomarkets.app/v1/orders/<orderId>` every 3 s until `terminal: true`. Report `status`, the `confirmation` (operator reference / eSIM LPA + install steps) and the `settlement_url`.
6. **If refunded** — tell the human the money is back at their address (`refund_url`) and offer to retry with another offer.

## Rules
- One quote = one payment. Quotes expire in 10 minutes; re-quote instead of retrying an expired one.
- Respect the wallet budget you were given; orders are capped at $50 and $200/payer/day server-side.
- Money transfers are irreversible once delivered: read the recipient details back to the human verbatim before `buy`.
- Keep the receipt (`order.receipt`) — it is the proof of delivery, verifiable with `GET /v1/pubkey`.
- Do not retry a `buy` on a timeout without first checking `order_status`; the settlement may have succeeded.

## Install as MCP (Claude Code / Codex / Cursor / Hermes / OpenClaw)

**Hosted — nothing to install, no key given to anyone:**
```json
{ "mcpServers": { "iomarkets-topup": { "url": "https://iomarkets.app/mcp" } } }
```
This server holds no wallet, so `buy` returns the x402 payment challenge (exact amount, asset, `payTo`,
facilitator) for **you** to pay from your own Algorand wallet, then you re-POST the quote with your payment
signature. Everything else — lookup, catalog, quote, order status, receipt verification, FX, ledger — works as-is.

**Local — the server pays for you, under a budget:**
```json
{ "mcpServers": { "iomarkets-topup": { "command": "pnpm", "args": ["--dir", "/path/to/iomarkets-app", "mcp"],
  "env": { "API_URL": "https://iomarkets.app", "AGENT_MNEMONIC_FILE": "/home/you/.secrets/agent.mnemonic", "AGENT_BUDGET_USD": "20" } } } }
```
Here `buy` signs and settles itself from the agent wallet, capped by `AGENT_BUDGET_USD` per session and
`AGENT_MAX_ORDER_USD` per order. Keep the mnemonic in a file (mode 0400) — never inline in the config.
