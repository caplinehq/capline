# Capline

### Cross-chain spend authority for AI agents.

**Jailbreak the model all you want. It still can't pay over its signed mandate.**

> The cap isn't in the prompt. It's a contract the LLM can't talk to.

**[▶ Live demo](https://capline-protocol.vercel.app)** · **[🤖 Agent gallery](https://capline-protocol.vercel.app/agents)** · **[📦 `npm i capline`](https://www.npmjs.com/package/capline)** · Solana · Avalanche · Stellar · x402 + AP2

This repo is the **protocol + the SDK**. The enforcement primitive is deployed natively on each chain; the [`capline`](https://www.npmjs.com/package/capline) npm package ([`sdk/`](sdk)) is the client that talks to all of them.

---

## The problem

`x402` gave agents the ability to **pay**. But the only thing telling an agent "never spend more than $5" is a sentence in a system prompt. **That is not a security control. The prompt is the attack surface.** One injection and your agent drains the wallet.

Capline moves the limit **out of the prompt and onto the chain.** A principal grants a bounded, revocable spend **mandate** to an agent, committed as a signed AP2 object the agent cannot renegotiate. Enforcement is in three layers:

- **Layer A** — an off-chain constrained signer that reads numbers, never language, and refuses to build an out-of-bounds payment.
- **Layer B** — an on-chain contract that reverts an out-of-mandate settle, even if the agent's key is stolen.
- **Cross-chain** — a coordinator enforcing one **global cumulative cap across every chain at once**, the invariant no single chain can see.

## Add it in 10 seconds

```ts
import { withCapline } from "capline/evm"; // or capline/solana

const xPayment = await withCapline({ signer, mandateId }).pay(paymentRequirements);
//   returns a signed x402 payment ONLY if it fits the on-chain mandate
//   throws MandateExceeded otherwise. the agent physically cannot overspend.
```

Also `capline/coordinator` (the cross-chain global cap) and `capline` (chains + AP2 hashing). Full docs in [`sdk/README.md`](sdk/README.md).

## Deployed, live, verifiable

| Chain | Enforcement primitive | Address |
|---|---|---|
| **Solana** (devnet) | Anchor program | [`DRNWDxtJ3P5hQCdGcmL3XXMW9NtnE345HTaWkk9dUhHp`](https://solscan.io/account/DRNWDxtJ3P5hQCdGcmL3XXMW9NtnE345HTaWkk9dUhHp?cluster=devnet) |
| **Avalanche** (Fuji) | `MandateRegistry.sol` | [`0x40367742b16c3DDa51B123751699032c5E446aF5`](https://testnet.snowtrace.io/address/0x40367742b16c3DDa51B123751699032c5E446aF5) |
| **Stellar** (testnet) | Soroban contract | [`CAXVTUT64BWTPDH3DBSUPAIWCF2WNLXWZVWLT54DPQFFXHAFRU67CCXE`](https://stellar.expert/explorer/testnet/contract/CAXVTUT64BWTPDH3DBSUPAIWCF2WNLXWZVWLT54DPQFFXHAFRU67CCXE) |

## Layout

| Path | What |
|---|---|
| [`sdk/`](sdk) | The published **`capline`** npm package. Subpath exports `capline`, `/coordinator`, `/evm`, `/solana`. |
| [`contracts/`](contracts) | Foundry (EVM). `MandateRegistry.sol` + minimal ERC-8004 `IdentityRegistry.sol` + tests. |
| [`solana/`](solana) | Anchor program. `create_mandate` / `settle` / `revoke` / on-chain ed25519 AP2 attestation. LiteSVM tests. |
| [`soroban/`](soroban) | Soroban contract (Stellar). Mirrors the primitive; deployed to testnet. |

## The rest of Capline

- **App** (landing + agent gallery + Control Room + coordinator): [`capline-app`](https://github.com/caplinehq/capline-app) · live at [capline-protocol.vercel.app](https://capline-protocol.vercel.app)
- **MCP server** (`npx capline-mcp`): [`capline-mcp`](https://github.com/caplinehq/capline-mcp)
- **Clone-and-run agent example**: [`capline-example`](https://github.com/caplinehq/capline-example)

Built on [x402](https://github.com/coinbase/x402) + [AP2](https://github.com/google-agentic-commerce/AP2) + [ERC-8004](https://eips.ethereum.org/EIPS/eip-8004). MIT.

> The model obeyed you. The chain didn't.
