# AiFinPay — Integration Guide for AI Agent Builders

AiFinPay provides non-custodial payment and identity primitives for autonomous
agents through the SDK, MCP and HTTP 402 flows.

## Protocol map

| Protocol | Responsibility | Economics |
|---|---|---|
| **AIFP-1** | Merchant paywalls, quotes, settlement and metered receipts | payer pays quoted gross; merchant 99%; AiFinPay 1%; creator 0% |
| **AIFP-2 / x402** | Provider HTTP 402 negotiation and payment transport | provider 100%; AiFinPay 0%; creator 0%; gas separate |
| **AIFP-3** | Agent Passport identity/attestation | no settlement economics |

AIFP-1 is gross-inclusive; nothing is added on top. AIFP-2/x402 has a temporary
0% AiFinPay protocol fee for the current launch period.

## MCP

```json
{
  "mcpServers": {
    "aifinpay": {
      "command": "npx",
      "args": ["@aifinpay/mcp"],
      "env": {
        "AIFINPAY_AGENT_SECRET": "<persistent local secret>",
        "AIFINPAY_MAX_USD": "0.50"
      }
    }
  }
}
```

| Tool | Status |
|---|---|
| `agent_address` | Read-only local identity. |
| `agent_quote` | Read-only challenge inspection. |
| `payable_fetch` | Fund-moving only when all target/value/runtime gates pass. |
| `agent_call` | Registry-resolved provider call with the same gates. |
| `pay_with_split` | Retired compatibility tool; moves no funds. |
| `quote_split` | Retired compatibility tool; use a canonical AIFP-1 quote. |

## SDK

```bash
pip install aifinpay-agent
npm install @aifinpay/agent
```

Use `SettlementClient` for canonical v1.3 AIFP settlement. Before a wallet
signs, validate the invoice and independently verify the trusted route pin:

- route class and chain ID;
- splitter v1.3 address and exact runtime hash;
- on-chain BPS profile (AIFP-1 `100/0`, AIFP-2 `0/0`);
- merchant/provider target and asset metadata;
- quote expiry and exact gross calldata;
- operator USD ceiling and trusted asset price.

A backend response cannot be its own trust anchor.

## Production boundary

A merged contract, address, logo, explorer link or historical transaction does
not prove that a current route is payment-live. Activation additionally
requires a clean package build, controlled paid E2E for the exact release and no
unresolved Critical/High fund-loss audit issue. Otherwise the SDK/MCP path must
fail closed.

## Agent Passport

AIFP-3 is the Agent Passport identity layer. Identity ownership, attestations
and reputation can be referenced during a payment flow, but AIFP-3 does not
mint a payment receipt and does not change AIFP-1/AIFP-2 economics.

## References

- Protocols: `github.com/AiFinPay/AIFP-1`, `AIFP-2`, `AIFP-3`
- SDK/MCP: `github.com/AiFinPay/sdk`
- Documentation: `github.com/AiFinPay/docs`
- Canonical domain: `https://aifinpay.io`

MIT © AiFinPay
