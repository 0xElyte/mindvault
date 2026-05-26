# Contributing to MindVault

Thanks for your interest in building MindVault! This guide gets you from a fresh
clone to a running stack and a first pull request.

MindVault is a payment-protected vault for digital resources on Stellar, using
HTTP 402 and the x402 protocol. Everything runs on **Stellar testnet** — no real
funds are at risk.

## Repository layout

```
mindvault/
  server/     Express backend, x402 middleware, Supabase, verification agent
  web/        React frontend, Stellar wallet connection, Tailwind   (imported separately)
  mcp/        MCP server for AI agent access                        (imported separately)
  contract/   Soroban smart contracts (Rust) — on-chain vault registry
```

This is a **pnpm workspace** for the JavaScript/TypeScript packages. `contract/`
is a separate Rust/Cargo workspace managed with the Stellar CLI.

## Prerequisites

- **Node.js 20+** and **pnpm** (`npm i -g pnpm`)
- **Rust** + the **Stellar CLI** for contract work — see
  [Stellar setup](https://developers.stellar.org/docs/build/smart-contracts/getting-started/setup)
- A **Supabase** project (free tier) for the backend
- Stellar testnet wallets funded with USDC from [faucet.circle.com](https://faucet.circle.com)

## First-time setup

```bash
git clone <your-fork-url> mindvault && cd mindvault

# Install all JS/TS workspace packages at once
pnpm install

# Configure the backend
cp server/.env.example server/.env
# Fill in Supabase, Stellar, and OpenRouter credentials.
# NEVER commit server/.env — it is gitignored and holds secret keys.

# Database
pnpm db:generate && pnpm db:migrate

# Generate wallets (run twice for separate platform + agent wallets)
pnpm generate-wallet
```

## Running

From the repo root:

```bash
pnpm dev:server      # backend on :4021
# pnpm dev:web       # frontend on :5173 (once web/ is imported)
```

## Smart contract development

```bash
pnpm contract:build  # build the wasm (stellar contract build)
pnpm contract:test   # cargo test

# or directly inside contract/
cd contract && cargo test
```

See [`contract/README.md`](contract/README.md) for the registry interface and
deployment steps.

## Working on a change

1. **Fork** and create a branch: `git checkout -b feat/short-description`
2. Keep changes focused — one logical change per PR.
3. Make sure things build/pass before pushing:
   - Backend: `pnpm build:server`
   - Contract: `pnpm contract:test`
4. Use clear commit messages (e.g. `feat: add catalog search`, `fix: cors header`).
5. Open a PR against `main` describing **what** changed and **why**, and how you
   tested it.

## Good first issues

These are drawn from the README's "What Is Not Yet Built" — great starting points:

- **Search & filtering** on the catalog (`server/` + `web/`)
- **Recurring access / time-limited leases** instead of per-request payment
- **Refund mechanism** (potentially a Soroban escrow contract under `contract/`)
- **Rate limiting** on the API
- **Wire the backend to `vault-registry`** — record resources on-chain at publish
  time and read prices from the registry
- **TypeScript bindings** for the contract via `stellar contract bindings typescript`

If you want to take something larger on, open an issue first so we can align on
the approach.

## Security

- Never commit secrets. Only `*.env.example` files belong in git.
- All payments are testnet only; do not point this at mainnet.
- Found a vulnerability? Open an issue describing the impact (avoid posting a
  working exploit publicly).

Happy building. ⚡
