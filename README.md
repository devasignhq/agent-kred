# AgentKred

Handle payments for long-running tasks from x402 agents through our escrow system, and use task outcomes as a credibility layer for those agents.

## How it works

```
Escrow settles ──▶ Registry contract (Soroban) ──▶ Scoring library ──▶ MCP tool / x402 catalogs
                         │
                         └──▶ Stellar 8004 mirror (feedback entry linking back to the record)
```

- An approved escrow source settles a job as accepted, refunded, or disputed.
- The escrow (or the adapter, after verifying final state on-chain) writes a record to the registry.
- The record is mirrored to the Stellar 8004 reputation registry.
- The scoring library reads records from contract events.
- Agents query scores through the MCP tool or see them inside x402 catalogs.

## Record format

Follows ERC-8004 identity and reputation registries. Job states follow ERC-8183.

| Field | Description |
|---|---|
| `outcome` | `accepted` \| `refunded` \| `disputed` |
| `client` | Paying party's Stellar address |
| `worker` | Worker's Stellar address |
| `amount` | Settled amount |
| `asset` | Asset code (USDC) |
| `escrow_ref` | Reference to the settled escrow |
| `evidence` | Link to proof of work, e.g. merged PR |

Only approved escrow sources can write. Records are immutable.

## Components

| Component | Stack | Description |
|---|---|---|
| `registry` | Soroban (Rust) | Identity and reputation registries |
| `adapter` | Soroban | Converts settled escrows into records after on-chain state check |
| `mirror` | Soroban | Writes each record as feedback to Stellar 8004 |
| `scoring` | TypeScript | Open-source library: reads events, groups wallets by funding account, computes scores |
| `mcp` | TypeScript | MCP tool for agents to check a seller before paying |

## Integration levels

Catalogs choose how much to adopt. Records match listings by the seller's wallet address.

| Level | What changes |
|---|---|
| Check | Nothing in the catalog. Agent queries Tally before paying. |
| Show | Record appears next to a listing. Ranking unchanged. |
| Rank | Record becomes one input to the catalog's credibility score. |

## Escrow sources

- DevAsign (live on mainnet, first source)
- Trustless Work (planned)
- Offer Hub, Drips Wave (same adapter)

## License

This project is licensed under the Apache 2.0 License. See [LICENSE](https://github.com/devasignhq/agentkred/blob/main/LICENSE) for more details.
