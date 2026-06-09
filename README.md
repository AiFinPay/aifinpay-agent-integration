# AiFinPay × AutoGPT — B2B Integration Guide

**Give your agents a wallet.** AiFinPay is the payment rail for autonomous
AI agents — *Stripe for agents*. One line of code lets an AutoGPT agent
hold a wallet, pay for any service per call, and settle a real on-chain
payment (Polygon / Solana mainnet). Non-custodial, no KYC, no API key,
no custodian.

This guide shows the AutoGPT team the three ways to integrate, leading
with the one that fits AutoGPT best: **MCP**.

---

## Why this fits AutoGPT

AutoGPT agents run unattended and call paid services (inference, search,
vector DBs, APIs). Today each of those needs a human's API key and credit
card. With AiFinPay, the **agent itself** carries a wallet and pays
per call — exactly the "Stop building workflows, start hiring agents"
vision, with the missing piece (money) filled in.

The agent gets: its own address, a USDC balance, a monthly spend limit,
and a transaction history of what it spent on which service — the wallet
panel you sketched drops straight onto this.

---

## Option 1 — MCP (recommended, zero-code)

AutoGPT (and Claude Desktop, Cursor, Windsurf, Continue, Cline) speak
**Model Context Protocol**. Add one block and every agent gains five
payment tools automatically — no SDK calls to wire by hand.

```json
{
  "mcpServers": {
    "aifinpay": {
      "command": "npx",
      "args": ["@aifinpay/mcp"]
    }
  }
}
```

The model now has these tools:

| Tool | What it does |
|---|---|
| `agent_address` | Returns the agent's wallet address (to fund / display) |
| `agent_quote` | Price-checks an x402-gated URL before paying |
| `payable_fetch` | Fetches a URL; auto-settles the 402 challenge on-chain |
| `pay_with_split` | Direct B2B payment with the on-chain split |
| `quote_split` | Previews the split (merchant / treasury / fee) |

That's the whole integration. The agent can now autonomously pay any
x402-gated API and get the gated response back.

---

## Option 2 — Python / Node SDK (programmatic)

For deeper control inside AutoGPT's own code:

```bash
pip install aifinpay-agent --pre      # Python
npm install @aifinpay/agent@alpha     # Node / TypeScript
```

**The "agent pays in a few lines" example** — this is the core of the
whole product:

```python
from aifinpay import Agent

agent = Agent.new()                       # fresh wallet/keypair
print("Fund this address:", agent.address)

# Autonomously settles the 402 challenge on-chain, returns the response
resp = agent.pay(
    "https://bridge.aifinpay.company/io-net/chat/completions",
    body={"model": "meta-llama/Llama-3.3-70B-Instruct",
          "messages": [{"role": "user", "content": "Hello"}]},
)
print(resp.json()["choices"][0]["message"]["content"])
print("tx hash:", resp.headers.get("x-payment-receipt"))
```

Node is identical: `Agent.new()` → `agent.pay(url, { body })`.

Persist `agent.secret_b58` to give an agent a durable identity across
restarts (`Agent.from_secret(...)`), and read its balance with
`agent.balance(asset="USDC", chain="polygon")` — that's what powers the
wallet panel (balance + spend limit + history).

---

## Option 3 — Drop-in AutoGPT loop (already written)

There is a **ready-made AutoGPT example** in the SDK: a headless,
self-funding agent that funds itself once, then buys inference + search
calls in a perpetual loop until its balance hits a floor.

```
github.com/AiFinPay/sdk → examples/autogpt/loop.py
```

```bash
pip install aifinpay-agent --pre openai
export OPENAI_API_KEY=sk-...
python loop.py        # prints an address to fund, then runs unattended
```

This is the canonical pattern for any headless agent operating on a fixed
budget — the AutoGPT team can run it as-is to see the full lifecycle.

---

## What happens under the hood (settlement)

```
agent → SDK / MCP → x402 challenge (HTTP 402) → agent signs + pays
      → on-chain atomic split → gated response returned to agent
```

- Settlement **is** the transaction — non-custodial, no intermediate hold.
- Every successful call emits one on-chain `Payment` event on the verified
  `B2BSplitter` contract (Polygon mainnet).
- The protocol fee (1%) is collected automatically by the splitter —
  no revenue-share bookkeeping, no contract to sign.

### Webhooks (real-time events)

Partners subscribe a URL to events and receive HMAC-signed deliveries
with automatic retries:

- Events: `payment.intent`, `payment.completed`, `seat.reserved`,
  `b2b.pay_with_split.invoice`
- `POST /api/webhooks` to subscribe; each delivery is signed
  (`x-aifinpay-signature: t=<ts>,v1=<hmac>`) — verify with the shared
  secret, timing-safe.

---

## Becoming a paid service (the other side)

If AutoGPT wants to *sell* a service to agents (not just spend), they can
stand up a paid bridge in ~30 minutes and earn per call — full steps in
the SDK's `PARTNER_ONBOARDING.md`. They keep their API key and upstream
relationship; AiFinPay handles the payment rail and agent identity.

---

## Reference links

| Resource | Where |
|---|---|
| SDK + all examples | `github.com/AiFinPay/sdk` |
| Quickstart (60-second first paid call) | `QUICKSTART.md` |
| MCP client matrix | `MCP_CONFIG.md` |
| AutoGPT example | `examples/autogpt/` |
| Partner onboarding (sell a service) | `PARTNER_ONBOARDING.md` |
| Live site | `aifinpay.company` |

---

## Suggested next step with AutoGPT

1. They add the MCP block (Option 1) — instant payment tools.
2. They run `examples/autogpt/loop.py` to see a self-funding agent live.
3. Follow-up call to design the embedded wallet UI in their interface
   (balance + spend limit + per-service history) on top of the MCP tools.
