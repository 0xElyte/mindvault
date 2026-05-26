# MindVault Contracts (Soroban)

Soroban smart contracts for MindVault. Today there is one:

## `vault-registry`

An on-chain registry of vault resources. It is the transparent source of truth
for **what** exists in the vault, **who** owns it, and **what it costs** —
anyone can read it directly from the chain without trusting the MindVault API.

Payments themselves do **not** run through this contract. They continue to flow
through x402 and the USDC Stellar Asset Contract (see the root README). The
registry complements that: the server settles payment via x402, and records /
reads the canonical resource entry here.

### Interface

| Function | Auth | Description |
|----------|------|-------------|
| `register(creator, id, price, metadata)` | creator | Register a new resource. Errors if `id` exists or `price <= 0`. |
| `set_price(id, new_price)` | creator | Update the price. |
| `update_metadata(id, metadata)` | creator | Update the metadata pointer (e.g. IPFS URI / content hash). |
| `transfer_ownership(id, new_creator)` | creator | Hand the resource to a new owner. |
| `get(id) -> Resource` | — | Read a resource. Errors `NotFound` if absent. |
| `exists(id) -> bool` | — | Whether a resource is registered. |
| `count() -> u32` | — | Total resources ever registered. |

`price` is an `i128` in USDC stroops (7 decimals — `1_000_000` = 0.10 USDC).
`id` is the resource's cuid2 string, matching the server's resource IDs.

### Develop

```bash
cargo test                                           # run unit tests
stellar contract build --manifest-path Cargo.toml    # build wasm
```

### Deploy (testnet)

```bash
# One-time: create & fund an identity
stellar keys generate deployer --network testnet --fund

stellar contract deploy \
  --wasm target/wasm32v1-none/release/vault_registry.wasm \
  --source deployer \
  --network testnet
```

The command prints the deployed contract ID — wire it into the server config so
the backend can record resources on registration.

### Ideas for contributors

- A `list` / pagination method (current `get` is by id only).
- Categories or tags stored alongside each resource.
- Optional escrow/refund extension (see the root README's "Not Yet Built").
- A TypeScript binding generated via `stellar contract bindings typescript`
  for the `server/` and `web/` packages to consume.
